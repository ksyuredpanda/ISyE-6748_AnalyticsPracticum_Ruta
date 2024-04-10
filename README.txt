Instructions for running the Jupyter Notebook and simulating data 
for ISYE 6748 - Applied Analytics Practicum "Detecting Wind Turbine Blade Damage using Supervised Machine Learning Algorithms", Spring 2024.
_____________________________________________________________________________

1. DATA SIMULATION:

The project relies on the simulation data obtained using free and open-source software OpenFAST provided by the National Renewable Energy Laboratory (NREL). The present project utilizes the simulated data, located in the "Data & Code" folder. There are 2 xlsx files containing wind tower vibration signals - "Tower_Vibration_healthy.xlsx", containing "healthy" vibration signals, and "Tower_Vibration_faulty.xlsx", containing "faulty" vibration signals (the damage (crack) on one of the blades is simulated by reducing the structural stiffness by 30% at one point of the blade). All the input files and precompiled Windows binaries needed to replicate the simulated data are provided in the folder "DataSimulation". For a more detailed description of OpenFAST features and usage please refer to "OpenFASTWorkshop_NAWEAWindTech2023_Practical_Branlard.pptx" included in the folder "DataSimulation".

Key points for data simulation:

1. All necessary files for running the simulation are provided in the "DataSimulation" folder. If you want to learn more about OpenFAST installation instructions, please refer to https://openfast.readthedocs.io/en/main/source/install/index.html
- Windows: https://github.com/OpenFAST/openfast/releases.

2. Various input files required for simulation are provided in the "DataSimulation" folder. More details on each input can be found on readthedocs: https://openfast.readthedocs.io/en/

3. NREL 5-MW Reference Wind Turbine is used in this study for vibration data simulation. The specification of this turbine can be found at https://www.nrel.gov/docs/fy09osti/38060.pdf.

4. The following eight signals representing wind tower displacements and accelerations were selected for simulation:
"Q_TFA1"		  - Displacement of 1st tower fore-aft bending mode DOF
"Q_TSS1"		  - Displacement of 1st tower side-to-side bending mode DOF
"Q_TFA2"		  - Displacement of 2nd tower fore-aft bending mode DOF
"Q_TSS2"		  - Displacement of 2nd tower side-to-side bending mode DOF
"QD2_TFA1"		  - Acceleration of 1st tower fore-aft bending mode DOF
"QD2_TSS1"		  - Acceleration of 1st tower side-to-side bending mode DOF
"QD2_TFA2"		  - Acceleration of 2nd tower fore-aft bending mode DOF
"QD2_TSS2"		  - Acceleration of 2nd tower side-to-side bending mode DOF

These signals were added to the "OUTPUT" portion of the "ElastoDyn.dat" file. The list of outputs available for each module in the OpenFAST can be found in the  "OutListParameters.xlsx" file available in the "DataSimulation" folder.

5. The parameter TMax was set to 10000 s in "Main.fst" to simulate the signal of length 10000 sec.:
10000   TMax          - Total run time (s)

6. The parameter IECturbc was set to 10 in "TurbSimBox_5MW_8mps.inp" to simulate wind turbulence intensity of 10%:
"10"       IECturbc        - IEC turbulence characteristic ("A", "B", "C" or the turbulence intensity in percent)

7. Additional file "ElastoDyn_Blade1.dat" was created, where the parameter FlpStff (flapweise stiffness) was reduced by 30% at one point (0.5561 of the blade length from blade root (BlFract)) to simulate the crack on one of the blades at this specific point. This file was then used as an input to "ElastoDyn.dat" when simulating "faulty" signals.

---------------------- BLADE ---------------------------------------------------
         17   BldNodes    - Number of blade nodes (per blade) used for analysis (-)
