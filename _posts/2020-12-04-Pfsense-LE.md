---
title: Letsencrypt Certificate Management with Pfsense
date: 2020-12-04 12:00:00 -500
categories: [Pfsense]
tags: [pfsense,letsencrypt,certificates]
---

## Background

I have been using Letsencrypt from quite some now for all my testing purpose. It has come a long way now and I am sure many organizations have been using it as well.
The Letsencrypt cert has to renewed every three months. In the past I use to manually create certificates using acme client and in combination with bash script I would automate the renewal. 

I do now run a dedicated pfsense box and there is an __*ACME*__ package available for pfsense and hence it becomes an obvious choice to handle the certificate generation, validation, and renewal processes for my lab servers and testing needs, whether they are internal or external. This helps me to secure my external facing servers but also for my internal servers I can get rid of self-sign certs.Below is what we are trying to achieve as per this guide. This one only covers lets encrypt part, i'd cover HA proxy in next blog.

![Pfsense with Letsencrypt & HA-Proxy](https://techboyaviblogpictures.blob.core.windows.net/pfsense/LE-PFSense.PNG)

## Installation of ACME Package

In order to configure letsencrypt to issue certificate we need an ACME package installed on the pfsense.

To install the acme package go pfsense package manager.

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/Acme-Package-01.png)

look for acme package, ideally it should be on the top of the list. Few simple clicks to insatll the same.
![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/acmepackage.PNG)

Once the package is installed, you find a new tab under the services section which then can be used to configure the letsencrypt service.

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/acmecert.PNG)


## ACME account key setup

Now that we have the ACME package installed, we need to set up the account keys which eventually will help us to validate the account against Letsencrypt servers.

We would need to set up two account keys, one for staging enviornment and another one for production enviornment.

To set up the keys, go to Services -> ACME Certificates -> Account Keys -> Add

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/accountkeys.PNG)

now fill in the details to generate the keys. I'd start with staging first.

	• Name : Staging 
	• Description: Staging
	• ACME Server: Let's Encrypt Staging ACME v2(for Testing Purpose)
	• Email Address : needtobevalidemail@email.com
	• Create Account Key - 
	• Register ACME entered key

  finally click on __*save*__ and you're done.

  ![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/account-key-staging.PNG)

  Repeat the same process for production keys however change the ACME server to production ACME server. The final output should look like below:
  ![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/accntkeyfinal.PNG)

Now we can generate the certificates for production and staging environment however before that we need to make sure we can do the domain validation.

## Domain Validation

The certificates issues by Let’s Encrypt are domain validated.Effectivly, this validation ensures that the system requesting the certificate has authority over the domain for which certificate is requested. This validation can be performed in a number of ways, such as by proving ownership of the domain’s DNS records or hosting a file on a web server for the domain. In my case I would be using domains DNS to validate.

My Domain registrar is *Godaddy* and to validate the domain ownership we need to create/provide API keys from the developer portal *<https://developer.godaddy.com>*

I created two keys, one offcourse for staging and another one for production.

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/keys.PNG)

## Certificate Generation and Renewal

Now that we have the api keys for the domain registrar, we will add our first certificate.

__Services -> ACME Certificates -> Add__

I am going to create the wildcrad certificate that will suffice my requirements.Here it is important to ensure that have both the naked domain as well *.domain name mentioned while generating the wild card certificate. Also, ensure that *.domain is on the top of the list.

I have also configured DNS sleep time for 120 seconds and *auto renewal to 75 days*. This gives me enough time to troubleshoot in case of issues.

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/cert_gen-1.PNG)

We should now create the certificates. If all the rules are setup properly, DNS validation will happen using the API keys we have provided. ACME clinet will add a temporary record in registars DNS in my case go-daddy.

It is important that we first create staging cert because production certificates are rate limited. Hence all testing should be done against staging server and once everything is in place we should flip the same to production server.

## Certificate export

If we need to export the certificate, we can easily do so by going to certificate manager. This is not required s we will be placing HA proxy in front of our enviornment, however there are times when we really need an export.

Go to system -> Certificate Manager -> certificates.

![](https://techboyaviblogpictures.blob.core.windows.net/pfsense/certs.png)

## Conclusion

To setup letsencrypt client is straight forward task. Please not that I did open port 80 and port 443 on router in order for Pfsense to reach the go-daddy and get the domain validation working.
Also, just because we now have a valid certificate available does not really mean that we should start to expose all our services externally! 





