# Comparative Study: Contrastive vs. Predictive Vision-Language Alignment

This repository contains a systematic comparative analysis of custom symmetric contrastive (**CLIP-style**) versus asymmetric predictive (**JEPA-style**) architectures for vision-language alignment. The project evaluates cross-modal retrieval performance on the Flickr8k dataset, specifically engineered to operate efficiently under hardware-constrained environments (a single T4 GPU).

## Architectural Approaches

**1. Symmetric Contrastive Baseline (Mini-CLIP)**
*   **Vision Backbone:** EfficientNet-V2-S
*   **Text Backbone:** DistilBERT
*   **Mechanism:** Standard InfoNCE symmetric contrastive loss. Maps both modalities into a shared 256D embedding space using trainable projection heads.

**2. Asymmetric Predictive Distillation (VL-JEPA)**
*   **Target Space (Frozen Blueprint):** DistilBERT (Raw 768D, mask-aware mean pooling). Acts as immutable semantic anchors without projection heads.
*   **Vision Backbone:** EfficientNet-V2-S (1280D).
*   **Predictor Network:** Deep Residual MLP mapping 1280D $\rightarrow$ 1024D $\rightarrow$ 768D.
*   **Loss Function:** Combines cross-modal cosine similarity with a KL-divergence spread penalty ($\alpha$) to successfully prevent representation collapse.

## Repository Structure

![Repository Files](image_60f863.png)

*   `CLIP_symmetric256.ipynb`: Baseline dual-encoder contrastive model mapping to 256D.
*   `CLIP_asymmetric.ipynb`: Locked-text contrastive tuning testing direct foundation mapping.
*   `JEPA_alpha0_5.ipynb`: Predictive model training with a lower spread penalty weight ($\alpha=0.5$).
*   `JEPA_alpha2_0.ipynb`: Predictive model training optimized with a high spread penalty weight ($\alpha=2.0$).

## Quantitative Results

Demonstrated that mapping 1280D vision features directly into a frozen 768D foundation language space using cosine distance and a KL-based spread loss successfully avoids representation collapse. 

This predictive **JEPA-style** approach outperformed the contrastive baseline by **65%**:
*   **Recall@1:** 17.0% (vs. Baseline 10.3%)
*   **Recall@5:** 38.0% (vs. Baseline 31.1%)
*   **Recall@10:** 50.3% (vs. Baseline 42.6%)

## Qualitative Retrieval Examples

### Image-to-Text Retrieval (Captioning)
The model successfully evaluates a query image against a pool of candidate captions and identifies the most semantically relevant text.

![Orca Retrieval](image_60f8fe.png)
*The model correctly isolates the semantic concept of the image, assigning the highest similarity score (0.3562) to the accurate caption regarding orca whales.*

![Child Retrieval](image_614b1f.png)
*Demonstrating model confidence, the system correctly identifies the true caption ("a wet child shivers...") with a 40.5% probability against a pool of hard random distractors.*

### Text-to-Image Retrieval
Querying the image gallery using natural language prompts to retrieve the closest visual matches.

![Mouse Search](image_614e9d.png)
*Results for the query "a mouse hiding in a tree". While challenged by the specific noun "mouse", the model successfully isolates and retrieves the semantic context of "hiding/climbing in a tree or woods" across the top 5 visual candidates.*
