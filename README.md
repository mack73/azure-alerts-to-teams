# azure-alerts-to-teams
Arm template to deploy a logic app and alert that publishes Azure Service Health alerts to a Teams channel. 
*NOTE* At this time, the logic app teams connector to post a message (v3) is still in preview. 

Currently the Service Health alerts in Azure do not allow notifications to be pushed to Teams. To fill this gap, these Arm templates can be used to deploy the following:
A. An API connection to Teams
B. A logic App that parses the Service Health alerts json and publishes it as a human readable message on a Teams channel
C. An action group to trigger the logic app
D. An alert to kick off the process when a Service Health alert is generated for the Azure environment. The alert scope is for ** ALL ** services in ** ALL ** regions. Adjust as necessary after deployment. See the section 'alert scope' for instruction on how to accomplish this at the bottom

This set of instructions deploys a complete set of resources for each alert type. This might not be required for your deployment if you want the alerts to be published to the same Teams Channel. See the section 'Combined Alerting' for instructions on how to accomplish this at the bottom. 

## Installation Instructions:
### 1. Deploy 'api-connection.json' template. 
Required parameter: 'api_connection_username' You will need to include the email address of the account used to authenticate to teams. Optional parameter: 'api_connection_name' you can specify the API connection name. The default is value is 'teams'

#### Example deployment using CLI: 
    az deployment group create --resource-group teams-alert-prod --parameters api_connection_username=user@contoso.com api_connection_name=teams --template-file api-connection.json


### 2. Authorize the API connection 
1. Login to the Azure portal and locate the API connection 
2. Click on 'Edit API Connection' on the left panel
3. Click the blue 'Authorize' button
4. Complete the login process
5. Click 'Save' 

![API Connection Settings in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/api-connection-screenshot1.png)


### 3. Deploy 'logic-ag-alert.json' template
This will deploy the following:
* A logic App
* An action group
* An alert

Since there are currently 4 types of Service Health alerts, you will repeat this step 4 times, each time indicating a different type of alert. Required parameters: 'region_name' The region name must be specified due to a bug parsing resourceGroup().Location, 'alertEventType' this is used to specify which of the 4 alert types you want to deploy (Incident=Service Issues, Maintenance=Planned Maintenance, Informational=Health Advisory, Security=Security Advisory), 'default-name' the name you want to label the various resources in Azure. I suggest you use a name similar to the alert type in lowercase separated by hyphens (e.g service-issues). Optional - Review the parameters section in the json template as you can override the default naming convention for each of the resouces by including specific parameters during deployment. 

#### Example deployment using CLI: 
1. Service Issues: 
    > az deployment group create --resource-group teams-alert-prod --parameters region_name=eastus alertEventType=Incident default-name=service-issues --template-file logic-ag-alert.json
2. Planned Maintenance: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=eastus alertEventType=Maintenance default-name=planned-maintenance --template-file logic-ag-alert.json
3. Health Advisory: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=eastus alertEventType=Informational default-name=health-advisory --template-file logic-ag-alert.json
4. Security Advisory: 
    az deployment group create --resource-group teams-alert-prod --parameters region_name=eastus alertEventType=Security default-name=security-advisory --template-file logic-ag-alert.json


### 4. Edit the Logic App using the designer to specify the Teams Channel 
1. Login to the Azure portal and locate the logic app
2. Click on 'Logic app designer' on the left panel
3. Click on 'Post a message' to expand the step 
4. Select the desired 'Team' and 'Channel' from the dropdowns
5. Click 'Save' at the top. Repeat these steps for all 4 logic apps

![Logic App Designer in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/logicapp-designer-screenshot1.png)


## Optional Information

#### Alert Scope
The alerts deployed are set to be triggered for all services in all regions. Instead, the alerts should be tailored to your specific services and regions. 
1. Login to the Azure portal and navigate to Alerts
2. Click 'Manage alert rules' at the bottom
3. Select an alert
4. Use the 'Services' and 'Regions' dropdowns to reduce the scope to match your environment
5. Click 'save' 

![API Connection Settings in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/alert-scope-screenshot1.png)

#### Combined Alerting
This set of instructions deploys a complete set of resources for each alert type. This might not be required for your deployment. The Logic App is mapped to a single Teams channel. If the channel is shared between all alert types you do not need to deploy multiple instances of logic apps, action groups or alerts. The installation process can be changed to have all 4 alerts trigger the same logic app as the Service Health alerts are deployed using the Common Alert Schema meaning the JSON parsing is the same for all 4 alert types. 
    
To deploy all 4 alerts to the same Teams channel:
1. Complete the deployment process for 1 alert type. The default name can be set to something universal to prevent confusion 
2. Login to the Azure portal and navigate to Alerts
3. Click 'Manage alert rules' at the bottom
4. Select the alert
5. Use the 'Event type' drop down to select the alert types that should trigger the message to be posted on Teams
6. Click 'Save'

![Combined Alerts in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/alert-screenshot1.png)