---
layout: post
title: "WorldPlace: Finding Any Location in the World from an Image"
date: 2026-02-23 12:00:00 +0100
slug: worldplace-en
categories: [Artificial Intelligence, Computer Vision]
tags: [PIGEON, Geoguessr, Deep Learning, Mapillary, Google Street View, Geoint]
toc: true
image:
  path: /assets/img/worldplace/cover_eye.jpg
---

<br>

## Introduction

The game GeoGuessr has become very popular in recent years, bringing together a vast community around a simple challenge: given a Street View image taken anywhere on Earth, is it possible to find its precise location? At a planetary scale, the diversity of architectures, seasonal changes, weather, viewing angles, climate impacts, and many other factors make precise geolocation a complex and still largely unsolved problem.

While several artificial intelligence models have attempted to take on this challenge, they often present major flaws. The model I chose for my project, titled World Place, addresses some of these shortcomings. This document aims to explain the workings of the PIGEON model (Predicting Image Geolocation) in an accessible way, even for someone new to artificial intelligence (Deep Learning, Computer Vision).

For my project, I adapted and trained it on my own dataset. Due to the complexity of the subject and limited time, the project currently focuses on Paris and its inner suburbs. My dataset consists of 50,000 images from Mapillary, with 20 training cycles (epochs) for this first version.

The PIGEON model was developed by 3 Stanford University students and is trained on 360-degree panoramic images from Google Street View. It is the first model capable of reliably beating the world's best GeoGuessr players, ranking among the top 0.01% of players and having beaten expert Rainbolt in six consecutive games.

