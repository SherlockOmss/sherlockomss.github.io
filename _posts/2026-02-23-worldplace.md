---
layout: post
title: "WorldPlace : Retrouver n'importe quel endroit dans le monde à partir d'une image"
date: 2026-02-23 12:00:00 +0100
slug : worldplace
toc: true
image:
  path: /assets/img/worldplace/cover_eye.jpg
  alt: Un œil reflétant la Terre, image de couverture du projet WorldPlace.
---

<br>

## Introduction

Le jeu GeoGuessr est devenu très populaire ces dernières années, rassemblant une vaste communauté autour d'un défi simple: à partir d'une image Street View prise n'importe où sur Terre, est-il possible de retrouver sa localisation précise? À l'échelle de la planète, la diversité des architectures, les changements de saison, le temps, l'angle de vue, ou les impacts du climat et beaucoup d'autres facteurs font de la géolocalisation précise un problème complexe et encore largement irrésolu. 

Si plusieurs modèles d'intelligence artificielle ont tenté de relever le défi, ils présentent souvent des défauts majeurs. Le modèle que j'ai choisi pour mon projet, intitulé World Place, pallie une partie de ces défauts. Ce document a pour but de vulgariser le fonctionnement du modèle PIGEON (Predicting Image Geolocation) afin de le rendre accessible, même pour un novice en intelligence artificielle (Deep Learning, Vision par ordinateur). 

Pour mon projet je l'ai adapté et entraîné sur mon propre jeu de données (dataset). En raison de la complexité du sujet et du temps limité, le projet se concentre pour le moment sur Paris et sa petite couronne. Mon dataset est composé de 50 000 images issues de Mapillary, avec un entraînement de 20 cycles (epochs) pour cette première version.

Le modèle PIGEON a été développé par 3 étudiants de l'université de Stanford et est entraîné sur des images panoramiques à 360° degrés issues de Google Street View. C'est le premier modèle capable de battre les meilleurs joueurs mondiaux de GeoGuessr de manière fiable, se classant parmi le top 0,01% des joueurs et ayant battu l'expert Rainbolt lors de six parties consécutives.

Pour approfondir, vous pouvez consulter le rapport officiel du projet PIGEON ici : [Lien vers le PDF d'origine (arXiv)](https://arxiv.org/pdf/2307.05845) et explorer le code source sur leur dépôt GitHub officiel : [https://github.com/LukasHaas/PIGEON](https://github.com/LukasHaas/PIGEON).

<br>

![Architecture globale du modèle PIGEON](/assets/img/worldplace/pigeon_architecture.png){: .shadow .rounded-10 }
_Figure 1 : Vue d'ensemble des 4 étapes clés du modèle PIGEON (Découpage, Analyse, Calcul d'erreur, Raffinement)._

<br>

---

<br>

## 1. Le découpage de la carte : Passer d'une recherche infinie à un choix ciblé

Le principal obstacle à la géolocalisation automatique est l'immensité de l'espace de recherche. Demander à un modèle de pointer un endroit précis sur une carte du monde sans guide revient à lui demander de trouver une aiguille dans une botte de foin de la taille de la Terre : les possibilités sont infinies, et la marge d'erreur est immense. Pour bien mesurer la difficulté, chercher une position précise au mètre près sur la planète revient à tenter de deviner un mot de passe complexe en utilisant de la force brute. Sans découpe, le modèle devrait choisir parmi plus de 500 millions de milliards de points à l'échelle mondiale.

Pour simplifier cette tâche, le modèle utilise une technique de discrétisation. Au lieu de laisser le modèle naviguer sur une carte "lisse" et continue, nous transformons le terrain en cellules. **Pour faire une analogie, chercher une localisation exacte sur Terre revient à chercher un ouvrage précis dans une bibliothèque infinie. Plutôt que d'examiner chaque livre un par un au hasard, ce système permet d'identifier d'abord la bonne étagère (la *Geocell*). Une fois l'étagère trouvée, dénicher le livre exact devient infiniment plus simple.**

