# azure-alerts-to-teams
Arm template to publish a logic app that publishes Azure Service Health alerts to a Teams channel.

Currently the Service Health alerts in Azure do not allow notiifcations to be pushed to Teams. To fill this gap, these Arm templates can be used to deploy the following:
1. An API connection to Teams
2. A logic App that parses the Service Health alerts and publishes the message on a Teams channel
3. An action group to trigger the logic app
4. An alert to kick off the process when a Service Health alert is generated for the Azure environment. 