For further reading, you can consult the official PIGEON project report here: [Link to original PDF (arXiv)](https://arxiv.org/pdf/2307.05845) and explore the source code on their official GitHub repository: [https://github.com/LukasHaas/PIGEON](https://github.com/LukasHaas/PIGEON).

<br>

![Overall architecture of the PIGEON model](/assets/img/worldplace/pigeon_architecture.png){: .shadow .rounded-10 }
_Figure 1: Overview of the 4 key stages of the PIGEON model (Partitioning, Analysis, Error Calculation, Refinement)._

<br>

---

<br>

## 1. Map Partitioning: From Infinite Search to Targeted Selection

The main obstacle to automatic geolocation is the immensity of the search space. Asking a model to point to a precise location on a world map without guidance is like asking it to find a needle in a haystack the size of Earth: the possibilities are infinite, and the margin of error is immense. To fully appreciate the difficulty, searching for a precise position down to the meter on the planet is equivalent to trying to guess a complex password using brute force. Without partitioning, the model would have to choose among more than 500 quadrillion points worldwide.

To simplify this task, the model uses a discretization technique. Instead of letting the model navigate on a "smooth" and continuous map, we transform the terrain into cells. **To make an analogy, searching for an exact location on Earth is like looking for a specific book in an infinite library. Rather than examining each book one by one at random, this system allows us to first identify the right shelf (the *Geocell*). Once the shelf is found, locating the exact book becomes infinitely easier.**

* **Cell creation:** The map is divided into thousands of distinct zones, the Geocells (geographic cells).
* **Problem simplification:** Thanks to this partitioning, the model no longer searches for random coordinates. The model faces a "spatial multiple-choice test": its role is to classify the image into the most appropriate box. By transforming this problem into a choice among a few thousand cells, we reduce complexity by an immense factor, allowing the model to focus on recognizing coherent zones rather than conducting an infinite mathematical search.
* **Pruning unnecessary data:** This method allows for elimination. As soon as the model identifies the most probable cell, it can prune (remove) everything else from its consideration. This filtering mechanism allows the model to focus all its attention on the visual details specific to the selected zone, thus drastically increasing its precision. Partitioning into Geocells acts as a guide: it reduces the search space to just a few tens of thousands of choices, transforming an insurmountable mathematical challenge into a quick and efficient classification problem.

<br>

### 1.1 Evolution: From Rigid Squares to Custom Partitioning

Before PIGEON, researchers primarily used two methods to partition the map, but they had limitations:
* **Classic grid:** The Earth was divided into perfect squares. The problem is that these squares don't respect natural boundaries (a border, river, or mountain range might be cut in two).
* **Arbitrary partitioning:** Zones of equal photo counts were created, but without geographic logic.

PIGEON's innovation is using semantic partitioning: the puzzle follows real boundaries (cities, districts) because that's where street appearances change the most.

<br>

### 1.2 Clustering: Identifying Areas of Interest

In a city like Paris, photos accumulate massively around monuments. To manage this, the model uses a clustering algorithm called OPTICS (Ordering Points To Identify the Clustering Structure).
* **The concept:** The algorithm scans the map and identifies places where photos "cluster."
* **The utility:** This allows the model to understand where detail-rich zones are located. If a street has 1,000 photos, it makes sense to create a tiny specific cell for maximum precision.

<br>

### 1.3 Voronoi Tessellation: Drawing Intelligent Boundaries

Once points of interest are identified, the physical boundaries of each puzzle piece must be drawn. A mathematical method called Voronoi Tessellation is used.

<br>

![Comparison between rectangular cells and semantic cells in Paris](/assets/img/worldplace/semantic_geocells_paris.png){: .shadow .rounded-10 }
_Figure 2: On the left, classic partitioning; on the right, semantic partitioning using Voronoi tessellation._

<br>

* **How it works:** Imagine that each group of photos is an "epicenter." The Voronoi method draws boundaries so that each point inside a cell is attached to the nearest center.
* **The result:** We get a puzzle with irregular shapes. This system is much more efficient than a grid of squares because it adapts to the actual geometry of the city.

<br>

### 1.4 The Major Contribution of This Model

The main improvement over previous work is flexibility. By combining real boundaries (cities, countries) and mathematical partitioning (Voronoi), PIGEON succeeds in creating a puzzle that:
* **Balances data:** No puzzle piece is "empty" or "overfilled," which helps the model learn faster.
* **Respects meaning:** Cells don't cross important boundaries, which prevents the model from confusing two radically different countries or neighborhoods.

<br>

---

<br>

## 2. Image Analysis: The StreetCLIP Architecture

After defining the geographic framework (the Geocells), the system must be able to interpret the visual content of an image to link it to a position. This crucial step relies on a computer vision engine named StreetCLIP.

<br>

![Architecture of the StreetCLIP model](/assets/img/worldplace/street_clip_architecture.png){: .shadow .rounded-10 }
_Figure 3: Principle of contrastive learning (Contrastive Pre-training) linking text to image._

<br>

### 2.1 The CLIP Model Principle: Linking Image to Context

The base architecture used is CLIP (Contrastive Language-Image Pre-training). Unlike classic models that simply name objects (e.g., "this is a tree"), CLIP is designed to understand relationships between images and language.
* **Context analysis (Contrastive Learning):** The model was trained on millions of "image-text" pairs. Its role is to learn to identify context and make logical connections:
  * If it sees a Haussmann-style building, it places it mathematically close to the "Paris" label.
  * Conversely, it learns to push far away elements that have nothing in common (for example, a palm tree far from the "Norway" label).
* **Global understanding:** Thanks to this method, the model doesn't just memorize pixels; it develops a true visual culture of the world, capable of linking an architectural style to a geographic concept.

<br>

### 2.2 Geographic Specialization: StreetCLIP

For geolocation purposes, the standard CLIP model was fine-tuned to become StreetCLIP. The goal is to transform a generalist model into an expert in "landscape reading."

PIGEON was trained using synthetic captions enriched with auxiliary data. The model thus learns to identify key geographic clues that the human eye might overlook:
* **Climate data:** Distinction between tropical, temperate, or arid vegetation.
* **Infrastructure clues:** Recognition of driving side, signage style, or road markings.
* **Temporal context:** By correlating the image capture date (present in Street View metadata) with visual appearance (snow, fallen leaves, sunlight), the model understands that these elements are cyclical variables and not permanent terrain characteristics.

Seasonality is a classic trap for geolocation AIs:
* **Snow:** Without temporal attribute management, a model might systematically place a snowy image in Nordic countries or at high altitude.
* **Sunlight:** Strong zenithal sunlight could skew latitude calculations toward the equator.

<br>

### 2.3 Feature Extraction and "Embeddings"

When an image is submitted to the World Place project, it passes through a visual encoder that translates it into a numerical signature called an Embedding.
* **Vector comparison:** This signature is then compared to the signatures of known geographic zones.
* **Zone identification:** The model doesn't look for an exact match but calculates the "proximity" between the test image's signature and those of different Geocells. The closer the signatures, the higher the probability that the image belongs to that cell.
* **Application to World Place project:** In this work on Paris and its suburbs, StreetCLIP allows distinguishing subtle architectural nuances, such as the difference between a dressed stone building in the city center and the pavilion or industrial structures of the periphery, by translating these visual elements into geographic probabilities.

<br>

---

<br>

## 3. The Loss Function: Learning Geographic Proximity

Once the model has made a prediction (choosing a cell), it is necessary to measure the gap between this answer and the actual location. In Deep Learning, this measurement is performed by a loss function. This indicator guides the model's learning.

<br>

### 3.1 Limitations of Classic Approaches

Traditionally, classification models treat errors in a binary way: an answer is either correct or wrong.
* **The isolation problem:** In a standard system, if the model selects a neighboring cell of the target, it is penalized the same way as if it had chosen a different continent.
* **The consequence:** The model doesn't understand the notion of "proximity." It learns its representations in isolation, without grasping the geographic continuity of the world.

<br>

### 3.2 Integration of the Haversine Distance

To address this gap, the model relies on the Haversine formula. This mathematical equation calculates the shortest distance between two points on a sphere, called a geodesic—the actual "as the crow flies" distance on Earth's curvature.

<br>

![Representation of geodesic Haversine distance on a sphere](/assets/img/worldplace/haversine_distance.png){: .shadow .rounded-10 }
_Figure 4: Calculation of geodesic distance between two points P and Q._

<br>

* **Technical contribution:** By integrating this formula, the model "becomes aware" of geography. It no longer analyzes abstract labels but positions linked by concrete kilometer distances.

<br>

### 3.3 "Smoothing" (Haversine Smoothing): A Major Innovation

PIGEON's innovation lies in using Haversine Smoothing. Instead of only validating the exact cell (the "all or nothing" principle), the model distributes probability over surrounding cells based on their distance.
* **The smoothing principle:** If the model points to a neighboring cell, the system considers the error minor. It receives a positive "partial score." The further the predicted cell is from reality, the more the penalty increases exponentially.
* **Impact on learning:** This method encourages the model to be coherent. It teaches it that the world is a continuous space. This stabilizes predictions: in case of visual uncertainty, the model will favor a nearby zone rather than a completely absurd location on the other side of the planet.
* **Application to World Place project:** In the context of Paris and its suburbs, this system is decisive. Many neighborhoods have strong architectural similarities. Smoothing allows the model to understand that making a mistake between two neighboring districts is not a negligible error, which refines the overall reliability of results.

<br>

![Impact of Haversine smoothing on neighboring cells in Accra, Ghana](/assets/img/worldplace/haversine_ghana.jpg){: .shadow .rounded-10 .w-100 }
_Figure 5: Impact of applying Haversine smoothing to neighboring geographic cells for a location in Accra, Ghana (left without smoothing, right with smoothing)._

<br>

---

<br>

## 4. Refinement: From Zone to Precise Point

Once the model has narrowed the search field to a specific Voronoi cell—the stage we call classification—it must refine its analysis to go from a geographic zone of several kilometers to exact coordinates. To do this, the system abandons the "multiple choice" logic for a hybrid approach combining visual comparison and mathematical calculation.

<br>

### 4.1 The Concept of "Reference Groups" (Clustering)

To gain precision, each Voronoi cell is itself subdivided into "reference groups." If global classification finds the shelf (the cell), these groups identify the exact book. The model no longer just situates the image in a region; it links it to a cluster of photos whose numerical signatures (embeddings) are very close. This creates ultra-local reference points within a neighborhood or city.

<br>

### 4.2 Similarity Search

At this stage, the model performs what is called *retrieval* (information retrieval). It compares the unknown image to thousands of images from its reference dataset located in the preselected zone. By calculating the mathematical distance between the test image vectors and those of known images, the AI identifies the most probable neighbors. The more unique the architectural environment, the more effective this similarity search becomes.

<br>

### 4.3 Extracting Final Coordinates

How does PIGEON achieve surgical precision? To go from a simple estimate to meter-level localization, the model doesn't just guess a zone; it performs a precision mathematical calculation in two key steps:

**1. Moving from "sector" to "precise point"**
* Instead of simply classifying an image into a category (e.g., "Paris"), the model uses a weighted average method.
* **Why "generally"?** In most cases, the AI identifies several nearby reference points. It then calculates its position based on its relative distance from them. However, if it recognizes a unique and unambiguous element (a specific monument or precise street sign), it can point directly to that location without needing to average.

**2. The importance of "Neighbors" and "Sub-Cluster"**
Final precision relies on a hierarchical structure:
* **The Sub-Cluster:** Before calculating coordinates, the AI narrows its search field to a "sub-cluster" (a small grouping of geographically very dense images). The more compact and image-rich this group is, the more comparison points the AI has.
* **The "k" nearest neighbors:** Once in this group, the AI selects a defined number of points (the "k neighbors") that most resemble the analyzed image.
  * If it takes too many neighbors, precision dilutes.
  * If it takes too few, it risks copying a geolocation error from a single image.
* Optimal tuning allows the AI to "triangulate" its position: it deduces that it is, for example, 60% distance from neighbor A and 40% from neighbor B.
* **Application to World Place project:** In my work on Paris, this step is crucial. It allows the model to not just say "It's in Boulogne-Billancourt," but to point precisely to a specific intersection or street by recognizing the visual patterns (facades, street furniture) it has already encountered in its dataset of 50,000 images.

<br>

---

<br>

## 5. The "Meta": Hidden Clues in Google Street View

The model has developed deduction logic similar to that of expert GeoGuessr players. It is capable of detecting more subtle clues, often related to the legal or technical context of the capture. Here are some concrete examples identified by the AI:

<br>

### Legislative Constraints

In some countries, legislation requires Google to modify how it captures or distributes images.

<br>

#### Germany ("Blurmany")
The model identifies the very high blurring rate of buildings, resulting from a law allowing citizens to object to the publication of their homes.

<br>

![Blurred buildings in Germany](/assets/img/worldplace/allemagne.jpg){: .shadow .rounded-10 .w-100 }

<br>

#### Switzerland (and Japan / Austria)
The law prohibiting filming of private spaces not visible from the street means Google often uses a "Low-Cam" (camera placed lower). The model detects this perspective change.

<br>

![Low camera (Low-Cam) in Switzerland](/assets/img/worldplace/suisse.png){: .shadow .rounded-10 .w-100 }

<br>

### Security Protocols

#### Nigeria
It is the only country where police or security vehicle escort is systematic and visible in the camera's field of view.

<br>

![Police car escorting the Google Car in Nigeria](/assets/img/worldplace/nigeria.jpg){: .shadow .rounded-10 .w-100 }

<br>

### Vehicle Equipment (Car Meta)

The model learns to identify the carrier vehicle, certain elements of which are not completely erased by the algorithm.

<br>

#### Mongolia
The model easily recognizes the specific roof bars. Often, they carry bulky camping equipment, usually covered with a characteristic protective tarp, adapted to the country's long off-road trails.

<br>

![Equipment on the car roof in Mongolia](/assets/img/worldplace/mongolie.jpg){: .shadow .rounded-10 .w-100 }

<br>

### AI Visual Attention

In these images, we can see the zones that attract the model's attention to find the geolocation. We can see that it observes vegetation and architecture, but we can also see many isolated points: these are camera blur. The model has learned on its own to use camera blur as a clue, and other strategies similar to a human player while perfecting them.

<br>

![Visual attention maps showing AI points of interest](/assets/img/worldplace/attention_maps.png){: .shadow .rounded-10 .w-100 }
_Figure 6: Attention maps (Heatmaps) revealing the zones analyzed by the model, such as vegetation or camera blur._

<br>

---

<br>

## 6. Performance Evaluation and Comparative Analysis

The framework's effectiveness is not measured solely by a raw score, but by its ability to surpass current standards of artificial intelligence and human expertise. To validate this advancement, the researchers developed two complementary models: PIGEON (Street View specialist) and PIGEOTTO (generalist).

<br>

### PIGEOTTO: Generalization at Planetary Scale

While PIGEON is a specialist of the urban environment (Street View), PIGEOTTO was designed to be capable of locating any image from the globe. Its performance marks a turning point in image geolocation research:

* **Training on varied data:** Unlike PIGEON, PIGEOTTO is trained on a massive dataset of 4 million images from Flickr and Wikipedia. This allows it to understand varied visual contexts (natural landscapes, monuments, vacation photos) that are not necessarily found on Street View.
* **SOTA Benchmark Domination:** The model was tested on reference datasets like Im2GPS and Im2GPS3k. The results are conclusive:
  * **Country Level:** It surpasses the previous SOTA with a spectacular jump of 38.8 percentage points.
  * **City Level:** It improves precision by 7.7 points, a significant progression in a field where gains are usually measured in fractions of points.
* **The "Generalization" breakthrough:** PIGEOTTO's major innovation is its ability to generalize to unvisited places. Previously, models tended to "memorize" places present in their training base. PIGEOTTO is the first model capable of analyzing universal visual characteristics to deduce a position on geographic sites it has never encountered during its training.
* **A robust architecture:** By using the same vision base (CLIP) but adapted to non-panoramic images, PIGEOTTO proves that the hierarchical framework (country -> region -> cluster) is the most effective method for geolocation at planetary scale.

<br>

### Performance Statistics (PIGEON)

On the massive test set of 5,000 Street View images, PIGEON displays remarkable results:
**Average GeoGuessr Score:** The model achieves an average of 4,525 points out of a maximum of 5,000. This score reflects exceptional consistency across very diverse environments.

| Distance Radius | Success Rate |
| :--- | :--- |
| **Street** (1 km) | 5.36% |
| **City** (25 km) | 40.36% |
| **Region** (200 km) | 78.28% |
| **Country** (750 km) | 94.52% |
| **Continent** (2500 km) | 98.56% |

<br>

### Ablation Analysis: The Pillars of Performance

The ablation study (progressive removal of components) confirms that the model's superiority rests on precise architectural choices:

<br>

![Table of performances and ablation study of the PIGEON model](/assets/img/worldplace/performances_metrics.png){: .shadow .rounded-10 }
_Figure 7: Ablation study demonstrating the critical impact of each component (such as Haversine smoothing) on geographic precision._

<br>

* **The Panorama (360° View):** Using 4 images instead of just one reduces the median error from 131.1 km to 44.35 km. Peripheral vision is therefore crucial for precision.
* **Neighborhood Refinement (k nearest neighbors):** Without the final calculation based on nearest neighbors, kilometer-level precision drops from 5.36% to 1.32%.
* **The StreetCLIP Foundation:** Specialized pre-training is the engine of success; it allows the model to identify the correct country in 91.96% of cases.

<br>

### The Human Benchmark: Expert Validation

The duel against Trevor Rainbolt, world reference in GeoGuessr, served as final validation. By winning 6 matches out of 6, PIGEON proved that its combination of semantic classification and mathematical interpolation surpasses even the sharpest human intuition.

<br>

---

<br>

## Conclusion: From Local Limits to Global Security Stakes

The World Place project has highlighted the complexity of artificial intelligence-based geolocation systems. While the original PIGEON model displays impressive performance at a global scale, adapting this technology to a local scale (Paris and its inner suburbs) revealed several major challenges that explain a much more limited precision than hoped in practice.

On one hand, the architectural homogeneity of the capital (Haussmann buildings, standardized street furniture) deprives the model of the strong contrasts it needs to orient itself, making the classification task extremely difficult. On the other hand, the noise in data from Mapillary contributors (cameras of varying quality, obstructed angles) degrades visual feature extraction. Finally, training a CLIP architecture requires massive computational resources and long timeframes that limit the adjustment capabilities of a student project.

Despite these practical obstacles at my scale, this project constitutes a solid proof of concept of this architecture's effectiveness. Above all, it demonstrates one crucial thing: if such a system lays the groundwork for precision geolocation for a student, its potential in the hands of state actors is terrifying. This contrast brings us to a first-order ethical issue.

It is important to remember that almost all research on image geolocation between 2018 and 2023 was funded by the military. It is both fascinating and frightening to observe how one can go from a tool developed to excel at a simple game (GeoGuessr) to a potential weapon of mass destruction.

This tool is indeed so powerful that the Stanford researchers made the radical decision not to publish the complete parameters (weights) of their model, nor to make their dataset freely accessible. Modern ballistic systems today rely heavily on satellite networks to geolocate themselves and calculate their trajectory. Faced with signal jammers, these missiles lose their position and end up crashing. By integrating a computer vision model as powerful as PIGEON, a missile could use its own video feed to analyze the terrain it's flying over and recalibrate its trajectory in complete autonomy, thus becoming virtually impervious to electronic jamming.

By consciously choosing to keep their model closed, the research team has very probably delayed the development of these military guidance technologies by about ten years. Through this single ethical decision not to publish their work in full, they have potentially saved thousands of lives.

<br>

---

**The video that inspired me for this project:** [Finding any location in 0.1 second](https://www.youtube.com/watch?v=t-6ILUMrxWg) (by _What a Fail !_).

Thank you for reading to the end! If you have questions or want to discuss, feel free to contact me on Discord: **omss**
