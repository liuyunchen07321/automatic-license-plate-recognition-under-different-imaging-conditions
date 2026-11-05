# Automatic License Plate Recognition under Different Imaging Conditions

Automatic license plate recognition systems often achieve strong results under favorable imaging conditions, but their reliability can decline when images are blurred, poorly illuminated, captured at an angle, or contain low-resolution license plates.

This project combines a convolutional neural network (CNN) with mathematical modeling to investigate a practical question:

&gt; Which imaging conditions have the greatest influence on recognition performance, and how should limited resources be allocated to improve them?

The project has two complementary components:

1. **Recognition model:** A seven-position CNN classifier generates binary recognition outcomes on license plate images.
2. **Probability model:** A five-factor conditional probability function is fitted to the experimental outcomes and analyzed using sensitivity analysis and constrained optimization.

## Key Results

| Experiment | Result |
|---|---:|
| Validation character-level accuracy | **97.90%** |
| Validation exact-plate accuracy | **88.82%** |
| Exact-plate accuracy on 3,000 held-out experimental images | **88.67%** |
| Correctly recognized experimental images | **2,660 / 3,000** |
| Training images | **30,000** |
| Validation images | **5,000** |

## Research Workflow

```text
CCPD2019 dataset
      |
      v
Multivariate proportionate stratified sampling
      |
      v
Seven-position CNN license plate classifier
      |
      v
Binary recognition outcomes on 3,000 unseen images
      |
      v
Bernoulli maximum likelihood estimation
      |
      v
Five-factor recognition probability model
      |
        Partial derivatives and Hessian matrix
      |
        Lagrange-multiplier constrained optimization
```

## Dataset and Sampling

The project uses the **Chinese City Parking Dataset (CCPD2019)**, specifically the **CCPD_Base** subset, which contains **95,774 license plate images** captured under diverse real-world conditions.

CCPD filenames encode useful annotations, including:

- License plate area and bounding-box information
- Horizontal and vertical camera tilt
- Brightness
- Blurriness
- Ground-truth license plate number

To create a representative experimental dataset while retaining variation in imaging conditions, each of five explanatory variables was divided into low, medium, and high categories using approximately the 33rd and 67th percentiles. This produced up to **3⁵ = 243 strata**.

A sample of **3,000 images** was selected using multivariate proportionate stratified sampling. These images were reserved for the final experiment and were not used to train the recognition model.

## CNN Recognition Model

Before recognition, each license plate was cropped using the bounding-box annotation supplied by CCPD. No geometric correction was applied, ensuring that the effects of camera angle and the other imaging conditions remained present in the experimental images.

Preprocessing included:

- Conversion to grayscale
- Resizing to **32 × 160 pixels**
- Pixel-intensity normalization

The CNN directly predicts all seven character positions of a Chinese license plate:

1. Province character
2. Alphabetic character
3–7. Alphanumeric characters

A separate cross-entropy loss is calculated for each position, and the seven losses are summed to obtain the total training loss.

A preliminary model trained on 10,000 images for five epochs achieved approximately **92.99% character-level accuracy** and **62.25% exact-plate accuracy**. Further training on 30,000 images with 5,000 validation images and a reduced learning rate produced the final validation results reported above.

## Five-Factor Probability Model

The recognition probability is modeled as a function of five variables:

- **R:** Effective license plate resolution
- **B:** Motion blur
- **L:** Illumination
- **θₕ:** Horizontal camera tilt
- **θᵥ:** Vertical camera tilt

The component functions capture different expected behaviors:

| Factor | Functional form | Interpretation |
|---|---|---|
| Resolution | Logistic function | Recognition improves with resolution, with diminishing returns |
| Motion blur | Exponential decay | Recognition decreases as blur increases |
| Illumination | Gaussian-shaped function | Both insufficient and excessive illumination reduce performance |
| Camera angles | Squared-angle exponential decay | Larger horizontal or vertical deviations reduce recognition probability |

Assuming conditional independence among the factor contributions, the combined recognition probability is the product of the individual component functions.

## Parameter Estimation

Each experimental observation is represented as a Bernoulli outcome:

- **Y = 1:** The complete license plate is recognized correctly.
- **Y = 0:** The license plate is recognized incorrectly.

The model parameters are estimated by numerically maximizing the log-likelihood of the 3,000 experimental observations, subject to physically meaningful parameter constraints.

The fitted model produced the following estimates:

| Parameter | Estimate |
|---|---:|
| a | 0.01496 |
| R₀ | 2.00 |
| k₁ | 1.73 × 10⁻⁴ |
| L₀ | 28.00 |
| σ | 399.62 |
| kₕ | 4.62 × 10⁻⁴ |
| kᵥ | 1.86 × 10⁻⁴ |

## Sensitivity Analysis and Optimization

### Partial derivatives and Hessian matrix

First-order partial derivatives measure how sensitive recognition probability is to changes in each imaging condition. Second-order derivatives and the Hessian matrix describe local curvature and interactions between variables.

The analysis shows that:

- Recognition probability increases monotonically with effective resolution within the model.
- Recognition probability decreases monotonically with motion blur.
- The fitted optimum illumination occurs at **L = 28**.
- Zero horizontal and vertical tilt correspond to local maxima of the angle components.
- The fitted horizontal-tilt coefficient is approximately **2.5 times** the vertical-tilt coefficient, indicating greater sensitivity to horizontal misalignment in this model.

Because resolution and blur attain their best values at the feasible boundaries, the unconstrained objective does not have an interior stationary optimum. This motivates the use of a constrained optimization formulation.

### Lagrange multipliers

A budget constraint is introduced to represent the cost of improving resolution, reducing blur, controlling illumination, and correcting camera angles. Lagrange multipliers are then used to characterize resource-efficient optima.

At an interior optimum, the marginal improvement in recognition probability per unit of cost must be equal across all funded improvements. A unique numerical optimal configuration is not calculated because actual engineering cost data are unavailable.

## Main Findings

- Model architecture alone does not determine ALPR reliability; input imaging conditions also play an important role.
- Within the fitted sample, illumination has a relatively weak effect compared with the other modeled factors.
- Most sampled plates have sufficient effective resolution for the CNN, resulting in a high resolution component for much of the experimental range.
- Blur has a more noticeable influence on predicted recognition probability.
- Horizontal camera misalignment has a stronger estimated effect than vertical misalignment.
- Mathematical optimization can provide a general framework for allocating imaging-system improvements, but practical numerical recommendations require real cost data.

## Limitations

- The component contributions are assumed to be conditionally independent in the combined probability model.
- The fitted relationships reflect the CCPD2019 sample and the specific CNN recognizer; they may not generalize directly to other datasets or recognition systems.
- Only simple cropping and preprocessing are applied, without geometric plate correction.
- Some CCPD annotation values act as proxies for the physical imaging variables in the mathematical model.
- Engineering costs are not available, so the constrained optimization remains a general condition rather than a numerical deployment recommendation.


## Dataset Reference

*Towards End-to-End License Plate Detection and Recognition: A Large Dataset and Baseline* — Chinese City Parking Dataset (CCPD2019).

Dataset repository: [detectRecog/CCPD](https://github.com/detectRecog/CCPD)

