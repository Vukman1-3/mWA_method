# XRD Peak Profile Analysis for Dislocation Density

This repository contains Python scripts to perform a modified Warren-Averbach (WA) analysis on X-Ray Diffraction (XRD) data.


The goal of this workflow is to calculate the **dislocation density** and **dislocation arrangement radius** from raw XRD peak profiles.

It does this by:
1.  Taking raw $2\theta$-Intensity XRD data.
2.  Converting the peaks into Fourier space to get $A(L)$ coefficients.
3.  Using the Warren-Averbach method to separate size and strain effects.
4.  Fitting the strain coefficients to a physical model for dislocations.

---

## Critical: Instrumental Broadening Correction

Your measured XRD peak is a combination of your sample's true profile and broadening from the instrument itself. You **must** correct for this.

The measured Fourier coefficients $A_{\text{Measured}}(L)$ are a product of the true coefficients $A_{\text{Corrected}}(L)$ and the instrument's coefficients $A_{\text{Instrumental}}(L)$.

$$A_{\text{Measured}}(L) = A_{\text{Corrected}}(L) \cdot A_{\text{Instrumental}}(L)$$

In Fourier space, this correction is a simple division:

$$A_{\text{Corrected}}(L) = \frac{A_{\text{Measured}}(L)}{A_{\text{Instrumental}}(L)}$$

### How to Apply the Correction:

1.  **Run a Standard:** Collect an XRD scan from a "perfect" calibrant material (like NIST LaB₆) that has no strain and very large crystals.
2.  **Process the Standard:** Run **Step 1** of the workflow (Script `compute_A_for_all_peaks_commonL`) on your calibrant's data. This gives you $A_{\text{Instrumental}}(L)$.
3.  **Process Your Sample:** Run **Step 1** on your sample data to get $A_{\text{Measured}}(L)$.
4.  **Divide:** Manually divide your sample's $A(L)$ list by the calibrant's $A(L)$ list.
5.  **Continue Analysis:** Use this new, $A_{\text{Corrected}}(L)$ for all following steps.

**All subsequent analysis (Steps 2 and 3) must be performed on these corrected $A(L)$ values.**

---

## High-Level Workflow

The analysis is broken into three main steps, corresponding to the provided scripts.

### 1. Calculate A(L) Fourier Coefficients

* **Script:** `compute_A_for_all_peaks_commonL.py` (Script 2)
* **What it does:** Loads raw `.dat` files (XRD patterns), performs baseline correction, normalization, and the Fourier transform to calculate the $A(L)$ coefficients.
* **Note:** This step provides $A_{\text{Measured}}$ (from samples) and $A_{\text{Instrumental}}$ (from a standard).

### 2. Warren-Averbach "Inner" Fit

* **Script:** `compute_wa_inner_fits.py` (Script 3)
* **What it does:** Performs the WA "inner fit" ($\ln(A)$ vs. $K^2C$) to separate the size component (intercept) from the strain/dislocation component (slope).
* **Note:** **Must be run on your corrected $A(L)$ data.**

### 3. WA "Outer" Fit (Dislocation Model)

* **Script:** `wa_outer_fit_Y_vs_L_YvsLcols.py` (Script 4)
* **What it does:** Fits the slope from Step 2 to a physical model for dislocations to get the final dislocation density ($\rho$) and arrangement radius ($R_e$).