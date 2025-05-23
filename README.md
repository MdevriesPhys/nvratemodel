# nvratemodel
Updated fork of numerical rate models to simulate the photophysics of the negatively-charged nitrogen-vacancy (NV<sup>-</sup>) centre in diamond.

What is this?
--------------
Fork of https://https://github.com/sernstETH/nvratemodel with NumPy's depreciated ``` np.float_ ``` function being replaced with ```np.float64```.

What this library can do:
- Everything the original library can do, but with NumPy 2.x compatablity. Go give the [original repo](https://github.com/sernstETH/nvratemodel) some love, and read their [paper](https://arxiv.org/abs/2304.02521) and [thesis](https://www.research-collection.ethz.ch/handle/20.500.11850/674178). Specifically:
  - Simulate NV<sup>-</sup> photophysics over commonly used temperatures (cryo to room temperatures), strain & electric fields, and magnetic fields.
  - Simulate at high computational speed the NV<sup>-</sup> centre photophysics at elevated temperature (e.g. room temperature).

What this library can't do (but hopefully will be made to do later):
- Simulate non-classical optical processes (spin coherence, etc)
- Simulate hyperfine structure
- Include the effects of strain/electic field on the ground state, and the effect of its on-axis components in general.

Installation
-------------
You need to install this library into a python environment. Consider the following steps:

- If you do not yet have a python environment (ideally >= python 3.8), first create one: For an example with Anaconda (recommended for unexperienced users) see https://conda.io/projects/conda/en/latest/user-guide/getting-started.html.
  Open the Anaconda Prompt console and create your ```myNVenv``` with
  ```
  conda create --name myNVenv python=3.9
  ```
- Activate the environment and install in the following order the required packages by running the following lines successively:
  ```
  activate myNVenv
  ```
  ```
  conda install numpy scipy matplotlib numba git
  ```
  If you use an already existing environment, and you are uncertain which packages you have installed already, check this via:
  ```
  conda list
  ```
  Please make sure all packages are **installed via conda, and not via pip**, as the performance of conda numpy with conda numba works best.

- Now we install this library into the environment by:
  ```
  pip install git+https://github.com/MdeVriesPhys/nvratemodel.git
  ```
  If you wish to uninstall it again later, use:
  ```
  pip uninstall nvratemodel
  ```

First steps
-----------
Check the installation in python by entering in the Conda prompt from above (with ```myNVenv``` active)
```
python
```
and import the library as:
```
import nvratemodel as nv
```
Then, run an example
```
nv.example_PRL_Fig3a.run(path=None)
```
which should look like the plot in Ernst et. al.'s [publication](https://arxiv.org/abs/2301.05091).
You can further test the following examples
```
example_PRB_Fig7a
example_PRB_Fig4h
example_PRB_Fig3b
example_PRB_Fig7b
```
which should look like plots in Ernst et. al.'s [related publication](https://arxiv.org/abs/2304.02521).

The code of these examples should serve as a guide on how to use the nvratemodel library. The various plot routines readily provided by the library, as well as the formalism of ```modeldict```s (dictionaries that contain a set of NV parameters) and ```NVrateModel```s (objects to simulate the behavior of a set of NV parameters) are further explained below.

There are two more things to note before you get started:

- Saving simulation results:
  
  In all examples above, as well as all simulation routines (see below), the optional argument ```path``` can be set to ```PATH_DATA```, which defaults to ```your_user_folder/nvratemodel_data/```. If you wish to set a different path, modify the ```PATH_DATA``` parameter in the ```GLOBAL.py``` file located in the installed nvratemodel root directory. In the same file, you can also modify the default values for the  NV parameters used by the model. Of course, all these default values are only used if no other values are explicitly provided by the user in the function calls.
  Test 
  ```
  nv.example_PRL_Fig3a.run(path=nv.PATH_DATA)
  ```
  to see whether it works and to have a look at the saved data structure (a human-readable ```.json``` file).

- Generating a LUT of the Phonon-Integral:
  
  As you probably noticed in the example above, the library printed a note about a missing lookup table (LUT) of the Phonon-Integral. If you just want to test the nvratemodel library a bit or if you just intend to use it at room temperature (```HighTmodel```), you can just ignore this. But if you want to use the ```MEmodel``` and are interested in a faster computational speed, do as the printed note tells you: generate (and automatically save in the root directory of the nvratemodel) a LUT, which will take on the order of 25min, but saves you time on all computations later on.
  Call 
  ```
  nv.updatePhononIntegralFullLUT(0.168)
  ```
  for the default Debye cutoff energy of ```phonCutoff=0.168```eV. You can also set other values of ```modeldict['phonCutoff']``` (see below) - the library will then ask you again to update the LUT with the values requested.

Using modeldicts and NVrateModels
---------------------------------
This library has two core objects to handle simulations:

- ```modeldict```s are python dictionaries that contain a full set of NV parameters. Such parameters are e.g. the magnetic field, its direction, or also the ```phonCutoff```mentioned above. They are generated by the function:
  ```
  mymodeldict = nv.makeModelDict(**kwargs)
  ```
  For possible ```kwargs```, which at the same time are the keys of a ```modeldict```, have a look at the doc string:
  ```
  help(nv.makeModelDict)
  ```
  These ```kwargs``` also correspond to the parameters given in Tab. 1 of https://arxiv.org/abs/2304.02521. You can view the default values that are used if not all possible ```kwargs``` are provided by:
  ```
  nv.printModelDict(nv.makeModelDict())
  ```
  You can also load the set of NV parameters of a simulation you previously saved. After running ```nv.example_PRL_Fig3a.run(path=nv.PATH_DATA)``` from above, you can now load the ```modeldict``` as:
  ```
  mymodeldict = nv.loadModelDictFromFile(nv.PATH_DATA+'\\yyyy-mm-dd_hh-mm-ss_2Dmap\\2Dmap.json')
  ```  
  
- ```NVrateModel```s are objects to simulate the population dynamics, and thus photoluminescence (PL), of a set of NV parameters. Several ```NVrateModel```-types are available:
  - ```MEmodel```: Master equation based model to simulate the NV center population dynamics over the full range of strain/el. field, magnetic field, and temperature from cryogenic to room temperature.
  - ```LowTmodel```: Classical rate equation model to simulate the NV center population dynamics at cryogenic temperature. Up to around 30K, the classical ```LowTmodel``` shows a similar behavior as the ```MEmodel``` and might be useful as an approximation with significantly enhanced computational speed. The correct treatment of orbital hopping due to electron-phonon coupling is the ```MEmodel```, though. In the ```LowTmodel```, orbital hopping is introduced as a classical rate that also destroys the spin space coherences, which is wrong. The ```LowTmodel``` introduces the rates in the same bases as the ```MEmodel```, see Fig. 2(d) of https://arxiv.org/abs/2304.02521.
  - ```HighTmodel```: Classical rate equation model to simulate the NV center population dynamics around room temperature, as often employed in the literature. As the orbital hopping rate of the ```MEmodel``` dominates the population dynamics, the ```MEmodel``` starts to resemble the behavior of this classical ```HighTmodel```. It can thus be used for computations at high speeds at elevated temperatures. Note that for high strain ```modeldict['Eperp']```, some orbital properties remain at elevated temperatures. These are only included in ```HighTmodel``` when ```modeldict['highT_trf']=True```. For more details see https://arxiv.org/abs/2304.02521.
  
  The NV parameters can be provided in the instantiation of an ```NVrateModel``` object as kwargs based on a ```modeldict```. The following three code blocks generate the same ```NVrateModel``` object called ```myMEmodel```:
  ```
  myMEmodel = nv.MEmodel(B=10e-3, T=300)
  ```
  ```
  mymodeldict = nv.makeModelDict(B=10e-3, T=300)
  myMEmodel = nv.MEmodel(**mymodeldict)
  ```
  ```
  mymodeldict = nv.makeModelDict()
  mymodeldict['B'] = 10e-3
  mymodeldict['T'] = 300
  myMEmodel = nv.MEmodel(**mymodeldict)
  ```
  To obtain e.g. the steady-state PL we can call:
  ```
  myMEmodel.PL()
  ```
  Since the ```MEmodel``` should give the same result at room temperature (```T=300```K), we can compare this to the PL of the classical rate model with the same parameter set:
  ```
  nv.HighTmodel(**mymodeldict).PL()
  ```
  To learn more about the methods that ```NVrateModel``` objects have, take a look at the doc strings of the methods of the base class ```NVrateModel```:
  ```
  nv.NVrateModel.population
  nv.NVrateModel.getPureState
  nv.NVrateModel.getGSPolarization_ms0
  nv.NVrateModel.getESPolarization_ms0
  nv.NVrateModel.steadyState
  nv.NVrateModel.PL
  nv.NVrateModel.propagateState
  nv.NVrateModel.calcTimeTrace
  ```
  
A word on computational speed and the numba package
----------------------------------------
Maybe you noticed above, that calling e.g. ```nv.HighTmodel(**mymodeldict).PL()``` for the first time is dramatically slower than calling it for the second time. This is because the numba package has to translate python code to fast machine code when the required functions are called for the first time.
If you wish to disable the usage of numba, you can easily do this by setting
```
NUMBA_OPT = False
```
in the ```GLOBAL.py``` file located in the installed nvratemodel root directory. This might be helpful when you cannot make sense of cryptic error messages generated by numba.

Overview of readily available simulation routines
--------------------------------------------------
The code of the examples in the example folder of this library should serve as a guide on how to use the nvratemodel library.
All high-level routines to simulate (and by default ```plot=True``` but not save ```path=None```) are:
```
simulateReadoutVsParam
simulate2DMap
simulateEigenVsParam
simulateIvsT
simulatekmixvsT
simulatePulseVsParam
simulatePopTimeTrace
fitMagnetAlignment
```
Take a look at their doc strings by ```help(nv.function_name)``` to learn more about how to use them or get an idea of their output by just calling them with their defaults ```function_name()```. Just the more involved ```simulatePopTimeTrace``` function needs some arguments, of which an example can be found in ```example_PRB_Fig3b.py```.

Programming simulations yourself
---------------------------------
Details on the basis states used in this library can be found in the dictionary containing them:
```
help(nv.basisStateNames)
```

All functions provided by the library to the user are listed below. Use ```help(nv.function_name)``` to learn more about them.

Hamiltonian (H) related functions:
```
get_Hes_EZ
get_H_EZ_withSS
partialTraceOrbit
partialTraceSpin
get_avgH_EZ_withSS
get_avgHTRF_EZ_withSS
get_orbitalSplitting
get_LarmorFrequ
get_GSresonances
```

Functions related to rates of the model:
```
lifetimeBoseEinsteinFunc
kmix1Full
kmix2TwoEmissions
DetailedBallanceRatio
DebyeIntegrandFull
PhononIntegralFull
PhononIntegralFull_fromLUT
updatePhononIntegralFullLUT
kmix2Full
getOrbitalRates
getPL
```

Functions related to ```modeldict```s:
```
loadModelDictFromFile
makeModelDict
switchToReducedAngles
scaleParam
formatParamValue
getParamStr
printModelDict
```

Functions to work with ```NVrateModels```s:
```
makeTstepsFromDurations
calcTimeTrace
makeStepsForLaserRise
getContrastOf2pointODMRTrace
sensitivityGauss
sensitivityEquation
sensitivityLor
sensitivityEquation_Lor
readoutSNR
SNREquation
getAmountMs0ForSequ
initState
piPulse
twoPointODMRTrace
getContrast
getInitFidelity_ms0
getReadoutFidelity_ms0
```

Jump operators:
```
LindbladOp_DecayOfEyToEx_HF
LindbladOp_DecayOfExToEy_HF
LindbladOp_DecayOfExToEx_HF
LindbladOp_DecayOfEyToEy_HF
LindbladOp_GS_msp1_ypiPulse_EZ
LindbladOp_GS_msp1_xpiPulse_EZ
```

Some useful functions and definitions:
```
printMatrix
printPop
purerho
vecToDensityMatrix
compositeDiagMatrix
expectationValue
sortarrays
getIndx
ensure_dir
inverse
conjTransp
eig
eigh
basisTrafo
get_T_XtoY
kB, h, hbar, S_z_opp, S_x_opp, S_y_opp, sigma_z, sigma_x, sigma_y, Id2, Id3, Id6, Id7, Id9, Id10
B_convfactor_GSandES
```

Functions for improved spin initialization and readout:
```
makeStepsForImprovedReadInit
getDepositedLaserEnergy
initStateImproved
getInitFidelity_ms0Improved
getContrastImproved
getSNRforImprovedReadInit
getPolarizationForImprovedReadInit
getOptimizedParamsForImprovedReadInit
```


Spin coherence during optical excitation and decay
---------------------------------------------------
The default setting ```spinCoherentOptics = True``` in the ```GLOBAL.py``` file implements the optical excitation and decay processes such that spin state coherences are preserved (https://arxiv.org/abs/1111.3687).
The transitions between the electronic states (ground state and excited state) are implemented as incoherent classical processes.
