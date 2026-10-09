---
name: relecture-scenario
description: Relit un scénario de qualification Brief Mandataire (estimation, achat ou vente) et le contrôle point par point contre la spécification reference/24.1-Immo.md. Produit un rapport clair, en français simple, avec un verdict et la liste des corrections. À utiliser quand le porteur du projet colle ou dépose un scénario, demande « relis mon scénario », « vérifie le scénario achat/estimation », ou invoque /relecture-scenario.
---

# Relecture de scénario — Brief Mandataire

Tu relis un scénario rédigé par le porteur du projet. **Tu ne le réécris pas.** Tu contrôles, tu expliques, tu proposes. Le scénario reste le sien (CLAUDE.md, « Façon de travailler »).

## Étape 0 — Récupérer le scénario et les règles

1. Récupère le scénario : texte collé dans la conversation, ou fichier indiqué (souvent sous `scenarios/`). Si tu ne le trouves pas, demande-le, puis arrête-toi.
2. Identifie de quel scénario il s'agit : **vente**, **estimation** ou **achat**.
3. Relis dans `reference/24.1-Immo.md` les sections : 7.1 (qualification), chapitre 8 (le modèle complet de la vente), chapitre 9 (catalogue des scénarios), 11.1 (questions prioritaires), 11.2 (priorité et alerte), 4.1.1 (périmètre).
4. Relis l'annexe A de `Plan-Brief-Mandataire-V1.md`, qui est le gabarit attendu.

Cite toujours la section de la spécification quand tu signales un problème. Si une règle te semble absente ou ambiguë dans la spécification, dis-le au lieu de l'inventer.

## Étape 1 — Contrôles

Pour chaque contrôle, note l'un des trois résultats : ✅ conforme, ❌ non conforme (à corriger), ⚠️ à vérifier ou manquant.

### A. Version écran (ch. 8)

- A1. Chaque écran a une question rédigée en entier. C'est ce texte exact qui sera repris mot pour mot dans les questions prioritaires du brief (11.1).
- A2. Le type de chaque écran est indiqué : choix unique, choix multiple, champ libre ou champ de recherche.
- A3. Les options sont listées en entier ; elles deviennent la nomenclature restituée dans le brief.
- A4. **Aucun mot banni** face au prospect : diagnostic, formulaire, questionnaire, enquête, quiz (ch. 3).
- A5. Face au prospect, le professionnel est appelé « conseiller immobilier », jamais « mandataire » (1.4).
- A6. Le scénario n'évoque aucune relance après l'appel : pas de SMS, de lien, de courriel ni de rappel automatique (ch. 3).
- A7. Il reste dans le périmètre : pas de location, de gestion locative, de syndic ni de copropriété (4.1.1). Les autres biens et le viager passent par « Autre ».

### B. Version vocale (7.1)

- B1. **5 questions vocales au maximum**, sous-questions conditionnelles comprises. Compte-les et donne le total.
- B2. Seuls le tour de clôture et la question d'aiguillage sont hors plafond. Aucun autre tour ne doit être déclaré hors plafond.
- B3. **4 options au maximum à l'oral** pour chaque question. Quand l'écran en a plus, il faut un regroupement documenté, avec « Autre » pour le reste.
- B4. Chaque question vocale est classée **socle** ou **ajustement**. Les questions d'ajustement ont un ordre de priorité (1, 2…).
- B5. L'ordre d'énonciation reste celui des écrans. Une question d'ajustement ne passe jamais devant une question socle qui la précède.
- B6. Les questions à choix multiple et les champs libres ne sont pas posés à la voix. Deux exceptions seulement : un champ de recherche devient une réponse libre courte, et le tour de clôture.
- B7. Les écrans non posés à la voix sont listés : ils deviendront des questions prioritaires du brief. Vérifie que leur texte se lit bien comme une question à poser pendant le rappel.

### C. Priorité (11.2)

