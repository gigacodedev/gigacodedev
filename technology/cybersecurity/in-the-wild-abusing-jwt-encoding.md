# 🚨 In the Wild - Abusing JWT Encoding

In the course of securing and supporting ourselves and our customers, we often come across new TTPs, or Tactics Techniques and Procedures, as they become used. As the defenders, _we_ are the guinea pigs the threat actors test new stuff out on. This post reviews a real-life detection of what may be a new technique to further aid with ongoing efforts to evade URL sandboxing and defense. Data in this post may be obfuscated to protect our organization.

## Initial Discovery

Typically, my day starts with reviewing the general security state of our organization. This includes, among other tasks, reviewing our Microsoft Defender Incidents and Alerts to identify any detections overnight that weren't immediate enough for our SOC partner to raise the alarm. As such, these are typically user reported spam emails and Defender for 365 captured Phishing emails. These sorts of alerts can sit until the AM as Defender has already Quarantined or soft deleted the message after its Automated Investigation and Response (AIR) investigation. This is where the fun started

### The Incident

This particular incident did not seem out of the ordinary. Multiple phishing emails were sent to key individuals at our organization, Defender for 365 nabbed one, determined it to be malicious, and then the ZAP (Zero-hour Auto Purge) feature quickly removed all copies that had been delivered already. Typically, the extent of the review at this point is to document the alert, quickly spot check the AIR logs to make sure there's no further pending actions and that all entities were addressed, and then close out the alert. However, something caught my eye...

### The URL

The URL in the mail entity detection was similar to the following:

`https://drip[.]la/c/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJtb25vbGl0aCIsImlhdCI6MTY5NTc3Nzc2NSwiZXhwIjoxNzI3MzEzNzY1LCJhdWQiOiJkZXRvdXIiLCJzdWIiOiJkZXRvdXJfbGluayIsImFjY291bnRfaWQiOiI1MTY1ODE2ODEiLCJ0cmlnZ2VyX2lkIjoiOTQ4OTI4Mjk3NTE2NTEiLCJkeW5hbWljX3VybCI6Im5vX3VybF9mb3JfeW91IiwidXJsIjoibmljZV90cnkifQ.I2NvAWEujTf7ednkh6RLujdzfyzDmPmbmrZWshAXaWY`

Notice anything familiar? That URL slug certainly looks like an authentication token to me. But why would an attacker be sending us a token? Isn't it usually the other way around? This caught my eye, and my morning was quickly sidetracked

## Understanding the Technique

