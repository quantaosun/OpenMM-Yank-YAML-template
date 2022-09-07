# YAML_example

# A script specially designed for YANK simulation on goolge colab.

### On the basis of https://github.com/choderalab/yank-examples/blob/master/examples/binding/t4-lysozyme/p-xylene-explicit.yaml, here is the modified YAML
### to be used on Google Colab, for a binding free energy simulation of a protein and a bound ligand.

## The simulation is designed to run until the returned dG errror reaches the defined value ``` 1 kT ```, in a way it swaps simulation for ```complex``` ### and ```solvent``` every ```100``` steps. 

# To achieve that goal, there modifications were appliced on top of example above.

## Modification 1, 
### define infinite steps ```.inf```, define checkpoint frequence ``` 50 ```,  and swap simulation between legs ``` 100```

```
options:
  minimize: yes
  verbose: yes
  default_number_of_iterations: 20000 # don't use .inf if sampler error requirement < 1 was set, otherwise an "overflow" error will encountered. 
  default_nsteps_per_iteration: 500 # Increase this if report indicating decorrelation is bad.
  checkpoint_interval: 50
  temperature: 300*kelvin
  pressure: 1*atmosphere # To run with NVT ensemble, set this option to "null". Defaults to 1 atm if excluded
  platform: OpenCL
  output_dir: p-xylene-explicit-output
  switch_phase_interval: 100

```


## Modification 2, 
### insert a ```sampler``` block, after ```system``` but before ```protocal```


```
samplers:
     stop_after_reaching_1:
         type: ReplicaExchangeSampler   # don't need "mcmc-move" since by default it already there.
         replica_mixing_scheme: swap-all  # or "swap-neighbour", since we chose "type" as "ReplicaExchangeSampler" we have to include this.
         online_analysis_interval: 100.   # Increase this if simulation is slow.
         online_analysis_target_error: 1.0 
```
## Modification 3, 
### add ```sampler``` to experiment, after ```protocal``` but before ```restraint```
```
experiments:
  system: t4-xylene
  protocol: absolute-binding
  sampler: stop_after_reaching_1
  restraint:
    type: Harmonic
```

# Combine everything together, below is the full script for the simulation.

```
# Set the general options of our simulation
options:
  minimize: yes
  verbose: yes
  default_number_of_iterations: 20000
  default_nsteps_per_iteration: 500
  checkpoint_interval: 50
  temperature: 300*kelvin
  pressure: 1*atmosphere 
  platform: OpenCL
  output_dir: p-xylene-explicit-output
  switch_phase_interval: 100

# Configure the specific molecules we will use for our systems
# Note: We do not specify what the "receptor" and what the "ligand" is yet
molecules:
  # Define our receptor, T4-Lysozyme, we can call it whatever we want so we just use its name here as the directive
  t4-lysozyme:
    filepath: input/receptor.pdbfixer.pdb
  # Define our ligand molecule
  p-xylene:
    filepath: input/ligand.tripos.mol2
    # Get the partial charges for the ligand by generating them from antechamber with the AM1-BCC charge method
    antechamber:
      charge_method: bcc

# Define the solvent for our system, here we use Particle Mesh Ewald for long range interactions
solvents:
  # We can title this solvent whatever we want. We just call it "pme" for easy remembering
  pme:
    nonbonded_method: PME # Main definition of the nonbonded method
    nonbonded_cutoff: 9*angstroms # Cutoff between short- and long-range interactions
    # Define how far away the periodic boundaries are from the receptor.
    # The volume will be filled with TIP3P water through LEaP
    clearance: 16*angstroms
    # If ions are needed to neutralize the system, add these specific ions
    positive_ion: Na+
    negative_ion: Cl-

# Define the systems: What is the ligand, receptor, and solvent we put them in
systems:
  # We can call our system anything we want, this example just uses a short name for the receptor hyphenated with the ligand
  t4-xylene:
    # These names all use the names we defined previously
    receptor: t4-lysozyme
    ligand: p-xylene
    solvent: pme
    leap:
      parameters: [leaprc.protein.ff14SB, leaprc.gaff2, leaprc.water.tip4pew]
      
 # Define the Sampler, set error requirement before stop the phase simulaiton
 
samplers:
     stop_after_reaching_1:
         type: ReplicaExchangeSampler
         replica_mixing_scheme: swap-all
         online_analysis_interval: 100
         online_analysis_target_error: 1.0

# The protocols define the alchemical path each phase will take, we use the same lambda values, though they could be different
protocols:
  # Call the protocol whatever you would like, here we name it based on the type of calculation we are running
  absolute-binding:
    complex:
      alchemical_path:
        lambda_electrostatics: [1.00, 1.00, 1.00, 1.00, 1.00, 0.90, 0.80, 0.70, 0.60, 0.1000, 0.40, 0.30, 0.20, 0.10, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00]
        lambda_sterics:        [1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 0.90, 0.80, 0.70, 0.60, 0.1000, 0.40, 0.30, 0.20, 0.10, 0.00]
        # Set lambda restraints reverse of coupling parameter (see below in "restraint" for reason)
        lambda_restraints:     [0.00, 0.25, 0.1000, 0.75, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00]
    solvent:
      alchemical_path:
        lambda_electrostatics: [1.00, 0.90, 0.80, 0.70, 0.60, 0.1000, 0.40, 0.30, 0.20, 0.10, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00]
        lambda_sterics:        [1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 0.90, 0.80, 0.70, 0.60, 0.1000, 0.40, 0.30, 0.20, 0.10, 0.00]

# Here we combine the system and the protocol to make an experiment
experiments:
  system: t4-xylene
  protocol: absolute-binding
  sampler: stop_after_reaching_1
  restraint:
    type: Harmonic # Keep the ligand near the protein
