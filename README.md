# azure-alerts-to-teams
Arm template to deploy a logic app and alert that publishes Azure Service Health alerts to a Teams channel. 
*NOTE* At this time, the logic app teams connector to post a message (v3) is still in preview. 

Currently the Service Health alerts in Azure do not allow notifications to be pushed to Teams. To fill this gap, these Arm templates can be used to deploy the following:
A. An API connection to Teams
B. A logic App that parses the Service Health alerts json and publishes it a human readable message on a Teams channel
C. An action group to trigger the logic app
D. An alert to kick off the process when a Service Health alert is generated for the Azure environment. 

Installation Instructions:
1. Deploy 'api-connection.json' template. You will need to include The email address of the account used to authenticate to teams as a parameter. Optional - you can include the parameter 'api_connection_name' to change the API connection name. The default is 'teams'

Example deployment uisng CLI: az deployment group create --resource-group teams-alert-prod --parameters api_connection_username=user@contoso.com api_connection_name=teams --template-file api-connection.json

Upon successful deployment of the template. You will need to authorize the API connection. 
    A. Login to the Azure portal and locate the API connection. 
    B. Click on 'Edit API Connection' on the left panel. 
    C. Click the blue 'Authorize' button
    D. Complete the login process.
    E. Click 'Save' 

![API Connection Settings in Azure Portal](https://github.com/mack73/azure-alerts-to-teams/blob/master/readme-images/api-connection-screenshot1.png)