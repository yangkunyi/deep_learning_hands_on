# UM-SJTU-JI Deep learning Hands-on Tutorial 
# Session 8 - Image Generation using Generative Adversarial Networks (GAN)

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Dataset and Preprocessing](#dataset-and-preprocessing)
- [Building the GAN](#building-the-gan)
- [Training the GAN](#training-the-gan)
- [Testing and Visualization](#testing-and-visualization)
- [Conclusion](#conclusion)

---

## Introduction

This session focuses on building a Generative Adversarial Network (GAN) to generate new images using the PyTorch framework, particularly focusing on the MNIST dataset.

---

## Prerequisites

Ensure you have installed:

- PyTorch
- Torchvision
- Numpy
- Matplotlib

```bash
pip install torch torchvision numpy matplotlib
```

---

## Dataset and Preprocessing

### Step 1: Load and Preprocess Data

```python
import torch
from torchvision import datasets, transforms

transform = transforms.Compose([transforms.ToTensor()])
train_dataset = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
train_loader = torch.utils.data.DataLoader(train_dataset, batch_size=64, shuffle=True)
```

---

## Building the GAN

### Step 2: Define the Generator and Discriminator

```python
import torch.nn as nn

class Generator(nn.Module):
    def __init__(self):
        super(Generator, self).__init__()
        self.fc = nn.Linear(100, 256)  # 100 is the size of the latent vector
        self.layer = nn.Sequential(
            nn.Linear(256, 512),
            nn.ReLU(),
            nn.Linear(512, 28*28),
            nn.Sigmoid()
        )

    def forward(self, x):
        x = self.fc(x)
        x = self.layer(x)
        return x.view(-1, 1, 28, 28)

class Discriminator(nn.Module):
    def __init__(self):
        super(Discriminator, self).__init__()
        self.layer = nn.Sequential(
            nn.Linear(28*28, 512),
            nn.ReLU(),
            nn.Linear(512, 256),
            nn.ReLU(),
            nn.Linear(256, 1),
            nn.Sigmoid()
        )

    def forward(self, x):
        x = x.view(-1, 28*28)
        return self.layer(x)
```

---

## Training the GAN

### Step 3: Train the Model

```python
import torch.optim as optim

generator = Generator()
discriminator = Discriminator()

# Optimizers
g_optimizer = optim.Adam(generator.parameters(), lr=0.0002)
d_optimizer = optim.Adam(discriminator.parameters(), lr=0.0002)

# Loss
criterion = nn.BCELoss()

num_epochs = 100

for epoch in range(num_epochs):
    for batch, (real_images, _) in enumerate(train_loader):
        batch_size = real_images.size(0)
        
        # Train Discriminator
        d_optimizer.zero_grad()
        real_labels = torch.ones(batch_size, 1)
        fake_labels = torch.zeros(batch_size, 1)

        outputs = discriminator(real_images)
        d_loss_real = criterion(outputs, real_labels)

        z = torch.randn(batch_size, 100)
        fake_images = generator(z)
        outputs = discriminator(fake_images.detach())
        d_loss_fake = criterion(outputs, fake_labels)

        d_loss = d_loss_real + d_loss_fake
        d_loss.backward()
        d_optimizer.step()

        # Train Generator
        g_optimizer.zero_grad()
        outputs = discriminator(fake_images)
        g_loss = criterion(outputs, real_labels)
        g_loss.backward()
        g_optimizer.step()

    print(f'Epoch [{epoch+1}/{num_epochs}], d_loss: {d_loss.item():.4f}, g_loss: {g_loss.item():.4f}')
```

---

## Testing and Visualization

### Step 4: Visualize Generated Images

```python
import matplotlib.pyplot as plt

# Generate images from the trained generator
z = torch.randn(6, 100)
generated_images = generator(z)

# Display Images
fig, axes = plt.subplots(figsize=(12, 3), ncols=6)
for i, ax in enumerate(axes):
    ax.imshow(generated_images[i].detach().numpy().squeeze(), cmap='gray')
    ax.axis('off')

plt.show()
```

This will display a row of images generated by your trained GAN after all epochs. Make sure to train enough epochs to get plausible images.

---

## Conclusion

In this session, you learned to implement a Generative Adversarial Network (GAN) using PyTorch, train it on the MNIST dataset, and visualize generated images. GANs can be applied in numerous interesting projects including image-to-image translation, super-resolution, and data augmentation. Feel free to explore more complexities and variations in the architecture to further enhance your generative models!
