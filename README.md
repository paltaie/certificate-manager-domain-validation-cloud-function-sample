# Certificate Order - DNS Domain Validation sample
## General
IBM Cloud Certificate Manager lets you order a TLS certificate signed by Let’s Encrypt. When you order a certificate, you have to prove that you own the domains for which you are requesting a certificate. This repository contains sample code to show you how to prove you own these domains.

The sample demonstrates how to implement an IBM Cloud Function action that receives a domain validation challenge from Certificate Manager, and answers this challenge. 


When you order a certificate from Certificate Manager, the Cloud Function action receives a notification from Certificate Manager containing a DNS txt record challenge, and then adds this to the DNS provider where the requested domain is registered. Once the domain validation challenge is answered, the Cloud Function also clears this TXT record from the DNS service.

In this sample we used IBM Cloud Internet Services as our DNS provider, where we registered our domains, but you can use this sample with any other DNS provider that offers an API for updating TXT records.

## Prerequisites
 
1. An instance of [IBM Cloud Certificate Manager.](https://cloud.ibm.com/docs/services/certificate-manager)
2. An instance of [IBM Cloud Internet Services.](https://cloud.ibm.com/docs/infrastructure/cis) with a configured domain

## Configuration
### IBM Cloud Function action
1. Create a new [IBM Cloud Function action](https://cloud.ibm.com/docs/openwhisk/index.html#openwhisk_start_hello_world)
2. [Bind parameters to the action](https://cloud.ibm.com/docs/openwhisk/parameters.html#default-params-action) 
    1. `iamApiKey` - an API key with `manager` role on the specific domain on CIS for the `Reliability` functional scope.
    
        * Create an API key in your IBM Cloud account -> Manage -> Access (IAM) -> IBM Cloud API keys
    2. `allowedCertificateManagerCRNs` - a JSON Object containing a list of Certificate Manager instances that are allowed to invoke this function.
        Apply it in order to protect your cloud function public api from being invoked by unauthorized clients.  
        E.g. `{"CRN1":true,"CRN2":true}` 
        
        * Find your Certificate Manager instance CRN from the Notifications side bar menu -> Settings tab
        * Or from CLI: `ibmcloud resource service-instance [INSTANCE NAME]`, grab the `ID` value.

### IBM Cloud Certificate Manager
1. Setup a Callback URL [Notification Channel](https://cloud.ibm.com/docs/services/certificate-manager?topic=certificate-manager-configuring-notifications#callback) in your Certificate Manager instance.

### Modify and deploy the sample
In **main.js**:
1. Update the `cisCrn` parameter with your CIS instance CRN value
2. Update the `certificateManagerApiUrl` parameter with your Certificate Manager service instance region value. Can be one of: `us-south`, `eu-gb`, `eu-de`, `jp-tok`
3. Deploy the code to your IBM Cloud Function action
 
## Test
1. [Order a Lets Encrypt certificate](https://cloud.ibm.com/docs/services/certificate-manager?topic=certificate-manager-managing-certificates-from-the-dashboard#importing-a-certificate) using the Certificate Manager console
2. Wait for the certificate to be in "Valid" state

## Q&A
**Q: My order status is in `pending` state for a long time**   
**A:** On slow DNS networks an order may take up to about 20 minutes to complete successfully.

**Q: Which response code should I return from my cloud function to Certificate Manager?**  
**A:** Return code should be **200 OK**.

**Q: For how long will Certificate Manager try to validate the domain?**  
**A:** After sending the domain validation challenge, Certificate Manager will try to validate the domain for up to 10 minutes.

**Q: How do I check the certificate order status, using the Certificate Manager public API?**  
**A:** Use the `Get certificate metadata` API to poll the certificate order status.

**Q: Can I be notified once my order is complete?**  
**A:** You can [configure a notification channel](https://cloud.ibm.com/docs/services/certificate-manager?topic=certificate-manager-configuring-notifications#configuring-notifications) to be notified when a certificate is issued, or if your order failed.

**Q: My order failed. Why?**  
**A:** You can find an error code and error message in the certificate metadata, UI, and in the notification you received when your order failed.

**Q: How can I troubleshoot errors in the execution of my cloud function?**  
**A:** You can review the Cloud Function log to find any errors that occurred during execution.

