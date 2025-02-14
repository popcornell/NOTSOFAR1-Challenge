# Introduction 
Welcome to the "NOTSOFAR-1: Distant Meeting Transcription with a Single Device" Challenge.

This repo contains the baseline system code for the NOTSOFAR-1 Challenge.

For more details see:
1. CHiME website: https://www.chimechallenge.org/current/task2/index
2. Preprint: https://arxiv.org/abs/2401.08887


# Project Setup
The following steps will guide you through setting up the project on your machine. <br>

### Windows Users
This project is compatible with **Linux** only, Windows users should use WSL2 to run it. <br>
Follow the instructions in the [WSL2 Installation Guide](https://learn.microsoft.com/en-us/windows/wsl/install) to install WSL2 on your machine. <br>
Next, install Ubuntu 20.04 from the [Microsoft Store](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71?activetab=pivot:overviewtab). <br>

Alternatively, you can run and work on the project in a [devctonainer](https://containers.dev/) using, for example, the [Dev Containers VSCode Extension](https://code.visualstudio.com/docs/devcontainers/containers).


## Cloning the Repository

Clone the `NOTSOFAR1-Challenge` repository from GitHub. Open your terminal and run the following command:

```bash
sudo apt-get install git
cd path/to/your/projects/directory
git clone https://github.com/microsoft/NOTSOFAR1-Challenge.git
```


## Setting up the environment

### Conda

#### Step 1: Install Conda

Conda is a package manager that is used to install Python and other dependencies.<br>
To install Miniconda, which is a minimal version of Conda, run the following commands:

```bash
miniconda_dir="$HOME/miniconda3"
script="Miniconda3-latest-Linux-$(uname -m).sh"
wget --tries=3 "https://repo.anaconda.com/miniconda/${script}"
bash "${script}" -b -p "${miniconda_dir}"
export PATH="${miniconda_dir}/bin:$PATH"
````
*** You may change the `miniconda_dir` variable to install Miniconda in a different directory.


#### Step 2: Create a Conda Environment 

Conda Environments are used to isolate Python dependencies. <br> 
To set it up, run the following commands:

```bash
source "/path/to/conda/dir/etc/profile.d/conda.sh"
conda create --name notsofar python=3.10 -y
conda activate notsofar 
cd /path/to/NOTSOFAR-Repo
python -m pip install --upgrade pip
pip install --upgrade setuptools wheel Cython fasttext-wheel
pip install azure-cli 
pip install -r requirements.txt
```

### PIP

#### Step 1: Install Python 3.10

Python 3.10 is required to run the project. To install it, run the following commands:

```bash
sudo apt update && sudo apt upgrade
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install python3.10
```

#### Step 2: Set Up the Python Virtual Environment

Python virtual environments are used to isolate Python dependencies. <br> 
To set it up, run the following commands:

```bash
sudo apt-get install python3.10-venv
python3.10 -m venv /path/to/virtualenvs/NOTSOFAR
source /path/to/virtualenvs/NOTSOFAR/bin/activate
```

#### Step 3: Install Python Dependencies

Navigate to the cloned repository and install the required Python dependencies:

```bash
cd /path/to/NOTSOFAR-Repo
python -m pip install --upgrade pip
pip install --upgrade setuptools wheel Cython fasttext-wheel
sudo apt-get install python3.10-dev ffmpeg build-essential
pip install -r requirements.txt
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### Step 4: Install Azure CLI

Azure CLI is required to download the datasets. To install it, run the following commands:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

# Running evaluation - the inference pipeline
The following command will download the entire dev-set of the recorded meeting dataset and run the inference pipeline
according to selected configuration. The default is configured to `debug_inference.yaml` for quick debugging, running on a single session.
```bash
cd /path/to/NOTSOFAR-Repo
python run_inference.py
```

The first time you run it, it will automatically download these required models and datasets from blob storage:


1. The development set of the meeting dataset (dev-set) will be stored in the `artifacts/meeting_data` directory.
2. The CSS models required to run the inference pipeline will be stored in the `artifacts/css_models` directory. 

Outputs will be written to the `artifacts/outputs` directory.

### Running on a subset of the dev-set meeting data
`run_inference.py` by default points to the config yaml that loads the full meeting dataset: 

```
conf_file = project_root / 'configs/inference/inference_v1.yaml'
```

For debugging, to run on only one meeting and the Whisper 'tiny' model, you can use the following config:
```
conf_file = project_root / 'configs/inference/debug_inference.yaml'
```

The `session_query` argument found in the yaml config file offers more control over filtering meetings.
Note that to submit results on the dev-set, you must evaluate on the full set and no filtering must be performed.


# Integrating your own models 
The inference pipeline is modular, designed for easy research and extension.
Begin by exploring the following components:
- **Continuous Speech Separation (CSS)**: See `css_inference` in `css.py` . We provide a model pre-trained on NOTSOFAR's simulated training dataset, as well as inference and training code. For more information, refer to the [CSS section](#running-css-continuous-speech-separation-training).
- **Automatic Speech Recognition (ASR)**: See `asr_inference` in `asr.py`. The baseline implementation relies on [Whisper](https://github.com/openai/whisper). 
- **Speaker Diarization**: See `diarization_inference` in `diarization.py`. The baseline implementation relies on the [NeMo toolkit](https://github.com/NVIDIA/NeMo).



# Running CSS (continuous speech separation) training

## 1. Local training on a data sample for development and debugging
The following command will run CSS training on the 10-second simulated training data sample in `sample_data/css_train_set`.
```bash
cd /path/to/NOTSOFAR-Repo
python run_training_css_local.py
```

## 2. Training on the full simulated training dataset

### Step 1: Download the simulated training dataset
You can use the `download_simulated_subset` function in `utils/azure_storage.py` to download the training dataset from blob storage.
You have the option to download either the complete dataset, comprising almost 1000 hours, or a smaller, 200-hour subset.

For example, to download the entire 1000-hour dataset, make the following calls to download both the training and validation subsets:
```python
ver='v1.4'  # this should point to the lateset and greatest version of the dataset.

# Option 1: Download the training and validation sets of the entire 1000-hour dataset. 
train_set_path = download_simulated_subset(
    version=ver, volume='1000hrs', subset_name='train', destination_dir=os.path.join(my_dir, 'train'))

val_set_path = download_simulated_subset(
    version=ver, volume='1000hrs', subset_name='val', destination_dir=os.path.join(my_dir, 'val'))


# Option 2: Download the training and validation sets of the smaller 200-hour dataset.
train_set_path = download_simulated_subset(
    version=ver, volume='200hrs', subset_name='train', destination_dir=os.path.join(my_dir, 'train'))

val_set_path = download_simulated_subset(
    version=ver, volume='200hrs', subset_name='val', destination_dir=os.path.join(my_dir, 'val'))
```

### Step 2: Run CSS training
Once you have downloaded the training dataset, you can run CSS training on it using the `run_training_css` function in `css/training/train.py`.
The `main` function in `run_training_css.py` provides an entry point with `conf`, `data_root_in`, and `data_root_out` arguments that you can use to configure the run.

It is important to note that the setup and provisioning of a compute cloud environment for running this training process is the responsibility of the user. Our code is designed to support **PyTorch's Distributed Data Parallel (DDP)** framework. This means you can leverage multiple GPUs across several nodes efficiently.

### Step 3: Customizing the CSS model
To add a new CSS model, you need to do the following:
1. Have your model implement the same interface as our baseline CSS model class `ConformerCssWrapper` which is located
in `css/training/conformer_wrapper.py`. Note that in addition to the `forward` method, it must also implement the 
`separate`, `stft`, and `istft` methods. The latter three methods will be used in the inference pipeline and to 
calculate the loss when training.
2. Create a configuration dataclass for your model. Add it as a member of the `TrainCfg` dataclass in 
`css/training/train.py`.
3. Add your model to the `get_model` function in `css/training/train.py`.



# NOTSOFAR-1 Datasets - Download Instructions
This section is for those specifically interested in downloading the NOTSOFAR datasets.<br>
The NOTSOFAR-1 Challenge provides two datasets: a recorded meeting dataset and a simulated training dataset. <br>
The datasets are stored in Azure Blob Storage, to download them, you will need to install `Azure CLI` (part of the environment setup instructions above).

You can use either the python utilities in `utils/azure_storage.py` or the `az storage copy` command to download the datasets as described below.

To install Azure CLI, run the following command:
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```



### Meeting Dataset for Benchmarking and Training

The NOTSOFAR-1 Recorded Meeting Dataset is a collection of 315 meetings, each averaging 6 minutes, recorded across 30 conference rooms with 4-8 attendees, featuring a total of 35 unique speakers. This dataset captures a broad spectrum of real-world acoustic conditions and conversational dynamics.

### Download

To download the dataset, you can call the python function `download_meeting_subset` within `utils/azure_storage.py`.

Alternatively, using Azure CLI, set these arguments and run the following command:

`--destination` - replace with a path to the directory where you want to download the benchmarking dataset (destination directory must exist). <br>
`--include-path` - replace with the dataset you want to download: <br>
- `subset_name`: name of split to download (`dev_set` / `eval_set` / `train_set`).
- `version`: version to download (`240103g` / etc.). it's best to use the latest.
Currently only **dev_set** (no GT) and **train_set** are available. See timeline on the [NOTSOFAR page](https://www.chimechallenge.org/current/task2/index) for when the other sets will be released.
See doc in `download_meeting_subset` function in `utils/azure_storage.py` for latest available versions.

```bash
az storage copy --recursive --only-show-errors --destination <path to NOTSOFAR datasets>/benchmark --source https://notsofarsa.blob.core.windows.net/benchmark-datasets --include-path <subset_name>/<version>/MTG
```

Example:
```bash
az storage copy --recursive --only-show-errors --destination . --source https://notsofarsa.blob.core.windows.net/benchmark-datasets --include-path dev_set/240103g/MTG
````


### Simulated Training Dataset

The NOTSOFAR-1 Training Dataset is a 1000-hour simulated training dataset, synthesized with enhanced authenticity for real-world generalization, incorporating 15,000 real acoustic transfer functions.

### Download


To download the dataset, you can call the python function `download_simulated_subset` within `utils/azure_storage.py`.
Alternatively, using Azure CLI, set these arguments and run the following command:

`--destination` - replace with a path to the directory where you want to download the benchmarking dataset (destination directory must exist). <br>
`--include-path` - replace with the dataset you want to download: <br>
- `version`: version of the train data to download (`v1` / `v1.1` / `v1.2` / `v1.3` / `1.4` / etc.)
- `volume` - volume of the train data to download (`200hrs` / `1000hrs` / etc.)
- `subset_name`: train data type to download (`train` / `val`)

```bash
az storage copy --recursive --only-show-errors --destination <path to NOTSOFAR datasets>/simulated --source https://notsofarsa.blob.core.windows.net/css-datasets --include-path <version>/<volume>/<subset name>
```

Example:
```bash
az storage copy --recursive --only-show-errors --destination . --source https://notsofarsa.blob.core.windows.net/css-datasets --include-path v1.4/1000hrs/train
```


## Data License
This public data is currently licensed for use exclusively in the NOTSOFAR challenge event. 
We appreciate your understanding that it is not yet available for academic or commercial use. 
However, we are actively working towards expanding its availability for these purposes. 
We anticipate a forthcoming announcement that will enable broader and more impactful use of this data. Stay tuned for updates. 
Thank you for your interest and patience.