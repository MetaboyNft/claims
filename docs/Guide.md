# Introduction
This is the documentation guide on how to setup the MetaBoy claims system. You will need experience with the following for a successful setup:

- Azure
- .NET 6
- Microsoft SQL Server / DBA experience
- Azure Service Bus
- Visual Studio 2022
- Discord Bot Development
- GitHub forking

## 1. Sign up for Azure
If you don't have an Azure account, please sign up for one here: https://azure.microsoft.com/

## 2. Provision the Azure resources
While in Azure you will need to provision the following to your account:

- Windows App Service Plan for .NET 6 with a deployment pointing to a fork of the MetaBoy API repo: https://github.com/MetaboyNft/MetaboyApi
- Microsoft SQL Server with the following tables and columns:
- ![image](https://user-images.githubusercontent.com/5258063/202931174-3af41ea3-cdca-4143-b0a4-c735915e5fe1.png)
- Azure Service Bus with a queue called "main"

 ## 3. Setting up the MetaBoy API:
 You will need to 
