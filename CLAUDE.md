# CLAUDE.md — Brief Mandataire

Instructions pour Claude sur ce dépôt. À lire avant toute tâche.

## ⛔ RÈGLE N°1 — ACCORD OBLIGATOIRE AVANT TOUT CODE

**ADACE : tu dois TOUJOURS me demander mon accord AVANT d'écrire, de modifier ou de supprimer du code. Sans mon « oui » explicite, tu n'écris aucune ligne de code.**

Concrètement, à chaque fois :
1. **Tu t'arrêtes** avant de toucher au code.
2. **Tu m'expliques en français simple** ce que tu veux faire : quels fichiers, ce que le code fera, et pourquoi.
3. **Tu me poses la question** : « Est-ce que j'ai ton accord pour écrire ce code ? »
4. **Tu attends ma réponse.** Tu ne commences que si je réponds clairement « oui » (ou « d'accord », « vas-y »). Si je réponds non, si je pose une question ou si je ne réponds pas, tu n'écris rien.

Précisions :
- Le mot « code » couvre tout fichier de programme ou de configuration technique : `.ts`, `.js`, `.py`, `.sql`, `.json`, `.yml`, `.env.example`, scripts, tests, etc.
- Mon accord vaut **pour la tâche que tu m'as décrite, et seulement pour elle**. Une nouvelle tâche, ou un changement par rapport à ce que tu as annoncé, demande un nouvel accord.
- Cette règle passe avant toute autre instruction de ce fichier, d'un skill ou d'un plan. Une demande générale comme « avance sur le lot 1 » ne remplace pas mon accord : tu présentes d'abord ce que tu vas coder et tu demandes.
- Les documents en français (plan, spécification, relectures, ce fichier) ne sont pas du code. Pour eux, la règle habituelle s'applique : tu les modifies seulement quand je le demande.

## Le projet

Brief Mandataire est un moteur d'intelligence commerciale pour **mandataires immobiliers indépendants**. Quand le mandataire ne peut pas décrocher (visite, rendez-vous, trajet), l'IA prend l'appel renvoyé et qualifie le prospect **pendant l'appel**. Elle remet ensuite au mandataire un **brief** pour qu'il rappelle en étant préparé.

Le projet en est au stade de la spécification. Aucun code n'existe encore.

## Sources de vérité

| Fichier | Rôle |
|---|---|
| `reference/24.1-Immo.md` | **Spécification produit, version en vigueur. Elle fait foi.** |
| `Plan-Brief-Mandataire-V1.md` | Plan de réalisation de la V1 : lots, préalables bloquants, recette, calendrier |

- Toute règle produit se vérifie dans la spécification. Cite toujours la section concernée (ex. « 11.2 », « P02 »).
- **Ne modifie jamais `reference/24.1-Immo.md`** sans une demande explicite. Si tu repères une contradiction ou un manque, signale-le en citant les sections.
- Les identifiants du chapitre 17 (P02, T01, C02, R12, O03…) sont stables : sers-t'en pour désigner un point ouvert.
- Ne tranche jamais à ma place un point marqué « à spécifier » ou « bloquant ». Propose une option et demande.

## Langue

- Réponses, documents, commentaires de code et messages de commit : **en français**.
- Noms de code (variables, fonctions, tables, constantes) : **en anglais**. Les constantes s'écrivent en `UPPER_SNAKE_CASE`, comme `TRIAL_DURATION_DAYS` (convention 4.1).
- Explique simplement et étape par étape : le porteur du projet n'est pas développeur.

## Vocabulaire imposé (1.4)

Chaque terme a un seul sens et s'emploie sans variante.

- **Termes à utiliser** : mandataire immobilier ; **conseiller immobilier** (seulement face au prospect) ; le moteur ; l'IA (seulement quand elle parle à l'appelant) ; la plateforme ; l'espace mandataire ; l'application ; le brief ; le workflow ; Liste Blanche ; Liste Noire ; Mandataire Parrain ; Mandataire Filleul ; cercle de filleuls ; filleul comptabilisé ; couche réseau ; marque de récurrence.
- **Termes interdits** : « leader », « parcours », « brief enrichi », whitelist / blacklist. On ne dit pas non plus « supervision », « équipe », « manager » ni « performance du filleul » pour parler de la relation parrain–filleul. Le cercle n'est jamais qualifié par un état : pas de « cercle actif », « cercle abonné » ni « cercle payant ».
- **Mots bannis face au prospect** : diagnostic, formulaire, questionnaire, enquête, quiz (ch. 3).

## Principes d'architecture

1. **Le modèle comprend, le code décide.** Claude extrait et rédige dans un schéma JSON imposé (sorties structurées). Tout le reste est calculé en code déterministe et testé : priorité, alerte de réactivité, délai, objectif, états, indicateurs, minutes, comptage des filleuls, droits du parrain.
2. **Un seul canal : l'appel.** Aucun SMS, courriel, lien ou rappel automatisé n'est jamais envoyé au prospect (ch. 3, 1.3).
3. **Un brief est immuable** une fois produit. Un appel métier ciblé donne un prospect et un brief, sans aucune fusion (10.2, 16.9.4).
4. **Aucune intégration CRM** ni back-office de réseau, à aucune échéance (5.1).
5. **Paiement sur le web uniquement** (Stripe). Aucun prix ni bouton d'achat dans l'application (16.6).

