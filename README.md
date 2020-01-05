nginx-cors
----------
Turn on the CORS by Feng Shui... maybe 🙃

### Featuring

- [Easy and clear to use](#compare)
- Does not use `add_header` into `if` (therefore, your headers will not be lost)
- Configurable and flexibility

---

### Articles

- [NGINX vs. CORS (ru)](https://medium.com/@ibnRubaXa/nginx-vs-cors-7a63029d9a34)

---

### Install

```sh
cd /usr/local/etc/nginx/
git clone git@github.com:RubaXa/nginx-cors.git
```

---

### Hot to use

```sh
cd /usr/local/etc/nginx/
vim ./cors-enabled.conf
```

#### 1. Create `./cors.setup.conf`
CORS confirugration

```nginx
# If in your project setted larger, then this directive is not required
map_hash_bucket_size 256;

# Setup the $cors_path variable
map $request_uri $cors_path {
  ~^(?<path>[^?]+) $path;
}

# Convert Endpoints to CORS service
map "$scheme://$host$cors_path" $cors_service {
  "https://api.project.com/user/info" "cors.service.user-info";
  ~^https:\/\/api.auth.project.com\/(login|logout)$ "cors.service.auth";
  default "<<unknown>>";
}

# Convert Origin to CORS client
map "$http_origin" $cors_client {
  # "cors.client.foo" or "cors.client.bar";
  ~^https:\/\/(foo|bar)\.client\.com$ "cors.client.$1";
  default "<<unknown>>";
}

# Turn on CORS by client and service map
map "$cors_client -> $cors_service" $cors_enabled {
  # Access for 'foo' client
  "cors.client.foo -> cors.service.auth"      "true";
  "cors.client.foo -> cors.service.user-info" "true";

  # Access for 'bar' client
  "cors.client.bar -> cors.service.auth" "true";
}
```

#### 2. Enable CORS on any endpoint 

```nginx
http {
  # 1️⃣Setup CORS for all services (!)
  include 'cors.setup.conf';

  server {
    listen 80;
    server_name api.project.com;

    location /user/info {
      # 2️⃣Enable CORS without credentials/cookies (!!)
      include 'nginx-cors/cors.conf';
    }
  }

  server {
    listen 80;
    server_name api.auth.project.com;

    location ~/(login|logout) {
      # 3️⃣ Enable CORS with credentials/cookies (!!!)
      set $cors_allow_credentials "true";
      include 'nginx-cors/cors.conf';
    }
  }
}
```

### OR without cors service/client (not recommended, see [compare](#compare))

```nginx
location /api/user/exists {
  if ($http_origin = 'https://client.com') {
    set $cors_enabled 'true';
  }
  include 'nginx-cors/cors.conf';
}
```

---

### Variables

See detail in source: [cors.conf](./cors.conf#L1-L9)

---

### Please send a PR if you know how to make it more beautiful and correctly.

Thx.

---

<a name="compare"></a>

### Compare

#### Classical bugy way 💩

- It's hard to read who exactly enabled CORS
- It is easy to make a mistake in RegExp
- If there is an add_header inside, it will not work

```nginx
location ~ '/api/auth/(login|logout)' {
  if ($http_origin ~ '^https?://(about|company|account)\.(((alpha|beta|omega)\.))?test\.)?)?business\.com$')
    set $cors 'enabled';
  }

  add_header X-My-Header "true"; # it will not work 🐞

  if ($request_method = 'OPTIONS') {
    set $cors "preflight-${cors}";
  }

  if ($cors = 'preflight-enabled') {
    add_header Access-Control-Allow-Origin $http_origin;
    add_header Access-Control-Allow-Headers "...";
    add_header Access-Control-Allow-Methods "...";
    add_header Access-Control-Expose-Headers "X-My-Header";
    return 204;
  }

  if ($cors = 'enabled') {
    add_header X-My-Header "true"; # but here it will work (1)
    add_header Access-Control-Allow-Origin $http_origin;
  }

  # bla-bla-bla
}
```


### True way 👍

- Easy to read and maintain.
- Flexible configuration
- Operating not at the RegExp level, but client -> service

```nginx
# Setup the $cors_path variable
map $request_uri $cors_path {
  ~^(?<path>[^?]+) $path;
}

# Convert endpoints to CORS service
map "$scheme://$host$cors_path" $cors_service {
  ~^https:\/\/project.com/api/(login|logout)$ "cors.service.auth";
  default "<<unknown>>";
}

# Convert Origin to CORS client
map "$http_origin" $cors_client {
  # Yes, it would be possible to write in one regexp, but it is much more readable and more flexible.
  # In addition, you need to list subdomains! Why so? See next `map` ;]
  ~^https:\/\/([^\.]+)\.business\.com$" "cors.client.business.$1";
  ~^https:\/\/([^\.]+)\.test\.business\.com$" "cors.client.business.$1";
  ~^https:\/\/([^\.]+)\.(alpha|beta|omega)\.test\.business\.com$" "cors.client.business.$1";
  default "<<unknown>>";
}

# Turn on CORS by client and service map
map "$cors_client -> $cors_service" $cors_enabled {
  # And now we can clearly and elegantly include rights
  # based on the "$cors_client" and the "$cors_service" to which "client" invoke!
  "cors.client.business.about -> cors.service.auth"   "false";
  "cors.client.business.company -> cors.service.auth" "true";
  "cors.client.business.account -> cors.service.auth" "true";
  default "false";
}
```

and

```nginx
location ~ '/api/auth/(login|logout)' {
  add_header X-My-Header "true"; # It's work! 🎉
  set $cors_allow_expose_headers "X-My-Header";
  include "ngxin-cors/cors.conf";
}
```
