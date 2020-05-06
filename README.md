# Azure Service Health Alerts to Teams
Currently the Service Health alerts in Azure do not support pushing notifications to Teams. To fill this gap, these templates can be used to deploy a logic app and accompanying alerts that publish Azure Service Health alerts to a Teams channel. *NOTE* At this time, the logic app teams connector to post a message (v3) is still in preview. 

The Arm templates deploys the following:
* An API connection to Teams
* A logic App that parses the Service Health alerts json and publishes it as a human readable message on a Teams channel
* An action group to trigger the logic app
* An alert to kick off the process when a Service Health alert is generated for the Azure environment. The alert scope is for *ALL* services in *ALL* regions. Adjust as necessary after deployment. See the section 'alert scope' for instruction on how to accomplish this at the bottom

This set of instructions deploys a complete set of resources for each alert type. This might not be required for your deployment if all of the alerts can be published to the same Teams Channel. See the section 'Combined Alerting' for instructions on how to accomplish this at the bottom. 

## Installation Instructions:
### 1. Deploy 'api-connection.json' template. 
Create the API Connection used by the Logic App to connect to Teams. Only 1 API connection is needed as it can be shared by all logic apps. Required parameter: 'api_connection_username' - the email address of the account used to authenticate to Teams. Optional parameter: 'api_connection_name' - the name given to the API connection. The default is value is 'teams'.

#### Example deployment using CLI: 
    az deployment group create --resource-group teams-alert-prod --parameters api_connection_username=user@contoso.com --template-file api-connection.json


### 2. Authorize the API connection 
The API connection will need to be authorized after deployment 
1. Login to the Azure portal and locate the API connection 
2. Click on 'Edit API Connection' on the left panel
3. Click the blue 'Authorize' button
4. Complete the process
5. Click 'Save' 

![API Connection Settings in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/api-connection-screenshot1.png)


### 3. Deploy 'logic-ag-alert.json' template
There are currently 4 types of Service Health alerts (Service Issues, Planned Maintenance, Health Advisories, Security Advisories). Each deployment will create 1 set of resources for 1 alert type. Repeat this step 4 times, 1 for each alert type.  This might not be required for your deployment if all of the alerts can be published to the same Teams Channel. See the section 'Combined Alerting' at the bottom for more information. 

This templae will deploy the following:
* A logic App
* An action group
* An alert

Required parameters: 'region_name' - the Azure region must be explicitly defined as it cannot be inherited from the resource group due to a current logic app deployment bug when parsing resourceGroup().Location. 'alertEventType' - select the alert type to deploy (Incident=Service Issues, Maintenance=Planned Maintenance, Informational=Health Advisories, Security=Security Advisories). 'default-name'- the name used to label the various resources in Azure. I suggest using a name similar to the alert type in lowercase separated by hyphens (e.g service-issues or planned-maintenance). Optional parameters: Review the parameters section in the json template if you would like to override the default naming convention for each of the resouces during deployment. 

#### Example deployment using CLI: 
##### Service Issues: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=southcentralus alertEventType=Incident default-name=service-issues --template-file logic-ag-alert.json

##### Planned Maintenance: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=southcentralus alertEventType=Maintenance default-name=planned-maintenance --template-file logic-ag-alert.json

##### Health Advisory: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=southcentralus alertEventType=Informational default-name=health-advisory --template-file logic-ag-alert.json

##### Security Advisory: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=southcentralus alertEventType=Security default-name=security-advisory --template-file logic-ag-alert.json


### 4. Edit the Logic App using the designer to specify the Teams Channel 
The logic app 'post a message to Teams' step needs to be configured with the Teams channel where the message is posted.
1. Login to the Azure portal and locate the logic app
2. Click 'Logic app designer' on the left panel
3. Click on 'Post a message' to expand the step 
4. Select the desired 'Team' and 'Channel' from the dropdowns
5. Click 'Save' at the top. Repeat these steps for all 4 logic apps

![Logic App Designer in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/logicapp-designer-screenshot1.png)


## Optional Information

### Alert Scope
The alert is set to be triggered for all services in all regions. Instead, the alert should be tailored to your specific services and regions. 
1. Login to the Azure portal and navigate to Alerts
2. Click 'Manage alert rules' at the bottom
3. Select an alert
4. Use the 'Services' and 'Regions' dropdowns to reduce the scope to match your environment
5. Click 'save' 

![API Connection Settings in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/alert-scope-screenshot1.png)

### Combined Alerting
The 'logic-ag-alert.json' template deploys a complete set of resources for each alert type (logic app, action group, alert). The Azure Service Health alerts are deployed using the Common Alert Schema which will deliever all 4 alerts using the same JSON schema. If the Teams channel will be shared between all alert types only 1 set of resrouces is needed.
    
To deploy all 4 alerts to the same Teams channel:
1. Deploy 'logic-ag-alert.json' for 1 alert type. The type selected does not matter. The default name can be set to something universal to prevent confusion 
2. Login to the Azure portal and navigate to Alerts
3. Click 'Manage alert rules' at the bottom
4. Select the alert
5. Use the 'Event type' drop down to select the alert types that should trigger the message to be posted on Teams
6. Click 'Save'

Now when any of 4 alerts are generated it will trigger the logic app to post the message to the Teams channel. 

![Combined Alerts in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/alert-screenshot1.png)
