# HumanCortex_fMRI_Proc_V261014

## 1. Introduction

**HumanCortex_fMRI_Proc_V261014** is a toolbox developed for human fMRI analysis based on **SPM12**, capable of processing both single-echo and multi-echo data. At the current stage, only data acquired from United Imaging Healthcare (UIH) scanners are supported. In particular, we would like to express our sincere gratitude to the developers of the DPABI<sup>[1]</sup> (https://github.com/Chaogan-Yan/DPABI), RESTplus<sup>[2]</sup> (http://restfmri.net/forum/restplus), NIfTI_20140122 and dicm2nii (https://github.com/xiangruili/dicm2nii), from which the code in `preprocessing\basic` and `config\dicm2nii-master` was primarily adapted, for providing robust and convenient functions that have greatly facilitated fMRI data preprocessing and analysis.

## 2. Installation

1. Download **SPM12** (https://www.fil.ion.ucl.ac.uk/spm/software/spm12/), and add them to the MATLAB environment path.
2. Extract the **HumanCortex_fMRI_Proc_V261014** package and add it to the MATLAB environment path (subfolders must be included).
3. MATLAB version must be **R2022a or above**.

## 3. Data Input Format

The project root directory contains the folders `Fun`, `FunME`, `T1`, and `FieldMap`. The directory structure is organized as follows:

> Raw scanning data can be converted into this format through the batch workflow described in [7.DICOM Sort and Organization](#7-dicom-sort-and-organization).

```text
ProjectRoot/
├── Fun/                       % Single-echo functional data
│   ├── Subject001/
│   │   ├── S1/                   % Session level (if applicable)
│   │   │   └── *.dcm             % DICOM data for this session
│   │   ├── S2/
│   │   │   └── *.dcm
│   │   └── ...
│   ├── Subject002/
│   │   ├── S1/
│   │   │   └── *.dcm
│   │   └── ...
│   └── ...
│
├── FunME/                     % Multi-echo functional data
│   ├── Subject001/
│   │   ├── S1/                   % Session level (if applicable)
│   │   │   ├── Echo1/
│   │   │   │   └── *.dcm         % DICOM data for Echo1
│   │   │   ├── Echo2/
│   │   │   │   └── *.dcm         % DICOM data for Echo2
│   │   │   └── ...
│   │   ├── Echo1/                % If no session level exists, Echo folders
│   │   │   └── *.dcm             % are placed directly under the subject folder
│   │   ├── Echo2/
│   │   │   └── *.dcm
│   │   └── ...
│   ├── Subject002/
│   │   ├── Echo1/
│   │   │   └── *.dcm
│   │   ├── Echo2/
│   │   │   └── *.dcm
│   │   └── ...
│   └── ...
│
├── T1/
│   ├── Subject001/
│   │   └── *.dcm                 % DICOM data directly under the subject folder
│   ├── Subject002/
│   │   └── *.dcm
│   └── ...
│
└── FieldMap/
    ├── Magnitude1/
    │   ├── Subject001/
    │   │   └── *.dcm             % DICOM data for Magnitude1
    │   ├── Subject002/
    │   │   └── *.dcm
    │   └── ...
    ├── Magnitude2/
    │   ├── Subject001/
    │   │   └── *.dcm             % DICOM data for Magnitude2
    │   ├── Subject002/
    │   │   └── *.dcm
    │   └── ...
    └── PhaseDiff/
        ├── Subject001/
        │   └── *.dcm             % DICOM data for PhaseDiff
        ├── Subject002/
        │   └── *.dcm
        └── ...

```

**Notes:**

| Folder / File | Required | Description |
| --- | --- | --- |
| `Fun/` | Yes | Single-echo functional data; each subject folder contains session-level folders (if applicable) with DICOM data, or DICOM data directly under the subject folder if no sessions exist |
| `FunME/` | Yes | Multi-echo functional data; each subject folder contains session-level folders (if applicable), and each session (or the subject folder itself if no sessions exist) contains `Echo1/`, `Echo2/`, ... folders holding the corresponding DICOM data |
| `T1/` | No | Structural data; each subject folder contains DICOM data directly |
| `FieldMap/Magnitude1/` | No | Magnitude1 field map data; each subject folder contains DICOM data directly |
| `FieldMap/Magnitude2/` | No | Magnitude2 field map data; each subject folder contains DICOM data directly |
| `FieldMap/PhaseDiff/` | No | Phase difference field map data; each subject folder contains DICOM data directly |

> File names are not fixed; they only need to correspond one-to-one in the parameter configuration file.

> Multi-echo and single-echo data do not need to coexist. Only one of them is required, depending on the needs of the study.

> If the input is dcm, no additional processing is required.

> Before running the pipeline, the raw DICOM data must be organized into the format above. If the raw data is in a non-standard format (e.g., compressed archives, overly long filenames, or non-standard directory structures), please refer to **Appendix A: DICOM Sort User Manual** for the data organization workflow.

## 4. Parameter Configuration File

The parameter configuration file specifies the parameters for each step of the pipeline (e.g., body position rotation, origin alignment, left-right flip, template selection). This file must be selected along with the data path during runtime. The `Process_Type` field determines which pipeline is executed.

### 4.1 Example Configuration File (Preprocessing)

The following is an example for resting-state preprocessing:

```json
{
    "Config_INFO":{
        "Template":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\MNI152_T1_1mm.nii",
        "TPM":{
            "GM_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,1",
            "WM_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,2",
            "CSF_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,3",
            "Skull_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,4",
            "Scalp_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,5",
            "Background_Path":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\TPM.nii,6"
        },
        "Mask":{
            "Brain":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\mask\\BrainMask.nii",
            "WM":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\mask\\WhiteMask.nii",
            "CSF":"D:\\HumanCortex_fMRI_Proc_V261014\\config\\mask\\CsfMask.nii"
        }
    },
    "Process_Type": "Prep",
    "Import_INFO":{
        "Start_Fun_Name": "Fun",
        "Start_T1_Name": "T1"
    },
    "Data_INFO":{
        "TR": [],
        "Snum": 1,
        "Enum": 3,
        "RefEcho": 1
    },
    "Process_INFO":{
        "Step_H": {"Name": "DICOM_2_Nii", "Brain_Size_mm": 150},
        "Step_T": {"Name": "Remove_First_n_Timepoints", "Del_Tps": 10},
        "Step_A": {"Name": "Slice_Timing", "Slice_Order": [], "Ref_Slice": []},
        "Step_R": {"Name": "Realign"},
        "Step_Cb": { "Name": "ME_Combine" },
        "Step_W": {"Name": "Normalize",
                "IsDARTEL": 0,
                "ResVoxelSize": [2, 2, 2]
                },
        "Step_S": {"Name": "Smooth", "FWHM": [4, 4, 4]},
        "Step_D": {"Name": "Detrend"},
        "Step_C": {"Name": "Covs_Regressing",
            "IsWholeBrain": 0,
            "IsCSF": 1,
            "IsWhiteMatter": 1,
            "IsHeadMotion": 4
        },
        "Step_F": {"Name": "Filter", "Band": [0.01, 0.1]}
    },
    "Delete_Folder_INFO": []
}
```

#### 4.1.1 Parameter Description

##### `Config_INFO`

Configuration information related to templates and masks. The TPM and mask files here can be user-customized, as long as they conform to the SPM12 format and are spatially consistent (i.e., in the same space as the template).

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `Template` | String | Yes | Path to the template image used for normalization and alignment. Should be a NIfTI file. |
| `TPM.GM_Path` | String | Yes | Path to the gray matter (GM) tissue probability map. The suffix `,1` indicates the first volume in the TPM file. |
| `TPM.WM_Path` | String | Yes | Path to the white matter (WM) tissue probability map. The suffix `,2` indicates the second volume in the TPM file. |
| `TPM.CSF_Path` | String | Yes | Path to the cerebrospinal fluid (CSF) tissue probability map. The suffix `,3` indicates the third volume in the TPM file. |
| `TPM.Skull_Path` | String | No | Path to the skull tissue probability map. The suffix `,4` indicates the fourth volume in the TPM file. |
| `TPM.Scalp_Path` | String | No | Path to the scalp tissue probability map. The suffix `,5` indicates the fifth volume in the TPM file. |
| `TPM.Background_Path` | String | No | Path to the background tissue probability map. The suffix `,6` indicates the sixth volume in the TPM file. |
| `Mask.Brain` | String | Yes | Path to the whole-brain mask. |
| `Mask.WM` | String | Yes | Path to the white matter mask. |
| `Mask.CSF` | String | Yes | Path to the CSF mask. |

> At minimum, the TPM files for **GM**, **WM**, and **CSF** are required to run the pipeline. The `Skull`, `Scalp`, and `Background` TPM files are optional. The `Template` and all `Mask` files are required.

##### `Process_Type`

| Field | Type | Description |
| --- | --- | --- |
| `Process_Type` | String | Type of processing pipeline to run. Example values: `"Prep"` (preprocessing), `"Rest"` (resting-state post-processing), `"Task"` (task post-processing), `"Group"` (group-level statistics). |

##### `Import_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Start_Fun_Name` | String | Name of the root folder containing functional data (e.g., `"Fun"`). |
| `Start_T1_Name` | String | Name of the root folder containing structural data (e.g., `"T1"`). |

##### `Data_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `TR` | Array | Repetition time (in seconds). Empty array `[]` means it will be read from the data or left unspecified. |
| `Snum` | Integer | Number of sessions contained in the dataset. |
| `Enum` | Integer | Number of echoes contained in the dataset. |
| `RefEcho` | Integer | Reference echo used for multi-echo combination. |

##### `Process_INFO`

Each step is defined as a key-value pair. The key is the step ID (e.g., `Step_H`), and the value is an object containing the step `Name` and its parameters.

| Step | `Name` | Parameters | Description |
| --- | --- | --- | --- |
| `Step_H` | `DICOM_2_Nii` | `Brain_Size_mm` | Convert DICOM to NIfTI. `Brain_Size_mm` specifies the preset brain size (in mm) for FOV cropping. For human data, a value of `150` is typical. |
| `Step_T` | `Remove_First_n_Timepoints` | `Del_Tps` | Remove the first *n* time points. `Del_Tps` specifies the number of time points to delete. |
| `Step_A` | `Slice_Timing` | `Slice_Order`, `Ref_Slice` | Slice-timing correction. `Slice_Order` is the slice acquisition order; `Ref_Slice` is the reference slice. |
| `Step_R` | `Realign` | — | Realign functional images to correct for head motion. |
| `Step_Cb` | `ME_Combine` | — | Combine multi-echo data into a single echo time series. |
| `Step_C` | `Covs_Regressing` | `IsWholeBrain`, `IsCSF`, `IsWhiteMatter`, `IsHeadMotion`, `AddMean` | Regress out nuisance covariates. `IsWholeBrain`/`IsCSF`/`IsWhiteMatter` are flags (0/1) for regressing whole-brain, CSF, and WM signals, respectively. `IsHeadMotion` specifies the head-motion model order (e.g., `4` for 24-parameter model). `AddMean` adds the mean signal back after regression. |
| `Step_W` | `Normalize` | `IsDARTEL`, `ResVoxelSize` | Normalize to template space. `IsDARTEL` toggles DARTEL-based normalization (0/1). `ResVoxelSize` is the target voxel size (e.g., `[2, 2, 2]`). |
| `Step_S` | `Smooth` | `FWHM` | Spatial smoothing. `FWHM` is the Gaussian kernel size in mm (e.g., `[4, 4, 4]`). |
| `Step_D` | `Detrend` | — | Remove linear (and optionally higher-order) trends from the time series. |
| `Step_F` | `Filter` | `Band` | Temporal filtering. `Band` is the frequency band in Hz (e.g., `[0.01, 0.1]` for resting-state FC analysis). |

##### `Delete_Folder_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Delete_Folder_INFO` | Array | List of intermediate folders to delete after processing. Empty array `[]` means no folders are deleted. |

> The order of steps in `Process_INFO` determines the order in which they are executed. To disable a step, simply remove its entry from `Process_INFO`.

### 4.2 Example Configuration File (Resting-State Post-Processing)

The following is an example for resting-state post-processing, including ALFF, ReHo, seed-based FC, and FC matrix calculation (`Rest_PostPipe_all.json`):

```json
{
    "Process_Type": "Rest",
    "Import_INFO": {
        "Custom_BrainMask": ""
    },
    "Data_INFO": {
        "TR": [],
        "Snum": 1,
        "IS_Normalized": 1
    },
    "Process_INFO": {
        "FWHM": [],
        "Session_Weights": [],
        "ALFF_INFO": {
            "Calc_Fun_Name": "FunHTARWDC",
            "Band": [0.01, 0.1]
        },
        "ReHo_INFO": {
            "Calc_Fun_Name": "FunHTARWDC"
        },
        "FCseed_INFO": {
            "Calc_Fun_Name": "FunHTARWSDCF",
            "Seed_INFO": {
                "Default": 1,
                "Seed": [
                    [0, -52, 30, 6],
                    [-64, -20, 6, 6]
                ],
                "Mask": [],
                "TimeSeries": "",
                "Atlas": {
                    "File": "",
                    "ROI_Indices": [1, 2, 3, "113:115"]
                }
            }
        },
        "FCmatrix_INFO": {
            "Calc_Fun_Name": "FunHTARWSDCF",
            "Atlas": "D:\\HumanCortex_fMRI_Proc_V261014\\config\\template\\Schaefer2018_400Parcels_7Networks_order_FSLMNI152_2mm.nii"
        }
    }
}
```

#### 4.2.1 Parameter Description

##### `Process_Type`

| Field | Type | Description |
| --- | --- | --- |
| `Process_Type` | String | Set to `"Rest"` for resting-state post-processing. |

##### `Import_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Custom_BrainMask` | String | Path to a custom brain mask. Leave empty (`""`) to use the default brain mask. |

##### `Data_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `TR` | Array | Repetition time (in seconds). Empty array `[]` means it will be read from the data or left unspecified. |
| `Snum` | Integer | Number of sessions contained in the dataset. |
| `IS_Normalized` | Integer | Flag (0/1) indicating whether the input data has already been normalized to template space. `1` means the data is already normalized. |

##### `Process_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `FWHM` | Array | Full width at half maximum (in mm) for spatial smoothing. Empty array `[]` means no additional smoothing is applied. |
| `Session_Weights` | Array | Weights for combining multiple sessions. Empty array `[]` means sessions are combined with equal weights or handled automatically. |

##### `ALFF_INFO`

Parameters for amplitude of low-frequency fluctuation (ALFF) calculation.

| Field | Type | Description |
| --- | --- | --- |
| `Calc_Fun_Name` | String | Name of the folder containing the preprocessed functional data used for ALFF calculation (e.g., `"FunHTARWDC"`). |
| `Band` | Array | Frequency band (in Hz) used for ALFF/fALFF calculation (e.g., `[0.01, 0.1]`). |

##### `ReHo_INFO`

Parameters for regional homogeneity (ReHo) calculation.

| Field | Type | Description |
| --- | --- | --- |
| `Calc_Fun_Name` | String | Name of the folder containing the preprocessed functional data used for ReHo calculation (e.g., `"FunHTARWDC"`). |

##### `FCseed_INFO`

Parameters for seed-based functional connectivity (FC) calculation.

| Field | Type | Description |
| --- | --- | --- |
| `Calc_Fun_Name` | String | Name of the folder containing the preprocessed functional data used for seed-based FC calculation (e.g., `"FunHTARWSDCF"`). |
| `Seed_INFO.Default` | Integer | Flag (0/1) indicating whether to use the default seed set. `1` means the default seeds are used. |
| `Seed_INFO.Seed` | Array | List of seeds defined as `[x, y, z, radius]` in MNI space. Each seed is a 4-element array. |
| `Seed_INFO.Mask` | Array | List of mask files used to define seeds. Empty array `[]` means no mask-based seeds are used. |
| `Seed_INFO.TimeSeries` | String | Path to a pre-extracted time series file used as the seed. Empty string `""` means no external time series is used. |
| `Seed_INFO.Atlas.File` | String | Path to an atlas file used to define seeds. Empty string `""` means no atlas-based seeds are used. |
| `Seed_INFO.Atlas.ROI_Indices` | Array | ROI indices within the atlas used as seeds. Supports single indices (e.g., `1`, `2`, `3`) and range strings (e.g., `"113:115"`). |

When `Seed_INFO.Default` is set to `1`, the default seed set is used. It consists of six seeds<sup>[3-10]</sup> defined in MNI space, as shown below:

| ID | Net Name | ROI Center (MNI) | Radius (mm) |
| --- | --- | --- | --- |
| ROI 1 | Auditory | (55, -22, 9) | 4 |
| ROI 2 | Visual | (-8, -76, 10) | 4 |
| ROI 3 | Supplementary Motor | (0, -8, 58) | 4 |
| ROI 4 | Dorsal Attention | (-27, -52, 57) | 4 |
| ROI 5 | Primary Motor | (-35, -24, 51) | 4 |
| ROI 6 | Default Mode Network | (0, -53, 26) | 4 |

##### `FCmatrix_INFO`

Parameters for FC matrix calculation.

| Field | Type | Description |
| --- | --- | --- |
| `Calc_Fun_Name` | String | Name of the folder containing the preprocessed functional data used for FC matrix calculation (e.g., `"FunHTARWSDCF"`). |
| `Atlas` | String | Path to the atlas file used for defining ROIs in the FC matrix (e.g., `Schaefer2018_400Parcels_7Networks_order_FSLMNI152_2mm.nii`, data source: https://github.com/ThomasYeoLab/CBIG/tree/master/stable_projects/brain_parcellation/Schaefer2018_LocalGlobal/Parcellations/MNI). |

> The `Calc_Fun_Name` fields refer to the names of the preprocessed functional data folders produced by the preprocessing pipeline, so that the post-processing steps can locate the correct input data automatically.

### 4.3 Example Configuration File (Task Post-Processing)

The following is an example for task-based post-processing (`Task_PostPipe.json`):

```json
{
    "Process_Type": "Task",
    "Import_INFO": {
        "Calc_Fun_Name": "FunHTARWS",
        "IS_Use_Same_Onsets": 0
    },
    "Data_INFO": {
        "TR": [],
        "Snum": 1,
        "SliceTiming_INFO": {
            "IS_SliceTiminged": 1,
            "Slice_Order": [],
            "Ref_Slice": []
        },
        "IS_Normalized": 1
    },
    "Process_INFO": {
        "Design_Type": "Block",
        "Time_Unit": "scans",
        "Regressor_INFO": {
            "IS_Regress_RP": 1,
            "IS_Regress_Others": 0,
            "IS_Extract_ACC0_as_ErrorCond": 0
        },
        "Mask_INFO": {
            "IS_Use_Mask": 1,
            "Custom_Mask_File": ""
        },
        "Session_INFO": {
            "IS_Combine_Sessions": 0,
            "Session_Weights": []
        }
    }
}
```

#### 4.3.1 Parameter Description

##### `Process_Type`

| Field | Type | Description |
| --- | --- | --- |
| `Process_Type` | String | Set to `"Task"` for task-based post-processing. |

##### `Import_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Calc_Fun_Name` | String | Name of the folder containing the preprocessed functional data used for task analysis (e.g., `"FunHTARWS"`). |
| `IS_Use_Same_Onsets` | Integer | Flag (0/1) indicating whether to use the same onsets across sessions/runs. `0` means onsets are specified separately. |

##### `Data_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `TR` | Array | Repetition time (in seconds). Empty array `[]` means it will be read from the data or left unspecified. |
| `Snum` | Integer | Number of sessions contained in the dataset. |
| `SliceTiming_INFO.IS_SliceTiminged` | Integer | Flag (0/1) indicating whether slice-timing correction has already been applied. `1` means it has been applied. |
| `SliceTiming_INFO.Slice_Order` | Array | Slice acquisition order. Empty array `[]` means the default order is used. |
| `SliceTiming_INFO.Ref_Slice` | Array | Reference slice. Empty array `[]` means the default reference slice is used. |
| `IS_Normalized` | Integer | Flag (0/1) indicating whether the input data has already been normalized to template space. `1` means the data is already normalized. |

##### `Process_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Design_Type` | String | Type of experimental design. Example values: `"Block"`, `"Event"`. |
| `Time_Unit` | String | Unit of time used for onsets/durations. Example values: `"scans"`, `"seconds"`. |
| `Regressor_INFO.IS_Regress_RP` | Integer | Flag (0/1) indicating whether to include realignment parameters (head motion) as nuisance regressors. |
| `Regressor_INFO.IS_Regress_Others` | Integer | Flag (0/1) indicating whether to include additional nuisance regressors. |
| `Regressor_INFO.IS_Extract_ACC0_as_ErrorCond` | Integer | Flag (0/1) indicating whether to extract ACC=0 trials as an error condition. |
| `Mask_INFO.IS_Use_Mask` | Integer | Flag (0/1) indicating whether to apply a mask during analysis. |
| `Mask_INFO.Custom_Mask_File` | String | Path to a custom mask file. Empty string `""` means the default mask is used. |
| `Session_INFO.IS_Combine_Sessions` | Integer | Flag (0/1) indicating whether to combine multiple sessions. |
| `Session_INFO.Session_Weights` | Array | Weights for combining sessions. Empty array `[]` means equal weights or automatic handling. |

### 4.4 Example Configuration File (Group-Level Post-Processing)

The following is an example for group-level statistics (`Group_PostPipe.json`):

```json
{
    "Process_Type": "Group",
    "Import_INFO": {
        "Stats_Results_Name": "Results_Rest",
        "GroupInfo_File": "C:\\Users\\1996b\\Desktop\\ExampleData\\Task\\GroupInfo_TwoSampleT.xlsx",
        "Custom_Mask_File": ""
    },
    "Data_INFO": {
        "Snum": 1
    },
    "Process_INFO": {
        "Stats_Type": "TwoSample_T_Test",
        "Stats_Indices": ["ALFF", "ReHo", "FCseed_*"]
    }
}
```

#### 4.4.1 Parameter Description

##### `Process_Type`

| Field | Type | Description |
| --- | --- | --- |
| `Process_Type` | String | Set to `"Group"` for group-level statistics. |

##### `Import_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Stats_Results_Name` | String | Name of the folder containing the individual-level statistical results to be used for group analysis (e.g., `"Results_Rest"`). |
| `GroupInfo_File` | String | Path to an Excel file containing group information (e.g., group labels, covariates) for group-level analysis. |
| `Custom_Mask_File` | String | Path to a custom mask file. Empty string `""` means the default mask is used. |

##### `Data_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Snum` | Integer | Number of sessions contained in the dataset. |

##### `Process_INFO`

| Field | Type | Description |
| --- | --- | --- |
| `Stats_Type` | String | Type of group-level statistical test. Example values: `"TwoSample_T_Test"`, `"OneSample_T_Test"`, `"ANOVA"`, `"Correlation"`. |
| `Stats_Indices` | Array | List of statistical indices (metrics) to be analyzed at the group level. Supports wildcard patterns (e.g., `"FCseed_*"` matches all seed-based FC results). |

## 5. Usage

1. Run `hc_fmri_proc` in the MATLAB command line.
2. Select the data path.
3. After confirmation, select the parameter configuration JSON file corresponding to the pipeline you want to run:
   - Preprocessing: e.g., `1echo_Rest_preprocess_FC.json`
   - Resting-state post-processing: e.g., `Rest_PostPipe_all.json`
   - Task post-processing: e.g., `Task_PostPipe.json`
   - Group-level statistics: e.g., `Group_PostPipe.json`
4. The processing will then begin.

---

> **Note:** Before running the code, add the NIfTI_20140122 and SPM12 toolboxes to the MATLAB working path. 

## 6. Output Data Structure

### 6.1 Preprocessing Output

After preprocessing, a set of intermediate/output folders is generated for each subject, such as `FunHTARWSDC` and `FunHTARWSDCF`. The internal structure of these folders is identical to that of `FunH` or `FunMEH`, and each subject folder contains only one NIfTI file and one JSON file.

```text
ProjectRoot/
├── FunHTARWSDC/                  % Preprocessed functional data (after Covs_Regressing and before filtering)
│   ├── Subject001/
│   │   ├── S1/                   % Session level (if applicable)
│   │   │   ├── Sub_1.nii         % Preprocessed NIfTI data
│   │   │   └── Sub_1.json        % Associated parameter/metadata file
│   │   └── ...
│   ├── Subject002/
│   │   ├── S1/
│   │   │   ├── Sub_1.nii
│   │   │   └── Sub_1.json
│   │   └── ...
│   └── ...
│
├── FunHTARWSDCF/                 % Preprocessed functional data (after filtering)
│   ├── Subject001/
│   │   ├── S1/
│   │   │   ├── Sub_1.nii
│   │   │   └── Sub_1.json
│   │   └── ...
│   ├── Subject002/
│   │   ├── S1/
│   │   │   ├── Sub_1.nii
│   │   │   └── Sub_1.json
│   │   └── ...
│   └── ...
```

**Notes:**

| Folder / File | Description |
| --- | --- |
| `FunHTARWSDC/` | Preprocessed functional data without filtering (e.g., after dcm2nii, remove time points, slice timing, head-motion correction, registration, normalization, nuisance regression, detrending) |
| `FunHTARWSDCF/` | Preprocessed functional data with filtering (e.g., band-pass filtering applied after `FunHTARWSDC`) |
| `Sub_*.nii` | Preprocessed functional image for the corresponding subject; the file name is not fixed to the subject's ID |
| `Sub_*.json` | Parameter/metadata file associated with the preprocessing steps |

> The exact folder name suffixes (e.g., `HTARWSDC`, `HTARWSDCF`) reflect the preprocessing steps applied, and follow the same naming convention as `FunH` / `FunMEH`.

### 6.2 Resting-State Post-Processing Output

After resting-state post-processing, a `Results_Rest` folder is created, containing metric-specific subfolders for each subject. The structure is organized as follows:

```text
ProjectRoot/
└── Results_Rest/
    ├── ALFF/
    │   ├── mALFF/
    │   │   └── Sub_1.nii
    │   ├── rALFF/
    │   │   └── Sub_1.nii
    │   └── zALFF/
    │       └── Sub_1.nii
    │
    ├── fALFF/
    │   ├── mfALFF/
    │   │   └── Sub_1.nii
    │   ├── rfALFF/
    │   │   └── Sub_1.nii
    │   └── zfALFF/
    │       └── Sub_1.nii
    │
    ├── ReHo/
    │   ├── mReHo/
    │   │   └── Sub_1.nii
    │   ├── rReHo/
    │   │   └── Sub_1.nii
    │   └── zReHo/
    │       └── Sub_1.nii
    │
    ├── FCmatrix/
    │   ├── FCmatrix_Excel/
    │   │   ├── Sub_1.mat
    │   │   ├── Sub_1.tif
    │   │   └── Sub_1.xlsx
    │   ├── TimeSeries_Excel/
    │   │   └── Sub_1.xlsx
    │   └── zFCmatrix_Excel/
    │       ├── Sub_1.mat
    │       ├── Sub_1.tif
    │       └── Sub_1.xlsx
    │
    ├── FCseed_Default/
    │   ├── rDefault_ROI1/ ~ rDefault_ROI6/
    │   │   └── Sub_1.nii
    │   └── zDefault_ROI1/ ~ zDefault_ROI6/
    │       └── Sub_1.nii
    │
    ├── FCseed_Seed/
    │   ├── rSeed_ROI1/ ~ rSeed_ROI2/
    │   │   └── Sub_1.nii
    │   └── zSeed_ROI1/ ~ zSeed_ROI2/
    │       └── Sub_1.nii
    │
    ├── sALFF/
    │   ├── smALFF/
    │   │   └── Sub_1.nii
    │   ├── srALFF/
    │   │   └── Sub_1.nii
    │   └── szALFF/
    │       └── Sub_1.nii
    │
    ├── sfALFF/
    │   ├── smfALFF/
    │   │   └── Sub_1.nii
    │   ├── srfALFF/
    │   │   └── Sub_1.nii
    │   └── szfALFF/
    │       └── Sub_1.nii
    │
    ├── sReHo/
    │   ├── smReHo/
    │   │   └── Sub_1.nii
    │   ├── srReHo/
    │   │   └── Sub_1.nii
    │   └── szReHo/
    │       └── Sub_1.nii
    │
    ├── sFCseed_Default/
    │   ├── srDefault_ROI1/ ~ srDefault_ROI6/
    │   │   └── Sub_1.nii
    │   └── szDefault_ROI1/ ~ szDefault_ROI6/
    │       └── Sub_1.nii
    │
    └── sFCseed_Seed/
        ├── srSeed_ROI1/ ~ srSeed_ROI2/
        │   └── Sub_1.nii
        └── szSeed_ROI1/ ~ szSeed_ROI2/
            └── Sub_1.nii
```

**Notes:**

| Folder / File | Description |
| --- | --- |
| `ALFF/` | Amplitude of low-frequency fluctuation results, including `mALFF`, `rALFF`, and `zALFF` subfolders |
| `fALFF/` | Fractional ALFF results, including `mfALFF`, `rfALFF`, and `zfALFF` subfolders |
| `ReHo/` | Regional homogeneity results, including `mReHo`, `rReHo`, and `zReHo` subfolders |
| `FCmatrix/` | Functional connectivity matrix results, including raw and z-scored matrices, time series, and their Excel/TIFF/MAT exports |
| `FCseed_Default/` | Seed-based FC results using the six default seeds, including raw (`rDefault_ROI*`) and z-scored (`zDefault_ROI*`) subfolders |
| `FCseed_Seed/` | Seed-based FC results using user-defined seeds (`Seed_INFO.Seed`), including raw (`rSeed_ROI*`) and z-scored (`zSeed_ROI*`) subfolders |
| `sALFF/` | Smoothed ALFF results, including `smALFF`, `srALFF`, and `szALFF` subfolders |
| `sfALFF/` | Smoothed fALFF results, including `smfALFF`, `srfALFF`, and `szfALFF` subfolders |
| `sReHo/` | Smoothed ReHo results, including `smReHo`, `srReHo`, and `szReHo` subfolders |
| `sFCseed_Default/` | Smoothed seed-based FC results using default seeds, including raw (`srDefault_ROI*`) and z-scored (`szDefault_ROI*`) subfolders |
| `sFCseed_Seed/` | Smoothed seed-based FC results using user-defined seeds, including raw (`srSeed_ROI*`) and z-scored (`szSeed_ROI*`) subfolders |
| `Sub_*.nii` | Metric map for the corresponding subject |
| `Sub_*.mat` | MATLAB data file for the corresponding subject (e.g., FC matrix) |
| `Sub_*.tif` | TIFF visualization of the corresponding matrix |
| `Sub_*.xlsx` | Excel export of the corresponding matrix or time series |

> The prefix `s` in folders such as `sALFF`, `sfALFF`, `sReHo`, `sFCseed_Default`, and `sFCseed_Seed` denotes results computed on **smoothed** data.
>
> The prefixes `m` / `r` / `z` denote **mean**, **raw**, and **z-scored** variants of the corresponding metric.

## 7. DICOM Sort and Organization

### 7.1 Introduction

The DICOM Sort toolkit is a collection of **MATLAB scripts for organizing and archiving UIH (United Imaging Healthcare) MRI data**. It progressively processes raw DICOM data (which may contain compressed archives, overly long filenames, non-standard directory structures, etc.) into the standard UIH archive format, and then, according to user-defined rules, converts the data into the **HC_fMRI_Proc format** (FunRaw / T1Raw / T2Raw / FieldMap / dMRIRaw, etc.) required by the preprocessing pipeline described above.

The entire workflow consists of three main steps:

1. **`ds_achieve`** — Decompress, rename, and organize data into the standard UIH archive format (**only required for non-standard format data**)
2. **`ds_excel`** — Scan the archive directory and generate `Sorted_Info.xlsx` for manual editing
3. **`ds_sort`** — Read the edited Excel and sort the data into HC_fMRI_Proc format according to user markings

### 7.2 Directory Structure Convention

The **standard UIH archive format** has the following hierarchy:

```text
<SortedPath>/
├── <AcquisitionDate>/                       % e.g. 20240513
│   ├── <PatientID><Name><Time>/             % e.g. PID162310-xxxxxxx_xxx_121738
│   │   ├── <SeriesDescription>_<SeriesNumber>/   % e.g. epi_bold_3mmiso_Task1_Ref_501
│   │   │   ├── 1.dcm
│   │   │   ├── 2.dcm
│   │   │   └── ...
```

The **HC_fMRI_Proc format** directory structure after sorting:

```text
<SortedPath>_Sorted/
├── <Custom_DataFolder>/              % e.g., REST, AUD, Task1
│   ├── FunRaw/    Sub_1/ [Sessions/] [Echoes/]
│   ├── T1Raw/     Sub_1/
│   ├── T2Raw/     Sub_1/
│   ├── FieldMap/  Sub_1/ PhaseDiffRaw | Magnitude1Raw | Magnitude2Raw
│   └── dMRIRaw/   Sub_1/ (dMRIRaw | dMRIRaw_b0)
└── ...
```

> The `Custom_DataFolder` column determines the top-level output directory name (e.g., `REST`, `Task1`, etc.).

### 7.3 Workflow

#### 7.3.1 Step 0 (Optional): Organize Non-Standard Format Data — `ds_achieve`

**Applicable scenarios**: Raw data is in compressed archives (`.zip` / `.tar`), contains overly long filenames (> 50 characters), or has a non-standard directory structure.

```matlab
>> ds_achieve
```

> If your data is already in the standard UIH archive format, you can skip this step.

#### 7.3.2 Step 1: Generate `Sorted_Info.xlsx` — `ds_excel`

```matlab
>> ds_excel
```

Steps:

1. A directory selection dialog appears; select the UIH archive folder (the `*_Achieved` directory produced by `ds_achieve`).
2. Generates `Sorted_Info.xlsx` in the archive root directory, with formatting applied (alternating gray/white rows, bold headers, column widths set).

Excel column description:

| Column | Field | Description |
| --- | --- | --- |
| A | `SrcDir` | Archive root directory (cell B1) |
| — | `SrcDate` | Acquisition date |
| — | `SrcSubject` | Patient directory name |
| — | `SrcSequence` | Series directory name |
| — | `DcmNum` | Number of DICOM files |
| — | `MRI_Mode` | `fMRI` / `fMRI_me` / `T1w` / `T2w` / `FieldMap` / `dMRI` |
| — | `Custom_DataFolder` | **[Edit required]** Top-level output directory name (e.g., `REST`, `Task1`) |
| — | `Custom_Mode` | Output mode (`FunRaw`, `T1Raw`, `PhaseDiffRaw`, etc., auto-filled) |
| — | `Custom_SubjID` | Subject ID (`Sub_1`, `Sub_2`, ... auto-numbered) |
| — | `Custom_Session` | **[Editable]** Session name (leave empty for no Session layer) |
| — | `Custom_Echo` | Multi-echo fMRI echo number (`Echo1`, `Echo2`, auto-filled) |

#### 7.3.3 Step 2: Manually Edit the Excel

In `Sorted_Info.xlsx`, manually edit the columns according to your experimental design. The rules for each column are listed below.

##### Column Editing Rules

- **`Custom_DataFolder`** — Top-level output directory name.
  - Enter a **custom project name** (e.g., `REST`, `AUD`, `Task1`) to keep the sequence and place it under that folder.
  - Enter **`USE_IT`** for `T1w` / `FieldMap` that should be **shared across all fMRI tasks** for the same subject.
  - Leave **empty** to **discard** the sequence.

- **`Custom_Mode`** — Predefined output mode name.
  - Auto-filled by the script (e.g., `FunRaw`, `T1Raw`, `PhaseDiffRaw`, `Magnitude1Raw`, `Magnitude2Raw`, `dMRIRaw`).
  - **Usually no need to modify.**

- **`Custom_SubjID`** — Subject ID.
  - Auto-numbered as `Sub_1`, `Sub_2`, ... by the script.
  - Can be manually adjusted if a specific subject numbering is desired.

- **`Custom_Session`** — Session name.
  - Leave **empty** if the subject has no session layer.
  - Enter a session label (e.g., `S1`, `S2`) if the subject contains multiple sessions.

- **`Custom_Echo`** — Multi-echo fMRI echo number.
  - Auto-filled as `Echo1`, `Echo2`, ... for `fMRI_me` sequences.
  - Leave **empty** for single-echo fMRI.
  - Must be consistent with the downstream pipeline conventions.

##### Example

This example shows a single subject with a `T1w`, a `FieldMap`, a single-echo resting-state run (`REST`), a discarded task run, and a multi-echo, multi-session task run (`AUD`). `T1w` and `FieldMap` are marked `USE_IT` so they can be shared across tasks.

| SrcDate | SrcSubject | SrcSequence | DcmNum | MRI_Mode | Custom_DataFolder | Custom_Mode | Custom_SubjID | Custom_Session | Custom_Echo |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | T1w_grefsp_iso0.7mm_NIF_401 | 256 | T1w | USE_IT | T1Raw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | REST_b0fieldmap_tra_B0Map_603 | 68 | FieldMap | REST | PhaseDiffRaw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | REST_b0fieldmap_tra_Echo1_601 | 68 | FieldMap | REST | Magnitude1Raw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | REST_b0fieldmap_tra_Echo2_602 | 68 | FieldMap | REST | Magnitude2Raw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | REST_TR1s_iso2mm_tra_SaveBySlc_702 | 500 | fMRI | REST | FunRaw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | TASK_Aud_iso1.6mm_tra_-20_SaveBySlc_2002 | 5 | fMRI | | FunRaw | Sub_1 | | |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo1_801 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S1 | Echo1 |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo2_802 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S1 | Echo2 |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo3_803 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S1 | Echo3 |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo1_901 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S2 | Echo1 |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo2_902 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S2 | Echo2 |
| 20250101 | PID-20250101-120000-0001_ABC_120500 | AUD_me_bold_iso2mm_tra_Echo3_903 | 300 | fMRI_me | AUD | FunRaw | Sub_1 | S2 | Echo3 |

Notes on the example:

- The `TASK_Aud_iso1.6mm_tra_-20_SaveBySlc_2002` row has an **empty** `Custom_DataFolder`, so it is discarded during sorting.
- The `T1w` row is marked `USE_IT`, so it can be reused by both `REST` and `AUD`.
- The `AUD` rows are multi-echo and multi-session: `Custom_Session` = `S1` / `S2`, and `Custom_Echo` = `Echo1` / `Echo2` / `Echo3`.

> The `USE_IT` rows are shared across project folders, so `T1Raw` and `FieldMap` appear under both `REST/` and `AUD/` for the same subject.

#### 7.3.4 Step 3: Run the Sort — `ds_sort`

```matlab
>> ds_sort
```
Steps:
1. A file selection dialog appears; select the edited `Sorted_Info.xlsx`.

2. Generating directory structure after sorting would be:

```text
<SortedPath>_Sorted/
├── REST/
│   ├── FunRaw/
│   │   └── Sub_1/
│   ├── T1Raw/
│   │   └── Sub_1/
│   └── FieldMap/
│       └── Sub_1/
│           ├── PhaseDiffRaw/
│           ├── Magnitude1Raw/
│           └── Magnitude2Raw/
│
└── AUD/
    ├── FunRaw/
    │   └── Sub_1/
    │       ├── S1/
    │       │   ├── Echo1/
    │       │   ├── Echo2/
    │       │   └── Echo3/
    │       └── S2/
    │           ├── Echo1/
    │           ├── Echo2/
    │           └── Echo3/
    ├── T1Raw/
    │   └── Sub_1/
    └── FieldMap/
        └── Sub_1/
            ├── PhaseDiffRaw/
            ├── Magnitude1Raw/
            └── Magnitude2Raw/
```

## References

> [1] Yan, Chao-Gan et al. "DPABI: Data Processing & Analysis for (Resting-State) Brain Imaging." Neuroinformatics vol. 14,3 (2016): 339-51. doi:10.1007/s12021-016-9299-4

> [2] Jia, Xi-Ze et al. "RESTplus: an improved toolkit for resting-state functional magnetic resonance imaging data processing." Science bulletin vol. 64,14 (2019): 953-954. doi:10.1016/j.scib.2019.05.008

> [3] Zang YF, et al. Altered baseline brain activity in children with ADHD revealed by resting-state functional MRI. Brain Dev. 2007.

> [4] Zang Y, et al. Regional homogeneity approach to fMRI data analysis. Neuroimage. 2004.

> [5] Van Dijk KR, et al. The influence of head motion on intrinsic functional connectivity MRI. Neuroimage. 2012.

> [6] Wang J, et al. Alternations of functional connectivity in amblyopia patients: A resting-state fMRI study. Progress in Biomedical Optics and Imaging - Proceedings of SPIE. 2014.

> [7] Schmidt SA, et al. Default mode, dorsal attention and auditory resting state networks exhibit differential functional connectivity in tinnitus and hearing loss. PLoS One. 2013.

> [8] Qiu TM, et al. Sensorimotor cortical changes assessed with resting-state fMRI following total brachial plexus root avulsion. J Neurol Neurosurg Psychiatry. 2014.

> [9] Gao W, et al. The synchronization within and interaction between the default and dorsal attention networks in early infancy. Cereb Cortex. 2013.

> [10] Koenig KA, et al. Functional connectivity of primary motor cortex is dependent on genetic burden in prodromal Huntington disease. Brain Connect. 2014.