
# Create-Deploy-Cloud-Data-Fusion-Pipeline

  • Configure Cloud Data Fusion  

  • Create a Cloud Data Fusion data transformation pipeline  

  • Connect Cloud Data Fusion to a couple of data sources  

  • Apply basic transformations  

  • Join the two data sources using Cloud Data Fusion  

  • Split data to perform an A/B experiment  

  • Write data to a sink


You want to create custom marketing materials for an ongoing campaign promotion, and you want drivers to distribute the materials directly to customers' home mailboxes.

Your campaign has two constraints:

Location: You only deliver to customers in California, New York, and Alabama.
You need to create a random parameter for each customer to be able to perform an A/B test and split the customers into two groups.

### A/B Testing:
An experiment where two or more variants of a material are shown to random users, and statistical analysis is used to determine which variation performs better for a given conversion goal.

To generate the list of customer addresses for the campaign, you use:

A Data Pipeline to create a pipeline that joins the customer data with a public dataset that contains state abbreviations. You then store the cleaned and joined data in a BigQuery table. You can query the table using BigQuery or analyze it with with Google Data Studio.

The function, positions, to extract characters from a field.


Check project permissions
Before you begin your work on Google Cloud, you need to ensure that your project has the correct permissions within Identity and Access Management (IAM).

In the Google Cloud console, on the Navigation menu (nav-menu.png), click IAM & Admin > IAM.

Confirm that the default compute Service Account {project-number}-compute@developer.gserviceaccount.com is present and has the editor role assigned. The account prefix is the project number, which you can find on Navigation menu > Home.


If the account is not present in IAM or does not have the editor role, follow the steps below to assign the required role.


On the Navigation menu, click IAM & Admin > IAM.

At the top of the IAM page, click Add.

For New members, type:

{project-number}-compute@developer.gserviceaccount.com

Replace {project-number} with your project number.

For Role, select Project > Editor. Click Save.


## Enable the Cloud Data Fusion API

Enable the required Cloud APIs

Cloud Data Fusion requires that the BigQuery, Cloud Storage, Dataproc, and Cloud Data Fusion APIs are all enabled. Most of these APIs are enabled by default for you in Qwiklabs but you must manually enable the Cloud Data Fusion API.

In the Google Cloud Console, click Activate Cloud Shell (Cloud Shell) in the upper right corner and issue the command below. It will take a few minutes to complete. A message will be be returned that the API enablement operation successfully completed (your operation identifier will vary). Your output will appear similar to the screenshot that follows the command.

` gcloud services enable datafusion.googleapis.com `

datafusionapi

Note: This may take up to 3 minutes.
When you enable the API, the Cloud Data Fusion option is added to the Navigation menu in the Cloud Console.

## Create a Cloud Data Fusion instance

Cloud Data Fusion creates Google-managed service accounts for the Cloud Data Fusion services associated with your instances. When your instance has been created, you must add this account to the Service Management > Cloud Data Fusion API Service role for your project to allow it to manage your Cloud Data Fusion instances. These steps are detailed for you in the next section.

Cloud Data Fusion executes pipelines using ephemeral Cloud Dataproc clusters created in your project.

Create a Cloud Data Fusion instance
The Cloud Data Fusion console shows an Instances page where you can manage your Cloud Data Fusion instances. When no instances exist, the page has a link to create an instance, along with some useful links to documentation and samples.

In the console, in the Navigation menu, click Data Fusion.
Click Create an Instance.
Name your instance marketinganalytics.
The name must start with a lowercase letter followed by up to 63 lowercase letters, numbers, or underscores.
Set the zone to us-central1.
Set the Edition to Basic.
For information about editions, see the pricing page.
Leave all other options at the defaults settings:

Click Create.

Note: It takes up to 30 minutes to fully complete the instance creation process. You can proceed to the next step, setting up permissions for the Data Fusion service account while the instance is being initialized.

