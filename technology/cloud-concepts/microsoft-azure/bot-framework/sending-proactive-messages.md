---
description: Everyone could use some more messages!
---

# Sending Proactive Messages

So far we've learned the basics of how the Azure Bot Service works, how to create our own bot and associated resources, how to receive messages, and then how to reply to them. But what if we want to send a message to a user before they message us first? With a Conversation ID and User ID we can!

### What you'll need

* An Azure Bot integrated to Teams
  * Haven't been following along? [Start here!](intro-to-bot-framework.md)
* An App Registration for your bot
  * You'll also need the Client ID and Secret from this to request access tokens
* Some sort of database (OPTIONAL)
  * We'll go into this a bit further later on, but some attributes of the conversation with a user, such as its ID, generally won't change once they've been assigned. It is much quicker and more efficient to query these from a database.

### Getting conversation and user IDs

Before you can send a message, you need to know where to send it to. The user part of this is easy, this can be just a user's Entra Identity User Object ID (aka AAD Object ID). However, we also need to have a valid conversation in teams to send it to. Lucky for us, one is created automatically in a few scenarios:

* A user installs your bot as an app
  * We'll cover this later, so don't worry too much about it now
* A user messages your bot
* A user starts a new chat with your bot
* A user adds your bot to an existing conversation

In this guide, we'll mainly be focusing on conversation IDs from prior messages and new chat sessions. In the next post, we'll review how to package your bot as an installable native Teams app for the first option.

So, let's get some IDs! Here's a trimmed down version of what you get when someone messages your bot:

```json
{
  "id": "1693284ccccc",
  "from": {
    "id": "29:1mZd6lzxxxxx",
    "name": "Brandon Martinez",
    "aadObjectId": "xxxx"
  },
  "text": "Hello!",
  "type": "message",
  "channelId": "msteams",
  "conversation": {
    "id": "a:xxxxxxxx",
    "tenantId": "xxxxx",
    "conversationType": "personal"
  }
}
```

From top to bottom, we have an ID to reference back to for the message itself, from information including the name and AAD Object ID of the user, and the conversation ID. Perfect! I'll set my RPA platform to go ahead and save these values in a database table, but if you're following along without one, you can just note these down.

{% hint style="info" %}
The from ID and conversation ID are different things despite being in the same format. We want just the conversation ID. Be sure not to mix them up!
{% endhint %}

What about when someone starts a new chat or adds your bot? You'll get something that looks like this:

```
{
  "id": "",
  "type": "conversationUpdate",
  "channelId": "msteams",
  "conversation": {
    "id": "a:xxxxx",
    "tenantId": "xxxxx",
    "conversationType": "personal"
  },
  "membersAdded": [
    {
      "id": "29:xxxxx",
      "aadObjectId": "xxxxx"
    },
    {
      "id": "xxxxx"
    }
  ]
}
```

We can see the same data is available here, but structured slightly differently. Note how the `membersAdded` property is a list of two objects, one containing an `aadObjectId` and the other with just a generic ID. Since technically both the user and your bot were added to the same conversation, they both appear in the list of added members. Your bot won't have an `aadObjectId` attribute, so it's easy to distinguish which one is your user. The `id` value in the object without it will also match your bot's ID.&#x20;

{% hint style="info" %}
Curious about how it works when the bot is added as an app? We'll come back to that in the next post
{% endhint %}

### Sending the message

{% hint style="info" %}
We'll can use the same Access\_Token we've used when replying to a message, provided it's not expired. If it is, the same request can be resent to generate a new one. Not sure how to get an access token? Check out this prior post: [#getting-authenticated](interacting-with-your-bot-services-api.md#getting-authenticated "mention")
{% endhint %}

Now that we have the information we need, we can proceed to actually sending the message. Believe it or not, this is the easiest part. Why? You've already done it! Helpfully, the same JSON schema is used for replying and sending proactive messages. This should look familiar:

```sh
curl --request POST \
  --url https://smba.trafficmanager.net/amer/v3/conversations/{conversationId}/activities \
  --header 'Authorization: Bearer {Access_Token from prior step}' \
  --header 'Content-Type: application/json' \
  --data '{
    "type": "message",
    "from": {
        "id": "BotID",
        "name": "BotName"
    },
    "conversation": {
        "id": "ConversationId"
    },
   "recipient": {
        "id": "UserID",
        "name": "UserName"
    },
    "text": "Hello, Human!"
}'
```

And, when you've successfully sent the request, you'll get an ID back just the same:

```json
{
	"id": "1693680152973"
}
```

