**Step-by-Step Instructions to Deploy the Java Azure Function on Azure**

---

### **Prerequisites**

1. **Azure Account**
   - Ensure you have an active [Azure subscription](https://azure.microsoft.com/en-us/free/).

2. **Development Environment**
   - **Java Development Kit (JDK) 11** or later.
   - **Maven** installed ([Download Maven](https://maven.apache.org/download.cgi)).
   - **Azure CLI** installed ([Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)).
   - **Docker** installed (optional, for local testing) ([Install Docker](https://www.docker.com/get-started)).

3. **Azure Tools**
   - **Azure Functions Core Tools** ([Install Azure Functions Core Tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local#v2)).

---

### **1. Clone or Create the Function Project**

If you haven't already, set up your Java Azure Function project.

```bash
# Create a new directory for your project
mkdir PollAndPublishFunction
cd PollAndPublishFunction

# Initialize a new Maven project (if not already done)
mvn archetype:generate -DgroupId=com.example -DartifactId=PollAndPublishFunction -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

cd PollAndPublishFunction
```

---

### **2. Update `pom.xml`**

Ensure your `pom.xml` includes the necessary dependencies and Azure Functions Maven plugin.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>PollAndPublishFunction</artifactId>
    <version>1.0.0</version>
    <properties>
        <java.version>11</java.version>
        <azure.functions.java.version>1.4.2</azure.functions.java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>com.microsoft.azure.functions</groupId>
            <artifactId>azure-functions-java-library</artifactId>
            <version>${azure.functions.java.version}</version>
        </dependency>
        <!-- Add any additional dependencies if needed -->
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>com.microsoft.azure</groupId>
                <artifactId>azure-functions-maven-plugin</artifactId>
                <version>1.13.0</version>
                <configuration>
                    <resourceGroup>your-resource-group</resourceGroup>
                    <appName>your-function-app-name</appName>
                    <region>your-region</region>
                    <pricingTier>Consumption</pricingTier>
                    <runtime>
                        <os>linux</os>
                        <javaVersion>11</javaVersion>
                        <webContainer>JAVA11</webContainer>
                    </runtime>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**Replace:**
- `your-resource-group` with your desired Azure Resource Group name.
- `your-function-app-name` with a unique name for your Function App.
- `your-region` with your preferred Azure region (e.g., `eastus`).

---

### **3. Configure `local.settings.json`**

Set up your local settings for development and testing.

Create a `local.settings.json` file in the `src/main/resources` directory:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "EventHubConnection": "Endpoint=sb://<your-event-hub-namespace>.servicebus.windows.net/;SharedAccessKeyName=<key-name>;SharedAccessKey=<access-key>"
  }
}
```

**Replace:**
- `<your-event-hub-namespace>` with your Event Hub namespace.
- `<key-name>` and `<access-key>` with your Event Hub's shared access policy name and key.

---

### **4. Create Azure Resources**

Use Azure CLI to create necessary resources.

```bash
# Log in to Azure
az login

# Create a Resource Group
az group create --name your-resource-group --location your-region
 
az group create --name RG-Data-Eng-CUS-001 --location "Central US"

# Create an Event Hub Namespace
az eventhubs namespace create --resource-group your-resource-group --name your-event-hub-namespace --location your-region --sku Standard
 
az eventhubs namespace create --resource-group RG-Data-Eng-CUS-001 --name poll-publish-ev-hub-ns --location centralus --sku standard

# Create an Event Hub
az eventhubs eventhub create --resource-group your-resource-group --namespace-name your-event-hub-namespace --name your-event-hub-name
 
az eventhubs eventhub create --resource-group RG-Data-Eng-CUS-001 --namespace-name poll-publish-ev-hub-ns --name poll-publish-ev-hub-001

# Create a Shared Access Policy for Event Hub
az eventhubs namespace authorization-rule create --resource-group your-resource-group --namespace-name your-event-hub-namespace --name your-policy-name --rights Send Listen
 
az eventhubs namespace authorization-rule create --resource-group RG-Data-Eng-CUS-001 --namespace-name poll-publish-ev-hub-ns --name poll-publish-policy --rights Send Listen
 
# Get the connection string
EVENTHUB_CONNECTION=$(az eventhubs namespace authorization-rule keys list --resource-group your-resource-group --namespace-name your-event-hub-namespace --name your-policy-name --query primaryConnectionString --output tsv)
 
EVENTHUB_CONNECTION=$(az eventhubs namespace authorization-rule keys list --resource-group RG-Data-Eng-CUS-001 --namespace-name poll-publish-ev-hub-ns --name poll-publish-policy --query primaryConnectionString --output tsv)
```

**Replace:**
- `your-resource-group`, `your-region`, `your-event-hub-namespace`, `your-event-hub-name`, and `your-policy-name` with your specific names.

---

### **5. Update `local.settings.json` with Connection String**

Update the `EventHubConnection` in `local.settings.json` with the obtained connection string.

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "java",
    "EventHubConnection": "Endpoint=sb://your-event-hub-namespace.servicebus.windows.net/;SharedAccessKeyName=your-policy-name;SharedAccessKey=your-access-key"
  }
}
```

---

### **6. Build the Project**

Compile your project using Maven.

```bash
mvn clean package
```

---

### **7. Deploy the Function to Azure**

Use the Azure Functions Maven plugin to deploy.

```bash
mvn azure-functions:deploy
```

**Notes:**
- Ensure that the `azure-functions-maven-plugin` is correctly configured in your `pom.xml`.
- The first deployment may take several minutes.

---

### **8. Verify Deployment**

1. **Check Function App in Azure Portal**
   - Navigate to the [Azure Portal](https://portal.azure.com/).
   - Go to your **Function App** (`your-function-app-name`).
   - Verify that the `PollAndPublish` function is listed.

2. **Monitor Logs**
   - In the Function App, select **Monitor** to view invocation logs and ensure the function is running as expected.

3. **Test the Function**
   - The function is set to trigger every 10 seconds (note Azure Functions may have a minimum trigger interval of 1 minute).
   - Verify that events are being published to your **Event Hub** by checking the **Event Hub metrics** or using tools like **Azure Portal's Event Hub Capture**.

---

### **9. Configure Production Settings**

1. **Update `pom.xml` for Production**
   - Ensure all production configurations (resource group, app name, region) are correctly set.

2. **Set Environment Variables in Azure**
   - In the Azure Portal, navigate to your Function App.
   - Go to **Configuration** > **Application Settings**.
   - Add or update the `EventHubConnection` with the production connection string.

3. **Enable Monitoring and Alerts**
   - Set up **Application Insights** for detailed monitoring.
   - Configure alerts for function failures or performance issues.

---

### **10. Secure Your Function**

1. **Use Managed Identity (Optional)**
   - Enable **Managed Identity** for your Function App.
   - Assign appropriate roles to access Event Hubs securely without connection strings.

2. **Restrict Access**
   - Configure network restrictions to allow only trusted sources to access your Function App and Event Hubs.

3. **Encrypt Sensitive Data**
   - Use **Azure Key Vault** to store sensitive information and access them securely from your Function App.

---

**Deployment Complete!**

Your Java Azure Function is now deployed on Azure, polling the specified REST API every 10 seconds and publishing transactions to your Event Hub.

---

**Next Steps:**

- **Integrate with Other Services:** Connect your Event Hub to downstream services (e.g., Stream Analytics, SignalR) for real-time dashboard updates.
- **Implement Error Handling:** Enhance the function with robust error handling and retries.
- **Scale as Needed:** Monitor performance and scale your Function App and Event Hub based on load.

---
  
**Reference:**
  
https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-java?tabs=macos%2Cbash%2Cazure-cli%2Cbrowser