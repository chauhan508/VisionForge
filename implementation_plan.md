# Infrared Image Colorization and Enhancement Framework

This plan outlines the architecture and steps to build an end-to-end deep learning framework that converts low-resolution, monochrome Infrared (IR) satellite imagery into high-resolution, colorized RGB images, preserving semantic integrity (e.g., mapping thermal signatures of water to blue).

## User Review Required

> [!IMPORTANT]  
> **Dataset Size & Scope:** Landsat 8/9 satellite data is extremely large (often gigabytes per scene). We need to determine if you want to start with a small downloaded sample to build and test the pipeline locally, or if you already have the dataset ready in the `ISRO` folder.
> 
> **Compute Environment:** Deep learning models for image translation require significant compute. If your local machine doesn't have a dedicated NVIDIA GPU, we should structure this code so you can easily upload it to Google Colab, Kaggle, or an AWS GPU instance for actual training.

## Open Questions

> [!WARNING]  
> 1. **Data Availability:** Do you already have paired IR and RGB Landsat 8/9 datasets prepared, or would you like me to write a script to download and pre-process a sample dataset (e.g., using `rasterio` and EarthExplorer APIs)?
> 2. **Architecture Approach:** We can use a two-stage approach (Model A for Super-Resolution, Model B for Colorization) or an End-to-End single network (a combined SR+Colorization Generator, which usually has faster inference times). Do you have a preference?
> 3. **Semantic Mask:** Do you have existing land-cover masks to enforce semantic constraints (e.g., knowing where water/vegetation is), or should we use an unsupervised approach?

## Proposed Changes

We will build a Python-based deep learning project using PyTorch. The directory structure will separate data handling, model definitions, training, and evaluation.

---

### Core Environment Setup

Defines the dependencies and instructions for running the project.

#### [NEW] [requirements.txt](file:///c:/Users/Dushyant/Documents/ISRO/requirements.txt)
Will include `torch`, `torchvision`, `opencv-python`, `rasterio`, `scikit-image`, `pytorch-fid`, and `albumentations` for augmentations.

#### [NEW] [README.md](file:///c:/Users/Dushyant/Documents/ISRO/README.md)
Instructions on how to install dependencies, prepare data, and run training/inference.

---

### Data Processing Module

Scripts to handle geospatial TIFF imagery, coregister IR and RGB pairs, and create manageable patches for training.

#### [NEW] [src/data/preprocess.py](file:///c:/Users/Dushyant/Documents/ISRO/src/data/preprocess.py)
Functions to read raw Landsat 8/9 `.tif` files, coregister them (ensure pixel-perfect alignment between IR and RGB), and crop them into smaller tiles (e.g., 256x256).

#### [NEW] [src/data/dataset.py](file:///c:/Users/Dushyant/Documents/ISRO/src/data/dataset.py)
A custom PyTorch `Dataset` class to load the IR-RGB pairs, apply necessary normalizations, and perform data augmentation.

---

### Models (Deep Learning Architecture)

The core neural networks. We will utilize a GAN (Generative Adversarial Network) approach similar to Pix2Pix or CycleGAN, modified to incorporate super-resolution capabilities.

#### [NEW] [src/models/generator.py](file:///c:/Users/Dushyant/Documents/ISRO/src/models/generator.py)
The Generator network. It will take a low-resolution IR image, upscale it (using pixel shuffle or transposed convolutions), and output a high-resolution RGB image.

#### [NEW] [src/models/discriminator.py](file:///c:/Users/Dushyant/Documents/ISRO/src/models/discriminator.py)
A PatchGAN Discriminator that evaluates whether the generated high-resolution RGB images are real or fake, pushing the generator to produce realistic textures and colors.

#### [NEW] [src/models/losses.py](file:///c:/Users/Dushyant/Documents/ISRO/src/models/losses.py)
Contains the GAN loss, L1/L2 reconstruction loss, and the **Semantic Constraint Loss**. The semantic loss will penalize the model if the colorization violates natural rules (e.g., coloring a thermal water signature as green instead of blue).

---

### Training & Evaluation Pipelines

Scripts to execute the pipeline, track metrics, and run inference.

#### [NEW] [train.py](file:///c:/Users/Dushyant/Documents/ISRO/train.py)
The main training loop. Handles optimizers, learning rate scheduling, model checkpointing, and logs training progress.

#### [NEW] [evaluate.py](file:///c:/Users/Dushyant/Documents/ISRO/evaluate.py)
Calculates the required image quality metrics on the validation set:
- **PSNR & SSIM** using `scikit-image`
- **FID** using `pytorch-fid`

#### [NEW] [inference.py](file:///c:/Users/Dushyant/Documents/ISRO/inference.py)
Script to process a new, unseen satellite IR tile and output the final colorized RGB image. Includes logic to track and print the inference time per tile to ensure scalability.

## Verification Plan

### Automated Tests
- Run a forward pass on `Generator` and `Discriminator` with dummy PyTorch tensors to ensure architectural constraints and shape sizes match.
- Test the loss functions (`losses.py`) and metrics (`evaluate.py`) with artificial data arrays to verify mathematical correctness.

### Manual Verification
- Execute `preprocess.py` on a sample remote sensing image and visually inspect the output patches to ensure proper alignment and cropping.
- Run `train.py` for 2-3 epochs to confirm the loss converges and no out-of-memory (OOM) errors occur.
- Run `inference.py` on a test image and visually inspect the colorized output for artifacts or "hallucinations".
