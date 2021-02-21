# UTH-Timer
"Gate delay estimation with library compatible CSMs and multiple Ceff", The TimeKeepers, University of Thessaly, Greece

## Introduction

This README provides details on how to use UTH-Timer, TimeKeepers' CCS delay calculation toolkit.

This toolkit is an extended version of the [TAU 2020 contest Delay Calculator Toolkit (DCTK)](https://github.com/geochrist/dctk/tree/tau2020).

## Disclaimer
* This version of the tool supports only the [ASU ASAP 7nm predictive PDK](https://github.com/The-OpenROAD-Project/asap7) for demonstration purposes. It is not guaranteed that the toolkit functions properly with other libraries.

* ASU ASAP7 library contains SPICE models and gate netlists that may not function properly with Xyce simulator. Cadence SPECTRE simulator is mandatory in order to properly simulate all kind of standard-cells.

* Note that the SPICE models are level 72 (BSIM-CMG v107 MOSFET). To get these models to work with Xyce, the level value has been changed to 107.

## Scripts

### ./generate_nets

This csh script generates the dataset (Circuits YAML, SPEF, SPICE decks) with the ASU library:

[Executes gen_random_nets binary]
* generates a testsuite with n random circuits (YAML and SPEF files)

[Executes delay_calc_tool binary]
* generates SPICE decks for the testsuite


### ./run_delay_calculator

This csh script reads the results from ./generate_nets script (Circuits YAML, SPEF, SPICE decks), computes the delays/slews for all circuits using UTH-Timer CCS delay calculator and SPICE simulation, merges the results, and writes the statistics in the merged YAML file:

[Executes delay_calc_tool binary]
* runs UTH-Timer on the testsuite (using all available threads)
* runs Xyce/SPECTRE on the testsuite to produce golden results (using all available threads)
* merges UTH-Timer's results with the golden SPICE results and dumps statistics into a merged YAML file

## Binaries

### bin/gen_random_nets

This application is the dataset generator 
* generate YAML and SPEF files

```
bin/gen_random_nets options

options:
   --liberty LIBERTY_FILE     Liberty Models for cells (required)
   --dataset DATASET_NAME     YAML and SPEF file name (required)
   -n DATASET_SIZE            Dataset test circuits number (required)
   --pi_models                Must be enabled to produce pi model nets (required)
   --waveform WAVEFORM_TYPE   Waveform (ramp or asu_exp). NOTE: asu_exp must be used with ASU 7nm library (required)
   
```

### bin/delay_calc_tool

This application is the heart of the toolkit. It runs in the following modes:

* generate SPICE decks and dump them into a folder
* run UTH-Timer and dump results into a YAML file
* run SPICE simulations (Xyce or SPECTRE) and dump results into a YAML file
* merge UTH-Timer's results with SPICE golden results and dump statistics into a YAML file

##### generate SPICE decks

```
bin/delay_calc_tool --gen_SPICE_decks options

options:
   --liberty LIBERTY_FILE     Liberty Models for cells (required)
   --circuits CIRCUITS        *.circuits.yaml (required)
   --spef SPEF                *.spef corresponding to the *.circuits.yaml (required)
   --SPICE_dir DIRECTORY      directory for SPICE netlists (required)
   --SPICE_lib SPICE_MODEL    SPICE models (required)
   --simulator SIMULATOR      simulator (xyce or spectre) (required) 

```

Note that the SPICE decks generated are simulator-specific:

* Xyce SPICE decks include a preprocessing step
* MEASURE syntax is different between Xyce and SPECTRE

##### run delay calculation
```
bin/delay_calc_tool --dc_file DC_OUTPUT options

options:
   --liberty LIBERTY_FILE     Liberty Models for cells (required)
   --circuits CIRCUITS        *.circuits.yaml (required)
   --spef SPEF                *.spef corresponding to the *.circuits.yaml (required)

```

`DC_OUTPUT` is the output *.dc.circuits.yaml file that contains the UTH-Timer CCS delay calculation output.

##### run SPICE simulations

```
bin/delay_calc_tool --run_sims options

options:
   --circuits CIRCUITS.       *.circuits.yaml (required)
   --SPICE_dir DIRECTORY      directory for SPICE netlists (required)
   --simulator SIMULATOR      simulator (xyce or spectre) (required) 

```

This writes the SPICE simulation results into files (`*.mt0` for Xyce and `*.measure` for SPECTRE)

This type of invocation does not regenerate the SPICE decks.

##### merge and score
```
bin/delay_calc_tool --merge_circuits MERGE_OUTPUT options

options:
   --circuits CIRCUITS        *.circuits.yaml (required)
   --SPICE_dir SPICE_DIR      directory with SPICE results (required)
   --simulator SIMULATOR      simulator (xyce or spectre) (required) 
```

`MERGE_OUTPUT` is the *.merged.circuits.yaml file that includes SPICE results, UTH-Timer results, and statistics.

## Binary dependencies

* GLIBC > 2.27
* Intel Threading Building Blocks (TBB) library (libtbb.so.2)
* The binaries have been successfully tested on CentOS 8 and Ubuntu 18.04

## Conventions

* testsuite.circuits.yaml -- Circuit files containing only the testsuite
* testsuite.dc.circuits.yaml -- Circuit files containing delay calculation results
* testsuite.merge.circuits.yaml -- Circuit files containing both golden and delay calculation results (and scoring)
* testsuite.spef -- pi-model RC parasitics for driver interconnect (admittance model)
* testsuite.SPICE_decks -- Folder that contains all generated SPICE decks and results


## A typical example that demonstrates UTH-Timer vs SPICE comparison

* Inputs: ASU .lib, SPICE models, SPICE netlists of standard cells
* Run "./generate_nets testsuite" to create YAML and SPEF files (testsuite.circuits.yaml, testsuite.spef, and .sp files in testsuite.SPICE_decks)
* Run "./run_delay_calculator testsuite" to run UTH-Timer, SPICE and create a file with the results
* Outputs: testsuite.dc.circuits.yaml and testsuite.merge.circuits.yaml

### Download and install Xyce

  Go to [xyce.sandia.gov](xyce.sandia.gov) for more information

