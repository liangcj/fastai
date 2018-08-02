# Setting up the fast.ai course materials on Biowulf
For those taking the fast.ai deep learning course, here are instructions for setting up your own environment on Biowulf so you can take advantage of the HPC NVIDIA GPUs.

## Things you only have to do once

#### Get a Biowulf account
If you do not already have a Biowulf account, speak to your boss or PI about getting an account. Log in to your Biowulf account.

#### Install Anaconda Python 3.6
Anaconda does a lot of things; particularly for us, it includes `conda`, which is a Python package manager and Python environment manager. Since the course and deep learning packages rely on many different Python packages, this tool ensures that all the necessary packages (and sometimes specific package versions) are installed.

According to NIH HPC, Python environments should be created in data directories and **not** the home directory, as the installations can get large. So first, in bash, navigate to your data directory (replace "user" with your username)

```bash
cd /data/user
mkdir -p temp
cd temp
```

Download the miniconda installer and install to `/data/user/conda`:

```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -p /data/user/conda -b
```
Complete some initial setup:

```bash
source /data/user/conda/etc/profile.d/conda.sh
conda activate base
which python
conda update conda
conda clean --all --yes
```

source: https://github.com/reshamas/fastai_deeplearn_part1/blob/master/tools/setup_personal_dl_box.md and https://hpc.nih.gov/apps/python.html#envs

#### Clone fastai GitHub repo to your Biowulf data directory
Go to your data directory and clone the fastai GitHub repo. This includes course materials like IPython notebooks and course-specific Python code. In bash, again replacing "user" with your username:

```bash
cd /data/user
git clone https://github.com/fastai/fastai.git
```
Now you should have a `/data/user/fastai` directory containing all the course materials

#### Install fastai virtual environment using `conda`
Go to your new fastai directory, where the `environment.yml` file should be located. This is basically a text file listing all the Python packages that are needed for this course. We will now direct `conda` to use this list of packages to create a new fastai environment:

```bash
cd /data/user/fastai
conda env create -f environment.yml
```
This will kick off the downloading and installation of many Python packages. I think this took about 30-60 minutes.

source: https://github.com/reshamas/fastai_deeplearn_part1/blob/master/tools/setup_personal_dl_box.md

#### Download "dogs and cats" dataset
This is a toy dataset that is used in the course. I put it in my `/data/liangcj` directory:

```bash
cd /data/user
wget http://files.fast.ai/data/dogscats.zip
unzip dogscats.zip
```

## Things you have to do each time you log in

#### Start an interactive session with GPU allocation
A simple way to get a session with GPU access is an interactive session:

```bash
sinteractive --gres=gpu:k80:1 --mem=30g
```

Feel free to play around with the gpu choice. Above I specified "k80" but I think "v100" might be newer/faster. Also so far 30 GB of memory seems to have been sufficient.

When your interactive session is allocated, take note of the compute node (e.g. **cn4187**), as you will need it later.

Now you need to activate your fastai Python environment so that all those packages are available to you:

```bash
source /data/user/conda/etc/profile.d/conda.sh
conda activate fastai
```

When it successfully activates you will see **(fastai)** in front of your bash prompt. I chose to add the above two lines to my `.bashrc` file (located in your home directory) so that it is automatically run whenever I start a new bash session

source: https://hpc.nih.gov/docs/deep_learning.html

#### Start Jupyter server and tunnel
Start tmux so that if you get accidentally logged off your session will continue and you can pickup where you left off:

```bash
module load tmux
tmux
```

Go to your data directory and start a Jupyter server:

```bash
cd /data/user
jupyter notebook --port 1111 --no-browser
```

Make sure you choose a different port number than **1111*** as each user neeeds their own unique port to avoid clashes. Once successful there should be a long URL at the bottom of your screen, which you will come back to later.

In Windows, open a new PuTTY session:

 - In the field which says "Host Name (or IP Address)" enter biowulf.nih.gov as usual. 
 - On the left panel, click the plus-sign next to "SSH", and click on "Tunnels". 
 - Under "Source port" enter your port number (in our case **1111**). 
 - Under "Destination", enter "localhost:1111" (remember to replace the port number with your own).
 - Click the "Add" button.
 - Click the "Open" button at the bottom.

Log in as usual. Now create a tunnel to the compute node on your port where the Jupyter server is running, since compute nodes are not directly accessible. Remember to replace the port number and compute node ID with your specific ones:

```bash
ssh -L 1111:localhost:1111 cn4187
```

source: https://hpc.nih.gov/apps/jupyter.html

#### Open Jupyter notebook in your browser
Now go back and copy that long URL from the initial session and paste it into your browser. For lesson one, navigate to `/fastai/courses/dl1/lesson1.ipynb`
