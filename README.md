# priority-pathogens
Welcome to the GitHub repository for the priority pathogen project.

## Set-up

This repository has been set up as an orderly project. Please follow the 
installation instructions for orderly2 available here: 
https://github.com/mrc-ide/orderly2


Also install orderly.sharedfile plugin.

``` 
remotes::install_github("mrc-ide/orderly.sharedfile") 

```

This plug-in allows orderly to read files that outside the orderly
project. In our project, we read the Access DB files that are in the
shared drive, without copying them across to the project.

Note: When you run any of the orderly tasks for the first time, you may get an
error saying that you need to run orderly2::orderly_init("pathway"), please run
the line of code it tells you to.

*IMPORTANT* If you do not want to run any of the tasks yourself, but
still want to use the outputs, please see the instructions on sharing
outputs in the FAQ below. 

### Task 1: db_extraction

*IMPORTANT* This task can only run on a Windows machine because as far
as we know, Microsoft does not provide a (free) Mac driver for 
Access DB. Mac users can use their Windows VM to run this task by
connecting the priority-pathogens repo to Rstudio on their VM. If you
are unsure about the steps, please feel free to message Rebecca or
Sangeeta. 

Once data extractions are complete and you want to compile the databases together:
* Clone the latest priority-pathogens repo;
* Open the priority-pathogens R project on your (windows) machine;
* Edit orderly_config.yml file to replace the "singledb", "doubledb", and 
"doubledb2" fields to the appropriate folder pathways. These fields should 
contain the fully qualified name of the folder where the database files are 
located, *as seen from your machine*. For instance, I have mapped the 
PriorityPathogens shared drive to Y: locally. Hence for me, the entries are 
"Y:/Ebola/databases/Single extraction databases" etc. Note that you
may not need all three enteries. For instance, if all the database
files for a pathogen are placed in the same folder on the shared
drive, then you only need one entry in the orderly_config.yml
corresponding to this location.
* Update the function database_files in shared/utils.R with all the database file names you want to
  include. The comments provide detailed instructions on the format in
  which the files should be listed.
* Then run the following (specifying the pathogen):

```
orderly2::orderly_run("db_extraction",
                        parameters = list(pathogen = "EBOLA"))
```
Replace EBOLA with the pathogen of interest.

This orderly task will :

1. create a new ID for rows of each individual database (so that they
can be put together with unique ids);
2. combine all the individual extraction databases into article,
model, parameter and outbreak .csv files;
3. identify which papers have been double extracted (using duplicated
covidence ids) and create separate article, model, parameter and outbreak files
for single and double extracted papers.
4. produce an errors.rds file which is intended to be helpful during the early
stages of cleaning. This will highlight any entries with missing covidence IDs,
publication dates that appear incorrect, duplicate entries, empty entries,
parameter entries with an empty 'parameter_type' variable, etc. You can address
these errors by adding to the pathogen_cleaning.R script. (You can do more thorough
cleaning later in the db_compilation step.)

Combined data for single extracted papers will be in:
articles_single.csv, parameters_single.csv, models_single.csv, outbreaks_single.csv

Combined data for double extracted papers will be in:
articles_double.csv, parameters_double.csv, models_double.csv, outbreaks_double.csv

These files can be found within the “archive/db_extraction” folder in the 
priority-pathogens directory. 


### Task 2: db_double

This task takes the article, parameter, model and outbreak csv files for the 
double extracted papers and identifies the entries that match and those that 
need to be given back to the extractors to be fixed. To run the task, 

```
orderly2::orderly_run("db_double", parameters = list(pathogen = "EBOLA"))
```
Data that matches between extractors will be in:

qa_matching.csv, model_matching.csv, parameter_matching.csv, outbreak_matching.csv

Data that does not match between extractors will be in:

"qa_fixing.csv", "model_fixing.csv", "parameter_fixing.csv", "outbreak_fixing.csv"

These fixing files will need to go back to the extractors to be resolved.

### Task 3: db_compilation

