# OE IR DCE VFA

## Expected output maps:

### OE IR T1-mapping
- Baseline T1 (yes, madym_output/T1_IR/T1.nii.gz)
- Baseline R1 (no)
- p1_estimate (?)
- p2_estimate (?)

### OE time-series
- Delta R1(t) (yes, OE_output_maps/delta_R1.nii.gz)
- P-values for enhancing R1(t) (yes, OE_output_maps/R1_p_vals.nii.gz)
- P-values for enhancing S(t) (yes, OE_output_maps/S_p_vals.nii.gz)

- Significance mask for R1(t) at 5%  (yes, OE_output_maps/R1_p_vals_sig_5pct.nii.gz)
- Significance mask for S(t) at 5%  (yes, OE_output_maps/S_p_vals_sig_5pct.nii.gz)
- Significance mask for R1(t) at 5%, Bonferroni corrected  (yes, OE_output_maps/R1_p_vals_sig_5pct_bf.nii.gz)
- Significance mask for S(t) at 5%, Bonferroni corrected  (yes, OE_output_maps/S_p_vals_sig_5pct_bf.nii.gz)
- Significance mask for R1(t), Forman corrected  (yes, OE_output_maps/R1_p_vals_sig_forman.nii.gz)
- Significance mask for S(t), Forman corrected  (yes, OE_output_maps/S_p_vals_sig_forman.nii.gz)

### DCE VFA T1-mapping
- Baseline T1 (yes, madym_output/T1_VFA/T1.nii.gz)
- Baseline R1 (no)

### DCE time-series, using first echo
- Delta C(t) (yes, DCE_output_maps/delta_C.nii.gz)
- P-values for enhancing C(t) (yes, DCE_output_maps/C_p_vals.nii.gz)
- P-values for enhancing S(t) (yes, DCE_output_maps/S_p_vals.nii.gz)

- AUC 60 (yes, madym_output/ETM_pop/IAUC60.nii.gz)
- Binary map of AUC 60 > 0 (no)

- Significance mask for C(t) at 5%  (yes, DCE_output_maps/C_p_vals_sig_5pct.nii.gz)
- Significance mask for S(t) at 5%  (yes, DCE_output_maps/S_p_vals_sig_5pct.nii.gz)

### Combined
- combinedimage_R1_5pc combinedmask_60_R1_5pc_
- combinedimage_S_5pc combinedmask_S_60_5pc_
- combinedimage_AUC60_5pc combinedmask_AUC60_5pc_