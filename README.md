# Alchemical Transfer Method

Input files and instructions to run the Alchemical Transfer simulations for the estimation of binding free energies of small to medium sized solute molecules using OpenMM described in [Alchemical Transfer Approach to Absolute Binding Free Energy Estimation](https://arxiv.org/abs/2101.07894), currently under review in the Journal of Chemical Theory and Computation.

## Contributors

Joe Wu [jwu1@gradcenter.cuny.edu](jwu1@gradcenter.cuny.edu)

Solmaz Azimi [sazimi@gradcenter.cuny.edu](sazimi@gradcenter.cuny.edu)

Sheenam Khuttan [ssheenam@gradcenter.cuny.edu](ssheenam@gradcenter.cuny.edu)

Emilio Gallicchio [egallicchio@brooklyn.cuny.edu](egallicchio@brooklyn.cuny.edu)

## Credits

The repository is maintained by the Gallicchio's laboratory in the Department of Chemistry at Brooklyn College of CUNY [(compmolbiophysbc.org)](compmolbiophysbc.org). The work is supported in part from a grant from the National Science Foundation (CAREER 1750511).


## Documentation and Tutorials

The project walks you over the steps to setup and run alchemical absolute binding free energy calculations for a series of host-guest complexes in explicit water (TIP3P model) using the [Single Decoupling (SDM) OpenMM's plugin](https://github.com/rajatkrpal/openmm_sdm_plugin). Instructions for gathering and installing software cools can be obtained by following the [SDM Workflow](https://github.com/Gallicchio-Lab/openmm_sdm_workflow).

## Run the ASyncRE simulations

Directories of the SAMPL6 host-guest complexes are available in the `complexes` repository.

Go to the simulation directories of each complex and launch the ASyncRE simulations. For example, assuming ASyncRE is installed under ```$HOME/devel/async_re-openmm```:

To run simulation of the OA-G6-0 complex, go to directory `complexes/OA-G6-0/OA-G6-0-leg1` and do:

```
./runopenmm $HOME/devel/async_re-openmm/bedamtemptpbconedmsfile_async_re.py hg-oag6-0_asyncre.cntl
```

Adjust the settings in the ```runopenmm``` script to reflect your environment.

Set the `nodefile` to point to a set of GPUs on the local machine. For example:

```
locahost,0:0,1,OpenCL,,/tmp
locahost,0:1,1,OpenCL,,/tmp
locahost,0:2,1,OpenCL,,/tmp
locahost,0:3,1,OpenCL,,/tmp
```

for a compute server with 4 GPUs when the OpenCL platform is the first platform (0).

Each Replica Exchange (RE) simulation is set to run for a total time of ```WALL_TIME``` x ```CYCLE_TIME```. For example, a ```WALL_TIME``` OF 72 minutes each cycle x ```CYCLE_TIME``` of 10 cycles will run for 720 total minutes. You can change the duration of the simulation run by changing the ```WALL_TIME``` and/or ```CYCLE_TIME``` in the ```*_asyncre.cntl``` file present in each complex directory.
Monitor the progress of each ASyncRE simulation by inspecting the ```*_stat.txt``` file. For example:

```
cd $HOME/complexes/OA-G6-0/OA-G6-0-leg1/
cat hg-oag6-0_stat.txt
```
Each replica runs in a separate subdirectory named ```r0```, ```r1```, etc. Each cycle generates ```.out```, ```.dms```, ```.log```, ```.pdb```, and ```.dcd``` files. For example the binding energy samples for the first cycle of replica 2 are stored in the file ```r2/hg-oag6-0.out``` . The .dms files are used to start the following cycle. The .pdb and .dcd files are used for trajectory visualization.
 
The ASyncRE process can be killed at any time with ```^C``` and optionally restarted. However, replicas currently running on remote machines are likely to keep running and may have to be killed before ASyncRE can be restarted. To start from scratch (that is from the first cycle) remove all of the replicas directories by doing ```rm -r r? r??```.

## Output

Once the simulation has started, replica directories pointing at each lambda value in the alchemical schedule are produced. 
In each replica directory, a  ```*.out``` is created, producing columns of values in the order: temperature, lambda, lambda1, lambda2, alpha, u0, w0, total potential energy, binding energy.

This output file can then be used to analyze the average value of different parameters over the course of simulation.

Additionally, to visualize the trajectories of the solute-complex binding, ```*.dcd``` files are produced which can be used along with the ```*.dms``` file in the complex directory with VMD.

```
cd $HOME/complexes/OA-G6-0/OA-G6-0-leg1/r0
vmd -f hg-oag6-0_ckp.dms hg-oag6-0.dcd
```
