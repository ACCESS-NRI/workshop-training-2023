# ACCESS-NRI 2023 Workshop Introduction to ESMValTool

This exercise will introduce you to <a href="https://www.esmvaltool.org/" target="_blank">ESMValTool</a>, a tool to evaluate Earth System Models (ESMs) against observations like those available through the <a href="https://esgf.llnl.gov/index.html" target="_blank">Earth System Grid Federation (ESGF)</a>.

In this document, we will use the **Virtual Desktop Infrastructure (VDI)**. We also provide an adjusted Jupyter version [for advanced users of ESMValTool](./Introduction_to_ESMValTool.ipynb).

Running ESMValTool on <i>Gadi</i> is supported by ACCESS-NRI with further information on <a href="https://access-hive.org.au/model_evaluation/model_evaluation_on_gadi/model_evaluation_on_gadi_esmvaltool/" target="_blank">this ACCESS-Hive page</a> to supplement the <a href="https://docs.esmvaltool.org/en/latest/" target="_blank">official ESMValTool documentation</a>.

## Step 0: Pre-workshop
To run this exercise, you need to be a member of the following NCI projects:
```
nf33, xp65, al33, rr3, r87
```

## Step 1:
Go to the [Australian Research Environment](https://are.nci.org.au/) website and login with your **NCI username and password**. If you don't have an NCI account, you can sign up for one at the [NCI website](https://my.nci.org.au/mancini/login?next=/mancini/).

<p align="center"><img src="../assets/ARE_setup_guide/setup_image1.png" alt="drawing" width="50%"/></p>

## Step 2:
Click on `Virtual Desktop` under *Featured Apps* to configure a new VDI instance. This option is also available under the *All Apps* section at the bottom of the page and the *Interactive Apps* dropdown located in the top menu.

<p align="center"><img src="../assets/access_rose_cylc/setup_vdi1.png" alt="drawing" width="50%"/></p>

## Step 3:
You will now be presented with the main VDI instance configuration form. Please complete **only** the fields below - leave all other fields blank or to their default values.

- *3.1* **Walltime**: The number of hours the VDI instance will run. `1` hour is sufficient for each of the tutorials.

<p align="center"><img src="../assets/ARE_setup_guide/setup_image3.png" alt="drawing" width="50%"/></p>

- *3.2* **Compute Size**: Select `small(2 cpus, 9 mem)` from the dropdown menu.

<p align="center"><img src="../assets/ARE_setup_guide/setup_image4.png" alt="drawing" width="50%"/></p>

- *3.3* **Project**: Please enter `nf33`. This will allocate SU usage to the workshop project.

<p align="center"><img src="../assets/ARE_setup_guide/setup_image5.png" alt="drawing" width="50%"/></p>

- *3.4* **Storage**: This is the list of `/g/data/` project data storage locations required to complete the workshop tutorials. In ARE, storage locations need to be explicitly defined to access these data from within a VDI instance. Please enter the following string:
```
gdata/nf33+gdata/xp65
```

<p align="center"><img src="../assets/ARE_setup_guide/setup_image6_1.png" alt="drawing" width="50%"/></p>

- *3.5* Click `Advanced options ...`

- *3.6* **PBS Flags**
The **xp65** conda environment is a containerised environment that requires the `SINGULARITY_OVERLAYIMAGE` environment variable to be defined.
Copy and paste the following:
```
-v SINGULARITY_OVERLAYIMAGE=/g/data/xp65/public/apps/med_conda/envs/access-med-0.3.sqsh
```
in the **PBS Flags** field of the **advanced options** section:

<p align="center"><img src="../assets/ILAMB/pbsflag.png" alt="drawing" width="50%"/></p>

## Step 4

Once the VDI instance has started (this usually takes around 30 seconds) and this status window should update and look something like the following, reporting that the instance has started and the time remaining. More detailed information on the instance can be accessed by clicking the Session ID link.

<p align="center"><img src="../assets/ILAMB/running.png" alt="drawing" width="50%"/></p>

All that remains to get started is to click `Launch VDI Desktop`.

## Suggestion: Copy + paste from your local machine to VDI

- click on the control bar in the center left of the VDI window
- click on the clipboard: you can copy text from your local machine into this with the usual shortkeys
- right-click and click *Paste* to paste the content in VDI

<p align="center"><img src="../assets/ARE_setup_guide/vdi_copy_paste.png" alt="drawing" width="50%"/></p>

## Step 5
Start a terminal in the VDI session.

<p align="center"><img src="../assets/ILAMB/vdi_desktop.png" alt="drawing" width="60%"/></p>


Then open a terminal, change the directory to your directory in this training section

```
cd /scratch/nf33/$USER
```

## Step 6
In this directory, we need you to clone the whole repo from GitHub with the command below (if you already have this repo in your directory, you can jump to STEP 7):

```
git clone https://github.com/ACCESS-NRI/workshop-training-2023.git
```

<p align="center"><img src="../assets/ILAMB/gitclone.png" alt="drawing" width="60%"/></p>

Then you are all set to start the exercises.

### Step 7: Move to the `esmvaltool` training directory

In the terminal, prompt:
```bash
cd /scratch/nf33/$USER/workshop-training-2023/esmvaltool
```

### Step 8: Check the ESMValTool environment by accessing the help for ESMValTool

```bash
module use /g/data/xp65/public/modules
module load conda/access-med

esmvaltool --help
```

Prompting this help command should produce the following output:

![Screenshot of the terminal when prompting esmvalltool with the help argument](../assets/ESMValTool/esmvaltool_help.png)

### Step 9: The configuration file

In the next step, we want to have a look at the esmvaltool configuration file that we will use in this tutorial. You can use a text editor of your choice. In this tutorial, we will simply print the content via `more`:

```bash
more config-user-on-gadi-v2.9.yml
```

This file contains the information for:

- Output settings
- Destination directory
- Download and auxiliary data directories
- Number of tasks that can be run in parallel
- Rootpath to input data
- Directory structure for the data from different projects

**KEY POINTS**

- The `config-user-on-gadi-v2.9.yml` tells ESMValTool where to find input data.
- `output_dir` defines the destination directory.
- `rootpath` defines the root path of the data.
- `drs` defines the directory structure of the data.

#### Output settings

The configuration file starts with output settings that inform ESMValTool about your preference for output. You can turn on or off the setting by true or false values. Most of these settings are fairly self-explanatory.

#### Destination directory

The destination directory is the rootpath where ESMValTool will store its output folders containing e.g. figures, data, logs, etc. With every run, ESMValTool automatically generates a new output folder determined by recipe name, and date and time using the format: YYYYMMDD_HHMMSS.

```yaml
# Destination directory where all output will be written
# Includes log files and performance stats.
output_dir: esmvaltool_output
```

#### Rootpath to input data

ESMValTool uses several categories (in ESMValTool, this is referred to as projects) for input data based on their source. The current categories in the configuration file are mentioned below. For example, CMIP is used for a dataset from the Climate Model Intercomparison Project whereas OBS may be used for an observational dataset. More information about the projects used in ESMValTool is available in the official <a href="https://docs.esmvaltool.org/en/latest/" target="_blank">ESMValTool documentation</a>. When using ESMValTool on your own machine, you can create a directory to download climate model data or observation data sets and let the tool use data from there. It is also possible to ask ESMValTool to download climate model data as needed. This can be done by specifying a download directory and by setting the option to download data as shown below.

#### Directories for downloading climate data and auxiliary data

```yaml
# Directory for storing downloaded climate data and find auxiliary data
download_dir: esmvaltool_climate_data
auxiliary_data_dir: /g/data/xp65/public/apps/cartopy-data
search_esgf: never
```

If you are working offline or do not want to download the data then set the option above to `never`. If you want to download data only when the necessary files are missing at the usual location, you can set the option to `when_missing`. In particular, `cartopy` will be needed as auxiliary data for several plots. We provide them through `xp65` as shown above.

The `rootpath` specifies the directories where ESMValTool will look for input data. For each category, you can define either one path or several paths as a list. For example:

```yaml
# Rootpaths to the data from different projects
# This default setting will work if files have been downloaded by the
# ESMValTool via ``offline=False``. Lists are also possible. For site-specific
# entries and more examples, see below. Comment out these when using a
# site-specific path.
rootpath:
  default: esmvaltool_climate_data
  CMIP5: [/g/data/r87/DRSv3/CMIP5, /g/data/al33/replicas/CMIP5/combined, /g/data/rr3/publications/CMIP5/output1]
  native6: /g/data/nf33/public/data/ESMValTool/obsdata
```
#### Directory structure for the data from different projects

Input data can be from various models, observations and reanalysis data that adhere to the CF/CMOR standard.

The `drs` setting describes the file structure for several projects (e.g. CMIP6, CMIP5, obs4mips, OBS6, OBS) on several key machines (e.g. BADC, CP4CDS, DKRZ, ETHZ, SMHI, BSC, NCI). For more information about `drs`, you can visit the ESMValTool documentation on <a href="https://docs.esmvaltool.org/projects/ESMValCore/en/latest/quickstart/find_data.html#data-types" target="_blank">Data types and the Data Reference Syntax (DRS)</a>.

```yaml
# Directory structure for input data --- [default]/ESGF/BADC/DKRZ/ETHZ/etc.
# This default setting will work if files have been downloaded by the
# ESMValTool via ``offline=False``. See ``config-developer.yml`` for
# definitions. Comment out/replace as per needed.
drs:
  CMIP5: BADC
```

## Step 10: The ESMValTool recipe

To see all the recipes that are shipped with ESMValTool, type

```bash
esmvaltool recipes list
```

![Collage of screenshots of the terminal window when printing the available ESMValTool recipes list on Gadi.](../assets/ESMValTool/esmvaltool_recipe_list.png)

For this tutorial, we will choose `recipe_climwip_test_basic.yml` as an example recipe. ACCESS-NRI is working on supporting all the above recipes on NCI. You can check the current support status [here](https://github.com/ACCESS-NRI/ESMValTool-workflow).

Use the following command to copy the recipe to your working directory

```bash
esmvaltool recipes get recipe_climwip_test_basic.yml
```

Now you should see the recipe file in your working directory (type `ls` to verify). Use your text editor to open this file or display the contents via `more`:

```bash
more recipe_climwip_test_basic.yml
```
Have a look at the recipe structure:

- Documentation with relevant (citation) information
- Datasets that should be analysed
- Preprocessors groups of common preprocessing steps
- Diagnostics scripts performing more specific evaluation steps

## Climate model Weighting by Independence and Performance (ClimWIP)

Projections of future climate change are often based on multi-model
ensembles of global climate models such as CMIP6. To condense the
information from these models they are often combined into
probabilistic estimates such as mean and a related uncertainty range
(such as the standard deviation). However, not all models in a given
multi-model ensemble are always equally fit for purpose and it can
make sense to weight models based on their ability to simulate
observed quantities related to the target. In addition, multi-model
ensembles, such as CMIP can contain several models based on a very
similar code base (sharing of components, only differences in
resolution, etc.) leading to complex inter-dependencies between the
models. Adjusting for this by weighting models according to their
independence helps to adjust for this.


This recipe implements the **Climate model Weighting by Independence and Performance
(ClimWIP)** method. It is based on work by [Knutti et al. (2017)](https://doi.org/10.1002/2016GL072012),
[Lorenz et al. (2018)](https://doi.org/10.1029/2017JD027992),
[Brunner et al. (2019)](https://doi.org/10.1088/1748-9326/ab492f),
[Merrifield et al. (2020)](https://doi.org/10.5194/esd-11-807-2020),
[Brunner et al. (2020)](https://doi.org/10.5194/esd-11-995-2020). Weights are
calculated based on historical model performance in several metrics (which can be
defined by the ``performance_contributions`` parameter) as well as by their independence
to all the other models in the ensemble based on their output fields in several metrics
(which can be defined by the ``independence_contributions`` parameter). These weights
can be used in subsequent evaluation scripts (some of which are implemented as part of
this diagnostic).

**Note**: This recipe is still being developed! A more comprehensive (yet older)
implementation can be found on GitHub:  https://github.com/lukasbrunner/ClimWIP

## Step 11: Run a recipe inside a PBS Job

Because of the computational costs, we will submit a job to Gadi through the Portable Batch System. To do so, you need to use a submission script, for example the one that we already provide. Open the `launch_recipe_climwip_test_basic.pbs` file:

```bash
#!/bin/bash -l 

# For help with PBS directives on Gadi, go to https://opus.nci.org.au/display/Help/PBS+Directives+Explained
#PBS -S /bin/bash
#PBS -P nf33
#PBS -l storage=gdata/rr3+gdata/xp65+gdata/al33+gdata/nf33+scratch/nf33
#PBS -N recipe_climwip_test_basic
#PBS -l wd
#PBS -q normal
#PBS -l walltime=01:00:00
#PBS -l mem=64GB
#PBS -l ncpus=10

module use /g/data/xp65/public/modules
module load conda/access-med

esmvaltool run --config_file config-user-on-gadi-v2.9.yml recipe_climwip_test_basic.yml
```

Submit the job to the queue system:

```bash
qsub launch_recipe_climwip_test_basic.pbs
```

To monitor the progress, you can use the status prompt for the job ID
```
qstat
```

## Step 12: Investigating the log messages

Once the job is finished, you can open the log message (`recipe_climwip_test_basic.o*`) and check a few things:

After the banner and general information, the output starts with some important locations.

- Did ESMValTool use the right config file?
- What is the path to the example recipe?
- What is the main output folder generated by ESMValTool?
- Can you guess what the different output directories are for?
- ESMValTool creates two log files. What is the difference?

## Step 13: Visualise outputs with a VDI

Open a new terminal (top left of the VDI screen) and navigate to the `esmvaltool_output` directory, them use the commmand below to start a local  HTTP server.

```bash
cd /scratch/nf33/$USER/workshop-training-2023/esmvaltool/esmvaltool_output
python3 -m http.server
```

You can then start Firefox in the VDI screen and access the following localhost address to navigate into your specific `recipe*` directory and its `index.html`:

```
http://0.0.0.0:8000/
```

From there you can navigate to through the different directories to show the different evaluation plots:

![Screenshot of the VDI browser window showing results of the ESMValTool comparison](../assets/ESMValTool/esmvaltool_results_1.png)

![Screenshot of the VDI browser window showing results of the ESMValTool comparison](../assets/ESMValTool/esmvaltool_results_2.png)

![Screenshot of the VDI browser window showing results of the ESMValTool comparison](../assets/ESMValTool/esmvaltool_results_3.png)

## Step 14: Close servers and VDI session

- Close the browser window
- Close the `http` server by prompting `ctrl+C` in the terminal, then prompt `exit` to close the terminal
- In the menu bar (top left), click on `System` and then `Log Out` and close the browser tab or delete the session in *My Interactive Sessions* of the ARE
