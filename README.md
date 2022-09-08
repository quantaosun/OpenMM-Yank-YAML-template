# YAML_example

# A script template to be used on goolge colab for the ```Yank``` package [1].

## It is designed to run the simulation until the returned dG error is smaller than ``` 1 kT ```, it swaps simulation for ```complex``` and ```solvent``` every ```100``` steps. 

# To achieve that goal, 3 modifications were appliced to this example [2].

## Modification 1, 
### define a large number of steps ```200000```, define checkpoint frequency ``` 50 ```,  and swap frequency between legs ``` 100```

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
### Insert a ```sampler``` block, after ```system```, before ```protocal```


```
samplers:
     stop_after_reaching_1:
         type: ReplicaExchangeSampler   # doesn't need "mcmc-move", by default, it already there.
         replica_mixing_scheme: swap-all  # or "swap-neighbour", since we chose "type" as "ReplicaExchangeSampler" we have to include this.
         online_analysis_interval: 100.   # Increase this if the simulation is slow.
         online_analysis_target_error: 1.0 
```
## Modification 3, 
### Include ```sampler``` to experiment, after ```protocol```, before ```restraint```
```
experiments:
  system: t4-xylene
  protocol: absolute-binding
  sampler: stop_after_reaching_1
  restraint:
    type: Harmonic
```
## Other Modifications

The lambda windows are doubled for ```complex```

the ```default_nsteps_per_iteration``` was incresed from ```500``` to ```1000```

# Combine everything. Below is the full script.

```
# Set the general options of our simulation
options:
  minimize: yes
  verbose: yes
  default_number_of_iterations: 200000
  default_nsteps_per_iteration: 1000
  checkpoint_interval: 50
  temperature: 300*kelvin
  pressure: 1*atmosphere # To run with NVT ensemble, set this option to "null". Defaults to 1 atm if excluded
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

samplers:
     stop_after_reaching_1:
         type: ReplicaExchangeSampler
         replica_mixing_scheme: swap-all
         online_analysis_interval: 300
         online_analysis_target_error: 1.0

# The protocols define the alchemical path each phase will take, we use the same lambda values, though they could be different
protocols:
  # Call the protocol whatever you would like, here we name it based on the type of calculation we are running
  absolute-binding:
    complex:
      alchemical_path:
        lambda_electrostatics: [1.00, 1.00, 1.00, 1.00, 1.00, 0.95, 0.90, 0.85, 0.80, 0.75, 0.70, 0.65, 0.60, 0.55, 0.50, 0.45, 0.40, 0.35, 0.30, 0.25, 0.20, 0.15, 0.10, 0.05, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00]
        lambda_sterics:        [1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 0.95, 0.90, 0.85, 0.80, 0.75, 0.70, 0.65, 0.60, 0.55, 0.50, 0.45, 0.40, 0.35, 0.30, 0.25, 0.20, 0.15, 0.10, 0.05, 0.00]
        # Set lambda restraints reverse of coupling parameter (see below in "restraint" for reason)
        lambda_restraints:     [0.00, 0.25, 0.50, 0.75, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00]
    solvent:
      alchemical_path:
        lambda_electrostatics: [1.00,  0.90, 0.80, 0.70, 0.60, 0.50, 0.40, 0.30, 0.20, 0.10, 0.05, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00, 0.00]
        lambda_sterics:        [1.00,  1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 1.00, 0.90, 0.80, 0.70, 0.60, 0.50, 0.40, 0.30, 0.20, 0.10, 0.00]

# Here we combine the system and the protocol to make an experiment
experiments:
  system: t4-xylene
  protocol: absolute-binding
  sampler: stop_after_reaching_1
  restraint:
    type: Harmonic # Keep the ligand near the protein
```
## References
### [1] https://github.com/choderalab/yank
### [2] https://github.com/choderalab/yank-examples/blob/master/examples/binding/t4-lysozyme/p-xylene-explicit.yaml
