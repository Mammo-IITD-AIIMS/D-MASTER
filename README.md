
<div align="center">
<figure class="center-figure"> <img src="Images/architecture.png" width="100%"></figure>
</div>

<h1 align="left">
    D-MASTER: Mask Annealed Transformer for Unsupervised Domain Adaptation in Breast Cancer Detection from Mammograms
</h1>

<div align="left">

[![](https://img.shields.io/badge/website-stark-purple?style=plastic&logo=Google%20chrome)](https://stark.stanford.edu/)
[![](https://img.shields.io/badge/Dataset-yellow?style=plastic&logo=Hugging%20face)](https://huggingface.co/datasets/snap-stanford/stark)
[![](https://img.shields.io/badge/SKB_Explorer-online-yellow?style=plastic&logo=Hugging%20face)](https://stark.stanford.edu/skb_explorer.html)
[![](https://img.shields.io/badge/Arxiv-paper-red?style=plastic&logo=arxiv)](https://arxiv.org/abs/2404.13207)
[![](https://img.shields.io/badge/pip-stark--qa-brightgreen?style=plastic&logo=Python)](https://pypi.org/project/stark-qa/) 
[![](https://img.shields.io/badge/doc-online-blue?style=plastic&logo=Read%20the%20Docs)](https://stark.stanford.edu/docs/index.html)
[![](https://img.shields.io/badge/-Linkedin-blue?style=plastic&logo=Linkedin)](https://www.linkedin.com/posts/leskovec_reduce-llm-hallucinations-with-rag-over-textual-activity-7190745116339302401-da4n?utm_source=share&utm_medium=member_desktop) 
[![](https://img.shields.io/badge/-Twitter-cyan?style=plastic&logo=X)](https://twitter.com/ShirleyYXWu/status/1784970920383402433) 
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
</div>



## NEWS
 **[July 2024]** We publicly release source code and pre-trained D-MASTER model weights!
- **[Jun 2024]** D-MASTER is accepted in [MICCAI 2024](https://conferences.miccai.org/2024/en/) Congratulations to all the authors. See you all at [MICCAI 2024](https://conferences.miccai.org/2024/en/) under the Moroccan sun!
- **[June 2024]** We released an arxiv version.. See more details in our [updated arxiv]()! 
- **[June 2024]** We release [RSNA-BSD1K Dataset](),  a bounding box annotated subset of 1000 mammograms from the RSNA Breast Screening Dataset (referred to as RSNA-BSD1K) to support further research in BCDM!
- **[May 2024]** We release the D-MASTER benchmark.

## What is D-MASTER?
D-MASTER is a transformer-based Domain-invariant Mask Annealed Student Teacher Autoencoder Framework for cross-domain breast cancer detection from mammograms (BCDM). It integrates a novel mask-annealing technique and an adaptive confidence refinement module. Unlike traditional pretraining with Mask Autoencoders (MAEs) that leverage massive datasets before fine-tuning on smaller datasets, D-MASTER introduces a novel learnable masking technique for the MAE branch. This technique generates masks of varying complexities, which are then reconstructed by the DefDETR encoder and decoder. By applying this self-supervised task on target images, our approach enables the encoder to acquire domain-invariant features and improve target representations.


🔥 Check out our [website](https://dmaster-iitd.github.io/webpage/) for more overview!


## What is RSBA-BSD1K Data?

RSNA-BSD1K is a bounding box annotated subset of 1,000 mammograms from the RSNA Breast Screening Dataset, designed to support further research in breast cancer detection from mammograms (BCDM). The original RSNA dataset consists of 54,706 screening mammograms, containing 1,000 malignancies from 8,000 patients. From this, we curated RSNA-BSD1K, which includes 1,000 mammograms with 200 malignant cases, annotated at the bounding box level by two expert radiologists.

🔥 Check out our released  [Dataset](https://dmaster-iitd.github.io/webpage/) for more details!

# Access benchmark data


## 1. Installation

### 1.1 Requirements

- Linux, CUDA >= 11.1, GCC >= 8.4

- Python >= 3.8

- torch >= 1.10.1, torchvision >= 0.11.2

- Other requirements

  ```bash
  pip install -r requirements.txt
  ```

### 1.2 Compiling Deformable DETR CUDA operators

```bash
cd ./models/ops
sh ./make.sh
# unit test (should see all checking is True)
python test.py
```



## 2. Usage

### 2.1 Data preparation

We provide the 2 benchmarks in our paper: 

- city2foggy: cityscapes dataset is used as source domain, and foggy_cityscapes(0.02) is used as target domain.
- sim2city: sim10k dataset is used as source domain, and cityscapes which only record AP of cars is used as target domain.
- city2bdd: cityscapes dataset is used as source domain, and bdd100k-daytime is used as target domain.

You can download the raw data from the official websites: [cityscapes](https://www.cityscapes-dataset.com/downloads/),  [foggy_cityscapes](https://www.cityscapes-dataset.com/downloads/),  [sim10k](https://fcav.engin.umich.edu/projects/driving-in-the-matrix), [bdd100k](https://bdd-data.berkeley.edu/). We provide the annotations that are converted into coco style, download from [here](https://drive.google.com/file/d/1LB0wK9kO3eW8jpR2ZtponmYWe9x2KSiU/view?usp=sharing) and organize the datasets and annotations as following:

```bash
[data_root]
└─ cityscapes
	└─ annotations
		└─ cityscapes_train_cocostyle.json
		└─ cityscapes_train_caronly_cocostyle.json
		└─ cityscapes_val_cocostyle.json
		└─ cityscapes_val_caronly_cocostyle.json
	└─ leftImg8bit
		└─ train
		└─ val
└─ foggy_cityscapes
	└─ annotations
		└─ foggy_cityscapes_train_cocostyle.json
		└─ foggy_cityscapes_val_cocostyle.json
	└─ leftImg8bit_foggy
		└─ train
		└─ val
└─ sim10k
	└─ annotations
		└─ sim10k_train_cocostyle.json
		└─ sim10k_val_cocostyle.json
	└─ JPEGImages
└─ bdd10k
	└─ annotations
		└─ bdd100k_daytime_train_cocostyle.json
		└─ bdd100k_daytime_val_cocostyle.json
	└─ JPEGImages
```

To use additional datasets, you can edit [datasets/coco_style_dataset.py](https://github.com/JeremyZhao1998/MRT-release/blob/main/datasets/coco_style_dataset.py) and add key-value pairs to `CocoStyleDataset.img_dirs` and `CocoStyleDataset.anno_files` .

### 2.2 Training and evaluation

As has been discussed in implementation details in the paper, to save computation cost, our method is designed as a three-stage paradigm. We first perform `source_only` training which is trained standardly by labeled source domain. Then, we perform `cross_domain_mae` to train the model with MAE branch. Finally, we perform `teaching` which utilize a teacher-student framework with MAE branch and selective retraining.

For example, for `city2foggy` benchmark, first edit the files in `configs/def-detr-base/city2foggy/` to specify your own `DATA_ROOT` and `OUTPUT_DIR`, then run:

```bash
sh configs/def-detr-base/city2foggy/source_only.sh
sh configs/def-detr-base/city2foggy/cross_domain_mae.sh
sh configs/def-detr-base/city2foggy/teaching.sh
```

We use `tensorboard` to record the loss and results. Run the following command to see the curves during training: 

```bash
tensorboard --logdir=<YOUR/LOG/DIR>
```

To evaluate the trained model and get the predicted results, run:

```bash
sh configs/def-detr-base/city2foggy/evaluation.sh
```



## 3. Results and Model Parameters

We conduct all experiments with batch size 8 (for source_only stage, 8 labeled samples; for cross_domain_mae and MRT teaching stage, 8 labeled samples and 8 unlabeled samples), on 2 NVIDIA A100 GPUs.

**inhouse2inbreast**: Inhouse → INBreast

| backbone | encoder layers | decoder layers | training stage   | R@0.3 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 29.5  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) |
| resnet50 | 6              | 6              | cross_domain_mae | 35.8  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) |
| resnet50 | 6              | 6              | MRT teaching     | 51.2  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) |

**inhouse2rsna**: Inhouse → RSNA-BSD1K

| backbone | encoder layers | decoder layers | training stage   | R@0.3 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 53.2  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/18vKJqqzNil95JnI2Lvp0XJcpQHtU40AE?usp=sharing) |
| resnet50 | 6              | 6              | cross_domain_mae | 57.1  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/18vKJqqzNil95JnI2Lvp0XJcpQHtU40AE?usp=sharing) |
| resnet50 | 6              | 6              | MRT teaching     | 62.0  | [logs](https://drive.google.com/drive/folders/1VxlVdCAIRHGXkJXw0PyT9r_1HPSDhavP?usp=sharing) & [weights](https://drive.google.com/drive/folders/18vKJqqzNil95JnI2Lvp0XJcpQHtU40AE?usp=sharing) |

**ddsm2inhouse**: DDSM → Inhouse

| backbone | encoder layers | decoder layers | training stage   | R@0.3 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 29.6  | [logs](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) & [weights](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) |
| resnet50 | 6              | 6              | cross_domain_mae | 31.1  | [logs](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) & [weights](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) |
| resnet50 | 6              | 6              | MRT teaching     | 33.7  | [logs](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) & [weights](https://drive.google.com/drive/folders/1PmP6sENzjjJH0vkGsQYJwYcjwfvBNVJe?usp=sharing) |

**ddsm2inbreast**: DDSM → INBreeast

| backbone | encoder layers | decoder layers | training stage   | R@0.3 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 29.6  | [logs](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) & [weights](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) |
| resnet50 | 6              | 6              | cross_domain_mae | 31.1  | [logs](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) & [weights](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) |
| resnet50 | 6              | 6              | MRT teaching     | 33.7  | [logs](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) & [weights](https://drive.google.com/drive/folders/1nGzb7EHl8tspbCOX7VRfK_vgSHsD1XjD?usp=sharing) |


**city2foggy**: cityscapes → foggy cityscapes(0.02)

| backbone | encoder layers | decoder layers | training stage   | AP@50 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 29.5  | [logs](https://drive.google.com/file/d/1O-B-OXBf8clOSNMJLtJEPuNQvo5W2CuU/view?usp=drive_link) & [weights](https://drive.google.com/file/d/1J6PpDsKvWvTJthwctFuYV8kUEnGGTVUk/view?usp=drive_link) |
| resnet50 | 6              | 6              | cross_domain_mae | 35.8  | [logs](https://drive.google.com/file/d/1gUYJDX9eE5FIKWMbR_tK6leMnM5q06dj/view?usp=sharing) & [weights](https://drive.google.com/file/d/1X-STx26799Q2vAUle1QjXj_1gzwvZrRk/view?usp=drive_link) |
| resnet50 | 6              | 6              | MRT teaching     | 51.2  | [logs](https://drive.google.com/file/d/1YwLUo3t2KJ1pjENFAr5vECZlrRFWwKG2/view?usp=sharing) & [weights](https://drive.google.com/file/d/1BooqcIdzP97I3ax7JN6ULZWoZcvRKLlm/view?usp=sharing) |

**sim2city**: sim10k → cityscapes(car only)

| backbone | encoder layers | decoder layers | training stage   | AP@50 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 53.2  | [logs](https://drive.google.com/file/d/1qfdHLuUX8N3SRUTNmclf0Y3PJ-deOF4r/view?usp=sharing) & [weights](https://drive.google.com/file/d/1mkqKxrWannqJN1_tJdh76t7ZAGIzDsIs/view?usp=sharing) |
| resnet50 | 6              | 6              | cross_domain_mae | 57.1  | [logs](https://drive.google.com/file/d/1bDNux81HhHZhmuoABwU-N4ALZFjKQWHR/view?usp=drive_link) & [weights](https://drive.google.com/file/d/14cTFm8pM9DmN2UcV7NGaMJxOJVfOvANP/view?usp=sharing) |
| resnet50 | 6              | 6              | MRT teaching     | 62.0  | [logs](https://drive.google.com/file/d/1S_GiAb9Ujfndh6XHnBz6qmCawpEDY102/view?usp=sharing) & [weights](https://drive.google.com/file/d/1dsSuk24_jEq3k4DBpoPr4AH3mxL0DspP/view?usp=sharing) |

**city2bdd**: cityscapes → bdd100k(daytime)

| backbone | encoder layers | decoder layers | training stage   | AP@50 | logs & weights                                               |
| -------- | -------------- | -------------- | ---------------- | ----- | ------------------------------------------------------------ |
| resnet50 | 6              | 6              | source_only      | 29.6  | [logs](https://drive.google.com/file/d/1KIydqXkj0LIlDlHHDW4TfxIh3-rmaWQM/view?usp=drive_link) & [weights](https://drive.google.com/file/d/1IAzbKozA_Rq-2H-KzdcvGp3LGJrZ4J5G/view?usp=drive_link) |
| resnet50 | 6              | 6              | cross_domain_mae | 31.1  | [logs](https://drive.google.com/file/d/1gUYJDX9eE5FIKWMbR_tK6leMnM5q06dj/view?usp=drive_link) & [weights](https://drive.google.com/file/d/1X-STx26799Q2vAUle1QjXj_1gzwvZrRk/view?usp=drive_link) |
| resnet50 | 6              | 6              | MRT teaching     | 33.7  | [logs](https://drive.google.com/file/d/13jgRrsKVDap0O9rUiY-ZhZp-kL6di4EH/view?usp=sharing) & [weights](https://drive.google.com/file/d/1VRtNy_2bXdkpLr1h6v-ZusEuTR7hAu_v/view?usp=sharing) |



## 4. Citation

This repository is constructed and maintained by [Tajamul Ashraf](https://github.com/Tajamul21).

If you find our paper or project useful, please cite our work in the following BibTeX:

```
@inproceedings{zhao2023masked,
  title={Masked retraining teacher-student framework for domain adaptive object detection},
  author={Zhao, Zijing and Wei, Sitong and Chen, Qingchao and Li, Dehui and Yang, Yifan and Peng, Yuxin and Liu, Yang},
  booktitle={Proceedings of the IEEE/CVF International Conference on Computer Vision},
  pages={19039--19049},
  year={2023}
}
```

Thanks for your attention.