* **La création de cellules :** La carte est découpée en milliers de zones distinctes, les Geocells (cellules géographiques).
* **La simplification du problème :** Grâce à ce découpage, le modèle ne cherche plus une coordonnée au hasard. Le modèle fait face à un "QCM spatial" : son rôle est de classer l'image dans la case qui lui correspond le mieux. En transformant ce problème en un choix parmi quelques milliers de cellules, nous réduisons la complexité d'un facteur immense, permettant au modèle de se concentrer sur la reconnaissance de zones cohérentes plutôt que sur une recherche mathématique infinie.
* **L'élagage des données inutiles :** Cette méthode permet de procéder par élimination. Dès que le modèle identifie la cellule la plus probable, il peut élaguer (supprimer) tout le reste de la carte de sa réflexion. Ce mécanisme de filtre permet au modèle de concentrer toute son attention sur les détails visuels propres à la zone sélectionnée, augmentant ainsi drastiquement sa précision. Le découpage en Geocells agit comme un guide : il réduit l'espace de recherche à quelques dizaines de milliers de choix seulement, transformant un défi mathématique insurmontable en un problème de classification rapide et efficace.

<br>

### 1.1 Évolution : Des carrés rigides au découpage sur-mesure

Avant PIGEON, les chercheurs utilisaient principalement deux méthodes pour découper la carte, mais elles présentaient des limites :
* **Le quadrillage classique :** On découpait la Terre en carrés parfaits. Le problème est que ces carrés ne respectent pas les frontières naturelles (une frontière, un fleuve ou une chaîne de montagne peut se retrouver coupée en deux).
* **Le découpage arbitraire :** On créait des zones de tailles égales en nombre de photos, mais sans logique géographique.

L'innovation de PIGEON est d'utiliser un découpage sémantique : le puzzle suit les vraies frontières (villes, arrondissements) car c'est là que l'apparence des rues change le plus.

<br>

### 1.2 Le Clustering : Repérer les zones d'intérêt

Dans une ville comme Paris, les photos s'accumulent massivement autour des monuments. Pour gérer cela, le modèle utilise un algorithme de Clustering (regroupement), appelé OPTICS (ordering points to identify the clustering structure).
* **Le concept :** L'algorithme scanne la carte et repère les endroits où les photos se "regroupent".
* **L'utilité :** Cela permet au modèle de comprendre où se situent les zones riches en détails. Si une rue possède 1 000 photos, il est logique de lui créer une cellule spécifique minuscule pour être très précis.

<br>

### 1.3 La Tessellation de Voronoï : Tracer des frontières intelligentes

Une fois les points d'intérêt repérés, il faut tracer les limites physiques de chaque pièce du puzzle. On utilise une méthode mathématique appelée la Tessellation de Voronoï.

<br>

![Comparaison entre les cellules rectangulaires et les cellules sémantiques sur Paris](/assets/img/worldplace/semantic_geocells_paris.png){: .shadow .rounded-10 }
_Figure 2 : À gauche un découpage classique, à droite le découpage sémantique (Voronoï) de WorldPlace sur Paris et sa couronne._

<br>

* **Le fonctionnement :** Imaginez que chaque groupe de photos est un "épicentre". La méthode de Voronoï trace des frontières de sorte que chaque point à l'intérieur d'une cellule soit rattaché au centre le plus proche.
* **Le résultat :** On obtient un puzzle aux formes irrégulières. Ce système est bien plus efficace qu'une grille de carrés, car il s'adapte à la géométrie réelle de la ville.

<br>

### 1.4 L'apport majeur de ce modèle

L'amélioration principale par rapport aux travaux précédents est la flexibilité. En combinant les frontières réelles (villes, pays) et le découpage mathématique (Voronoï), PIGEON réussit à créer un puzzle qui :
* **Équilibre les données :** Aucune pièce du puzzle n'est "vide" ou "trop pleine", ce qui aide le modèle à apprendre plus vite.
* **Respecte le sens :** Les cellules ne traversent pas les frontières importantes, ce qui évite au modèle de confondre deux pays ou deux quartiers radicalement différents.

<br>

---

<br>

## 2. L'analyse de l'image : L'architecture StreetCLIP

Après avoir défini le cadre géographique (les Geocells), le système doit être capable d'interpréter le contenu visuel d'une image pour le lier à une position. Cette étape cruciale repose sur un moteur de vision artificielle nommé StreetCLIP.

