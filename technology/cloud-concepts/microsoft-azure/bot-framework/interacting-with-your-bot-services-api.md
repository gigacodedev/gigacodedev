---
description: Who needs an SDK?
---

# Interacting with your Bot Service's API

With the configuration done, we can now test out our Bot Service. We'll need to connect a messaging endpoint to receive the messages and send a response. For the sake of testing, we'll use the awesome tool [https://webhook.site](https://webhook.site). This will allow us to generate a webhook and capture any data sent to it for inspection. Copy your custom webhook URL

<figure><img src="../../../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go back to your Bot Service and click on Configuration in the left navigation bar. Enter your webhook URL into the Messaging Endpoint field and apply the settings

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Go back to the Channels tab in the left navigation bar, and click "Open in Teams" under Actions for your Teams channel connector

<figure><img src="../../../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

Teams will now open a conversation with your bot! Send it a message so it can send data to Webhook.Site

<figure><img src="../../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

## Receiving messages

Back in our Webhook.Site tab after we sent something to our bot chat, you'll find data that was sent as a POST to our webhook. This includes information on what the message says, where it came from, who sent it, and how to reply to it

<figure><img src="../../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

## Responding to Messages

To respond to our message, we'll need to send a POST back to the bot service. There's a variety of apps that can help with this, the most common being Insomnia and Postman. I'll be using Insomnia, but the steps on other API tools should be similar. I'll also include `curl` commands for those who prefer a command line.

Before we start sending requests, we need to collect some data from the POST to our webhook:

* `serviceUrl`
* `conversation.id`
* `recipient.id`
* `recipient.name`

### Getting Authenticated

We'll start by getting our authorization token to respond to this message. With your Client ID and Secret, send the following request

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default// Some code
```

In Insomnia, it looks like this:

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

To send in `curl`, format it as:

```bash
curl --data-urlencode "grant_type=client_credentials" \
--data-urlencode "client_id={clientID}" \
--data-urlencode "client_secret={clientSecret}" \
--data-urlencode "scope=https://api.botframework.com/.default" \
https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
```

When sent, the response should look like this:

```json
{
	"token_type": "Bearer",
	"expires_in": 86399,
	"ext_expires_in": 86399,
	"access_token": "eyJ0eXAiOiJKV1.....ALotMoreCharacters"
}
```

Note the `access_token` value for later use. We now have all the information we need to send our response!

### Sending a Reply

First, we need to know where to send the reply to. Since Microsoft has several regions for Teams, the message to our endpoint with the message details also contains a `serviceUrl`. This will be formed to make the base URL. The base URL will be the service URL, plus `/v3/`, plus the API endpoint.&#x20;

The endpoint we need to reply to a chat message is `conversations/{conversationId}/activities`. Therefore, with the `serviceURL` of `https://smba.trafficmanager.net/amer/`, our reply URL should be `https://smba.trafficmanager.net/amer/v3/conversations/{conversationId}/activities`

The body for the reply to a chat message should be in JSON and at minimum contain:

```json
{
    "type": "message",
    "from": {
        "id": "BotID",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234"
    },
   "recipient": {
        "id": "UserID",
        "name": "user's name"
    },
    "text": "Response to human!"
}
```

Putting this all together, in our case this would look like the following in Insomnia

<figure><img src="../../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

In `curl`, this would be:

```
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

When successfully sent, you'll get a `201` success code for the request and the response body of an ID for your message

<figure><img src="../../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Your message is then sent to the user!

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>
