# Troubleshooting

> _**This document contains common problems and their solutions.**_
>
> _If you came across an issue where the solution was not immediately obvious, consider adding it to this list to help other users._

### Contents
- [Refused to Connect in Web Content View](#refused-to-connect-in-modal-or-workspace-view)
- [404 On Static Hosting](#404-on-static-hosting)
- [Yarn Build or Run Error](#yarn-error)
- [Auth Validation Error: "should be object"](#auth-validation-error-should-be-object)
- [Config Not Updating](#config-not-updating)
- [Config Still not Updating](#config-still-not-updating)
- [Styles and Assets not Updating](#styles-and-assets-not-updating)
- [DockerHub toomanyrequests](#dockerhub-toomanyrequests)
- [Config Validation Errors](#config-validation-errors)
- [Node Sass unsupported environment](#node-sass-does-not-yet-support-your-current-environment)
- [Cannot find module './_baseValues'](#error-cannot-find-module-_basevalues)
- [Ngrok Invalid Host Headers](#invalid-host-header-while-running-through-ngrok)
- [Warnings in the Console during deploy](#warnings-in-the-console-during-deploy)
- [Docker Login Fails on Ubuntu](#docker-login-fails-on-ubuntu)
- [Status Checks Failing](#status-checks-failing)
- [How-To Open Browser Console](#how-to-open-browser-console)
- [Git Contributions not Displaying](#git-contributions-not-displaying)

---
## `Refused to Connect` in Modal or Workspace View

This is not an issue with Dashy, but instead caused by the target app preventing direct access through embedded elements. It can be fixed by setting the [`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) HTTP header set to `ALLOW [path to Dashy]` or `SAMEORIGIN`, as defined in [RFC-7034](https://datatracker.ietf.org/doc/html/rfc7034). These settings are usually set in the config file for the web server that's hosting the target application, here are some examples of how to enable cross-origin access with common web servers:

### NGINX
In NGINX, you can use the [`add_header`](https://nginx.org/en/docs/http/ngx_http_headers_module.html) module within the app block.
```
server {
  ...
  add_header X-Frame-Options SAMEORIGIN always;
}
```
Then reload with `service nginx reload`

### Caddy

In Caddy, you can use the [`header`](https://caddyserver.com/docs/caddyfile/directives/header) directive.

```yaml
header {
  X-Frame-Options SAMEORIGIN
}
```

### Apache

In Apache, you can use the [`mod_headers`](https://httpd.apache.org/docs/current/mod/mod_headers.html) module to set the `X-Frame-Options` in your config file. This file is usually located somewhere like `/etc/apache2/httpd.conf

```
Header set X-Frame-Options: "ALLOW-FROM http://[dashy-location]/" 
```

---

## 404 On Static Hosting

If you're seeing Dashy's 404 page on initial load/ refresh, and then the main app when you go back to Home, then this is likely caused by the Vue router, and if so can be fixed in one of two ways. 

The first solution is to switch the routing mode, from HTML5 `history` mode to `hash` mode, by setting `appConfig.routingMode` to `hash`.

If this works, but you wish to continue using HTML5 history mode, then a bit of extra [server configuration](/docs/management.md#web-server-configuration) is required. This is explained in more detaail in the [Vue Docs](https://router.vuejs.org/guide/essentials/history-mode.html). Once completed, you can then use `routingMode: history` again, for neater URLs.

---

## Yarn Error

For more info, see [Issue #1](https://github.com/Lissy93/dashy/issues/1)

First of all, check that you've got yarn installed correctly - see the [yarn installation docs](https://classic.yarnpkg.com/en/docs/install) for more info.

If you're getting an error about scenarios, then you've likely installed the wrong yarn... (you're [not](https://github.com/yarnpkg/yarn/issues/2821) the only one!). You can fix it by uninstalling, adding the correct repo, and reinstalling, for example, in Debian:
- `sudo apt remove yarn`
- `curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -`
- `echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list`
- `sudo apt update && sudo apt install yarn`

Alternatively, as a workaround, you have several options:
- Try using [NPM](https://www.npmjs.com/get-npm) instead: So clone, cd, then run `npm install`, `npm run build` and `npm start`
- Try using [Docker](https://www.docker.com/get-started) instead, and all of the system setup and dependencies will already be taken care of. So from within the directory, just run `docker build -t lissy93/dashy .` to build, and then use docker start to run the project, e.g: `docker run -it -p 8080:80 lissy93/dashy` (see the [deploying docs](https://github.com/Lissy93/dashy/blob/master/docs/deployment.md#deploy-with-docker) for more info)

---

## Auth Validation Error: "should be object"

In V 1.6.5 an update was made that in the future will become a breaking change. You will need to update you config to reflect this before V 2.0.0 is released. In the meantime, your previous config will continue to function normally, but you will see a validation warning. The change means that the structure of the `appConfig.auth` object is now an object, which has a `users` property.

For more info, see [this announcement](https://github.com/Lissy93/dashy/discussions/177).

You can fix this by replacing:

```yaml
auth:
- user: xxx
  hash: xxx
```

with

```yaml
auth:
  users:
  - user: xxx
    hash: xxx
```

---

## Config Not Updating

Dashy has the option to save settings and config locally, in browser storage. Anything here will take precedence over whatever is in your config file, sometimes with unintended consequences. If you've updated the config file manually, and are not seeing changes reflected in the UI, then try visiting the site in Incognito mode. If that works, then the solution is just to clear local storage. This can be done from the config menu, under "Clear Local Settings".

---

## Config Still not Updating

Sometimes your text editor updates files [inode](https://linuxhandbook.com/inode-linux/), meaning changes will not be picked up by the Docker container. This [article](https://medium.com/@jonsbun/why-need-to-be-careful-when-mounting-single-files-into-a-docker-container-4f929340834) explains things further.

---

## Styles and Assets not Updating

If you find that your styles and other visual assets work when visiting `ip:port` by not `dashy.domain.com`, then this is usually caused by caching. In your browser, do a hard-refresh (<kbd>Ctrl</kbd> + <kbd>F5</kbd>). If you use Cloudflare, then you can clear the cache through the management console, or set the cache level to Bypass for certain files, under the Rules tab.

---

## DockerHub `toomanyrequests`

This situation relates to error messages similar to one of the following, returned when pulling, updating or running the Docker container from Docker Hub.

```
Continuing execution. Pulling image lissy93/dashy:release-1.6.0 
error pulling image configuration: toomanyrequests
```
or
```
You have reached your pull rate limit. You may increase the limit by authenticating and upgrading: https://www.docker.com/increase-rate-limit
```

When DockerHub returns one of these errors, or a `429` status, that means you've hit your rate limit. This was [introduced](https://www.docker.com/blog/scaling-docker-to-serve-millions-more-developers-network-egress/) last year, and prevents unauthenticated or free users from running docker pull more than 100 times per 6 hours.
You can [check your rate limit status](https://www.docker.com/blog/checking-your-current-docker-pull-rate-limits-and-status/) by looking for the `ratelimit-remaining` header in any DockerHub responses. 

#### Solution 1 - Use an alternate container registry
- Dashy is also availible through GHCR, which at present does not have any hard limits. Just use `docker pull ghcr.io/lissy93/dashy:latest` to fetch the image
- You can also build the image from source, by cloning the repo, and running `docker build -t dashy .` or use the pre-made docker compose

#### Solution 2 - Increase your rate limits
- Logging in to DockerHub will increase your rate limit from 100 requests to 200 requests per 6 hour period
- Upgrading to a Pro for $5/month will increase your image requests to 5,000 per day, and any plans above have no rate limits
- Since rate limits are counted based on your IP address, proxying your requests, or using a VPN may work

---

## Config Validation Errors
The configuration file is validated against [Dashy's Schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.json) using AJV.

First, check that your syntax is valid, using [YAML Validator](https://codebeautify.org/yaml-validator/) or [JSON Validator](https://codebeautify.org/jsonvalidator). If the issue persists, then take a look at the [schema](https://github.com/Lissy93/dashy/blob/master/src/utils/ConfigSchema.json), and verify that the field you are trying to add/ modify matches the required format. You can also use [this tool](https://www.jsonschemavalidator.net/s/JFUj7X9J) to validate your JSON config against the schema, or run `yarn validate-config`.

If you're trying to use a recently released feature, and are getting a warning, this is likely because you've not yet updated the the current latest version of Dashy.

If the issue still persists, you should raise an issue.

---

## Node Sass does not yet support your current environment
Caused by node-sass's binaries being built for a for a different architecture
To fix this, just run: `yarn rebuild node-sass`

---

## Error: Cannot find module './_baseValues'
Clearing the cache should fix this: `yarn cache clean`
If the issue persists, remove (`rm -rf node_modules\ yarn.lock`) and reinstall (`yarn`) node_modules

---

## Invalid Host Header while running through ngrok
Just add the [-host-header](https://ngrok.com/docs#http-host-header) flag, e.g. `ngrok http 8080 -host-header="localhost:8080"`

---

## Warnings in the Console during deploy
Please acknowledge the difference between errors and warnings before raising an issue about messages in the console. It's not unusual to see warnings about a new version of a certain package being available, an asset bundle bing oversized or a service worker not yet having a cache. These shouldn't have any impact on the running application, so please don't raise issues about these unless it directly relates to a bug or issue you're experiencing. Errors on the other hand should not appear in the console, and they are worth looking into further.

---

## Docker Login Fails on Ubuntu

Run `sudo apt install gnupg2 pass && gpg2 -k`

---

## Status Checks Failing
If you're using status checks, and despite a given service being online, the check is displaying an error, there are a couple of things you can look at:

If your service requires requests to include any authorization in the headers, then use the  `statusCheckHeaders` property, as described in the [docs](/docs/status-indicators.md#setting-custom-headers).

If you are still having issues, it may be because your target application is blocking requests from Dashy's IP. This is a [CORS error](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), and can be fixed by setting the headers on your target app, to include:
```
Access-Control-Allow-Origin: https://location-of-dashy/
Vary: Origin
```
If the URL you are checking has an unsigned certificate, or is not using HTTPS, then you may need to disable the rejection of insecure requests. This can be done by setting `statusCheckAllowInsecure` to true for a given item.

If you get an error, like `Service Unavailable: Server resulted in a fatal error`, even when it's definitely online, this is most likely caused by missing the protocol. Don't forget to include `https://` (or whatever protocol) before the URL, and ensure that if needed, you've specified the port.

Currently, the status check needs a page to be rendered, so if this URL in your browser does not return anything, then status checks will not work. This may be modified in the future, but in the meantime, a fix would be to make your own status service, which just checks if your app responds with whatever code you'd like, and then return a 200 plus renders an arbitrary message. Then just point `statusCheckUrl` to your custom page.

For further troubleshooting, use an application like [Postman](https://postman.com) to diagnose the issue. Set the parameter to `GET`, and then make a call to: `https://[url-of-dashy]/status-check/?&url=[service-url]`. Where the service URL must have first been encoded (e.g. with `encodeURIComponent()` or [urlencoder.io](https://www.urlencoder.io/))

If you're serving Dashy though a CDN, instead of using the Node server or Docker image, then the Node endpoint that makes requests will not be available to you, and all requests will fail. A workaround for this may be implemented in the future, but in the meantime, your only option is to use the Docker or Node deployment method. 

---

## How-To Open Browser Console
When raising a bug, one crucial piece of info needed is the browser's console output. This will help the developer diagnose and fix the issue.

If you've been asked for this info, but are unsure where to find it, then it is under the "Console" tab, in the browsers developer tools, which can be opened with <kbd>F12</kbd>. You can right-click the console, and select Save As to download the log.

To open dev tools, and jump straight to the console:
- Win / Linux: <kbd>Ctrl</kbd> + <kbd>Shift</kbd> + <kbd>J</kbd>
- MacOS: <kbd>Cmd</kbd> + <kbd>Option</kbd> + <kbd>J</kbd>

For more detailed walk through, see [this article](https://support.shortpoint.com/support/solutions/articles/1000222881-save-browser-console-file).

---

## Git Contributions not Displaying

If you've contributed to Dashy (or any other project), but your contributions are not showing up on your GH profile, or in Dashy's [Credits Page](https://github.com/Lissy93/dashy/blob/master/docs/credits.md), then this is likely a git config issue.

These statistics are generated using the username / email associated with commits. This info needs to be setup on your local machine using [`git config`](https://git-scm.com/docs/git-config).

Run the following commands (replacing name + email with your info):
- `git config --global user.name "John Doe"`
- `git config --global user.email johndoe@example.com`

For more info, see [Git First Time Setup Docs](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup).

Note that only contributions to the master / main branch or a project are counted
