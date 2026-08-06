---
layout: post
title: "ChatGPT résout un problème majeur en math et révolutionne le domaine de la recherche"
date: 2026-08-06 12:00:00 +0200
slug: probleme_distance_unitaire
categories: [Intelligence Artificielle, Mathématiques, Recherche]
tags: [Math, LLM, IA, ChatGPT, OpenAI, DeepMind, Erdős, Algorithmique, Cryptographie] 
toc: true
image:
  path: /assets/img/unit_distance/ai_math.webp
---

<br>

## Introduction

En mai 2026, **un modèle interne d'OpenAI** résout l'un des **problèmes ouverts d'Erdős**, le **problème 90** sur les distances unitaires. Il devient ainsi le premier modèle à résoudre un problème ouvert considéré comme majeur, en utilisant une approche "**innovante**" jamais explorée par les chercheurs en plus de **80 ans**. En réfutant cette **conjecture d'Erdős** (posée en 1946), cet événement marque un tournant historique dans le domaine de la recherche.

Les **problèmes d'Erdős** sont une collection de **1217 problèmes mathématiques** posés par le célèbre mathématicien hongrois **Paul Erdős**. Ce sont des problèmes simples en apparence, mais extrêmement complexes à résoudre. Au moment où j'écris cet article, **565** problèmes sur **1217** ont été résolus, soit **46%**. 

La résolution de ce problème par une IA est un **tournant majeur dans le domaine des mathématiques et de l'IA** pour plusieurs raisons. La solution a été validée par plusieurs grands experts indépendants de la communauté, tels que les **médaillés Fields** **Timothy Gowers** et **Terence Tao**. C'est le premier résultat produit par une IA qui mériterait une publication dans une **revue mathématique de premier rang** ainsi que la Médaille Fields (l'équivalent du **prix Nobel** en mathématiques) s'il avait été trouvé **par des humains sans IA**. 

<br>

---

<br>

## L'évolution des LLM

* **Novembre 2022** : Au lancement de **ChatGPT** fin novembre 2022, les LLM faisaient beaucoup d'erreurs de calculs et d'arithmétique basique, affirmant par exemple que **"9,11 > 9,9"**. Il leur arrivait souvent d'halluciner des raisonnements complètement incohérents.  

* **Décembre 2023** : **FunSearch** (*searching in the function space*) est un modèle développé par **Google DeepMind** qui résout des problèmes de **maths et d'algorithmie**. Il utilise plusieurs **LLM** combinés à des évaluateurs automatiques pour trouver de nouvelles solutions. Il a permis de résoudre ou d'améliorer plusieurs problèmes, notamment le **problème de capset** et **l'algorithme de bin packing**.

* **Décembre 2024** : **DeepSeekMath-V2** obtient un score de **118 sur 120** au concours **Putnam 2024**, l'un des concours de mathématiques les plus durs au monde, surpassant le **meilleur score de 90 obtenu par un humain** cette année-là. Seules 5 personnes ont obtenu un score parfait depuis la création du concours en 1938, sur plus de 150 000 participants.

* **Juillet 2025** : **DeepMind** (**Gemini 2.5 Deep Think**) et **OpenAI** obtiennent la **médaille d'or** aux **Olympiades mathématiques** en utilisant des modèles généralistes, sans outils externes.

* **6 janvier 2026** : **Kevin Barreto** soumet une solution au **problème 728 d'Erdős** générée par **GPT 5.2 Pro**.

* **11 janvier 2026** : **Neel Somani** soumet une solution au **problème 397 d'Erdős**, également générée par **GPT 5.2 Pro**. 

<br>

---

<br>

## Le problème des distances unitaires

Le problème de **distance unitaire** est un problème simple en apparence, mais bien plus complexe qu'il n'y paraît. En 1946, **Paul Erdős** a posé une question fondamentale d'**optimisation spatiale** : si l'on dispose un grand nombre de points sur une surface plane, combien de paires de points peuvent être séparées par une distance d'exactement une unité ?

Erdős avait trouvé une structure en forme de grille permettant de maximiser le nombre de paires, et en avait déduit qu'il existait un plafond impossible à dépasser.

Son raisonnement s'appuyait sur une contrainte physique de la géométrie en deux dimensions. Pour qu'un point A et un point B soient tous deux à exactement un mètre d'un point C, ce point C doit obligatoirement se trouver à l'intersection de deux cercles d'un mètre de rayon tracés autour de A et B. Or, sur un plan plat, deux cercles distincts ne peuvent se croiser qu'en deux endroits au maximum. Il est donc impossible d'avoir une densité de points élevée tout en respectant cette distance.

![Construction faites par Erdős](/assets/img/unit_distance/construction.svg){: .shadow .rounded-10 }
_Construction précédemment connue permettant d’obtenir un grand nombre de distances unitaires à partir d’une grille carrée mise à l’échelle._

<br>

---

<br>

## La méthodologie de l'intelligence artificielle 

Pendant plus de 80 ans, les chercheurs ont essayé de résoudre ce problème en dessinant des figures de plus en plus complexes pour tenter de trouver la configuration optimale. Le **modèle d'OpenAI** a trouvé une solution en utilisant une **approche plus théorique et abstraite**. 

Plutôt que de chercher à placer des points sur un plan, l'IA a transposé le problème et a exploré une autre branche des mathématiques : la **théorie des nombres** (la branche qui étudie les propriétés des nombres entiers).

* **L'abstraction spatiale :** L'IA a exploré des espaces abstraits comportant de multiples dimensions. Dans ces espaces, certaines contraintes de l'espace physique (comme la limite d'intersection des cercles) ne s'appliquent plus.
* **La structuration algébrique :** En utilisant des théorèmes complexes, l'IA a généré une configuration théorique où un nombre énorme de points était à une unité de distance les uns des autres. Elle a pu contourner la limite de densité physique parce qu'elle manipulait des équations, et non des points sur une surface à deux dimensions.
* **La projection sur le plan réel :** L'étape finale, et la plus décisive, a consisté à prendre cette immense structure abstraite et à la "projeter" mathématiquement pour la ramener sur un plan classique en deux dimensions, de la même manière qu'un architecte réduit un bâtiment 3D sur un plan 2D.

