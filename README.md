# Fall 2025 Take-Home Project  
**Author:** Jiwon Park  
**UID:** jp65979  

---

## Project Overview
The goal of this project is to build a classifier that identifies **forest vs. non-forest regions** in aerial imagery. The approach combines **baseline methods** (thresholding, texture/edge filtering, clustering) with a **supervised machine learning classifier**, and then compares their effectiveness.  

This README is divided into three parts:  
1. Initial Approach: Observations and Hypothesis  
2. Analysis of Baseline Forest Classification  
3. Analysis of Machine Learning Classifier  

---

## Part 1: Initial Approach and Observations

### Defining the Problem
A **forest** was defined as an area with a **high density of trees**, relatively free from human-built structures. However, identifying trees in aerial images is non-trivial:
- Trees may appear **green, brown, or red** depending on the season.  
- In winter, forests may appear as **bare branches**.  
- Grass, farmland, and even human structures (e.g., green tennis courts) can be similar in color.  

This motivated the need for **texture- and structure-based approaches**, not just color.

---

### Observations from Data Sources
#### USGS Earth Explorer (color images):
- Forests appear as **dense clusters of circles** (individual tree canopies).  
- Shadows between trees emphasize texture.  
- Forests have **irregular shapes** compared to the geometric patterns of farms.  
- Human structures are typically **white/gray** and regular in shape.  

#### UGA Aerial Photography Collection (grayscale, lower resolution):
- Trees still show **subtle texture differences** even in grayscale.  
- **Water bodies and forests** may look similar, but forests show more texture.  
- Grass fields and farmlands are **smooth, bright, and patterned** (lines, streaks).  

---

### Initial Hypothesis
From these observations, I hypothesized that **texture-based filtering** would be the most effective method.  
- Forests = rough, textured regions with irregular edges.  
- Non-forests (grass, farmland, water, human structures) = smoother, more uniform regions.  

Planned preprocessing for each method:  
- **Thresholding**: Grayscale + Histogram Equalization.  
- **Texture-based filtering**: Grayscale + Sharpening (initially), later testing blurring.  
- **Clustering**: Grayscale (for consistency) + Histogram Equalization + Blurring to reduce noise.  

---

### Data Collection & Labeling
Initially, I considered cropping images manually. Instead, I implemented an **interactive labeling tool**:
- Users can draw bounding boxes on an image and assign them as **forest** or **non-forest**.  
- Each labeled region is saved into `forests/` or `non-forests/` directories.  
- This created a reusable dataset for training and validation.  

---

## Part 2: Analysis of Baseline Methods

### Thresholding
- Preprocessing: Grayscale + Histogram Equalization.  
- Worked reasonably, but **forests and water bodies often both appeared white**, making them hard to distinguish.  
- Adding blurring reduced helpful texture differences, so the best result used **no blur**.  

### Texture-Based Filtering
- Preprocessing: Grayscale + Blurring + Histogram Equalization.  
- Tested sharpening vs. blurring:  
  - **Sharpening** created too much noise.  
  - **Blurring** reduced noise while keeping canopy texture visible.  
- This method best highlighted **forests vs. water/grass fields**, though small clustered buildings sometimes mimicked forest texture.  

### Clustering (K-Means)
- Preprocessing: Grayscale + Blurring + Histogram Equalization.  
- Tested `k=2,3,4`:  
  - **k=2**: could not distinguish forests from fields/water.  
  - **k=3**: differentiated forests vs. grass fields, but still confused forests vs. water.  
  - **k=4**: added extra clusters but over-segmented forests.  
- **k=3 performed best** but still weaker than texture filtering.  

---

### Baseline Method Conclusion
Among the three baselines, **texture-based filtering with blurring** produced the most reliable forest vs. non-forest separation. Thresholding and clustering were useful benchmarks but less robust in distinguishing forests from water or dense urban areas.  

---

## Part 3: Machine Learning Classifier

### Data Preparation
- Using the interactive labeling tool, I created a dataset of **forest vs. non-forest patches**.  
- Each patch was saved into its respective folder for transparency and verification.  
- To reduce bias, I ensured non-forest samples covered diverse cases: **roads, buildings, water, fields**.  

### Training
- Model: **Random Forest Classifier** (scikit-learn).  
- Input features: **RGB pixel values** (not grayscale, since color provides strong vegetation cues).  
- Training/Testing split: 80/20.  

### Results
- **Initial model accuracy:** ~0.92.  
- After rebalancing and isolating object categories in non-forest samples, accuracy improved to **0.96**.  
- Precision/Recall:  
  - Non-forest classification was very strong (>0.95).  
  - Forest recall was slightly lower, reflecting challenges with seasonal variation and small urban-tree patches.  

---

## Final Takeaways
- **Baselines**: Simple methods (thresholding, clustering) struggled with water/grass vs. forest confusion. Texture filtering worked best but misclassified dense urban areas.  
- **Machine Learning**: With labeled data, the Random Forest model achieved strong accuracy (0.96) and was more robust than baselines.  
- **Key insight**: RGB features and diverse labeled samples were critical for distinguishing forests from visually similar non-forests.  

---
