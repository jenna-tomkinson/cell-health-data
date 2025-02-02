# Segment Cell Painting Images

In this module, we present our pipeline for segmenting nuclei and cyoplasm from the Cell Health data.

### Segmentation

We use the CellPose segmentation algorithm to segment the nuclei from the Hoechst channel of each Cell Painting image.
CellPose was first introduced in [Stringer, C., Wang, T., Michaelos, M. et al., 2020](https://doi.org/10.1038/s41592-020-01018-x) and we use the [python implementation](https://github.com/mouseland/cellpose).

Stringer et al. trained the CellPose segmentation models on a diverse set of cell images and is therefore a good selection for our use case.
The CellPose python implementation was particularly useful for building reproducible pipelines.

### Nuclei Segmentation

After manually experimenting with CellPose on about 10 Cell Health nuclei images (various cell lines), we settled on the following parameters for CellPose nuclei segmentation:
- `model_type : "cyto"` This parameter forces CellPose to use the cytoplasm model, which we found segments nuclei in the Cell Health data substantially better than the nucleus model. 
More information about CellPose models can be found at https://cellpose.readthedocs.io/en/latest/models.html.
- `channels : [0,0]` This parameter forces the model to segment cells in grayscale (in the case of Cell Health data single channel images).
- `diameter : 80` This parameter indicates to the model that the average cell diameter is 80 pixels.
- `flow_threshold : 0` This paramenter decreases the maximum allowed error of the flows for each mask (default is `flow_threshold : 0.4`).
- `cellprob_threshold : 0` This parameter is used to determine ROIs (default is `cellprob_threshold : 0`).
- `remove_edge_masks : True` This parameter removes any masks from CellPose that are touching an edge of the image.

When manually experimenting with CellPose parameters to segment nuclei, we looked at nuclei shape/texture to determine how we believe it should be segmented.

For example, in the image below, the nucleus is a continuous shape with a continuous, flat texture (left).
Thus, these segmented nuclei should be segmented as one nucleus.
CellPose thinks the nucleus below is two nuclei when `diameter` is set to 0 (middle).
With our parameters, Cellpose correctly segments the nucleus (right).

![Nuclei Compiled](imgs/nuc_compiled.png "Nuclei Compiled")

### Cytoplasm Segmentation

After manually experimenting with CellPose on about 10 Cell Health cytoplasm images (various cell lines), we settled on the following parameters for CellPose cytoplasm segmentation:
- `model_type : "cyto"` This parameter forces CellPose to use the cytoplasm model for segmentation.
More information about CellPose models can be found at https://cellpose.readthedocs.io/en/latest/models.html.
- `channels : [1,3]` This parameter forces the model to segment cells using the nuclei and RNA channels of cell painting images. 
We overlay nuclei, ER, and RNA channels for CellPose with our `overlay_images()` function in [segment-cell-health-data.ipynb](segment-cell-health-data.ipynb).
ER channel is not used by CellPose during segmentation (CellPose can only use 2 channels for segmentation), but we found the overlayed ER channel useful to evaluate segmentation while manually experimenting with CellPose.

**Note:** The channel numbers 1,3 correspond to the CellPose colors red and blue respectively.
The `overlay()` function makes RNA the red channel of the image and DNA the blue channel of the image.
Thus, `channels : [1,3]` forces CellPose to segment the RNA channel using the DNA channel as its base.

- `diameter : 0` This parameter forces the CellPose model to estimate the diameter of cells being segmented.
- `flow_threshold : 0` This paramenter decreases the maximum allowed error of the flows for each mask (default is `flow_threshold : 0.4`).
- `cellprob_threshold : 0.4` This parameter is used to determine ROIs (default is `cellprob_threshold : 0`).
- `remove_edge_masks : True` This parameter removes any masks from CellPose that are touching an edge of the image.

More information about CellPose settings can be found at https://cellpose.readthedocs.io/en/latest/settings.html.

When manually experimenting with CellPose parameters to segment cytoplasm, we used DNA (blue), ER (green), and RNA (red) channels to inform our decision on how to segment cytoplasm.

For example, in the image below, the ER of the bottom cell stops abruptly below the top nucleus (left).
Thus, this should be segmented as two cells.
With default parameters for `flow_threshold` and `cellprob_threshold`, CellPose completely fails to segment the top cell (middle).
With our parameters, CellPose correctly segments both cells (right).

![Cytoplasm Compiled](imgs/cyto_compiled.png "Cytoplasm Compiled")

### Cell Identification

After finding the nuclei center coordinates and the cytoplasm outlines, we determine which nuclei and cytoplasms are related.
First, we assign a `Cell_ID` to every cytoplasm.
Then, we create a [matplotlib.path](https://matplotlib.org/stable/api/path_api.html) (polygon) from cytoplasm outlines.
If any nuclei center coordinates are within the cytoplasm polygon, they are assigned the same `Cell_ID` as the cytoplasm.
We discard any cytoplasm that does not have an associated nucleus.
This controls for segmentation errors (every cytoplasm must have at least one nucleus).

## Step 1: Setup Segmentation Environment

### Step 1a: Create Segmentation Environment

```sh
# Run this command to create the conda environment for Segmentation data
conda env create -f 1.segment-env.yml
```

### Step 1b: Activate Segmentation Environment

```sh
# Run this command to activate the conda environment for Segmentation data
conda activate 1.segment-data-cell-health
```

### Step 1c (Optional): Complete PyTorch GPU Setup

If you would like use PyTorch GPU when using CellPose, follow [these instructions](https://github.com/MouseLand/cellpose#gpu-version-cuda-on-windows-or-linux) to complete the PyTorch GPU setup.
We use PyTorch GPU while segmenting Cell Health data.

## Step 2: Define Data Paths

Inside the notebook [segment-cell-health-data.ipynb](segment-cell-health-data.ipynb), the variables`load_path` and `save_path` need to be changed to reflect the desired load/save locations.
We used an external harddrive and therefore needed to use specific paths.

## Step 3: Execute Training Data Segmentation

```bash
# Run this script to segment Cell Health data
bash 1.segment-data.sh
```

**Note:** With GPU enabled, our estimated run time was about **100 hours** to segment all Cell Health data.
The total data for all segmentation data are about **20 GB**.

Our runtime was heavily GPU dependent.
We use a NVIDIA GeForce RTX 3060 with the following specificaitons:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 515.48.07    Driver Version: 515.48.07    CUDA Version: 11.7     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:2D:00.0  On |                  N/A |
|  0%   53C    P8    22W / 170W |    404MiB / 12288MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
```