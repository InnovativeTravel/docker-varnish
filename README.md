docker-varnish
==============

Simple docker varnish image with throttle module.

## Pulling

```
$ docker pull zenedith/varnish
```

## Running

```
$ docker run -d -e BACKEND_PORT_80_TCP_ADDR=example.com -e BACKEND_ENV_PORT=80 -p 8080:80 zenedith/varnish
```

You can pass environmental variables to customize configuration:

```
LISTEN_ADDR 0.0.0.0
LISTEN_PORT 80
BACKEND_ADDR 0.0.0.0
BAKCEND_PORT 80
TELNET_ADDR 0.0.0.0
TELNET_PORT 6083
CACHE_SIZE 25MB
THROTTLE_LIMIT 150req/30s
VCL_FILE /etc/varnish/default.vcl
GRACE_TTL 30s
GRACE_MAX 1h
```

## Building

From sources:

```
$ docker build github.com/Zenedith/docker-varnish
```

## Fig Dockerenv
* Port 8080 - Listen (Can be anything except port 80 which is reserved on dockerenv)
* Port 6083 - varnishadm administrative / telnet port (Optional)

```yml
varnish:
  image: avatarnewyork/dockerenv-varnish
  volumes:
    - /var/www/sandbox/varnish/etc/varnish:/etc/varnish
  ports:
    - "8080"
    - "6083"
  links:
    - sandbox # Your apache instance
  environment:
    VCL_FILE: /etc/varnish/default.vcl
    LISTEN_PORT: 8080
```

### Testing
Assuming your Listen port is 8080, bring up your host URL in the web broser with the associated docker port.  For example:

`0.0.0.0:49475->8080/tcp`

Then the url would be `http://[host_IP]:49475` - you should see the cached version of the site you have defined in your vcl file.

### Varnish Config 
Due to the way varnish works with it's config file, there is no easy way to pass environment variables to it.  People have come up with a work-around: http://stackoverflow.com/questions/21056450/how-to-inject-environment-variables-in-varnish-configuration - and that is actually similar to what our image is doing.  Basically, you'll have 2 files:

```
/etc/varnish/default.vcl.source
/etc/varnish/default.vcl
```

The source file contains any environment variables you want parsed (that are available).  The run command (part of the image) replaces the defined variables in the source file with the environment variable values using sed and outputs it to the .vcl file.  Then it starts varnish and reads the .vcl file.  So you can think of default.vcl.source as a template and the .vcl file as throwaway (as it get's overwritten every startup).

One thing to note is whatever you define in the yml file (VCL_FILE: /etc/varnish/default.vcl) needs a matching .source 

#### Example _default.vcl.source_
```vcl
# This is a basic VCL configuration file for varnish.  See the vcl(7)
# man page for details on VCL syntax and semantics.
#
# Default backend definition.  Set this to point to your content
# server.
#

backend default {
  .host = "$PATSANDBOX_1_PORT_80_TCP_ADDR";
  .port = "$PATSANDBOX_1_PORT_80_TCP_PORT";
}
```

### Dynamic Reload
You can dynamicly reload your vcl file in dockerenv.  To do so, follow these steps:

1. ensure you have exposed port `6083` in your fig file (see above)
2. ensure your vcl file you want to reload has the exact path: `/etc/varnish/default.vcl`
3. Next run the dockerenv command: `varnish-reload [PROJECT_NAME]`
4. Your file has been reloaded if it compiled ok.  You can now re-test.


MIT License
-------

Copyright (c) 2014 Mateusz Stępniak


Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