While you wait, watch this introduction to Cloud Data Fusion at Google Next 2019 for more information on what Cloud Data Fusion can do for you and how it works, and then set up permissions for the Cloud Data Fusion service account.


Set up permissions for the Cloud Data Fusion service account
When the instance is complete, the Google managed service account associated with the instance must have the permissions that it requires on your project. You can grant these permissions while the instance initializes.

Navigate to the Instance detail page by clicking the instance name.


Copy the Google managed service account name that has been assigned to your Cloud Data Fusion instance.


On the Navigation menu, click IAM & admin Page > IAM to open the Permissions page for your project.
This page is where you grant the required permissions to the Google managed Cloud Data Fusion service account by adding it to the Cloud Data Fusion API Service role for your project.

On the IAM Permissions page, click Add at the top.

Paste the Service Account name you copied in the last step into the New Members text box.

In the Select a Role drop down navigate to Service Management > Cloud Data Fusion API Service Agent or simply start typing Cloud Data Fusion API Service Agent and select the role when you can see it.


Click Save.

Once these steps are done, return to the Cloud Data Fusion console (Navigation menu > Data Fusion) and wait for the instance to finish initializing.

You must grant permissions using IAM to the Google managed Cloud Data Fusion service account to allow it to deploy resources in your projects for your Cloud Data Fusion pipelines.


Click View Instance to start using Cloud Data Fusion.

Alternatively, click the Cloud Data Fusion URL on the Instance details window.
Note: It takes up to 30 minutes to complete the instance creation process and the instance might not respond for a few minutes after the console indicates that initialization is complete. 

## Prepare input datasets in Cloud Data Fusion

We use 2 input datasets:

Sample customer data: A CSV file customers.csv, located in the public bucket named campaign-tutorial.
State abbreviations: A BigQuery table state_abbreviations in the campaign_tutorial dataset in the public BigQuery sample datasets.
These datasets are pre-configured in your Cloud Data Fusion instance in Wrangler.

Load the customer data
The source data for this lab is stored in a CSV file in a Cloud Storage bucket. In this pipeline, Cloud Data Fusion reads data out of a storage bucket that you will create and use to store the source data CSV file.

In Cloud Shell, execute the following commands to create a new bucket and copy the source data into it:

` export PROJECT_ID=$(gcloud info --format='value(config.project)') `

` gsutil mb gs://$PROJECT_ID `

` gsutil cp gs://cloud-training/CBL179/marketing-data/crm_customers.csv gs://$PROJECT_ID `

Execute the following command to create a second bucket for the temporary storage that Cloud Data Fusion requires.

` gsutil mb gs://$PROJECT_ID-temp `

In the Cloud Data Fusion browser tab, click the Cloud Data Fusion menu in the top left.
A panel opens on the left. Use this panel to access pre-configured connections to your data, including the Cloud Storage connection.

On the left panel, click Wrangler, and then, still in the left panel, click Google Cloud Storage > Cloud Storage Default.

On the right side of the console you see a list of buckets in the Cloud Storage connection, select the bucket that matches your Project ID.

Your Project ID is shown in the left panel of the lab.
Select the crm_customers.csv file.
The crm_customer data loads into the Wrangler screen in row/column form.

Clean the customer data
Organize the data by parsing the customer data into table format, setting the schema, and filtering the customer data to present only the target audience you need.

Transform the customer data
In the body drop-down, click Parse > CSV.
Set the delimiter to comma, and then click Apply.
The data splits into multiple columns.


Because you don't need the body column , remove it from file. Click the body drop-down, and then click Delete column.

There are other columns you don't need for this tutorial. In the next steps you will remove: first_name, last_name, gender, job_title, age.
In the columns tab on the right, click the boxes to select the body_2 to body_6 columns. deletecolumn.png

Click on the drop-down for any of the selected columns and then click Delete selected columns.

Set the schema of the data by assigning appropriate names to the table columns. Instead of bodyXX, rename the columns to capture the information they present.

In the right pane, in the Columns tab, click the Column names drop-down and select Set all.
In the Bulk set column names dialog box, enter the following:
Id,City,State,Country

