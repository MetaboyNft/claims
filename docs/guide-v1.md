# Introduction
This is the documentation guide on how to setup the MetaBoy claims system. This pertains to V1 of the system. You will need experience with the following for a successful deployment:

- Azure
- .NET 6
- Microsoft SQL Server / Database Administration experience
- Azure Service Bus
- Visual Studio 2022
- Discord Bot Development
- GitHub forking
- Loopring API

# Architecture
![MetaBoy NFT Claims Diagram Final 2](https://user-images.githubusercontent.com/5258063/202937688-82a7cc3c-5916-429d-ba71-0ffaa6427627.png)

## Why a message queue is used
NFT Transfers on Loopring rely on the Storage Id being processed in the correct order, if operations are done in parallel the storage id wont be correct. A storage id is always +2 from the previous one. The Azure Service Bus allows us to process messages one at a time which allows for the correct storage id to be computed.

# Setup
## 1. Sign up for Azure
If you don't have an Azure account, please sign up for one here: [https://azure.microsoft.com](https://azure.microsoft.com)

## 2. Provision the Azure resources
While in Azure you will need to provision the following to your account:

### 2.1. Microsoft SQL Server Database
Provision a Microsoft SQL Server Database with the following tables.

![image](https://user-images.githubusercontent.com/5258063/202931174-3af41ea3-cdca-4143-b0a4-c735915e5fe1.png)

The SQL statements for table creation are as followings

```sql
CREATE TABLE AllowList (
    Address varchar(255),
    NftData varchar(255),
    Amount varchar(255)
);

CREATE TABLE Claimable (
    NftName varchar(255),
    NftData varchar(255),
);

CREATE TABLE Claimed (
    Address varchar(255),
    NftData varchar(255),
    ClaimedDate varchar(255),
    Amount varchar(255)
);
```
The allow list table contains data about loopring addresses and nfts that can be claimed, the claimable table contains data about nfts that can be claimed, the claimed table contains data about successful claims. The nftData can be obtained from the Loopring API and is not the same as the nftId. All addresses need to be in hex format(0xblahblah) and not be an ENS.

#### Retrieving nftData
To retrieve nftData you must query the Loopring api. Once you have an API key for Loopring you can send a GET request to the following endpoint(replacing account id with the distributon wallet account id you will be using), [https://api3.loopring.io/api/v3/user/nft/balances?accountId=191813&limit=50&metadata=true&offset=0](https://api3.loopring.io/api/v3/user/nft/balances?accountId=191813&limit=50&metadata=true&offset=0)

### 2.2 Azure Service Bus
Provision an Azure Service Bus with a queue named main .

### 2.3 Linux App Service Plan
Provision a Basic Linux App Service Plan for .NET 6 with source control for the deployment pointing to a fork of the MetaBoy API repo: [https://github.com/MetaboyNft/MetaboyApi](https://github.com/MetaboyNft/MetaboyApi). You may need to delete the "github/workflows" folder first after forking the repo. Then once deployed, under the configuration section of the deployment, set up two AppSetting configuration variables, one named AzureSqlConnectionString and one named AzureServiceBusConnectionString that correspond with the SQL server and Azure Service Bus connection string you setup previously.

## 3. Setting up the MetaBoy API Message Processor:
1. Fork the MetaBoy API Message Processor: [https://github.com/MetaboyNft/MetaboyApiMessageProcessor](https://github.com/MetaboyNft/MetaboyApiMessageProcessor)
2. Open up the solution file in Visual Studio 2022. 
3. While in Visual Studio 2022, create an appsettings.json file in the root directory with the build action "Copy to Output Directory" option set to "Copy always". It needs these details below, you can export out the Loopring details from Loopring.io, all Loopring details must come from a MetaMask or GameStop wallet. Use the same settings as previous for the AzureServiceBusConnectionString and AzureSqlConnectionString:
```json
{
  "Settings": {
    "LoopringApiKey": "", //Your loopring api key.  DO NOT SHARE THIS AT ALL.
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
4. While in Visual Studio 2022, publish the solution as a Continuous WebJob under a S1 Windows App Service Plan.

## 4. Setting up the Discord Bot
1. You need to create a Discord Bot through the Discord Developer Portal with the following permissions and invite it to your Discord.(Be sure to save all neccessary tokens for future steps)
![image](https://user-images.githubusercontent.com/5258063/202933785-d37aee16-6e17-4031-9aeb-aec8dfe9a2fd.png)
2. Fork the following repo: [https://github.com/MetaboyNft/Gaia](https://github.com/MetaboyNft/Gaia)
3. Open the solution file in Visual Studio 2022, create an appsettings.json file in the solution directory like below with the build action "Copy to Output Directory" option set to "Copy always". Use the same sql server connection strings as previous steps, use the discord token that was setup with the intial discord bot creation and use the discord server id for your own discord server. For ApiEndpointUrl use the url for the API site you setup previously. Finally, create two new channels in your discord. One for claims for regular users, use this channel id for ClaimsChannelId. Then one for the administration of claims for admins, use this channel id for ClaimsAdminChannelId.
```json
{
  "Settings": {
    "ApiEndpointUrl": "https://yourSiteName.azurewebsites.net",
    "SqlServerConnectionString": "",
    "ClaimsChannelId": 1044389196980310028,
    "ClaimsAdminChannelId": 1044389220715868160,
    "DiscordServerId": 933963129652674671, //Discord Server Id 
    "DiscordToken": "" //Discord Token
  }
}
```
5. Once the bot appsettings.json has been modified for your needs, publish the solution using Visual Studio as a Continuous WebJob to the same Windows App Service Plan that was setup previously.

# Discord bot slash commands
## /claim
This command checks for claims with the address provided. If valid it will contact the API to process the claim into the message queue.

## /claimable_add
This command adds an NFT that can be claimed by addresses in the allow list. You must use the nftData attribute from the Loopring API. The nftData attribute is not the same as the nftId.

## /claimable_show
This command shows all NFTs that can be claimed.

## /claimable_remove
This command removes NFTs that can be claimed.

## /allowlist_add
This command adds addresses to the allow list that can claim the claimable nfts

## /allowlist_remove
This command removes addresses from the allow list.

## /allowlist_check
This command checks if addresses are in the allow list.

## /claimed_check
This command checks if an address has recieved a claim

## /claimed_remove
This command removes an address that has recieved a claim

# Bulk operations for claims
If you need to do operations in bulk, you can use raw SQL commands to add bulk addresses to the allow list table in the database.

1. Adding a claimable nft(should only need to add once):
```sql
insert into claimable (NftName,NftData) Values ('MetaBoy #9987','0x09c3a263e3cb7e893af1fe73b0cc373850f942842c92560bef6016c2711d5ca0');
```

2. Adding bulk addresses for allow list: 

```sql
insert into allowlist (Address,NftData,Amount) Values ('0x9Da766D34E5df44A1113ca3A38C4DBf400a37ceA','0x09c3a263e3cb7e893af1fe73b0cc373850f942842c92560bef6016c2711d5ca0','1');
insert into allowlist (Address,NftData,Amount) Values ('0x36Cd6b3b9329c04df55d55D41C257a5fdD387ACd','0x09c3a263e3cb7e893af1fe73b0cc373850f942842c92560bef6016c2711d5ca0','1');
```

