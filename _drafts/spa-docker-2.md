---
layout: post
title: "Running a Single Page App in Docker - Part 2"
---

## Introduction

In my [last post][last-post], I left off with 2 docker containers: one running the Go server back-end and one running the Nginx front-end serving the JavaScript for the site.  In this post, I will describe how I got the whole app running using [Docker Compose][docker-compose].

Docker Compose is a tool that lets you specify several "services" (i.e. containers with their own Dockerfiles) in a single yaml file and then bring them all up together with `docker-compose up`.  This makes it easy to bring up multiple containers.


## Aside: Arriving at Docker Compose

Before I stumbled upon `docker-compose`, I figured that I wanted a single container with both my front-end and back-end running in it as separate processes.  I quickly realized this was going to be difficult because the two containers had different base images: The back-end started from debian, and the front-end started from alpine.  (It turns out there is a [Go image for alpine][go-alpine], but I did not realize this at the time).  I started looking at the Dockerfile [docs][dockerfile-docs] to see if I could have multiple `FROM` statements in a single Dockerfile.  This was not supported, and for good reason.  It wouldn't make sense to have two different linux distributions running in the same container.

I saw the light when I came across the [Docker FAQ][docker-faq] which had an answer about multiple processes:
>*How do I run more than one process in a Docker container?*
>
> This approach is discouraged for most use cases. For maximum efficiency and isolation, each container should address one specific area of concern. However, if you need to run multiple services within a single container, see [Run multiple services in a container][docker-multi].

## Getting Started

I included a cast of my foray into docker-compose below, but as you will be able to tell by the preceding play-by-play, I took a winding path with many detours, so I will summarize it for you.

Here is what my final `docker-compose.yaml` looked like:

```
version: '2'
services:
  backend:
    build: .
  frontend:
    build: frontend/.
    ports:
      - "5000:80"
```

First, we specify which version of the `docker-compose` parser we are going to use.  Then each service starts with a name, a build file and a list of ports.

The name is the name of the service and also becomes the hostname for the container running that service.  For instance, in the `frontend` container, I can send requests to `backend:5001` to talk to the API on that port.  

The `build` portion points to a relative path that contains a `Dockerfile`. In this case, the `back-end` Dockerfile is at the root of my repo since I have some go-modules also defined at that level, and the `front-end` has its dockerfile specified in the `front-end` directory.

The ports are what ports are forwarded to localhost.  In this case, I have `localhost:5000` forwarding to `frontend:80`.  The `nginx` server is running on port 80 on the `frontend` container, but I want to have it be `localhost:5000` because that is the port I have configured through the Google Cloud Console for the allowed origin of an OAuth request.

I can then run `docker-compose up -d` in the root of my directory, and my app will come up listening on `localhost:5000`.  (Side note: The `-d` option starts in headless mode, so to bring it down you say `docker-compose down`)

## Nginx Reverse Proxy

The only other piece I needed to configure was setting up nginx to forward requests to the back-end.  This makes it appear to the frontend JavaScript that the static files and API responses are coming from the same place.

It turns out to be relatively easy to match URL's based on a path and then direct them to different hosts. I originally saw this technique in a [blog post][decouple] about decoupling front-ends from back-ends and later saw some more detail about it in a [gist][nginx-proxy]. Here is the `nginx.conf` I ended up with:

```
http {
    ...

    server {
      listen    80;
      server_name   localhost;

      location / {
        root /usr/share/nginx/html;
        index  index.html  index.htm;
      }

      location /api {
        proxy_pass http://api;
      }
    }

    upstream api {
      # Keep this in sync with the host name given in docker-compose.yaml
      server backend:5001;
    }
}
```

This specifies that for the `/` URL's, serve from the static directory, but for any URL starting with `/api`, forward it to `http://api` which we have defined in the `upstream api` block as being `backend:5001`.

## Snags

`docker-compose` and `nginx` worked as advertised in my experience.  I ran into a few unrelated snags a long the way that I'll mention here and you can witness in the cast below.

The first was `curl`-ing the docker-compose tool.  I kept goofing up the `sudo tee` incantation to allow me to dump the file to `/usr/local/bin`.  Here is the one I ended up with:
```
curl -L https://github.com/docker/compose/releases/download/1.14.0-rc2/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose >/dev/null
```

The next two were related to cookies.  I make two `fetch` calls in my client JavaScript: 1) to send the JWT to login and 2) to fetch the initial batch of transactions.  I only included the `credentials: same-origin` in the second call.  I didn't realize this would cause the first call to ignore any `Set-Cookie` commands in the response.  This was due to my experimentation in cookies leading me to manually set the cookie and not realize that the code itself wasn't handling it correctly.

The other cookie issue I ran into was that I wasn't setting a proper expiration date, so the cookies were expiring immediately.  I thought this was why the cookie's weren't being set, but it turned out to be the JS problem above.  Still, this would have caused me issues if I tried to revisit the site and expected to be "remembered" because the cookies would be gone.

## The Cast

Here is the [`asciinema`](https://asciinema.org) cast I recorded of this process.

<script type="text/javascript" src="https://asciinema.org/a/124559.js" id="asciicast-124559" async></script>


## What's Next

Here are some unanswered questions and things I would like to do next:

  - QOL improvement: How do I not re-fetch all my go dependencies every time I build that image?
  - Where is all this stuff is stored, i.e. am I filling up my harddrive with slightly different versions of the same container?
  - Is there a full VM running in order to run different base images?
  - Deploy to GCE? or ECS?
  - Get an actual storage engine running to store the tokens
  - Run the actual bank account flow from Plaid
  - Get an HTTPS connection going with Lets Encrypt.
  - Clean up the backend code to eliminate the OAuth flow

Thanks for reading! As always feel free to reach out [on twitter][https://twitter.com/paulcarletonjr] with your thoughts.

[last-post]:{% post_url 2017-06-07-spa-docker %}

[nginx-proxy]:https://gist.github.com/soheilhy/8b94347ff8336d971ad0
[decouple]:https://www.linkedin.com/pulse/decouple-your-single-page-app-from-backend-nginx-tom-bray
[nginx-docker]:https://github.com/nginxinc/docker-nginx/blob/master/mainline/alpine/Dockerfile
[dockerfile-docs]:https://docs.docker.com/engine/reference/builder/#format
[docker-multi]:https://docs.docker.com/engine/admin/multi-service_container/
[docker-faq]:https://docs.docker.com/engine/faq/#how-do-i-run-more-than-one-process-in-a-docker-container
[docker-compose]:https://docs.docker.com/compose/
