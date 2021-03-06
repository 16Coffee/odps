# Billing {#concept_f32_fpm_xdb .concept}

**MaxCompute is billed according to:**

-   The projects you create and the items you use within each project. 

-   Billable items can include the storage, computing resources and data downloads of a project.

-   MaxCompute fees are calculated daily.


## Storage pricing {#section_efw_kpm_xdb .section}

The data that is stored in MaxCompute, including tables and resources, is billed according to the storage used. The billing cycle is **one day**.

MaxCompute records the storage used for each project on an hourly basis. The average storage over the day is then calculated for each project space at the end of each day.

The daily MaxCompute fee is calculated by applying the tiered unit prices in the table below to the average storage used. Up to 1 GB of storage is free each day, while the next 99 GB of storage costs 0.0028 USD for each gigabyte and so on. Storage that exceeds 100 GB but does not exceed 1 TB will cost $0.0014 per Gigabit, storage that exceeds 1 TB but does not exceed 10 TB will cost $0.0013 per Gigabit, storage that exceeds 10 TB but does not exceed 100 TB will cost $0.0011 per Gigabit, storage that exceeds 100 TB but does not exceed 1 Pb will cost $0.0009 per Gigabit, the cost of more than 1 PB of storage will require you to contact us through a job order.

|Less than 1 GB USD/GB/Day|1 GB to 100 GB  USD/GB/Day|100 GB to 1 TB USD/GB/Day|1 TB to 10 TB USD/GB/Day|10 TB to 100 TB USD/GB/Day|100 TB to 1 PB USD/GB/Day|More than 1 PB USD/GB/Day|
|:------------------------|:-------------------------|:------------------------|:-----------------------|:-------------------------|:------------------------|-------------------------|
|Free|0.0028|0.0014|0.0013|0.0011|0.0009|Please contact us|

For example, if you store 50 TB data in MaxCompute, the bill is calculated as follows.

```
(100GB-1) x 0.0028 USD/GB/day
+(1024-100)GB x 0.0014 USD/GB/day
+(10240-1024)GB x 0.0013 USD/GB/day
+(50 x1024-10240)GB x 0.0011 USD/GB/day
=58.61 USD/day
```

**Note:** 

-   Because MaxCompute compresses and stores user data, the bill is based on the capacity size of the data after compressed. This means the size of the stored data is different from the size of the data as measured by you before storage. The compression ratio is generally about 5.

-   Generally, MaxCompute fees are deducted no more than 6 hours after the daily fee calculation is completed,  and are automatically deducted from the corresponding account balance. 

-   On the MaxCompute console, you can view your consumption details under **Bill Management**.


## Computation pricing {#section_lfw_kpm_xdb .section}

MaxCompute supports two kinds of billing methods.

-   Pay-As-You-Go:  Each task is measured according to the input size by job cost.

-   Subscription \(CU cost\):  Users can subscribe the usage of a part of resource in advance. This method is only supported on **Alibaba Cloud DTPlus Platform**.


Currently,  MaxCompute supports the following computing task types: SQL, UDF, MapReduce, Graph, and machine learning. Charges apply for SQL \(excluding UDF\) computing tasks and for MapReduce tasks \(charges introduced 2017-12-19\). There are no plans to charge for other types in future. 

**Pay-As-You-Go**

Pay-As-You-Go is for SQL tasks and MapReduce tasks.

SQL tasks are paid after volume, I .e. SQL is charged after I/O:

Every SQL task is billed according to **Data Input size** and **SQL Complexity**. Once the SQL task is completed, MaxCompute sends its metering information to the billing system, which calculates the fee and adds it to the next payment.

The MaxCompute SQL task is charged according to I/O for each job. All daily measurement information is paid in the next day.

The billing of a SQL task is calculated as follows.

```
 Computing Cost of One SQL = DataInputSize x SQLComplexity x SQL Price
```

The current ：

|price for | a |
|:---------|:--|
|SQL task | 0.0438 USD/GB.|

-   **Data Input Size**: The actual size of the data that an SQL statement scans. Most SQL statements have partition filtering and column pruning, so this value is generally less than the source table data size.

    -   Column pruning: For example, the submitted SQL is `select f1,f2,f3 from t1;`.  Only the data size of three columns \(f1, f2, and f3\) in t1 are charged.

    -   Partition filtering: For example, a SQL statement includes  ds\>”20130101”. The “ds” is a partition column. The data size is calculated only according to the read partition, rather than the data of other partitions, and then billed.

