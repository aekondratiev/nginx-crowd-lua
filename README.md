# About #
This repository contains a simple [Atlassian Crowd](https://www.atlassian.com/software/crowd) authentication script for [nginx](http://nginx.org/), written
in [Lua](http://www.lua.org/), for use with the [access_by_lua_file](https://github.com/chaoslawful/lua-nginx-module#access_by_lua_file) directive.

This is used in production on Centos7, running the latest
[openresty](https://openresty.org/en/linux-packages.html) nginx package. An attempt was made to use as
much "off-the-shelf" packaging as possible. This script relies
on the use of [lua-Spore](http://fperrad.github.io/lua-Spore/) REST
client/library.

Please note there is an actual nginx module available that can accomplish the
same, however it requires building your own packages. It is available
[here](https://github.com/kare/ngx_http_auth_crowd_module). I am making this
available as, in all things, it is preferrable (for me, at any rate), to use
packages when available over building packages or compiling software directly.

Also, please see the related [JIRA Issue](https://jira.atlassian.com/browse/CWD-2754).

## Installation and Configuration ##

- Install openresty package and lua-Spore dependency:

```
# add the yum repo:
sudo wget https://openresty.org/package/centos/openresty.repo -O /etc/yum.repos.d/openresty.repo

# then you can install a package, say, openresty, like this:
sudo yum install openresty

# luarocks and lua-Spore
wget http://luarocks.org/releases/luarocks-2.0.13.tar.gz
tar -xzvf luarocks-2.0.13.tar.gz
cd luarocks-2.0.13/
./configure --prefix=/usr/local/openresty/luajit \
    --with-lua=/usr/local/openresty/luajit/ \
    --lua-suffix=jit \
    --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1
make
sudo make install

/usr/local/openresty/luajit/bin/luarocks install luasec OPENSSL_LIBDIR=/usr/lib64
/usr/local/openresty/luajit/bin/luarocks install lua-spore
```

- Copy the `crowd-auth.lua` to somewhere accessible by nginx:

```
mkdir -p /etc/nginx/lua && cp crowd-auth.lua /etc/nginx/lua
```

- [Add a new application in
  Crowd](https://confluence.atlassian.com/display/CROWD/Adding+an+Application),
  and test the connectivity with the users/groups you wish to authenticate
  with.

- Modify the `crowd-auth.lua`, replacing `<CROWD_APP_URL>`, `<CROWD_APP_NAME>`,
  `<CROWD_APP_PASS>` with the Crowd base url, the application name and the
  application password created in the previous step.

- Add a `access_by_lua_file` directive in a nginx site stanza, similar to the following:

```
server {
  listen 80;

  server_name host.example.com;
  root /var/www/host.example.com;

  location /protected {
    access_by_lua_file /etc/nginx/lua/crowd-auth.lua;
  }
}
```

