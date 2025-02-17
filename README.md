# inviz

This tool helps you explore the results of running MCMC posterior sampling on your cosmological model. Selecting a point on the sample distribution will automatically run CLASS on that sample and display the output. 

### Installation
Installation is straightforward with pip:

    pip install inviz
Or, if you want to test the latest changes, you can clone the repository with
    
    git clone https://github.com/wen-jams/inviz
    cd inviz
    python setup.py install
### Dependencies
Currently, only Python versions $\geq$ 3.7 and $<$ 3.11 are supported.  You will also need the Cosmology Boltzmann code CLASS (either the default or your own modified version). Follow the instructions [here](https://cobaya.readthedocs.io/en/latest/theory_class.html) to install classy, the Python wrapper for CLASS.

## Getting Started

### Test Installation
To verify that inviz and all the dependencies have been installed correctly, open a Jupyter Notebook and run:
```python
from inviz import *
hv.extension('bokeh')
pn.extension()
```
If no errors appear, all the dependencies were installed correctly and we're ready to start visualizing!

### Tutorial
For the following example, we're going to try visualizing the testing dataset located in the [data/test_IDM_n_0](data/test_IDM_n_0) folder. This data was created using a custom version of CLASS, which we aren't yet ready to release to the public. So, visualizing this specific set of data on your own won't work. However, if you have your own chains and your own version of CLASS installed, this is what the code would look like:
```python
# load in the data
param_names = load_params('data/test_IDM_n_0/2022-05-04_75000_.paramnames')
df = pd.DataFrame(columns=param_names)
for i in trange(1,56):
    temp = load_data('data/test_IDM_n_0/2022-05-04_75000__{}.txt'.format(i), column_names=param_names)
    df = pd.concat([df,temp]).reset_index(drop=True)
df_slice = df[::500].reset_index(drop=True)

# prepare your data for CLASS computation. this step is specific to your dataset
# remove nuisance parameters
cosmo_df = df.drop(columns=['z_reio', 'A_s', 'sigma8', '100theta_s', 'A_cib_217', 'xi_sz_cib', 'A_sz', 'ps_A_100_100', 'ps_A_143_143', 'ps_A_143_217', 'ps_A_217_217', 'ksz_norm', 
                 'gal545_A_100', 'gal545_A_143', 'gal545_A_143_217', 'gal545_A_217', 'galf_TE_A_100', 'galf_TE_A_100_143', 'galf_TE_A_100_217', 'galf_TE_A_143', 'galf_TE_A_143_217', 
                 'galf_TE_A_217', 'calib_100T', 'calib_217T', 'A_planck'])
cosmo_df['omega_b'] = df['omega_b'] * 1e-2
cosmo_df['sigma_dmeff'] = df['sigma_dmeff'] * 1e-25
cosmo_df = cosmo_df.rename(columns={'H0':'h'})
cosmo_df['h'] = cosmo_df['h'] * 1e-2
cosmo_df['omega_cdm'] = 1e-15
cosmo_df['npow_dmeff'] = 0.0
cosmo_df['Vrel_dmeff'] = 0.0
cosmo_df['dmeff_target'] = 'baryons'
cosmo_df['m_dmeff'] = 1e-3

# format for CDM version
cosmoCDM_df = cosmo_df.drop(columns=['sigma_dmeff', 'omega_cdm', 'npow_dmeff', 'Vrel_dmeff', 'dmeff_target', 'm_dmeff'])
cosmoCDM_df = cosmoCDM_df.rename(columns={'omega_dmeff':'omega_cdm'})

# slice for faster computation. make sure you slice each dataframe the same way
cosmo_df_slice = cosmo_df[::500].reset_index(drop=True)
cosmoCDM_df_slice = cosmoCDM_df[::500].reset_index(drop=True)
df_slice = df[::500].reset_index(drop=True)

# call the visualizer function
viz(df_slice, cosmo_df_slice, cosmoCDM_df_slice, class_enabled=True)
```
And the result should look like this:
![example output](images/example1.png)