# Earth System Metrics and Diagnostics Standards (EMDS)

## Author list
Lead author  
Ana Ordonez, Lawrence Livermore National Laboratory  

The following authors are presented in order of last name, with their affiliation at the time of contribution:   

Chih-Chieh Chen, National Center for Atmospheric Research  
Dani Coleman, National Center for Atmospheric Research  
Andrew Gettelman, National Center for Atmospheric Research  
Thomas Jackson, Science Applications International Corporation  
John Krasting, NOAA Geophysical Fluid Dynamics Laboratory  
Yi-Hung Kuo, NOAA Geophysical Fluid Dynamics Laboratory  
Jessica Liptak, NOAA Geophysical Fluid Dynamics Laboratory  
Eric Maloney, Colorado State University  
Elizabeth Maroon, University of Wisconsin-Madison  
David Neelin, University of California Los Angeles   
Aparna Radhakrishnan, Princeton University CIMES  
Paul Ullrich, University of California Davis   


## Table of Contents
1. [Best Practices](#bestpractices)
    1. [Nomenclature](#nomenclature)
    2. [Supported Languages](#languages)
    3. [Digital Object Identification](#doi)
3. [DEM Framework File Format](#demframework)
    1. [Settings file](#settings)
    2. [Contents file](#contents)
4. [Runtime Environment](#runtimeenvironment)
5. [Common Output Bundle Format](#commonoutput)
6. [Observational Data Format](#obsdataformat)
7. [Observational Data Server](#obsdataserver)
8. [Common Metric Output Format](#commonmetric)
    1. [DIMENSIONS](#dimensions)
    2. [RESULTS](#results)
9. [Governance](#governance)
    1. [Changes to standards](#gov_new_stan)
    2. [Adding governance members](#gov_new_mem)
    3. [Removing governance members](#gov_remove_mem)
    4. [Votes](#gov_votes) 


## Best Practices <a name="bestpractices"></a>

### Nomenclature <a name="nomenclature"></a>
In this document we refer to [MDTF](https://mdtf-diagnostics.readthedocs.io/en/latest/index.html) process-oriented diagnostics (PODs) and [CMEC](https://cmec.llnl.gov) metric kernels (MKs) as data evaluation modules (DEMs). A workflow within the DEM which has its own settings JSON is referred to as a configuration.

### Supported Languages <a name="languages"></a>
It is recommended that DEMs are developed in languages that are portable, flexible, and unrestricted. Python generally meets these requirements, and frameworks based on the EMDS standards may require the use of Python. That being said, any routine that enables command line execution of the DEM (with a single executable driver script per configuration; see "DEM Framework File Format: Settings file" below) can be made to work under these standards.

If compiled code is used, a robust build system should be provided.  Compiled code should minimally rely on third-party dependencies and/or should be available through a conda package.


### Digital Object Identification <a name="doi"></a>

The DEM developer should register the DEM for a Digital Object Identifiers (DOI). This includes PODs that are merged into the MDTF framework source code. A service such as [Zenodo](https://zenodo.org/) can be used to obtain DOIs for software. 

A Digital Object Identifier for your DEM does not need to point to a specific version of source code. We will distinguish between "concept" DOIs, which apply to any and all versions of the source code, and "version" DOIs, which apply to a single, specific version of the source code. This is the system used by [Zenodo](https://zenodo.org/) (see Zenodo's [Frequently Asked Questions](https://help.zenodo.org/) page for their usage).

At a minimum, the DEM should have a "concept" DOI that applies to any and all version of the software. The DEM may also have version specific DOIs. If a POD has any version specific DOIs, there must be a DOI for the version that is merged into the MDTF source code.


## DEM Framework File Format <a name="demframework"></a>

### Settings file <a name="settings"></a>

All DEM configurations must have a unique settings file in JSON that describes how the module will be executed.  If there is only one configuration, the file must be called “settings.json”.  If there are multiple configurations, each settings file should have a unique name which must be listed in the contents file under the “contents” key.  The contents file is described at the end of this section.

The required base level keys of the settings file are as follows:
| settings    | The module configuration                     |
--------------|----------------------------------------------|
| varlist     | Required model data variables                |
| obslist     | Required reference datasets and variables    |
| coordinates | Constraints on data supported by this module | 

<p>&nbsp;</p>

Under the settings key, the following key/value pairs are specified:
| Key  | Definition |
|------|------------|
| driver | The name of the top-level script (entry point) to call to start running the module. This should be a file path relative to the DEM’s code directory. |
| async | (optional) An indicator that this module is executed asynchronously. Asynchronous modules should provide a status message on complete. |
| long_name | A human readable name of module or configuration. |
| description | A few sentences to describe the module or configuration and its potential applications. |
| runtime | A list of either names or key-value tables specifying the executables and third-party libraries (modules, packages, ...) required to run all code in the module, specified as follows: <br> a. The names or keys in the list are the names of executable programs needed, in version-string format (described below). <br>b. The values associated with the keys are a list of third-party libraries for that language (as named in that language's library repository), again in version-string format. Only libraries not included as part of that language's standard installation need to be listed. <br>c. The format for version strings is one of '(name)', '(name)==(version)', '(name)>=(version)' or '(name)<=(version)'. Both (name) and (version) may be arbitrary strings (not containing =, < or >). |
| selftest_script | (optional) This should be a UNIX command-line string, to be executed in the module’s code directory, which runs a script to perform any automated tests of the module’s code. <br>a. Module authors are encouraged to write unit and integration tests for their code in order to simplify debugging and maintenance. <br>b. The executable and any libraries required to run the script should be included in the runtime specification above. This includes UNIX shells. <br>c. The test script will run in the environment specified in “Runtime Environment” below, except for the model data features. In particular, the script may access files in the DEM’s code and observational data folders. <br>d. The test script should not assume the DEM has access to the example model data. <br>e. The test script should obey all the restrictions in the "Running the DEM" section. <br>f. The test script should return exit code 0 only if all tests pass, and != 0 otherwise. All output should be written to either stdout or stderr. |

<p>&nbsp;</p>

The varlist specifies the variables that are needed from the model data.  Each entry in the list requests one or more model datasets for a given variable.  Required varlist keys:
| Key  | Definition |
|------|------------|
| var_name | This should be one of the CF (version 1.7) [standard names](http://cfconventions.org/standard-names.html), or an abbreviation used in CMOR 3.5 (see [CMIP6-CMOR Tables](https://github.com/PCMDI/cmip6-cmor-tables/tree/master/Tables) under key "variable_entry"). If the name exists the variable must be scalar-valued: for example, (u,v,w) wind velocity components should be requested as three separate entries. If the module requires non-standard data that can be computed from CF variables (e.g., column integrated water vapor), the module should perform that computation internally.  |
| units |  A UDUnits-compatible string specifying the variable's units. If not provided, the CF convention's canonical units will be assumed.  |
| frequency | This should be one of the CMOR3.5 values. The frequency value should include information about sampling or averaging frequencies. Use “any” if any frequency is acceptable. |
| z_levels | (optional) If applicable to the variable, a list of one or more integers or floats specifying the vertical slice to extract; or this can be “column integral” for vertically-integrated quantities. |
| z_units | (optional) If applicable to the variable, a UDUnits-compatible string specifying the variable’s units.  If not provided, the CF convention's canonical units will be assumed.  |

<p>&nbsp;</p>

The obslist specifies the list of observational datasets needed for the module:
| Key  | Definition |
|------|------------|
| obs_name | Short name of the observational dataset. |
| version | (optional) Required version of the observational data. |

<p>&nbsp;</p>

The coordinates keys specify ranges of acceptable time and spatial resolution:
| Key  | Definition |
|------|------------|
| min_time_range<br>max_time_range | (optional) CMOR3.5 or UDUnits-compatible strings specifying the minimum and/or maximum length of analysis period the DEM requires. For example, a DEM that uses monthly data may need to set a minimum time range in order to guarantee it has enough data to compute statistically meaningful results. If this is not specified, any time duration is allowed. |
| min_resolution<br>max_resolution  | (optional) Should be one of the CMIP6 nominal_resolution values, with "nominal resolution" defined as in the [CMIP6 Global Attributes document](http://goo.gl/v1drZl). These specify the minimum and/or maximum spatial resolution the DEM’s data should have for the DEM to produce meaningful results. If this is not specified, any resolution is allowed. |

<p>&nbsp;</p>

### Contents file <a name="contents"></a>

If there are multiple configurations within the DEM, there must be a JSON file named “contents.json” that contains the module name and a list of the settings files within the DEM.

The required base level keys of the settings file are as follows:
| Key  | Definition |
|------|------------|
| module | Provides the DEM name. |
| contents | List of all DEM settings file names (relative to DEM directory). |

<p>&nbsp;</p>

The following module keys are required:
| Key  | Definition |
|------|------------|
| name | Short name of DEM |
| long_name | A human readable name of module |

<p>&nbsp;</p>

## Runtime Environment <a name="runtimeenvironment"></a>

This section describes the runtime environment, as initialized by the framework. When executed through the CMEC/MDTF interface, the DEMs may assume that all conditions below are met.

All executables and third-party libraries requested in the settings file will be available on the $PATH seen by the DEM.  

The following environment variables will be set:

| Key  | Definition |
|------|------------|
| CMEC_CODE_DIR <br>MDTF_POD_CODE_DIR| The directory where the DEM’s code can be found |
| CMEC_OBS_DATA<br>MDTF_POD_OBS_DATA| The directory where the relevant observational data can be found. |
| CMEC_MODEL_DATA<br>MDTF_MODEL_DATA | The directory containing the model input data.  If autocurated metadata has been generated, it can be found in the file autocurator.json in the root directory. |
| CMEC_WK_DIR<br>MDTF_POD_WK_DIR | The working directory for output from the DEM. |
| MDTF_CASENAME<br>MDTF_FIRSTYR<br>MDTF_LASTYR | (optional) Variables used by MDTF PODs for labeling plots. |
| CMEC_DEBUG<br>MDTF_DEBUG | (optional) If set to 1 then the DEM should output more verbose logging information to help with debugging. |
| CMEC_SAVE_NC<br>MDTF_SAVE_NC | (optional) If set to 0 then the DEM should skip any steps where raw output data is saved in netCDF format. |

<p>&nbsp;</p>

The following restrictions hold on DEM operation:  

DEMs should not refer to, or attempt to access, any files outside of the directories specified by the environment variables above.
All files in *_CODE_DIR, *_OBS_DATA, or *_MODEL_DATA should be treated as read-only.  

Output from DEMs should only be written to *_WK_DIR. Any intermediate files not part of the output bundle should be deleted before exiting. DEMs should not assume persistence of any files across runs.  

Temporary files, if needed, can be written to the working directory, preferably in the “temp” subfolder. This directory should be removed once execution is completed successfully.  

DEMs should not attempt to open network connections or access resources on remote hosts.  

DEM output can be logged to stdout or stderr. Both outputs may not be displayed to the user directly, but will be logged to a run log.

## Common Output Bundle Format <a name="commonoutput"></a>

Output must be accompanied by a JSON file describing what is present in the output directory. It is recommended that the DEM has defaults in place for circumstances where observation or model metadata is missing.

Base level keys (at least one should be specified):
| Key | Definition |
|------|------|
| index | Short name of the plot/html/metric that should be opened when the user chooses to “open” the output bundle. |
| provenance | Command line and version information used to execute the DEM. This includes environment variables and the observational datasets used (including dataset versions).  |
| data | (optional) Dictionary of data files produced with the output. |
| plots | (optional) Dictionary of plots produced with the output. |
| html | (optional) Dictionary of html files produced with the output. |
| metrics | (optional) Dictionary of metric files produced with the output.  |

<p>&nbsp;</p>

Required provenance keys:
| Key  | Definition |
|------|------------|
| environment | Key/value pairs listing all relevant diagnostic and framework environment variables. |
| modeldata | Path to the model data used in this analysis. |
| obsdata | Key/value pairs containing short names and versions of all observational datasets used. |
| log | Filename of a free format log file written during execution. |

<p>&nbsp;</p>

Required data::<short_name> descriptors:
| Key  | Definition |
|------|------------|
| filename | Filename of plot produced (relative to output directory path) |
| long_name | Human readable name describing the plot |
| description | Description of what is depicted in the plot |

<p>&nbsp;</p>

Required plots::<short_name> descriptors:
| Key  | Definition |
|------|------------|
| filename | Filename of plot produced (relative to output directory path) |
| long_name | Human readable name describing the plot |
| description | Description of what is depicted in the plot |

<p>&nbsp;</p>


Required html::<short_name> descriptors:
| Key  | Definition |
|------|------------|
| filename | Filename of plot produced (relative to output directory path) |
| long_name | Human readable name describing the plot |
| description | Description of what is depicted in the plot |

<p>&nbsp;</p>

Required metrics::<short_name> descriptors:
| Key  | Definition |
|------|------------|
| filename | Filename of metric file produced (relative to output directory path) |
| long_name | Human readable name describing the metric file |
| description | Description of what is presented in the metric file |

<p>&nbsp;</p>

## Observational Data Format <a name="obsdataformat"></a>

Post-processed observational data is in a directory structure as follows:
```
obs_data_directory
- obs_data_short_name
-- version
```

The version subdirectory must contain a metadata descriptor file for the observational dataset.  Required base level keys:
| Key  | Definition |
|------|------------|
| short_name | Short name of observational dataset |
| long_name | Human readable name of observational dataset |
| description | A few sentences to describe the observational dataset |

<p>&nbsp;</p>

A version directory should also contain the script used to postprocess the undigested data that is used. Historical versions are useful for provenance and reproducibility, and so should be maintained to the extent possible.

Post-processed observational data can be subsetted by time period or geography. The subset period should be documented in the metadata descriptor file.

## Observational Data Server <a name="obsdataserver"></a>

The observational data server allows external users to query the database of available observational datasets and download datasets that are needed for their DEM. Versioning is employed (1) to enable users to ensure they have the latest version of a particular dataset and (2) to enable reproducibility by allowing users to download older versions of each dataset. The data server must support both a query of available observational datasets and a request for a particular dataset.

One lightweight method of satisfying the above goals is simply through HTML:

The query of observational datasets could be performed by downloading a JSON formatted index file from the data server. The JSON file would contain URLs of each observational dataset and their corresponding version along with checksums to ensure the data is uncorrupted on download.  

Downloading of a particular observational dataset would then simply require downloading of all files linked in the JSON file and placing them in the observational data repository on the local system.

## Common Metric Output Format <a name="commonmetric"></a>

Here we describe the common format for metric information calculated by DEMs. The output file format described herein is designed to allow for easy visualization and interaction within the CMEC visualization module.

The expected format of the common model evaluation metric output file is JSON. This format allows for the file to be easily parsed and analyzed in web-based tools, and many programming languages support JSON parsing. The required base-level keys are DIMENSIONS, and RESULTS. Optional keys are PROVENANCE, DISCLAIMER, and NOTES, along with any other keys required by the DEM.
 
### DIMENSIONS <a name="dimensions"></a>

The base-level key DIMENSIONS is required to contain at least one sub-key json_structure which is a list of dimensions used in this file for organizing metric information, arranged in a hierarchical order. The order chosen is at the discretion of the model evaluation package developer. Optional metadata can be provided to describe dimensions such as “region” or “season”.

The following is an example of the DIMENSIONS key from the PCMDI Metrics Package output. The required field is DIMENSIONS:json_structure. The other metadata fields shown are optional.
``` 
"DIMENSIONS": {
    "json_structure": [
        "region",
        "model",
        "metric"
    ]
    "metric": {
        "rms_xyt": {
            "Name": "Spatio-Temporal Root Mean Square",
            "Abstract": "Compute Spatial and Temporal Root Mean Square",
            "URI": "http://uvcdat.llnl.gov/documentation/utilities/utilities-2.html"
        },
    },
    "region": {
        "Global": {},
        "TROPICS": {
            "generator": "cdutil.region.domain(latitude=(-30.0, 30))"
        }
    }
},
```
Developers should consider minimizing the number of dimensions used for compatibility with visualization tools and general ease of use. 

### RESULTS <a name="results"></a>

The base-level key RESULTS provides a tree containing the results of metric evaluation. The nesting structure of this tree must be identical to the order specified in DIMENSIONS.json_structure. Each level of the tree contains one optional key “attributes” that can be used to specify additional metadata at this level of the tree. All other child keys refer to a further specification of the subset of the provided data. 

The strings for each child key may contain a double colon (::) delimiter, which denotes a further hierarchical specification of the value at that level. The strings may also contain the double exclamation mark (!!) delimiter which may be used to denote a further refinement of the available data. These considerations are best conveyed by example:
```
"RESULTS": {
      "Global": {
            "E3SM::low_res": {
                  "Ecosystem and Carbon Cycle::Biomass": {
                        "Overall Score": 0.35,
                        "Seasonal Cycle Score": 0.20
                  },
                  "Ecosystem and Carbon Cycle::Biomass!!GEOCARBON": {
                        "Overall Score": 0.37,
                        "Seasonal Cycle Score": 0.19
                  },
                  ...
            },
            ...
      },
      ...
},
```

## Governance <a name="governance"></a>

Steering committee members: Ana Ordonez, Aparna Radhakrishnan, J. David Neelin, John Krasting, Paul Ullrich, Jessica Liptak
 
### Procedure for proposing new changes to the standards <a name="gov_new_stan"></a>
 
Changes should be made in a new branch in a personal fork of the EMDS repository. The branch should have a descriptive and unique name. A pull request (PR) can be opened from that branch when the changes are ready to be proposed. Draft PRs are permitted and should be marked as a draft in GitHub.
 
The pull request must include a description that addresses these two points:
1. An explanation of why the change is being made
2. Any expected backwards compatibility issues
 
The PR author should tag 1-2 reviewers on the PR. The PR must be approved by all members of the steering committee before it can be merged. All comments left on the pull request must be addressed before merging. The PR should be merged by a member of the steering committee who is not the original author of the PR.
 
### Adding new members <a name="gov_new_mem"></a>
 
Individuals who wish to be added to the Earth-Systems-Diagnostics-Standards organization on GitHub can contact the EMDS steering committee by email with their request.
 
Individuals who wish to join the EMDS steering committee can open an issue with their request. Individuals wishing to volunteer should have familiarity with relevant diagnostics and metrics frameworks and/or be participating in the relevant model diagnostics communities.  
 
The EMDS steering committee can invite individuals to join the steering committee and the EMDS organization on GitHub. New members can be added to the steering committee by a consensus vote.
 
Granting new “Owner” privileges to a member of the organization should not be done without the knowledge and approval of the steering committee. The EMDS organization should have at least two members with “Owner” privileges at any time.
 
### Removing members <a name="gov_remove_mem"></a>
 
Members of the EMDS organization on GitHub are free to leave the organization at any time. The organization Owners have the ability to remove members from EMDS on GitHub. Owners should not remove members from EMDS without majority approval of the steering committee. Exceptions are allowed for emergency situations.
 
Members of the EMDS steering committee are free to leave the steering committee at any time. Members of the steering committee can be expelled by a majority vote of the steering committee.
 
### Votes <a name="gov_votes"></a>
 
A simple majority of the steering committee is required for a quorum. A vote may be conducted in-person, via a virtual conference, or via a voting tool such as Google Forms.


## Release number
Release number: LLNL-WEB-836141