Once the double extraction fixes are complete, all of the single extracted data
and double extracted data can then be compiled together. This task produces
separate article, parameter and model csv files that will be used for the 
analysis. This step will remove the old IDs generated by the access databases 
(as these were replaced with new IDs in db_extraction) and will also remove the 
names of extractors.

For this task, 

* You will first need to manually add the corrected "qa_fixing.csv", "model_fixing.csv",
"parameter_fixing.csv", and "outbreak_fixing.csv" files to the "src/db_compilation"
folder. Copy them with your pathogen name in the file,
e.g. "ebola_qa_fixing.csv". The exact name does not matter - 
the pathogen is specified in the file name to distinguish it from
the same files created by different groups. 
* Update the "orderly_resource" function in orderly.R to list the files you have
manually copied. This will ensure that these files are available to
the orderly task.
* Update the list fixing_files in orderly.R, creating a new component
  with your pathogen name and add the files here. 
  
  ```
  fixing_files <- list(
  EBOLA = list(
    params_fix = "ebola_params_fixing.csv",
    models_fix = "ebola_models_fixing.csv",
    qa_fix = "ebola_qa_fixing.csv"
  )
  ## LASSA Team update this and uncomment
  ## ,LASSA = list(
  ##     params_fix = "lassa_params_fixing.csv",
  ##     models_fix = "lassa_models_fixing.csv",
  ##     qa_fix = "lassa_qa_fixing.csv"
  ## )
)

  ```
* Then run the following, specifying the pathogen e.g.:

```
orderly2::orderly_run("db_compilation", parameters = list(pathogen = "EBOLA"))
```

The db_compilation task will remove the IDs generated by the access databases,
remove the names of extractors, reorder the columns, and clean the data. If
you need to perform any pathogen-specific cleaning, add this to the cleaning.R
script in the src/db_compilation folder. The results will be in the 
"archive/db_compilation" folder.

### FAQ

#### Sharing outputs across users

Can I still get the outputs of an orderly task that I did not run myself?
Yes, you can. Say you are a Mac user and don't want to run the
db_compilation task. Or say if you have run it on your Windows VM and
want to copy the outputs over to your local machine. This is very easy
with orderly2. 

We will place all outputs that need to be shared across users on the
"orderly-outputs" folder on the shared drive under
"PriorityPathogens".

1. Say Alice runs task 'db_extraction' on her machine and wants to
   share the outputs with Bob. She will first map the shared drive on her machine.
2. Then Alice adds the orderly
   folder on the shared drive as an orderly "location" as follows:
   ``` 
   orderly2::orderly_location_add(name = "pp-network-drive", args
   = list(path = "/Volumes/PriorityPathogens/orderly-outputs/"), type =
   "path")
   ```
Here, "name" can be anything, and "args" should be the fully
   qualified name of the "orderly-outputs" folder as seen from Alice's machine.
   
3. Then Alice "pushes" the output of the "db_extraction" task to this
location as follows
```
orderly_location_push(packet_id, "pp-network-drive")
```
where 'packet_id' is replaced by the id of the orderly run that Alice
wants to push. Alice can find this id by looking at the
archive/db_extraction folder.

4. To get this packet, Bob also adds the orderly-outputs folder as an
orderly location to his orderly project following the instructions
above. Then, Bob runs

```
orderly_location_pull_metadata(location = NULL, root = NULL, locate = TRUE)
```

This allows orderly2 to retrieve the necessary metadata from all
locations.

5. Finally, Bob can pull the outputs from the shared drive
   
```
orderly2::orderly_location_pull_packet(<ids>)
```
where "ids" is the set of ids that Bob wants to pull to your local
machine. Bob can look into the orderly-outputs folder to get the id,
   or ask Alice for it. Alternatively, he can simply run 

   
```
orderly2::orderly_location_pull_packet()
```

which will pull all the outputs to his local machine.

If you encounter any issues with the above steps please reach out to Sangeeta
or Rebecca for help.
