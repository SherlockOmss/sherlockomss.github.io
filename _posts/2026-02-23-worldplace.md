---
layout: post
title: "WorldPlace : Retrouver n'importe quel endroit dans le monde à partir d'une image"
date: 2026-02-23 12:00:00 +0100
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

Le principal obstacle à la géolocalisation automatique est l'immensité de l'espace de recherche. Demander à un modèle de pointer un endroit précis sur une carte du monde sans guide revient à lui demander de trouver une aiguille dans une botte de foin de la taille de la Terre: les possibilités sont infinies, et la marge d'erreur est immense.

Pour simplifier cette tâche, le modèle utilise une technique de discrétisation. Au lieu de laisser le modèle naviguer sur une carte "lisse" et continue, nous transformons le terrain en cellules.

* **La création de cellules :** La carte est découpée en milliers de zones distinctes, les Geocells.
* **La simplification :** Le modèle fait face à un "QCM spatial": son rôle est de classer l'image dans la case qui lui correspond le mieux.
* **L'élagage :** Dès que le modèle identifie la cellule la plus probable, il peut élaguer (supprimer) tout le reste de la carte de sa réflexion.

<br>

### De la grille rigide à la Tessellation de Voronoï

Avant PIGEON, les chercheurs utilisaient principalement le quadrillage classique. Le problème est que ces carrés ne respectent pas les frontières naturelles (une frontière, un fleuve ou une chaîne de montagne peut se retrouver coupée en deux). L'innovation de PIGEON est d'utiliser un découpage sémantique: le puzzle suit les vraies frontières (villes, arrondissements) car c'est là que l'apparence des rues change le plus.

<br>

![Comparaison entre les cellules rectangulaires et les cellules sémantiques sur Paris](/assets/img/worldplace/semantic_geocells_paris.png){: .shadow .rounded-10 }
_Figure 2 : À gauche un découpage classique, à droite le découpage sémantique (Voronoï) de WorldPlace sur Paris et sa couronne._

<br>

Pour y parvenir, le modèle s'appuie sur deux algorithmes :
1.  **Le Clustering (OPTICS) :** L'algorithme scanne la carte et repère les endroits où les photos se "regroupent". Si une rue possède 1 000 photos, il est logique de lui créer une cellule spécifique minuscule pour être très précis.
2.  **La Tessellation de Voronoï :** Imaginez que chaque groupe de photos est un "épicentre". La méthode de Voronoï trace des frontières de sorte que chaque point à l'intérieur d'une cellule soit rattaché au centre le plus proche.

<br>

---

<br>

## 2. L'analyse de l'image : L'architecture StreetCLIP

Après avoir défini le cadre géographique (les Geocells), le système doit être capable d'interpréter le contenu visuel d'une image pour le lier à une position. Cette étape cruciale repose sur un moteur de vision artificielle nommé StreetCLIP.

<br>

![Architecture du modèle StreetCLIP](/assets/img/worldplace/street_clip_architecture.png){: .shadow .rounded-10 }
_Figure 3 : Principe de l'apprentissage par contraste (Contrastive Pre-training) liant le texte à l'image._

<br>

L'architecture de base utilisée est CLIP (Contrastive Language-Image Pre-training). Son rôle est d'apprendre à identifier le contexte et faire des liens logiques : S'il voit un immeuble haussmannien, il le place mathématiquement près de l'étiquette "Paris". À l'inverse, il apprend à repousser très loin les éléments qui n'ont rien en commun.

<br>

### L'extraction de caractéristiques géographiques

Pour les besoins de la géolocalisation, le modèle CLIP standard a été affiné (on parle de fine-tuning) pour devenir StreetCLIP. Le modèle apprend ainsi à identifier des indices géographiques clés que l'œil humain pourrait négliger :
* **Données climatiques :** Distinction entre les végétations tropicales, tempérées ou arides.
* **Indices d'infrastructure :** Reconnaissance du côté de conduite, du style des panneaux de signalisation ou du marquage au sol.
* **Contexte temporel :** En corrélant la date de capture de l'image avec l'aspect visuel (neige, feuilles mortes, ensoleillement), le modèle comprend que ces éléments sont des variables cycliques et non des caractéristiques permanentes du terrain.

Lorsqu'une image est soumise au projet World Place, elle passe par un encodeur visuel qui la traduit en une signature numérique appelée Embedding. Cette signature est ensuite comparée aux signatures des zones géographiques connues. Dans le cadre de ce travail sur Paris et sa couronne, StreetCLIP permet de distinguer des nuances architecturales subtiles.

