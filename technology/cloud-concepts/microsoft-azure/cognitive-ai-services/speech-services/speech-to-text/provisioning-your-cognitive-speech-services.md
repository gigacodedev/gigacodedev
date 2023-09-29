# Provisioning your Cognitive Speech Services

Before we start making use of these services, we need to provision them first! This is a relatively quick process:

1. In your Azure Portal, navigate to the Resource Group that will be housing your cognitive services API deployment
2.  Select the Create button\


    <div align="left">

    <figure><img src="../../../../../../.gitbook/assets/image (17).png" alt="" width="563"><figcaption></figcaption></figure>

    </div>
3. Search for "Speech" and proceed to create the resource\
   ![](<../../../../../../.gitbook/assets/image (2) (1).png>)
4.  Set the deployment options\


    <figure><img src="../../../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

    **Be mindful of the pricing tier. The free SKU does **_**not**_** currently support batch transcriptions. While the standard S0 tier will cost you some money, the rate for speech transcription is currently about $0.0003 per second, or $0.016 per minute, which should be pretty negligible for our usage**
5. Proceed directly to the Review + Create screen and provision the resource
