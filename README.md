nginx-cors
----------
Turn on the CORS by Feng Shui... maybe 🙃

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

#### ./cors-enabled.conf
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
	~https://api.auth.project.com/(login|logout)$ "cors.service.auth";
	default "<<unknown>>";
}

# Convert Origin to CORS client
map "$http_origin" $cors_client {
	~https://(foo|bar)\.client\.com "cors.client.$1"; # "cors.client.foo" or "cors.client.bar";
	default "<<unknown>>";
}

# Turn on CORS by client and service map
map "$cors_client -> $cors_service" $cors_enabled {
	# Access for 'foo' client
	"cors.client.foo -> cors.service.auth" "true";
	"cors.client.foo -> cors.service.user-info" "true";

	# Access for 'bar' client
	"cors.client.bar -> cors.service.auth" "true";
}
```

#### ./nginx.conf

```nginx
http {
	# One (!)
	include 'cors-enabled.conf';

	server {
		listen 80;
		server_name api.project.com;

		location /user/info {
			# Two (!!)
			include 'nginx-cors/cors.conf';
		}
	}

	server {
		listen 80;
		server_name api.auth.project.com;

		location ~/(login|logout) {
			# Three (!!!)
			include 'nginx-cors/cors.conf';
		}
	}
}
```

### OR

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

See detail in source: [./cors.conf](./cors.conf#L1-L8)

---

### Please send a PR if you know how to make it more beautiful and correctly.

Thx.