<br>

![Architecture du modèle StreetCLIP](/assets/img/worldplace/street_clip_architecture.png){: .shadow .rounded-10 }
_Figure 3 : Principe de l'apprentissage par contraste (Contrastive Pre-training) liant le texte à l'image._

<br>

### 2.1 Le principe du modèle CLIP : relier l’image à son contexte

L'architecture de base utilisée est CLIP (Contrastive Language-Image Pre-training). Contrairement aux modèles classiques qui se contentent de nommer des objets (ex: "ceci est un arbre"), CLIP est conçu pour comprendre les relations entre les images et le langage.
* **Analyse du contexte (Apprentissage par contraste) :** Le modèle a été entraîné sur des millions de paires "image-texte". Son rôle est d’apprendre à identifier le contexte et faire des liens logiques :
  * S'il voit un immeuble haussmannien, il le place mathématiquement près de l'étiquette "Paris".
  * À l'inverse, il apprend à repousser très loin les éléments qui n'ont rien en commun (par exemple, un palmier loin de l'étiquette “Norvège”).
* **Une compréhension globale :** Grâce à cette méthode, le modèle ne se contente pas de mémoriser des pixels ; il développe une véritable culture visuelle du monde, capable de lier un style architectural à un concept géographique.

<br>

### 2.2 La spécialisation géographique : StreetCLIP

Pour les besoins de la géolocalisation, le modèle CLIP standard a été affiné (on parle de fine-tuning) pour devenir StreetCLIP. L'objectif est de transformer un modèle généraliste en un expert en "lecture du paysage".

PIGEON a été entraîné en utilisant des légendes synthétiques enrichies de données auxiliaires. Le modèle apprend ainsi à identifier des indices géographiques clés que l'œil humain pourrait négliger :
* **Données climatiques :** Distinction entre les végétations tropicales, tempérées ou arides.
* **Indices d'infrastructure :** Reconnaissance du côté de conduite, du style des panneaux de signalisation ou du marquage au sol.
* **Contexte temporel :** En corrélant la date de capture de l'image (présente dans les métadonnées de Street View) avec l'aspect visuel (neige, feuilles mortes, ensoleillement), le modèle comprend que ces éléments sont des variables cycliques et non des caractéristiques permanentes du terrain.

La saisonnalité est un piège classique pour les IA de géolocalisation :
* **La neige :** Sans gestion des attributs temporels, un modèle pourrait placer systématiquement une image enneigée dans les pays nordiques ou à haute altitude.
* **L'ensoleillement :** Un fort ensoleillement zénithal pourrait fausser le calcul de la latitude vers l'équateur.

<br>

### 2.3 Extraction de caractéristiques et "Embeddings"

Lorsqu'une image est soumise au projet World Place, elle passe par un encodeur visuel qui la traduit en une signature numérique appelée Embedding.
* **Comparaison vectorielle :** Cette signature est ensuite comparée aux signatures des zones géographiques connues.
* **Identification de la zone :** Le modèle ne cherche pas une correspondance exacte, mais calcule la "proximité" entre la signature de l'image de test et celle des différentes Geocells. Plus les signatures sont proches, plus la probabilité que l'image appartienne à cette cellule est élevée.
* **Application au projet World Place :** Dans le cadre de ce travail sur Paris et sa couronne, StreetCLIP permet de distinguer des nuances architecturales subtiles, comme la différence entre un immeuble pierre de taille du centre-ville et les structures pavillonnaires ou industrielles de la périphérie, en traduisant ces éléments visuels en probabilités géographiques.

<br>

---

<br>

## 3. La fonction de perte : L’apprentissage de la proximité géographique

Une fois que le modèle a formulé une prédiction (le choix d'une cellule), il est nécessaire de mesurer l'écart entre cette réponse et la localisation réelle. Dans le domaine du Deep Learning, cette mesure est effectuée par une fonction de perte (loss function). C'est cet indicateur qui guide l'apprentissage du modèle.

<br>

### 3.1 Les limites des approches classiques

Traditionnellement, les modèles de classification traitent les erreurs de manière binaire : une réponse est soit correcte, soit erronée.
* **Le problème de l'isolement :** Dans un système standard, si le modèle sélectionne une cellule voisine de la cible, il est pénalisé de la même manière que s'il avait choisi un continent différent.
* **La conséquence :** Le modèle ne comprend pas la notion de "proximité". Il apprend ses représentations de manière isolée, sans saisir la continuité géographique du monde.

<br>

### 3.2 L'intégration de la Distance de Haversine

Pour pallier cette lacune, le modèle s'appuie sur la formule de Haversine. Cette équation mathématique permet de calculer la distance la plus courte entre deux points sur une sphère, on appelle cela une géodésique c'est-à-dire la distance réelle "à vol d'oiseau" sur la courbure terrestre.

<br>

![Représentation de la distance géodésique de Haversine sur une sphère](/assets/img/worldplace/haversine_distance.png){: .shadow .rounded-10 }
_Figure 4 : Calcul de la distance géodésique entre deux points P et Q._

<br>

* **L'apport technique :** En intégrant cette formule, le modèle "prend conscience" de la géographie. Il n'analyse plus des étiquettes abstraites, mais des positions liées par des distances kilométriques concrètes.

<br>

### 3.3 Le "Lissage" (Haversine Smoothing) : Une innovation majeure

L'innovation de PIGEON réside dans l'utilisation du Lissage Haversine (Haversine Smoothing). Au lieu de valider uniquement la cellule exacte (le principe du "tout ou rien"), le modèle distribue la probabilité sur les cellules environnantes en fonction de leur distance.
* **Le principe du lissage :** Si le modèle pointe une cellule limitrophe, le système considère que l'erreur est mineure. On lui attribue une "note partielle" positive. Plus la cellule prédite est éloignée de la réalité, plus la pénalité augmente de façon exponentielle.
* **L'impact sur l'apprentissage :** Cette méthode encourage le modèle à être cohérent. Elle lui apprend que le monde est un espace continu. Cela stabilise les prédictions : en cas d'incertitude visuelle, le modèle privilégiera une zone à proximité plutôt qu'une localisation totalement aberrante à l'autre bout de la planète.
* **Application au projet World Place :** Dans le contexte de Paris et de sa couronne, ce système est déterminant. De nombreux quartiers présentent des similitudes architecturales fortes. Le lissage permet au modèle de comprendre que se tromper entre deux arrondissements voisins est une erreur non négligeable, ce qui affine la fiabilité globale des résultats.

<br>

![Impact du lissage Haversine sur des cellules voisines à Accra, Ghana](/assets/img/worldplace/haversine_ghana.jpg){: .shadow .rounded-10 .w-100 }
_Figure 5 : Impact de l'application du lissage Haversine sur des cellules géographiques voisines pour une localisation à Accra, Ghana (à gauche sans lissage, à droite avec lissage)._

<br>

---

<br>

## 4. Le Raffinement : Passer de la zone au point précis

Une fois que le modèle a réduit le champ de recherche à une cellule de Voronoï spécifique on passe à l'étape suivante que l’on appelle la classification, il doit affiner son analyse pour passer d'une zone géographique de plusieurs kilomètres à une coordonnée exacte. Pour ce faire, le système abandonne la logique de "choix multiple" pour une approche hybride mêlant comparaison visuelle et calcul mathématique.

<br>

### 4.1 Le concept des "Groupes de référence" (Clustering)

Pour gagner en précision, chaque cellule de Voronoï est elle-même subdivisée en "groupes de référence". Si la classification globale permet de trouver l’étagère (la cellule), ces groupes permettent d’identifier le livre exact. Le modèle ne se contente plus de situer l’image dans une région ; il la rattache à une grappe de photos dont les signatures numériques (embeddings) sont très proches. Cela permet de créer des points de repère ultra-locaux au sein même d'un quartier ou d'ville.

<br>

### 4.2 La recherche par similitude

À cette étape, le modèle effectue ce que l'on appelle du *retrieval* (récupération d'information). Il compare l'image inconnue aux milliers d'images de son dataset de référence situées dans la zone présélectionnée. En calculant la distance mathématique entre les vecteurs de l'image de test et ceux des images connues, l'IA identifie les voisins les plus probables. Plus l'environnement architectural est unique, plus cette recherche de similitude est performante.

<br>

### 4.3 L'extraction des coordonnées finales

Comment PIGEON atteint-il une précision chirurgicale ? Pour passer d'une simple estimation à une localisation au mètre près, le modèle ne se contente pas de deviner une zone ; il effectue un calcul mathématique de précision en deux étapes clés :

**1. Le passage du "secteur" au "point précis"**
* Au lieu de simplement classer une image dans une catégorie (ex: "Paris"), le modèle utilise une méthode de moyenne pondérée.
* **Pourquoi "généralement" ?** Dans la plupart des cas, l'IA identifie plusieurs points de repère proches. Elle calcule alors sa position en fonction de sa distance relative par rapport à eux. Cependant, si elle reconnaît un élément unique et sans ambiguïté (un monument spécifique ou une plaque de rue précise), elle peut pointer directement ce lieu sans avoir besoin de faire de moyenne.

**2. L'importance des "Voisins" et du "Sous-Cluster"**
La précision finale repose sur une structure hiérarchique :
* **Le Sous-Cluster :** Avant de calculer les coordonnées, l'IA réduit son champ de recherche à un "sous-cluster" (un petit groupement d'images très denses géographiquement). Plus ce groupe est compact et riche en images, plus l'IA a de points de comparaison.
* **Les "k" voisins les plus proches :** Une fois dans ce groupe, l'IA sélectionne un nombre défini de points (les "k voisins") qui ressemblent le plus à l'image analysée.
  * Si elle prend trop de voisins, la précision se dilue.
  * Si elle en prend trop peu, elle risque de copier une erreur de géolocalisation d'une seule image.