- C1. Les éléments qui servent à calculer la priorité sont listés : une liste fermée d'informations de niveau A recueillies à la voix.
- C2. **Chacun de ces éléments est au socle.** Sinon, la priorité devient incalculable.
- C3. La grille couvre **toutes** les combinaisons de réponses. Énumère-les et repère les cases manquantes.
- C4. Aucun élément déduit (niveau B) et pas le tour de clôture : la priorité ne repose que sur des faits.
- C5. Un élément non recueilli n'est jamais lu comme un « non ». Sans aucun élément, la priorité est **normale**.

### D. Alerte de réactivité (11.2)

- D1. Les signaux d'alerte du scénario sont listés : des informations de niveau A, et **toutes au socle**.
- D2. **Achat uniquement** : le socle contient obligatoirement la **référence du bien / l'annonce en cours** et la **demande de visite** (ch. 9).
- D3. Au moins 2 signaux peuvent être réunis dans le même appel. Sinon, l'alerte ne se déclenchera jamais pour ce scénario (elle exige priorité élevée et 2 signaux).
- D4. Aucun signal ne repose sur une question non posée à la voix ni sur le tour de clôture.

### E. Délai et objectif (11.2, P06)

- E1. Une règle d'affectation est proposée pour les 4 délais : dès la sortie de visite, dans l'heure, aujourd'hui, selon le délai du projet. Elle repose sur les seuls éléments de priorité, **sans aucun délai exprimé en minutes**.
- E2. L'objectif du rappel est pris dans la liste fermée. Seule la valeur « autre objectif correspondant au scénario » permet un libellé propre au scénario.

### F. Spécificités du scénario (ch. 9, 13.1)

- **Estimation** : l'objectif de la demande (vente, succession, partage, investissement, simple information) est prévu. Le scénario ne promet jamais de valeur ni de fourchette de prix, car le moteur ne fait aucune estimation (13.1).
- **Achat** : il intègre les questions de l'appel sur annonce (référence, disponibilité, visite, financement, bien à vendre au préalable, critères élargis) et celles de l'acquéreur investisseur (objectif patrimonial, budget, rendement, horizon, zone). Ce sont des **questions du même scénario, sans sous-scénarios**. Il y a plus de 5 questions candidates : vérifie que l'ordre de priorité des questions d'ajustement est réfléchi.
- **Investissement** : c'est une **motivation** (10.1) ou une **question du scénario d'achat**, jamais un type de projet. Les deux emplois ne doivent pas se confondre.

### G. Appel type (plan, annexe A.7)

- G1. Un appel type est fourni, avec les réponses du prospect.
- G2. Déroule-le toi-même avec les règles :
  - quelles questions sont posées, en tenant compte de l'économie de question (on ne saute une question que si la réponse a été **exprimée**) ;
  - quelle priorité en résulte ;
  - si l'alerte se déclenche ;
  - quelles questions prioritaires figurent dans le brief.

  Signale toute incohérence avec ce que l'auteur attendait.

## Étape 2 — Rapport

Réponds **en français simple**, sans jargon technique, avec cette structure :

```
## Relecture — Scénario <nom>

**Verdict :** ✅ prêt pour le développement / 🔶 corrections mineures / ❌ à reprendre
**Résumé :** <2-3 phrases : ce qui va bien, ce qui bloque>

### À corriger (bloquant)
1. <problème> — <pourquoi, avec la section, ex. (7.1)> — <proposition de correction>

### À vérifier / compléter
1. ...

### Points conformes
- <liste courte>

### Déroulé de l'appel type
| Écran | Posé ? | Pourquoi |
|---|---|---|
Priorité obtenue : … · Alerte : oui/non · Questions prioritaires : …

### Questions pour toi
1. <décision que seul l'auteur peut prendre>
```

Règles du rapport :
- Mets les problèmes bloquants en premier, le plus grave d'abord.
- Une proposition de correction est une **suggestion**. N'écris pas le scénario à la place de l'auteur.
- N'affiche jamais un verdict « prêt » s'il reste un ❌.
- Si le scénario est enregistré dans un fichier du dépôt, ne le modifie pas sans accord. Demande d'abord : « Veux-tu que j'applique ces corrections ? »