"ElastoDyn_Blade1.dat"                        BldFile(1)  - Name of file containing properties for blade 1 (quoted string)
"ElastoDyn_Blade.dat"                         BldFile(2)  - Name of file containing properties for blade 2 (quoted string)
"ElastoDyn_Blade.dat"                         BldFile(3)  - Name of file containing properties for blade 3 (quoted string) [unused for 2 blades]

8. "ElastoDyn.dat" is subsequently used as an input file for "Main.fst":

---------------------- INPUT FILES ---------------------------------------------
"ElastoDyn.dat"  EDFile          - Name of file containing ElastoDyn input parameters (quoted string)
"unused"         BDBldFile(1)    - Name of file containing BeamDyn input parameters for blade 1 (quoted string)
"unused"         BDBldFile(2)    - Name of file containing BeamDyn input parameters for blade 2 (quoted string)
"unused"         BDBldFile(3)    - Name of file containing BeamDyn input parameters for blade 3 (quoted string)
"InflowWind.dat" InflowFile      - Name of file containing inflow wind input parameters (quoted string)
"AeroDyn.dat"    AeroFile        - Name of file containing aerodynamic input parameters (quoted string)
"ServoDyn.dat"   ServoFile       - Name of file containing control and electrical-drive input parameters (quoted string)
"unused"         HydroFile       - Name of file containing hydrodynamic input parameters (quoted string)
"unused"         SubFile         - Name of file containing sub-structural input parameters (quoted string)
"unused"         MooringFile     - Name of file containing mooring system input parameters (quoted string)
"unused"         IceFile         - Name of file containing ice input parameters (quoted string)

9. Turbulent wind field is created by using the TurbSim program (part of OpenFAST). To run TurbSim:
- Open a terminal/command line
Windows: from the File explorer: File -> Open Windows PowerShell
- Navigate to the folder where the TurbSim executable ("TurbSim_x64.exe") and input file ("TurbSimBox_5MW_8mps.inp") are:
cd path/to/folder
- Call the TurbSim executable with the input file as argument:
 ./TurbSim_x64.exe TurbSimBox_5MW_8mps.inp

10. To run OpenFAST:
- Open a terminal/command line
Windows: from the File explorer: File -> Open Windows PowerShell
- Navigate to the folder where the OpenFAST executable ("openfast_x64.exe") and main input file ("Main.fst") are:
cd path/to/folder
- Call the OpenFAST executable with the main input file as argument:
./openfast_x64.exe Main.fst
It is recommended to run OpenFAST from the directory where the "Main.fst" file is located.


2. JUPYTER NOTEBOOK

Key points:

1. The code is contained in a single Python Jupyter Notebook file: code blocks must be run sequentially. 

2. The following Excel reference files must be present in the **same working directory** as the Jupyter Notebook:
   "Tower_Vibration_healthy.xlsx"
   "Tower_Vibration_faulty.xlsx"

3. You may need to install the following packages: skfeature-chappers, pywavelets, kymatio
   pip install skfeature-chappers
   pip install pywavelets
   pip install kymatio

4. Classification using multi-domain statistical features as input data was performed for three different subsets of features: the full set, a subset of 15 features selected by Random Forest Feature Importance technique and a subset of 15 features selected by Fisher's Score technique. Optimal model parameters are provided for the subset of 15 features selected by Fisher's Score, as this set of features yielded the highest model accuracy. However, you can use any of these three feature sets by uncommenting and running corresponding code chunks provided.    

5. Hyperparameter tunings for each classifier and each of the 4 input vectors are in the separate code blocks. Optimal parameters for each of the models, found via 5-Fold cross-validation, are provided for your convenience in case you would like to use them directly without running a time consuming hyperparameter tuning code blocks. There are additional chunks of code provided for each model, allowing You to use optimal parameters directly. They are followed by the chunks of code performing hyperparameter tuning. If you still wish to perform hyperparameter tuning, simply uncomment these code blocks.

6. Enjoy!

This was a rewarding project!
   


    






 





    



 
