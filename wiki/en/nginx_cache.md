# How to manage caching in NGINX

## Default behavior

NGINX by default doesn't cache
To cache files, configure it in your configuration file.

## Clear Cache

- Check configuration of your nginx.
  - It may be located in `/etc/nginx/nginx.conf`
  - To see if it exists, run `ls /etc/nginx/`

- See where cache files are stored

  ```
  cd /etc/nginx/
  cat nginx.conf

  ->
  http {

    proxy_cache_path /var/cache/nginx/cache levels=1:2 keys_zone=zone1:4m inactive=7d max_size=50m;
    proxy_temp_path /var/cache/nginx/tmp;

  }
  ```

  - the value `proxy_cache_path` is the directory of your cache files

- Remove your cache files
  - move to that directory and remove files you want to

  ```
  cd /var/cache/nginx/ & ls
  rm -rf
  ```

## Cache with NGINX, Unicorn and Ruby on Rails on VirtualBox


When you don't see the changes you made in your browser,
- Browser(client-side) render cached file
- Web server(NGINX) returns a cache
- Application server(Unicorn) returns a cache

To clear cache in unicorn, run this
`bundle exec rake tmp:cache:clear`

### Problem in VirtualBox

Make sure you set `sendfile` to `off` in `nginx.conf` file.

> Sendfile is used to ‘copy data between one file descriptor and another‘ and apparently has some real trouble when run in a virtual machine environment, or at least when run through Virtualbox. Turning this config off in nginx causes the static file to be served via a different method and your changes will be reflected immediately and without question

Reference: [StackOverFlow](https://stackoverflow.com/a/13116771)
