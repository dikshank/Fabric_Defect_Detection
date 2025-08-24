# Fabric_Defect_Detection

I built a small anomaly detection pipeline around ResNet and a feature autoencoder. Here's what the code is doing step by step:

## Feature Extractor (ResNet backbone)
I'm using a pretrained ResNet-18, but instead of the full model I only keep layers up to layer3. From there, I extract feature maps and apply global average pooling to get a compact 256-dimensional feature vector for each image. This way, I'm not training from raw pixels but from semantically meaningful features.

## Autoencoder on Features
I designed a simple fully connected autoencoder that learns to reconstruct those 256-dimensional feature vectors. The encoder reduces the dimensionality down to a latent space (128 dimensions), and then the decoder tries to recover the original features. If the reconstruction is good, it means the feature looked like something the model has seen during training (i.e. "normal" data).

## Dataset Loader
I wrote a small dataset class that just loads images from a single folder, applies basic transformations (resize, normalization, flips, color jitter), and returns them as tensors. This keeps it simple since my training set is just a folder of good (non-defect) images.

## Training Loop
During training, I take images, extract their ResNet features, pass those features through the autoencoder, and then compare the original vs reconstructed features using cosine similarity. The loss is defined as **1 - mean(cosine similarity)**, so the model learns to maximize similarity. I use Adam optimizer and train for a number of epochs, printing average loss per epoch. At the end, the autoencoder weights are saved.

## Anomaly Scoring
For evaluation, I run the same feature extraction on test images, reconstruct with the autoencoder, and compute the cosine similarity between original and reconstructed features. The anomaly score is simply **1 - cosine similarity**:

1. If the score is close to 0, the image is likely normal (reconstruction was good).

2. If the score is higher, the autoencoder struggled to reconstruct, meaning the image might contain an anomaly.

<img width="743" height="537" alt="{1B07E375-8AD5-4A14-8FFE-4FE3BA0C529B}" src="https://github.com/user-attachments/assets/fe701bce-cc19-49f0-b49e-001e1c97a424" />
