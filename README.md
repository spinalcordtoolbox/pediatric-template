# Pediatric Template
Repository for generating pediatric template

## Data

`data.neuro.polymtl.ca:datasets/philadelphia-pediatric`

Instructions to download: https://intranet.neuro.polymtl.ca/computing-resources/data/git-datasets.html#download

## Generating T1w and T2w Templates
Step-by-step walk-through on how to generate T1w and T2w pediatric templates from philadelphia-pediatric dataset.
All code below should be run in your command-line (bash) on CCDB. Each step begins from your **working directory**: `/PATH/TO/pediatric-template`.

### 1. Get data from git annex
```
git clone git@data.neuro.polymtl.ca:datasets/philadelphia-pediatric
cd philadelphia-pediatric
git checkout 31aea09ec124e2aebffef28927731ae635db3f4f
git annex get .
cd ..
```


### 2. Dependencies

Install SCT
```
git clone git@github.com:spinalcordtoolbox/spinalcordtoolbox.git
cd spinalcordtoolbox
git checkout -b 49a40673e6d1521eb7c2d1d6d7b338ab6811448d
./install_sct
conda activate python/envs/venv_sct/
cd ..
```

Install template pipeline (https://github.com/neuropoly/template)
```
git clone git@github.com:neuropoly/template.git
cd template
git checkout a7915f4ccfa075a5d31f4ea84bb9761d42710e9e
cd ..
```

> [!NOTE]  
> The T1w template and the T2w template should be in the same template space.

### 3. T1w data preprocessing and normalization
```
1. cd template
2. cp ../configuration_T1w.json configuration.json
3. python preprocess_normalize.py configuration.json

# Rename derivatives such that the T2w data normalization does not override the outputs of the T1w data normalization
4. mv ../philadelphia-pediatric-processing/derivatives/sct_straighten_spinalcord ../philadelphia-pediatric-processing/derivatives/sct_straighten_spinalcord_T1w
5. mv ../philadelphia-pediatric-processing/derivatives/template ../philadelphia-pediatric-processing/derivatives/template_T1w
```


### 4. T2w data preprocessing and normalization

Once the template space for using the `T1w` data, a new template space is **not** needed to be created using the `T2w` images. This is to ensure that both the templates are in the same template space. For this, follow the steps below:

``` 
1. Comment out the `generate_initial_template_space` method in the preprocess_normalize.py script
2. cp ../configuration_T2w.json configuration.json
3. python preprocess_normalize.py configuration.json

# Rename derivatives
4. mv ../philadelphia-pediatric-processing/derivatives/sct_straighten_spinalcord ../philadelphia-pediatric-processing/derivatives/sct_straighten_spinalcord_T2w
5. mv ../philadelphia-pediatric-processing/derivatives/template ../philadelphia-pediatric-processing/derivatives/template_T2w
```


### 5. Log in to Digital Research Alliance of Canada (the Alliance) High-Performance Computer (HCP)
Refer to [NeuroPoly Lab Manual](https://intranet.neuro.polymtl.ca/computing-resources/compute-canada.html) for further specifications.


### 6. Set up correct environment on Alliance HPC
```
cd PATH/TO/scratch
mkdir pediatric-template
cd pediatric-template
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
pip install --upgrade pip
pip install scoop
git clone https://github.com/vfonov/nist_mni_pipelines.git
nano ~/.bashrc
    :'
    export PYTHONPATH="${PYTHONPATH}:/PATH/TO/scratch/philadelphia-pediatric/derivatives/model_nl_all_T2w/code/nist_mni_pipelines"
    export PYTHONPATH="${PYTHONPATH}:/PATH/TO/scratch/philadelphia-pediatric/derivatives/model_nl_all_T2w/code/nist_mni_pipelines/"
    export PYTHONPATH="${PYTHONPATH}:/PATH/TO/scratch/philadelphia-pediatric/derivatives/model_nl_all_T2w/code/nist_mni_pipelines/ipl/"
    export PYTHONPATH="${PYTHONPATH}:/PATH/TO/scratch/philadelphia-pediatric/derivatives/model_nl_all_T2w/code/nist_mni_pipelines/ipl"
    '
source ~/.bashrc
pip install "git+https://github.com/NIST-MNI/minc2-simple.git@develop_new_build#subdirectory=python"
```


### 7. Bring necessary data to Alliance HCP using rsync
The data contained in `philadelphia-pediatric-processing/derivatives/template_T1w` and `philadelphia-pediatric-processing/derivatives/template_T2w`\
    should be on your Alliance HCP scratch directory.
Your directory `PATH/TO/SCRATCH/template-pediatric` should now have the following structure:
```
nist_mni_pipelines
my_job.sh
make_subjects_csv.sh
generate_template.py
template_T1w/
    sub*T1w_straight_norm.mnc
    template_mask.mnc
template_T2w/
    sub*T2w_straight_norm.mnc
    template_mask.mnc
```


### 8. Bring necessary data to Alliance HCP using rsync
The data contained in `philadelphia-pediatric-processing/derivatives/template_T1w` and `philadelphia-pediatric-processing/derivatives/template_T2w`\
    should be on your Alliance HCP scratch directory.
Your directory `PATH/TO/SCRATCH/template-pediatric` should now have the following structure:
```
nist_mni_pipelines
my_job.sh
make_subjects_csv.sh
generate_template.py
template_T1w/
    sub*T1w_straight_norm.mnc
    template_mask.mnc
template_T2w/
    sub*T2w_straight_norm.mnc
    template_mask.mnc
```


### 9. Create T1w template

The template-generation process is an iterative averaging process.
* It does a voxel-wise averaging of `sub-*_T2w_straight_norm.mnc` in `template_mask.mnc` space to create `avg.001.mnc`
* This process is then repeated in `avg.001.mnc`, to generate `avg.002.mnc`.
* etc.

```
cd PATH/TO/SCRATCH/template-pediatric
# generate subjects.csv (list of full path to `sub-*_T1w_straight_norm.mnc` files and `template_mask.mnc`, used by `generate_template.py`)
./make_subjects_csv.sh template_T1w  
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
sbatch --account=def-jcohen --time=48:00:00 --mem-per-cpu 8000 my_job.sh
```

Keep running for several iterations. Sometimes, there is not enough time for an iteration to finish completing and the job needs to be resubmitted.
Each iteration i generates the following files upon successful completion:
```
model_nl_all/
    i/
        # 201 .mnc files
        # 112 .xfm files
    avg.i.mnc
    avg.i_mask.mnc
    sd.i.mnc
```
> `avg.i.mnc` is the template generated by iteration _i_.


### 10. Convert to T1w template to NIFTI

After N iterations, if you are satisfied with your template (`avg.N.mnc`), you can convert it to NIFTI format and save it!
```
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
cd PATH/TO/SCRATCH/template-pediatric
mv model_nl_all model_nl_all_T1w # such that T2w template-generation does not use the T1w averages
cd model_nl_all_T1w
mnc2nii avg.N.mnc
gzip avg.N.nii
mv avg.N.nii.gz pediatric-template_T1w.nii.gz
```


### 11. Create T2w template

Same as steps 10 and 9, but with T2w data!

```
cd PATH/TO/SCRATCH/template-pediatric
# generate subjects.csv (list of full path to `sub-*_T2w_straight_norm.mnc` files and `template_mask.mnc`, used by `generate_template.py`)
./make_subjects_csv.sh template_T2w  
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
sbatch --account=def-jcohen --time=48:00:00 --mem-per-cpu 8000 my_job.sh
```
When you are satisfied with your template, you can convert it to NIFTI:
```
module load StdEnv/2020  gcc/9.3.0 minc-toolkit/1.9.18.1 python/3.8.10
mv model_nl_all model_nl_all_T2w
cd model_nl_all_T2w
mnc2nii avg.N.mnc
gzip avg.N.nii
mv avg.N.nii.gz pediatric-template_T2w.nii.gz
```