Click Apply.

Filter the customer data
The first row is a duplicate header. To delete it, filter it out.

Click the Id drop-down > Filter.
In the filter window, do the following:
Click Remove rows.
In the If drop-down, select value is.
Enter the value: id
Click Apply.

Finally, filter the data to display only customers that live in California, New York, or Alabama. To do this, remove all rows that contain values other than these three states.

Click the State column drop-down > Filter.
In the filter window, do the following:
Click Keep rows.
In the If drop-down, select value matches regex.
Enter the following regular expression:
^(California|New York|Alabama)$

Click Apply to filter.
The values in the State column are California, New York, or Alabama.

## Create the Cloud Data Fusion pipeline
You've cleaned your data, and then run transformations on a subset of your data. Now create a batch pipeline to run transformations on all your data.

Note: Cloud Data Fusion translates your visually built pipeline into an Apache Spark or MapReduce program that executes transformations in parallel on an ephemeral Dataproc cluster. This enables you to easily execute complex transformations over vast quantities of data in a scalable, reliable manner, without having to wrestle with infrastructure and technology.
Still in Wrangler, click Create a Pipeline in the top right.
In the Create a Pipeline dialog, select Batch pipeline.
Pipeline Studio opens.

In the top left, make sure Data Pipeline - Batch is the pipeline type.
The Data Pipelines window shows a GCSFile source node connected to a Wrangler node.


The Wrangler node contains all the transformations you applied in the Wrangler view captured as a directive grammar.

Hover over the Wrangler node and click Properties to see the Wrangler details.
At this stage, you can apply more transformations by reopening the Wrangler interface.

For example, create a column with the last digits of each Id which we can use for our A/B testing.

To reopen the Wrangler interface, click Wrangle in the Directives section.
Click the Id drop-down > Extract fields > Using positions.
Select the last digit of an id.
A box opens, name the new column: last_digit.
Click Apply.

Click Apply in the top left to return to Wrangler Properties window. You can now see that cut-character Id last_digit 10-10 has been added to the Wrangler Recipe.

Click Validate on the top right to Save the changes.

To close the Wrangler area, click X in the top right.

Abbreviate the state names
In the lab, the navigation system only recognizes addresses that contain abbreviated state names (CA not California). Currently, your customer data contains full state names.

The public state_abbreviations BigQuery table contains two columns: one with the full state names and one with the abbreviated state names. Use this table to update the state names in your customer data.

View the state names data in BigQuery
View the data in BigQuery:

In the Cloud Console, in the Navigation menu, click BigQuery.
BigQuery opens in the Cloud Console.

Enter the following query in the Query Editor, and then click Run:
SELECT * FROM `dis-user-guide.campaign_tutorial.state_abbreviations`
In the Query results, you see the state_abbreviations table that has the names of all states in the United States and their abbreviations.


Access the BigQuery table
To access the state_abbreviations table from the data fusion instance, add a source in your pipeline to access this BigQuery table.

In the Cloud Data Fusion Studio console, in the left panel, choose the BigQuery source from the Source section.
If necessary, click the Cloud Data Fusion menu in the top left to open the left panel.
big-query-source.png

A BigQuery source node appears on the canvas with the two other nodes.

Hover over the BigQuery source node and click Properties.
Set the following fields, leave all others at the default value:
Field	Description
Label	state_abbreviations
Reference Name	state_abbreviations (used to identify this data source for lineage purposes.)
Project ID	your GCP Project ID found in the left panel of the lab
Dataset Project ID	dis-user-guide
Dataset	campaign_tutorial
Table	state_abbreviations
Temporary Bucket Name	gs://[PROJECT-ID]-temp where [PROJECT-ID] is your GCP Project ID
The BigQuery Project, Dataset and Table names reference the table you queried previously in BigQuery.


To populate the schema from BigQuery, click Get Schema.
Scroll to the bottom of the dialog to view the Output Schema.


