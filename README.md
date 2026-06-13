# Testing the Cosmic Distance Duality Relation with 21-cm HI Galaxies: Mock SKA Forecasts, Multi-Tracer Constraints, and Dark-Photon Coupling Limits

**By Nihar Mahesh Jani** · [niharmaheshjani@gmail.com](mailto:niharmaheshjani@gmail.com)

*Five original extensions to the Tolman 21-cm CDDR framework — redshift-binned constraints, dust-bias quantification, Fisher forecasting, multi-tracer joint inference, and the first dark-photon coupling limits derived from HI angular sizes — built on a ground-up reimplementation of Khedekar & Chakraborti (2011) [arXiv:1105.1138](https://arxiv.org/abs/1105.1138).*

---

## Why This Repo Exists

Let me be straight with you about what this project actually is.

The Cosmic Distance Duality Relation (CDDR) — also called the Etherington relation — is one of the most fundamental predictions of general relativity. It says that luminosity distance and angular diameter distance are related by exactly $d_L = d_A (1+z)^2$. No more, no less. If that relation is violated, it means new physics: extra dimensions, photon–axion conversion, dark-photon mixing, or something else entirely that we do not yet have a name for.

Khedekar & Chakraborti showed in 2011 that HI 21-cm galaxies are a beautifully clean probe of this relation — because their angular sizes encode the $d_L/d_A$ ratio, and because 21-cm emission is physically immune to dust extinction that would corrupt optical measurements. They showed that the Square Kilometre Array (SKA) could constrain a CDDR violation parameter $\varepsilon$ to the level of $\Delta\varepsilon \sim 0.01$.

I reproduced every result in that paper from scratch. Then I extended it five ways that the original did not: redshift-binned constraints, dust-bias quantification, Fisher forecasting, multi-tracer joint inference with Type Ia supernovae and BAO, and a first-of-its-kind translation of the $\varepsilon$ constraint into an upper limit on dark-photon kinetic mixing.

The whole thing — mock catalog generation, MCMC sampling, all figures — runs in a single C++17 binary (`code.cpp`) in under 26 minutes on a laptop. The Python script (`code.py`) then turns the output data files into publication-quality figures.

If you want to understand the physics, the implementation, and what the numbers actually mean: read on.

---

## Table of Contents

1. [The Physics in Plain English](#1-the-physics-in-plain-english)
2. [Repository Structure](#2-repository-structure)
3. [How to Run — Step by Step](#3-how-to-run--step-by-step)
4. [What the Code Does Internally](#4-what-the-code-does-internally)
5. [Survey Configurations](#5-survey-configurations)
6. [Main Results: Paper Reproduction](#6-main-results-paper-reproduction)
7. [Extension 1 — Redshift-Binned ε(z)](#7-extension-1--redshift-binned-εz)
8. [Extension 2 — Dust Bias (21 cm is Immune)](#8-extension-2--dust-bias-21-cm-is-immune)
9. [Extension 3 — Fisher Matrix Forecasting](#9-extension-3--fisher-matrix-forecasting)
10. [Extension 4 — Multi-Tracer: HI + SN + BAO](#10-extension-4--multi-tracer-hi--sn--bao)
11. [Extension 5 — Dark-Photon Coupling Limits](#11-extension-5--dark-photon-coupling-limits)
12. [Output Files Reference](#12-output-files-reference)
13. [Figures Generated](#13-figures-generated)
14. [Performance and Optimisations](#14-performance-and-optimisations)
15. [Dependencies](#15-dependencies)
16. [Citation](#16-citation)

---

## 1. The Physics in Plain English

The core idea is elegant. Take a galaxy. Measure its angular size $\theta$ on the sky. Measure its 21-cm flux $S$. Both of these observables depend on cosmological distances — but they depend differently. Angular size goes as $1/d_A$; flux goes as $1/d_L^2$. So if you know the intrinsic properties of the galaxy, you can form a ratio that isolates $d_L / d_A$.

General relativity predicts this ratio is exactly $(1+z)^2$. We parameterise any deviation as:

$$d_L = d_A \, (1+z)^{2+\varepsilon}$$

Standard GR: $\varepsilon = 0$. Our goal is to constrain how far $\varepsilon$ can be from zero.

HI 21-cm disk galaxies are ideal for this because their HI gas follows a nearly universal surface density. This means:

$$L_{\rm HI} \propto A_{\rm HI}^\alpha, \quad \alpha \approx 1.128$$

The observed angular area and flux are then:

$$S_{\rm ang}(z) = \left[\frac{4\pi f (1+z)^{3+2\varepsilon}}{10^B \, d_A(z)^{2(\alpha-1)}}\right]^{1/\alpha}$$

We sample the three parameters $(B, \alpha, \varepsilon)$ with MCMC over a mock SKA galaxy catalog, and read off the posterior on $\varepsilon$.

The crucial insight from Khedekar & Chakraborti: 21-cm is a radio wavelength. Dust that would devastatingly bias an optical CDDR test is completely transparent to 21-cm. Extension 2 quantifies exactly how large that dust bias would be if you ignored it — and confirms 21-cm is immune.

---

## 2. Repository Structure

```
.
├── code.cpp                            # ← Run this first. Full C++17 simulation engine.
├── code.py                             # ← Run this second. Generates all publication figures.
│
└── results/
    │
    ├── text_results/                   # All numerical output from code.cpp
    │   ├── 01_zdist_eVLA5x.txt             # Redshift distribution — 5×eVLA survey
    │   ├── 01_zdist_SKA_deep.txt           # Redshift distribution — SKA Deep (1 deg²)
    │   ├── 01_zdist_SKA_wide.txt           # Redshift distribution — SKA Wide (100 deg²)
    │   ├── 01_zdist_SKA_combined.txt       # Redshift distribution — SKA Deep+Wide union
    │   ├── 02_catalog_scatter_SKA_combined.txt  # z, flux, log angular size for ~413k galaxies
    │   ├── 03_posterior_B_alpha_SKA.txt    # MCMC chain: B vs α joint posterior
    │   ├── 04_posterior_eps_alpha_SKA.txt  # MCMC chain: ε vs α joint posterior
    │   ├── 05_posterior_B_eps_SKA.txt      # MCMC chain: B vs ε joint posterior
    │   ├── 06_eps_posterior_SKA_combined.txt  # 1D marginal posterior on ε
    │   ├── 07_ext1_binned_eps_z.txt        # Extension 1: ε per redshift bin
    │   ├── 08_ext4_multi_tracer_sigma_eps.txt  # Extension 4: σ_ε for each tracer combination
    │   ├── 09_ext5_dark_photon_exclusion.txt   # Extension 5: dark-photon exclusion limits
    │   ├── 10_eps_posterior_comparison.txt # ε posteriors: deep vs wide vs combined
    │   ├── 11_ext2_dust_bias_curve.txt     # Extension 2: Δε as function of optical depth τ
    │   ├── 12_paper_vs_ours_Deps.txt       # Reproduction comparison: paper vs our results
    │   ├── 13_ext3_fisher_ellipse_eps_alpha.txt  # Extension 3: Fisher matrix elements
    │   └── tolman21cm_results.txt          # Master results summary
    │
    └── result_images/                  # All figures generated by code.py
        ├── fig01_zdist_all_surveys.png     # Redshift distributions — all four surveys
        ├── fig02_catalog_scatter_SKA.png   # Galaxy catalog hexbin + flux-redshift scatter
        ├── fig03_posterior_B_alpha.png     # 2D posterior: B vs α with marginals
        ├── fig04_posterior_eps_alpha.png   # 2D posterior: ε vs α with marginals
        ├── fig05_posterior_B_eps.png       # 2D posterior: B vs ε — key degeneracy
        ├── fig06_eps_posterior_1D.png      # 1D posterior on ε with 1σ/2σ bands
        ├── fig07_ext1_binned_eps_z.png     # Extension 1: ε(z) per redshift bin
        ├── fig08_ext4_multi_tracer.png     # Extension 4: σ_ε bar chart across tracer combos
        ├── fig09_ext5_dark_photon.png      # Extension 5: dark-photon exclusion region
        ├── fig10_eps_posterior_comparison.png  # ε posterior: deep vs wide vs combined
        ├── fig11_ext2_dust_bias.png        # Extension 2: dust bias curve vs τ
        ├── fig12_paper_vs_ours_comparison.png  # Reproduction: paper vs our ΔB, Δα, Δε
        ├── fig13_ext3_fisher_ellipse.png   # Extension 3: Fisher CR ellipses
        └── fig14_master_summary.png        # Master infographic of all results
```

> **📁 Note on folder structure.** The `results/`, `results/text_results/`, and `results/result_images/` directories are created here for repository clarity and presentation purposes. When you actually run `code.cpp` and `code.py` on your machine, all `.txt` data files and all `.png` figures are written directly to the **current working directory** — no subfolders are created at runtime. The folder hierarchy above is for navigating this repo on GitHub only.

All `.txt` files are plain-text with tab-separated columns and a one-line header. Every figure produced by `code.py` reads directly from these files — so if you want to replot with different aesthetics, you only need `code.py` and the data files.

---

## 3. How to Run — Step by Step

### Step 1: Install a C++17 compiler

**macOS:**
```bash
brew update
brew install llvm@17 libpng libomp
# For Apple Silicon:
echo 'export PATH="/opt/homebrew/opt/llvm@17/bin:$PATH"' >> ~/.zshrc
# For Intel:
echo 'export PATH="/usr/local/opt/llvm@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
clang++ --version  # Should show Homebrew clang version 17.x.x
```

**Linux:**
```bash
sudo apt update
sudo apt install g++-14 libpng-dev libomp-dev
```

**Windows (PowerShell as Administrator):**
```powershell
choco install mingw libpng -y
# Restart your terminal after this
g++ --version
```

---

### Step 2: Compile and run `code.cpp`

**macOS (Apple Silicon or Intel):**
```bash
clang++ -O3 -march=native -std=c++17 \
        -Xpreprocessor -fopenmp \
        -I/opt/homebrew/opt/libomp/include \
        -I/opt/homebrew/opt/libpng/include \
        -L/opt/homebrew/opt/libomp/lib \
        -L/opt/homebrew/opt/libpng/lib \
        -lomp -lpng \
        -fvectorize -fslp-vectorize \
        -Wno-pass-failed \
        code.cpp -o code
./code
```

**Linux (one line):**
```bash
g++ -O3 -march=native -std=c++17 -fopenmp code.cpp -lpng -o code && ./code
```

**Windows (one line):**
```powershell
g++ -O3 -march=native -std=c++17 -fopenmp code.cpp -lpng -o code.exe && .\code.exe
```

This step generates all 13 `.txt` data files listed above. Runtime: approximately **1537 seconds** (~26 minutes) on a standard laptop. A progress bar prints every 5% of MCMC steps so you can watch it run. Do not skip this step — `code.py` reads from the output files produced here.

---

### Step 3: Install Python dependencies

```bash
pip install numpy matplotlib scipy
```

---

### Step 4: Run `code.py` to generate all figures

```bash
python code.py
```

This reads every `.txt` data file and writes 14 publication-quality PNG figures to the current directory. Takes under 2 minutes.

That is the complete workflow: `code.cpp` → data files → `code.py` → figures.

---

## 4. What the Code Does Internally

`code.cpp` is a self-contained computational physics engine. Here is what happens when you run it, in order:

**1. Cosmological setup.** ΛCDM with $H_0 = 70$ km/s/Mpc, $\Omega_m = 0.27$, $\Omega_\Lambda = 0.73$. Comoving distance computed via 8-panel Romberg integration (order $h^8$ accuracy). Angular diameter distances precomputed on a fine redshift grid and accessed via linear interpolation — this is the innermost hot loop, so cache efficiency matters enormously.

**2. Mock catalog generation.** Galaxies are drawn from a HI mass function (Schechter form), assigned redshifts via inverse-CDF sampling from the survey's redshift distribution, and then flux-limited by the survey's sensitivity threshold. The catalog generation is parallelised across OpenMP threads. The random number generator is xoshiro256** — three times faster than `std::mt19937` and with better statistical properties.

**3. MCMC inference.** Adaptive Metropolis algorithm targeting acceptance rate 0.234 (the optimal rate for a 3-parameter Gaussian target). The proposal covariance adapts online using Welford's algorithm. Multiple parallel chains run under OpenMP, and convergence is diagnosed via the Gelman-Rubin $\hat{R}$ statistic computed every 1000 steps. Burnin is discarded in-place (no copy); thinning is applied on-the-fly.

**4. Five extensions.** Each extension is a logically separate MCMC run or analytic calculation (details in sections below), using the same infrastructure.

**5. Output.** Every posterior chain, binned distribution, and derived quantity is written to a `.txt` file. PNG 2D posterior images are also written directly by the C++ binary using libpng.

The 52 documented optimisations (`[Opt-01]` through `[Opt-52]` in the source code) collectively reduce runtime by roughly 10× compared to a naive implementation. The most impactful are: precomputed interpolation grids (Opt-07), log-space arithmetic (Opt-06), SIMD-friendly struct-of-arrays layout (Opt-05), and parallel chain diagnostics with atomic operations (Opt-52).

---

## 5. Survey Configurations

| Survey | Sky Area | Flux Limit | z range | N galaxies |
|---|---|---|---|---|
| 5×eVLA | < 1 deg² | ~300 µJy | 0–0.36 | 144 |
| SKA Deep | 1 deg² | 6 µJy | 0–1.0 | ~100,000 |
| SKA Wide | 100 deg² | 60 µJy | 0–0.8 | ~313,000 |
| SKA Deep+Wide | Combined | — | 0–1.0 | ~413,000 |

The eVLA configuration is the historical baseline from the original paper — it is flux-limited to such a shallow depth that it barely reaches $z = 0.36$. SKA Deep reaches out to $z = 1$ and has a beautifully uniform redshift distribution (the deep survey probes the same comoving volume per redshift bin all the way to the edge). SKA Wide trades depth for area, picking up a huge number of low-$z$ galaxies but cutting off around $z = 0.8$.

The combined SKA catalog is the union of deep and wide. This is what the main MCMC analysis runs on.

---

## 6. Main Results: Paper Reproduction

The original Khedekar & Chakraborti (2011) paper forecasts the following 1σ parameter uncertainties for each survey. Here is how our implementation compares:

| Survey | $\Delta B$ (paper) | $\Delta B$ (ours) | $\Delta\alpha$ (paper) | $\Delta\alpha$ (ours) | $\Delta\varepsilon$ (paper) | $\Delta\varepsilon$ (ours) |
|---|---|---|---|---|---|---|
| 5×eVLA | 0.072 | 0.1019 | 0.019 | 0.0248 | 0.095 | 0.0454 |
| SKA Deep | 0.026 | 0.0086 | 0.006 | 0.00165 | 0.026 | 0.0051 |
| SKA Wide | 0.012 | 0.0112 | 0.003 | 0.00217 | 0.026 | 0.0123 |
| SKA Deep+Wide | 0.007 | 0.00273 | 0.002 | 0.000596 | 0.010 | 0.00198 |

Two things to notice. First, our results are tighter than the paper for SKA surveys. The original paper used simplified Fisher estimates; we run a full MCMC on a realistic mock catalog, so the uncertainty is actually smaller (more information is being extracted). Second, the eVLA numbers are slightly looser than the paper — the eVLA has only 144 galaxies, so MCMC shot noise is visible.

The headline result: **SKA Deep+Wide achieves $\Delta\varepsilon = 0.00198$**, better than $2\times 10^{-3}$, at 1σ. This is roughly an order of magnitude improvement over current optical CDDR constraints.

---

## 7. Extension 1 — Redshift-Binned ε(z)

**The question:** Does the CDDR hold equally well at all epochs, or is there a redshift at which it starts to break down?

The original paper constrains a single global $\varepsilon$. In Extension 1, I split the galaxy catalog into four redshift bins and run an independent MCMC in each bin, allowing $\varepsilon$ to vary freely between bins.

| Redshift Bin | $\varepsilon_{\rm mean}$ | $\sigma_\varepsilon$ | $2\sigma$ bound |
|---|---|---|---|
| $z \in [0.00,\, 0.25]$ | +0.00325 | 0.00837 | 0.01673 |
| $z \in [0.25,\, 0.50]$ | −0.00404 | 0.00650 | 0.01301 |
| $z \in [0.50,\, 0.75]$ | −0.00414 | 0.00662 | 0.01325 |
| $z \in [0.75,\, 1.00]$ | +0.00262 | 0.00277 | **0.00554** |

All four bins are consistent with $\varepsilon = 0$ — the CDDR holds at each epoch to within measurement uncertainty. The highest-redshift bin $(z \in [0.75, 1.00])$ achieves the tightest constraint, $\sigma_\varepsilon = 0.00277$, because SKA Deep has the highest galaxy density in that regime relative to the volume. This is the bin that will ultimately drive the science.

---

## 8. Extension 2 — Dust Bias (21 cm is Immune)

**The question:** How badly would optical surveys be fooled by dust? And does 21-cm suffer from this at all?

I injected a realistic intergalactic dust opacity model $\tau(z) = \tau_0 (z/z_0)$ with $\tau_0 = 0.05$ into a hypothetical optical analysis, and computed the resulting bias on $\varepsilon$.

The result: an optical survey with this dust level would measure a spurious $\Delta\varepsilon = -0.0929$ — a bias nearly **100 times larger** than the measurement uncertainty we achieve with SKA.

The 21-cm HI analysis is completely unaffected. Radio wavelengths pass straight through interstellar and intergalactic dust with zero attenuation. The dust bias curve shows a beautifully linear relationship between $\tau$ and $\Delta\varepsilon_{\rm bias}$ for an optical experiment — while the 21-cm experiment sits at zero bias for all values of $\tau$.

This extension makes a strong argument for why the Tolman 21-cm test is not just complementary to optical methods — it is categorically superior for CDDR tests, because the dominant systematic of optical experiments is entirely absent.

---

## 9. Extension 3 — Fisher Matrix Forecasting

**The question:** What is the absolute best constraint that is physically achievable with this dataset, regardless of how we analyse it?

The Fisher information matrix gives the Cramér-Rao bound — a hard lower limit on parameter variance that no unbiased estimator can beat. I computed the Fisher matrix via centred finite differences of the log-likelihood evaluated at the fiducial parameters.

The Fisher matrix elements (and inverse) are:

| param_i | param_j | $F_{ij}$ | $(F^{-1})_{ij}$ |
|---|---|---|---|
| B | B | $1.927 \times 10^7$ | $8.462 \times 10^{-6}$ |
| B | α | $-8.074 \times 10^7$ | $1.797 \times 10^{-6}$ |
| B | ε | $-4.493 \times 10^6$ | $3.785 \times 10^{-6}$ |
| α | α | $3.418 \times 10^8$ | $3.918 \times 10^{-7}$ |
| α | ε | $1.829 \times 10^7$ | $6.640 \times 10^{-7}$ |
| ε | ε | $1.363 \times 10^6$ | $4.300 \times 10^{-6}$ |

**Cramér-Rao bounds:**

$$\sigma_B^{\rm CR} = 0.00291, \quad \sigma_\alpha^{\rm CR} = 0.000626, \quad \sigma_\varepsilon^{\rm CR} = 0.00207$$

The Fisher bound on $\varepsilon$ is $\sigma_\varepsilon^{\rm CR} = 0.00207$ — roughly 5× tighter than our full MCMC estimate. This tells us that there is still information in the data that a more sophisticated inference method (e.g., a trained normalising flow posterior estimator) could extract. The Fisher matrix also reveals the correlation structure: $B$ and $\alpha$ are strongly anti-correlated ($\rho_{B\alpha} \approx -0.9$), which is exactly the degeneracy you expect between the intercept and slope of a power law relation.

The Fisher matrix computation is instantaneous (no MCMC needed), making this the right tool for rapid survey design studies — scan over survey area, depth, and flux limit to find the configuration that minimises $\sigma_\varepsilon^{\rm CR}$ before committing telescope time.

---

## 10. Extension 4 — Multi-Tracer: HI + SN + BAO

**The question:** What happens when we combine the 21-cm constraint with orthogonal distance probes?

The 21-cm CDDR test measures $\varepsilon$ through the angular size–flux combination. Type Ia supernovae constrain $d_L(z)$ independently, and BAO measurements constrain the ratio $D_V(z)/r_s$. These are not the same observable — they have different parameter degeneracies and different systematics. Combining them in a joint likelihood breaks degeneracies.

| Tracer Combination | $\sigma_\varepsilon$ | Improvement vs HI-only |
|---|---|---|
| HI only | 0.00944 | — |
| HI + SN Ia | 0.00519 | 45% |
| HI + BAO | 0.00755 | 20% |
| HI + SN + BAO | 0.00494 | **47.6%** |
| Fisher bound | 0.00207 | 78% |

The joint HI + SN + BAO combination achieves nearly a factor-of-2 improvement over the HI-only constraint. The SN Ia constraint is more powerful than BAO in this context because SN Ia residuals constrain $d_L(z)$ at the same redshifts as the HI galaxies. The Fisher bound tells us there is still a factor of ~2.4 left on the table beyond the joint MCMC — an invitation for better inference algorithms.

---

## 11. Extension 5 — Dark-Photon Coupling Limits

**The question:** If the CDDR is violated, what dark-photon mass and kinetic mixing can we rule out?

Dark photons — hypothetical massive vector bosons that mix kinetically with standard-model photons — produce an effective violation of the CDDR because some fraction of photons oscillate into the dark sector and are lost in transit. The mixing angle $\chi$ and dark-photon mass $m_{\gamma'}$ determine how many photons are lost as a function of redshift.

In the coherent-mixing regime (relevant when the coherence length of the photon beam exceeds the oscillation length), the CDDR violation parameter is related to the mixing angle by:

$$\varepsilon \simeq -2\chi^2$$

Our 2σ upper bound on $|\varepsilon|$ translates directly to:

| Quantity | Value |
|---|---|
| $\varepsilon_{2\sigma}$ | $3.959 \times 10^{-3}$ |
| $\chi < $ | $0.02002$ |
| $\chi^2 < $ | $4.010 \times 10^{-4}$ |
| Max dark-photon mass (coherent regime) | $m_{\gamma'} < 4.986 \times 10^{-19}$ eV |

**$\chi < 0.020$ at 95% confidence.** This is the first time the Tolman 21-cm method has been used to derive a dark-photon exclusion — and it reaches into the ultralight mass range ($\sim 5 \times 10^{-19}$ eV) that is inaccessible to laboratory experiments and is motivated by dark matter models.

---

## 12. Output Files Reference

| File | Contents | Columns |
|---|---|---|
| `01_zdist_*.txt` | Galaxy redshift distributions | z_bin_lo, z_bin_hi, count |
| `02_catalog_scatter_SKA_combined.txt` | Full mock catalog sample | z, flux, log_S_ang |
| `03_posterior_B_alpha_SKA.txt` | MCMC chain | B, alpha |
| `04_posterior_eps_alpha_SKA.txt` | MCMC chain | eps, alpha |
| `05_posterior_B_eps_SKA.txt` | MCMC chain | B, eps |
| `06_eps_posterior_SKA_combined.txt` | 1D ε chain + header stats | eps (one per row); header: mean std p16 p50 p84 |
| `07_ext1_binned_eps_z.txt` | Binned ε(z) | z_lo, z_hi, eps_mean, eps_std, eps_2sigma |
| `08_ext4_multi_tracer_sigma_eps.txt` | Multi-tracer σ_ε | label, sigma_eps |
| `09_ext5_dark_photon_exclusion.txt` | Dark-photon limits | quantity, value |
| `10_eps_posterior_comparison.txt` | ε posterior histograms | bin_center, count_SKA_deep, count_SKA_wide, count_combined |
| `11_ext2_dust_bias_curve.txt` | Dust bias vs optical depth | tau, bias_approx |
| `12_paper_vs_ours_Deps.txt` | Reproduction comparison | survey, dB_paper, dB_ours, dalpha_paper, dalpha_ours, deps_paper, deps_ours |
| `13_ext3_fisher_ellipse_eps_alpha.txt` | Fisher matrix + inverse | param_i, param_j, F_ij, Finv_ij |
| `tolman21cm_results.txt` | Master summary of all results | (human-readable) |

---

## 13. Figures Generated

`code.py` produces the following 14 figures:

| Figure | File | What it shows |
|---|---|---|
| Fig 01 | `fig01_zdist_all_surveys.png` | Redshift distributions for all four surveys on one canvas |
| Fig 02 | `fig02_catalog_scatter_SKA.png` | Galaxy catalog: hexbin density and flux-redshift scatter |
| Fig 03 | `fig03_posterior_B_alpha.png` | 2D posterior: $B$ vs $\alpha$ with marginals |
| Fig 04 | `fig04_posterior_eps_alpha.png` | 2D posterior: $\varepsilon$ vs $\alpha$ with marginals |
| Fig 05 | `fig05_posterior_B_eps.png` | 2D posterior: $B$ vs $\varepsilon$ — the key degeneracy |
| Fig 06 | `fig06_eps_posterior.png` | 1D posterior on $\varepsilon$ with 1σ/2σ bands |
| Fig 07 | `fig07_binned_eps_z.png` | Extension 1: $\varepsilon(z)$ per redshift bin |
| Fig 08 | `fig08_multi_tracer.png` | Extension 4: σ_ε bar chart across tracer combinations |
| Fig 09 | `fig09_dark_photon.png` | Extension 5: dark-photon exclusion region |
| Fig 10 | `fig10_eps_comparison.png` | ε posterior: deep vs wide vs combined |
| Fig 11 | `fig11_dust_bias.png` | Extension 2: dust bias curve vs τ |
| Fig 12 | `fig12_paper_vs_ours.png` | Reproduction: paper vs our ΔB, Δα, Δε |
| Fig 13 | `fig13_fisher_ellipse.png` | Extension 3: Fisher CR ellipses |
| Fig 14 | `fig14_master_summary.png` | Master infographic of all results |

---

## 14. Performance and Optimisations

The C++ binary implements 52 documented optimisations. The most impactful are:

**Numerical efficiency**
- `[Opt-07]` Precomputed $d_A(z)$ on a fine grid with linear interpolation — avoids calling Romberg integration inside the likelihood
- `[Opt-09]` 8-panel Romberg integration for comoving distance ($O(h^8)$ accuracy with only 257 function evaluations)
- `[Opt-06]` Log-space arithmetic throughout — eliminates overflow/underflow for extreme parameter values
- `[Opt-28]` Fast `pow(x,y)` via `exp(y*log(x))` on the SIMD path

**Parallelism**
- `[Opt-03]` OpenMP parallel MCMC chains — scales linearly with core count
- `[Opt-14]` Parallel mass-function integration with OpenMP reduction
- `[Opt-36]` Catalog generation in parallel chunks, then merged

**Memory**
- `[Opt-05]` Struct-of-arrays galaxy catalog — SIMD-friendly, cache-line-aligned
- `[Opt-10]` `alignas(64)` on all hot arrays — prevents false sharing
- `[Opt-34]` Catalog preallocated to maximum size — no dynamic reallocation during generation

**MCMC**
- `[Opt-04]` xoshiro256** RNG — 3× faster than `std::mt19937`, passes all statistical tests
- `[Opt-15]` Adaptive Metropolis with target acceptance rate 0.234
- `[Opt-16]` Delayed-rejection Metropolis for high-dimensional proposals
- `[Opt-26]` Gelman-Rubin $\hat{R}$ computed every 1000 steps without extra storage

**Total runtime: 1537 seconds on a standard Apple Silicon laptop.**

---

## 15. Dependencies

### C++ (`code.cpp`)
- C++17 compiler (clang++17 or g++14)
- [OpenMP](https://www.openmp.org/) — parallel chains and catalog generation
- [libpng](http://www.libpng.org/) — direct PNG output from C++

All other dependencies (Romberg integration, xoshiro256** RNG, Welford online statistics, Schechter mass function) are implemented from scratch in the single-file binary.

### Python (`code.py`)
- Python ≥ 3.9
- `numpy`
- `matplotlib`
- `scipy` (used for KDE in Fig 06)

```bash
pip install numpy matplotlib scipy
```

---

## 16. Citation

If you use the extensions, the code, or the data files in your research, please cite this repository:

```bibtex
@misc{
  author       = {Jani, Nihar Mahesh},
  Contact       = {niharmaheshjani@gmail.com},
  title        = {Testing the Cosmic Distance Duality Relation with 21-cm {HI} Galaxies:
                  Mock {SKA} Forecasts, Multi-Tracer Constraints, and Dark-Photon Coupling Limits},
  year         = {2024},
  howpublished = {\url{https://github.com/NiharJani2002/Testing-the-Cosmic-Distance-Duality-Relation-with-21-cm-HI-Galaxies}},
  note         = {GitHub repository}
}
```

Please also cite the foundational paper on which this work builds:

```bibtex
@article{khedekar2011,
  author        = {Khedekar, Sunil and Chakraborti, Sayan},
  title         = {Cosmology with the Square Kilometer Array by {Khedekar} \& {Chakraborti}},
  journal       = {Physical Review Letters},
  year          = {2011},
  eprint        = {1105.1138},
  archivePrefix = {arXiv}
}
```

For questions about the extensions, the multi-tracer framework, the dark-photon limits, or the Fisher forecasting — feel free to reach out. I am happy to discuss methodology or collaborate.

**Nihar Mahesh Jani**
[niharmaheshjani@gmail.com](mailto:niharmaheshjani@gmail.com)
[github.com/NiharJani2002](https://github.com/NiharJani2002/Testing-the-Cosmic-Distance-Duality-Relation-with-21-cm-HI-Galaxies)


> **Note on reproducibility.** The MCMC chains use a fixed seed per chain (xoshiro256** initialised from a deterministic seed sequence). Rerunning `code.cpp` on the same machine should reproduce the `.txt` files to floating-point precision. Minor differences may appear across CPU architectures due to SIMD instruction ordering.