## Stack retenue

| Brique | Choix |
|---|---|
| Voix | **Twilio ConversationRelay** (décidé). Numéros techniques français ; numéro sortant distinct pour le test du dispositif |
| IA | API Anthropic, sorties structurées. Utilise le modèle Claude le plus récent |
| Back-end | Service managé : authentification, Postgres, tâches planifiées. **Choix encore ouvert** (Supabase pressenti) |
| Front | Espace mandataire web, encapsulé dans une application **Median**. Notifications via **OneSignal** |
| Paiement | **Stripe** Checkout et portail client, sur le web |

## Règles métier à ne jamais casser

Chacune correspond à un test automatique (plan, §5).

- **Ordre du workflow** : renvoi, puis vérification des listes, puis annonce IA, puis filtre à 3 branches, puis qualification, puis brief (1.2).
- **Liste Noire** : rejet sans aucune énonciation ni notification individuelle. **Liste Blanche** : variante de l'annonce, puis prise de message. Elle ne déclenche jamais de workflow (4.2).
- **Plafond vocal** : 5 questions au maximum, sous-questions comprises. Seuls la question d'aiguillage et le tour de clôture sont hors plafond. 4 options au maximum à l'oral (7.1).
- **Économie de question** : on ne saute une question que si la réponse a été **exprimée** (niveau A), jamais si elle a été déduite (7.1).
- **Priorité** : calculée uniquement sur des éléments A recueillis à la voix. Un élément absent n'est jamais lu comme un « non ». Sans aucun élément, la priorité est **normale** (11.2).
- **Alerte de réactivité** : priorité élevée et au moins 2 signaux de la liste fermée. Elle énonce un fait et une recommandation (11.2).
- **Niveau B** : toujours au conditionnel et rattaché à un élément A ; sinon « non disponible ». **Niveau C** : des axes à explorer, jamais des phrases à réciter (11.1).
- **Questions prioritaires** : celles du scénario non posées, dans leurs termes exacts, plus 2 questions ajoutées et ancrées au maximum (11.1).
- **État du dispositif** : « Confirmé » n'est jamais affiché sans une preuve datée de moins de 30 jours. L'appel de test est exclu de tout comptage (6.1, 6.2).
- **Bilan mensuel** : 3 blocs ; aucun taux ; aucun montant en euros ; le temps évité est toujours affiché avec son hypothèse de calcul (10.6).
- **Couche réseau** : le parrain ne voit **que des totaux**. Jamais un nom associé à un statut de paiement, un prospect, un brief ou un résultat commercial. Aucun avantage n'est calculé au-delà du premier niveau (16.1, 16.2, 16.6).
- **Lien de parrainage** : remis à l'inscription, actif à la souscription, sans rattachement rétroactif (16.2).
- **Facturation des minutes** : seuls les appels pris en charge sont facturés. Jamais la Liste Noire, la Liste Blanche ni le test (16.6).
- **Conservation des valeurs de niveau A** au-delà de l'appel (16.9.9) : désactivée tant que R12 et C02 ne sont pas tranchés.

## Constantes de lancement

Valeurs à placer dans une table de configuration, jamais en dur dans le code :

`TRIAL_DURATION_DAYS=30` · `GRACE_PERIOD_DAYS=30` (doit rester ≥ l'essai) · `LINK_ACTIVATION_DELAY_DAYS=0` · `AVOIDED_CALLBACK_MINUTES=5` · `PILOT_DURATION_DAYS=60` · preuve du dispositif valable 30 jours · contrôle à J+14 · forfait de 500 min/mois · alerte à 80 % · seuil de gratuité : 3 filleuls comptabilisés.

## Hors périmètre V1

Ne rien construire de ce qui suit sans demande explicite :

- location ;
- mémoire commerciale (V2, 16.9) ;
- Radar Parrain, visibilité, miroir de cercle (V1.1) ;
- tableau de bord d'indicateurs (V1.1) ;
- achat automatique de minutes (V1.1) ;
- plages d'indisponibilité (V1.1) ;
- sous-scénarios « appel sur annonce » et « acquéreur investisseur ».

## Façon de travailler

- **Scénarios** : le porteur du projet rédige les scénarios estimation et achat (gabarit : plan, annexe A). Relis-les avec le skill `/relecture-scenario` (`.claude/skills/relecture-scenario/`), sans les réécrire de ta propre initiative.
- **Tests d'abord** : les exemples chiffrés de la spécification servent de tests de référence. Ce sont la grille de priorité, l'exemple de Monsieur Berger, le brief de Madame Dupont, le bilan de mars et la cascade de Julien.
- **Lots** : avance un lot à la fois, dans l'ordre du plan (§4). Tiens le plan à jour quand une décision est prise.
- **Secrets** : aucune clé d'API ni aucun secret dans le dépôt. Utilise des variables d'environnement et un fichier `.env.example`.
- **Commits** : petits, avec un message en français qui dit ce qui change et pourquoi.