To save this schema, click Validate and click X to close the BigQuery Properties dialog.

Join the two data sources
Now you can join the two data sources—customer data and state abbreviations—to generate output that contains customer data with abbreviated state names.

In the left pane, click Analytics > Joiner.

A Joiner node appears on the canvas.

Drag a connection arrow > from the right edge of each of the source nodes and drop it on the destination node to connect the Wrangler node and the BigQuery node to the Joiner node.
You may have to arrange the nodes on the canvas.
Your pipeline should look similar to this:


Configure this new Joiner node, which provides functionality that is similar to a SQL JOIN.

Click Properties on the Joiner node.

In the Join section

In the state_abbreviations drop-down, uncheck name, and check abbreviation.
Change the alias for the abbreviations field to State for easy identification later
In the Wrangler drop-down, check Id, City, and last_digit. Uncheck State and Country.
You can define aliases here to disambiguate any duplicate column names. Since you want only the abbreviated state name and not the full name, exclude the State field from the Wrangler node and the Name field from the State Abbreviations node. The abbreviated state name is present in the abbreviation field in the State Abbreviations node.
Leave the Join type as Outer.
Check Wrangler as the only required input.
Set the Join condition to join the name column from the State Abbreviations node to the State column from the Wrangler node.

Click Get Schema to generate the schema of the resulting join, and then click Validate.
View the Ouput Schema on the right pane:


Click X to close the Joiner Properties dialog.
You're ready to proceed to the final step of adding a sink to this pipeline.

Store the output to BigQuery
You store the result of your pipeline into a BigQuery table in a sink.

In the left pane, click Sink > BigQuery.
A BigQuery node appears on the canvas.


Connect the Joiner node to the BigQuery node.

Hover over the BigQuery node and click Properties.

Set the following fields and leave all others at their default value.

Field	Value
Label	BigQuery Customer Data
Reference Name	customer_data_abbreviated_states (identifies the data source for lineage purposes)
Project ID	autodetect
Dataset	dis_user_guide
Table	campaign_targets (automatically created when the pipeline runs)
Temporary Bucket Name	gs://[PROJECT-ID]-temp (Replace [PROJECT-ID] with your GCP Project ID)
Click Validate to save the BigQuery Properties .

Click X to close the BigQuery Properties dialog for the BigQuery sink.

On the action bar, click Configure.

In the left pane, click Engine config, and then click Spark to run your batch pipeline.


Click Save.

At the top left, name your pipeline CampaignPipeline and click OK.


Note: Do not use any spaces in the pipeline name as they will prevent you deploying the pipeline.

Once you have configured a suitable pipeline name you can test the pipeline using the Preview and then Run toolbar icons. If you enable the preview option to test the pipeline you will have to disable it again before moving on to the next step, deploying the pipeline.

Click Save in the action bar to save the pipeline.
That's it. You've created your first pipeline and can deploy and run the pipeline.

## Deploy and run the Cloud Data Fusion pipeline

To deploy Campaign Pipeline:

In the action bar, click Deploy.

Once deployed, click Run and wait for the pipeline to run to completion.
Note: When you run a pipeline, Cloud Data Fusion provisions an ephemeral Dataproc cluster, runs the pipeline, and then tears down the cluster. This will take a few minutes as it starts up an ephemeral Dataproc cluster to run the pipeline.
You can observe the status of the pipeline transition from Provisioning to Starting to Running to Deprovisioning to Succeeded during this time.

If you click Logs you can see much more detail about what is happening as Cloud Data Fusion manages the life cycle of the pipeline.

View the results
To view the results after the pipeline runs:

Switch back to BigQuery and query the dis_user_guide.campaign_targets table in the Query editor.

SELECT * FROM dis_user_guide.campaign_targets

You can now use the last digit column to perform your experimentation. For example all even numbers can be used for test A and all odd numbers can be used for test B.

## Conclusion:

We have built a use case end to end with Cloud Data Fusion.
This is an example that shows you how you can effectively activate the data once ingested and analysed.
