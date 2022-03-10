# Subcortical Reproducibility

This repository holds the Jupyter notebooks for assessing subcortical reproducibility.
Within each notebook, there are related functions to load the files, process the data as necessary, 
create figures and view tractography in 3D via `DIPY`.  All notebooks are linted with `Black` prior
to saving. A full list of imported libraries can be found at the end of each notebook. 

_Note: The embedded table of contents does not seem to work on Github or `jupyter lab` (which
has its own table of contents module). It does however work in `jupyter notebook`_

Every time the notebooks are updated, this repository will also be updated! If the notebook cannot 
be loaded on Git, you may need to try again in a bit. Unfortunately `nbviewer` does not work with 
private repositories.

## Data processing
The data was oriignally processed with a pre-SnakeBIDS version of the [dbsc](https://github.com/kaitj/dbsc) 
workflow. The SnakeBIDS version of this workflow is in development to make processing easier!

## Access the data

The data is available on `Graham` (Compute Canada). If you wish to access the data directly, it is
recommended to create a Python virtual environment and install the necessary dependencies
(including `Jupyter`) via `pip`. To access the full repository, there are a few options.

### 1. On Graham
If you have a Compute Canada account, you can access the data directly after logging
in. The data is stored at the following location: `/scratch/tkai/Zona`. If you do
not have access to this directory, please contact **kaitj (tkai on Khanlab Slack)**.

### 2. Local (`sshfs`)
If you have Graham mounted locally, you should be able to access the data as well. You may need to
a symlink to one of the directories you have mounts (`home` or `scratch`) or mount
the data directly. Here is an example of the mounting command:

```sshfs -o reconnect,ServerAliveInterval=15,ServerAliveCountMax=3,follow_symlinks <user>@graham.computecanada.ca:/scratch/tkai/Zona <Local mounting path>```

### 3. CBS Server
The data is also available via the CBS server. To access the data, follow similar instructions to 
local access. Available to CBS is auto mounting of Graham, requiring only a symlink
to the directory of interest. For full instructions, take a look at the CBS wiki:
https://osf.io/k89fh/wiki/Computational%20Core%20Server/

## File Formats

There are a few file formats specific to the diffusion / tractography data to be aware of:
* `tck` - this is a tractography file format for MRtrix and can be loaded via `DIPY` or `mrview`
* `vtk` - this is a generic file format for 3D models and contain models for ROIs and tractography
(can be loaded with `DIPY`, `Slicer`, `Paraview` or any program that can handle `vtk`)
* `mif` - this is an imaging file format for `MRtrix` and can be loaded via `mrview`

Lastly, due to the large number of files, much of the data has been stored within a tarball 
archive. Any necessary data to view the Jupyter notebooks should already be available
via Graham or have the associated commands within code cells to retrieve files from the archive.

## Environment

If you are accessing the notebooks on a local copy or on `Graham` you will need to set up the 
virtual environment to be able to run the code cells. The easiest and recommended way to do this
is via `poetry` (v1.2.0a2). You set up the environment with the following command:

```
poetry install --with analysis
poetry run python -m ipykernel install --user --name=subcortical_py3
```

After installing the required libraries, fire up a Jupyter instance with the following command
`poetry run jupyter lab`. Make sure you select the `subcuortical_py3` kernel installed!

If you prefer to set up a virtual Python environment (preferably Python 3.7). 
You can use the following block of code to create the necessary virtual environment.

```
# Load Python module if on Graham / CBS
# This is not necessary if running on a personal computer
module load python/3.7

# Replace <venv_dir> with the path to set up the environment
python -m venv <venv_dir> 
source <venv_dir>/bin/activate

# This will install all the necessary libraries into the environment.
pip install -r <requirements.txt>

# Install jupyter and jupyter lab 
pip install jupyter jupyterlab
python -m ipykernel install --user --name=subcortical_py3
```

### Installation Notes
If you are trying to create a multipanel figure to perform QC across subjects,
`matplotlib` will need to be upgraded from `3.3.4` to `3.4.3` (this will break some functionality in
analysis notebooks).

If you are using Poetry, you can edit version listed in `pyproject.toml` from `~3.3.4` to `~3.4.3`. 
After updating, run `poetry update`. You can then run the JupyterLab as before.

If you are using a virtual Python environment, it is easiest to create a new environment. Follow the
instructions above, replacing `pip install -r requirements.txt` with 
`pip install -r  requirements_multipanel.txt`
