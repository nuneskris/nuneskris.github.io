---
title: 'ConversationAI: Integrating DialogflowCX with backends data sources using webhooks'
collection: talks
type: "Integrated backend SQL Spanner data store to DialogflowCX via Cloud Function"
permalink: /talks//ConversationAIDialogflowCXWebhooks
date: 2024-08-17
---

<img width="690" alt="image" src="https://github.com/user-attachments/assets/9fc73e3e-3afe-4797-b905-0726c671aeaf">

I explained the [setup of DialogflowCX ](https://nuneskris.github.io/teaching/DialogFlowCXCloudFunction) and I explained how to create a conversational flow design and demonstrated how NLU can interpret user requests and understand context. I also used Entities and Intents to navigate and use variables as parameters provided by the user.

This approach does not focus on machine learning but provides integrations with Dialogflow, a Conversational AI Platform.


# Cloud Spanner

![image](https://github.com/user-attachments/assets/4be6ceca-84e9-4fd8-83c1-4ec6f159846e)

I have uploaded the data to.

![image](https://github.com/user-attachments/assets/20a27235-f622-4de4-a5f9-1700cad6af8c)


# Cloud Function - Request Handler
* We will use cloud function as a request handler.
* Ensure the spanner and cloud function roles are assigned to the servce agent.
* Also update the requirements.txt as I forgot and wasted sometime debuging this.

![image](https://github.com/user-attachments/assets/39c33915-a7bf-4613-87b3-f554182d35a1)


There are 3 parts to this cloud function:
1. parse the request from DialogflowCX
```python
intent = req.get('intentInfo').get('displayName')
parameters = req.get('intentInfo').get('parameters')
```
3. Cloud Function to query Cloud Spanner.
```python
  with database.snapshot() as snapshot:
    results = snapshot.execute_sql(query)
```
5. Frame the response to DialogflowCX
```json
  "session_info": {
            "parameters": {
                "param-url": output
            }
        }
```

Below is the cloud funciton
```python
import json
import os
from flask import Flask, request, jsonify
from google.cloud import spanner

# Replace with your instance ID and database ID
INSTANCE_ID = "knowledgebase"
DATABASE_ID = "knowledgebase"
# Initialize the Spanner client and connect to the database
spanner_client = spanner.Client()
instance = spanner_client.instance(INSTANCE_ID)
database = instance.database(DATABASE_ID)

app = Flask(__name__)

@app.route('/', methods=['POST'])
def webhook(request):
    # Part 1: parse the request
    # Print the request data for debugging
    print(f"Request data: {request.get_json()}")

    # Get the request data from Dialogflow
    req = request.get_json(silent=True, force=True)

    # Check if req is None
    if req is None:
        return jsonify({"fulfillmentText": "Error: No request data received."})

    # Extract relevant information from the request
    intent = req.get('intentInfo').get('displayName')
    parameters = req.get('intentInfo').get('parameters')

    print(f"parameters: {parameters}")

    # Access the 'topic' parameter
    topic = parameters.get('topic', {}).get('resolvedValue')

    # Part 2: Cloud Function to query Cloud Spanner.

    # Your SQL query
    query = f"SELECT url FROM urlmapping where topic = '{topic}'"

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(query)

        # Process the results
        output = []
        for row in results:
            output.append(f"URL: {row}")


    response_text = "Some text from the cloud function: Please go to the URL for more information."


    # Part 3: Frame the response.
    response_payload = {
        "fulfillment_response": {
            "messages": [
                {
                    "text": {
                        "text": [response_text]
                    }
                }
            ]
        },
        "session_info": {
            "parameters": {
                "param-url": output
            }
        },
        "payload": {
            "customField1": "customValue1"
        }
    }

    # Return the response as a JSON object
    return jsonify(response_payload)

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(debug=True, host='0.0.0.0', port=port)
```
# Webhook

Creating a webhook is very simple. Just point to the HTTP URL of the cloud function and we ready to go.

![image](https://github.com/user-attachments/assets/2eb7de3e-516f-4e41-9920-ffa813a53784)


# Fullfilment from the webhook
Now we need to inform the intent to use the webook for fullfilment of the reposnse.

![image](https://github.com/user-attachments/assets/d27003b0-ac39-48d4-b0d3-939f0c7e9e1c)

We update the Agent reposnse to read the paramter which we send from the cloud funciton.
![image](https://github.com/user-attachments/assets/d7cb69ab-5be0-40be-9ec9-7eab92346c36)

# Testing this

1. DFCX with NLU interpreted the request parameter as a query for foundational basics.
2. DFCX then calls a webhook, which triggers a Cloud Function.
3. The Cloud Function parses the parameter 'topic.' Since the value is 'foundation,' it queries the Spanner table.
4. The Spanner table returns the following URL: 1, foundation, https://nuneskris.github.io/portfolio/.
5. The Cloud Function sends the URL back to DFCX: https://nuneskris.github.io/portfolio/.
6. DFCX responds to the user with the provided URL.

![image](https://github.com/user-attachments/assets/7f6aa754-4e25-4021-9e16-2db70d6cc43f)

