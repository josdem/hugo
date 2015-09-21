+++
date = "2015-09-20T20:17:18-05:00"
draft = true
title = "Security"

+++

## What is OWASP?

* OWASP Open Web Application Secutity Project is a worldwide not-for-profit organization focused on improving the security of software.
* Everyone is free to participate in OWASP and all materials are available under a free and open software licence.
* [OWASP WebSite](www.owasp.org)

**Developing secure software**

* Everything is disable by default
* Security is an design issue
* Keep in your mind security goals
* Security is the key feature
* Only necessary privileges
* Security as layers
* Error management
* Standard cryptography
* Standard libraries

**Password Recovery**

* Password length: Passwords should be at least eight (8) characters long. Combining this length with complexity makes a password difficult to guess and/or brute force.
* Password characters: Should be a combination of alphanumeric characters. Alphanumeric characters consist of letters, numbers, punctuation marks, mathematical and other conventional symbols.
* Passwords should be key sensitive
* Use strong approved cryptographic algorithms: AES, RSA public key cryptography, and SHA-256 or better for hashing.
* Disable accounts after failed authentication attemps
* Do not send username or passwords by email

**Authorization**

* Profiles allow to users to access only granted information
* Profiles are set by authentication
* Process are granted by profiles
* Time out on activities
* Do not send sensitive data in URL

**Common attack types**

* XSS Cross site scripting
* SQL Injection

