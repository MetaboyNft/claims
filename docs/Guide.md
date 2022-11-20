# Introduction
This is the documentation guide on how to setup the MetaBoy claims system. You will need experience with the following for a successful deployment:

- Azure
- .NET 6
- Microsoft SQL Server / Database Administration experience
- Azure Service Bus
- Visual Studio 2022
- Discord Bot Development
- GitHub forking
- Loopring API

## 1. Sign up for Azure
If you don't have an Azure account, please sign up for one here: https://azure.microsoft.com/

## 2. Provision the Azure resources
While in Azure you will need to provision the following to your account:

1. Microsoft SQL Server Database with the following tables and columns:
- ![image](https://user-images.githubusercontent.com/5258063/202931174-3af41ea3-cdca-4143-b0a4-c735915e5fe1.png)
2. Azure Service Bus with a queue named main
3. Windows App Service Plan for .NET 6 with a deployment pointing to a fork of the MetaBoy API repo: https://github.com/MetaboyNft/MetaboyApi. Under the configuration section of the deployment, set up two AppSetting configuration variables, one named AzureSqlConnectionString and one named AzureServiceBusConnectionString that correspond with the SQL server and Azure Service Bus connection string you setup previously.

 ## 3. Setting up the MetaBoy API Message Processor:
1. Fork the MetaBoy API Message Processor: https://github.com/MetaboyNft/MetaboyApiMessageProcessor
2. Open up the solution file in Visual Studio 2022. 
3. While in Visual Studio 2022, create an appsettings.json file in the root directory with the build action "Copy if Newer". It needs these details below, you can export out the Loopring details from Loopring.io, all Loopring details must come from a MetaMask or GameStop wallet. Use the same settings as previous for the AzureServiceBusConnectionString and AzureSqlConnectionString:
```json
{
  "Settings": {
    "LoopringApiKey": ", //Your loopring api key.  DO NOT SHARE THIS AT ALL.
    "LoopringPrivateKey": "", //Your loopring private key.  DO NOT SHARE THIS AT ALL.
    "LoopringAddress": "", //Your loopring address. NOT YOUR ENS
    "LoopringAccountId": 191813, //Your loopring account id
    "ValidUntil": 1700000000, //How long this transfer should be valid for. Shouldn't have to change this value
    "MaxFeeTokenId": 1, //The token id for the fee. 0 for ETH, 1 for LRC
    "Exchange": "0x0BABA1Ad5bE3a5C0a66E7ac838a129Bf948f1eA4", //Loopring Exchange address,
    "MMorGMEPrivateKey": "", //Private key from metamask or gamestop wallet. DO NOT SHARE THIS AT ALL.
    "AzureServiceBusConnectionString": "", //Service Bus connection, DO NOT SHARE THIS AT ALL
    "AzureSqlConnectionString": "" //Sql Connection DO NOT SHARE THIS AT ALL
  }
}
```
4. While in Visual Studio 2022, publish the solution as a Continuous WebJob under the Windows App Service Plan for the API.  
