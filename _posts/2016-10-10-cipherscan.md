---
layout: post
title: Cipherscan 
date: 2016-10-10 18:34:00
categories: servo
short_description: cipherscan results and discussion
image_preview: http://ih0.redbubble.net/image.10706037.1609/flat,800x800,070,f.u2.jpg
---

# Cipherscan
During a dev-servo discussion about the merits of various TLS libraries, the question of web compatibility came up--what cryptographic protocols does Servo need to support in order to serve most of the web?

I decided to run [cipherscan](https://github.com/mozilla/cipherscan) and get some updated results.

# Results
You can see the full results [here](http://pastebin.com/aCazxpAq), but I'll highlight what I consider the most interesting in this case.

## Ciphers

Supported Ciphers        | Count     |Percent
-------------------------|---------|-------
3DES                      |472578    |87.0229
3DES Only                 |491       |0.0904
3DES Preferred            |1450      |0.267
3DES forced in TLS1.1+    |854       |0.1573
AES                       |539545    |99.3546
AES Only                  |49098     |9.0412
AES-CBC                   |539101    |99.2728
AES-CBC Only              |4403      |0.8108
AES-GCM                   |458011    |84.3405
AES-GCM Only              |405       |0.0746
CAMELLIA                  |245146    |45.1424
CHACHA20                  |68015     |12.5246
CHACHA20 Only             |1         |0.0002
Insecure                  |42351     |7.7987
RC4                       |131983    |24.304
RC4 Only                  |111       |0.0204
RC4 Preferred             |11067     |2.0379
RC4 forced in TLS1.1+     |6031      |1.1106

## Protocols

Supported Protocols       |Count    | Percent
-------------------------|---------|-------
SSL2                      |12822     |2.3611
SSL2 Only                 |10        |0.0018
SSL3                      |76905     |14.1617
SSL3 Only                 |272       |0.0501
SSL3 or TLS1 Only         |46262     |8.5189
SSL3 or lower Only        |278       |0.0512
TLS1                      |530593    |97.7061
TLS1 Only                 |26478     |4.8758
TLS1 or lower Only        |58211     |10.7193
TLS1.1                    |476556    |87.7555
TLS1.1 Only               |40        |0.0074
TLS1.1 or up Only         |12044     |2.2178
TLS1.2                    |483237    |88.9857
TLS1.2 Only               |3365      |0.6196
TLS1.2, 1.0 but not 1.1   |4564      |0.8404

Supported Handshakes      |Count     |Percent
-------------------------|---------|-------
ADH                       |781       |0.1438
AECDH                     |8790      |1.6186
DHE                       |302890    |55.7757
ECDHE                     |476401    |87.7269
ECDHE and DHE             |263183    |48.4639
RSA                       |472454    |87.0001


## Certificates

Certificate sig alg     |Count     |Percent 
-------------------------|---------|--------
None                      |9328      |1.7177   
ecdsa-with-SHA256         |47343     |8.718    
sha1WithRSAEncryption     |14180     |2.6112   
sha256WithRSAEncryption   |495204    |91.1894  
sha384WithRSAEncryption   |7         |0.0013   
sha512WithRSAEncryption   |62        |0.0114 

OCSP stapling             |Count     |Percent 
-------------------------|---------|--------
Supported                 |132214    |24.3466  
Unsupported               |410836    |75.6534

# Discussion

I initially compiled these stats because the Servo community brought up the possibility of using [rustls](https://github.com/ctz/rustls)() in Servo. Currently, rustls only supports TLS1.2, so it seemed like a good time to get some updated cipherscan results. Over 10% of the sites surveyed only supported TLS1.1 or lower. In a production browser, this would be way too high.

Mostly, I think these numbers can be useful for prioritization right now.

