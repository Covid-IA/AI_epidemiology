# Using Azure Batch

This folder includes a Python notebook that handles all the automation of Azure Batch.

Azure Batch allows you to define a **Pool** of machines running any operating system. These machines will sit idle until some task is assigned to them. We configure **Jobs** to be run on those machines (on demand, or on a schedule). Each job is a collection of **Tasks**. A task is just some execution of a work item (usually the execution of a application, like rendering a frame on an animation job, or processing a subset of data on large data science projects). As we create jobs and tasks, Azure Batch will manage the execution of these work items within the pool.

Everything can be setup on the portal, but a good practice is to automate all steps through code.

Inside the sample_application folder there is a Python application built with no information of Azure Batch or Azure Storage. It simply reads one of the files in the input_data folder, applies some transformations and writes an output file. It's a dummy application built as an example. Nevertheless, we can make this application run in Azure Batch and retrieve/store data from Azure Storage without changing its code.

## Running the demo

While executing the **Covidia Batch** notebook several steps are taken:

1. The configuration file settings.json is read to configure access to the Azure Batch account and a support Azure Storage account where we keep all the needed files

2. In the Storage Account we create 3 containers:
    - application: to store the application we want to run in Batch
    - input: to store any input files for the tasks
    - output: to store the output of each task

3. We zip the contents of the sample_application folder and upload it to the application container in Azure Storage
4. We create a shell script to install Python and we upload it to the application container in Azure Storage
5. Using the Azure Batch Python SDK, we create a pool of machines based on the chosen configuration - we also setup a StartTask at the Pool level which will make sure Python and Pip are installed in each of the nodes. As machines are added to the pool, this task will be run to ensure each node is ready once a task is started.
6. We create a Job to gather all our tasks - a Job Preparation Task is defined to download the sample_application from Azure Storage, decompress it and copy it to a known location. We also install any python packages required by the application through it's requirements.txt file
7. We create one task per file in the input_data folder, essentially creating 60 tasks. Each task consists of a single execution of the sample_application passing one of the files as input. The output files are also collected and automatically sent to Azure Storage and kept on the output container
8. After waiting for all tasks to complete, we clean up the job and pool to drop the cost to zero.

## How to adapt to your own Script

- It's easy to change the notebook to compress and upload your application instead of the sample_application.
- Your application should include a requirements.txt file with all the packages that are needed.
- Change the settings.json file to define the size of the pool, name of the jobs, etc.
- The only part that really needs changing is the Tasks generation as that is specific to your application (e.g which arguments to pass, how outputs are collected, etc.)
