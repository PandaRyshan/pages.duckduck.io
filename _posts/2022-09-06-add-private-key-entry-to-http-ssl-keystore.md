---
title: "Add PrivateKey to HTTP PKCS - Unable to create enrollment token Error"
date: 2022-09-06 20:23 +0800
author: aold619
description: "Follow this post to solve the ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate."
categories: [Tutorial]
tags: [tutorial, elasticsearch]
---

## Problem

Yesterday I installed Elasticsearch for the first time, and after I configured the security manully according to the docs: [Basic Security](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html) / [Basic Security plus HTTPS](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup-https.html), I try to generate token for Kibana, and I met the error:

> ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate

I searched some pages and didn't get the answer, then I found a [discussion](https://stackoverflow.com/questions/24974324/import-certificate-as-privatekeyentry) on StackOverflow, and it solved my problem. I think maybe I should share my experience with those newbies like me.

---

## Solution

### Step 1. Generate Keystore contains CA Cert and HTTP Keystore [optional]

***Note:*** *Usually we have done this step, so if you didn't do anything else to try to change these Keystores, skip this step.*

Following [Basic Security](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html) we will get a PKCS12 Keystore containing CA Cert, default filename is **elastic-stack-ca.p12**.

Following [Basic Security plus HTTPS](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup-https.html), we will get a PKCS12 Keystore contains a *TrustedCertEntry* that import from **elastic-stack-ca.p12**, the default filename of HTTP Keystore is **http.p12**.

### Step 2. Import CA Cert Keystore to HTTP Keystore as PrivateKey type

```shell
keytool -importkeystore -destkeystore <filename-http-PKCS12> -srckeystore <filename-PKCS12-contains-CA-Cert.p12> -srcstoretype PKCS12
```

This step changes the CA Cert that was automatically imported into the original HTTP Keystore as TrustCertEntry to PrivateKeyEntry, and this is exactly what that ERROR tells us to do.

> Importing keystore config/certs/es-ca.p12 to config/certs/http.p12...
> Enter destination keystore password:
> Enter source keystore password:
> Existing entry alias ca exists, overwrite? [no]: yes
> Entry for alias ca successfully imported.
> Import command completed: 1 entries successfully imported, 0 entries failed or cancelled

## Check the result

And you can use following command to check:

```shell
keytool -keystore <filename-HTTP-PKCS12.p12> -list
```

The result should contains two PrivateKey entries like:

> Keystore type: PKCS12
> Keystore provider: SUN
>
> Your keystore contains 2 entries
>
> ca, Sep 5, 2022, PrivateKeyEntry,
> Certificate fingerprint (SHA-256): 53:F4:9A:9D:56:A9:3A:AF:90:94:41:FA:D7:15:3F:DF:C1:39:AC:BA:FF:12:44:C0:36:4D:15:4C:20:14:1E:3D
> http, Sep 5, 2022, PrivateKeyEntry,
> Certificate fingerprint (SHA-256): EF:8D:78:EC:F0:C5:97:1B:7B:58:EF:5F:E3:73:A5:D0:7E:1B:FE:B3:75:B0:B4:D9:CB:80:FC:B3:8E:5D:A5:74


Before generating a new token, make sure you added the password for your private key to the security settings in Elasticsearch.

```shell
./bin/elasticsearch-keystore [show|add] xpack.security.http.ssl.keystore.secure_password
```

## Re-Generate token

Now start the node, and re-generate your token:

```shell
./bin/elasticsearch-create-enrollment-token -s [kibana|node]
```

---

[Here is an issue that track the problem.](https://github.com/elastic/elasticsearch/issues/89017)
[This is same content post on elastic discuss.](https://discuss.elastic.co/t/import-ca-cert-as-privatekeyentry-to-http-keystore-solve-unable-to-create-enrollment-token-error/)
[This is same content Q&A in Chinese.](https://segmentfault.com/q/1010000041427143/a-1020000042440364)

