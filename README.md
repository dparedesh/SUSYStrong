# Table of contents
1. [Introduction](#introduction)
2. [Get project from Git](#git)
3. [Setup instructions](#setup)
    * [Setup the tool for the first time ](#setup1)
    * [Every time that you start a new session](#setup2)
3. [Preparing the HF input files](#prepareInput)
4. [How to use the tool?](#useTool)
   * [About the scripts to be used...](#mainScripts)
   * [Before running... check that you fulfill the following requirements](#beforeRunning)
   * [Generating the HistFitter commands](#generateCommands)
5. [How to get exclusion limits?](#exclusion)
6. [How to get the exclusion plots for the `best limits`?](#bestLimits)
7. [How to get bkg-only fits?](#bkg)
8. [How to get discovery fits?](#discovery)
9. [How to do a binned fit?](#binnedFit)
10. [How to normalise a background in a CR ?](#normBkg) 
11. [How to add a sample specific weight for a determined sample?](#scaleBkg) 
12. [How to add a normalisation factor for a determined sample?](#NFBkg) 
13. [How to run jobs in lxplus batch?](#batch)
   * [How to check that everything went well in batch?](#batchCheck)
   * [How to submit failed jobs?](#batchResubmit)
14. [How to get yields and systematic tables?](#tables)
15. [How to get pull plots? ](#pull)
16. [How to get the summary plots and tables?](#summary)
17. [To get inspired... commands used to get the results in the paper](#paper)

# Introduction <a name="introduction"></a>
This package is used:

1. Generate the input root files for HistFitter
2. Generate the HistFitter commands to do all kinds of fit, to get yields or plots 


# Get project from Git <a name="git"></a>:

 Clone the tool from gitlab and get HistFitter:

  ```
    setupATLAS
    lsetup git
    git clone --recursive ssh://git@gitlab.cern.ch:7999/atlas-phys-susy-wg/InclusiveSquarkGluino/ss3l_2nd/ss3l_hf.git 
    cd ss3l_hf
    
  ```

 *The ```--recursive``` option allows to download directly the HF version 65 already patched with our files in HFpatch folder. 

# Setup instructions <a name="setup"></a>


### 1) Setup the tool for the first time <a name="setup1"></a> : 

1. From the main directory, *ss3l_hf*, setup the whole the package:
  
         
        source setup.sh
        

  >:mag_right: What is the setup.sh doing? 
  > * Setup ATLAS environment and the ROOT version (6.18.04)
  > * Setup HistFitter package  
  > * Export useful PATH
  > * Create **_HistFitterUser_** directory if it does not exist
  > * Create **_HistFitter/InputTrees_** directory it does not exist


### 2) Every time that you start a new session <a name="setup2"></a>:
    
   From the main directory, *ss3l_hf*, setup the whole package:
    
         . setup.sh 
     
    
  >:mag_right: **TIP:** In this step, if you want to speed up the process, you can comment out the lines **_make clean_** and **_make_** in the file setup.sh, which will avoid to compile HF every time that you start a new sesion allowing you to save time. 
    


## Preparing the HF input files: <a name="prepareInput"></a>

### General instruction on HF input files
* The input files must be divided in bg, signal, data
* the bg files should be grouped acoording to type AND TO EXPECTED THEORY UNCERTAINTY, since the uncertainties will be attached on a per-tree basis
* copyTree.C++ should be modified to generate the input root files for HF
* run root, .L copyTree.C++, doAll(lumi)


### Using alternate input ntuples
* copyTree.C has some level of branch-name-mapping, so in principle it should be possible to adapt the code to read ntuples with different branch names and output the format expected by HistFitter. Look for example at the guessBranchMapping function and at how it is being used. The tricky part will be to adapt the types of the local variables to the ones used in the alternate ntuple.
* If you implement the mapping for different ntuples, just let me have the code and I'll commit it.
* please refrain from adapting the HF config itself to deal with the different branches. The intermediate translation is meant to leave the HF config untouched and be able to compare different ntuples.

### Update xsections files

* Keep xsections files updated
* For results in Rel21, we are using the file in prepare/xsections/mc16_13TeV/PMGxsecDB_mc16.txt

### Set target Lumi
N.B. NEW TARGET LUMI OPTION
* Set the target Lumi (in pb-1) in doAll(lumi mc16a, lumi mc16d, lumi mc16e) with the corresponding lumi for each mc16 campaign in copyTree.C;
* The total lumi will be calculated directly and set as name of the output file, while 3 different rescaleLumi values will be calculated for each MC sample;
* The MC sample for the different mc16 campaign can be placed in the same directory, since the code is able to recognize the campaign from the r-tag in the name;
* Set the total target Lumi (in pb-1) in the main HF script (generateCommands.py). This will be propagated to the HF configuration through command-lines argument. The lumi tag is also used for defining the output file names, for the legends of the plots, and to decide which input files to read, so you'd better set it right.

# How to use the tool? <a name="useTool"></a> 


## About the scripts to be used... <a name="mainScripts"></a>
1. Most important python scripts are **python/generateCommands.py**, **python/fitConfigSS3L.py**, **macrosSS/plotContours.py**, and **python/plottingConfig.py**

    1. **generateCommands.py**: Main control script. It passes the needed parameters to **fitConfigSS3L.py** and  **plotContours.py**. In addition, it runs other scripts 
    generating the intermediate steps to produce the final exclusion plots.
    
    
    1. **fitConfigSS3L.py**: Script containing the fit configuration to be used by HistFitter. 
    
    3. **plotContours.py**: Script generating the plots for the 2D exclusion limits and 1D upper limits on the cross-section.
    4. **plottingConfig.py**: File containining all the hardcoded configuration for the exclusion plots (this script is called internally by *plotContours.py*), which are given as input to the _class Model_:

        * _x_ and _y_ axis ranges: input parameters to the functions `setXrange(xLow,xUp)` and `setYrange(yLow,yUp)`
        
        * Axis labels and process description: input parameters of the function `setLabels(xLabel, yLabel, processDescription)` 
        
        * Forbidden cuts and labels: input parameters of `setForbiddenCutsAndLabels(forbiddenCut,forbiddenLabelX,forbiddenLabelY,labelTex,forbiddenFunction)`
        
        * Position of the line and position of the label over this line to be shown: the input parameters of the function `setValLine(valX,valY,useForbiddenRegion)` help to determine these positions. Usually the upper value for the y-range is provided,
        but it is not always the case. You must play with that to draw the line. The option _useForbiddedRegion_ is a bool parameter that will say to the tool if it must use the 
        forbidden region to determine those positions. The way that it works can be seen in the function _update()_ of the class Model. 
        
        * Previous limits: input parameters of `setPreviousLimits(tag,legend,hep_data.csv,line,f)`. You can add as many previous limits as you want. They are
        saved in an internal dictionary with the key *tag*. If this *tag* is called in the script *plotContours.py*, then the previous limit associated with this *tag* will be shown. As in any dictionary, different 
        previous limits must be provided with a different key. The previous limits are the *.csv files downloaded directly from the HepData website. You must ensure that this file contains only the desired
        curve. Remaining info in this file must be deleted by hand. 
        
           For plots of upper limit on cross-sections: You must ensure that the parameter `line=True`. Upper limits on cross-sections are by default in `fb`, you want to change it to pb for example, then set `f=0.001`
    

        
2. Other useful and needed scripts are stored in dir named **python**
    * **pathUtilities.py:** File used to store all useful PATH. It must be carefully modified to fit your needs,  ***before doing anything*** . 
    * **massCheck.py:** Script used to check if there are signal samples with same mass parameters
    * **FillJsonFilesWithMissingInfo.py:**  Script used to fill the missing information into JSON files, needed to produce the exclusion plots.
    * **jsonToBands.py:**   Script used to read the mass points `m0` and all `excludedXsec*` from the json file and create expected, observed, and 1 and 2-sigma bands in TGraphs.    Also, it will save the plots for `theory x-section`, including the systematic variations. This script is **not** used to create 2D exclusion contours, but only when the goal is to plot the upper-limit on x-sections as a function of the generated mass point `m0`
    * **CollectSignalYields.py (TO BE UPDATED):** is used to get yields, fitting results, nuisance parameters informations e.g: ranking, pulling plots



3. **susyGridFiles**: Directory used to store the  SUSY signal samples used in the analysis. The model's name provided to **generateCommands.py** must match the name of a file in this directory. 
4. **InputTrees**: Directory used to place the HF inputs root files
5. **HistFitterUser**: Directory used to store all results of the exclusion plots.  




## Before running... check that you fulfill the following requirements: <a name="beforeRunning"></a>

1. Do the setup:

```
. setup.sh
````

Above command will set up the framework.

2. Ensure that you have modified the file *python/pathUtilities.py* to fit your needs. 

3. Add the HF ntuples to the *HistFitter/InputTrees* directory. The files should be named:

   * ```<bkg>.<lumi>.root```
   * ```signal.<lumi>.root```  (needed just for exclusion fits)
   * ```data.<lumi>.root```  (needed just if you are not going to use the hardcoded data yields). 

   where ```<lumi>``` must be in pb-1, and ```<bkg>``` is the name for every background process as defined in the *fitConfigSS3L.py* with the desired splitting. 
   
   >:mag_right: The recommendation is to have the ntuples stored in EOS, and create symbolic links to them from InputTrees. Don't run with ntuples stored in AFS, since it will kill the system.
   

4. Open the file *generateCommands.py* and check that the variable ```lumiTag=<lumi>``` agree with the value of the ntuples just above. 


5. Ensure that the SRs/VRs/CRs to be used are defined in the `configMgr.cutsDict[]` of the *fitConfigSS3L.py* script. If it is not the case, you must define them.

6. In the section `---- to define before running ------` of the file *fitConfigSS3L.py*:

    * Check the background splitting that you want to use, usually less  splitting is used for exclusion limits since the execution will be faster,
 while more is used to produce yields and systematic tables. If you want to use a different splitting you must define the samples and add them to the object `allBkg`. Only samples added to that object will be
used in the fit. Have a look a the *Helper.py* class to see how they are defined.  In particular, be carefull with the data-driven background since for those samples where the yields to be used are hardcoded (usually Fakes and MisCharge) must be
provided here. 

    * If you are going to use the data yields from the ntuples set `useDataSample=True`

7. Ensure that your region is also defined in the file `hardCodedYields.json` in the dictionary `binning`. The keys in this dictionary are the regions. Those at the same time are dictionaries of variables, and for every variable a list of the binning (**equal bin width**) to be used for that variable must be given

   ```ruby  
    'binning'={'region1': {'var':[nbins,low,up]}
              }

    ``` 
where `nbins` is the number of bins to be used for `var` and `low` and `up` are the lower and upper ranges of `var`. If the region is unbinned or single bin use ``` 'region':{'cuts':[1,0.5,1.5]}' ```

8. For the hardcoded yields (usually Fakes, MisCharge and data): Check the file `hardCodedYields.json` and ensure that the samples that you want 
to use are defined there for the `nominal, statistical and systematic variations`,  and for the regions and variables 
that you want to use. 

   * Yields/statistical/systematics must be provided as keys containing a dictionary of regions, containing a dictionary of variables per region, where the bin content for that variable is defined:
   
   ```ruby  
    'sample'={'region1': {'var1':[bin1,bin_2,...,binN],
                          'var2':[bin1,bin_2,...,binN},
                          'varN':[bin1,bin_2,...,binN},
              'region2': {'var1':[bin1,bin_2,...,binN],
                          'var2':[bin1,bin_2,...,binN},
                          'varN':[bin1,bin_2,...,binN},
              'regionN': {'var1':[bin1,bin_2,...,binN],
                          'var2':[bin1,bin_2,...,binN},
                          'varN':[bin1,bin_2,...,binN}}
    ```     
       

    If the region is unbinned then the variable defined must be `cuts`.
        
    Example: the lines below defines the nominal yields for the fakes in several regions for the unbinned fit (`cuts` variable), and a binned fit on the  variable 'nJets25' with 9 bins  for the VRWZ5j:
        
    ```python    
     'fakeCount' =  { 'VRWZ4j': {'cuts': [15.414]}, 
                    'VRttV': {'cuts': [14.107]}, 
                    'Rpc3LSS1b': {'cuts': [2.576]}, 
                    'Rpc2L1b': {'cuts': [1.253]}, 
                    'Rpc2L0b': {'cuts': [1.109]}, 
                    'Rpv2L' : {'cuts': [1.402]}, 
                    'Rpc2L2b': {'cuts': [1.36]}, 
                    'VRWZ5j': {'cuts': [13.729],
                                'nJets25':[0,8.08306,3.58574,1.39007,0.600395,0.0585532,0.00509548,0.00708743,0]}}
    ```    
        



 :warning: **WARNING:**
     
 * **For nominal and statitical uncertainties the bin content is absolute, while for the systematics the bin content to be provided  is relative.** 
 * **The names of the dictionaries provided here must agree with those called in the fitConfigSS3L.py in the lines  with the dictionary ```yieldsCoded```. If for some reason 
 you change those names you must ensure that the input provided when creating the samples matches those names everytime that you call ```yieldsCoded```.** 
  

 9. For the theory uncertainties, check the file ```theorySystematics.json``` and ensure that for every region that
 you want to use, both systematic variations, i.e. up and down, are provided for every sample. Currently this systematic is affecting 
 only the normalization of the sample. The values are provided as keys containing a dictionary of regions where the value assigned to each region is the relative systematic variation. 

    Example: up and down variations for the ttZ sample:
    
    ```python
    'hUnc_ttZUp'={"Rpc2L0b"   : 0.284500,
                "Rpc2L1b"   : 0.329800,
                "Rpc2L2b"   : 0.236100,
                "Rpc3LSS1b" : 0.388000,
                "Rpv2L"     : 0.449800,
                "VRttV"     : 0.224400,
                "VRWZ4j"    : 0.214400,
                "VRWZ5j"    : 0.235400}

    'thUnc_ttZDown'={ "Rpc2L0b"   : 0.229900,
                    "Rpc2L1b"   : 0.331300,
                    "Rpc2L2b"   : 0.213700,
                    "Rpc3LSS1b" : 0.374700,
                    "Rpv2L"     : 0.411600,
                    "VRttV"     : 0.225500,
                    "VRWZ4j"    : 0.217900,
                    "VRWZ5j"    : 0.243500}
    ```

 :warning: **WARNING:**
    
  * **The names of the dictionaries provided here must agree with those imported in the fitConfigSS3L.py everytime that you call the dictionary ```theorySyst```. If for some reason 
  you change those names you must ensure that the input provided when creating the samples matches those names.** 


10. For exclusion limits, ensure that the SUSY grid that you want to use is stored in `susyGridFiles` directory. If it is not, you must add it. 

11. Change the variable `runNameTag` to what you prefer. That will define the name of your output files.

## Generating the HistFitter commands: <a name="generateCommands"></a>

Running instructions:

    python generateCommands.py --FUNCTION --doBlind <option> --fitmodel FITMODEL --Model MODEL --SR REGION --Sys SYSLIST
1. ```--FUNCTION```: choose from `fit`, `merge`, `plot`, `yields`, `SysBreak`, `FitResults` according to what you want
2. ```--doBlind ```: choose from `ALL`, `SR`, `VR`, `CR`. It will blind the options provided. If more than one, they must be comma separated. 
3. ```--fitmodel```: choose from `excl`, `disc`, `bkg` according to what fit you want to perform
4. ```--Model```: set your fitting signal model. It must exist in the `choices` argument of the parser. 
5. ```--SR```: set your interested signal region. If more than one SR they must be comma separated. They can be binned.
6. ```--CR```: set your interested control region.  If more than one CR they must be comma separated. They can be binned. 
7. ```--VR```: set your interested validation regions. You can add as many VRs as you want, as long as they are comma separated. They are unbinned. 
6. ```--Sys```: set the systematics that you want to included. `None` for nothing, `ALL` to include all systematics. For signal region optimisation studies and preliminary results (when systematic ntuples are not available), you can use a flat systematic on the total background. You can use the string  `FLAT<number>`, where ```<number>``` is the value that you want to apply, for example  ```--Sys FLAT30``` will apply 30% of systematic to the total background.  
7. ```--batch```: use batch to run the fit. Currently only IHEP HTCondor  and HTcondor in lxplus batch systems are supported. 
    The following options works only on lxplus:
  
     * ```--extra <syst:bsub,queue:8nh,storage:20000>```: provide system, queue and storage required for batch. Options can be found in the script generateCommands.py. 
     * ```--resubmit  <filename>```: provide filename containing the name of the failed jobs. This file is automatically created when running in batch.      

7. ```--doUL```: it will compute the upper limits on the cross-section for ```--fit --fitmodel excl```. It will merge 
the UL results for ```--merge```, and it will draw the UL for ```--plot```. 

     * ```--draw1D```: Use this option together with --doUL if you can to produce 1D plots, i.e. upper limits on the cross-section as a function of the mass point. 

8. The options below make sense only for ```--plot``` option in the exclusion plots:

     * ```--drawCLs```: it will draw the CLs values.
    
     * ```--makeBest```: if the model uses more than one SR, then it will choose the best one for the contour plot. In order to run succesfully this option, 
    the full results for the model for every involved region must be ready, i.e. they must have been run with ```--plot``` for every region. It will read the results previously produced and generate the 
    best contour plot. In addition, if the option ```--doUL``` is provided, it will draw the best UL on the cross-section. You must provide a comma separated list of the regions to be combined.

9. ```--extraFitConfig```: use this option to pass additional parameters to the script ```fitConfigSS3L.py```. Normally used to do binned fits and to normalize a bacground in a CR. Check available options in that script. Argument must be passed in quotas.
10. ```--extraHarvestToContours```: Extra options to be passed to ```harvestToContours.py```. Check available options in that HF script. Argument must be passed in quotas.
9. **Other possible options are all listed in generateCommands.py** 
8. **All SRs, CRs, and VRs, must be defined in ```fitConfigSS3L.py``` and in the ```binning``` dictionary of the file ```hardCodedYields.json```, even if they are unbinned the default option must be given**


The table  below represents the behavior of the tool depending on the input region (as provided to the tool, not as defined in the analysis) for the kind of fit requested. For example, if the fit is bkg-only but you input a SR or a VR, 
then internally it will be converted to a CR. 

| Input regions |  Bkg-only  | Exclusion | Discovery |
|----|:---:|:---:|:---:|
| SR | CR | SR | SR |
| VR | CR | - | - |
| CR | CR | - | - |
| CR, SR | CR, VR | CR, SR | CR, SR|
| CR, VR | CR, VR | - | - |
| SR, VR | CR, VR | - | - |
| CR, SR, VR | CR, VR, VR | - | - |


# How to get exclusion limits? <a name="exclusion"></a>

Assume you are in `python` dir...

Exclusion limits in this tool are obtained through three different steps: ```--fit```, ```--merge```, and ```--plot``` (in that order!). They are explained below.


## 1) Perform the fit:

            python generateCommands.py --fit --doBlind ALL --fitmodel excl --Model Mc16SusyGG2StepWZ --SR Rpc2L0b --Sys None

  
  This command will perform an `exclusion fit`, blinding `ALL` regions for the `GG2StepWZ` model at `Rpc2L0b` signal region `without systematics` included. One can change the options as they want. All results will be stored in `HistFitter/results/` (the main script used are `generateCommands.py`).



## 2) Merge the results:
 
            python generateCommands.py --merge --doBlind ALL  --Model Mc16SusyGG2StepWZ --SR Rpc2L0b --Sys None

  This line will create the proper  command to merge all related results for the exclusion fits with `ALL` regions blinded  for the`GG2StepWZ` model at `Rpc2L0b` signal region `without systematics` included. It will merge the results  into one file per theory variation, i.e. nom, up and down, which are needed for plotting part. 
  Results will be stored in `HistFitter/results/` (the main script used is `generateCommands.py`)


## 3) Generate exclusion 2D plot: 

    
       python generateCommands.py --plot --doBlind ALL  --Model Mc16SusyGG2StepWZ --SR Rpc2L0b --Sys None


  This command will generate proper command lines to `plot 2-D exclusion limits` for the exclusion fits with `ALL` regions blinded for the`GG2StepWZ` model at `Rpc2L0b` signal region `without systematics` included. All results and plots will be stored in `HistFitterUser` dir. The main scripts used are `generateCommands.py` and `plotContours.py`. 


  :warning: **WARNING:**
  * **In order to run this option, you must be sure that your model and its plotting parameters are defined in the script ```plottingConfig.py```. If it is
  not, you must add it.**
  
## 4) Generate exclusion 1D plot :

    
      python generateCommands.py --plot --Model Mc16SusyTT2Step --SR Rpc3LSS1b --Sys ALL --doUL --draw1D


 This command will generate proper command lines to plot upper limits on the cross-sections as a function of the generated mass points for 
 the Mc16SusyTT2Step model at `Rpc3LSS1b` signal region `with ALL systematics` included. All results and plots will be stored in `HistFitterUser` dir. The main scripts used are `generateCommands.py` and `plotContours.py`. 

    
:warning: **WARNING:**
**You must ensure that you have done the exclusion fit and merging with `--doUL`. When plotting, both options, `--doUL --draw1D` must be provided to get the final plot of upper limit on the cross-section as a function of the generated mass point.**



# How to get the exclusion plots for the `best limits`? <a name="bestLimits"></a> 

Assume you are in `python` dir...

If you want to get the `best limit` for a model with multiple signal regions, the key word in the command line is `--makeBest <SR1>,<SR2>,<SR>,..`:

        python generateCommands.py --plot  --doBlind ALL --Model Mc16SusyBtt --SR Rpc2L1b --Sys None  --makeBest Rpc2L1b,Rpc2L2b


This command will generate the proper commands to get the best contour plot for the exclusion fit with `ALL` 
regions blinded  for the`Mc16SusyBtt` model `without systematics` included for the list of signal regions provided: Rpc2L1b and Rpc2L2b. In addition to the best contour plot,
it will also produce the plot of the best signal region used per each signal point in the grid. 

All results and plots will be stored in `HistFitterUser` dir.

  :warning:  **WARNING:**
  * **In order to run this comand you must have already produced all the contour plots for every signal region involved.** 
 


# How to get a background-only fit?  <a name="bkg"></a>

  Assuming you are in `python` dir

       python generateCommands.py --fit --fitmodel bkg  --SR Rpc2L0b --Sys ALL


 That command will generate a bkg-only fit in the SR Rpc2L0b with ALL systematics included. All results will be stored in `HistFitter/results/`.

# How to get a discovery fit? <a name="discovery"></a>
  
   Assuming you are in `python` dir

        python generateCommands.py --fit --fitmodel disc --SR Rpc2L0b --Sys ALL


 That command will generate a discovery fit in the SR Rpc2L0b with ALL systematics included. All results will be stored in `HistFitter/results/.`


 To get the model independent upper limits follow the instructions from HF to use the script `UpperLimitTable.py`



# How to do a binned fit? <a name="binnedFit"></a>

Binned fits are done by providing the proper extra options to the configuration file `fitConfigSS3L.py`. This is done by adding the option `--extraFitConfig` to the script `generateCommands.py` with the proper string, in this case ```--binnedFitSR``` and/or ```--binnedFitCR``` For example:


```
--extraFitConfig '--binnedFitSR Rpc2L1b:meff,Rpc2L2b:met --binnedFitCR CRWZ5j:nJets' 

```

which means that it will do a binned and simultanous fit in the SRs Rpc2L1b and Rpc2L2b in the variables meff and met, respectively, and in nJets in the CRWZ5j.  


You can have a look at the file `fitConfigSS3L.py` for other options.

# How to normalise a background in a CR ? <a name="normBkg"></a>

Again, you must provide the proper options to the file `fitConfigSS3L.py`. This is done by adding the option `--extraFitConfig` to the script `generateCommands.py` with the proper option, in this case ```bkgToNormalize```. For example:


 ```
--extraFitConfig '--bkgToNormalize WZ:CRWZ5j ' 

```

which means that it will use the CRWZ5j to normalize the WZ background.  


# How to add a sample specific weight for a determined sample? <a name="scaleBkg"></a>

Here the option to be passed to the ```--extraFitConfig``` of the ```generateCommands.py``` is ```--extraSampleWeight```. For example 

```
--extraFitConfig '--extraSampleWeight <bkg1>:<weightName1>,<bkg2>:<weightName2>' 

```
which means that it will apply the weight ```<weightName1>``` only to the ```<bkg1>``` sample, similarly for ```<bkg2>```. If you use numbers there they will be taken as SFs. For example:

```
--extraFitConfig '--extraSampleWeight WZ:weightSpecial' 

```

which means that the branch of the ntuples ```weightSpecial``` will be used as addidtional weight for the WZ sample. 


:warning:  **WARNING:**

**No systematic variations are allowed for this weight. HF internally multiplies the nominal weight for this weight for the nominal and systematic variations, i.e. in order to include systematic variations for this particular weight, we will need to hack HF internally to remove that weight from the systematics when including its own systematics... If you need systematic variations for this additional weight, the easist work around is to add this weight as nominal for all samples, and for those samples where the weight is not needed, set the values to 1.** 

# How to add a normalisation factor for a determined sample? <a name="NFBkg"></a>

Here the option to be passed to the ```--extraFitConfig``` of the ```generateCommands.py``` is ```--NF```. For example 

```
--extraFitConfig '--NF <bkg1>,<bkg2>,...' 

```
which means that it will apply the NF (defined in the file ```hardCodedYields.json``` for the  ```<bkg1>``` sample, similarly for ```<bkg2>,...```.  For example:

```
--extraFitConfig '--NF WZ,ttW' 

```

which means that NF for the bkgs WZ and ttW will be used. As said before, this normalisation factors and their uncertainties must be defined in the file ```hardCodedYields.json```, where every bkg is a key in the dictionary ```NF```. For example:

```python
    "NF": {
        "ttZ":[2,0.5,0.2],
        "ttW":[1.2,0.3,0.5],
        "WZ":[1.2,0.3,0.5]
    },

```
   
where every list is defined like:  ```[nom, up, down]``` , where ```nom``` is the nominal value of the SF and ```up``` and ```down``` are the **relative systematic variations** of the SF.

This is a usual work, when bkgs are normalised in a CR and their normalisation factors are extrapolated to SRs but the fits in the CR and SR are not simultanous. 


# How to run jobs in lxplus batch?<a name="batch"></a>

Assume you are in `python` dir...

Jobs in batch are only supported for the `--fit` option. Currently only `HTcondor` in `lxplus` and `IHEP cluster` are supported. 

In order to submit the jobs to batch you just need to add the options `--batch  --extra <syst:bsub,queue:8nh,storage:20000>` to the main command line. At least `syst` and `queue` must be provided, `storage` is optional.

Example: 

  ```
  python generateCommands.py --fit --doBlind ALL --fitmodel excl --Model Mc16SusyGG2StepWZ --SR Rpc2L0b --Sys None --batch --extra syst:condor,queue:tomorrow
  ```
  
  This command will perform an `exclusion fit`, blinding `ALL` regions for the `GG2StepWZ` model at `Rpc2L0b` signal region `without systematics` included. A
  job per signal point will be submitted for exclusion fit. If `--doUL`, then one job for the exclusion and one job for the upper limit will be submitted per signal
  point.
  
  
  
### How to check that everything went well in batch?<a name="batchCheck"></a>

**This works only for those jobs that were run in `lxplus`**

Jobs in batch, and even locally, can fail for several reasons. The main one is the connection lost when reading the ntuples. In this case, HistFitter does not stop when executing
the fit, and it just skips the TTree that was not able to read. That is actually a common problem when reading the systematics, but it can happen as well for nominal trees. As a result,
you can have a fit, whose result is a bit weird, or it does not match the values obtained by other people. 

The macro `CheckJobs.C` can be used to check that everything went well in batch. It is designed to spot several of the reasons (which have
been identified along time) for which the fit can't go well. 

When jobs are submitted to lxplus batch, a .txt file named `Jobs_<grid><runNameTag>_<fitmodel>.txt` is generated. This file is the input for the macro CheckJobs.C

How to use the macro? check that the `lumiTag` variable at the beginning of the script matchs the one that you used. Then:

```
root -l 
.L CheckJobs.C
CheckJobs("filename.txt")
```

After execution, it will print a summary containing the reason for which the jobs failed. In addition, it will create a file called `ToSumit.txt` containing the names of the failed jobs. 



### How to resubmit failed jobs? <a name="batchResubmit"></a>

**This works only for those jobs that were run in `lxplus`**

The file created previosly can be used to resubmit  to batch the failed jobs using option `--resubmit ToSubmit.txt` in the main command line. Example:


```
  python generateCommands.py --fit --doBlind ALL --fitmodel excl --Model Mc16SusyGG2StepWZ --SR Rpc2L0b --Sys None --batch --extra syst:condor,queue:tomorrow --resubmit ToSumit.txt
  ```
  
  which will submit only those jobs whose match the name in the file ToSubmit.txt




# How to get the yields and systematical tables? *->TO BE UPDATED* <a name="tables"></a> 

 
 
:exclamation: The README file have not been udpated for this section... However you can test if they work. They will be updated soon. The current recommedation is to use the instructions from HistFitter to generate the tables.-

Assuming you are in `python` dir


    python generateCommands.py --yields  --fitmodel bkg --Model Mc16SusyGG_Rpv331 --SR Rpv2L --Sys ALL

    python generateCommands.py --SysBreak  --fitmodel bkg --Model Mc16SusyGG_Rpv331 --SR Rpv2L --Sys ALL

Above two lines will get `yields` table and `systematical breakdown` for `bkg-only fit` aiming at `GG_Rpv331` for `Rpv2L` region with all systematics included. All results will be stored in `${HistFitterUser/Table/}` dir or `${HistFitterUser/Sys_BreakDown}` dir.
 
 
# How to get pull plots? <a name="pull"></a>

Assume that you are in `ss3l_hf`

Use directly the script provided by HistFitter:

   ```
   cd HistFitter
   pull_maker.py  -i <log_file> -o <output_name>
   ```


where `<log_file>` must be created by you by dumping all the output printed in the screen  when running the fit. 



# How to get the summary plots and tables? <a name="summary"></a>

The macro `plotSR.py` is used to get the summary plots and tables shown in the paper, while the macro `PlotSystematicsGrouped.py` is used 
to get the grouped systematics plot. Both scripts have their own README section at the beggining of the file explaining how to use 
them. Those scripts can be found at the `PlottingMacros` directory. The script `SummaryPlot.sh` summarize the commands used to get the summary plots and tables for the fast paper, in order to run it just do:

Asumming that you are in `ss3l_hf`:


     cd PlottingMacros
     ./SummaryPlots.sh 
     
 
**In order to run these macros, you will first need to create all the yields and systematic tables using the proper HistFitter commands for all the regions that you want to include in the plots.**  
 
 
The script `plotSR.py` is designed to read  the workspace created by HistFitter for the `bkg-only` fits. By default it will use the prefit symmetric uncertainties. For assymmetric  uncertainties the proper option must be provided.
For fitted yields, only symmetric uncertainties are available. Options for blinding results and show significance plots at the bottom are available as well. 
 
At the moment of running `plotSR.py`, the file `AsymmetricUnc.json`  is created. This file will be read by the script `PlotSystematicsGrouped.py` to get the statistical uncertainty. Be sure that the file  `AsymmetricUnc.json` contain all the regions that you want to include in the grouped systematics plot. 



# To get inspired... commands used to get the results in the paper: <a name="paper"></a>


### Bkg-only fits:  

Yields and systematic tables were obtained with this kind of fit. Also summary plots and tables. The command used to get the results for the signal region Rpc2L0b is shown below.

```
python generateCommands.py --fit --fitmodel bkg --SR Rpc2L0b --Sys ALL --doBlind ALL 
```



### Discovery fits: 

The command used to get the fit for the signal region Rpc2L0b is shown below.  

```
python generateCommands.py --fit --fitmodel disc --SR Rpc2L0b --Sys ALL 
```



### Exclusion countour plots:


The commands used to the exclusion contour plots for the model Mc16SusyBtt are summarized below.

 
```
python generateCommands.py --fit --fitmodel excl --Model Mc16SusyBtt --SR Rpc2L1b --Sys ALL --batch --extra syst:condor,queue:tomorrow
python generateCommands.py --merge --Model Mc16SusyBtt --SR Rpc2L1b --Sys ALL
python generateCommands.py --plot --Model Mc16SusyBtt --SR Rpc2L1b --Sys ALL

python generateCommands.py --fit --fitmodel excl --Model Mc16SusyBtt --SR Rpc2L2b --Sys ALL --batch --extra syst:condor,queue:tomorrow
python generateCommands.py --merge --Model Mc16SusyBtt --SR Rpc2L2b --Sys ALL
python generateCommands.py --plot --Model Mc16SusyBtt --SR Rpc2L2b --Sys ALL

python generateCommands.py --plot --Model Mc16SusyBtt --SR Rpc2L2b --Sys ALL --makeBest Rpc2L1b,Rpc2L2b
```
