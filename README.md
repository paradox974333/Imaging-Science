# Homomorphic Filtering for Image Enhancement

## Overview

I developed this code to perform illumination correction and enhance image contrast using homomorphic filtering in the frequency domain. The approach separates lighting from surface details, enabling independent control over each component.

## Method Used

**Homomorphic Filtering** based on the model: `I(x,y) = L(x,y) × R(x,y)`

- `I(x,y)` = image intensity
- `L(x,y)` = illumination (low-frequency)
- `R(x,y)` = reflectance (high-frequency)

**Process:**
1. Log transform: converts multiplication to addition
2. 2D FFT to frequency domain
3. Gaussian high-pass emphasis filter: `H(u,v) = (γ_high - γ_low) × H_high + γ_low`
4. Inverse FFT back to spatial domain
5. Exponential transform to recover enhanced image

**Color Variants:**
- **Independent**: Filters each RGB channel separately
- **Chromaticity-Preserving**: Maintains color ratios while processing intensity
- **Joint Spectral**: Combines illumination from all channels

## Images Used

- Multiple test images with uneven lighting and shadows
- Personal photos and sample images for validation

## Input Parameters

**Grayscale:** `process_grayscale_image(image_path, cutoff_freq=40, gamma_low=0.3, gamma_high=2.0)`
**Color:** `process_color_image(image_path, method='chromaticity', cutoff_freq=40, gamma_low=0.3, gamma_high=2.0)`

- **image_path**: Input image file
- **cutoff_freq**: Filter cutoff (default: 40)
- **gamma_low**: Illumination suppression weight (default: 0.3)
- **gamma_high**: Detail enhancement weight (default: 2.0)
- **method**: 'independent', 'chromaticity', or 'joint' (color only)

## Output Parameters

**Returns:**
- Grayscale: `reflectance_norm`, `illumination_norm`
- Color: `result` (enhanced RGB image)

**Saved Visualizations:**
- `grayscale_result.png`: Original, log-domain, filter, reflectance, illumination, histogram
- `color_result_{method}.png`: Original, corrected, illumination, RGB histograms

## Usage

```python
# Grayscale
reflectance, illumination = process_grayscale_image('photo.jpg')

# Color
result = process_color_image('photo.jpg', method='chromaticity')
```
