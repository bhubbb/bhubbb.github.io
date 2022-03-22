---
layout: post
title: "Single Docker Endpoint for Nexus Repository Manager"
tags: 
  - docker
  - nginx
  - haproxy
  - nexus
  - repository
  - reverse proxy
---

I use [Sonatype Nexus Repository Manager OSS](https://www.sonatype.com/products/repository-oss) professionally to handle artifacts for our projects.
We also use containers a fair bit but how Nexus handles these out of the box is quite annoying (Each repo is a separate port and authentication point).

I was able to front Nexus with a reverse proxy and allow one endpoint for all of our internal docker repositories while maintaining individual authentication and access rules.

![hlad](/images/single-docker-endpoint-for-nexus-repository-manager/hlad.svg)

We can do this because the docker client uses http to communicate and it passes its own user-agent (`Docker-Client/[docker client version]`) and has a standard path when looking for the docker repository it wants to talk to (`/v2/[repository name]`). Once that is understood it is quite trivial to sent up a single endpoint `nexus.example.com` and delegate the docker traffic to the ports the repositories are listening on.

This really should just be a feature built into Nexus Repository Manager.

## Example Configs
### NGINX
This is just an all in one configuration however I do not see why an `include docker_repos/*.conf` could not be inside the user-agent if block.
```nginx
server {
  listen              443 ssl;

  ssl_certificate     cert;
  ssl_certificate_key key;

  proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;

  // Nexus Repository Manager
  location / {
    proxy_pass http://[nexus host]:[nexus host port];
  }

  // Docker Repositories
  if ($http_user_agent ~ ^Docker-Client) {
    location /v2/docker_repo_01 {
      proxy_pass http://[nexus host]:[nexus docker repo 01 port];
    }
    location /v2/docker_repo_02 {
      proxy_pass http://[nexus host]:[nexus docker repo 02 port];
    }
  }
}
```

### HAPROXY
The version of HAPROXY I am configuring for here is `1.8`.
```haproxy
frontend https_in
  bind    :443 ssl crt key.pem
  option  forwardfor
  # Set the docker repo redirects first
  # HAPROXY matches top to bottom
  use_backend docker_repo_01 if { hdr_beg(user-agent) -i Docker-Client } { path_beg -i /v2/docker_repo_01 }
  use_backend docker_repo_02 if { hdr_beg(user-agent) -i Docker-Client } { path_beg -i /v2/docker_repo_02 }
  # Nexus Repository Manager
  use_backend nexus_http

backend nexus_http
  server nexus_http [nexus host]:[nexus host port]

backend docker_repo_01
  server docker_repo_01 [nexus host]:[nexus docker repo 01 port] check

backend docker_repo_02
  server docker_repo_02 [nexus host]:[nexus docker repo 02 port] check
```
