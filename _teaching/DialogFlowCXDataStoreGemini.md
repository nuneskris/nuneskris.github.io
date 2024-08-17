---
title: "GCP DialogFlowCX Data store LLM generated agent responses based Knowledgebase in Storage."
collection: teaching
type: "AI/ML"
permalink: /teaching/DialogFlowCXDataStoreGemini
venue: "DialogflowCX"
location: "GCP"
date: 2024-08-16
---
<img width="740" alt="image" src="https://github.com/user-attachments/assets/5e167eaf-b5f4-486a-a513-0a38214bc7e5">

This is a continuation of the DialogFLowCX Sandbox setup which I had detailed in the page: [GCP DialogFlowCX: Setup, Flow, Intent, Entities and Parameters](https://nuneskris.github.io/teaching/DialogFlowCX)

I have built a integration of Cloud Function and Spanner as a Webhook to DiaglowFlowCX in this page: [ConversationAI: Integrating DialogflowCX with backends data sources using webhooks](https://nuneskris.github.io/talks/ConversationAIDialogflowCXWebhooks)

I will be continuing the sandbox in this page to bring in AI Capabilitieis into DialogFlowCX. I will create a Data Store Vertex AI Agent, which can be based on various sources like Cloud Storage, websites, or other data sources. I will be using Cloud Storage. When a user interacts with your agent, the agent would need to access information from the data store or knowledge base to provide a relevant response. Vertex AI provides conversational AI capabilities to the agent's responses to specific information stored in a data store.

# Creat a knowledge base in Cloud Storage
I copied multiple files from my githubpages over to Cloud Storage.

![image](https://github.com/user-attachments/assets/60cb288c-921a-4ba7-89bf-a507db77073f)

![image](https://github.com/user-attachments/assets/7b8e87d0-a466-4f75-b148-a4ac234fd9ef)

# Create a DataStore Agent

Data store agents are a special type of Dialogflow agent that can provide LLM generated agent responses based on your website content and uploaded data.

We just need to select the folder and we are good to go. There were around 60 files and it took just a few minutes to index. 

I wanted to index my github pages but thats for another time.

![image](https://github.com/user-attachments/assets/d948f513-f52d-43bd-83cf-e240ed558dc8)

# Connect the DataStore Agent to the DialogFlowCX Flow App

![image](https://github.com/user-attachments/assets/ae2e9426-59ae-4fb7-ae96-d7a0edccb6e4)

This is a very simple step. I have a page "Knowledge base" which I will use. Under StateHandler (Documentation is very poor on this) we can add data stores. 
Select the Data Store App.

![image](https://github.com/user-attachments/assets/017d0c53-3a1c-4849-8f82-c7acd311350a)

# Selecting the Generative AI which we be applied on the DataStore Agent

Under the Agent Settings, proceed to the Generative AI section, and then to DataStore. 
 
1. Low Grounding score. Know more below.
2. Selected gemini1.0pro001 as the prompt
3. Fall back was default to use generative AI

![image](https://github.com/user-attachments/assets/de639667-445f-49c9-a5d1-5f77b0e91a58)
![image](https://github.com/user-attachments/assets/2adec5ca-8ebe-4481-955e-563dd570fa45)

# Testing the agent
![image](https://github.com/user-attachments/assets/d987a963-4ef7-4cc5-bb38-dab4ef24a708)


# Playing with Grounding Score

Grounding means that the agent's responses are not just generic or made up, but are grounded in real, factual information retrieved from the data store. In other words, the agent uses the data store to "ground" its responses. This means it retrieves specific information from the data store and uses it to formulate its answers

The "grounding score" is a measure of how confident the system is that the chatbot's response is based on real, factual information from a knowledge base.
  
## High Grounding Score
* Dialogflow CX has a high level of confidence that the chatbot's response is grounded in the knowledge base. This means the system has found relevant information in the knowledge base that supports the response.
* Responses with high grounding scores are more likely to be accurate and factual.
* Users are more likely to trust a chatbot that provides responses with high grounding scores.

## Low Grounding Score
* Dialogflow CX has low confidence that the response is grounded in the knowledge base. This could mean that the system didn't find any relevant information or that the information it found was not very strong.
* Responses with low grounding scores might be less accurate or might contain information that is not fully supported by the knowledge base.
* Users might be less likely to trust a chatbot that provides responses with low grounding scores.

Test 1: Direct questions which I know I have a paragraph on.

> Side Note: NLU (another component of DialogFlowCX) was not able to handle a spelling mistake which I thought it should have. But maybe not.

![image](https://github.com/user-attachments/assets/aab1880a-4576-475a-ab3d-3c05f3efae31)

I did a compare and it made my answer better.

![image](https://github.com/user-attachments/assets/ee09735d-739e-4119-a659-2b41e17e88a6)

Test 2: Question which I have a page but not a paragraph

![image](https://github.com/user-attachments/assets/94123333-5d47-408a-8f3e-5fb2d272afbe)