* Le réglage optimal permet à l'IA de "trianguler" sa position : elle déduit qu'elle se trouve, par exemple, à 60% de distance du voisin A et 40% du voisin B.
* **Application au projet World Place :** Dans mon travail sur Paris, cette étape est capitale. Elle permet au modèle de ne pas se contenter de dire "C'est à Boulogne-Billancourt", mais de pointer précisément vers un carrefour ou une rue spécifique en reconnaissant les motifs visuels (façades, mobilier urbain) qu'il a déjà rencontrés dans son dataset de 50 000 images.

<br>

---

<br>

## 5. La "Méta" : Les indices cachés de Google Street View

Le modèle a développé une logique de déduction similaire à celle des joueurs experts sur Geoguessr. Il est capable de détecter des indices plus subtils, souvent liés au contexte légal ou technique de la capture. Voici quelques exemples concrets identifiés par l'IA :

<br>

### Les contraintes législatives

Dans certains pays, la législation oblige Google à modifier sa façon de capturer ou diffuser les images.

<br>

#### L'Allemagne ("Blurmany")
Le modèle identifie le taux très élevé de floutage des immeubles, résultant d'une loi permettant aux citoyens de s'opposer à la publication de leur domicile.

<br>

![Bâtiments floutés en Allemagne](/assets/img/worldplace/allemagne.jpg){: .shadow .rounded-10 .w-100 }

