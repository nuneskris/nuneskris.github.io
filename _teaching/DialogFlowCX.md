---
title: "GCP DialogFlowCX: Setup, Flow, Intent, Entities and Parameters"
collection: teaching
type: "AI/ML"
permalink: /teaching/DialogFlowCX
venue: "DialogflowCX"
location: "GCP"
date: 2024-08-16
---

<img width="278" alt="image" src="https://github.com/user-attachments/assets/e2ca4dd7-ed02-485b-857d-ed00c8c07899">

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
So there is component to know 
1. what the input message to ***route*** to the intent
2.  what the output response should be accoringly.

Lets open the intent to figure the above 2 out.

## 1. Training phrases
This is what we use to trigger route to this intent.

![image](https://github.com/user-attachments/assets/5118de52-0374-4af1-8b72-4d0ac2eb5fac)

I have added 2 more phrases. 
![image](https://github.com/user-attachments/assets/a5ffbceb-baac-4ce9-abcb-9404eb8436d2)

## 2. Agent Response
The agent response is what the agent responds. Note this can be configured both at the route and at the intent within the route. This seems to be redundant. But, when I am thinking about it, I am assuming the one on the route has more features like condioning etc. Not sure.

Any way I have added a response.

![image](https://github.com/user-attachments/assets/73f6cad7-4174-4a33-a158-421748c8e6d0)

## Time to test this

I types Hi, who are you, and it was able to respond the messsage which was configured.

![image](https://github.com/user-attachments/assets/b70d1664-8132-4516-943f-e098b5a11f90)


# Now lets route to a new page

I am creating a new intent which will handle questions on the locaiton of the website.

![image](https://github.com/user-attachments/assets/c3b0dccb-67c8-4c96-92ea-a8110484b585)

This will route will transition into a new page.

![image](https://github.com/user-attachments/assets/040d7143-0a75-4301-b33e-b90549d42eb8)

## Entry Fulfillment
We need to configure what the user reponds when we land on this page when they ask the question about the location of the url.
 
 > Entry fulfillment is the agent response for the end-user when the page initially becomes active

![image](https://github.com/user-attachments/assets/e96772a4-7e98-4492-9f1f-f45e9eee67dd)

## Another page to test routing to 2 pages.
I am creating another page to learn.

![image](https://github.com/user-attachments/assets/2358a7b8-f7b5-49fa-8171-3ed5d89ab2e1)

## Time to test

The DFCX agent is able to route to the right page.  Another cool observation is we can see the tool uses NLU to understand context and route to the right page even though we were in the site locaion page and it was able to direct the question to the know more page.

![image](https://github.com/user-attachments/assets/24b764dc-7786-4467-bc12-debf6b6af577)


## Parameters and Entities

We would like to use variables which can hold values based on user inputs and use these variables for conditions and flexibility. DFCX terms these variables as parameters and the values as entities.

We will have 2 types of parameters. Topics and components. Topics will be Foundation, architetcure, engineering and sandbox. Components will be Collect, curate, governance, integrate etc. So we can ask what the users want to know about and appropriatlly use these values to route.

I have created entiies on the various topics. We can also create synonyms.

![image](https://github.com/user-attachments/assets/a3a72683-832b-40dc-a130-e615c38f0a20)

Configuring a parameter on a page to hold the entity value provided by the user.
![image](https://github.com/user-attachments/assets/116303e0-c88e-4c35-b239-9f629060dd29)

Now that we have created a paramter which can hold the entities, we would like to be able to capture them in the users requests. DFCX  automatically annotates the entities in the training phrases. We can see both the entity and the sysnonyms get annontated. Let us test and see if the agent is able to capture this entity in the user request.

The agent response captures the entity by referencing to it via $session.params.sitetopic

I will stop here. I will continue with integration to an external webhook to handle a more complex agent response.
![image](https://github.com/user-attachments/assets/8d31a4a6-e622-498c-8b41-51381f7274ad)


