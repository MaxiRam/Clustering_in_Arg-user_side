# Containerizing a pyTorch application with Singularity (now Apptainer)

This tutorial is divided in three steps:
1. Test the PyTorch application within a Conda environment.
2. Use the application within a Singularity container as an interactive shell.
3. Use the container as a standalone file.

## Prerequisites

- Donwload miniconda from:  https://docs.conda.io/en/latest/miniconda.html
- Install Miniconda: 

```bash      
bash Miniconda3-latest-Linux-x86_64.sh
```
- If using a bash shell add the following lines to $HOME/.bashrc

```bash
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/mramos/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
eval "$__conda_setup"
else
if [ -f "/home/mramos/miniconda3/etc/profile.d/conda.sh" ]; then
        . "/home/mramos/miniconda3/etc/profile.d/conda.sh"
else
        export PATH="/home/mramos/miniconda3/bin:$PATH"
fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

- Execute `source $HOME/.bashrc`
  
  Now `(base)` should appear at the left side of your prompt, indicating that the **base** environment is active. My prompt looks like this:

```bash
(base) [mramos@mendieta ~]$
```

## 1. Test the PyTorch application within a Conda environment.
- I'll be using the REANN Package from [github](https://github.com/zhangylch/REANN), thus the conda environment will be called reann
```bash
#create a conda environment
conda create --name reann

#activate the environment
conda activate reann
```

Now the prompt should look similar to: `(reann) [mramos@mendieta ~]$` indicating that **reann** environment is now active.

```bash
#install conda-build package
conda install conda-build
```

- REANN requires PyTorch and opt_einsum packages. As Mendieta's A30 GPUs belong to the latest NIVIDIA architecture, they require cudatoolkit 11.x to work properly. 
```bash
# Install pytorch
conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 -c pytorch -c conda-forge

# Install opt_einsum
conda install opt_einsum -c conda-forge
```

- Download REANN package from github. I´ll place it at `$HOME/Software`

```bash
mkdir $HOME/Software
cd $HOME/Software
git clone https://github.com/zhangylch/REANN.git

# To make sure python will find all reann's scripts
conda develop $HOME/Software/REANN/reann
```

Now the application should work. I will use an interactive session in a compute node allocated by SLURM. This is really useful when testing your applications because you get a console whitin the node and can run the same way you do in your laptop or desktop. To get the interactive session run:

```bash
salloc -p short --time=1:00:00 -N 1 --gres=gpu:1 srun --pty --preserve-env $SHELL
```

This is a session using the **short** partition (`-p short`), with a time limit of **one hour** (`--time=1:00:00`), allocating **1 node** (`-N 1`) and reserving **1 GPU** (`--gres=gpu:1`).

```bash
conda activate reann
cd $HOME/Software/REANN/reann/example/co2+ni100
```

Modify the file `$HOME/Software/REANN/reann/example/co2+ni100/para/input_nn` with yout favorite text editor

- line 12: write `Epoch = 20` instead of 20000 so it will run a few seconds.
- line 42: modify the path between quotes so it points to `$HOME/Software/REANN/data/co2+Ni100/` but write the absolute path.

Now run the calculation:

```bash
export OMP_NUM_THREADS=1
python3 -m torch.distributed.run --nproc_per_node=1 --nnodes=1 $HOME/Software/REANN/reann/run/train.py
```

After a while the calculation will end seccesfully and convergence can be seen in `nn.err` file.

## 2. Use the application within a Singularity container as an interactive shell.

Now, we'll do the same but inside a container. Let's create a folder ofr our containers and download a PyTorch container from DockerHub:

```bash
mkdir $HOME/containers; cd $HOME/containers
apptainer build pytorch_1.11.0-cuda11.3-cudnn8-runtime.sif docker://pytorch/pytorch:1.11.0-cuda11.3-cudnn8-devel
```

After some minutes `pytorch_1.11.0-cuda11.3-cudnn8-runtime.sif` file must exist. This file is a container but it doesn't have the REANN package. Now, we will expand the container into a folder and then add the REANN package to it.

```bash
apptainer build --sandbox reann_container pytorch_1.11.0-cuda11.3-cudnn8-runtime.sif
```
Before entering the container execute the command `cat /etc/os-release` and you'll get the following output in Mendieta:

```bash
NAME="Rocky Linux"
VERSION="8.5 (Green Obsidian)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.5"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Rocky Linux 8.5 (Green Obsidian)"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:rocky:rocky:8:GA"
HOME_URL="https://rockylinux.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
ROCKY_SUPPORT_PRODUCT="Rocky Linux"
ROCKY_SUPPORT_PRODUCT_VERSION="8"
```

Now let's get a shell inside the container and check the Linux distribution:

```bash
apptainer shell --writable reann_container/
cat /etc/lsb-release
```

Now the prompt has changed to `Apptainer>` indicating that we are inside the container and the Linux distribution is:
```bash
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
DISTRIB_DESCRIPTION="Ubuntu 18.04.6 LTS"
```
