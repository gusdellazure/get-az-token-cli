
```sh
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

# Function to display tokens in a table format
display_tokens_table() {
    printf "%-36s | %-20s | %-64s\n" "Subscription ID" "Token Expires On" "Access Token"
    printf "%-36s-+-%-20s-+-%-64s\n" "------------------------------------" "--------------------" "----------------------------------------------------------------"
    for row in "${tokensTable[@]}"; do
        IFS='|' read -r -a columns <<< "$row"
        printf "%-36s | %-20s | %-64s\n" "${columns[0]}" "${columns[1]}" "${columns[2]}"
    done
}

log_message "Script execution started."

# Get the list of all subscriptions
subscriptions=$(az account list --query "[].id" -o tsv)

# Check if the subscriptions were retrieved successfully
if [ -z "$subscriptions" ]; then
    log_message "No subscriptions found or failed to retrieve subscriptions."
    exit 1
fi

# Initialize an array to store token information
tokensTable=()

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
    expiresOn=$(az account get-access-token --query "expiresOn" -o tsv)
    if [ $? -ne 0 ]; then
        log_message "Failed to retrieve the access token for ${subscriptionId}"
        continue
    fi
    
    # Store token information in the array
    tokensTable+=("${subscriptionId}|${expiresOn}|${token}")
    log_message "Access token for ${subscriptionId}: $token"
done

log_message "Script execution completed."

# Display the tokens in a table format
display_tokens_table

echo "Access tokens have been retrieved and logged."
```

### Explanation:
1. **Log Function**: `log_message` function logs messages with timestamps to both the console and a log file.
2. **Retrieve Subscriptions**: `az account list --query "[].id" -o tsv` retrieves all subscription IDs.
3. **Loop Through Subscriptions**: The script loops through each subscription, sets it as the current subscription using `az account set`, and retrieves the access token using `az account get-access-token`.
4. **Store Token Information**: The script stores token information (subscription ID, expiration time, access token) in an array `tokensTable`.
5. **Display Tokens Table**: The `display_tokens_table` function formats and displays the token information in a table format.

### Usage:
- Save the script as a `.sh` file (e.g., `RetrieveAzureTokens.sh`).
- Make the script executable: `chmod +x RetrieveAzureTokens.sh`.
- Run the script: `./RetrieveAzureTokens.sh`.

This script will loop through all Azure subscriptions, retrieve access tokens, and display the results in a formatted table list. If you need further assistance or modifications, feel free to ask!
