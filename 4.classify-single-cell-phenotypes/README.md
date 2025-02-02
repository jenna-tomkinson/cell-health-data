# 4. Classify Single Cell Phenotypes

In this module, we present our pipeline for classifying features.

### Feature Classification

We use the final and shuffled baseline logistic regression models trained in [phenotypic_profiling](https://github.com/WayScience/phenotypic_profiling_model) to classify nucleus features from Cell Health Data.
The final models have the highest average cross-validated F1 score while the shuffled baseline models were trained on feature values shuffled independent of their feature label (and thus serve as a baseline for training on features with no correlation).

The version of the classification models downloaded from `phenotypic_profiling` can be specified by the hash corresponding to a commit.
The current hash being used is `44e2741058c4d38edc137dc2caf5ea1f94b02410` which corresponds to [phenotypic_profiling/44e2741](https://github.com/WayScience/phenotypic_profiling_model/tree/44e2741058c4d38edc137dc2caf5ea1f94b02410). The `gh_hash` variable can be set in [classify-single-cell-phenotypes.ipynb](classify-single-cell-phenotypes.ipynb) to change which version of `phenotypic_profiling` is being accessed.

The specified classification models will be downloaded to [phenotypic_profiling_models/](phenotypic_profiling_models) and loaded from these files.

## Step 1: Setup Feature Classification Environment

### Step 1a: Create Feature Classification Environment

```sh
# Run this command to create the conda environment for classifying features
conda env create -f 4.classification-env.yml
```

### Step 1b: Activate Classification Environment

```sh
# Run this command to activate the conda environment for classifying features
conda activate 4.classify-single-cell-phenotypes
```

## Step 2: Define Folder Paths

Inside the notebook [classify-single-cell-phenotypes.ipynb](classify-single-cell-phenotypes.ipynb), the variable `normlized_plates_path` needs to be changed the reflect the paths of the normalized features from [3.preprocess-features](3.preprocess-features).
We used an external harddrive and therefore needed to use specific paths.

## Step 3: Classify Cell Health Features

```sh
# Run this script to classify Cell Health features
bash 4.classify-single-cell-phenotypes.sh
```
