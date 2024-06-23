# Setting up your tenant

As mentioned in [Welcome to Zero Trust](what-is-zero-trust.md), we'll be proceeding with Cloudflare Zero Trust in these guides. However, the overall concepts should apply to most other Zero Trust Network platforms.

## Registering your accounts

To get started, you'll need to have a Cloudflare account. You can register one for free at [https://dash.cloudflare.com/sign-up](https://dash.cloudflare.com/sign-up)

Once you have your account, you'll see the Cloudflare dashboard. Head over to the Zero Trust tabs to proceed through the setup. The exact process may differ depending on the current state of the Cloudflare portal's UI and your region, but you'll be prompted to setup your Team Name and select a plan. Set the Team Name to something unique and relevant to your organization, but don't stress over it too much right now - you can always change it later

### Choosing a plan

For most people starting off, either as a personal lab tenant or in the evaluation phase for your organization, and free tier plan will be fine. The main difference you get between the free and Pay-As-You-Go plans is a 100% Uptime SLA, chat/email support, more network locations, and longer log retention. You will also be limited to a maximum of 50 users on the free tier plan. For the rest of this guide, I will be working with features available in the free plan unless stated otherwise

{% hint style="info" %}
You may be asked to enter card information at this stage. This is required for the free plan as a $0 purchase, but you will not be charged unless you upgrade
{% endhint %}

### Initial settings

Once you're in the portal, we'll start in the settings tab. We won't hit all the options here now, but here's some quick things we can update to customize the experience

### Account Settings

Not much to do here upfront, but you can change your plan and payment information if you choose to upgrade. One field to note is the User Seat Expiration. This can be used to automatically remove a license from a user if they have not signed in for a specified amount of time. I recommend, however, setting up SCIM provisioning to handle this later on in the guides

### Custom Pages

Here we can update our team name. This can be changed at any time, but must be unique. Additionally, you can configure custom block and login pages. I recommend branding these to your organization's standards to add additional trust in the platform to end users

### Network

**Gateway Logging**: The settings here can control the amount of logs sent and retained in the plaform. We'll leave this default for now, but you can choose to modify these settings if you'd like. Cloudflare collects a good amount of PII, which can be excludeed here. Cloudflare considers the following as PII:

- Source IP
- User email
- User ID
- Device ID
- URL
- Referer
- User agent

You can also enable Enhanced Logging here. This will read connection information from the HTTP body rather than the headers. Be mindful this will cause a performance hit, especially in file heavy contexts. As always, configure these options to align with your organization's specific policies.

**Firewall**: Here you can configure the Firewall settings for when you are running the WARP gateway agent on endpoints, or if you have a WARP connector configured on your private network's edge router.

- Proxy
  - Let's enable this setting first. This will ensure all traffic is proxied through the WARP agent, and network policies are applied. You can select TCP, UDP, and ICMP traffic. We'll just start with TCP

{% hint style="warning" %}
Proxying UDP traffic may have significant negative effcts on your network, especially with VOIP performance. Be sure you fully understand the impact of enabling any proxy options before doing so in a production environment
{% endhint %}

- TLS decryption
  - With most traffic being HTTPS, you will likely be limited in the amount of information you can inspect. This option will allow you to scan HTTPS traffic by allowing a root CA certificate installed on the endpoint with WARP to decrypt HTTPS traffic

{% hint style="danger" %}
This is a *very* impactful setting, and is prone to breaking things. Many apps use certificate pinning, and will not function properly with TLS decryption enabled. Exclusions must be set for these applications. Before enabling this in a production environment, be sure you've tested all line of business apps to ensure compatibility. More information can be found [in this Cloudflare KB document, which includes a list of known incompatible applications.](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/troubleshooting/common-issues/#tls-decryption-is-enabled-and-the-app-or-site-does-certificate-pinning)
{% endhint %}

- Match exteneded email addresses
  - Allow for extended email addresses, often being "plus addressing" formatted, to be used as criteria for firewall policies. For example, `username+somethingElse@domain.com`
- AV inspection
  - Enable the scanning of files through the Cloudflare Gateway. The following criteria is used to determine if a file should be scanned:
    - If the Content-Disposition HTTP header is Attachment
    - If the byte signature of the body of the request matches a signature identified as one of the following file type categories:
      - Executable (e.g., `.exe`, `.bat`, `.dll`, `.wasm`)
      - Documents (e.g., `.doc`, `.docx`, `.pdf`, `.ppt`, `.xls`)
      - Compressed (e.g., `.7z`, `.gz`, `.zip`, `.rar`)
    - If the file name in the Content-Disposition header contains a file extension that indicates it is one of the file type categories above

**Bypass decryption of Microsoft 365 Traffic** is used to create a default Do Not Inspect policy for known Microsoft 365 traffic. This optimizes performance and interoperability, and I reccomend enabling it;

With that, we've configured the initial tenant level settings! We'll cover the remaining settings pages in the following documents.
