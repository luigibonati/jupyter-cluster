# Jupyter on Cluster

Modified version of the [Jupyter on Euler](https://gitlab.ethz.ch/sfux/Jupyter-on-Euler-or-Leonhard-Open) script to connect to a Jupyter notebook running on an HPC cluster.  

Adapted to work with a generic cluster and to support the PBS queue system (see detailed changelog in python file).

The original Jupyter on Euler is appended below (note that some of the instructions are specific to the ETH Euler cluster).

--- 

### _Original README_
This project aims to help beginner users to run simple jupyter notebooks on our HPC cluster Euler. It is not addressing advanced users that need a wide range of additional features going beyond simple jupyter notebooks.

When you run this shell script on your local computer, then it starts a Jupyter notebook in a batch job on Euler and connects your local browser with it.

### Requirements

The script assumes that you have setup SSH keys for passwordless access to the cluster. Please find some instructions on how to create SSH keys on the scicomp wiki:

https://scicomp.ethz.ch/wiki/Accessing_the_clusters#SSH_keys

Currently the script runs on Linux, Mac OS X and Windows (using WSL/WSL2 or git bash). When using a Linux computer, please make sure that xdg-open is installed. This package is used to automatically start your default browser. You can install it with the command

CentOS:

```
yum install xdg-utils
```

Ubuntu:

```
apt-get install xdg-utils
```

### Security token vs. password setup
Please note that a part of the script (parsing of the ports) requires that you use jupyter notebooks with the security tokens. If you configure a password instead, such that you can use the jupyter notebook without the security token, then the script will not work anymore (it cannot parse the port on the remote compute node) without adapting it.

### Using SSH keys with non-default names
Since the reopening of Euler after the cyber attack in May 2020, we recommend to the cluster users to use SSH keys.
```
$HOME/.ssh/id_ed25519_euler
```

You can either use the -k option of the script to specify the location of the SSH key, or even better use an SSH config file with the IdentityFile option

https://scicomp.ethz.ch/wiki/Accessing_the_clusters#How_to_use_keys_with_non-default_names

I would recommend to use the SSH config file as this works more reliably.

### Usage

#### Install

Download the repository with the commnad

```
git clone https://gitlab.ethz.ch/sfux/Jupyter-on-Euler-or-Leonhard-Open
```

Mac OS X:

```
git clone https://gitlab.ethz.ch/sfux/Jupyter-on-Euler-or-Leonhard-Open.git
```

After downloading the script from gitlab.ethz.ch, you need to change its permissions to make it executable

```
cd Jupyter-on-Euler-or-Leonhard-Open/
chmod 755 start_jupyter_nb.sh
```

#### Software stack
Please note that currently the old software stack is still set a default (this will change). The script is using the new software stack (unless you explicitly request the old software stack with the option -s old (or --softwarestack old). Therefore please make sure that you set the new software stack as permanent default by using the command

```
set_software_stack.sh new
```

You can find more information about this script on our wiki:

```
https://scicomp.ethz.ch/wiki/Setting_permanent_default_for_software_stack_upon_login
```

#### Run Jupyter in a batch job

The start_jupyer_nb.sh script needs to be executed on your local computer. Please find below the list of options that can be used with the script:

```
$ ./start_jupyter_nb.sh --help
./start_jupyter_nb.sh: Script to start jupyter notebook/lab on Euler from a local computer

Usage: start_jupyter_nb.sh [options]

Required options:

        -u | --username       USERNAME         ETH username for SSH connection to Euler

Optional arguments:

        -b | --batch_sys      BATCHSYS         Batch system to use (LSF or SLURM)
        -c | --config         CONFIG_FILE      Configuration file for specifying options
        -g | --numgpu         NUM_GPU          Number of GPUs to be used on the cluster
        -h | --help                            Display help for this script and quit
        -i | --interval       INTERVAL         Time interval for checking if the job on the cluster already started
        -j | --julia          BOOL             Start jupyter notebook with (BOOL=TRUE) or without (BOOL=FALSE) julia kernel enabled
        -k | --key            SSH_KEY_PATH     Path to SSH key with non-standard name
        -l | --lab                             Start jupyter lab instead of a jupyter notebook
        -m | --memory         MEM_PER_CORE     Memory limit in MB per core
        -n | --numcores       NUM_CPU          Number of CPU cores to be used on the cluster
        -s | --softwarestack  SOFTWARE_STACK   Software stack to be used (old, new)
        -v | --version                         Display version of the script and exit
        -w | --workdir        WORKING_DIR      Working directory for the jupyter notebook/lab
             --extra-modules  EXTRA_MODULES    Load additional cluster modules before starting
             --module-use     MODULE_USE       Use additional cluster module collection before starting
             --pythonpath     PYTHONPATH       Set PYTHONPATH before starting
        -W | --runtime        RUN_TIME         Run time limit for jupyter notebook/lab in hours and minutes HH:MM

Examples:

        ./start_jupyter_nb.sh -u sfux -b SLURM -n 4 -W 04:00 -m 2048 -w /cluster/scratch/sfux

        ./start_jupyter_nb.sh -u sfux -b SLURM -n 1 -W 01:00 -m 1024 -j TRUE

        ./start_jupyter_nb.sh --username sfux --batch_sys SLURM --numcores 2 --runtime 01:30 --memory 2048 --softwarestack new

        ./start_jupyter_nb.sh -c /c/Users/sfux/.jnb_config

Format of configuration file:

JNB_USERNAME=""             # ETH username for SSH connection to Euler
JNB_BATCH="SLURM"           # Choose SLURM or LSF as batch system
JNB_EXTRA_MODULES           # Additional modules to be loaded
JNB_NUM_CPU=1               # Number of CPU cores to be used on the cluster
JNB_NUM_GPU=0               # Number of GPUs to be used on the cluster
JNB_RUN_TIME="01:00"        # Run time limit for jupyter notebook/lab in hours and minutes HH:MM
JNB_MEM_PER_CPU_CORE=1024   # Memory limit in MB per core
JNB_WAITING_INTERVAL=60     # Time interval to check if the job on the cluster already started
JNB_SSH_KEY_PATH=""         # Path to SSH key with non-standard name
JNB_SOFTWARE_STACK="new"    # Software stack to be used (old, new)
JNB_WORKING_DIR=""          # Working directory for jupyter notebook/lab
JNB_ENV=""                  # Path to virtual environment
JNB_JLAB=""                 # "lab" -> start jupyter lab; "" -> start jupyter notebook
JNB_JKERNEL="FALSE"         # "FALSE" -> no Julia kernel; "TRUE" -> Julia kernel

```

#### Reconnect to a Jupyter notebook
When running the script, it creates a local file called reconnect_info in the installation directory, which contains all information regarding the used ports, the remote ip address, the command for the SSH tunnel and the URL for the browser. This information should be sufficient to reconnect to a Jupyter notebook if connection was lost.

#### Running multiple notebooks in a single Jupyter instance
If you run Jupyter on the Leonhard cluster, using GPUs, then you need to make sure a notebook is correctly terminated before you can start another one.

If you don't properly close the first notebook and run a second one, then the previous notebook will still occupy some GPU memory and have processes running, which will throw some errors, when executing the second notebook.

Therefore please make sure that you stop running kernels in the "running" tab in the browser, before starting a new notebook.

#### Terminate the Jupyter session

Please note that when you finish working with the jupyter notebook, you need to click on the "Quit" or "Logout" button in your Browser. "Quit" will stop the batch job running on Euler, "Logout" will just log you out from the session but not stop the batch job (in this case you need to login to the cluster, identify the job with bjobs and then kill it with the bkill command, using the jobid as parameter). Afterwards you also need to clean up the SSH tunnel that is running in the background. Example:

```
samfux@bullvalene:~/Jupyter-on-Euler-or-Leonhard-Open$ ps -u | grep -m1 -- "-L" | grep -- "-N"
samfux    8729  0.0  0.0  59404  6636 pts/5    S    13:46   0:00 ssh sfux@euler.ethz.ch -L 51339:10.205.4.122:8888 -N
samfux@bullvalene:~/jupyter-on-Euler-or-Leonhard-Open$ kill 8729
```

#### Additional kernels

When using this script, you can either use the Python 3.6 Kernel, or in addition a bash kernel or an R kernel (3.6.0 on Euler, 3.5.1 on Leonhard Open)

#### Installing additional Python and R packages locally

When starting a Jupyter notebook with this script, then it will use a central Python and R installation:

```
Old software stack: module load new gcc/4.8.2 python/3.6.1
New software stack: module load gcc/6.3.0 python/3.8.5
```

Therefore you can only use packages that are centrally installed out-of-the-box. But you have the option to install additional packages locally in your home directory, which can afterwards be used.

For installing a Python package from inside a Jupyter notebook, you would need to run the following command:

```
!pip install --user package_name
```

This will install <tt>package_name</tt> into <tt>$HOME/.local</tt>, as described on our wiki page about Python:

```
https://scicomp.ethz.ch/wiki/Python#Installing_a_Python_package.2C_using_PIP
```

The command to locally install an R package:

```
install.packages("package_name")
```

Then follow the instructions provided on our wiki:

```
https://scicomp.ethz.ch/wiki/R#Extensions
```

### Main author
* Samuel Fux

### Contributions
* Andrei Plamada
* Urban Borstnik
* Steven Armstrong
* Swen Vermeul
* Jarunan Panyasantisuk
* Gül Sena Altıntaş
* Mikolaj Rybinski

