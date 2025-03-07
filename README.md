# Feature-Splatting with Fientuned CLIP
Fork from https://github.com/vuer-ai/feature-splatting

Using a Finetuned CLIP ViT-16/B Model to compare to LeRF, a Mask R-CNN Model and a combination out of ResNet50 and SAM.

Prepared Data can be found [here](https://bwsyncandshare.kit.edu/s/qTZ3NamgqGTPxk4).

## Include CLIP-Model
The CLIP-Model (which can be finetuned with the [finetune an eval repository](https://github.com/LksWllmnn/finetune_eval_th) or a pre-finetuned Model [from here](https://bwsyncandshare.kit.edu/s/WKsJY3EdDQcX7kK)) can be included in line 11 in the script [clip_text_encoder.py](./feature_splatting/utils/clip_text_encoder.py) and line 73 in script [feature_extractor.py](./feature_splatting/feature_extractor.py).

## Prepare Model
The data can be prepared with this command:
```
ns-process-data --data [Path to Scen images] --output-dir [path where to put the nerfstudio transform.json] --skip-colmap --colmap-model-path [path to the Colmap infos] --skip-image-processing
```

## Train Model
To save the a cache for the scene, save the path in line 95 in script [feature_splatting_datagr.py](./feature_splatting/feature_splatting_datamgr.py)
The Model can be trained with the following commands:
```
ns-train lerf-lite --data [path to the nerfstudio transform.json] --pipeline.model.camera-optimizer.mode off nerfstudio-data --orientation-method none --center-method none
```

## Render the results
To render the results with a CLIP-Poitiv and with a specific Renderer you have to change line 115 in the script [model.py](./lerf/model.py) and use the following command:
```
ns-render camera-path --load-config [path to the trained configuration json-File of the NeRF] --camera-path-filename [Path to the created camerapath.json] --rendered-output-names my_output --output-format images --output-path [path where the images should be saved]
```

To create just the masks use the ```my_output``` --rendered-output-names. Use the camera-path "2024-12-29-08-10-12.json" from the lerf-lite-data.


## Use created Scenes from thesis
To use the created Scenes download the lef-lite-data from [here](https://bwsyncandshare.kit.edu/s/qTZ3NamgqGTPxk4). 
Make sure that the folder has been saved under this path: ```F:\Study\Master\Thesis\data\perception\usefull_data``` or change the config.yml files to customize the project dates for nerfstudio.

The Big-Surround finetuned Scene can be used with this command:
```
ns-viewer --load-config "F:\Studium\Master\Thesis\data\perception\usefull_data\lerf-lite-data\outputs\lerf-lite-data\feature-splatting\2025-01-29_205255\config.yml"
```

The Scene finetuned Scene can be used with this command:

```
ns-viewer --load-config "F:\Studium\Master\Thesis\data\perception\usefull_data\lerf-lite-data\outputs\lerf-lite-data\feature-splatting\2025-01-29_215613\config.yml"
```

The Surround finetuned Scene can be used with this command:
```
ns-viewer --load-config "F:\Studium\Master\Thesis\data\perception\usefull_data\lerf-lite-data\outputs\lerf-lite-data\feature-splatting\2025-01-29_221905\config.yml"
```

Make shure to save the fitting finetuned CLIP Model for the specific Scene and the cache.


# feature-splatting-ns

Official Nerfstudio implementation of Feature Splatting.

**Note:** Nerfstudio version is designed to be easy-to-use and efficient, which is done via several
tradeoffs than the original feature splatting paper, such as replacing SAM with MobileSAMV2 and
using simple bbox to select Gaussians for editing. We recommend using this repo to check the quality
of features. To reproduce full physics effects and examples on the website, please check out our
[original codebase based on INRIA 3DGS](https://github.com/vuer-ai/feature-splatting-inria).

## Instructions

Follow the [NerfStudio instllation instructions](https://docs.nerf.studio/quickstart/installation.html) to install a conda environment. For convenience,
here are the commands I run to install nerfstudio on two machines.

```bash
# Create an isolated conda environment
conda create --name feature_splatting_ns -y python=3.8
conda activate feature_splatting_ns

# Install necessary NS dependencies
pip install torch==2.1.2+cu118 torchvision==0.16.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118
conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit
pip install ninja git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch

# Insatll nerfstudio
pip install nerfstudio
```

As of this version (0.0.3), we use the gsplat kernel, which comes with NS installation. If you just want to try out feature splatting,
you can run,

```bash
pip install git+https://github.com/vuer-ai/feature-splatting
```

or, for dev purpose, run,

```bash
# Clone and cd to this repository
pip install -e .
```

After this is done, you can train feature-splatting on any nerfstudio-format datasets. An example command is given here,

```bash
ns-download-data nerfstudio --capture-name=poster
ns-train feature-splatting --data data/nerfstudio/poster
```

Specifically, check out various custom outputs defined by nerfstudio under `Output Type`. The `consistent_latent_pca` is used to
project high-dimensional features to low dimensions without flickering effects. After any text is supplied to `Positive Text Queries`,
a new output, `similarity`, will show up in the `Output Type` dropdown menu, which visualizes heatmap response to the language queries.

**NOTE:** Please **PAUSE TRAINING** before using any editing utility. Otherwise it seems to lead to race conditions. Unfortunately fix of this
issue seems to require modifying the core component of NerfStudio, which can not be gracefully implemented as a part of the extension plugin.

### TODOs

- Remove the simple bbox selection and implement better segmentation
- Support more feature extractors
- Improve the object segmentation workflow. Sometimes it causes unexpected error that only prints out in the terminal
- Add estimated gravity / ground plane estimation
- Improve thread safety that seems to lead to race condition when editing / training happen together.
