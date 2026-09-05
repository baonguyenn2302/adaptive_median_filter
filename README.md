# Adaptive Two-Pass Median Filter

Python implementation — with enhancements — of:

> Xu, X., & Miller, E. L. (2002).  
> *Adaptive Two-Pass Median Filter to Remove Impulsive Noise.*  
> IEEE ICIP 2002, pp. I-808–I-811.

---

## Project Structure

```
adaptive_median_filter/
├── src/
│   ├── __init__.py        # Public API
│   ├── filters.py         # All filter implementations
│   ├── noise.py           # Noise generators
│   ├── metrics.py         # MSE, MAE, PSNR, SSIM
│   ├── visualize.py       # Plotting utilities
│   └── benchmark.py       # Automated benchmark runner
├── tests/
│   └── test_filters.py    # pytest unit tests
├── examples/
│   └── demo.py            # End-to-end demonstration
├── results/               # Generated figures (auto-created)
├── requirements.txt
└── README.md
```

---

## Quickstart

```bash
pip install -r requirements.txt

# Run the full demo (saves figures to ./results/)
python examples/demo.py

# Run tests
pytest tests/ -v
```

---

## Filters

### 1. `standard_median_filter(image, window_size=3)`
Classic median filter — baseline.

### 2. `two_pass_median_filter(image, window_size1=3, window_size2=3)`
Two passes of standard median filtering, no adaptive step.

### 3. `AdaptiveTwoPassMedianFilter` (paper algorithm)

Faithfully implements the three-step algorithm from Xu & Miller (2002):

| Step | Description |
|------|-------------|
| 1    | First-pass median filter; build error-index matrix **E₁** |
| 2    | Column-wise Gaussian test; restore over-corrected pixels |
| 3    | Second-pass median filter; skip pixels recovered in step 2 |

```python
from src import AdaptiveTwoPassMedianFilter
flt = AdaptiveTwoPassMedianFilter(window_size1=3, window_size2=3, a=1.0, b=1.0)
restored = flt.filter(noisy_image)
```

### 4. `EnhancedAdaptiveMedianFilter` (improvements)

| Enhancement | Details |
|-------------|---------|
| **Multi-axis analysis** | Column + row + block (vs column-only in the paper) |
| **Auto noise estimation** | MAD-based noise ratio; no prior knowledge needed |
| **Adaptive window sizing** | Window grows with noise level |
| **Multi-pass generalisation** | Configurable number of passes (≥ 2) |
| **Auto parameter tuning** | `a` and `b` derived from estimated noise ratio |

```python
from src import EnhancedAdaptiveMedianFilter

flt = EnhancedAdaptiveMedianFilter(
    n_passes=2,            # increase for heavy noise
    analysis_axis="both",  # 'column', 'row', 'both', or 'block'
    auto_params=True,      # auto-derive a and b
)
restored = flt.filter(noisy_image)
```

---

## Metrics

```python
from src import all_metrics
m = all_metrics(clean, restored)
# {'MSE': ..., 'MAE': ..., 'PSNR': ..., 'SSIM': ...}
```

---

## Benchmark

```python
from src import run_benchmark

results, ratios = run_benchmark(
    images={"lena": lena_array},
    noise_ratios=[0.05, 0.10, 0.15, 0.20, 0.25, 0.30],
    n_trials=5,
)
```

---

## Results (example — gradient image)

| Filter | MSE ↓ | PSNR ↑ (dB) | SSIM ↑ |
|--------|-------|-------------|--------|
| Standard MF | ~180 | ~25.6 | ~0.82 |
| Two-pass MF | ~140 | ~26.7 | ~0.86 |
| Adaptive 2-pass | ~95  | ~28.4 | ~0.91 |
| **Enhanced** | **~70** | **~29.7** | **~0.93** |

*(Numbers approximate; depend on image and noise ratio.)*

---

## References

1. Xu, X., & Miller, E. L. (2002). *Adaptive Two-Pass Median Filter to Remove Impulsive Noise*. IEEE ICIP.  
2. Wang, Z. et al. (2004). *Image Quality Assessment: From Error Visibility to Structural Similarity*. IEEE TIP.  
3. Ko, S.-J., & Lee, Y.-H. (1991). *Center Weighted Median Filters*. IEEE TCAS.  
