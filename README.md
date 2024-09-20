

```js
const { app } = require('@azure/functions');
const df = require('durable-functions');

// Define the Orchestrator function
df.app.orchestration('reportOrchestrator', function* (context) {
    try {
        // Get the current orchestration's instance ID
        const instanceId = context.df.instanceId;
        context.log(`Orchestration instance ID: ${instanceId}`);

        // Pass the instanceId to the activity function
        const token = yield context.df.callActivity('TokenManager', { instanceID: instanceId });

        context.log(`Token retrieved: ${token}`);

        // Further processing can be done here with the token
        // Example: calling another activity
        // const result = yield context.df.callActivity('PipelineQuery', { token });

        return token;

    } catch (error) {
        context.log(`Error in orchestration: ${error.message}`);
        throw error;  // Rethrow the error to be logged by Durable Functions
    }
});

// Define the Activity function
df.app.activity('TokenManager', {
    handler: async (input, context) => {
        const log = context.log;
        const instanceID = input.instanceID;  // Extract the instanceID from the input

        if (!instanceID) {
            throw new Error("InstanceID is missing or undefined.");
        }

        log(`Orchestrator Instance ID is ${instanceID}`);

        const logicAppURL = "<Your Logic App URL>"; // Replace with your Logic App URL
        let functionID = "POST Reports Orchestrator";

        try {
            // Make the POST request to retrieve the token
            const tokenResponse = await axios.post(logicAppURL, {
                requestType: "GET_TOKEN",
                functionID: functionID,
            });

            // Check if token was successfully retrieved
            if (tokenResponse.data && tokenResponse.data.token) {
                log("Token successfully retrieved");
                return tokenResponse.data.token;
            } else {
                log("Token could not be retrieved. Returning an empty token.");
                return "";  // Return empty token if retrieval fails
            }

        } catch (error) {
            log(`Error retrieving token: ${error.message}`);
            throw error;  // Throw the error so it's logged properly by Durable Functions
        }
    }
});

// Define the HTTP trigger to start the orchestration
app.http('startReportOrchestrator', {
    route: 'orchestrators/{orchestratorName}',
    extraInputs: [df.input.durableClient()],
    handler: async (request, context) => {
        const client = df.getClient(context);

        // Parse the request body for any additional parameters (if needed)
        const body = await request.json();
        
        // Start the orchestration with the specified orchestratorName
        const instanceId = await client.startNew(request.params.orchestratorName, undefined, body);

        context.log(`Started orchestration with ID = '${instanceId}'.`);

        // Return the status check response so the client can track the orchestration
        return client.createCheckStatusResponse(request, instanceId);
    },
});

```

**Output:**



<img width="722" alt="image" src="https://github.com/user-attachments/assets/bbf29496-1d0e-4d5f-8c73-9a49cbf7ac49">
