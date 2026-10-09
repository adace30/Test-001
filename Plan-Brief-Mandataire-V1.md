# Plan de réalisation — Brief Mandataire, Version 1

*Établi d'après le document de spécification « 24.1 Immo ». Les renvois entre parenthèses (4.2, 11.1, P02…) désignent les sections et les points du chapitre 17 de ce document.*

---

## 0. Le périmètre en une page

**Ce qu'on construit en Version 1**

1. **Prise d'appel**, par renvoi conditionnel vers un numéro technique : occupé, non-réponse ou injoignable, plus le renvoi immédiat en mode « Ne pas déranger » (ch. 6).
2. **Aiguillage** : Liste Noire, Liste Blanche, annonce IA, puis filtre d'intention à 3 branches (4.2, 4.3, 13.2).
3. **Qualification vocale** sur 3 scénarios (vente, estimation, achat) : 5 questions au maximum, un socle et des questions d'ajustement, l'économie de question et un tour de clôture (7.1, ch. 8, ch. 9).
4. **Brief structuré** : ligne d'identification, zone opérationnelle, 12 champs, marques de provenance A/B/C, marque de récurrence (11.1 à 11.3, 16.5).
5. **Suivi du prospect** : 6 états, bouton « Rappeler », saisie du résultat en un geste (11.4).
6. **Mesure** : référentiel d'indicateurs calculé par requêtes, sans écran, et bilan mensuel calculé à la demande (10.4, 10.6).
7. **État et test du dispositif**, guide par opérateur (6.1 à 6.3).
8. **Abonnement** : Stripe sur le web uniquement, essai de 30 jours, forfait de 500 minutes avec alerte à 80 % (16.6).
9. **Programme Ambassadeur** : lien et code, invitation par partage natif, compteurs, avantage à 3 filleuls comptabilisés, cascade, préavis (16.1, 16.2, 16.6).
10. **Pilote** : 3 à 5 parrains, 60 jours (ch. 18).

**Ce qu'on ne construit pas**, décision ferme : aucune relance du prospect, aucune intégration CRM ou back-office (y compris en V2), pas de location, pas de mémoire commerciale (V2), pas de Radar, de visibilité ni de miroir (V1.1), pas de tableau de bord d'indicateurs (V1.1), pas d'achat automatique de minutes (V1.1), pas de plages d'indisponibilité (V1.1).

**Principe d'architecture qui commande tout le plan**

> **Le modèle comprend, le code décide.** Claude extrait et rédige dans un schéma imposé. La priorité, l'alerte, le délai, les états, les indicateurs, le comptage des filleuls et les droits sont calculés en **code déterministe** (ch. 5, 11.2).

---

## 1. Préalables bloquants, à lancer dès la semaine 1

Ces points ne relèvent pas du développement, mais le lancement ne peut pas se faire sans eux (ch. 17). On les lance en parallèle du développement.

