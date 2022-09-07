---
title: "Import CA Cert as PrivateKeyEntry - Solve Unable to create enrollment token Error"
date: 2022-09-06 20:23 +0800
author: aold619
description: "Follow this post to solve the ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate."
categories: [Tutorial]
tags: [tutorial, elasticsearch]
---

## Problem

After installed Elasticsearch and configured the security manully according to the docs: [Basic Security](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html) / [Basic Security plus HTTPS](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup-https.html), we may met the error:

> ERROR: Unable to create an enrollment token. Elasticsearch node HTTP layer SSL configuration Keystore doesn't contain any PrivateKey entries where the associated certificate is a CA certificate

The error says that the Keystore for the HTTPS doesn't contain a CA Cert as PrivateKeyEntry type, let's check:

```shell
keytool -keystore config/certs/<filename-http-pkcs12-keystore.p12> -list
```

The results may be as follow:

> ca, Sep 6, 2022, trustedCertEntry
> Certificate fingerprint (SHA-256): 25:27:7D:EE:FE:F6:54:57:47:BE:B5:10:C4:90:DF:28:BF:1B:3B:F9:5E:47:F5:34:5F:03:38:1E:84:0A:23:E7
>
> http, Sep 6, 2022, PrivateKeyEntry
> Certificate fingerprint (SHA-256): D4:EF:60:2C:E5:2D:4C:A8:33:C0:49:44:F4:B5:38:19:92:97:72:CB:5D:85:20:A4:97:9B:90:24:D0:0C:D1:FB

The *TrustedCertEntry* is automatically imported into HTTP PKCS12 Keystore as a CA Cert, that is from the Keystore we generated following [Basic Security](https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html).

But ES need a *PrivateKeyEntry* like:

> http_ca, Sep 6, 2022, PrivateKeyEntry
> Certificate fingerprint (SHA-256): 9E:56:B1:38:F7:49:C7:7D:07:F0:E1:8A:D6:EC:9B:56:DF:86:7D:D6:90:46:86:77:99:6B:D1:9E:9C:4F:F0:03

So let's replace *TrustedCertEntry* with *PrivateKeyEntry*.

---

## Solution

### Step 1. Generate Keystore contains CA Cert and HTTP Keystore

***Note:*** *Usually we have done this step, so if you didn't do anything else to change these Keystores, skip this step.*

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
> Your keystore contains 2 entries
>
> ca, Sep 5, 2022, PrivateKeyEntry
> Certificate fingerprint (SHA-256): 53:F4:9A:9D:56:A9:3A:AF:90:94:41:FA:D7:15:3F:DF:C1:39:AC:BA:FF:12:44:C0:36:4D:15:4C:20:14:1E:3D
>
> http, Sep 5, 2022, PrivateKeyEntry
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

## Others

Offical issue has is tracking the problem. [Generating enrollment token for Kibana should not require the CA key · Issue #89017 · elastic/elasticsearch · GitHub](https://github.com/elastic/elasticsearch/issues/89017)

Here is one Q&A in Chinese. [elasticsearch-create-enrollment-token证书https环境下无法生成口令](https://segmentfault.com/q/1010000041427143/a-1020000042440364)
