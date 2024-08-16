---
title: "GCP DialogFlowCX with CloudFunction Backend"
collection: teaching
type: "AI/ML"
permalink: /teaching/DialogFlowCXCloudFunction
venue: "DialogflowCX"
location: "GCP"
date: 2024-08-16
---

# Setup
* Enable relevant APIs. The console will guide
* Diaglogflow requires us to enable billing. Annoying. I created a new project which I will tear down
* There are some nomenclature which I need to get used to. So I will establish a few explicitly, so that it is easier to understand the documentaiton and the tool.

       gcloud auth application-default set-quota-project newproject

* Security: Provide access controls for the service agent to invoke cloud function and storage.

# Agent

> Agent: The actual chatbot assistant. It is what we will configure to build out the chatbot. We only provide region and name.

![image](https://github.com/user-attachments/assets/47379265-a4ea-4ddf-b879-72b566772e1b)

# Page

A page represents a specific step or stage in a conversation flow within Dialogflow CX. It's like a screen or a section in a guided process. In a flight booking chatbot, you might have pages for:
Greeting the user and asking for their destination. Collecting the destination city. Asking for travel dates. Displaying available flight options. Confirming the booking details.

> A page is a fundamental building block of your agent flow. It represents a specific step or stage in the conversation with the user. Think of it like a screen or a section in a guided process

A default Start page is already provided.

![image](https://github.com/user-attachments/assets/092b832d-adf9-45d4-9a46-2a3ffdbccd42)

# Routes

A route defines how the conversation transitions from one page to another. It's like a path or a connection between pages.

> Every page will have routes. When a page recieves a request message from the user a route gets activated based on the end-user input matches an intent and/or some condition on the session status is met.

A route with an intent requirement is called an intent route. A route with only condition requirement is called a condition route. Lets try to know the what they are.

For our flow, we have a start page which is default and there is a route by default.

![image](https://github.com/user-attachments/assets/4cf5be30-8e98-423e-b936-db741bea9174)

# Intent

When we are on a page, we would like to the user's goal or desired action. This is an intent and they are user goals or actions that drive the conversation. Intents help Dialogflow CX understand the user's intent and trigger the appropriate flow of conversation.

When we say Hi on a chatbot, we want our start page to respond with a welcome messsage. 
So there is component to know what the input message to ***route*** to the intent and what the output response should be accoringly.







