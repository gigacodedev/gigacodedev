# Batch Transcription

Consider the following scenario:

* Your job as service desk manager is to keep the wheels on the helpdesk spinning smoothly. Part of that is ensuring verbal phone conversations are following company policy and are polite
* You need to perform this task once per week, and even with a random selection of calls, it still can take 1-2 hours to review
* The end result of this is a simple summary of the call and general sentiment. You realize this can be automated!

With the above criteria, we have what we need to start automating! But let's get a handle on how this API works first

### Gathering recordings

Before we process speech, we need something to give the API to process. With batch transcriptions, you must provide a URI to download the call audio from. In practice, you would want to point this to the recordings endpoint of your calling software that houses the recordings. For the purposes of demonstration, I used AI Text-to-Speech to make a demo call and published it to a publicly available storage blog for ease of access. [You can listen to it here](https://gigalabstorage.blob.core.windows.net/call-audio/DemoCall.wav). We can use this same link to feed the audio into the speech service

## Step 1: Authentication

We'll be using our API keys for this demonstration. These can be found in the Keys and Endpoint section of your Speech Service in the Azure portal:\


<figure><img src="../../../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Copy Key 1 somewhere safe for now. We need a secure way to get these keys, so let's use a KeyVault!

1. Provision the keyvault resource. Similar to the speech service, click the Create button in your Resource Group, then search for and choose Key Vault\
   ![](<../../../../../../.gitbook/assets/image (5).png>)
2.  Configure the deployment details on the Basics page\


    <figure><img src="../../../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

    Similar to the Speech service, the cost of a KeyVault is very small. 10,000 secrets transactions, much more than we will need, only costs $0.03. The Standard pricing tier is sufficient for most things.
3. Proceed to Review + Create, then create the resource
4.  In your keyvault, head to the secrets page and select Generate/Import\


    <figure><img src="../../../../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
5. Enter the name and paste your API key into the Secret Value box. Ensure the secret is enabled and click Create

### Authenticating to our KeyVault

Now that we've securely stored our key somewhere we can pull from, we need a way to authenticate to the vault itself. If you've worked with Azure Service Principals, these steps should be familiar



1. In the Entra Identity portal, select Applications, then App Registrations. Create a New Registration\
   ![](<../../../../../../.gitbook/assets/image (8).png>)\
   ![](<../../../../../../.gitbook/assets/image (9).png>)
2.  Enter the name, select the Single Tenant account type, and create the registration\


    <figure><img src="../../../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
3.  In the app registration, go to the Secrets page, then create a new secret\


    <figure><img src="../../../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

    **This secret will not be displayed again! Save it somewhere safe**
4.  Go to the overview and also note down the Application (client) ID and the Directory (tenant) ID. I've saved all as environment variables in my API interaction tool, Insomnia\


    <figure><img src="../../../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>


5.  Lastly, we need to authorize this app registration to get secrets from our KeyVault. In the KeyVault, go to the Access Control page and add a new role assignment\


    <figure><img src="../../../../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>
6.  Select the role Key Vault Secrets User, then add the Service Principal we just created\


    <figure><img src="../../../../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## Step 2: Creating the Batch job

First up, we need our API key to interact with the service. Let's get it from our KeyVault. Using our API tool, we can send the following request to get an access token:

```sh
curl --request POST \
  --url https://login.microsoftonline.com/YourTenantId/oauth2/v2.0/token \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data client_id=YourClientID \
  --data 'client_secret=YourSecret' \
  --data grant_type=client_credentials \
  --data scope=https://vault.azure.net/.default
```

The response should contain access\_token, token\_type, and expiration values. Note the access token down.

```json
{
	"token_type": "Bearer",
	"expires_in": 3599,
	"ext_expires_in": 3599,
	"access_token": "abc123"
}
```

We can now request our secret from the vault by using the following request:

```
curl --request GET \  
  --url 'https://{VaultURL}/secrets/{SecretName}?api-version=7.4' \
  --header 'Authorization: Bearer {YourAccessToken}'
```

The response should contain a "Value", which is the secret. Note this down

```
{
	"value": "abc123",
	"id": "https://{VaultURL}/secrets/{SecretName}/{SecretVersion}",
	"attributes": {
		"enabled": true,
		"created": {UNIXTimeStamp},
		"updated": {UNIXTimeStamp},
		"recoveryLevel": "Recoverable+Purgeable",
		"recoverableDays": 90
	},
	"tags": {}
}
```

Now that we have our authentication, let's ask the Speech API to make us a new batch job. The endpoint you need to send it to is based on the region the service was deployed in. For example, I deployed to US West 3, so my endpoint URL starts with `https://westus3.api.cognitive.microsoft.com`. You can find this on the overview of your provisioned service.&#x20;

\
We want to ask the Transcriptions service to do something, so the full URL should look like this: `https://westus3.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions`. The first part tells it what region to look in for our service, what service we'll be using, the API version, and finally the task we want to happen.

The body for this request should be formatted as:

```json
{ 
    "contentUrls": [
        "URLToRecording"
    ], 
    "locale": "en-US", 
    "displayName": "UniqueName", 
    "model": null, 
    "properties": { 
        "wordLevelTimestampsEnabled": true
    }
}
```

This sets the property for our batch transcription job. If we wanted to provide multiple URLs they can be added to the `contentUrls` array. Properties sets the properties of the job itself, be sure to add any additional `candidateLocales` if needed.

{% hint style="info" %}
The display name of the batch job must be unique. Since we will be pulling these via an automation once implemented to your favorite RPA platform, we can simply use a UNIX timestamp to ensure a unique name
{% endhint %}

Lastly, we need to auth the request. The API key we grabbed from KeyVault should be put in a header named "Ocp-Apim-Subscription-Key". All Togther, the request looks like:

```sh
curl --request POST \
  --url https://{Region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions \
  --header 'Content-Type: application/json' \
  --header 'Ocp-Apim-Subscription-Key: {API KEY}' \
  --data '{
  "contentUrls": [{ContentURLs}],
  "locale": "en-US",
  "displayName": "{UniqueName}",
  "model": null,
  "properties": {
    "wordLevelTimestampsEnabled": true
  }
}'
```

Then we can expect a response of:

```json
{
  "self": "https://{Region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions/{GUID}",
  "model": {
    "self": "https://{Region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions/{GUID}"
  },
  "links": {
    "files": "https://{Region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions/{GUID}/files"
  },
  "properties": {
    "diarizationEnabled": false,
    "wordLevelTimestampsEnabled": true,
    "displayFormWordLevelTimestampsEnabled": false,
    "channels": [
      0,
      1
    ],
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  },
  "lastActionDateTime": "2023-09-02T21:02:04Z",
  "status": "NotStarted",
  "createdDateTime": "2023-09-02T21:02:04Z",
  "locale": "en-US",
  "displayName": "{Name}"
}
```

The important bits here are the `displayName` and the top level `self` link. Note them down.

Now we've created and started our batch job, and we want to make sure it's finished. By running a simple GET request against the `self` URL, we will receive a report for the batch job. This will contain all the above attribute, but with the addition of a `status`. This `status` string will show `Succeeded` once processed. Once successful, you'll also see a new Files object with a new URL. Query the batch job until it shows Successful:

```sh
curl --request GET
--url {SelfUrl}
--header 'Ocp-Apim-Subscription-Key: {API Key}'
```

Now we can check where our content can be found for this transcription. Query the same URL, but with `/files` appended

```sh
curl --request GET
--url {SelfUrl}/files
--header 'Ocp-Apim-Subscription-Key: {API Key}'
```

Your response should look like:

```json
{
    "values": [
        {
            "self": "https://{region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions/{GUID}/files/{GUID}",
            "name": "contenturl_0.json",
            "kind": "Transcription",
            "properties": {
                "size": 67095
            },
            "createdDateTime": "2023-09-02T21:02:21Z",
            "links": {
                "contentUrl": "{Results URL}"
                }
            },
            {
            "self": "https://{Region}.api.cognitive.microsoft.com/speechtotext/v3.1/transcriptions/{GUID}/files/{GUID}",
            "name": "report.json",
            "kind": "TranscriptionReport",
            "properties": {
                "size": 222
            },
            "createdDateTime": "2023-09-02T21:02:21Z",
            "links": {
                "contentUrl": "{Report URL}"
            }
        }
    ]
}
```

With that, we're just about done! Very last step is to actually get the results. Query the contentURL for the `contenturl_0.json` file and inspect the results. You'll see an object for CombinedRecognizedPhrases, then one for `lexical` within that - That's our transcription!

{% code overflow="wrap" %}
```
"hello it support desk how may i help you today hi my computer says i need to apply an update for my new software but i need administrator access can you provide that for me sure i will remote into your pc in and take care of this thank you if you watch what happens when i click the update it brings up this prompt asking you to confirm you need administrator access for this application go ahead and click the next button for me ok perfect now you'll see a change to an approval countdown ok got it and now it shows approved going forward it will automatically allow you to run this update application as an administrator this process works on all other applications so you can send in a request yourself next time as well or we'd be happy to walk you through it again if needed ok that makes sense thank you is there anything else i can help you with today while we are connected no that is all thank you very much you're welcome have a great day you too bye bye bye"
```
{% endcode %}

## Next steps

As you can see, that output is not quite human readable. But that's OK, now we can do a number of things to this output to make it something useful! Now that we have a transcript, we can send it to an AI Language Model like GPT-4 to create a summary, for example. That's a bit outside the scope of this doc, but here's what you can expect as an output:

{% code overflow="wrap" %}
```
In this call, the customer needed administrator access to update new software on their computer. The IT support agent remotely accessed the customer's PC to assist with the process. The agent demonstrated how to gain administrator approval for the software update and confirmed that the change was successful. The agent also informed the customer that this approval process can be applied to other applications and that the customer could either send in a request themselves in the future or seek assistance again. The customer had no further issues and expressed their gratitude before ending the call. Overall, the issue was efficiently resolved.
```
{% endcode %}
