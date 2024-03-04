Please take a look at our project overview video here: https://youtu.be/RI7mAwF30wY

Our project was made by three person data team from the same organization. We manage a P1 capacity with a little over 300 monthly users. In 2024 we are actively evaluating every aspect of Microsoft Fabric and moving over workloads that were previously based on Azure Data Factory and Synapse Dedicated SQL Pools. 

The objective of our Hackathon project was to take a traditional subject matter area for Power BI and see how we would implement a solution using as many aspects of Fabric as we could squeeze in, and obviously to make good use of the AI related offerings in Fabric. Along the way we'll highlight how we used Copilot in our development cycle, as well as how we built a prediction model. 

The subject of our analysis is Accounts Payable, which is relevant to any organization. If you work with Power BI in an enterprise setting, you've probably built or at least seen AP reports. Well, we're going to build this set of AP reports using data sourced from an ERP system, specifically SAP S/4HANA. One of our goals is that any of the thousands of other organizations who use SAP and Fabric could follow along with our implementation and even recreate it with their own data. Along the way we will implement a prediction model that estimates the payment date of all in-progress invoices and therefore whether they will be paid early, on-time, or late. The end result of this model will be written down to a lakehouse and integrated back into our Power BI reports for consumption. So, our overall architecture includes a workspace with three lakehouses in a Medallion setup, several notebooks to transform data and build our predictive model, and then a semantic model with our table relationships and DAX measures, and then a report on top. Everything is built with real-world application in mind. After the competition is over we will continue to develop this solution into something that we truly use in production.

First, we need our source data. An anonymized dataset from a test system will be included with our submission, but for those who want to recreate with data from your own SAP systems, just know that we based everything on a classic ODP extractor named `0FI_AP_4`. This has all of our payable line items, both past history and current open items. Traditionally you would use this data to analyze AP performance and creating aging reports in an external BI system. This extractor can be landed in OneLake using any method you choose. In our case we used an ETL tool that offers native Azure Data Lake gen2 compatibility to land SAP data sources as parquet files into our Bronze lakehouse. Most of these tables you see in Bronze were from initial testing and are not relevant to the end result. To recreate our solution, you will need AP line items. _Optionally_ you can create dimensions as we did for payment terms, document types, and vendors.

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/728c6ca6-40ab-4704-a35e-ba03bbd7a405)

From here, we write the data into a Silver lakehouse as delta tables. We do this with a simple notebook code block that executes a full load every time. 

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/10bb7e16-881c-4f1b-8cd5-e6d8c52a53dc)

From Silver, we create facts and dimensions with business-friendly names and write them down to Gold. We write a lot of view in our existing Synapse warehouse, so we liked the flexibility of doing all our joins and case statements using Spark SQL code blocks.

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/85c40912-4754-4a20-babf-4acf5738eb94)

We used Copilot for Data Factory to create the calendar/date table for the model. This was done in a Dataflow Gen2, where Copilot was able to follw instructions to generate the data table for a given range of dates. Copilot was also able to add custom columns according to our requirements, which worked out quite well.

![Screenshot 2024-02-28 190851](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/15e2968c-684a-46dd-a894-f65b9af28559)

![Screenshot 2024-02-28 190916](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/a2584fce-06d8-4a3e-b787-b09a9477dbfd)

For our prediction model, we have a full video walkthrough of the notebook `NB_AP_EDA_Modeling` here: https://youtu.be/EXAY1rVWJZA

To summarize, the model uses random forest regression to predict the number of days an invoice will be paid before or after it's due date. This number is also bucketed into three categories: early, on-time, and late.

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/b89db10b-a1d5-45e2-a85e-e904b5a1543a)
![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/74156cd0-e414-4936-a06d-850263ab9dc9)
![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/bded1012-eee4-429d-852d-ce37f80ec892)


The semantic model uses several DAX patterns that may be of interest. The structure of our AP data has every relevant date that we would need, such as the document date, due date, posting date, and clearing date. Multiple date relationships are used in the measures. We correclty written measures and a before-Date filter, we can also "rewind the clock" to view what was open or overdue at any date in the past. This is common pattern with payables and receivables reporting.

For example:

```dax
Overdue Balance (GC) = 
    CALCULATE(
        SUM(ap_line_items_scored[amount_group_currency]),
        OR(
            ISBLANK(ap_line_items_scored[clearing_date]),
            ap_line_items_scored[clearing_date] > MAX('calendar'[date])
        ),
        ap_line_items_scored[net_due_date] < MAX('calendar'[date]),
        FILTER(
            ALL('calendar'[date]),
            'calendar'[date] <= MAX('calendar'[date])
        )
    )
```

This measure is flexible when setting a before-date filter, and can show you the overdue amount not only currently, but as of any past date.

To put the finishing touch on our semantic model, we documented all the measures using Copilot. This is a brand new feature from the February 2024 Fabric update that actually released during the Hackathon. Copilot was able to quickly document the logic of each measure with high accuracy and only minimal adjustments needed. We're looking forward to this feature being available for tables and table columns as well. Hopefully in the future Copilot can kick-start the documentation for an entire semantic model with a single click.

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/40ed8465-763d-4960-b530-3edec6b28140)

Our data ends at February 23rd. In our `Aging Line Items` report we can see our open invoices, as well as the result the model has predicted for each one. We see the number of days before or after the due date we are predicted to pay, as well as an overall prediction of whether the invoice will be paid late, early, or on time. This is information the payables team can consider as they are prioritizing their workload and trying to utilize their payment terms correctly for each vendor.

![image](https://github.com/aboerger/Fabric-Hackathon-AP-Payment-Prediction/assets/78093277/9e77184d-b24a-41c1-b574-86b22f966933)


Than you for considering our submission - we had a great time developing this and we'll be following along closely to Microsoft Fabric in 2024.
