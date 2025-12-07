# Cyclotron
Author: John Francis Dec 2025

# How to build the app
The easiest way to build the app is using docker. I have pushed a fully-contained docker environment for convenience. To use, just make sure you have docker installed and then run:

***Windows***
```sh
docker run --rm -id -v ${PWD}/build:/home/app/build john9francis/cyclotron_2025
```
***Mac***
```sh
docker run --rm -it -v $(pwd)/build:/home/app/build john9francis/cyclotron_2025
```
Here's some explanation about each command
- `--rm` removes the container after use. This helps to keep docker clean and not having tons of old runs hanging around
- `-it` gives you an interactive terminal to interface with the container. This is where we will run our commands to interact with the Cyclotron app.
- `-v $(pwd)/build:/home/app/build` creates a folder called "build" in your current working directory that exactly mirrors the one inside the container. This is where we will edit our macros, analysis scripts, and see our outputs. 

# How to use the app
We can either use the app interactively or through mac files. To run the app interactively, we run:
```sh
./G4_Cyclotron
```
Then we can run commands like `/run/initialize` `/gun/energy 10 MeV` or `/run/beamOn 50000`.

To run with a certain mac file, we do:
```sh
./G4_Cyclotron mac_file.mac
```

And if we want to save the output, we can run
```sh
./G4_Cyclotron mac_file.mac > output.txt
```

# Macro commands
Here are a few helpful Geant4 macro commands

- `/run/initialize` 
  - Initializes the geometry, physics, and gets the run ready
- `/run/beamOn <nparticles>`
  - e.g. `/run/beamOn 100000`
  - Runs particles. The more particles we run the better statistics we get, it doesn't neccessarily translate to "number of protons"
- `/gun/energy <energy> <unit>` 
  - Sets energy of primary protons
  - e.g. `/gun/energy 12 MeV`
- `/analysis/setFileName <my_file.root>`
  - Sets filename of output data
  - e.g. `/analysis/setFileName 11MeV.root`
- `/run/reinitializeGeometry`
  - Restarts geometry after any changes have been made
  - Such as changing target or foil thickness or Titanium enrichment


For this specific application, we have some more commands at our disposal.

Note: After running many of these commands we must run `/run/reinitializeGeometry`
- `/cyclotron/beam/setBeamCurrent <double> <unit>`
 - default unit: `microampere`
- `/cyclotron/beam/setTimeOfIrradiation <double> <unit>`
 - default unit: `hour`
- `/cyclotron/beam/setBeamEnergy <double> <unit>`
 - default unit: `MeV`
 - This is the proton beam energy
 - Note: this is pointless, we can just use `/gun/energy`
- `/cyclotron/target/setThickness <double> <unit>`
 - default unit: `mm`
 - Must be between 0.1 and 2 mm
- `/cyclotron/target/changeTarget <isotope>`
 - `<isotope>` can be either `Ti48` or `Ni64`
- `/cyclotron/target/setFoilThickness <double> <unit>`
 - default unit `mm`
 - Must be between 0 and 1 mm
- `/cyclotron/target/setTiEnrichment <double>`
 - Sets the Titanium 48 enrichment. Note that the target must be Ti48 for this to work.
 - Must be between 0 and 100, perferrably between 50 and 100.

For even more detailed documentation about the cyclotron commands, check these links:
- [cyclotron](./_cyclotron_.html)
- [cyclotron/target](./_cyclotron_target_.html)
- [cyclotron/beam](./_cyclotron_beam_.html)

# How to use the analysis
The analysis graphs are generated via named root files and a python script. For example, here is how you can generate graph 1: Activity vs Energy:
1. Run the yield.mac file
  - This mac file looks like this:
```
/run/initialize

/analysis/setFileName 11MeV.root
/gun/energy 11 MeV
/run/beamOn 100000

/analysis/setFileName 12MeV.root
/gun/energy 12 MeV
/run/beamOn 100000
...
```
Basically we change the root file to an energy and then run that energy. The python script will be looking for the following root files in order to run the script successfully:
```
11MeV.root
12MeV.root
13MeV.root
14MeV.root
15MeV.root
16MeV.root
```
Now, the graph doesn't say anything about target thickness, current, time irradiated, or anything like that so I would suggest setting those beforehand in the mac file and then specifying in the figure caption which parameters we used. The defaults are:
```
Ti-48 enrichment: 99%
Ti-46: 0.2%
Ti-47: 0.2%
Ti-49: 0.3%
Ti-50: 0.3%
Target thickness: 0.6 mm
Target radius: 3 mm
Degrader thickness: 0.32 mm
Beam Current: 20 μA
Irradiation Time: 4 hours
```

For all functions in the `phase2_analysis.py` scripts, the following filenames are needed:
```
// plot_activity_vs_energy
11MeV.root
12MeV.root
13MeV.root
14MeV.root
15MeV.root
16MeV.root
// plot_yield_vs_thickness
03mm.root
04mm.root
05mm.root
06mm.root
08mm.root
10mm.root
// plot_activity_vs_current
10uA.root
20uA.root
30uA.root
40uA.root
50uA.root
60uA.root
// plot_yeild_vs_radius
Note: this graph is just from "extrapolation" not data from this simulation
// plot_saturation_comparison
06mm.root
// plot_activity_vs_time
30uA.root
```
The `spectacular_visualizations.py` file also uses the same files in it's analysis.
