---
title: "Azure Data Management Pipeline Application"
layout: post
---

In this project, I designed and implemented a robust data management pipeline using Microsoft Azure's cloud services. Setup of the infrastructure was managed with Terraform. The complete project can be reviewed in my [GitHub repository](https://github.com/drjodyannjones/azure-data-management-pipeline){:target="\_blank"}.

### Project Overview

This **Azure Data Management Pipeline** project focuses on the integration of various Azure services to create a scalable and efficient pipeline for data ingestion, processing, and storage. The pipeline facilitates advanced data analysis and is tailored to support enterprises in making agile, informed business decisions. Terraform is utilized to programmatically create, modify, and remove resources on the Microsoft Azure cloud platform.

### Technologies Employed

- **Terraform** For programatically managing cloud infrastructure.
- **Azure Data Factory:** For orchestrating and automating data flows between various Azure services.
- **Azure Databricks:** Utilized for data processing and running big data analytics.
- **Azure SQL Database:** Used for storing processed data in a structured format.
- **Azure Storage:** Employed for durable, scalable storage of raw data.

### Execution Instructions

To engage with this project, follow these steps:

1. Clone the repository: `git clone https://github.com/drjodyannjones/azure-data-management-pipeline.git`
2. Set up the Azure services as detailed in the project's documentation.
3. Deploy the Azure Data Factory pipelines and monitor the workflow execution within Azure Portal.

### Sample Code Snippet: main.tf

Here is the main Terraform script that manages the infrastructure:

{% highlight python %}
terraform {
required_providers {
azurerm = {
source = "hashicorp/azurerm"
version = "~> 3.0.2"
}
}

required_version = ">= 1.1.0"
}

provider "azurerm" {
features {}
}

resource "azurerm_resource_group" "rg" {
name = var.resource_group_name
location = var.location
tags = var.tags
}

module "storage_account" {
source = "./modules/storage_account/storage_account"

resource_group_name = var.resource_group_name
storage_account_name = var.storage_account_name
location = var.location
source_folder_name = var.source_folder_name
destination_folder_name = var.destination_folder_name

depends_on = [
azurerm_resource_group.rg
]

}

module "data_factory" {
source = "./modules/data_factory/data_factory"

df_name = var.df_name
location = var.location
resource_group_name = var.resource_group_name
storage_account_name = var.storage_account_name

depends_on = [
module.storage_account
]
}
{% endhighlight %}