<br>

#### La Suisse (et le Japon / l'Autriche)
La loi interdisant de filmer les espaces privés non visibles depuis la rue, Google utilise souvent une "Low-Cam" (caméra placée plus bas). Le modèle détecte ce changement de perspective.

<br>

![Caméra basse (Low-Cam) en Suisse](/assets/img/worldplace/suisse.png){: .shadow .rounded-10 .w-100 }

<br>

### Les protocoles de sécurité

#### Le Nigeria
C'est le seul pays où l'escorte par un véhicule de police ou de sécurité est systématique et visible dans le champ de vision de la caméra.

<br>

![Voiture de police escortant la Google Car au Nigeria](/assets/img/worldplace/nigeria.jpg){: .shadow .rounded-10 .w-100 }

<br>

### L'équipement du véhicule (Car Meta)

Le modèle apprend à identifier le véhicule porteur, dont certains éléments ne sont pas totalement effacés par l'algorithme.

<br>

#### La Mongolie
Le modèle reconnaît facilement les barres de toit spécifiques. Souvent, elles transportent un équipement de camping volumineux, généralement recouvert d'une bâche de protection caractéristique, adapté aux longues pistes tout-terrain du pays.

<br>

![Équipement sur le toit de la voiture en Mongolie](/assets/img/worldplace/mongolie.jpg){: .shadow .rounded-10 .w-100 }

