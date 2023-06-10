# Traefik Configuration Files

Once upon a time, when I first setup my homelab setup like I have it now, I was just about to throw my mini server right out of the window. The reason? Traefik.

Most example configs I found were overloaded or relied a lot on Docker labels. Traefiks documentation is also in parts rather confusing, which didn't help.

This repo contains a very simple, yet working config. You may have to adjust it according to your needs. Especially if you have special use cases requiring for middlewares, etc. However, for the very simplest task that a reserve proxy should accomplish, it suffices. SSL is preconfigured for use with a Cloudflare DNS challenge. In addition, SSL is configured to only use TLS v1.2 or up, along with preset lists of ciphers and curves.

This config relies much more on the configuration file "traefik.yml". In fact, your Trafik container does not need any labels to run. **If you already tried setting up Traefik, and have added commands or labels to its container, make sure to remove them.**

# Folder structure

```
└── traefik
      |── docker-compose.yml
      |── traefik.yml
      └── dynamic
            └── your-dynamic-config.yml
```

# Use of Docker labels

``` yml
    labels:
      traefik.enable: true
      traefik.http.routers.$SERVICE-secure.entrypoints: "websecure"
      traefik.http.routers.$SERVICE-secure.tls: "true"
      traefik.http.routers.$SERVICE-secure.tls.certresolver: "cloudflare"
      traefik.http.routers.$SERVICE-secure.rule: "Host(`service.example.com`)" # Change domain!
      traefik.http.services.$SERVICE-secure.loadbalancer.server.port: "8080" # Change port accordingly! 
```
To add services, simply add the following labels to the containers you want to add to Traefik. Adjust $SERVICE with the name for the service of your choice, the domain and the port accordingly.

If you want to add a middleware, simply include the following label:

``` yml
      traefik.http.routers.$SERVICE-secure.middlewares: "basicAuth@file" # Change middleware accordingly! 
```

## Something still doesn't work!

Check first if your container is accessible over IP. If it is, check the labels. Ensure they're correct. A novice mistake is to use the wrong port, so check to make sure you use the right one. If you define ports, always refer to the right port (e.g. 8888:**8080**). Please also check your logs accordingly, google the error you're shown.

If your container has access to two or more networks, you need to define which network Traefik is supposed to use to reach your container. Use the following label to do so (you may need to adjust the network name):

``` yml
      traefik.docker.network: "traefik_proxy"
```

If all that doesn't work, and you try to access your container over the internet, chances are it's either a setting with the container (e.g. "trusted proxy"), an SSL issue, or a network issue on your side (e.g. no port forward).


## I use Cloudflared and can't access my services!

If you use Cloudflared (aka. Cloudflare Tunnel), you'rely like to run into an SSL issue. This is especially the case if you use the Dashboard to configure your tunnel, rather than setting up your Ingress properly with a configuration file.

Use this guide: https://docs.ibracorp.io/cloudflare-tunnel/


## About Security of this config:

Exposing your server to the internet obviously poses risk. Only do so if you know what you're doing. Like I said, this config is meant to be simple. It's supposed to get you started with Traefik and nothing more. It is not designed to take the best practices of security into account. I cannot guarantee the integrity of your server. Proceed on your own risk. Please do your own research and follow best practices, in particular in regards to server hardening, and keep your software updated!