Le modèle d'OpenAI a réfuté l'hypothèse d'Erdős, mais n'a pas trouvé de solution propre au problème initial. C'est la **première fois** qu'un **LLM** résout un problème majeur de façon **autonome et "innovante"**.  

<br>

---

<br>

## Applications et Implications 

Bien qu'il s'agisse d'un problème de mathématiques pures et de géométrie discrète dont les retombées industrielles mettront des années à se matérialiser, les applications de cette découverte — et surtout de la méthode utilisée pour y parvenir — sont très concrètes.

Voici les principales applications et implications de cette résolution :

### 1. La validation de l'IA comme chercheur autonome
L'application immédiate la plus massive n'est pas la géométrie elle-même, mais la démonstration. Jusqu'à présent, l'IA excellait pour synthétiser des connaissances existantes. Ici, pour la première fois, le modèle a connecté deux branches des mathématiques (**la géométrie pure et la théorie algébrique des nombres**) pour générer une nouvelle construction mathématique, indétectée par les humains pendant 80 ans. 

Cela ouvre la voie à l'utilisation de l'IA pour automatiser la recherche fondamentale en physique, en biologie ou en science des matériaux. Il existe déjà des pipelines permettant de vérifier automatiquement les démonstrations mathématiques (comme **Lean** ou **Coq**). L'IA ne remplacera pas complètement le métier de chercheur (qui risque d'évoluer vers un rôle de consultant), mais il faudra toujours un humain pour vérifier que les preuves sont correctes et que les IA n'hallucinent pas.

<br>

### 2. Optimisation des réseaux de télécommunication
Le problème de la distance unitaire est intrinsèquement lié à la **théorie des graphes** (plus précisément les graphes de distance unité). Ces graphes sont utilisés pour modéliser des systèmes où les éléments ont une portée fixe :
* **Réseaux de capteurs sans fil :** Pour déterminer comment placer des capteurs qui ne peuvent communiquer qu'à une distance précise.
* **Allocation de fréquences (Réseaux cellulaires) :** Pour éviter les interférences entre des antennes relais situées à des distances spécifiques les unes des autres. Mieux comprendre les limites géométriques de ces réseaux permet de concevoir des topologies de routage plus efficaces.

<br>

