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

# Endpoints which can CORS supported
map "$scheme://$host$request_uri" $cors_supported {
	"https://api.project.com/user/info" "$host$request_uri";
	~https://api.auth.project.com/(login|logout) "$host$request_uri";
}

# Origin and Endpoints for which enabled CORS
map "$http_origin--->$cors_supported" $cors_enabled {
	~https://(foo|bar)\.client\.com--->api\.project\.com/user/info "true";
	~https://(foo|bar)\.client\.com--->api\.auth\.project\.com/(login|logout) "true";
}
```

#### ./nginx.conf

```nginx
http {
	# One (!)
	include "cors-enabled.conf";

	server {
		listen 80;
		server_name api.project.com;

		location /user/info {
			# Two (!!)
			include 'cors.conf';
		}
	}

	server {
		listen 80;
		server_name api.auth.project.com;

		location ~/(login|logout) {
			# Three (!!!)
			include 'cors.conf';
		}
	}
}
```

### OR

```nginx
location /api/user/exists {
	if ($http_origin = 'https://client.com' {
		set $cors_enabled 'true';
	}
	include 'cors.conf';
}
```

---

### Variables

See detail in source: [./cors.conf](./cors.conf#L1-L9)

---

### Please send a PR if you know how to make it more beautiful and correctly.

Thx.