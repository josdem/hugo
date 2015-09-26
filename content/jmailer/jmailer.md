+++
date = "2015-09-21T20:43:10-05:00"
draft = true
title = "Jmailer"

+++
It is a service that allow to your clients contact you by email Jmailer offers you:

* Never store any information from your clients, Jmailer only deliver your client messages
* Your clients can deliver emails from:
  * Your website
  * Your Android application
  * Your Stand alone application
* Jmailer is secure, we deliver to you a credentials in order to send emails
* You can deliver as many emails as Gmail supports

**Example: Sending emails from command line using curl**

To get a token:

```bash
curl -X -v -d 'username=YourUsername&password=YourPassword&client_id=JmailerClientId&client_secret=JmailerSecret&grant_type=password' -X POST "http://jmailer.josdem.io/jmailer/oauth/token"
```

Response:

```bash
{"access_token":"d7e25d9d-9922-46ea-85ca-9f4bcc28f0ac","token_type":"bearer","refresh_token":"e1c9d922-b4d9-4162-b797-ac26e8e6fd33","expires_in":59,"scope":"read,write"}
```

Where your access_token is your token to access Jmailer service.
Send an email using your access token:

```bash
curl -H "Content-Type: application/json" -H "Authorization: Bearer d7e25d9d-9922-46ea-85ca-9f4bcc28f0ac" -X POST -d '{"email":"joseluis.delacruz@josdem.io","message":"Hello from josdem.io!"}' 'http://jmailer.josdem.io/jmailer/services/emailer/message'
```

If you want to know more, please contact me in the [Home](/) section.

