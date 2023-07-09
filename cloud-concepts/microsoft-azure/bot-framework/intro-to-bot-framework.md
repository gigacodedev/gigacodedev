---
description: Hello, human...
---

# Intro to Bot Framework

## What is Bot Framework and Bot Service?

The Microsoft Bot Framework, which includes Azure Bot Services, is a library of building blocks to support creation of conversational bots. The concept of a Microsoft Teams helper app utilizing Adaptive Cards can be easily accomplished with Webhooks, however, Bot Framwork enables you to create a more natural interaction between your application and its users. The Bot Framework includes an SDK for C# and JavaScript (Java and Python SDKs are retiring 11/2023), and a RESTful API. For the first few documents, we'll be focusing on the REST APIs.

## What do I need to make a simple Teams Bot?

Bots contain a few major elements. From the start, an "Activity" is sent from your selected channels. This activity includes where the interaction came from, what happened, who sent it, and other conversational details. For example, a chat from Teams would include the user's display name and AzureAD Object ID, the conversation ID and message ID to reply to, and the contents of the message. These activities are then sent from the Bot Service to your Messaging Endpoint. This endpoint is a web app that is able to intake these activities, interpret them, take actions, and send a reply back. This can be created with the Bot Framework SDK, or any other automation platform that can support standard RESTful JSON API calls.&#x20;

## What methods will be shown to make a bot?

As this blog is mainly geared towards the MSP industry, I will be making these guides somewhat generic to what tool you would like to use. As long as it can interact with a REST API, it can work with bot service.

I personally utilize the RPA platform Rewst, so will make a separate post on how to set up a Rewst workflow as the messaging endpoint. This, however, is not required to be used.
