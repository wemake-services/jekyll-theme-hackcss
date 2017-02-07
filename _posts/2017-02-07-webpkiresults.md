---
layout: post
title: OpenSSL, WebPKI and Rustls performance in Servo 
date: 2016-02-07 16:48:00
categories: servo
short_description: certificate verification performance results
image_preview: http://www.newdesignfile.com/postpic/2009/11/security-icon_248072.png
---

# Certificate verification

Servo's security infrastructure is heavily dependent on OpenSSL. I created an experiment to compare certificate verification using [WebPKI](https://github.com/briansmith/webpki) and [OpenSSL](https://github.com/sfackler/rust-openssl/). I found it convenient to interact with WebPKI via [Rustls](https://github.com/ctz/rustls).

This is part of an effort to update hyper and rust-openssl in Servo. It's important to note that I'm not changing all of the code away from OpenSSL, just the certificate verification. OpenSSL still manages the connection, which results in some conversion overhead to the appropriate types for rustls/webpki.

In order to change the verification function, you have to modify `set_verify_callback` to point to the desired function when you're creating the SSL object. Then I wrapped different parts of the verification algorithms with timing data:

## Timing Measures

### OpenSSL:
* roots-creation: (subset of http-connector-creation) this is a rustls artifact that I forgot to delete for my openssl test
* verify-hostname: time spent verifying that a certificate is valid for a domain
* http-connector-creation: total time creating the connector object
* matches-wildcard
* matches-dns: 
* openssl-verif-call: measures the total time spent in the openssl verification function
* connection:  measures how long it actually takes to connect (i.e. run `ssl.connect`)
* total-openssl-verify

### Rustls:
* rustls-verify: measures the total time spent in the rustls verification function (either `parallel_verify_server_cert` or `verify_server_cert`)
* chain-creation: measures how long it takes to convert the openssl x509 chain to the rustls certificate chain
* roots-creation: (subset of http-connector-creation) time required to create the rustls root certificate store
* http-connector-creation: total time creating the connector object
* connection: measures how long it actually takes to connect (i.e. run `ssl.connect`)
* total-rustls-verify time: measures total time performing all steps required for rustls verification

All values are averaged over the number of times a method was called while opening a site.

