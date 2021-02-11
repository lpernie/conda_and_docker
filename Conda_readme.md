#### INFO
conda info

#### List
conda env list

#### Create from yml
conda env create -f environment_plus2.yaml python=3.9

#### Activate
conda activate basic

#### Deactivate
conda deactivate

#### Remove
conda remove --name plus2 --all

#### Force remove
conda clean --all

#### Update
conda env update -f environment_basic.yaml

#### Print yaml file
conda env export --name plus > plus.yaml

#### Add a pip package to an existing environment
conda activate plus2   
/opt/anaconda3/envs/plus2/bin/pip install pandasql
