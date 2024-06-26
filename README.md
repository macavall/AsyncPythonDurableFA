# Durable Function App with Asynchronous calls to the Activities 

The changes from the main [Documentation](https://learn.microsoft.com/en-us/azure/azure-functions/durable/quickstart-python-vscode?tabs=windows%2Cazure-cli-set-indexing-flag&pivots=python-mode-decorators)

## Orchestratos Code - The Main Change

```python
# Orchestrator
@myApp.orchestration_trigger(context_name="context")
def hello_orchestrator(context):
    tasks = []
    cities = ["Tokyo", "Seattle", "London"]
    
    for city in cities:
        tasks.append(context.call_activity('hello', city))
    
    results = yield context.task_all(tasks)

    return results
```

## Complete Code

  ```python
import azure.functions as func
import azure.durable_functions as df

myApp = df.DFApp(http_auth_level=func.AuthLevel.ANONYMOUS)

# An HTTP-Triggered Function with a Durable Functions Client binding
@myApp.route(route="orchestrators/{functionName}")
@myApp.durable_client_input(client_name="client")
async def http_start(req: func.HttpRequest, client):
    function_name = req.route_params.get('functionName')
    instance_id = await client.start_new(function_name)
    response = client.create_check_status_response(req, instance_id)
    return response

# Orchestrator
@myApp.orchestration_trigger(context_name="context")
def hello_orchestrator(context):
    tasks = []
    cities = ["Tokyo", "Seattle", "London"]
    
    for city in cities:
        tasks.append(context.call_activity('hello', city))
    
    results = yield context.task_all(tasks)

    return results

# Activity
@myApp.activity_trigger(input_name="city")
def hello(city: str):
    return f"Hello {city}"
```