### 3. Avancées en cryptographie et théorie de l'information
Pour réfuter la conjecture, l'IA ne s'est pas contentée de "dessiner" des points ; elle a utilisé des outils mathématiques très complexes issus de la **théorie algébrique des nombres**. Il s'avère que c'est exactement cette même branche qui fonde la cryptographie moderne (notamment la cryptographie sur les **courbes elliptiques** et les algorithmes résistants à l'informatique quantique). Les nouvelles constructions mathématiques découvertes par le modèle pourraient inspirer la création de nouveaux protocoles de chiffrement.

<br>

### 4. Modélisation spatiale (Cristallographie et Chimie)
Dans le monde physique, les atomes et les molécules s'organisent selon des distances très strictes en raison des forces de liaison. Le problème de la distance unité se penche sur la densité maximale de points espacés de manière identique. Les avancées sur ces limites théoriques aident les scientifiques des matériaux à comprendre ou prédire de nouvelles structures cristallines, des alliages ou des configurations moléculaires stables qui respectent ces contraintes de distance spatiale.

> **En résumé**, si la géométrie discrète pure est la première bénéficiaire de cette découverte, les outils mathématiques utilisés par l'IA pour y parvenir auront des applications directes dans la conception de **réseaux informatiques**, la **cryptographie** et la **chimie structurelle**.

<br>

---

<br>

## Conclusion 

C'est la première fois qu'un **LLM** résout un **problème majeur** en mathématiques. Ce qui est impressionnant, ce n'est pas tant la résolution du problème que la qualité du papier de recherche (qui atteint les standards des plus grandes revues mathématiques) et l'approche **innovante** utilisée par le modèle d'**OpenAI** pour réfuter la conjecture, en explorant une piste qui n'avait jamais été creusée par les chercheurs. 

En mathématiques, l'intérêt d'une résolution réside souvent moins dans la solution finale que dans la méthode et les outils développés. Ceux-ci peuvent être réutilisés pour résoudre d'autres problèmes, et permettent parfois de faire le pont entre différentes branches des mathématiques. C'est le cas pour ce problème, qui a permis de faire le lien entre la **géométrie discrète (théorie des graphes) et la théorie des nombres (algèbre abstraite)**.

Cette méthode de raisonnement pourrait, à l'avenir, révolutionner la recherche dans d'autres domaines scientifiques comme **la médecine, la chimie ou la physique**. En utilisant cette **méthode de transposition**, l'IA pourrait explorer des angles d'approche inexplorés par les chercheurs et trouver des solutions à d'autres **défis majeurs** :

<br>

### 1. Chimie et Science des Matériaux
Ce domaine est très proche du problème mathématique initial, car la chimie repose sur des distances et des angles spatiaux entre les atomes.
* **Découverte de nouveaux matériaux :** Pour concevoir des batteries plus denses, des panneaux solaires plus efficaces ou des supraconducteurs, les chercheurs sont limités par des contraintes de structure cristalline en 3D. L'IA pourrait concevoir des alliages dans des espaces abstraits avant de calculer comment les matérialiser dans l'espace physique.
* **Catalyseurs inédits :** En traduisant les contraintes des réactions chimiques en problèmes de topologie, l'IA pourrait découvrir de nouvelles façons de briser ou de lier des molécules, en utilisant des méthodes difficiles à visualiser pour un humain.

<br>

### 2. Médecine et Pharmacologie
La biologie moléculaire est souvent limitée par notre capacité à modéliser des interactions physiques complexes.
* **Conception de médicaments (repliement des protéines) :** La recherche fonctionne souvent sur le modèle "clé-serrure" (l'emboîtement 3D d'une molécule dans un récepteur). Si l'IA traduit ce problème biomécanique en un problème de théorie des graphes abstraits, elle pourrait concevoir des molécules aux architectures inédites, impossibles à simuler classiquement en raison du nombre de possibilités (de l'ordre de $10^{300}$).
* **Maladies multifactorielles :** Les réseaux d'interactions génétiques et protéiques liés aux cancers ou à la maladie d'Alzheimer sont très vastes. Une IA capable de traduire ces réseaux en structures algébriques pourrait y détecter des vulnérabilités ou des schémas cachés.

<br>

### 3. Physique Théorique et Appliquée
La physique bute actuellement sur des barrières théoriques, notamment l'incompatibilité entre la mécanique quantique et la relativité générale pour former un modèle universel.
* **L'unification des lois de la physique :** Tout comme elle a connecté la géométrie discrète et la théorie des nombres, l'IA pourrait trouver un lien entre les équations quantiques et gravitationnelles. En projetant ces théories dans un espace mathématique de dimension supérieure, elle pourrait trouver le cadre où elles coexistent sans contradiction.

**En conclusion**, au-delà de la résolution géométrique, cette avancée montre que l'intelligence artificielle peut passer du statut d'outil de synthèse à celui d'outil de recherche fondamentale autonome, capable de manipuler des concepts bien en dehors de notre perception spatiale habituelle.

<br>

---

<br>

### Sources :
* [OpenAI : Model disproves discrete geometry conjecture](https://openai.com/index/model-disproves-discrete-geometry-conjecture/)
* [Planète Grandes Écoles : Intelligence artificielle résout problème maths conjecture](https://www.planetegrandesecoles.com/intelligence-artificielle-resout-probleme-maths-conjecture)
* [SciTechDaily : For the first time ChatGPT has solved an unproven math problem in geometry](https://scitechdaily.com/for-the-first-time-chatgpt-has-solved-an-unproven-math-problem-in-geometry/)
* [Arxiv (2505.12575v1)](https://arxiv.org/html/2505.12575v1)
* [YouTube (Vidéo explicative)](https://www.youtube.com/watch?v=AYPQIntoJeE&t)
* [New Scientist : Deepmind and OpenAI claim gold in international mathematical olympiad](https://www.newscientist.com/article/2489248-deepmind-and-openai-claim-gold-in-international-mathematical-olympiad/)

### Papiers de recherche : 
* [OpenAI - Unit Distance Proof (PDF)](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-proof.pdf)
* [OpenAI - Unit Distance Remarks (PDF)](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-remarks.pdf)
* [OpenAI - Unit Distance CoT (PDF)](https://cdn.openai.com/pdf/1625eff6-5ac1-40d8-b1db-5d5cf925de8b/unit-distance-cot.pdf)