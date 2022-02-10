# Agent-
Sensitivity Analysis Primer

------

This document will describe how to (1) add new datasets for agent-based sensitivity analysis framework and (2) how the agent-based system works.

## 1) Adding new datasets.

The system expects data in the ensemble timeseries (ents) format, essentially as a specific collection of CSV files. Details on the ents format can be found [here](https://github.com/rampvisdevelopment/Ensemble_Time_Series_Format)

So the steps for adding a new dataset are:

0) Have data the ents format. Come up with a **name** for your dataset
1. Upload dataset in either zip format or on a service where they may be downloaded as a zip. When the SCRC data-pipeline is complete this can be used, until then upload to github works, [here is an example](https://github.com/rampvisdevelopment/ents_example_dataset).

2. Obtain URL/api endpoint for the dataset in zip format. For example: Github repos can be downloaded as zip using a url as *https://github.com/rampvisdevelopment/ents_example_dataset/archive/main.zip* 

3. Locate urls.json in [rampvis-api](https://github.com/ScottishCovidResponse/rampvis-api) (/data-api/manifest/urls.json). And equipped with the URL in step 2 add an entry in urls.json which looks like this for the example above
   ```{  
   "name": "ents example data set",  
   "url": "https://github.com/rampvisdevelopment/ents_example_dataset/archive/main.zip",  
   "save_to": "ents_format_datasets/example"  
   },```
   `name` is a free form text just used to inform anyone looking at the document what is being downloaded . `url` is the URL to the ents dataset as a zip file, and **IMPORTANT** `save_to` tells the programme where it should download the file, and for a dataset **named** `name_of_dataset` `save_to` should be `ents_format_datasets/name_of_dataset`, it is very important to use the name of the dataset here.

4. Good job, the dataset will now be downloaded correctly. But the analysis agents must be informed of how you would like this dataset to be analysed. 
   For example: how many clusters would you like to be found, what output parameters are you interested in, what scalar features of the output would you like studied, and do you hypothesise any important interactions between the parameters. This is information is supplied using the sensitivity inventory file, stored [here](https://gist.github.com/rampvisdevelopment/acb6a1e6e33d0358553d7de09d6232e0) currently. Make a new entry in this file for your dataset of the form 
   ```
   {
           "name": "name_of_dataset",
           "k": [
               3,
               4,
               5
           ],
           "metric": [
               "euclidean"
           ],
           "quantities_of_interest": [
               {
                   "name": "inc_case",
                   "mean": "inc_case_mean",
                   "variance": "inc_case_var"
               },
               {
                   "name": "inc_death",
                   "mean": "inc_death_mean",
                   "variance": "inc_death_var"
               },
               {
                   "name": "age_6_H",
                   "mean": "age_6_H_mean",
                   "variance": "age_6_H_var"
               },
               {
                   "name": "age_7_H",
                   "mean": "age_7_H_mean",
                   "variance": "age_7_H_var"
               }
           ],
           "scalar_features": [
               "sum",
               "max"
           ],
           "interactions": [
               "p:p_s o:+ p:p_inf",
               "p:p_s o:* p:p_inf",
               "p:p_s o:/ p:p_inf",
               "o:( p:p_s o:+ p:p_inf o:) o:* p:p_s"
           ]
       }
   ```
   Where `name` should match `name_of_dataset` in urls.json. `k` is the number of clusters to be used in the analysis, it must be a list of integers. `quantities of interest` contains the output quantities to be analysed and the information needed for this analysis: the names of the columns in the ents format containing the mean and variance of the quantity respectively. `Scalar features` are a list of the scalar features of the time series which are used for working out the Sobol indices with respect to these quantities and the y axis on the small multiple plots. Currently implemented options are: the `sum` and `max` of the time series. `interactions` are a list of combinations of paramters which one is interested in studying, these are computed and the values of the interaction parameters are shown alongside the model parameters in the analysis. `interaction` are a list of interactions as strings, expressed by putting a prefix p: before a parameter name and o: before an operation, allowed operations are: `+,-,*,/,(,)` so if you are interested in what value parameter P_1 times P_3 divided by P_3 takes for different outputs you would write `"p:P_1 o:* p:P_2 o:/ p:P_3"`.

5. Now the analysis will be performed on your data set. To see the data in the infrastructure-ui tell a member of the rampvis team to register the stream, you will then be able to propagate the visualisations to the new data.
   
   ## 2) How the system works
   
   **The function of the tool is to automatically analyse model data to produce data for a range of sensitivity analysis plots** 
   
   The system uses a collection of software agents, pieces of code which act independently to perform some action, to perform analysis and data administration tasks of all the datasets in the system. Each software agent performs a simple task when prompted (by a scheduler at the moment), the result of the collection of software agents is to allow a flexible flow of data through the system where complexity is compartmentalised in the agents. Flexible as the agents simple tasks can be adapted for different analysis. The agents can be seen in `data-api/app/controllers/agents` of the [rampvis-api](https://github.com/ScottishCovidResponse/rampvis-api/tree/main/data-api/app/controllers/agents). The different types of data in the system are distinguished by their prefix and suffix, with the scheme being that the prefix tells you about the output quantity the data stores and the suffix the type of the data, sometimes an additional suffix to do with what parameter took a special role in the analysis is included. For an overview the agents in the system as of 2022-01-31 may be shown pictorially as.
   
   ![alt text](figures/schematic.svg)

Where the orange boxed correspond to agents, and black arrows denote that the object at the end of the arrow is created by the object at the beginning og the arrow. Coloured arrows denot that the data at the start of the arrow is read/used by the object at the end of the arrow.