| Point | Objet | Ce qu'il bloque | Qui |
|---|---|---|---|
| **T01** | ~~Choix du fournisseur voix~~ → **tranché : Twilio ConversationRelay**. Reste : codes de renvoi par opérateur (Orange, SFR, Bouygues, Free), iOS/Android, double-SIM/eSIM, délai de sonnerie recommandé, codes « Ne pas déranger » | Lot 1, guide opérateur (6.3), test du dispositif | Tech + rédaction |
| **C02** | Rôles RGPD entre mandataire et réseau, responsable du rattachement parrain–filleul, **avis d'avocat sur la « vente à la boule de neige »** (16.6) | Lancement, programme ambassadeur, conservation | Juriste / avocat |
| **R12** | Durée de conservation des valeurs de niveau A et de la maturité, avec leur base juridique | La conservation du 16.9.9 (le code est prêt, mais désactivé tant que le point n'est pas tranché) | Juriste |
| **O02** | Tarif (49 €/mois est indicatif) et conditions de l'essai | Lot 7 (Stripe) | Fondateur |
| **O03** | Valeurs de lancement : preuve valable 30 j, `AVOIDED_CALLBACK_MINUTES` = 5, seuil 3, forfait 500 min, préavis 30 j, J+14, prix des packs vendus à la main en V1 (**non fixé**), seuil de récurrence R11, critères chiffrés de sortie de bêta (18.5) | Paramètres, pilote | Fondateur |
| **C01 / C03** | Compatibilité du socle avec les règles des réseaux ; mentions légales de l'annonce IA | Script d'annonce, page de souscription | Juriste |

**Spécifications produit à écrire avant de coder le module concerné :**

| Point | Contenu | Requis avant |
|---|---|---|
| **Ch. 9** | **Scénarios estimation et achat** — **rédigés par toi**, à rendre **fin S2** (gabarit en annexe A) : écrans, socle, ordre d'ajustement, regroupements à 4 options, éléments de priorité, signaux d'alerte. Le socle d'achat doit contenir la référence du bien et la demande de visite | Lot 2 (sinon ces appels sortent en priorité normale par défaut) |
| **P02** | Critères d'arrêt : refus, silence prolongé, demande de rappel | Lot 2 |
| **P06** | Règle d'affectation des 4 délais recommandés | Lot 3 |
| **P05** | Traçage de l'ouverture du brief | Lot 4 |
| **P09** | Contrôle de volumétrie, d'ancrage du niveau B et de correspondance stricte des attentes | Lot 3 |
| **P03 / P08 / P14 / R17** | Écrans de l'espace mandataire, rendu des marques de provenance, rendu du bilan, emplacement de la marque de récurrence | Lot 4 |
| **T04** | Import des contacts dans la Liste Blanche | Lot 4 |
| **A04 / A05 / R05 / R10** | Rattachement tardif ; résiliation ; changement de réseau ; résiliation en chaîne | Lot 8 |

---

## 2. Socle technique retenu (ch. 5)

| Brique | Choix | Remarque |
|---|---|---|
| Voix | **Twilio ConversationRelay** (tranché) — numéros techniques français et numéro sortant distinct pour le test du dispositif via l'API Twilio | Reconnaissance vocale, synthèse et interruptions sont gérées par le fournisseur. Le moteur reçoit du texte et renvoie du texte |
| IA | API Anthropic, **sorties structurées** (schéma JSON du brief) | Modèle le plus récent disponible. Deux usages : le dialogue (classification, extraction tour par tour) et la rédaction du brief |
| Back-end | Service managé : authentification, Postgres, tâches planifiées (ex. Supabase) | Webhooks voix, webhooks Stripe, tâches J+14, J+30, mensuelles |
| Front | Espace mandataire **web**, encapsulé dans une application **Median** (App Studio, pont JS, partage natif) | Les changements du site arrivent dans l'app sans republication |
| Notifications | OneSignal par Median : `register()`, `login(id)`, `logout()` | Aucun jeton stocké chez nous |
| Paiement | Stripe Checkout et portail client hébergés, **web uniquement** | Aucun prix dans l'app (App Store 3.1.3(f)) |
| Observabilité | Sentry, journaux des appels | Serveurs MCP Stripe, Postgres et Sentry pour le développement (de confiance uniquement) |

**Constantes de plateforme**, dans une table de configuration, sans écran d'administration :
`TRIAL_DURATION_DAYS=30`, `GRACE_PERIOD_DAYS=30` (contrainte vérifiée : ≥ essai), `LINK_ACTIVATION_DELAY_DAYS=0`, `AVOIDED_CALLBACK_MINUTES=5`, `PILOT_DURATION_DAYS=60`, `PROOF_VALIDITY_DAYS=30`, `TRIAL_CHECK_DAY=14`, `INCLUDED_MINUTES=500`, `MINUTES_ALERT_RATIO=0.8`, `FREE_THRESHOLD=3`.
Les paramètres propres au mandataire : `IDENTITY`, `ANSWERING_MODE=FALLBACK_UNAVAILABLE`, `NON_BUSINESS_ACTION=SIMPLE_VOICEMAIL` (valeur unique).

**Scénarios** : un fichier de configuration par scénario (écrans, options, regroupements vocaux, catégorie socle/ajustement, ordre de priorité, éléments de priorité, signaux d'alerte). On peut les modifier sans toucher au code (ch. 9).

---

## 3. Modèle de données (première version)

| Table | Champs clés | Règles |
|---|---|---|
| `agent` (mandataire) | identity, mobile, numéro technique, network, trial_started_at, subscription_status, referral_code, link_activated_at, pilot_link_override (case cochée par l'équipe, 18.2) | Suppression de compte possible depuis l'app (exigence des stores) |
| `number_list` | agent_id, numéro, type WHITE/BLACK, libellé | Données personnelles : consultables, modifiables et supprimables à tout moment |
| `call` | agent_id, from_number, masked, received_at, ended_at, path (`TEST`/`WHITE`/`BLACK`/`HANDLED`), branch (`TARGET`/`OUT_OF_SCOPE`/`NON_BUSINESS`), billable_minutes | Les appels `TEST` sont exclus de tout effectif. Seuls les appels `HANDLED` sont facturables |
| `device_test` | agent_id, protocol (principal/secours), started_at, result OK/KO | Déclenché uniquement par le mandataire, date conservée |
| `message` | call_id, appelant, heure, motif | Liste Blanche et hors-métier (`SIMPLE_VOICEMAIL`) |
| `contact_card` | call_id, coordonnées, objet | Hors périmètre. N'est jamais un prospect |
| `prospect` | call_id, state, priorité, délai recommandé | **Un appel `TARGET` donne un prospect**, sans fusion |
| `prospect_state_event` | prospect_id, from, to, at | Les **états atteints** servent au calcul des indicateurs (10.4) |
| `brief` | prospect_id, json (schéma), complete, produced_at, first_opened_at | **Immuable** une fois produit |
| `brief_value` | brief_id, champ, valeur, provenance A/B, collected_at | Conservation du 16.9.9. **Désactivée tant que R12 n'est pas tranché** |
| `referral` | filleul_id, parrain_id, attached_at | Créé uniquement à l'inscription, avec un lien déjà actif. **Jamais rétroactif** |
| `share_event` | agent_id, at | Compte seulement, sans destinataire |
| `subscription` | stripe ids, statut, remise 100 %, grace_until | Comptabilisé = abonnement actif, y compris à 0 € |
| `minute_usage` | agent_id, mois, minutes, alert_80_sent | Manual top-up crédité par l'équipe en V1 |

---

## 4. Lots de développement (ordre de construction)

### Lot 0 — Fondations (semaine 1)
- Dépôt, CI, environnements, authentification, Postgres, table des constantes, chargement des scénarios.
- **Tests de référence écrits d'abord**, à partir des exemples chiffrés du document (voir §6).

### Lot 1 — Téléphonie et aiguillage (4.1 à 4.3, ch. 6, 13.2)
- Compte Twilio, conformité réglementaire des numéros français (dossier « Regulatory Bundle » : justificatifs d'identité et d'adresse requis par Twilio pour les numéros FR — à déposer dès S1, délai de validation de plusieurs jours).
- Provisionnement du numéro technique français par mandataire. Webhook d'appel entrant (TwiML `<Connect><ConversationRelay>` vers notre WebSocket).
- Numéro sortant distinct pour le test du dispositif (6.2) : appel via l'API Twilio, reconnaissance du retour par corrélation sur la fenêtre de test.
- Ordre strict : **vérification des listes → annonce → filtre**.
  - Liste Noire : rejet, **aucune énonciation, aucune notification**. L'appel est seulement compté.
  - Liste Blanche : annonce dans sa variante Liste Blanche, puis prise de message et notification. Jamais de retransmission vers le mobile.
  - Autres numéros, y compris masqués : annonce standard, injectée depuis `IDENTITY`.
- Filtre à 3 branches. Pour le propriétaire d'un bien en location, la **question d'aiguillage** est posée hors plafond.
  - Hors périmètre : script d'orientation, fiche contact, aucun brief.
  - Hors-métier : `SIMPLE_VOICEMAIL`.
- Liste de mots bannis face au prospect : diagnostic, formulaire, questionnaire, enquête, quiz.

### Lot 2 — Qualification vocale (7.1, ch. 8, ch. 9)
Un **gestionnaire de dialogue déterministe** pilote les tours. Le modèle ne fait qu'extraire les réponses et repérer ce qui a déjà été dit.
- Plafond de 5 questions, sous-questions comprises. Deux tours seulement sont hors plafond : la question d'aiguillage et le tour de clôture.
- Sélection : tout le socle, puis les questions d'ajustement dans l'ordre déclaré. **L'ordre d'énonciation reste celui des écrans.**
- Économie de question : seulement si la réponse a été **exprimée** (niveau A). Une réponse déduite ne dispense jamais de poser la question.
- Format vocal : 4 options au maximum, avec regroupements (motivations : « changement de situation » ; types de bien : « Autre »).
- Tour de clôture : « Avant que [Nom] vous rappelle, y a-t-il un point important qu'il devrait connaître ? ». Il n'est pas posé si un critère d'arrêt a été reconnu (P02).
- Interruption : arrêt définitif, brief **incomplet**, aucune relance.

### Lot 3 — Moteur et brief (10.1, 10.2, 11.1, 11.2, 16.5)
- **Schéma JSON** du brief : ligne d'identification, zone opérationnelle, 12 champs, provenance portée par chaque donnée, mention complet/incomplet.
- Calculs en code :
  - **priorité** selon la grille du 11.2. Un élément non recueilli n'est jamais lu comme un « non » ; sans aucun élément, la priorité est **normale** ;
  - **alerte** si la priorité est élevée et qu'au moins 2 signaux de niveau A sont réunis ;
  - **délai recommandé** (P06), sans aucun délai en minutes ;
  - **objectif**, pris dans une liste fermée.
- **Questions prioritaires dérivées** : les questions du scénario non posées, dans leurs termes exacts, plus 2 questions ajoutées au maximum, ancrées et marquées.
- **Contrôle P09** avant mise à disposition :
  - plafonds de volumétrie : 3 points d'appui, 2 questions ajoutées, 3 points de vigilance ;
  - chaque énoncé B rattaché à un élément A et rédigé au conditionnel ;
  - attentes reprises mot pour mot de l'écran 6, sinon « non disponible ».
  - Si un contrôle échoue, on régénère une fois, puis on retire l'élément fautif.
- Le **numéro masqué** donne la mention « numéro non communiqué — rappel impossible », et pas de bouton « Rappeler ».
- **Marque de récurrence** : « 2ᵉ appel de ce numéro — précédent le JJ/MM ». Elle est rapprochée par le numéro seul, sans contenu reporté.
- Objectif de délai : le brief doit être disponible quelques secondes après la fin de l'appel (cet indicateur est mesuré).

### Lot 4 — Espace mandataire, web et application Median (5.1, 5.1.1, 6.1, 6.2, 6.3, 11.4)
- **Liste des briefs** : ceux qui portent une alerte en tête, puis tri par priorité.
- **Vue du brief** : lecture à deux vitesses (la zone « À faire » d'abord, puis A, B et C). Boutons **Copier** (texte intégral) et **Partager** (feuille de partage native).
- **Rappeler** : lance la composition du numéro, enregistre l'heure et fait passer à l'état « Rappel effectué ». **Saisie du résultat en un geste** : RDV obtenu, Pas de RDV, Pas joint, Sans suite. Machine d'états du 11.4 ; les états terminaux sont verrouillés.
- **Réglages** : `IDENTITY`, Liste Blanche et Liste Noire (avec l'import T04).
- **État du dispositif** : 3 états, et seulement 3. **« Confirmé » n'est jamais affiché sans une preuve datée de moins de 30 jours.** Le libellé nomme toujours la nature de la preuve.
- **Test du dispositif** : appel sortant depuis un numéro **distinct**, résultat binaire. Il ne produit ni brief, ni prospect, ni trafic.
- **Guide opérateur** : pages statiques par opérateur et par système, bouton « copier » pour chaque code (sur iOS, un lien ne peut pas composer de `*` ni de `#`).
- **Compteur de minutes**, sans aucun prix dans l'app. Suppression de compte.
- **Onboarding** : présenter avant la souscription les 4 informations du 6.3 (la messagerie vocale est remplacée, les appels renvoyés peuvent être facturés, le délai de sonnerie a un plafond, le mode NPD se règle séparément) ainsi que la mention C01.

### Lot 5 — Notifications (11.2)
- L'autorisation est demandée **après le premier test réussi**.
- Liste fermée de **6 déclencheurs** : brief prêt (alerte en tête), message pris, contrôle J+14 avec invitation à rejouer le test, dispositif inactif depuis 30 jours, bilan disponible, seuil de gratuité ou 80 % des minutes (y compris « il vous en manque 1 », « seuil atteint », « il vous manque un filleul comptabilisé »).

### Lot 6 — Mesure et tâches planifiées (10.4, 10.6, 6.1)
- **Vues SQL** pour les indicateurs du 10.4, calculées sur les états atteints. Le trafic traité n'est le dénominateur d'aucun indicateur. Les numéros masqués sont exclus des résultats commerciaux.
- **Bilan mensuel**, calculé à l'ouverture et jamais stocké :
  - 3 blocs séparés ;
  - **aucun taux** (la réactivité est restituée en « n sur N ») ;
  - **aucune valeur en euros** ;
  - temps évité arrondi vers le bas, toujours accompagné de son hypothèse de calcul, et calculé hors Liste Blanche ;
  - mois incomplet et mois sans activité gérés.
- **Tâches planifiées** : contrôle de trafic à J+14, alerte d'inactivité après 30 jours glissants, notification du bilan le 1er du mois, alerte à 80 % des minutes, fin de préavis, fin de pilote.

### Lot 7 — Abonnement et paiement (16.6, O02)
- Stripe Checkout et portail, avec un essai de 30 jours. **Web uniquement.**
- Avantage appliqué comme une **remise de 100 %** sur un abonnement qui reste actif, de sorte que le filleul reste comptabilisé (clause 3).
- Préavis : remise maintenue `GRACE_PERIOD_DAYS`, puis retirée automatiquement. **Aucune facture sans préavis.**
- Dépassement de minutes en V1 : procédure **manuelle** de l'équipe (lien Stripe personnalisé, puis crédit du compteur).

### Lot 8 — Programme Ambassadeur et couche réseau (16.1, 16.2, 16.6)
- Lien et code créés **à l'inscription**, avec la mention « actif dès votre souscription ». Le lien est activé à la souscription, et l'activation reste acquise ensuite. Exception : la case dérogatoire du pilote.
- Rattachement à l'inscription **uniquement si le lien est actif**. Sinon, le compte est créé sans parrain, avec un message explicite.
- Bouton **Inviter** : ouvre la feuille de partage native et incrémente le compteur `share_event`.
- Compteurs : inscrits, actifs (test réussi **et** un appel réel), abonnés, **comptabilisés**. Aucun avantage n'est calculé au-delà du premier niveau.
- Espace ambassadeur **en deux temps** : au départ, seulement le lien, l'état du lien, « Inviter » et le kit ; les compteurs et l'avantage apparaissent au premier filleul rattaché.
- **Garde-fous testés automatiquement** : l'API du parrain ne renvoie **que des totaux**. Jamais de nom associé à un statut, jamais de rapport comptabilisés/inscrits, jamais de donnée de prospect, de brief ou de résultat.

### Lot 9 — Publication sur les stores (ch. 5)
- Comptes développeur **au nom d'une organisation** (sinon, Google impose 12 testeurs pendant 14 jours).
- Service de publication Median, compte de démonstration pré-rempli, formulaires de confidentialité.

### Lot 10 — Pilote (ch. 18)
- Recrutement de 3 à 5 parrains, chacun avec 6 à 12 filleuls. Case dérogatoire. Accompagnement à l'installation.
- Points d'étape : J+14 (prise en main, test daté), **J+30** (lecture complète des indicateurs), J+45 (témoignages), J+60 (offre de conversion, sans renouvellement automatique).
- Requêtes du 18.4 : les taux de transformation sont **restitués en effectifs bruts**. Observations : alertes, consultation du bilan, questions posées, interruptions.

---

## 5. Recette : les « non-conformités » du document deviennent des tests

| # | Règle à vérifier automatiquement | Réf. |
|---|---|---|
| 1 | « Dispositif confirmé » n'est jamais affiché sans une preuve datée de moins de 30 jours | 6.1 |
| 2 | L'appel de test n'apparaît dans aucun brief, prospect, trafic ni indicateur | 6.2 |
| 3 | Aucun message (SMS, courriel, lien) n'est jamais adressé au prospect | ch. 3 |
| 4 | La Liste Noire ne produit ni énonciation ni notification individuelle | 4.2 |
| 5 | Un brief est produit pour 100 % des appels « métier ciblé », y compris interrompus | ch. 6 |
| 6 | La priorité n'utilise que des éléments A, et un élément absent n'est jamais lu comme un « non » | 11.2 |
| 7 | Tout énoncé B est au conditionnel et rattaché à un élément A | 11.1 |
| 8 | Les questions prioritaires reprennent mot pour mot le texte du scénario ; au plus 2 ajouts | 11.1 |
| 9 | Le bilan ne contient aucun taux ni aucun montant en euros ; le temps évité est toujours accompagné de son hypothèse | 10.6 |
| 10 | Le parrain ne reçoit que les compteurs de la liste fermée | 16.1, 16.2 |
| 11 | Aucun prix ni bouton d'achat n'apparaît dans l'application | 16.6 |
| 12 | Liste Noire, Liste Blanche et test ne sont jamais facturés | 16.6 |
| 13 | `GRACE_PERIOD_DAYS` ≥ `TRIAL_DURATION_DAYS`, vérifié au démarrage | 16.6 |
| 14 | Aucun rattachement rétroactif à un parrain | 16.2 |

## 6. Jeux d'essai tirés du document

- **Grille de priorité** : les 8 cases du 11.2, plus les cas où un élément est absent et celui où il n'y a aucun élément (attendu : normale).
- **Exemple B** (Monsieur Berger) : 3 questions posées, écrans 1 et 2 non posés, écran 4 posé, brief complet.
- **Brief de Madame Dupont** (11.3) : alerte déclenchée par 2 signaux, attentes « non disponible », 3 questions prioritaires.
- **Bilan de mars** : 19 + 5 + 10 + 8 = 42 ; assiette de 34 ; 170 min affichées « 2 h 30 » ; 19 briefs, 17 prospects qualifiés, 15 rappels, 11 sur 15.
- **Cascade de Julien** : solde de +2, Marc reste gratuit.
- **Préavis** : passage sous le seuil, puis 30 jours, puis reprise de la facturation, avec une notification à chaque étape.

---

## 7. Calendrier indicatif (équipe de 1 à 2 développeurs)

| Semaines | Contenu | Jalon |
|---|---|---|
| S1–S2 | Lot 0, Lot 1 ; lancement de T01, C02, O02 et O03 ; rédaction des scénarios estimation et achat (toi, fin S2) | Un appel réel reçoit l'annonce et est classé |
| S3–S5 | Lots 2 et 3 | Premier brief conforme de bout en bout (scénario vente) |
| S5–S7 | Lots 4 et 5 | Application Median en test interne, test du dispositif opérationnel |
| S7–S8 | Lots 6 et 7 | Bilan, tâches planifiées, Stripe en mode test |
| S8–S9 | Lot 8 | Parrainage de bout en bout, garde-fous testés |
| S9–S10 | Lot 9, recette complète (§5 et §6) | **Go/No-go du pilote** : T01, C02, R12 et O02 tranchés |
| S11–S19 | Lot 10, pilote de 60 jours (lecture à J+30) | Critères du 18.5 |

## 8. Principaux risques

| Risque | Parade |
|---|---|
| Renvoi conditionnel mal configuré, produit « muet » | Test au premier jour, contrôle à J+14, alerte à 30 jours, guide maintenu deux fois par an |
| Latence vocale ou mauvaise reconnaissance (communes, noms) | Réglage de ConversationRelay (langue fr-FR, choix STT/TTS, gestion des interruptions) validé sur des appels réels dès S1 ; la commune est une réponse libre courte |
| Hallucination dans le brief | Schéma imposé, calculs en code, contrôle P09, règle d'ancrage |
| Formule de parrainage requalifiée (« boule de neige ») | Avis d'avocat avant le lancement (C02) ; premier niveau seulement ; entrée conditionnée à un paiement |
| Coût télécom d'un gros consommateur | Forfait, alerte à 80 %, traitement manuel |
| Revue des stores (paiement, suppression de compte) | Aucun prix dans l'app ; suppression de compte ; compte de démonstration |
| Le pilote ne peut pas observer la 2ᵉ génération | Limite connue, consignée (18.5) ; suivi pendant un trimestre après le lancement |

## 9. Décisions à prendre maintenant

1. ~~Fournisseur voix~~ → **Twilio** (décidé).
2. **Back-end managé** : Supabase, ou autre ?
3. **Prix des packs de minutes vendus à la main en V1** : il n'est pas fixé (16.6).
4. ~~Rédaction des scénarios~~ → **toi**, échéance fin S2 (gabarit en annexe A).
5. **Dépôt de code** : créer un dépôt dédié « brief-mandataire » ? Les dépôts actuels SECURBTP concernent un autre produit.

---

## Annexe A — Gabarit de rédaction d'un scénario (modèle : chapitre 8)

À remplir une fois pour **Demande d'estimation / avis de valeur**, une fois pour **Projet d'achat**. Chaque rubrique répond à une règle du document ; la colonne « Contrôle » dit ce que je vérifierai à la relecture.

### A.1 Version écran (référence rédactionnelle)

Pour chaque écran, numéroté :

| Rubrique | À écrire | Contrôle |
|---|---|---|
| Phrase d'enchaînement | ex. « Merci. » | Aucun mot banni : diagnostic, formulaire, questionnaire, enquête, quiz (ch. 3) |
| Question | Texte exact | C'est ce texte qui sera restitué mot pour mot en question prioritaire (11.1) |
| Type | choix unique / choix multiple / champ libre / champ de recherche | — |
| Options | Liste complète | Les options **sont** la nomenclature restituée dans le brief |
| Sous-question conditionnelle | Si … alors « … » | Compte dans le plafond si posée à la voix (7.1) |

### A.2 Adaptation vocale

| Écran | Voix (oui/non) | Catégorie | Options à l'oral (≤ 4) | Regroupement |
|---|---|---|---|---|
| … | … | Socle / Ajustement — priorité n / Tour de clôture / — | … | ex. « Autre » couvre… |

Règles à respecter :
- **≤ 5 questions vocales** au total, sous-questions comprises ; le tour de clôture est commun et hors plafond (7.1).
- **≤ 4 options** à l'oral ; tout le reste passe par « Autre », précisé librement.
- Les questions d'ajustement portent un **ordre de priorité** (1, 2…) ; l'ordre d'énonciation reste celui des écrans.
- Un écran non posé à la voix devient question prioritaire du brief : vérifier que son texte se lit bien comme une question à poser au rappel.

### A.3 Priorité (11.2)

- **Éléments retenus** (liste fermée, niveau A, **obligatoirement au socle**) : …
- **Grille** complète, toutes combinaisons couvertes, sur le modèle du tableau horizon × autre professionnel :

| Élément 1 \ Élément 2 | valeur a | valeur b |
|---|---|---|
| … | élevée / normale / faible | … |

- Rappel : élément non recueilli = absent, jamais « non » ; aucun élément = **normale**.

### A.4 Signaux d'alerte de réactivité (11.2)

- Signaux du scénario (niveau A, **au socle**) : …
- **Achat — obligatoire au socle** : *référence du bien / annonce en cours* et *demande de visite* (ch. 9).
- L'alerte exige priorité élevée + au moins 2 signaux : vérifier qu'au moins 2 signaux peuvent être réunis dans ce scénario, sinon l'alerte n'y sera jamais déclenchable.

### A.5 Délai et objectif (11.2, P06)

- Proposition de règle d'affectation des 4 délais (dès la sortie de visite / dans l'heure / aujourd'hui / selon le délai du projet) à partir des seuls éléments de priorité — c'est l'occasion de trancher **P06** pour les trois scénarios.
- Objectif du rappel par défaut, pris dans la liste fermée (obtenir un rendez-vous, préparer une estimation, qualifier davantage, répondre à une attente précise, autre objectif du scénario — libellé à fournir).

### A.6 Spécificités

- **Estimation** : objectif de la demande (vente, succession, partage, investissement, simple information) — à placer au socle ou en ajustement ; rappel : le moteur ne donne **aucune valeur ni fourchette** (13.1).
- **Achat** : intègre « appel sur annonce » (référence, disponibilité, demande de visite, financement, bien à vendre au préalable, critères élargis) et « acquéreur investisseur » (objectif patrimonial, budget, rendement, horizon, zone) **sans sous-scénario** : ce sont des questions du même scénario. Avec plus de 5 questions candidates, c'est ici que l'économie de question prendra tout son sens (7.2) — soigner l'ordre de priorité des ajustements.

### A.7 Exemple de contrôle

Fournir pour chaque scénario **un appel type** (comme Madame Dupont / Monsieur Berger) avec les réponses attendues : il deviendra un test automatique (§6).
