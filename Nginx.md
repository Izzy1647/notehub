A reverse proxy.

```nginx
http {
	# import/include mime types
	include mime.types;
	
	server {
	    # Tells the nginx server to listen to pot 8008,
	    # and serve index.html at root
		listen 8080;
		root /User/zzy/Desktop/mysite;

		# regexp for location
		location ~* /count/[0-9] {
			root /User/zzy/Desktop/mysite;
			try_files /index.html =404
		}

		# set path: append /fruits to the root route
		location /fruits {
			root /User/zzy/Desktop/mysite;
		}

		# set path: no appending
		location /carbs {
			alias /User/zzy/Desktop/mystite/fruits;
		}

		location /vegetables {
			root /User/zzy/Desktop/mysite;
			try_files /vegetables/veggies.html /index.html =404;
			
		}
	}

}

events{

}
```

## Nginx as a load balancer
- a round robin algorithm
```nginx
http {
	upstream backendserver {
		server 127.0.0.1:1111;
		server 127.0.0.1:2222;
		server 127.0.0.1:3333;
		server 127.0.0.1:4444;
	}
	
	location / {
		# by default, round robin
		proxy_pass http://backendserver/;
	}
}
```
