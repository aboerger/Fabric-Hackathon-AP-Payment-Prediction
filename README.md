Our project was made by three person data team from the same organization. We manage a P1 capacity with a little over 300 monthly users. In 2024 we are actively evaluating every aspect of Microsoft Fabric and moving over workloads that were previously based on Azure Data Factory and Synapse Dedicated SQL Pools. 

The objective of our Hackathon project was to take a traditional subject matter area for Power BI and see how we would implement a solution using as many aspects of Fabric as we could squeeze in, and obviously to make good use of the AI related offerings in Fabric. Along the way we'll highlight how we used Copilot in our development cycle, as well as how we built a prediction model. 

The subject of our analysis is Accounts Payable, which is relevant to any organization. If you work with Power BI in an enterprise setting, you've probably built or at least seen AP reports. Well, we're going to build this set of AP reports using data sourced from an ERP system, specifically SAP S/4HANA. One of our goals is that any of the thousands of other organizations who use SAP and Fabric could follow along with our implementation and even recreate it with their own data. Along the way we will implement a prediction model that estimates the payment date of all in-progress invoices and therefore whether they will be paid early, on-time, or late. The end result of this model will be written down to a lakehouse and integrated back into our Power BI reports for consumption. So, our overall architecture includes a workspace with three lakehouses in a Medallion setup, several notebooks to transform data and build our predictive model, and then a semantic model with our table relationships and DAX measures, and then a report on top. Everything is built with real-world application in mind. After the competition is over we will continue to develop this solution into something that we truly use in production.

First, we need our source data. An anonymized dataset from a test system will be included with our submission, but for those who want to recreate with data from your own SAP systems, just know that we based everything on a classic ODP extractor named `0FI_AP_4`. This has all of our payable line items, both past history and current open items. Traditionally you would use this data to analyze AP performance and creating aging reports in an external BI system. This extractor can be landed in OneLake using any method you choose. In our case we used an ETL tool that offers native Azure Data Lake gen2 compatibility to land SAP data sources as parquet files into our bronze lakehouse. From here, we write the data into a silver lakehouse as delta tables. We do this with a simple notebook code block that executes a full load every time. From silver, we create facts and dimensions with business-friendly names. We write a lot of view in our existing Synapse warehouse, so we liked the flexibility of doing all our joins and case statements using Spark SQL code blocks.

For our prediction model, Bob has a full video walkthrough of the notebook `NB_AP_EDA_Modeling` here: https://youtu.be/EXAY1rVWJZA

This report uses several DAX patterns that may be of interest. The structure of our AP data has every relevant date that we would need, such as the document date, due date, posting date, and clearing date. Using multiple date relationships we can measure our performance, but with properly written measures we can also rewind the clock and view what was open or overdue at any date in the past. This is common requirement with payables and receivables.

Our data ends at February 23rd. Here we can see our open invoices, as we can see the result the model has predicted for each one. We see the number of days before or after the due date we are predicted to pay, as well as an overall prediction of whether the invoice will be paid late, early, or on time. This is information the payables team can consider as they are prioritizing their workload and trying to utilize their payment terms correctly for each vendor.

Than you for considering our submission, we had a great time developing this and we'll be following along closely to Microsoft Fabric in 2024.