# Containerizing a pyTorch application with Singularity (now Apptainer)

This tutorial is divided in three steps:
1. Test the pyTorch application within a Conda environment.
2. Use the application within a Singularity container as an interactive shell.
3. Use the container as a standalone file.

## Prerequisites

- Donwload miniconda from:  https://docs.conda.io/en/latest/miniconda.html
- Install Miniconda: 
        
        bash Miniconda3-latest-Linux-x86_64.sh
- If using a bash shell add the following lines to $HOME/.bashrc

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
- Execute `source $HOME/.bashrc`

## 1. Test the pyTorch application within a Conda environment.
- I'll be using the REANN Package from [github](https://github.com/zhangylch/REANN), thus the conda environment will be called reann

        #create a conda environment
        conda create --name reann

        #activate the environment
        conda activate reann
