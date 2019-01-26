---
layout: post
title: Reverse proxying Plex using nginx on NixOS
---

[Plex](https://www.plex.tv/) is a popular media server. I switched to it (from Kodi + a bunch of network shares) a while ago, primarily because it supports server-side transcoding, meaning I can watch things that the itty bitty RPis I use as media players just don't have the beef to decode, like 1080p HEVC files.

Plex, by default, allows access only from localhost. It can be configured to allow access from other systems on the same subnet, and binds to `INADDR_ANY`, but any access it doesn't recognize will result in a redirect to a `plex.tv` login page. The intent is that if you want to perform any sort of remote access, you need to make a Plex account, register your server with them, and go through their authentication servers.

I don't want to do that, though. I just want to self-host and use nginx's HTTP authentication to control access. So let's get started.

For the purposes of this blog post, we assume that the server plex is running on is called `plex`, with internal IP `192.168.86.100`, and it can be accessed from the outside as `plex.example.net`.

It's running NixOS, so installing Plex is easy enough:

    services.plex = {
      enable = true;
      openFirewall = true;
      dataDir = "/srv/plex";
      extraPlugins = [
        # Downloaded manually to add YouTube support.
        ./plex-plugins/YouTubeTV.bundle
      ];
    };

At this point it's listening on `http://plex:32400`, but serving redirects to anything but localhost. Let's fix that first, so that it can at least be used by other systems on the LAN:

    # sed -E -i "/srv/plex/Plex Media Server/Preferences.xml" \
      's:allowedNetworks="[^"]*":allowedNetworks="192.168.86.0/255.255.255.0,127.0.0.1/255.0.0.0":'
    # systemctl restart plex

And now we can (from a laptop, say) get at the web UI using `http://192.168.86.100:32400/`. Note, however, that `http://plex:32400/` won't work, even on the LAN! This is because Plex inspects not just the originating IP but also the `Host:`, `Origin:`, and `Referer:` HTTP headers.

So, we need an nginx configuration that not only enforces HTTPS and proxies the traffic correctly, but also rewrites those headers so that the server doesn't redirect us anyways.

Back to the NixOS configuration we go. First, some simple redirects so that going to `http://plex` on the LAN automatically redirects us to `plex.example.net`, so we only need to set up the proxy configuration in one place.

    nginx.virtualHosts."plex".locations."/" = {
      extraConfig = "return 301 https://plex.example.net/web/index.html;";
    };

Note that we redirect to `/web/index.html`; this is necessary, as going to `/` with the proxy configuration we're about to write will cause Plex to just serve us some XML about the server capabilities.

Now we write the actual proxy configuration, which will fire when the server is accessed as `plex.example.net`.

First, require SSL, and automatically get certificates from Let's Encrypt via ACME:

    nginx.virtualHosts."plex.example.net" = {
      forceSSL = true;
      enableACME = true;

Set up a username and password:

      basicAuth = {
        user = "password";
      };

Proxy all requests to the Plex server:

      locations."/".extraConfig = ''
        proxy_pass    http://127.0.0.1:32400;
      '';


And now, the actual proxy configuration. First, we clear the headers that Plex uses to figure out if this is "remote access" or not:

      extraConfig = ''
        proxy_set_header Host "127.0.0.1:32400";
        proxy_set_header Referer "";
        proxy_set_header Origin "http://127.0.0.1:32400";

Enable proxying of websockets:

        proxy_set_header Sec-WebSocket-Extensions $http_sec_websocket_extensions;
        proxy_set_header Sec-WebSocket-Key $http_sec_websocket_key;
        proxy_set_header Sec-WebSocket-Version $http_sec_websocket_version;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

And finally turn off buffering to reduce latency when watching video:

        proxy_redirect off;
        proxy_buffering off;
      '';
    };

Et voila! It's not perfect -- we need to go to `plex.example.net/web/index.html` rather than just `plex.example.net` -- but this gives us access from both inside and outside the network, and requires HTTP authentication before we can access it.

There's just one wart, though: inside the LAN, going to `http://plex` still prompts us for a username and password. It would be nice if, for LAN access, it didn't require this. There's probably some way to make `basicAuth` conditional on the originating IP in nginx, but there's an easier way: modify the redirect to embed the credentials in the URL.

    nginx.virtualHosts."plex".locations."/" = {
      extraConfig = "return 301 https://user:password@plex.example.net/web/index.html;";
    };

If, for whatever reason, you want people on the LAN to be able to easily access the server without being able to see the credentials used to access it remotely, you could also redirect to the IP, bypassing nginx entirely:

    nginx.virtualHosts."plex".locations."/" = {
      extraConfig = "return 301 http://192.168.86.100:32400/web/index.html;";
    };

And there we go! Plex accessible from both inside and outside the LAN, the latter only when authenticated to nginx, and all without needing a Plex account.
