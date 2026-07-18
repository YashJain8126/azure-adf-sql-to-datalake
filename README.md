# ☁️ Azure Data Ingestion Pipeline: SQL to Data Lake

![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure Data Factory](https://img.shields.io/badge/Data%20Factory-3E8EED?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure SQL](https://img.shields.io/badge/Azure%20SQL-00BCF2?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)

> This is my end-to-end data pipeline project in Azure! I built this to learn how to move and manage data in the cloud, specifically taking data from an Azure SQL Database and landing it safely into an Azure Data Lake using Azure Data Factory.

---

## 🏗️ Architecture Overview

I kept the infrastructure organized by putting everything inside a single Azure Resource Group. Azure Data Factory acts as the main engine here—it securely connects to the SQL database, pulls the data, and loads it into my Storage Account.

```mermaid
graph LR
    %% Styling for Azure Resources
    classDef azure fill:#0072C6,stroke:#fff,stroke-width:2px,color:#fff;
    classDef rg fill:#f8f9fa,stroke:#0072C6,stroke-width:2px,stroke-dasharray: 5 5;
    classDef sa fill:#e1f0fe,stroke:#0072C6,stroke-width:1px,stroke-dasharray: 2 2;

    %% Resource Group Boundary
    subgraph RG [Project Resource Group]
        direction LR
        
        %% Resources
        SQL[(Azure SQL Database)]:::azure
        ADF[Azure Data Factory]:::azure
        
        %% Storage Account Subgraph
        subgraph SA [Azure Storage Account]
            DL[Containers / Data Lake]:::azure
            TBL[Table Storage]:::azure
        end
        
        %% Pipeline Flow
        SQL -- "Extracts Data" --> ADF
        ADF -- "Loads Data (Sink)" --> DL
    end
    
    class RG rg;
    class SA sa;
```

## 🚀 What I Deployed

| Resource Type | What I Used It For |
| :--- | :--- |
| **Azure SQL Server & Database** | This is my **Source**. It holds the initial relational data that I needed to move. |
| **Azure Data Factory (ADF)** | The main orchestrator. I created a **Pipeline** here with a "Copy Data" activity to grab data from SQL and send it to the lake. |
| **Azure Storage Account (Data Lake)** | This is my **Sink/Destination**. I used the **Containers** service (ADLS Gen2) to store the raw data files. |
| **Azure Storage Account (Tables)** | I used this to experiment with structured, NoSQL data storage separately from my main data lake files. |

## ⚙️ How I Built It

1. **Setting up the Source:** I spun up an Azure SQL Server and created a SQL Database to hold my starting dataset.
2. **Setting up the Destination:** I created an Azure Storage Account. 
   * *Fun fact: Azure Storage Accounts actually give you 4 different options (Containers, File Shares, Queues, and Tables).* 
   * For this project, I used **Containers** for my Data Lake sink, and **Tables** to try storing some structured data.
3. **Connecting the Dots (Data Integration):** 
   * I created my Azure Data Factory instance.
   * I set up "Linked Services" (basically secure connection strings) for both my SQL DB and my Storage Account.
   * Finally, I built the pipeline that automates moving the data from the source to the sink.

## 🚧 Roadblocks & How I Fixed Them

Like any cloud project, things didn't work perfectly on the first try. Here are a few hurdles I ran into and how I solved them:

* **Azure SQL Firewall Blocks:** 
  * *The Issue:* When I first tried to connect Data Factory to SQL, it failed. By default, Azure SQL blocks all outside traffic.
  * *The Fix:* I had to go into the SQL Server networking settings and check the box for **"Allow Azure services and resources to access this server"** so my pipeline could get through.
* **Figuring Out Permissions:**
  * *The Issue:* I needed Data Factory to write to my Storage Account, but I didn't want to use raw, hardcoded access keys (bad practice!).
  * *The Fix:* I used a System-Assigned Managed Identity (SAMI). I basically gave my Data Factory instance the **"Storage Blob Data Contributor"** role on the storage account so they could securely talk to each other.
* **Schema Headaches:**
  * *The Issue:* Moving strict, typed data from a SQL table into a flexible Data Lake file can cause formatting errors.
  * *The Fix:* I spent some time configuring explicit schema mappings inside my ADF *Copy Data Activity* to make sure the columns lined up perfectly when they landed in the destination.

## 💡 What I Learned

* **Storage is Flexible:** It was really cool to see how one single Azure Storage Account could handle completely different types of storage (Blob Containers vs. NoSQL Tables) under the same roof.
* **Pipelines are Powerful:** Getting hands-on with Azure Data Factory really showed me how easy it can be to automate complex data movement once the connections (Linked Services) are set up correctly.

---
