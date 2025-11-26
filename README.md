# Installing and configuring MatFlow

We assume you have already installed micromamba. If not, please install it from here: https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html#

### 1. Download software and files

Everything required can be downloaded from the provided SharePoint link, or from the USB stick.

- Download the DAMASK docker image archive: `damask.tar` (`damask-mac.tar` for Mac users) 
- Download the MOOSE docker image archive `proteus.tar` (`proteus-mac.tar` for Mac users)
- Download the conda lock file: `matflow-lock.yml` (`matflow-osx-lock.yml` for Mac users)
- Download and unzip the tutorials directory `matflow-tutorials.zip`

#### 1b. Extra software for the afternoon DAMASK (crystal plasticity) tutorials only

- If you have not yet installed the MATLAB runtime, please install from here: https://uk.mathworks.com/products/compiler/matlab-runtime.html (or from the provided SharePoint link/USB stick).
  - Detailed instructions here: https://uk.mathworks.com/help/compiler/install-the-matlab-runtime.html
- If you have not yet installed Dream3D, please install from here: https://dream3d.bluequartz.net/Download/#prebuilt-binaries (or from the provided SharePoint link/USB stick).

### 2. Install the MatFlow micromamba enviroment

This micromamba environment includes MatFlow and other Python dependencies for the workflow we will run:

```
micromamba env create -y -n matflow -f /path/to/matflow-lock.yaml # or matflow-osx-lock.yaml on Mac
```
Note if you have an older version of micromamba, you might need to add `-c conda-forge` to the end of the command.

Activate the environment:

```
micromamba activate matflow
```

If micromamba complains about your shell not being initialised, please run the suggested command. There is an option to run an ad-hoc command if you prefer, which will not modify any of your shell init files like ~/.bashrc etc. Please see https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html for details.


### 3. Configure MatFlow environments

Define environments for various pre- and post-processing scripts in MatFlow:

```
matflow env setup python-all
```

Let MatFlow know where the DAMASK and MOOSE docker image archives are (these commands might take a minute or two):

```
matflow env setup damask --docker-archive /path/to/damask.tar
```

```
matflow env setup moose --docker-archive /path/to/proteus.tar
```

#### Extra steps for the afternoon DAMASK (crystal plasticity) tutorials only

Let MatFlow know where Dream3D is:

```
matflow env setup dream3d -p /absolute/path/to/pipeline-runner
```

This will live somewhere inside the Dream3D application folder. On Windows, it will be under:
`\DREAM3D-6.5.171-Win64\PipelineRunner.exe`. Please provide the full absolute (for example `C:\software\Dream3D\DREAM3D-6.5.171-Win64`). You may have to use quotes if there is a space in the file path. On Linux, it will be under a `bin` directory.

Let MatFlow know where the MATLAB runtime is (the path that ends in R2024b):

```
matflow env setup matlab --runtime-path /path/to/matlab/runtime/R2024b
```

### 4. Begin!

- Change directory to the unzipped `matflow-tutorials` folder
- Make sure your micromamba environment is activated (`micromamba activate matflow`)
- Start jupyter lab:

```
jupyter lab
```

> Please note: you can use classic Jupyter notebook or VSCode instead of Jupyter Lab, but some of the markdown formatting will be slightly off.

# Appendix

### Configuring on CSD3

If you wish to install MatFlow on CSD3, follow the previous steps as normal, but you will need to tell MatFlow to use the SLURM scheduler by default, with this command:

```
matflow config set default_scheduler slurm
```

When submitting workflows on CSD3, you'll need to add a `resources` block to the top of you workflow YAML files, in the following format:

```yaml
resources:
  any:
    scheduler_args:
      directives:
        --time: 10:00 # jobscript time limit
        --account: UKAEA-AP002-CPU # for example
        --partition: ukaea-spr # for example
```

### Using singularity instead of Docker

Configure the container environments via singularity like this:

For DAMASK:

```
matflow env setup docker --singularity-archive /path/to/docker/damask/archive.tar
```

or:

```
matflow env setup docker --singularity-sif /path/to/damask.sif
```

For MOOSE:

```
matflow env setup moose --singularity-archive /path/to/docker/proteus/archive.tar
```

or:

```
matflow env setup moose --singularity-sif /path/to/proteus.sif
```

### Pointing MatFlow to native installations of DAMASK and Proteus

We recommend using the provided Docker image of DAMASK, because it includes modifications not currently found in the public code. However, if you have a working installation of Proteus, you can tell MatFlow to use that instead of the Docker image. The easiest way
to do that is to open up MatFlow's environment definitions file. If you have run a previous
`matflow env setup` command (as above), then you should have a file: `~/.matflow/configured_envs.yaml`. You can add another environment definition to this file that looks something like this:

```yaml
- name: moose_env  
  specifiers: {}
  setup: | # add commands here to setup the Proteus environment (e.g. loading modules, source shell scripts)
    module load ...
    source ~/.proteus_profile 
  executables:
    - label: proteus
      instances:
        - parallel_mode:
          command: proteus --n-threads=$MATFLOW_RUN_NUM_THREADS # modify `proteus` as required
          num_cores:
            start: 1
            stop: 100
            step: 1
```
You can also prepend `mpirun` to the `command` if you like. The environment variable `$MATFLOW_RUN_NUM_CORES` contains the number of requested CPU cores for the workflow.

Note: this command should be the proteus invocation command without the argument to the MOOSE input file.
