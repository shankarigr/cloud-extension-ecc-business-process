# MOCKrele

This is a Mock Server for OData API's (e.g. from S/4HANA, ECC etc.).

This is a project fork from [this repo](https://github.tools.sap/refapps/s4hana-cloud-mock). This project implies to work as SAP S/4HANA Cloud Mock backend server for the Reference Applications use cases. The project is built on Cloud Application Programming ([CAP](https://cap.cloud.sap/docs/)) model with mocking capabilities.

## Why to use

1. Speed up mission implementation. Customer can get an overview of the scenario even without ERP/Cloud Connector configuration.
2. Troubleshooting. If something goes wrong it's nice to have an opportunity to limit the scope for cloud services only.
3. Fully functioning deliverables. If the automated deployment is used (e.g. BTP Setup Automator Tool) it has to be expected that the result is ready to use (even if further configuration is needed)
4. Automated integration tests?

## Features
General features:
- OData Endpoints for both versions - [V2](https://cap.cloud.sap/docs/advanced/odata#v2-support) and V4
- CSV files for [mock data](https://cap.cloud.sap/docs/guides/using-services#local-mocking)
- [Event emitting](https://cap.cloud.sap/docs/guides/messaging/#using-sap-event-mesh) on [data change](https://cap.cloud.sap/docs/guides/providing-services#registering-event-handlers)
- Enhancement possibilities (adding new services)
- In-memory [SQLite DB](https://cap.cloud.sap/docs/guides/databases#deploy-to-sqlite) is used (no DB instance is needed)

New features added:
- [SwaggerUI](https://cap.cloud.sap/docs/advanced/openapi#swagger-ui)
- Emitting events compliant with the following Discover Center missions:
    - [S/4HANA Extension with Addon](https://discovery-center.cloud.sap/protected/index.html#/missiondetail/3730/3769/)
    - [ECC Extension](https://discovery-center.cloud.sap/protected/index.html#/missiondetail/3338/3384/)
    - [S/4HANA Extension](https://github.com/SAP-samples/cloud-extension-ecc-business-process/blob/main/srv/service.js)???
- Hybrid Event Mesh [test](https://cap.cloud.sap/docs/advanced/hybrid-testing)
- [MTA Deployment](https://cap.cloud.sap/docs/guides/deployment/)
    - Event Mesh instance binding
    - Change destination content - **NOT POSSIBLE FOR NO AUTHENTICATION :(**

## Quick deploy in Cloud Foundry Environment

1. Clone this repository to your SAP Business Application Studio workspace
2. Open cloned project folder
3. Right click on *mta.yaml* file and select **Build MTA Project**. Note: if you have Event Mesh instance created during one of the Extension Missions, please use *mta_em.yaml* instead to build the project (you should rename it to *mta.yaml* beforehand).
6. After build is done right click on mta_archives/MOCKrele_1.0.0.mtar and select **Deploy MTA Archive**

## How to use

1. After the successful deployment you will get the following message:

    <code>Application "MOCKrele-srv" started and available at "..."</code>

2. Follow the given URL to get the CAP App Index Page. You can find *'/op-api-business-partner-srv'* link there. This URL can be used as OData V4 Endpoint. Use this URL in the corresponding destination for your mission.

3. If you need V2 Endpoint please add '/v2' to the endpoint URL to get something like that:

    <code>https://someDomain/v2/op-api-business-partner-srv</code>

3. On the index page there is also a link to SwaggerUI: *Open API Preview*. Open it and go to the section *POST /A_BusinessPartner*. Click the button *Try it out*. Use the following JSON as a payload (replace the proposed one):

```
    {  
	"BusinessPartner": "555",  
	"BusinessPartnerName": "Max Mustermann",  
	"FirstName": "Max",  
	"LastName": "Mustermann",  
	"BusinessPartnerIsBlocked": true,  
	"to_BusinessPartnerAddress": [{  
        "BusinessPartner": "555",  
		"AddressID": "1",  
		"StreetName": "Platz der Republik",  
		"HouseNumber": "1",  
		"PostalCode": "10557",  
		"CityName": "Berlin",  
		"Country": "DE"  
		}]  
    }
```    

4. Click *Execute*. You should get a response with the code 201. This means that the entry was created in the database and the corresponding event was triggered (if the Event Mesh instance is binded).

## Hybrid test

With the hybrid testing capabilities, you stay in your local development environment and avoid long turn-around times of cloud deployment and you can use Event Mesh instance from the cloud.

It's assumed that the Event Mesh service and its key is already created beforehand.

Before the test you should be logged in to the Cloud Foundry Environment. To do it use *Ctrl+Shift+P* and select *CF: Login To Cloud Foundry*. Follow the next instructions on the screen.

To bind the Event Mesh service use the following command in terminal:

<code>cds bind messaging -2 BusinessPartnerValidation-ems:emkey</code>

After that the file *.cdsrc-private.json* will be created/updated with the corresponding information.

To start the Mock Server in hybrid mode use the following command:

<code>cds watch --profile hybrid</code>

## Project Structure

It contains these folders and files, following our recommended project layout:

File or Folder | Purpose
---------|----------
`app/` | content for UI frontends goes here
`img/` | images for README.md
`srv/` | service models and CSV data
`package.json` | project metadata and configuration
`readme.md` | this getting started guide
`mta.yaml` | ...
`server.js` | ...


- Open a new terminal and run `cds watch` 
- (in VS Code simply choose _**Terminal** > Run Task > cds watch_)

## Adding own services

To add another service from SAP API Business Hub follow the instructions [here](https://cap.cloud.sap/docs/guides/using-services#from-api-hub).

Note: old models have incorrect connection between object nodes. In the generated CDS it could look that:

```
entity API_BUSINESS_PARTNER.A_BusinessPartner{    

   /**
   * Please add  an ON condition  
   */    
   Association to many API_BUSINESS_PARTNER.A_BusinessPartnerAddress{ }    

}
```

This shoud be changed to:

```
entity API_BUSINESS_PARTNER.A_BusinessPartner{    

   Composition of  many API_BUSINESS_PARTNER.A_BusinessPartnerAddress
    on to_BusinessPartnerAddress.BusinessPartner = BusinessPartner;

}
```

The difference between compositions and association read [here](https://cap.cloud.sap/docs/cds/cdl#associations).