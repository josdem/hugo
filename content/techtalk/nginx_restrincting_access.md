+++
date = "2016-06-21T17:41:09-05:00"
draft = true
title = "Nginx Restrincting Access"

+++

Access can be denied by using HTTP basic authentication. To enable authentication, use the `auth_basic directive`. Users then must enter their valid username and password to get access to the website

```
server {
    ...
    auth_basic "closed website";
    auth_basic_user_file /etc/nginx/htpasswd;
}
```

You must generate that htpasswd file, as follow:

```
sudo htpasswd -c htpasswd ${username}
```

Next the prompt will ask for a new password, as soon as we done this step, a new htpasswd file will be generated with the username and encrypted password.

```
nginxUser:$apr1$TthgC50M$XNdndu.cVIQh6/wGQf.dV0
```

[Return to the main article](/techtalk/sysadmin)

