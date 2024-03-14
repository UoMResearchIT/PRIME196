# PRIME196 

This is an example pipeline for the [radnet project](https://github.com/UoMResearchIT/radnet-documents). It is meant to contain:

-  `bruker_OE_IR_VFA_DCE_session.json` -- a *session-template* that is used to identify sets of files produced by an MRI scanner as belonging to this particular study. See [`PreclinicalMRI/Bruker_pre_process.py`](https://github.com/UoMResearchIT/radnet_core_pipelines/blob/main/src/PreclinicalMRI/pipelines/bruker_pre_process.py)
-  `OE_IR_DCE_VFA.cwl` -- a *CWL-workflow* that defines the study pipeline, using tool-wrappers (and subworkflows from [cwl_madym](https://github.com/UoMResearchIT/cwl_madym))
-  `OE_IR_DCE_VFA_input.yml.template` -- an *input-template* that can be used to link the files matched by the *session-template* to the *CWL-workflow*.

The *CWL-workflow* and the *input-template* serve as public references of the study's methodology. 

> **FUTURE**: particular instances of this pipeline (together with data) should become [Research Object Crates](https://www.researchobject.org/ro-crate/)?

## OE-IR-DCE-VFA pipeline

Two independent chains: **OE-IR**, and **DCE-VFA**, merged at the end only to generate _combined-masks_.

**OE-IR** calculates T1 map using Inversion Recovery with Efficiency weighting, before (baseline) and after the delivery of supplemental oxygen (enhancing).

> "Elevated concentrations of dissolved oxygen increase tissue's longitudinal relaxation rate (R1). Following the delivery of supplemental oxygen, initially well-oxygenated regions where haemoglobin is well saturated develop an increase in dissolved molecular oxygen with consequential reduction in T1 times." (McCabe et al. 2023)

**DCE-VFA** (Dynamic Contrast Enhanced MRI) attempts to fit a mathematical model of how a tracer (CA) perfuses through tissue vasculature.

> "Quantitative DCE-MRI data acquisitions require three measurements: 1) recording of a map of the native T1 values before contrast administration; 2) acquisitions of T1-weighted images following CA introduction at a reasonably high temporal resolution to be able to characterize the kinetics of the CA entry and exit into tissue; and 3) a method to estimate the time rate of change of the concentration of the CA in the blood plasma, the so-called arterial input function (AIF).  
> ...  
> The most commonly used method of mapping T1 before administering the CA is to use a series of so-called spoiled gradient-recalled echo (SPGRE) images acquired with different degrees of T1-weighting... The idea is to collect SPGRE data at multiple flip angles [...] and fit the data to Eq. (3) with S0 and T1 as floating parameters." (Yankeelov & Gore, 2009).

### A1. OE IR T1-mapping (`madym_T1 --T1_method IR_E`)

#### Inputs:
- `IR/OE_*.nii.gz` volumes (with *.json metadata)
- Parameters: `T1_noise`, `TR` (read from metadata)
#### Outputs:
- Maps (`IRE/*.nii.gz`) for `T1`, `M0`, `efficiency`, and `error_tracker`
- Missing (not important) `p1_estimate`, `p2_estimate`

### A2. OE time-series (`OE_deltaR1`)
#### Inputs:
- `IRE/T1.nii.gz` and `IRE/efficiency.nii.gz` maps (from *OE T1-mapping*)
- `OE/OE_dyn.nii.gz` volume
- Parameters: `oe_limits`, `average_fun`, ...
#### Outputs:
- Maps (`OE_output_maps/*.nii.gz`) for `R1`, `delta_R1`, `R1_baseline`, `R1_enhancing`, `R1_p_vals`, and `S_p_vals`.

### B1. DCE VFA T1-mapping (`madym_T1 --T1_method VFA`)
#### Inputs:
- `VFA/VFA_*.nii.gz` volumes (with *.json metadata)
- Parameters: `T1_noise`; `TR`, `FA` (read from metadata)
#### Outputs:
- Maps (`T1_VFA/*.nii.gz`) for `T1`, `M0`, and `error_tracker`

### B2. DCE time-series, using first echo (`DCE_deltaCt`)
#### Inputs:
- `T1_VFA/T1.nii.gz` map (from *VFA T1-mapping*)
- `DCE/dce_dyn_echo1.nii.gz` volume
- Parameters: `dce_limits`, `relax_coeff`, `average_fun`, ...
#### Outputs:
- Maps (`DCE_output_maps/*.nii.gz`) for `Ct`, `delta_C`, `C_baseline`, `C_enhancing`, `C_p_vals`, and `S_p_vals`.

### B3. (`madym_DCE --model ETM`)
#### Inputs:
- `T1_VFA/T1.nii.gz` and `error_tracker` maps (from *VFA T1-mapping*)
- `DCE/dce_dyn_echo1.nii.gz` volume
- Parameters: `dose`, `hct`, `iauc`, `inj`, ...
#### Outputs (??? - untested!)
- AUC 60 (yes, ETM/IAUC60.nii.gz)
- Binary map of AUC 60 > 0 (no)

### [Combined] significance masks (`OE_DCE_summary`)
#### Inputs:
- `*_p_vals` maps from `OE_output_maps` and `DCE_output_maps`
- `sig_level` currently (hard-set?) at 5%
- ROI map (expected to be set semi-interactively)
#### Outputs:
- Significance masks for `R1`, `S(t)`, and `C(t)` at `sig_level`
- Forman and Bonferroni corrected significance masks
- Combined masks

> CHECK: are output map names `*_5pct_*` adjusted to `sig_level`?
