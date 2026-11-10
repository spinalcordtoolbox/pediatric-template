# Pediatric Template
Repository for generating pediatric spinal cord templates by age groups. 

The goal for this project is to generate T1w and T2w templates for the following age groups:
- 10-12 years
- 13-15 years
- 16-17 years

Each age group is processed independently. For each group, T1w and T2w images are preprocessed and normalized into the same template space before generating the corresponding population-specific templates. 

## 1. Data

The data used to generate the templates are available on the NeuroPoly data server (lab members only): `data.neuro.polymtl.ca:datasets/philadelphia-pediatric`. 

Instructions for downloading the dataset: https://intranet.neuro.polymtl.ca/data/git-datasets.html 

## 2. Overview of the processing pipeline
The pediatric template generation process consists of the following steps:
1. Download the `philadelphia-pediatric` dataset
2. Install the required dependencies
3. Identify participants belonging to each age group and define the configuration files
4. Prepare and normalize T1w images for each age group
5. Prepare and normalize T2w images for each age group
6. Set up the environment on a Digital Research Alliance of Canada high-performance computing (HCP) cluster
7. Bring necessary data and scripts to your Alliance HPC scratch directory
8. Generate the T1w and T2w templates independently for each age group
9. Convert the final MINC templates to NIFTI format

All preprocessing steps should be performed in your command-line bash, from your `/PATH/TO/pediatric-template` directory.

Template generation is performed on a Digital Research Alliance of Canada HPC cluster. 

# Part I : Data preparation and processing

## 1. Download the `philadelphia-pediatric` dataset

> [!NOTE]
> The Philadelphia Pediatric dataset can be cloned into any directory. It does not need to be located inside the `pediatric-template` working directory. Make sure to update the paths in the configuration files accordingly.

Clone the dataset using git annex:

```bash
git clone git@data.neuro.polymtl.ca:datasets/philadelphia-pediatric
cd philadelphia-pediatric

# Download the data
git annex get .
```

## 2. Install dependencies
### 2.1 Install the Spinal Cord Toolbox (SCT)
Install the Spinal Cord Toolbox (SCT) by following the official installation instructions: https://spinalcordtoolbox.com/stable/user_section/installation.html

### 2.2 Install SCT's template generation pipeline
The preprocessing pipeline is based on the following repository (used to generate the PAM50 adult template): https://github.com/neuropoly/template

Clone the repository:

```bash
git clone git@github.com:neuropoly/template.git
cd template
```

## 3 Update the configuration files 

Before processing, participants must be separated into three age groups:
- 10-12 years
- 13-15 years
- 16-17 years

Six configuration files are provided in the `pediatric-template` repository (i.e., one for each age group and contrast): 
- `configuration_T1w_10-12.json`
- `configuration_T1w_13-15.json`
- `configuration_T1w_16-17.json`
- `configuration_T2w_10-12.json`
- `configuration_T2w_13-15.json`
- `configuration_T2w_16-17.json`

For each configuration file, the list of participants to include must be specified under the `include_list` parameter. Make sure that the participant lists correspond to the appropriate age group and are consistent between T1w and T2w contrasts. 

The `path_data` parameter should also be updated to point to the location of your cloned `philadelphia-pediatric` dataset. 

## 4. T1w preprocessing and normalization

> [!NOTE] Make sure to activate the SCT Python environment before proceeding with the subsequent preprocessing steps.

T1w preprocessing and normalization is performed independently for each age group.

For example, to process the 10-12 year age group:
```bash
cd template #path to your clone of https://github.com/neuropoly/template

python preprocess_normalize.py path/to/pediatric-template/configuration_T1w_10-12.json
```

> [!IMPORTANT]
> Once preprocessing is complete, rename the output folders to identify the corresponding age group. This prevents the outputs from being overwritten when processing the next age group. For example, you could rename the folders as follows: 
> ```bash
> mv template template_T1w_10-12
> mv sct_straighten_spinalcord sct_straighten_spinalcord_T1w_10-12
> ```

Repeat for the 13-15 age group: 
```bash
python preprocess_normalize.py path/to/pediatric-template/configuration_T1w_13-15.json

# Rename output folders to identify the corresponding age group:
mv template template_T1w_13-15
mv sct_straighten_spinalcord sct_straighten_spinalcord_T1w_13-15
```

And finally, the 16-17 age group:
```bash
python preprocess_normalize.py path/to/pediatric-template/configuration_T1w_16-17.json

# Rename output folders to identify the corresponding age group:
mv template template_T1w_16-17
mv sct_straighten_spinalcord sct_straighten_spinalcord_T1w_16-17
```