-   **SQL Complexity**: First MaxCompute counts keywords in SQL statements, and then converts to SQL complexity. The details are shown as follows.

    -   SQL keyword number = join Number + group by number + order by number + distinct number + window function number +  max \(insert into Number -1, 1\)

    -   SQL complexity calculation:

        -   If SQL keyword number is less than or equal to 3, the complexity is 1.

        -   If SQL keyword number is less than or equal to 6, the complexity is 1.5.

        -   If SQL keyword number is less than or equal to 19, the complexity is 2.

        -   If SQL keyword number is greater than or equal to 20, the complexity is 4.


For example, the input SQL statement is shown as follows.

```
cost sql <SQL Sentence>;
```

Examples are as follows:.

```

odps@ $odps_project >cost sql SELECT DISTINCT total1 FROM
(SELECT id1, COUNT(f1) AS total1 FROM in1 GROUP BY id1) tmp1
ORDER BY total1 DESC LIMIT 100;

Complexity:1.5
```

The preceding SQL includes 4 keywords \(one DISTINCT, one COUNT, one GROUP BY, and one ORDER\),  so the SQL complexity is 1.5.  If the data volume of table “in1” is 1.7 GB, then the actual consumption is shown as follows.

```
1.7 x 1.5 x  0.0438 = 0.11 USD
```

**Note:** 

-   The bill invoicing time is usually before 06:00 the next day. After the computing task successfully ends, the system counts the data size and SQL complexity. After the bill is generated, the fee is automatically deducted from your account. If the SQL task is unsuccessful, no fee is deducted.

-   Similarly to storage, SQL computing also calculates and bills the data size after compression.


Pay-As-You-Go for MapReduce

In December 19, 2017, MaxCompute began charging for MapReduce tasks. The billing of a MapReduce computing task is calculated as follows.

```
Computing Cost of One MR task = TotalTime x MR Price(USD)
```

The current：

|price for |Price|
|:---------|:----|
|MR price| 0.0690 USD/Hour/Task.|

The calculating time for one successful MR task is: Execution time \(hours\) × Number of cores that task calls

If one MR task calls 100 cores, and the tasks takes 30 minutes to complete, the calculating time for the MR task is: 0.5 hours × 100 cores = 50 hours.

After the MR task is finished, MaxCompute sends its metering information to the billing system, which calculates the fee and adds it to the next payment.

**Note:** 

-   You are not charged if a task fails to run.

-   The calculating time does not include the time waiting for execution.

-   If you purchased the MaxCompute Subscription service, you can use MR computing tasks for free within the range of the services you purchased. No additional fee is charged.


**Subscription \(CU cost\)**

Payment by subscription is only available on the **Alibaba Cloud DTPlus Platform**.  Subscription allows you to pay an initial fee \(monthly or annually\) for your entire reserved resources. The basic unit of such resources is defined as CU \(Compute  Unit\). One CU includes 1 core CPU and 4 GB of memory.

|Resource definitions|Memory|CPU|USD/month|
|:-------------------|:-----|:--|:--------|
|1 CU|4GB|1 CPU|22.0|

We recommend that new users use the Pay-As-You-Go billing method, since this allows you to gauge your resource usage without unnecessary costs.  Payment by subscription is only available on the Alibaba Cloud DTPlus platform.

## Pay-As-You-Go data download {#section_fgw_kpm_xdb .section}

You can download data from the extranet through the MaxCompute Tunnel.  The billing of a data download is calculated as follows.

```
Download Cost from Extranet/time = Downloaded Data Volume x Download Price
```

The specific price is as follows:

|The current |USD/GB|
|:-----------|:-----|
|price for data download is |0.1166 USD/GB.|

**Note:** 

-   MaxCompute sends you messages to notify you of the size of your downloads, and to provide you with your download costs the next day.

-   Download data volume refers to the size of an HTTP body for one download request. The HTTP body that carries data uses protobuffer encoding, so it is generally smaller than the original data size, but larger than the data stored in MaxCompute after compression.

-   The different billing methods are applicable to different network environments, such as public networks, classic networks of Alibaba Cloud, or VPC networks. For more information, see Access domains and data centers.  For more information, see [Access domains and data centers](https://www.alibabacloud.com/help/zh/doc-detail/34951.htm).