<br>

### L'attention visuelle de l'IA

Sur ces images on peut voir les zones qui attirent l'attention du modèle pour retrouver la géolocalisation. On peut voir qu'elle observe la végétation et l'architecture, mais on peut également voir plein de points isolés : ce sont le flou de la caméra. Le modèle a appris de lui-même à utiliser le flou de la caméra comme indice, et d'autres stratégies similaires à un joueur humain en les perfectionnant.

<br>

![Cartes d'attention visuelle montrant les points d'intérêt de l'IA](/assets/img/worldplace/attention_maps.png){: .shadow .rounded-10 .w-100 }
_Figure 6 : Cartes d'attention (Heatmaps) révélant les zones analysées par le modèle, comme la végétation ou les flous de caméra._

<br>

---

<br>

## 6. Évaluation des performances et analyse comparative

L'efficacité du framework ne se mesure pas uniquement par un score brut, mais par sa capacité à surpasser les standards actuels de l'intelligence artificielle et de l'expertise humaine. Pour valider cette avancée, les chercheurs ont développé deux modèles complémentaires : PIGEON (spécialisé Street View) et PIGEOTTO (généraliste).

<br>

### PIGEOTTO : La généralisation à l'échelle planétaire

Alors que PIGEON est un spécialiste de l'environnement urbain (Street View), PIGEOTTO a été conçu pour être capable de localiser n'importe quelle image du globe. Ses performances marquent un tournant dans la recherche en géolocalisation d'images :

* **Un entraînement sur des données variées :** Contrairement à PIGEON, PIGEOTTO est entraîné sur un dataset massif de 4 millions d'images provenant de Flickr et Wikipedia. Cela lui permet de comprendre des contextes visuels variés (paysages naturels, monuments, photos de vacances) que l'on ne trouve pas forcément sur Street View.
* **Domination des Benchmarks SOTA :** Le modèle a été testé sur des jeux de données de référence comme Im2GPS et Im2GPS3k. Les résultats sont sans appel :
  * **Niveau Pays :** Il surpasse le précédent SOTA avec un bond spectaculaire de 38,8 points de pourcentage.
  * **Niveau Ville :** Il améliore la précision de 7,7 points, une progression significative dans un domaine où les gains se mesurent habituellement en fractions de points.
* **La percée de la "Généralisation" :** L'innovation majeure de PIGEOTTO est sa capacité à généraliser à des lieux non visités. Auparavant, les modèles avaient tendance à "mémoriser" les lieux présents dans leur base d'entraînement. PIGEOTTO est le premier modèle capable d'analyser des caractéristiques visuelles universelles pour déduire une position sur des sites géographiques qu'il n'a jamais rencontrés lors de son apprentissage.
* **Une architecture robuste :** En utilisant la même base de vision (CLIP) mais adaptée à des images non panoramiques, PIGEOTTO prouve que le framework hiérarchique (pays -> région -> cluster) est la méthode la plus efficace pour la géolocalisation à l'échelle de la planète.

<br>

### Statistiques de performance (PIGEON)

Sur l'ensemble de test massif de 5 000 images Street View, PIGEON affiche des résultats remarquables :
**Score GeoGuessr Moyen :** Le modèle atteint une moyenne de 4 525 points sur un maximum de 5 000. Ce score reflète une régularité exceptionnelle sur des environnements très diversifiés.

| Rayon de distance | Taux de Réussite |
| :--- | :--- |
| **Rue** (1 km) | 5,36% |
| **Ville** (25 km) | 40,36% |
| **Région** (200 km) | 78,28% |
| **Pays** (750 km) | 94,52% |
| **Continent** (2500 km) | 98,56% |

<br>

### L'analyse d'ablation : les piliers de la performance

L'étude d'ablation (retrait progressif des composants) confirme que la supériorité du modèle repose sur des choix architecturaux précis :

<br>

![Tableau des performances et étude d'ablation du modèle PIGEON](/assets/img/worldplace/performances_metrics.png){: .shadow .rounded-10 }
_Figure 7 : Étude d'ablation démontrant l'impact critique de chaque composant (comme le lissage Haversine) sur la précision géographique._

<br>

* **Le Panorama (Vue à 360°) :** L'utilisation de 4 images au lieu d'une seule réduit l'erreur médiane de 131,1 km à 44,35 km. La vision périphérique est donc cruciale pour la précision.
* **Le Raffinement par voisinage (k plus proches voisins) :** Sans le calcul final basé sur les voisins les plus proches, la précision au kilomètre près chute de 5,36 % à 1,32 %.
* **La Fondation StreetCLIP :** Le pré-entraînement spécialisé est le moteur de la réussite ; il permet au modèle d'identifier le bon pays dans 91,96 % des cas.

<br>

### Le benchmark humain : La validation par un expert

Le duel face à Trevor Rainbolt, référence mondiale du jeu GeoGuessr, a servi de validation finale. En remportant 6 matchs sur 6 avec une moyenne de 4 525 points par round, PIGEON a prouvé que sa combinaison de classification sémantique et d'interpolation mathématique surpasse l'intuition humaine la plus affûtée.

<br>

---

<br>

## Conclusion : Des limites locales aux enjeux de sécurité mondiale

Le projet World Place a permis de mettre en lumière la complexité des systèmes de géolocalisation par intelligence artificielle. Si le modèle PIGEON original affiche des performances impressionnantes à l'échelle mondiale, l'adaptation de cette technologie à une échelle locale (Paris et sa petite couronne) a révélé plusieurs défis majeurs qui expliquent une précision bien plus limitée qu'espérée en pratique.

D'une part, l'homogénéité architecturale de la capitale (bâtiments haussmanniens, mobilier urbain standardisé) prive le modèle des contrastes forts dont il a besoin pour se repérer, rendant la tâche de classification extrêmement ardue. D'autre part, le bruit des données issues de contributeurs Mapillary (caméras de qualités variées, angles obstrués) dégrade l'extraction des caractéristiques visuelles. Enfin, l'entraînement d'une architecture CLIP nécessite des ressources de calcul massives et un temps long qui limitent les capacités d'ajustement d'un projet étudiant.

Malgré ces obstacles pratiques à mon échelle, ce projet constitue une preuve de concept solide sur l'efficacité de cette architecture. Il démontre surtout une chose cruciale : si un tel système pose les bases d'une géolocalisation de précision pour un étudiant, son potentiel entre les mains d'acteurs étatiques est terrifiant. Ce contraste nous amène à une problématique éthique de premier plan.

Il est important de rappeler que la quasi-totalité des recherches sur la géolocalisation d'images entre 2018 et 2023 ont été financées par l'armée. Il est à la fois fascinant et effrayant d'observer comment l'on peut passer d'un outil développé pour exceller à un simple jeu (GeoGuessr) à une potentielle arme de destruction massive.

Cet outil est en effet d'une telle puissance que les chercheurs de Stanford ont pris la décision radicale de ne pas publier les paramètres complets (poids) de leur modèle, ni de mettre leur dataset en libre accès. Les systèmes balistiques modernes s'appuient aujourd'hui massivement sur les réseaux satellites pour se géolocaliser et calculer leur trajectoire. Face à des brouilleurs de signaux, ces missiles perdent leur position et finissent par s'écraser. En intégrant un modèle de vision par ordinateur aussi performant que PIGEON, un missile pourrait utiliser son propre flux vidéo pour analyser la topographie survolée et recalibrer sa trajectoire en totale autonomie, devenant ainsi virtuellement insensible au brouillage électronique.

En faisant le choix conscient de garder leur modèle fermé, l'équipe de recherche a très probablement retardé le développement de ces technologies de guidage militaire d'une dizaine d'années. Par cette seule décision éthique de ne pas publier l'intégralité de leurs travaux, ils ont potentiellement sauvé des milliers de vies.

<br>

---

**La vidéo qui m'a inspiré pour ce projet :** [Trouver n’importe quel endroit en 0.1 seconde](https://www.youtube.com/watch?v=t-6ILUMrxWg) (par _What a Fail !_).

Merci d'avoir lu jusqu'au bout ! Si vous avez des questions ou que vous voulez discuter, n'hésitez pas à me contacter sur Discord : **omss**.