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
