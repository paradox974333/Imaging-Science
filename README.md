# Homomorphic Filtering for Image Enhancement

A Python implementation of homomorphic filtering techniques for correcting uneven illumination and enhancing image contrast through frequency domain processing.

## What This Does

This project separates lighting effects from surface details in images, allowing independent manipulation of illumination and reflectance components. The code handles both grayscale and color images using different processing strategies.

## Technical Background

### Core Principle

Images are modeled as the product of illumination and reflectance:

```
I(x,y) = L(x,y) × R(x,y)
```

- **I(x,y)**: Observed image intensity
- **L(x,y)**: Illumination component (low-frequency, varies slowly)
- **R(x,y)**: Reflectance component (high-frequency, surface details)

### Processing Pipeline

1. **Log Transform**: Converts multiplication to addition: `ln(I) = ln(L) + ln(R)`
2. **Frequency Transform**: Apply 2D FFT to move to frequency domain
3. **Frequency Filtering**: Apply Gaussian high-pass emphasis filter
4. **Inverse Transform**: IFFT back to spatial domain
5. **Exponential Recovery**: `exp(filtered)` recovers the enhanced image

### Filter Design

The emphasis filter is constructed as:

```
H(u,v) = (γ_high - γ_low) × H_highpass + γ_low
```

- **H_highpass**: Gaussian high-pass filter (1 - exp(-D²/2σ²))
- **γ_low**: Weight for low frequencies (illumination suppression)
- **γ_high**: Weight for high frequencies (detail enhancement)

## Color Processing Methods

### Independent Channel Processing
Applies homomorphic filtering separately to each RGB channel. Simple but may alter color balance.

### Chromaticity-Preserving Method
Processes only the intensity while maintaining original color ratios. Preserves hue and saturation:
- Extracts intensity as mean of RGB channels
- Filters the intensity component
- Reconstructs RGB by scaling original color ratios

### Joint Spectral Method
Estimates illumination jointly across all channels using geometric mean, then adjusts each channel's reflectance with channel-specific normalization.

## Requirements

```python
numpy
Pillow (PIL)
matplotlib
```

## Usage Examples

### Grayscale Processing

```python
from homomorphic_filter import process_grayscale_image

reflectance, illumination = process_grayscale_image(
    'input_image.jpg',
    cutoff_freq=40,
    gamma_low=0.3,
    gamma_high=2.0
)
```

### Color Processing

```python
from homomorphic_filter import process_color_image

# Chromaticity-preserving (recommended)
result = process_color_image(
    'color_photo.jpg',
    method='chromaticity',
    cutoff_freq=40,
    gamma_low=0.3,
    gamma_high=2.0
)

# Alternative methods
result = process_color_image('photo.jpg', method='independent')
result = process_color_image('photo.jpg', method='joint')
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `image_path` | str | required | Path to input image file |
| `cutoff_freq` | float | 40 | Filter cutoff frequency (controls transition between low/high freq) |
| `gamma_low` | float | 0.3 | Illumination suppression factor (< 1 reduces low frequencies) |
| `gamma_high` | float | 2.0 | Detail enhancement factor (> 1 amplifies high frequencies) |
| `method` | str | 'chromaticity' | Color processing method: 'independent', 'chromaticity', or 'joint' |

### Parameter Tuning Guidelines

- **cutoff_freq**: Lower values (20-30) for strong illumination correction, higher values (50-70) for subtle adjustments
- **gamma_low**: 0.2-0.5 typical range; lower values suppress illumination more aggressively
- **gamma_high**: 1.5-2.5 typical range; higher values enhance details but may amplify noise

## Outputs

### Return Values

**Grayscale:**
- `reflectance_norm`: Enhanced image with corrected illumination (0-1 normalized)
- `illumination_norm`: Estimated illumination component (0-1 normalized)

**Color:**
- `result`: Enhanced RGB image with corrected illumination (0-1 normalized per channel)

### Saved Visualizations

**grayscale_result.png** contains:
- Original input image
- Log-transformed domain representation
- Emphasis filter visualization
- Recovered reflectance (enhanced output)
- Estimated illumination component
- Histogram comparison of intensity distributions

**color_result_{method}.png** contains:
- Original color image
- Corrected output image
- Estimated illumination map
- Per-channel histograms (R, G, B) showing before/after intensity distributions

## Implementation Details

### Frequency Domain Operations

The code implements manual FFT operations using NumPy's fft2/ifft2 with proper shifting for centered frequency representation. The Gaussian filter is constructed in the frequency domain using distance from center frequency.

### Numerical Stability

- Small epsilon values (1e-10) prevent log(0) and division by zero
- Images normalized to [0,1] range before processing
- Output clipping ensures valid pixel values

### Color Space Considerations

All color processing operates in RGB space. The chromaticity-preserving method maintains color information by separately processing intensity, which approximates perceptual brightness better than individual channel processing.

## Typical Use Cases

- Correcting shadows and uneven lighting in photographs
- Enhancing details in dark regions without overexposing bright areas
- Preprocessing for computer vision tasks requiring consistent illumination
- Document image enhancement
- Underwater image restoration
- Medical image enhancement

## Limitations

- May amplify noise in very dark regions
- Not suitable for images with extreme dynamic range without preprocessing
- Color methods assume multiplicative illumination model
- Processing time increases with image size due to FFT operations
