---
description: Some housekeeping before the fun stuff...
---

# Creating your Bot Service

## Creating a Bot Service resource

Before we can start working on the bot logic itself, we need to make the Bot Service that will support the rest of this project. In your Microsoft Azure portal, head to the resource group you want the resource to be created in. Then, click the Create button.

![](<../../../.gitbook/assets/image (27).png>)

In the Marketplace, search for Bot and create a new Azure Bot resource

{% hint style="info" %}
Tip: Check the "Azure Services Only" box to easier find Azure resources
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

On the Basics tab, create a handle for your bot. This is a unique ID for your bot, and while you can change its display name later, you cannot change the handle. Set the subscription, resource group, and data residency. Select the free plan under Pricing.

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

### Types of App Identities

Choosing the correct App Type under Microsoft App ID is crucial. The different types are:

* Single Tenant Identity
  * For Bots that will only be accessed from your tenant and will use your tenant for authentication. This is only supported by the C# and JavaScript SDK.&#x20;
  * Authentication is handled with Client ID and Secret
* Multi Tenant Identity
  * For bots that will be accessed from multiple Microsoft tenants and will use the bot service for authentication. Supported by all SDKs.
  * Authentication is handled with Client ID and Secret
* User Assigned Managed Identity
  * For bots that will be accessed from multiple Microsoft tenants and will use the bot service for authentication. Supported by only the C# and JavaScript SDK.
  * Authentication is handled by an Azure Managed Identity, and cannot be used with Client ID and Secret

Given these behaviors, we'll choose Multi-Tenant as we want a Client ID and Secret to be used by our RPA platform. If we were making a bot that was hosted on an Azure App Service, Managed Identity would be recommended.

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

You can now proceed to assign any tags you may want, and create the resource

## Configuring the Bot Service

#### Customizing your bot

Once your resource is created, navigate to it. Select Bot Profile under Settings in the left navigation bar

![](<../../../.gitbook/assets/image (8).png>)

On the Profile page you can set your bot's icon, name, and description

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption><p>Icon courtesy of DALL-E :)</p></figcaption></figure>

#### Connecting your bot to Teams

Click the Channels button in the left navigation bar

![](<../../../.gitbook/assets/image (14).png>)

Select the Microsoft Teams channel

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Accept the terms and click the Apply button to choose the Microsoft Teams Commercial channel. You'll now see Teams under your connected channels with a Healthy icon

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## Getting your bot's Authentication

Your bot will happily send requests to you all day, but when responding you need to specify who you are back. By selecting Multi-Tenant on our app type, a new App Registration has been created in our Azure Active Directory tenant. Head on over to the Entra portal -> Azure AD -> Applications -> App Registrations

![](<../../../.gitbook/assets/image (25).png>)

Under Owned Applications, select your bot

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

In the Essentials tab on the Overview page, note the Application (client) ID, and Directory (tenant) ID

![](<../../../.gitbook/assets/image (6).png>)

Go to Certificates & Secrets, then create a new Client Secret. Name this secret and set its validity length

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Copy the Secret Value to your clipboard before leaving the page! This secret will not be shown again, so document it somewhere safe

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>
