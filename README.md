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

The lambda window density were increased for ```complex```, especially for L-J function.

Highlighted red are increased for electrostatic interactions, green are increased for L-J interactions, as you can tell, I insert more states for LJ since this is more likely to have the so called hard core problem, compared to electrostatics.

<img width="920" alt="image" src="https://user-images.githubusercontent.com/75652473/189065665-916a92e9-6652-42c1-8af9-1d77193a3879.png">


the ```default_nsteps_per_iteration``` was incresed from ```500``` to ```1000```

# Combine everything, as listed above.
### Two versions, i.e., explicit solvent and implicit solvent are listed.


```
## References
### [1] https://github.com/choderalab/yank
### [2] https://github.com/choderalab/yank-examples/blob/master/examples/binding/t4-lysozyme/p-xylene-explicit.yaml
