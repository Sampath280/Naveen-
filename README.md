**For  retrieving a token in Durable functions refer to this code:**

```
const { app } = require('@azure/functions');
const df = require('durable-functions');

df.app.orchestration('reportOrchestrator', function* (context) {
    try {
      
        const instanceId = context.df.instanceId;
        context.log(`Orchestration instance ID: ${instanceId}`);
        const token = yield context.df.callActivity('TokenManager', { instanceID: instanceId });

        context.log(`Token retrieved: ${token}`);

        return token;

    } catch (error) {
        context.log(`Error in orchestration: ${error.message}`);
        throw error;  
    }
});

df.app.activity('TokenManager', {
    handler: async (input, context) => {
        const log = context.log;
        const instanceID = input.instanceID;  

        if (!instanceID) {
            throw new Error("InstanceID is missing or undefined.");
        }

        log(`Orchestrator Instance ID is ${instanceID}`);

        const logicAppURL = "<Your Logic App URL>"; 
       // let functionID = "POST Reports Orchestrator";

        try {
            
            const tokenResponse = await axios.post(logicAppURL, {
               // requestType: "GET_TOKEN",
               // functionID: functionID,
            });

           
            if (tokenResponse.data && tokenResponse.data.token) {
                log("Token successfully retrieved");
                return tokenResponse.data.token;
            } else {
                log("Token could not be retrieved. Returning an empty token.");
                return "";  
            }

        } catch (error) {
            log(`Error retrieving token: ${error.message}`);
            throw error;  
        }
    }
});

app.http('startReportOrchestrator', {
    route: 'orchestrators/{orchestratorName}',
    extraInputs: [df.input.durableClient()],
    handler: async (request, context) => {
        const client = df.getClient(context);

        const body = await request.json();
       
        const instanceId = await client.startNew(request.params.orchestratorName, undefined, body);

        context.log(`Started orchestration with ID = '${instanceId}'.`);
        return client.createCheckStatusResponse(request, instanceId);
    },
});


```



**For parameters refer to these steps below:**



The [Durable Functions](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) extension introduces three trigger bindings that control the execution of orchestrator, entity, and activity functions.

For your code I see only Activity function refer to this [git](https://github.com/Sampath280/Naveen-/blob/main/README.md) for full with token retrieval in Durable functions.



I have used this [doc](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-bindings?tabs=python-v2%2Cin-process%2C2x-durable-functions&pivots=programming-language-javascript) as a reference for sample code to pass and handle parameters from the body.

 We have to extract input parameter from the orchestration context
```js

    const input = context.df.getInput();
```
Call the activity function with the provided input or default values
```js
    for (const city of input.cities || ['Tokyo', 'Seattle', 'Cairo']) {
        outputs.push(yield context.df.callActivity(activityName, city));
    } 
 ```
We need to  Handle the input parameter and return a response  as below
  ```js
        return `Hello, ${input}`;
  ```
We need to  parse the body of the request to get the input parameter for the orchestrator   as below
```js
  const body = await request.json();
  ```
  Start the orchestration with the input parameter
  ```js
        const instanceId = await client.startNew(request.params.orchestratorName, undefined, body);
  ```

![enter image description here](https://i.imgur.com/ywmDkMR.png)

**Query parameter in a URL:**
 To pass a query parameter in a URL In the  function, We need to extract the `Naveen` query parameter from the URL and pass it to the orchestrator
```js
    const Naveen  = request.query.Naveen;

        const instanceId = await client.startNew(request.params.orchestratorName, { input: Naveen  });

        context.log(`Started orchestration with ID = '${instanceId}' and Naveen = '${Naveen }'.`);

        return client.createCheckStatusResponse(request, instanceId)

```
You can access value  passed as part of the `input` object to the orchestrator as below:
```js
 const outputs = [];
    const Naveen = context.df.getInput().input; 
    
    outputs.push(yield context.df.callActivity(activityName, Naveen));

```

![enter image description here](https://i.imgur.com/OJG21AR.png)