There are two versions of Rustls that I tested on--one does all verification steps in serial, while the other does them all in parallel. Parallel rustls still has room for improvement-- for more information, see [webpki](https://github.com/briansmith/webpki/issues/37).

## Sites

The sites I picked for testing were the 8 featured on servo's front page, plus a few I frequently use. Most of them use TLS ECDHE RSA with AES 128 GCM SHA256 and 128 bit keys, except those noted below

* [duck duck go](https://duckduckgo.com)
* [github](https://github.com)
* [wikipedia](https://wikipedia.com)
  * TLS ECDHE ECDSA with CHACHA20 POLY1305 SHA256, 256 bit keys
* [ars technica](https://arstechnica.com/)
* [reddit](https://reddit.com)
* [rust](https://rustlang.org)
* [servo](https://servo.org)
  * TLS AES 128 GCM SHA256 TLS 1.3
* [hacker news](https://news.ycombinator.com/)
* [twitter](https://twitter.com)
* [cnn](https://cnn.com)
  * no encryption
* [google](https://google.co.uk)
* [washington post](https://washingtonpost.com)
* [stack overflow](https://stackoverflow.com)

# Results

Times are shown in ms. I'm not sure what happened with hackernews-rustlsserial, and I'll try to get new numbers for it.

I compared results for total connection time and total verification time. For total connection time, openssl was fastest for 5/12 (excluding hackernews) sites, rustlsserial was fastest for 4, and rustlsparallel for the remaining 3. For overall verification time, openssl was fastest for 11 sites, and rustlsparallel was fastest on washingtonpost.com. 

It's interesting that openssl was almost uniformly quicker at just performing verification, but not for establishing a connection. I'm not sure why this would be. If anyone has ideas, let me know :)

site | method | roots creation | http connector creation | openssl/rustls verify call | total connection time | total verification time
--------------------|--------|-------|------|------|-------|------
arstechnica | openssl | 6.5937645 | 3.06476595 | 0.04212679 | 7.63642713 | 0.11863110
arstechnica | rustlsparallel | 0.6452306 | 1.8578739 | 0.9613585 | 10.6051663 | 0.9679321
arstechnica | rustlsserial | 1.7092342 | 4.8119877 | 0.8465297 | 17.3814024 | 0.8523814
cnn | openssl | 0.5902370 | 3.33709690 | 0.00156567 | 5.87286853 | 0.00410566
cnn | rustlsparallel | 0.4306289 | 2.5159100 | 0.3190672 | 9.7441715 | 0.3259468
cnn | rustlsserial | 0.5596800 | 2.3019310 | 0.2837703 | 6.2241893 | 0.2915407
duckduckgo | openssl | 0.5305152 | 2.91319830 | 0.00221078 | 7.69686665 | 0.00550160
duckduckgo | rustlsparallel | 0.4963679 | 3.2570464 | 0.4329721 | 6.4144129 | 0.4383328
duckduckgo | rustlsserial | 0.5634392 | 2.5714297 | 2.1426261 | 5.4416704 | 0.2206577
github | openssl | 0.5259603 | 2.41714400 | 0.00726623 | 10.90148742 | 0.01794724
github | rustlsparallel | 0.4670057 | 2.2619585 | 5.4297168 | 14.1635926 | 5.4348146
github | rustlsserial | 0.4736196 | 2.9527882 | 0.5425635 | 20.3668883 | 0.5584827
google | openssl | 0.4657012 | 3.11720415 | 0.00158668 | 5.99938325 | 0.00500675
google | rustlsparallel | 0.4081303 | 2.2226142 | 0.2490465 | 4.9481365 | 0.2554819
google | rustlsserial | 0.7276384 | 2.8976474 | 0.2454053 | 4.8469683 | 0.2522257
hackernews | openssl | 0.5978065 | 2.74829625 | 0.00170511 | 4.20313573 | 0.00576318
hackernews | rustlsparallel | 0.4728535 | 2.7132263 | 0.6241498 | 4.1972928 | 0.6394845
hackernews | rustlsserial | 2.3374316 | 0.4007136 |  |  | |
reddit | openssl | 0.3964899 | 2.13715120 | 0.08706821 | 7.40169574 | 0.21397945
reddit | rustlsparallel | 0.5623264 | 2.4189925 | 1.9196810 | 7.1750911 | 1.9263490
reddit | rustlsserial | 0.5554853 | 2.4069789 | 0.3140017 | 5.6085587 | 0.3186888
rust | openssl | 0.4763279 | 2.50936690 | 0.00071792 | 5.00488759 | 0.00246496
rust | rustlsparallel | 0.5137190 | 2.8609235 | 0.8248997 | 6.2613387 | i0.8308361
rust | rustlsserial | 0.7270381 | 2.8080940 | 0.7364159 | 5.9658525 | 0.7529244
servo | openssl | 0.4126936 | 2.46113245 | 0.01265213 | 3.54343740 | 0.04969500
servo | rustlsparallel | 0.3729247 | 2.2147630 | 0.4230856 | 4.2786723 | 0.4297226
servo | rustlsserial | 0.4271293 | 2.6142965 | 0.5498572 | 5.1212375 | 0.5624017
stack overflow | openssl | 0.4872833 | 2.54147045 | 0.00214393 | 6.69359618 | 0.00602048
stackoverflow | rustlsparallel | 0.6955762 | 2.4821835 | 0.3383331 | 4.6982153 | 0.3459493
stackoverflow | rustlsserial | 0.6805226 | 2.6676572 | 0.3023453 | 5.4262767 | 0.3090781
twitter | openssl | 0.5633570 | 3.06855305 | 0.00996953 | 25.13172682 | 0.02554954
twitter | rustlsparallel | 0.4326043 | 2.1039143 | 2.9250449 | 16.1298042 | 2.9300035
twitter | rustlsserial | 0.5732324 | 2.8023081 | 4.0729707 | 14.1060498 | 4.0807530
wapo | openssl | 5.2944900 | 2.82199855 | 0.18614084 | 8.39432315 | 0.63647240
wapo | rustlsparallel | 0.3269752 | 2.2430216 | 0.5247874 | 6.2113378 | 0.5317807
wapo | rustlsserial | 0.5703944 | 3.0341653 | 0.5972499 | 7.0022414 | 0.6031256
wikipedia | openssl | 0.5330987 | 2.39788525 | 0.00302343 | 5.13799420 | 0.00806534
wikipedia | rustlsparallel | 0.4611093 | 2.5344541 | 0.4680453 | 4.9266177 | 0.4759743
wikipedia | rustlsserial | 0.4883241 | 1.9458110 | 0.4012761 | 4.9726309 | 0.4095681

# Next steps

1. webpki benchmarks [ECDSA](https://github.com/briansmith/crypto-bench/issues/25) [RSA](https://github.com/briansmith/crypto-bench/issues/24)
2. parallel signature verification [webpki](https://github.com/briansmith/webpki/issues/37)
3. Improve test set and method (this started out as an adhoc measurement, but now seems like it could yield useful data as development continues)


