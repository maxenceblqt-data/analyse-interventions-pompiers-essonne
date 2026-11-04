# Les pompiers de l'Essonne : où interviennent-ils vraiment, et pour quoi ?

Projet d'analyse de données que j'ai réalisé avec Power BI, dans le cadre de ma reconversion vers le métier de data analyst. C'est un projet d'apprentissage : je débute, et l'idée était surtout de m'entraîner sur des vraies données plutôt que sur un jeu de données tout propre.

---

## La question que je me suis posée

Au départ, j'avais deux idées reçues en tête, un peu comme tout le monde je pense :

1. Les pompiers, ça éteint surtout des incendies.
2. Ils interviennent forcément plus dans les grandes villes.

J'ai voulu vérifier si c'était vrai. Ça m'a donné une problématique simple à creuser :

> **Où et pour quoi les pompiers de l'Essonne interviennent-ils le plus et est-ce que ça correspond à ce qu'on imagine ?**

Spoiler : les deux idées de départ se sont révélées fausses, ou au moins beaucoup plus compliquées que ça. Et c'est justement ce qui a rendu le projet intéressant.

---

## Les données que j'ai utilisées

Pour répondre, il me fallait deux choses.

D'abord les interventions elles-mêmes. Je les ai trouvées sur **data.gouv.fr** : un fichier qui recense, semaine par semaine et commune par commune, le nombre d'interventions des pompiers de l'Essonne entre 2010 et 2017, avec le type d'intervention. Ça fait quand même 455 794 lignes.

Ensuite, comme je voulais comparer les communes, il me fallait leur population. Je l'ai récupérée sur le site de l'**INSEE** (populations légales 2017).

| Source | Ce que ça contient |
|--------|--------------------|
| [data.gouv.fr](https://www.data.gouv.fr/datasets/interventions-des-pompiers/) | Interventions par commune et par semaine (2010-2017) |
| [INSEE](https://www.insee.fr/fr/statistiques/4265429) | Population de chaque commune |

Outils : **Power BI Desktop** (Power Query pour le nettoyage, DAX pour les calculs, et la partie visualisation).

---

## Comment je m'y suis pris

### 1. Nettoyer les données

<img width="1339" height="690" alt="image" src="https://github.com/user-attachments/assets/0d21becb-5a79-4e1a-9a24-21de527c6eb8" />


Rien n'était prêt à l'emploi. J'ai dû corriger pas mal de choses dans Power Query : des colonnes de chiffres qui étaient vues comme du texte (et du coup les calculs plantaient), et remplacer les codes du fichier par des mots lisibles (par exemple "SUAP" voulait dire "Secours à personne").

### 2. Relier les deux tables

<img width="849" height="393" alt="image" src="https://github.com/user-attachments/assets/d96a34f9-51e5-4ef1-a69b-eeccf7b15bf0" />


Pour pouvoir croiser interventions et population, j'ai relié mes deux fichiers grâce au **code INSEE** de chaque commune, qui est présent dans les deux.

### 3. Faire les calculs (DAX)

J'ai créé quelques mesures, dont le total des interventions :

```dax
Total_interventions = SUM('interventions-hebdo-2010-2017'[nb_ope])
```

Et surtout le nombre d'interventions rapporté à la population (j'explique pourquoi plus bas) :

```dax
Interv_pour_1000hab = DIVIDE([Total_interventions], SUM(Communes[PMUN])) * 1000
```

### 4. Construire le tableau de bord

<img width="1445" height="811" alt="image" src="https://github.com/user-attachments/assets/53c6114e-768e-4d94-a74b-22bfb1cb6b6e" />

---

## Ce que j'ai trouvé

**Première surprise : les pompiers n'éteignent pas surtout des feux.** 78,5 % des interventions sont du **secours à personne** (malaises, accidents, aide aux gens). Les incendies, c'est une petite partie seulement. Ça casse complètement l'image qu'on en a.

**Deuxième surprise : les grandes villes ne sont pas forcément les plus "sous tension".** En nombre brut, oui, Corbeil-Essonnes et Évry arrivent en tête. Mais c'est logique : il y a plus d'habitants, donc plus d'interventions. Ça n'apprend rien.

C'est là que j'ai compris un truc important : pour comparer des communes de tailles différentes, il faut **ramener les interventions à la population**. C'est ce qu'on appelle normaliser. J'ai donc calculé un **taux d'interventions pour 1000 habitants**, pour mettre toutes les communes sur un pied d'égalité.

<img width="1435" height="803" alt="image" src="https://github.com/user-attachments/assets/15c19ff4-c9f7-4346-804e-53024dc3dc15" />


Et là, le classement change complètement.

---

## Et c'est là que ça devient intéressant (les pièges)

En regardant ce nouveau classement, je me suis méfié, parce que certains chiffres étaient énormes. En creusant, j'ai trouvé trois pièges et c'est ça qui m'a le plus appris sur le métier.

**Piège 1 — je comparais des années différentes.** Mes interventions couvraient 8 ans, mais ma population n'était que celle de 2017. Donc je divisais 8 ans d'interventions par 1 an de population, ce qui gonflait tout. Je l'ai corrigé en ne gardant que l'année 2017 pour le calcul, grâce à la fonction `CALCULATE`.

**Piège 2 — les tout petits villages faussent le classement.** Dans une commune de 200 habitants, quelques interventions suffisent à faire exploser le taux. Ces chiffres-là sont instables, donc à prendre avec des pincettes.

**Piège 3 — les gens de passage.** Juvisy-sur-Orge ressort très haut, mais c'est une grosse gare. Les malaises de voyageurs comptent dans ses interventions, sauf que ces voyageurs n'habitent pas Juvisy. Du coup je rapporte des interventions de gens de passage à la population des seuls habitants : forcément, ça gonfle le taux. Le chiffre est "juste" mais il veut dire autre chose que ce qu'on croit.

---

## Les limites (et ce que je ferais avec plus de temps)

Mon taux reste faussé pour les communes avec beaucoup de passage (gares, grands axes routiers). Pour faire vraiment propre, il faudrait tenir compte des gens de passage par exemple en récupérant la fréquentation des gares (données SNCF) et le trafic routier, pour avoir une "population exposée" plus réaliste.

Le truc que je retiens surtout de ce projet : **le calcul en lui-même est rarement le problème. Le plus dur, et le plus intéressant, c'est de comprendre ce que le chiffre veut vraiment dire.**

---

## Ce sur quoi j'ai galéré

<!-- Section à compléter avec mes propres mots -->

- Des colonnes de chiffres lues comme du texte, qui faisaient planter les calculs.
- Les codes INSEE : en les transformant en nombre, le zéro de devant disparaissait, et je me suis retrouvé avec des communes de l'Ariège au lieu de l'Essonne. J'ai mis un moment à comprendre pourquoi.
- La mise en forme des graphiques dans Power BI, pas toujours intuitive.

---

## La suite

J'aimerais faire ensuite un projet plus orienté **SQL**, pour m'entraîner sur l'autre grosse compétence du métier.