<br>

---

<br>

## 3. L'apprentissage de la proximité géographique

Dans le domaine du Deep Learning, cette mesure est effectuée par une fonction de perte (loss function). Traditionnellement, si le modèle sélectionne une cellule voisine de la cible, il est pénalisé de la même manière que s'il avait choisi un continent différent. Le modèle ne comprend pas la notion de "proximité".

<br>

### Le "Lissage" Haversine

Pour pallier cette lacune, le modèle s'appuie sur la formule de Haversine. Cette équation mathématique permet de calculer la distance la plus courte entre deux points sur une sphère, on appelle cela une géodésique c'est-à-dire la distance réelle "à vol d'oiseau" sur la courbure terrestre.

<br>

![Représentation de la distance géodésique de Haversine sur une sphère](/assets/img/worldplace/haversine_distance.png){: .shadow .rounded-10 }
_Figure 4 : Calcul de la distance géodésique entre deux points P et Q._

<br>

L'innovation de PIGEON réside dans l'utilisation du Lissage Haversine (Haversine Smoothing). 
* Si le modèle pointe une cellule limitrophe, le système considère que l'erreur est mineure.
* On lui attribue une "note partielle" positive. Plus la cellule prédite est éloignée de la réalité, plus la pénalité augmente de façon exponentielle.

Dans le contexte de Paris et de sa couronne, de nombreux quartiers présentent des similitudes architecturales fortes. Le lissage permet au modèle de comprendre que se tromper entre deux arrondissements voisins est une erreur non négligeable, ce qui affine la fiabilité globale des résultats.

<br>

---

<br>

## 4. Le Raffinement : Passer de la zone au point précis

Une fois que le modèle a réduit le champ de recherche à une cellule de Voronoï spécifique, il doit affiner son analyse pour passer d'une zone géographique de plusieurs kilomètres à une coordonnée exacte.

Chaque cellule de Voronoï est elle-même subdivisée en "groupes de référence". Il compare l'image inconnue aux milliers d'images de son dataset de référence situées dans la zone présélectionnée. 

Pour passer d'une simple estimation à une localisation au mètre près, il effectue un calcul mathématique de précision :
1.  **Le Sous-Cluster :** L'IA réduit son champ de recherche à un petit groupement d'images très denses géographiquement.
2.  **Les $k$ voisins les plus proches :** L'IA sélectionne un nombre défini de points (les "k voisins") qui ressemblent le plus à l'image analysée.
3.  **Triangulation :** Le réglage optimal permet à l'IA de "trianguler" sa position: elle déduit qu'elle se trouve, par exemple, à 60% de distance du voisin A et 40% du voisin B.

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

Sur ces images on peut voir les zones qui attirent l'attention du modèle pour retrouver la géolocalisation, on peut voir qu'elle observe la végétation et l'architecture, mais on peut également voir plein de points isolés ce sont le flou de la caméra, le modèle a appris de lui-même à utiliser le flou de la caméra comme indice, et d'autres stratégies similaires à un joueur humain en les perfectionnant.

<br>

![Cartes d'attention visuelle montrant les points d'intérêt de l'IA](/assets/img/worldplace/attention_maps.png){: .shadow .rounded-10 .w-100 }
_Figure 5 : Cartes d'attention (Heatmaps) révélant les zones analysées par le modèle, comme la végétation ou les flous de caméra._

<br>

---

<br>

## 6. Évaluation des performances

L'efficacité du framework a été validée par une étude d'ablation sur un jeu de données de 5 000 images Street View. Cette méthode consiste à désactiver certains composants du modèle pour mesurer leur impact réel sur la précision.

<br>

![Tableau des performances et étude d'ablation du modèle PIGEON](/assets/img/worldplace/performances_metrics.png){: .shadow .rounded-10 }
_Figure 6 : Étude d'ablation démontrant l'impact critique de chaque composant (comme le lissage Haversine) sur la précision géographique._

<br>

Comme le démontrent ces tableaux, le modèle complet ("PIGEON") affiche une précision redoutable :
* **40,36%** des prédictions tombent dans un rayon de 25 kilomètres (échelle d'une ville).
* **91,96%** de précision pour identifier le bon pays.
* Le retrait de composants clés comme le panorama à 4 images ou le lissage Haversine fait drastiquement chuter les performances (l'erreur médiane passe de 44 km à 148 km sans le lissage).

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

Merci d'avoir lu jusqu'au bout ! Si vous avez des questions ou que vous voulez discuter, contactez-moi sur Discord : **omss**.