{% hint style="info" %}
Already familiar with JWTs?[ Jump ahead to when we get back to our detection!](in-the-wild-abusing-jwt-encoding.md#our-phishing-url)
{% endhint %}

### What's a JWT anyways?

A JWT, or JSON Web Token, is a way to format and encode data for the purposes of transmitting information over the internet as a JSON object. This string of characters is URL safe (doesn't require any additional encoding) and can easily be inserted into a URL, even if you generally do _not_ want to do that. The standard itself is defined in [RFC7519](https://datatracker.ietf.org/doc/html/rfc7519).

Due to the resulting object being relatively compact and in a common and consistent format, this is most often used to transmit authentication claims between services. If you've ever made an OAUTH2.0 request, you've seen this format before. However, as we will discover, JWTs can be crafted to transmit really any information formatted as JSON.

### How does a JWT work?

A JWT has three main components:&#x20;

* Header, containing information about the token itself
* Payload, containing the data to communicate formatted as JSON
* Signature, validating the message was not modified and sometimes verifying the sender

Each of the components is separated by a `.`, so a token typically looks like:

`header.payload.signature`&#x20;

Let's take a look step by step at each component:

#### The Header

Typically, the header contains two components: A `typ` value to inform the receiving end what this is, and if signed, a `alg` value notating the algorithm used to generate the signature. The most common algorithms are HMAC SHA256 (symmetric) and RSA (asymmetric). We'll start crafting a custom JWT by starting here:

```json
{
    "typ": "JWT",
    "alg": "HS256"
}
```

This states we are a JWT, and we are signed with HMAC SHA256. This is then encoded using Base64 URL Encoding, which is similar to Base64 but with a few URL unsafe characters changed. `+` is changed to a `-`, `/` is replaced with a `_`, the final `==` padding is omitted, and the rest is the same. This gives us a result of `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9` for our header.

#### The Payload

Next up is the meat of the request contaning our claims. While the standard from IETF for JWTs doesn't specify any claims as mandatory, they do specify several "registered" claim names to support interoperability. Generally, at minimum you will see the following claims in a JWT:

* `iss`: Issuer. This is the principal that issues the JWT
* `sub`: Subject. This is the principal that the JWT is providing information about
* `aud`: Audience. This is the recipient (or recipients) of the JWT
* `exp`: Expiration. This is an integer representing the seconds since Unix Epoch to denote when the token expires. This often includes some leeway for clock drift
* `nbf`: Not Before. This is an integer representing the seconds since Unix Epoch to denote when the token takes effect
* `iat`: Issued At. This is an integer representing the seconds since Unix Epoch to denote when the token

In addition to the registered claims, custom claims can also be created to pass along additional data. In the context of authentication, this is often data such as the scope of access. But it can be any valid JSON key/value pair. We'll make one up for this example, and put everything together as:

```json
{
    "iss": "gigacode",
    "sub": "blogUser",
    "aud": "gitbook",
    "exp": 1695977765,
    "nbf": 1695801647,
    "iat": 1695801647,
    "anyKeyWeWant": "AnyValue"
}
```

Using the same Base64 Web Encoding as the header, our payload is `eyJpc3MiOiAiZ2lnYWNvZGUiLCJzdWIiOiAiYmxvZ1VzZXIiLCJhdWQiOiAiZ2l0Ym9vayIsImV4cCI6IDE2OTU5Nzc3NjUsIm5iZiI6IDE2OTU4MDE2NDcsImlhdCI6IDE2OTU4MDE2NDcsImFueUtleVdlV2FudCI6ICJBbnlWYWx1ZSJ9`

#### The Signature

Finally, we need to create a signature. We'll use HMAC, or Hash-Based Message Authentication Code, to compute a hash using a symmetric secret key `gigacode`. We'll calculate the hash based off our plaintext token (`header.payload`) and encode the result as base64. This gives us a hash of `He2rSRvZpmMJmOgpLApGr3uurTNrKXy922FuOPiinq4`

#### Putting it All Together

With the signature, we're now ready to craft our full token. Separating each component with a `.`, we end up with `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiAiZ2lnYWNvZGUiLCJzdWIiOiAiYmxvZ1VzZXIiLCJhdWQiOiAiZ2l0Ym9vayIsImV4cCI6IDE2OTU5Nzc3NjUsIm5iZiI6IDE2OTU4MDE2NDcsImlhdCI6IDE2OTU4MDE2NDcsImFueUtleVdlV2FudCI6ICJBbnlWYWx1ZSJ9.He2rSRvZpmMJmOgpLApGr3uurTNrKXy922FuOPiinq4`

#### Decoding and Validating

Let's act as the receiving end now, we just got the above JWT and want to know what it means. First, let's take the header and payload portions alone, and then run them through a base64 decoder. We get:

```json
{
  "typ": "JWT",
  "alg": "HS256"
}.{
  "iss": "gigacode",
  "sub": "blogUser",
  "aud": "gitbook",
  "exp": 1695977765,
  "nbf": 1695801647,
  "iat": 1695801647,
  "anyKeyWeWant": "AnyValue"
}
```

Looks good! Now lets make sure nothing got tampered with and the token is valid. Using our secret key `gigacode`, we'll do the same signature generation. This gives us a match! This is a valid token.

### Our Phishing URL

Back to our phishing URL, let's decode this JWT value in the slug. Running through a base64 decoder, we get the (slightly modified for publishing) result of:

```
{
  "alg": "HS256"
}.{
  "aud": "detour",
  "iss": "monolith",
  "sub": "detour_link",
  "iat": 1695659124,
  "nbf": 1695659124,
  "account_id": "[redacted]",
  "trigger_id": "[redacted]",
  "dynamic_url": null,
  "url": "https://login-office365[.]cloud"
}.[Signature]
```

Very interesting! So what does this mean? Let's pick things apart.

#### Header

The header is fairly standard, with an HMAC SHA256 algorithm for the signature specified. It is missing the `typ` parameter, however, while recommended this is listed as optional in the IETF standard

#### Payload

The payload contains the usual audience, issuer, and subject. Interestingly, it also contains values for an issue and not valid before timestamp. These are not required, so it is curious they would be included. It's hard to tell for certain why they are there, whether the attackers are using an off the shelf tool to generate this and its easier just to keep it there or their web server actually utilizes this parameter. Next up though, we have the fun bits.

`account_id` and `trigger_id` are integer values that were different between the email addresses that received the detection. These are likely used so the attackers can correlate users sent the phish to who clicked the link, regardless of if information was entered. This kind of tracking is commonly observed across multiple phishing methods, and often leads to the subject being marked as an easy target for future attacks. In all instances observed, `dynamic_url` remained `null`. This is still very interesting and could indicate the attackers can craft custom redirect URLs for further evasion and/or social engineering. For example, in a spear phishing attack, this may include an organziations name or a url attribute specific to their login system to make the request appear more legitimate. Lastly, we have `url`, which is the URL to redirect the user to.

#### Signature

As we only have a hash to go off, we are unable to derive the secret key. However, it is safe to assume this would be a valid request.

## Investigating the Web Server

### Investigation Defenses

As with many instances of attacker-controlled webservers, some basic steps are taken to prevent investigation. Chiefly, if an attempt is made to directly navigate to the malicious domain, the webpage will simply exit itself. A generic 404 response is returned when any other path is attempted to be accessed. Further, a scan of the IPs linked to the malicious domain names show the service `awselb/2.0` on ports 80 and 443, indicating the domain resolves to an AWS Elastic Load Balancer at the edge. This essentially presents the same challenges as a reverse proxy, as the traffic is routed internally to its resource(s) assigned when the port listener receives traffic at the edge.

