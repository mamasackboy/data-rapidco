# RapidCo — Projet Datamining (Sujet 2)

Livraison urbaine : retards, durée de trajet, secteurs de livraison et plan de réorganisation.

## Organisation du dépôt

```
data/raw/            Données brutes de l'entreprise (ne jamais modifier à la main)
  RapidCo_livraisons.csv   714 livraisons
  RapidCo_chauffeurs.csv   18 chauffeurs
docs/consignes/      Énoncés fournis
  Sujet_2_RapidCo.pdf      Sujet et les 9 questions de la direction
  Projet_Consignes.pdf     Règles du projet, barème, soutenance
```

Les données nettoyées iront dans `data/clean/`, le code dans `src/`, les slides dans `slides/`.

## Démarche

À compléter au fil du projet : pour chaque question, ce qu'on a compris, la méthode retenue, pourquoi, et la conclusion.

# Q1 — Les retards : une affaire de personnes, de secteurs ou d'horaires ?

*5 novembre 2026*

## 1. La question

> « On accuse les chauffeurs. Avant de lancer un plan de formation, dites-nous si les retards sont une affaire de personnes, de secteurs ou d'horaires. »

Trois suspects sont à départager. Pour chacun, on utilise une colonne du fichier :

| Suspect | Colonne utilisée | Fichier |
|---|---|---|
| Les personnes | `chauffeur`, `anciennete_mois` | livraisons + chauffeurs |
| Les secteurs | `distance_km` (éloignement par rapport au dépôt) | livraisons |
| Les horaires | `creneau` (08-12, 12-16, 16-20) | livraisons |

## 2. La méthode retenue et pourquoi

**Méthode : le taux de retard par groupe, puis un tableau croisé.**

Pour chaque groupe (chaque créneau, chaque chauffeur, chaque tranche de distance), on calcule :

```
taux de retard = nombre de livraisons en retard ÷ nombre de livraisons du groupe
```

Si le taux change beaucoup d'un groupe à l'autre, le suspect explique les retards. S'il reste à peu près le même partout, le suspect n'explique rien.

