---
title: "GCP DialogFlowCX Data store LLM generated agent responses based Knowledgebase in Storage."
collection: teaching
type: "AI/ML"
permalink: /teaching/DialogFlowCXDataStoreGemini
venue: "DialogflowCX"
location: "GCP"
date: 2024-08-16
---

I will be continuing the sandbox page where I set up DialogFlow CX with a few routes.

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

1. Selected gemini1.0pro001 as the prompt
2. Allow low score grounding

![image](https://github.com/user-attachments/assets/2adec5ca-8ebe-4481-955e-563dd570fa45)

# Testing the agent
![image](https://github.com/user-attachments/assets/d987a963-4ef7-4cc5-bb38-dab4ef24a708)


![image](https://github.com/user-attachments/assets/de639667-445f-49c9-a5d1-5f77b0e91a58)






