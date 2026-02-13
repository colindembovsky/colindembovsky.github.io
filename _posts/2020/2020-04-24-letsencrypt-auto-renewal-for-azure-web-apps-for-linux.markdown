---
layout: post
title: LetsEncrypt Auto-Renewal For Azure Web Apps for Linux
date: '2020-04-24 22:34:56'
description: >
  In this post I show how I achieved automated LetsEncrypt cert registration and renewal for Azure Web Apps for Linux using nginx and CertBot.
tags:
- azure
---

1. TOC
{:toc}

This is my first post after converting my blog to Ghost. There are dozens of posts from all sorts of people about how they adopted/migrated to Ghost. I had some interesting challenges to get my site going, which I will post about. One of them was SSL security.

<!--kg-card-begin: markdown-->

My previous blog engine was a fork of [MiniBlog](https://github.com/madskristensen/MiniBlog) by Mads Kristensen. I customized it because I was previously on Blogger (remember that?) and had to import from Blogger. I also added Azure Storage rather than using file system and a search function. I was running the .NET framework version (which is fairly old) and was using Windows Live Writer (or now Open Live Writer) to author. Being able to author in markdown was the primary driver for me getting to Ghost!

<!--kg-card-end: markdown-->
## tl;dr

If you just want to jump straight to the code, head to the repo [here](https://github.com/colindembovsky/webapp-linux-letsencrypt-auto-renew). There's a detailed readme with instructions.

<!--kg-card-begin: markdown-->
## Ghost
<!--kg-card-end: markdown-->

There is a [Ghost docker image](https://hub.docker.com/_/ghost) that is stupid simple to use to get Ghost up and running. I won't bore you with how I converted my posts from my old blog, but extracting an import of all my old content was manageable. I was now ready to run this sucker live!

That's when I hit my first big snag - I wanted to enforce SSL (of course). No problem - I can just install the [LetsEncrypt Azure Web App extension](https://github.com/sjkp/letsencrypt-siteextension) and I'd be good to go, right? Wrong - _Web Apps for Linux can't have extensions!!_

No problem - I'll just run an [nginx](https://www.nginx.com/) side-car container reverse proxy using [multi-containers](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-multi-container-app) and let nginx handle the SSL termination. Except I could not get that to work.

I found this [great post](https://jessicadeen.com/how-to-manually-setup-a-lets-encrypt-ssl-cert-for-azure-web-app-with-linux/) by Jessica Deen on how to use SSL on Azure Linux Web Apps (coincidentally she was doing this for her Ghost blog!). While this looked promising, I didn't want to have to manually renew the cert every 90 days!

<!--kg-card-begin: markdown-->
## CertBot
<!--kg-card-end: markdown-->

I scrathed around and found a Docker image for registering (and renewing) certs called [CertBot](https://hub.docker.com/r/certbot/certbot/). I tried running this with nginx like [this post](https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71). It was exactly what I was trying to do - except that the cert magic happend outside the images and docker-compose!

Eventually it dawned on me - I could combine both approaches. To automate certificate registration using CertBot, CertBot issues a request to LetsEncrypt and listens for a HTTP request from LetsEncrypt (to the CDN you're registering). If it receives the call, it knows you're making the request from a domain you own and the cert is issued. So I'd need to route the challenge request to the certbot container. Of course all other calls needed to be routed to my app container.

After registration (or renewal) there's a hook for executing a script. So I could use some of Jessica's `az cli` code to register the cert to the web app! I could then just loop CertBot, checking for renewals. When a renewal is performed, the same hook could register the new cert for me - voila, automated cert renewal with LetsEncrypt!

## The Solution

Let's start with the `yml` file that describes the containers I spin up in my multi-container app:

~~~yaml
{% raw %}
    version: '3.3'
    
    services:
      app: # this name should be the value for APP_CONTAINER_NAME in the nginx config
        image: myregistry/myapp:1.0.0 # registry for your application image
        ports: 
        - "2368:2368" # port your app listens on (the EXPOSE port); the value for APP_EXPOSE_PORT in the nginx config
        restart: always
    
      nginx:
        depends_on:
        - app
        image: myregistry/my-nginx:latest # registry for your custom nginx with the nginx config
        ports:
        - "0:80" # must be this mapping to route all traffic to the web app to nginx
        restart: always
    
      certbot:
        depends_on:
        - nginx
        image: myregistry/my-certbot:latest # registry for your custom certbot image
        ports:
        - "80:80" # must be this mapping to respond to LetsEncrypt challenge
        restart: always
        volumes:
        - ${WEBAPP_STORAGE_HOME}/certbot/letsencrypt:/etc/letsencrypt # maps to persistent storage
{% endraw %}

Notes:

- There are 3 containers: `app`, `nginx` and `certbot` (the names are important for the nginx config file)
- The port mapping is important - nginx _must_ be on `0:80` so that it gets all traffic inbound from the web app. Certbot must be on `80:80` to correctly respond to the LetsEncrypt challenge. Finally, the app port should _not_ be 80 or 8080 - I could not get this to work if the app was using either of these ports.
- The `certbot` image is mapping a volume for the `/etc/letsencrypt` folder - this is required to retain the cert if the container restarts; otherwise certbot will request a cert every time it starts, which isn't what we want.

Let's now look at the `nginx.conf` file:

~~~nginx
{% raw %}
    user nginx;
    worker_processes 1;
    
    error_log /var/log/nginx/error.log warn;
    pid /var/run/nginx.pid;
    
    
    events {
        worker_connections 1024;
    }
    
    http {
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
    
        log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
    
        access_log /var/log/nginx/access.log main;
        client_max_body_size 10M;
    
        sendfile on;
        #tcp_nopush on;
    
        keepalive_timeout 65;
    
        #gzip on;
     
        server {
            listen 80 default_server;
            listen [::]:80 default_server;
    
            # certbot challenge
            location ~ /.well-known {
                proxy_pass http://certbot;
                proxy_redirect off;
            }
    
            location / {
                # APP_EXPOSE_PORT is the port the app container exposes    
                proxy_pass http://app:APP_EXPOSE_PORT;  
                proxy_set_header Host $host;
            }
        }
    }
{% endraw %}
~~~

Notes:

- The server listens on port 80 (the SSL termination occurs at the Web App layer, so traffic coming in at this point is http)
- The location `~ /.well-known` routes any route with `/.well-known` in the URL to certbot
- Location `/` forwards all other requests to the app container - make sure you update this to match your app port. In my example compose file above, this would be 2368.

**Note:** I spent many hours debugging an infinte loop of redirects - I found that I had to ensure that none of the directives below were specified in the location rules. This is something to do with how Azure Web Apps handles incoming traffic.

~~~nginx
{% raw %}
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
{% endraw %}
~~~

## CertBot Customization

To customize CertBot to handle certificate registration and renewal, I customized the `CMD` for the container to invoke this script:

~~~bash
{% raw %}
    #!/bin/sh
    
    rsa_key_size=4096
    if [-z $STAGING] || [$STAGING != "0"]; then staging_arg="--staging"; fi
    
    if [-z $EMAIL] || [-z $CDN]; then
      echo "Please set email and CDN environment variables!"
    else
        wwwArg=""
        if [-z $WWW] || [$WWW != "0"]; then
          echo "Adding www.$CDN to registration"
          wwwArg="-d www.$CDN" 
        fi
    
        if [! -f "$WORKING_PATH/live/$CDN/fullchain.pem"]; then
          echo "Creating cert"
          echo "Staging arg: $STAGING"
    
          certbot certonly --standalone \
            --preferred-challenges=http \
            --email $EMAIL \
            $staging_arg \
            --agree-tos \
            --no-eff-email \
            --manual-public-ip-logging-ok \
            --domain $CDN $wwwArg
          
          # run the script to register the cert with web apps
          deploy-cert-az-webapp.sh
        fi
    
        timeout="12h"
        if [! -z $DEBUG] && [$DEBUG == "TRUE"]; then
          timeout="30s"
        fi
    
        # loop infinitely and check for cert renewal every 12 hours
        # if the cert does not need renewing, certbot does nothing
        # after renewal, the deploy-cert-az-webapp.sh should fire to
        # register the renewed cert
        trap exit TERM; while :; do certbot renew --post-hook "deploy-cert-az-webapp.sh"; sleep $timeout & wait $!; done;
    fi
{% endraw %}
~~~

Notes:

- The script runs of environment variables like `$CDN` etc.
- Line 4: If staging (for test certificates) is set, `--staging` is added to the registration call
- Line 10: If you want to register `$CDN` and `www.$CDN` then you set `WWW` to 1. In my case, my CDN is `colinsalmcorner.com` and I wanted `www.colinsalmcorner.com` to be registered too, so I set `WWW` to 1. Subdomains like `blog.colinsalmcorner.com` should obviously set `WWW` to 0
- Line 15: Check if the cert exists, and make a registration request if it does not. This is why the persistent storage (the `${WEBAPP_STORAGE_HOME}` volume mapping) on the certbot image is so important.
- Lines 19-26: Register a request for a cert from LetsEncrypt. At this point, certbot will listen for the challenge from LetsEncrypt to `http://$CDN/.well-known/challenge/{some_random_goop}`which means that the DNS should be pointing to the Azure Web App and the custom domain registered on the Web App. 
- Line 29: invoke the script to register the cert with the Web App
- Line 41: Loop forever, calling `cerbot renew` every 12 hours. If the cert is not due for renewal, this ends as a no-op. If the cert(s) are renewed, the register script is invoked right after the renewal completes.

Here's the script to register the cert with Azure Web Apps:

~~~bash
{% raw %}
    #!/bin/sh
    
    certPath="$WORKING_PATH/live/$CDN"
    
    if [! -f "$WORKING_PATH/live/$CDN/fullchain.pem"]; then
      echo "ERROR: $WORKING_PATH/live/$CDN/fullchain.pem does not exist"
      exit 1
    fi
    
    # convert pem to pfx for azure web app
    echo "Converting pem to pfx"
    openssl pkcs12 \
        -password pass:$PFX_PASSWORD \
        -inkey $certPath/privkey.pem \
        -in $certPath/cert.pem \
        -export -out $certPath/cert.pfx
    
    # upload and get the thumbprint
    if [! -z $DEBUG] && [$DEBUG != "TRUE"]; then
        echo "DEBUG:: Running pfx upload and bind cert commands here"
        echo "DEBUG:: WebApp: $WEB_APP_NAME"
        echo "DEBUG:: Resource $RESOURCE_GROUP"
        echo "Contents of $certPath"
        ls -la $certPath
        
    else
        echo "Running az login"
        az login --service-principal -u $AZ_CLIENT_ID -p $AZ_CLIENT_KEY --tenant $AZ_TENANT_ID
    
        echo "Upload $certPath/cert.pfx to $WEB_APP_NAME in $RESOURCE_GROUP and get thumbprint"
        thumbprint=$(az webapp config ssl upload --certificate-file $certPath/cert.pfx \
                    --certificate-password $PFX_PASSWORD \
                    --name $WEB_APP_NAME --resource-group $RESOURCE_GROUP \
                    --query thumbprint --output tsv)
        
        # bind using the thumbprint
        echo "Bind cert"
        az webapp config ssl bind \
            --certificate-thumbprint $thumbprint \
            --ssl-type SNI \
            --name $WEB_APP_NAME --resource-group $RESOURCE_GROUP
    fi
    
    echo "Done!"
{% endraw %}
~~~

Notes:

- The script first converts the `pem` to a `pfx` using a password
- After that it uses `az cli` to login, upload the cert and bind the custom domain to the newly uploaded cert

## Wrapping Up

You can find all the steps and configuration settings you need to configure for this to work in the [readme](https://github.com/colindembovsky/webapp-linux-letsencrypt-auto-renew#steps). There's also [this script](https://github.com/colindembovsky/webapp-linux-letsencrypt-auto-renew/blob/master/sample-web-app-creation.sh) that shows how I spin up the site, configure the custom domain, upload the compose configuration, update the registry settings and then app settings and finally hit the site to start it. You should be able to get it going pretty quickly from here on!

Happy securing!

