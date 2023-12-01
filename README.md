
# Batch Processing in Mule4

## Introduction

This blog is to discuss about Mule4 Batch Processing, how to implement batch and nested batch processing and its usecases, limitations, considertaions with respect to thread pooling.

### Batch Processing

Mule Batch Processing is designed for reliability and processing large sets of data asynchronously in batches. Batch has mule components internally to process large sets of data. Batch has different componete for processing bulk/large set of data like  Batch Job, Batch Step, Batch aggregator, and on-complete. 

Batch Job automatically splits the source data into batches and stores it in persistent queues in internal memory of workers. This gives provision to process large sets of data asynchronously and along with reliability. In the event of application crash/redeployment there will not be any data loss and application continious to process remaining data where it was stopped. 

### Batch Architecture
Batch distributes the large sets of data/records into batches and then process those records and issue a report after completion. Process has one or more batch steps, optional batch aggregators as shown in the architecture snapshot,

![image](https://github.com/KathaSudharshan/mule-batch-processing/assets/138109855/7a614170-f5f9-4fe4-b487-5b5ade188e4a)


As shown in the architecture batch data processed internally sequence processing by default and this can be changed to round-robbin in batch job configurations. Each record is similar to a Mule event: Processors can access, modify, and route each record payload using the payload keyword, and they can work on Mule variables using vars. However, they cannot access or modify Mule attributes (attributes) from the input to the Batch Job component.

### Batch Job and Phases
Batch Job has 3 different phases while processing bulk data/records. Those are like,
- Load and dispatch
- Process Phase
- On-Complete

#### Load and Dispatch
In this phase batch job automatically splits the source data and stores it in persistant queues and the records will be ready for processing in batches based on the batch size configuration. This phase takes place in Batch Job. Each batch has its own assigned batch instance id to tarck the status of respective batch. By default records process sequentail order and can be changed round-robbin at batch job level configurations.

#### Process Phase

In this phase, batch steps component retrieves records concurrently from the persistance queue based on the batch size. By default batch size is 100.

Batch step process data parllely are batch step block level and withing batch step it process data sequentially, can be accessed, modified payload and variables. Batch aggregator also can be used to process transformed payload to target system as bulk using either size or stream. 

After completion of each record processing the data will be sent back to queue again for further batch steps to finish the same process until all the batch steps are completed.

Same record sends to all batch steps in sequentially and this tracking managed by Mule batch job internally to track the status. Scope of variables are within batch step only and  can not be accessed from other batch steps or even in the on-complete phase.

By defauly batch processing stops If any record failed while processing a batch due to by default failure count(maxFailedRecords) configurations at Batch Job level. Can be changed to specific failure count(maxFailedRecords) number or either mention -1 to continue the batch processing even after the failures in bathc data.

Please find the below snapshot for batch steps sequence processing,
![image](https://github.com/KathaSudharshan/mule-batch-processing/assets/138109855/d9c32c71-308d-4e9e-adde-b007b14cb693)

#### Batch aggregator
Batch Aggregator is part of Batch Step phase. Aggregator can be used to process bulk or set of data to target systems like Database, Salesforce, cloud, SFTP servers etc. 

Batch aggregator has 2 types like size based or stream. Size based is used to process block of records to Database or Salesforce etc. Streaming can be used to process data as soon as available instead of storing in blocks and streaming process runs until completion of all records. Batch aggregator accepts array.

Flow with Batch Aggregator with size as below,
![image](https://github.com/KathaSudharshan/mule-batch-processing/assets/138109855/75f75adf-5641-4473-9c3e-9f6a641b2bfa)

Flow with Batch Aggregator with stream as below,
![image](https://github.com/KathaSudharshan/mule-batch-processing/assets/138109855/78ff1cde-2457-4d75-9aa3-c1de35a738fb)

### On-Complete Phase
This is the last phase of Batch Processing and this phase is used to create report or summary of batch execution for which it processed for given batch instance id. It helps developers to understand how many records are success or failures. But can not be accessed records and vaiables which created in batch steps.

Below is the sample template of Batch Job and its phases:

```xml
<mule>
<flow name="mule-batch-flow" >
    <!-- Data source -->
  <batch:job name="Batch-job">
  <batch:process-records>
    <batch:step name="Batch-Step1">
      <batch:record-variable-transformer/>
      <ee:transform/>
    </batch:step>
    <batch:step name="Batch-Step2">
      <logger/>
      <http:request/>
    </batch:step>
  </batch:process-records>
  <batch:on-complete>
    <logger level="INFO" doc:name="Logger"
            message='#[payload as Object]'/>
  </batch:on-complete>
</batch:job>
</flow>
</mule>
```
## Use Cases:

- Processing bulk records to target systems/applications like Database, Salesforce, Queue etc.
- Migrating legacy data to new systems.
- Processing Files like CSV/excel/xml etc to target systems like Database, cloud servers etc.

### MaxConcurrency
MaxConcurrency is one of attribute in Batch Job this limits the number of bacthes to process. This value needs to be tweaked based on the data set size or size of the batch block. 


```bash
# NOTE: By default, the Batch Job component limits the maximum concurrency to twice the number of available cores. The capacity of the system running the Mule instance also limits concurrency.
```
## Nested Batch Processing
Nested batch processing is also possible to process sets and its subsets of data in Nested Batches. Here required to consider perofmance and available cores as this uses more threads runs parllely.

Examples of Nested batches considering have large sets and sub sets of data like,
- List of Stores and its Products
- List of Companies and Employees
- List of flight vendors and their filghts



    