**Pourquoi cette méthode et pas une corrélation :**
- le créneau et le chauffeur sont des catégories, pas des nombres : on ne peut pas calculer une corrélation avec ;
- un taux par groupe se lit en réunion, ce qu'un coefficient ne permet pas (cours Analyse exploratoire, « Découper en tranches ») ;
- un suspect peut en cacher un autre (un chauffeur peut avoir beaucoup de retards uniquement parce qu'il livre le soir), d'où le tableau croisé qui compare les suspects à créneau égal.

## 3. Ce qu'on a vérifié avant de calculer

| Vérification | Constat | Décision |
|---|---|---|
| Lignes en double | 35 lignes identiques (714 lignes pour 679 identifiants) | retirées → 679 livraisons |
| Cases vides | 14 poids et 14 types de client manquants | gardées, ces colonnes ne servent pas ici |
| Distances à 0 km | 4 livraisons | impossibles, signalées ; sans effet sur la conclusion |
| Durées à 999 min | 3 livraisons | code d'erreur, signalées ; colonne non utilisée ici |
| Unité de l'ancienneté | en mois (136 mois = 11 ans) | signalée, convertie en années quand on la présente |
| CH05 | un triporteur de 1 200 kg | invraisemblable, signalé |
| Taille des groupes | 27 à 45 livraisons par chauffeur | assez pour comparer, avec prudence |

Préparation : les deux fichiers sont reliés par la colonne `chauffeur`, et la colonne `en_retard` (oui/non) est transformée en `retard` (1/0). La moyenne d'une colonne de 0 et de 1 donne directement le taux de retard.

## 4. Suspect « horaires »

| Créneau | Livraisons | En retard | Calcul | Taux |
|---|---|---|---|---|
| 12-16 | 200 | 23 | 23 ÷ 200 | **11,5 %** |
| 08-12 | 233 | 62 | 62 ÷ 233 | **26,6 %** |
| 16-20 | 246 | 150 | 150 ÷ 246 | **61,0 %** |

Le taux est multiplié par plus de 5 entre l'après-midi et le soir. Le créneau 16-20 représente 36 % des livraisons mais 64 % des retards (150 sur 235).

→ **L'horaire compte.**

## 5. Suspect « secteurs »

Le dépôt se trouve au centre du plan, vers (5 ; 5). La distance parcourue est presque exactement la distance entre le dépôt et le point de livraison (corrélation de 0,93 avec √((x − 5)² + (y − 5)²)). La colonne `distance_km` résume donc l'éloignement du secteur en un seul nombre.

| Distance de la course | Livraisons | En retard | Calcul | Taux |
|---|---|---|---|---|
| moins de 3 km | 74 | 5 | 5 ÷ 74 | **6,8 %** |
| 3 à 5 km | 246 | 59 | 59 ÷ 246 | **24,0 %** |
| 5 à 7 km | 297 | 128 | 128 ÷ 297 | **43,1 %** |
| plus de 7 km | 62 | 43 | 43 ÷ 62 | **69,4 %** |

Le taux est multiplié par 10 entre une course courte et une course longue.

Remarque : la corrélation directe entre `est_ouest_km` et le retard vaut 0,03. Elle ne voit rien, parce que le lien n'est pas une droite (on est loin du dépôt à l'ouest comme à l'est). C'est le cas « lien évident, coefficient nul » du cours ; d'où le passage par la distance.

→ **Le secteur compte.**

## 6. Suspect « personnes »

### 6.1 Taux par chauffeur

| Chauffeur | Livraisons | En retard | Taux | Part de livraisons le soir | Ancienneté (mois) |
|---|---|---|---|---|---|
| CH07 | 32 | 5 | 16 % | 38 % | 116 |
| CH04 | 44 | 11 | 25 % | 34 % | 60 |
| CH14 | 34 | 9 | 26 % | 24 % | 64 |
| CH12 | 44 | 12 | 27 % | 41 % | 45 |
| CH18 | 36 | 10 | 28 % | 33 % | 93 |
| CH13 | 39 | 12 | 31 % | 36 % | 55 |
| CH06 | 41 | 13 | 32 % | 39 % | 122 |
| CH01 | 36 | 12 | 33 % | 42 % | 15 |
| CH08 | 34 | 12 | 35 % | 32 % | 94 |
| CH10 | 36 | 13 | 36 % | 36 % | 117 |
| CH03 | 41 | 15 | 37 % | 32 % | 67 |
| CH15 | 35 | 13 | 37 % | 29 % | 53 |
| CH05 | 45 | 17 | 38 % | 31 % | 7 |
| CH09 | 38 | 16 | 42 % | 42 % | 13 |
| CH17 | 35 | 15 | 43 % | 34 % | 9 |
| CH16 | 43 | 19 | 44 % | 37 % | 119 |
| CH02 | 27 | 12 | 44 % | 30 % | 5 |
| CH11 | 39 | 19 | 49 % | **59 %** | 136 |

Les taux vont de 16 % à 49 %. Mais à ce stade on ne peut pas conclure : CH11, le pire, est aussi celui qui livre le plus le soir (59 % de ses livraisons).

### 6.2 L'ancienneté

Corrélation entre `anciennete_mois` et `retard` : **−0,02**, soit aucun lien. Le plus ancien (CH11, 136 mois) est le pire ; le plus récent (CH02, 5 mois) est juste derrière. Des anciens et des nouveaux se trouvent en haut comme en bas du classement.

### 6.3 Le test décisif : chaque chauffeur dans chaque créneau

Si le problème venait des personnes, un mauvais chauffeur serait en retard à toute heure. Si le problème vient de l'horaire, tous les chauffeurs seraient plus en retard le soir.

| Chauffeur | 08-12 | 12-16 | 16-20 |
|---|---|---|---|
| CH07 | 11 % | 0 % | 33 % |
| CH04 | 16 % | 0 % | 53 % |
| CH14 | 27 % | 7 % | 62 % |
| CH12 | 7 % | 17 % | 50 % |
| CH18 | 25 % | 17 % | 42 % |
| CH13 | 42 % | 8 % | 43 % |
| CH06 | 23 % | 17 % | 50 % |
| CH01 | 23 % | 12 % | 53 % |
| CH08 | 46 % | 20 % | 36 % |
| CH10 | 20 % | 8 % | 77 % |
| CH03 | 50 % | 8 % | 46 % |
| CH15 | 7 % | 30 % | 90 % |
| CH05 | 32 % | 8 % | 71 % |
| CH09 | 18 % | 18 % | 75 % |
| CH17 | 40 % | 8 % | 83 % |
| CH16 | 21 % | 15 % | 88 % |
| CH02 | 55 % | 12 % | 62 % |
| CH11 | 18 % | 0 % | 74 % |
| **Étendue** | 7 à 55 % | **0 à 30 %** | **33 à 90 %** |

Exemple de calcul, CH11 : 39 livraisons, dont 23 le soir ; 17 de ses 19 retards ont lieu le soir → 17 ÷ 23 = 74 %. L'après-midi, 0 retard sur 5 livraisons, comme CH07.

Lecture : les 18 chauffeurs, sans exception, sont plus en retard le soir que l'après-midi. Même le meilleur (CH07) passe de 0 % à 33 %. À créneau égal, les écarts entre chauffeurs se resserrent fortement ; le taux global de CH11 s'explique par ses créneaux, pas par sa conduite.

→ **Les chauffeurs ne sont pas la cause.**

## 7. Contre-vérification : distance × créneau

Même test pour le secteur, pour s'assurer que les zones éloignées ne sont pas simplement celles où l'on livre le soir.

Nombre de livraisons (en retard / total) :

| Distance | 08-12 | 12-16 | 16-20 |
|---|---|---|---|
| moins de 3 km | 0 / 24 | 0 / 26 | 5 / 24 |
| 3 à 5 km | 9 / 66 | 2 / 77 | 48 / 103 |
| 5 à 7 km | 34 / 112 | 14 / 84 | 80 / 101 |
| plus de 7 km | 19 / 31 | 7 / 13 | 17 / 18 |

Taux de retard :

| Distance | 08-12 | 12-16 | 16-20 |
|---|---|---|---|
| moins de 3 km | 0 % | 0 % | 21 % |
| 3 à 5 km | 14 % | 3 % | 47 % |
| 5 à 7 km | 30 % | 17 % | 79 % |
| plus de 7 km | 61 % | 54 % | 94 % |

- De gauche à droite (même distance, créneau différent) : le taux monte → l'horaire compte.
- De haut en bas (même créneau, distance différente) : le taux monte → le secteur compte.

Contrairement aux chauffeurs, les écarts de distance **restent énormes** à créneau égal. Les deux effets s'additionnent : une longue course le soir est en retard 94 fois sur 100.

## 8. Bilan

| Suspect | Écart seul | À créneau égal | Verdict |
|---|---|---|---|
| Horaires | 11,5 % → 61 % | — | cause |
| Secteurs (distance) | 7 % → 69 % | les écarts restent | cause |
| Chauffeurs | 16 % → 49 % | les écarts se resserrent, tous touchés par le soir | pas une cause |

## 9. Réponse et recommandation

**Réponse.** Les retards sont une affaire d'horaires et de secteurs, pas de personnes. Le soir multiplie le taux de retard par 5 (11,5 % → 61 %), une course de plus de 7 km le multiplie par 10 (7 % → 69 %), et ces deux effets restent vrais l'un sans l'autre. Les 18 chauffeurs sont touchés de la même façon, et l'ancienneté n'a aucun lien avec les retards (corrélation −0,02).

**Décision.**
1. Ne pas lancer le plan de formation : il ne changerait rien.
2. Agir sur le créneau 16-20 : y affecter plus de chauffeurs, ou limiter le nombre de commandes acceptées le soir.
3. Redécouper les tournées pour raccourcir les courses des zones éloignées.