## 5. T2w preprocessing and normalization

Repeat the last step with the T2w data: 

- For the 10-12 age group:

```bash
python preprocess_normalize.py path/to/pediatric-template/configuration_T2w_10-12.json

# Rename output folders to identify the corresponding age group:
mv template template_T2w_10-12
mv sct_straighten_spinalcord sct_straighten_spinalcord_T2w_10-12
```

- For the 13-15 age group:
```bash
python preprocess_normalize.py path/to/pediatric-template/configuration_T2w_13-15.json

# Rename output folders to identify the corresponding age group:
mv template template_T2w_13-15
mv sct_straighten_spinalcord sct_straighten_spinalcord_T2w_13-15
```

- For the 16-17 age group:
```bash
python preprocess_normalize.py path/to/pediatric-template/configuration_T2w_16-17.json

# Rename output folders to identify the corresponding age group:
mv template template_T2w_16-17
mv sct_straighten_spinalcord sct_straighten_spinalcord_T2w_16-17
```

# Part II : Template generation

## 6. Set up the environment on a Digital Research Alliance of Canada HPC cluster.

### 6.1 Log in to the Digital Research Alliance of Canada HPC:
Refer to the NeuroPoly Lab manual for instructions to connect to the Digital Research Alliance of Canada HPC : https://intranet.neuro.polymtl.ca/computing-resources/compute-canada.html 

### 6.2 Create the working directory
**Navigate to your scratch directory** and create a folder for the pediatric templates project:
```bash
cd ~/scratch

mkdir -p pediatric_templates
cd pediatric_templates
```

### 6.3 Load the required modules
```bash
module load StdEnv/2020 gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
```

