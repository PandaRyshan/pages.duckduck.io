---
layout: post
title: Add PrivateKey entry to HTTP SSL Keystore - Solve ES Unable generate token Error
date: 2022-09-06 20:23 +0800
author: aold619
description: Follow this post to solve the ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate
image:
category: ["Tutorial"]
tags: ["toturial", "elasticsearch", "security"]
published: true
sitemap: false
---

## Problem

Yesterday I installed Elasticsearch for the first time, and after I configured the security manully according to the docs: [Basic Security](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html) / [Basic Security plus HTTPS](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup-https.html) / [Update certs with different CA](https://www.elastic.co/guide/en/elasticsearch/reference/master/update-node-certs-different.html), I try to generate token for Kibana, and I met the error:

> ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate

I searched some pages and didn't get the answer, then I found a [discussion](https://stackoverflow.com/questions/24974324/import-certificate-as-privatekeyentry) on StackOverflow, and it fixed my problem. I think maybe I should share my experience here with those newbies like me.

If I put the discussion in the wrong category, please help me move it. Thanks.

---

## Solution

### Step 1. Generate CA cert and Key

From the doc [Update certs with different CA](https://www.elastic.co/guide/en/elasticsearch/reference/master/update-node-certs-different.html), to add a CA PrivateKey Entry to my HTTP PKCS cert, I need to generate it with `bin/elasticsearch-certutil`, but if you get it from `ZeroSSL` or `Let's Encrypt` or your organization, then skip the step:

```shell
./bin/elasticsearch-certutil ca --pem
```
Enter a name for the compressed output file that will contain your certificate and key, or accept the default name of `elastic-stack-ca.zip`. Unzip the output file. The resulting directory contains a CA certificate (`ca.crt`) and a private key (`ca.key`).

### Step 2. Bundle the PrivateKey and add it to HTTP PKCS

Then we should generate (.p12) bundle file from the key and CA cert:

```shell
openssl pkcs12 -export -in <filename-certificate> -inkey <filename-key> -name <privatekey-entry-name> -out <filename-new-PKCS-12.p12>
```

Now, just add the bundle file (.p12 file) to a keystore by executing the following command:

```shell
keytool -importkeystore -destkeystore <filename-your-http-PKCS-12> -srckeystore <filename-new-PKCS-12.p12> -srcstoretype PKCS12
```

## Check the result

And you can use following command to check:

```shell
keytool -keystore <filename-your-HTTP-PKCS-12.p12> -list
```

The result should contains two PrivateKey entries like:

> Keystore type: PKCS12
> Keystore provider: SUN
>
> Your keystore contains 2 entries
>
> http, Sep 5, 2022, PrivateKeyEntry,
> Certificate fingerprint (SHA-256): EF:8D:78:EC:F0:C5:97:1B:7B:58:EF:5F:E3:73:A5:D0:7E:1B:FE:B3:75:B0:B4:D9:CB:80:FC:B3:8E:5D:A5:74
> http_ca, Sep 5, 2022, PrivateKeyEntry,
> Certificate fingerprint (SHA-256): 53:F4:9A:9D:56:A9:3A:AF:90:94:41:FA:D7:15:3F:DF:C1:39:AC:BA:FF:12:44:C0:36:4D:15:4C:20:14:1E:3D

It doesn't matter if you have a TrustedCert entry, that's what you got from the [Basic Security plus HTTPS](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup-https.html) doc before.

Last, before using `bin/elasticsearch-create-enrollment-token` to generate a new token, make sure you added the password for your private key to the secure settings in Elasticsearch.

```shell
./bin/elasticsearch-keystore [show|add] xpack.security.http.ssl.keystore.secure_password
```

## Re-generate token

Now start the node, and re-generate your token:

```shell
./bin/elasticsearch-create-enrollment-token -s [kibana|node]
```

---

[This post in Chinese](https://segmentfault.com/q/1010000041427143)
