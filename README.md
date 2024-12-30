Use az account get-access-token:
This command retrieves the access token for the current Azure CLI session.

az account get-access-token
This will output the access token along with additional details such as the token's expiration time.

Locate the Token Cache File:
Azure CLI stores tokens in a cache file. You can view this file to see all stored tokens.

On Unix-like systems (Linux/macOS):

cat ~/.azure/accessTokens.json
On Windows:

type %USERPROFILE%\.azure\accessTokens.json
Example:
sh
# Retrieve the current access token
az account get-access-token

# Output:
# {
#   "accessToken": "<your-access-token>",
#   "expiresOn": "2024-12-30 20:00:00.000000",
#   "subscription": "<your-subscription-id>",
#   "tenant": "<your-tenant-id>",
#   "tokenType": "Bearer"
# }

# View the token cache file on Unix-like systems
cat ~/.azure/accessTokens.json

# View the token cache file on Windows
type %USERPROFILE%\.azure\accessTokens.json
2 vulnerabilities detected
Explanation:
az account get-access-token: This command retrieves the current access token and displays it in a JSON format.
Token Cache File: The cache file (accessTokens.json) contains all tokens stored by Azure CLI. Viewing this file provides detailed information about all active tokens.
These steps will help you see the current Azure tokens using the Azure CLI.

how to create a cli script to loop trhou an azure envirromnet and create a tocket for all subcriptions
To create a CLI script that loops through all Azure subscriptions and retrieves an access token for each subscription, you can follow these steps. This script will use Azure CLI commands to list all subscriptions, set the current subscription, and retrieve an access token for each subscription.

Here is a Bash script that performs these tasks:

sh
#!/bin/bash

# Enable verbose output
set -x

# Log file path
logFilePath="./TokenRetrievalLog.txt"

# Function to log messages
log_message() {
    local message=$1
    local timestamp=$(date +"%Y-%m-%d %H:%M:%S")
    local logMessage="$timestamp - $message"
    echo "$logMessage"
    echo "$logMessage" >> "$logFilePath"
}

log_message "Script execution started."

# Get the list of all subscriptions
subscriptions=$(az account list --query "[].id" -o tsv)

# Check if the subscriptions were retrieved successfully
if [ -z "$subscriptions" ]; then
    log_message "No subscriptions found or failed to retrieve subscriptions."
    exit 1
fi

# Loop through each subscription
for subscriptionId in $subscriptions; do
    log_message "Processing subscription ID: ${subscriptionId}"
    
    # Set the current subscription
    az account set --subscription "$subscriptionId"
    if [ $? -ne 0 ]; then
        log_message "Failed to set the subscription to ${subscriptionId}"
        continue
    fi
    log_message "Set current subscription to ${subscriptionId}"
    
    # Retrieve the access token for the current subscription
    token=$(az account get-access-token --query "accessToken" -o tsv)
    if [ $? -ne 0 ]; then
        log_message "Failed to retrieve the access token for ${subscriptionId}"
        continue
    fi
    log_message "Access token for ${subscriptionId}: $token"
done

log_message "Script execution completed."

echo "Access tokens have been retrieved and logged."
Explanation:
Enable Verbose Output: set -x enables verbose output for debugging purposes.
Log Function: log_message function logs messages with timestamps to both the console and a log file.
Retrieve Subscriptions: az account list --query "[].id" -o tsv retrieves all subscription IDs.
Loop Through Subscriptions: The script loops through each subscription, sets it as the current subscription using az account set, and retrieves the access token using az account get-access-token.
Error Handling: The script includes error handling to log failures in setting the subscription or retrieving the access token.
Usage:
Save the script as a .sh file (e.g., RetrieveAzureTokens.sh).
Make the script executable: chmod +x RetrieveAzureTokens.sh.
Run the script: ./RetrieveAzureTokens.sh.# get-az-token-cli
get existing az tokens 
