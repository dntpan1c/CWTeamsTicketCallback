Based off of work completed by Adam Hancock at: https://github.com/adamhancock/cwteams.git

NOTE: READ THE ENTIRE PROCESS BEFORE CONTINUING. I KNOW YOU JUST WANT TO START DEPLOYING. READ THE INSTRUCTIONS FIRST. THIS ENTIRE PROCESS SHOULD TAKE UNDER 30 MINUTES. IF IT DOESN'T IT MEANS YOU ARE DOING SOMETHING WRONG AND DIDN'T READ THE INSTRUCTIONS!

# Purpose

This function places certain information from live updates made to Service Tickets in your ConnectWise Manage PSA (On-Premise and Cloud) on a specified Service Board to be displayed in Real Time with certain relevant information in Microsoft Teams in a specific Channel. The time between an update to a Ticket in ConnectWise Manage to a message in a Teams Channel is less than a few seconds. 

There is a "duplicate check" implemented here that will only show the first update to each ticket that the connector sees. If there is a duplicate callback, it will be processed but not pushed to the channel. 

Going into this, you will want to know the EXACT name (spaces and all) of the ConnectWise Service Board you will want information to flow from (this only supports a single board -- complete this process for each board you want to do this with) and which Teams Channel you want this to flow into. 

# Prerequisites

**You will need the following permissions in each application:** 
Teams - Permission to create a Webhook for the desired Channel
Azure - Create a Function App in your Subscription, and configure its settings. 
ConnectWise Manage - Create an Integrator Login with a Callback URL

**You will need to know the following infromation**
Teams - the Channel Name you want to see alerts
ConnectWise Manage - The EXACT name of the Service Board you want to see the alerts from (spaces and all)

# Disclaimer

You can read our license, but specifically: THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Step One: Teams Tasks to Complete

**1. Webhook URL**
- In Microsoft Teams, find the desired Channel that you would like to see these alerts come up in, we will need to configure a Webhook and get a "Webhook URL"
- To get the Webhook URL, right click the Channel and select Connectors. Search for "Incoming Webhook" and choose "Add" (If "Configure" is displayed, this means that the team already has the Webhook Connector Enabled, and you should choose "Configure" instead)
- NOTE: If you already have a Webhook for this specific Channel, then simply copy down the Webhook URL and proceed to **"Step Two: Azure Tasks to Complete"**
- Configure a Webhook by choosing a Name for the Webhook, and an image that you would like to be shown when the incoming messages are displayed. I chose to use the ConnectWise Manage logo so our team would know where the Data was coming from. 
- After creating the webhook, you will be given a Webhook URL
- Write down this Webhook URL to be used later. 


# Step Two: Azure Tasks to Complete

**1. Create the Function**
- Create a new function app inside your Azure Portal (https://portal.azure.com).
- Choose the function app name, and write it down as it will be used later. 
- Select from the menus the following settings: code running Node.js 14LTS in your desired region. 
- You should enable "Application Insights" for Troubleshooting Purposes
- Tag as needed, and Create the function. 

**2. Deploy the Function Code**
- Post Deployment, go to the Function App. 
- Go to Deployment Center (Under "Deployment") > Settings Tab, Choose the Source as "External Git". 
- Enter https://github.com/dntpan1c/CWTeamsTicketCallback.git as the Source, and Branch as "main" (unless you choose to use another branch). 
- NOTE: This locks you into using the code in this branch of the repo. If updates are made to this branch, and you choose to update, that code will import into your function!

**3. Configure the Function Code**
- In your Azure Function App, go to "Configuration" (Under "Settings") > Application Settings Tab, 
- Add a new Setting with the name as your ConnectWise Service Board name, and the value as the Webhook URL of the Teams Channel you want to post to. You get this information from the prior steps and prerequisites. 


# Step Three: ConnectWise Manage Tasks to Complete

**1. Create Credentials and Callback**
- In ConnectWise Manage go to "Setup Tables" > "Integrator Login" Table
- Create a New Integrator Login. The Username identifies the Integrator. The password doesn't matter here, but make the password a strong password anyway. You don't need to store it or write it down. You don't even need the Username as the only important thing is the Callback URL. 
- Select All Records. 
- Check off "Service Ticket" under the "API Name Section"
- Set the Callback URL of Service Ticket to https://< your function app name >.azurewebsites.net/api/CWIncoming?serviceid=
- Set the Serivce Board to the Desired ConnectWise Manage Service Board
- Save and Close!

# Last Step: Testing and Trobleshooting

That's it! It should be working! Try to update a Status or Contact of a Ticket on the Desired Service Board and watch the message come in through teams!

It should "just work" if you read all of the instructions. If it doesn't check your permissions for each application, and check under your function app under "Application Insights" > "Live Events". Also, compare your environment to our testing environment (below). It is possible that you might need certain Firewall adjustments if you are using ConnectWise On-Premise as opposed to the Cloud Hosted version. 

# Our Testing Environment 
- Azure: Commercial Cloud in East US
- ConnectWise Manage: Cloud Hosted in NA Cloud
- Teams: Standard Commercial deployment in US

# Additional Resources

If you would like more information on Callbacks in ConnectWise Manage, go to the ConnectWise Manage Developer Documentation on CallBacks Here: https://developer.connectwise.com/Best_Practices/Manage_Callbacks