### 6.4 Install Python dependencies
Upgrade pip and install [SCOOP](https://pypi.org/project/scoop/): 
```bash
pip install --upgrade pip
pip install --user scoop
```

The following package versions were used to ensure compatibility with Python 3.8.0:
```bash
python -m pip install --user "aiosignal==1.3.1"
python -m pip install --user "frozenlist==1.3.3"
python -m pip install --user "virtualenv==20.16.3"
python -m pip install --user "filelock==3.12.2"
python -m pip install --user "click==8.1.7"
python -m pip install --user "ray==2.3.1+computecanada"
```

### 6.5 Download NIST MNI pipelines
Clone the NIST MNI pipeline repository:
```bash
git clone https://github.com/vfonov/nist_mni_pipelines.git
```

The repository should now be located at:

`/path/to/your/scratch/pediatric_templates/nist_mni_pipelines`

### 6.6 Configure PYTHONPATH
Python needs to know where to find the NIST MNI pipeline modules. Open your `.bashrc` file:
```bash
nano ~/.bashrc
```
Add the following lines at the bottom:

```bash
# NIST MNI pipelines
export PYTHONPATH="${PYTHONPATH}:/home/sstonge/scratch/pediatric_templates/nist_mni_pipelines"
export PYTHONPATH="${PYTHONPATH}:/home/sstonge/scratch/pediatric_templates/nist_mni_pipelines/ipl"
```

Save the file and exit nano:

- `Ctrl + O`, then `Enter` to save.
- `Ctrl + X` to exit.

Activate the changes:
```bash
source ~/.bashrc
```

### 6.7 Install minc2-simple
Install the Python interface to the MINC2 library directly from GitHub:
```bash
pip install "git+https://github.com/NIST-MNI/minc2-simple.git@develop_new_build#subdirectory=python"
```

## 7. Bring necessary data and scripts to the scratch directory

### 7.1 Add preprocessed and normalized data

Inside `PATH/TO/SCRATCH/DIRECTORY/pediatric_templates`, create the following folders to store the normalized files for each age group:
```bash
mkdir template_T1w_10-12 
mkdir template_T1w_13-15 
mkdir template_T1w_16-17
mkdir template_T2w_10-12 
mkdir template_T2w_13-15 
mkdir template_T2w_16-17
```

Next, transfer the straightened and normalized `.mnc` files generated during preprocessing (Steps 4 and 5) to the corresponding directories in your Alliance scratch directory.

The normalized images and template masks are located in the `philadelphia-pediatric/derivatives/template_*/` directories. 

For each age group and imaging modality, copy the following files:
- **Normalized MRI images:** All straightened and normalized subject images (`sub-*_rec-composed_T*w_straight_norm.mnc`).
- **Template mask:** The corresponding template_mask.mnc generated during preprocessing.
- **Subject list:** The corresponding subject.csv file containing the paths to the normalized .mnc images that will be used for template generation.

> [!NOTE]
> To copy the files to your scratch directory, we recommend using `scp` in your local computer terminal. For example, to copy the `.mnc` files for the T2w 10-12 age group:
>```
> scp /PATH/TO/philadelphia-pediatric/derivatives/template_T2w_10-12/sub-*_rec-composed_T2w_straight_norm.mnc USERNAME@CLUSTER.computecanada.ca:/PATH/TO/scratch/pediatric_templates/template_T2w_10-12/ 
> ```
> Replace:
> - `/PATH/TO/philadelphia-pediatric/` with the location of the dataset on your local computer.
> - `USERNAME` with your Alliance username.
> - `/PATH/TO/scratch/` with the path to your scratch directory on the cluster.

Each age group and imaging modality must have its own directory containing the appropriate images, template mask, and CSV file.

Your scratch directory should look like this: 
```bash
pediatric_templates/
│
├── nist_mni_pipelines/
│
├── template_T1w_10-12/
│   ├── sub-*_rec-composed_T1w_straight_norm.mnc # all participant files corresponding to this template
│   ├── template_mask.mnc # template mask corresponding to this template
│   └── subject.csv  # csv file containing the paths to the .mnc fles
│
├── template_T1w_13-15/
│   ├── sub-*_rec-composed_T1w_straight_norm.mnc
│   ├── template_mask.mnc
│   └── subject.csv 
│
├── template_T1w_16-17/
│   ├── sub-*_rec-composed_T1w_straight_norm.mnc
│   ├── template_mask.mnc
│   └── subject.csv 
│
├── template_T2w_10-12/
│   ├── sub-*_rec-composed_T2w_straight_norm.mnc
│   ├── template_mask.mnc
│   └── subject.csv 
│
├── template_T2w_13-15/
│   ├── sub-*_rec-composed_T2w_straight_norm.mnc
│   ├── template_mask.mnc
│   └── subject.csv 
│
└── template_T2w_16-17/
    ├── sub-*_rec-composed_T2w_straight_norm.mnc
    ├── template_mask.mnc
    └── subject.csv 
```

### 7.2 Update paths inside `subject.csv` files
Each template directory in your Alliance scratch directory should now contains a `subject.csv` file listing the normalized MRI images to be used for template generation. 

Since the images have been transferred from your `philadelphia-pediatric/derivatives` folder, the paths listed in each `subject.csv` file must be updated to point to the new file locations in your Alliance scratch directory. 

Each row of the CSV file must contain two comma-separated columns:

1. The full path to a normalized `.mnc` image for one participant.
2. The full path to the corresponding `template_mask.mnc`. 

For example, `subject.csv` for the T1w template, ages 10-12 yrs, should contain: 

```
/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/sub-124_rec-composed_T2w_straight_norm.mnc,/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/template_mask.mnc
/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/sub-129_rec-composed_T2w_straight_norm.mnc,/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/template_mask.mnc
/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/sub-138_rec-composed_T2w_straight_norm.mnc,/PATH/TO/SCRATCH/pediatric_templates/template_T1w_10-12/template_mask.mnc
...

```

### 7.3 Add necessary scripts

#### 7.3.1 Prepare the Python template-generation scripts

The `generate_template.py` script can be found in your clone of the `neuropoly/template` repository.

Copy this script into your `PATH/TO/SCRATCH/DIRECTORY/pediatric_templates` directory.

Create six copies of the script, one for each template:
```
generate_template_T1w_10-12.py
generate_template_T1w_13-15.py
generate_template_T1w_16-17.py
generate_template_T2w_10-12.py
generate_template_T2w_13-15.py
generate_template_T2w_16-17.py
```

In each copy, replace `subject.csv` with the path that points to the corresponding `subject.csv` file from the previous step.

#### 7.3.2 Create the SLURM submission scripts

Create one bash submission script for each template. These scripts will be submitted to the Alliance HPC cluster using sbatch.

For example, `generate_template_T1w_10-12.sh` should contain:

```bash
#!/bin/bash

#SBATCH --job-name=T1w_10-12
#SBATCH --time=48:00:00
#SBATCH --mem-per-cpu=8000

export VOLUME_CACHE_THRESHOLD=-1
export OMP_NUM_THREADS=1

python -m scoop -vvv generate_template_T1w_10-12.py
```

> [!IMPORTANT]
> **`VOLUME_CACHE_THRESHOLD=-1`** prevents this crash:
>```
>`volume_io error: copy_volume(): copying cached volumes not implemented.`
>```
> Setting the value to `-1` tells MINC to always load the whole image into memory. Because of this, each registration needs more memory, so make sure your job requests enough (`--mem` in SLURM).

> [!NOTE]
> **`OMP_NUM_THREADS=1`** keeps each registration to one core, so the total stays within what you requested from SLURM. This setting only affects speed. It is not needed to avoid the crash.

Create equivalent scripts for the other five templates, updating the job name and Python script filename accordingly. 

Your scratch diectory should now look like this :
```bash
pediatric_templates/
│
├── nist_mni_pipelines/
│
├── template_T1w_10-12/
│   ├── generate_template_T1w_10-12.py # python script to generate template
│   ├── generate_template_T1w_10-12.sh # bash script to submit to Alliance HPC
│   ├── sub-*_rec-composed_T1w_straight_norm.mnc # all participant files corresponding to this template
│   ├── subject.csv  # csv file containing the paths to the .mnc fles
│   └── template_mask.mnc # template mask corresponding to this template
│
├── template_T1w_13-15/
│   └── ...
│
├── template_T1w_16-17/
│   └── ...
│
├── template_T2w_10-12/
│   └── ...
│
├── template_T2w_13-15/
│   └── ...
│
└── template_T2w_16-17/
│   └── ...
```

```bash
pediatric_templates/
│
├── generate_template_T1w_10-12.py # python script to generate template
├── generate_template_T1w_13-15.py
├── generate_template_T1w_16-17.py
├── generate_template_T2w_10-12.py
├── generate_template_T2w_13-15.py
├── generate_template_T2w_16-17.py
├── generate_template_T1w_10-12.sh # bash script to submit to Alliance HPC
├── generate_template_T1w_13-15.sh
├── generate_template_T1w_16-17.sh
├── generate_template_T2w_10-12.sh
├── generate_template_T2w_13-15.sh
├── generate_template_T2w_16-17.sh
├── nist_mni_pipelines/ # NIST MNI pipelines folder
├── template_T1w_13-15/ # folder containing the .mnc files of all participants to process for each template
├── template_T1w_16-17/
├── template_T1w_10-12/
├── template_T1w_13-15/
└── template_T1w_16-17/

```

### 8. Generate T1w templates

Template generation is performed independently for each age group and contrast. Each template should be created by submitting a slurm job to the Alliance HPC cluster. 

Before submitting the jobs, load the required software modules in your Alliance terminal:

```bash
module load StdEnv/2020 gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
```

Next, to submit a template-generation job, use the following command (example for the 10–12-year T1w template):

```bash
sbatch --account=YOUR_ALLIANCE_RESEARCH_GROUP_ACCOUNT generate_template_T1w_10-12.sh 
```

> [!IMPORTANT]
> Replace `YOUR_ALLIANCE_RESEARCH_GROUP_ACCOUNT` with the name of the research group allocation associated with your Alliance account (e.g., your supervisor's or research group's allocation). This is not your personal username.


The template generation process uses iterative nonlinear registration and averaging. At each iteration:

* It does a voxel-wise averaging of `sub-*_T2w_straight_norm.mnc` in `template_mask.mnc` space to create `avg.001.mnc`
* This process is then repeated in `avg.001.mnc`, to generate `avg.002.mnc`.
* etc.

Keep running for several iterations. Sometimes, there is not enough time for an iteration to finish completing and the job needs to be resubmitted. 
Each iteration generates the following files upon successful completion:

```
model_nl_all/
    i/
        # 201 .mnc files
        # 112 .xfm files
    avg.i.mnc
    avg.i_mask.mnc
    sd.i.mnc
```
> `avg.i.mnc` is the template generated by iteration `i`. 


### 9. Convert the templates to NIFTI format

After N iterations, if you are satisfied with your templates (`avg.N.mnc`), you can convert them to NIFTI format.

For each of the six templates, you can convert the template to NIFTI with the following:

```bash
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
cd PATH/TO/SCRATCH/template-pediatric
mv model_nl_all model_nl_all_T1w # such that T2w template-generation does not use the T1w averages
cd model_nl_all_T1w
mnc2nii avg.N.mnc
gzip avg.N.nii
mv avg.N.nii.gz pediatric-template_T1w.nii.gz
```

