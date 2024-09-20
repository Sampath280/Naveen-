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
