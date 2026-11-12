<p align="center">
  <a href="https://fusionproxy.net/?utm_source=github&utm_medium=proxy-cheatsheet">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/FusionProxies/.github/main/assets/logo-on-dark.png">
      <img src="https://raw.githubusercontent.com/FusionProxies/.github/main/assets/logo-on-light.png" alt="Fusion Proxy" height="52">
    </picture>
  </a>
</p>

# Proxy cheatsheet: git, npm, pip, Docker, apt, SSH, Windows, macOS and more

How to set (and unset) an HTTP or SOCKS proxy for every common developer tool, in one page. Copy the line you need.

In the examples, replace the proxy with yours:

```
http://USER:PASS@proxy.example.com:8080
```

If the password contains `@ : / # ?` or `%`, URL-encode it (`@` becomes `%40`, `:` becomes `%3A`), or the tool will split the URL in the wrong place.

## Contents

[Environment variables](#environment-variables) ·
[cURL](#curl) · [wget](#wget) ·
[git](#git) · [npm, yarn, pnpm](#npm-yarn-pnpm) · [pip](#pip) · [conda](#conda) · [Go](#go) · [Maven](#maven) · [Gradle](#gradle) · [Java](#java) ·
[Docker](#docker) · [apt](#apt-debian-ubuntu) · [dnf and yum](#dnf-and-yum-fedora-rhel) · [SSH](#ssh-through-a-proxy) ·
[Windows](#windows) · [macOS](#macos) · [Linux desktop](#linux-desktop-gnome) · [VS Code](#vs-code) · [Code](#code)

## Environment variables

Most command-line tools read these.

```bash
# Linux and macOS (bash, zsh)
export HTTP_PROXY="http://USER:PASS@proxy.example.com:8080"
export HTTPS_PROXY="$HTTP_PROXY"
export NO_PROXY="localhost,127.0.0.1,.internal.example"
export http_proxy="$HTTP_PROXY" https_proxy="$HTTPS_PROXY" no_proxy="$NO_PROXY"

# remove
unset HTTP_PROXY HTTPS_PROXY NO_PROXY http_proxy https_proxy no_proxy
```

```powershell
# Windows PowerShell (this session)
$env:HTTP_PROXY  = "http://USER:PASS@proxy.example.com:8080"
$env:HTTPS_PROXY = $env:HTTP_PROXY
$env:NO_PROXY    = "localhost,127.0.0.1"

# Windows, permanent for your user (new terminals)
setx HTTPS_PROXY "http://USER:PASS@proxy.example.com:8080"
```

```bat
:: Windows cmd (this session)
set HTTPS_PROXY=http://USER:PASS@proxy.example.com:8080
```

- Set both cases: some tools read only lowercase (`http_proxy` for cURL), others only uppercase.
- `HTTPS_PROXY` is the proxy for **https websites**. Its value is still usually `http://...`: the tool opens a tunnel to the proxy and TLS runs inside it.
- `ALL_PROXY=socks5h://...` sets a SOCKS proxy for tools that support it (cURL, git with libcurl).

## cURL

```bash
curl -x http://proxy.example.com:8080 -U "USER:PASS" https://ipinfo.io/json
curl -x socks5h://USER:PASS@proxy.example.com:1080 https://ipinfo.io/json
curl --noproxy "*" https://example.com          # ignore proxy variables once
```

Everything else about cURL and proxies: [curl-proxy](https://github.com/FusionProxies/curl-proxy).

## wget

```bash
wget -e use_proxy=yes -e https_proxy=http://proxy.example.com:8080 \
     --proxy-user=USER --proxy-password=PASS https://example.com/file.zip
wget --no-proxy https://example.com/file.zip
```

Permanent, in `~/.wgetrc`:

```
use_proxy = on
http_proxy = http://USER:PASS@proxy.example.com:8080
https_proxy = http://USER:PASS@proxy.example.com:8080
```

## git

```bash
git config --global http.proxy http://USER:PASS@proxy.example.com:8080

# only for one host
git config --global http.https://github.com.proxy http://USER:PASS@proxy.example.com:8080

# SOCKS5
git config --global http.proxy socks5h://USER:PASS@proxy.example.com:1080

# check and remove
git config --global --get http.proxy
git config --global --unset http.proxy

# one command only
git -c http.proxy=http://USER:PASS@proxy.example.com:8080 clone https://github.com/FusionProxies/proxy-checker.git
```

- `http.proxy` covers both `http://` and `https://` remotes. `https.proxy` is not a git setting; git ignores it.
- git also uses `HTTPS_PROXY` from the environment.
- `http.proxy` does not apply to `git@github.com:` (SSH) remotes; see [SSH](#ssh-through-a-proxy).

## npm, yarn, pnpm

```bash
# npm (pnpm reads the same settings)
npm config set proxy http://USER:PASS@proxy.example.com:8080
npm config set https-proxy http://USER:PASS@proxy.example.com:8080
npm config delete proxy
npm config delete https-proxy

# one command
npm install express --proxy=http://proxy.example.com:8080 --https-proxy=http://proxy.example.com:8080

# Yarn 1
yarn config set proxy http://USER:PASS@proxy.example.com:8080
yarn config set https-proxy http://USER:PASS@proxy.example.com:8080

# Yarn 2+ (Berry)
yarn config set httpProxy http://USER:PASS@proxy.example.com:8080
yarn config set httpsProxy http://USER:PASS@proxy.example.com:8080
```

The npm settings live in `~/.npmrc`, so the password is stored in plain text there.

## pip

```bash
pip install requests --proxy http://USER:PASS@proxy.example.com:8080
```

Permanent, in `pip.conf` (`~/.config/pip/pip.conf` on Linux, `~/Library/Application Support/pip/pip.conf` on macOS, `%APPDATA%\pip\pip.ini` on Windows):

```ini
[global]
proxy = http://USER:PASS@proxy.example.com:8080
```

pip also reads `HTTPS_PROXY`. For SOCKS, install `pysocks` first.

## conda

`~/.condarc`:

```yaml
proxy_servers:
  http: http://USER:PASS@proxy.example.com:8080
  https: http://USER:PASS@proxy.example.com:8080
```

## Go

`go get` and module downloads use `HTTPS_PROXY`:

```bash
HTTPS_PROXY=http://USER:PASS@proxy.example.com:8080 go mod download
```

`GOPROXY` is something else: the **module mirror** Go downloads from (`https://proxy.golang.org`), not a network proxy.

## Maven

`~/.m2/settings.xml`:

```xml
<settings>
  <proxies>
    <proxy>
      <id>default</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>proxy.example.com</host>
      <port>8080</port>
      <username>USER</username>
      <password>PASS</password>
      <nonProxyHosts>localhost|127.0.0.1</nonProxyHosts>
    </proxy>
  </proxies>
</settings>
```

## Gradle

`~/.gradle/gradle.properties`:

```properties
systemProp.http.proxyHost=proxy.example.com
systemProp.http.proxyPort=8080
systemProp.http.proxyUser=USER
systemProp.http.proxyPassword=PASS
systemProp.https.proxyHost=proxy.example.com
systemProp.https.proxyPort=8080
systemProp.https.proxyUser=USER
systemProp.https.proxyPassword=PASS
systemProp.http.nonProxyHosts=localhost|127.0.0.1
```

## Java

```bash
java -Dhttps.proxyHost=proxy.example.com -Dhttps.proxyPort=8080 \
     -Dhttp.proxyHost=proxy.example.com -Dhttp.proxyPort=8080 \
     -Djdk.http.auth.tunneling.disabledSchemes= -jar app.jar
```

Java has no system property for the proxy password; set an `Authenticator` in code. `jdk.http.auth.tunneling.disabledSchemes=` is needed for Basic proxy authentication on HTTPS URLs. Example: [ProxyExample.java](https://github.com/FusionProxies/fusionproxy/blob/main/examples/java/ProxyExample.java).

## Docker

Three different places, for three different things.

**Pulling images** (the Docker daemon). Docker Engine 23 and newer, `/etc/docker/daemon.json`:

```json
{
  "proxies": {
    "http-proxy": "http://USER:PASS@proxy.example.com:8080",
    "https-proxy": "http://USER:PASS@proxy.example.com:8080",
    "no-proxy": "localhost,127.0.0.1"
  }
}
```

Older engines use a systemd drop-in, `/etc/systemd/system/docker.service.d/http-proxy.conf`:

```ini
[Service]
Environment="HTTP_PROXY=http://USER:PASS@proxy.example.com:8080"
Environment="HTTPS_PROXY=http://USER:PASS@proxy.example.com:8080"
Environment="NO_PROXY=localhost,127.0.0.1"
```

Then `sudo systemctl daemon-reload && sudo systemctl restart docker`. Docker Desktop: Settings, Resources, Proxies.

**Inside containers and builds** (your client), `~/.docker/config.json`:

```json
{
  "proxies": {
    "default": {
      "httpProxy": "http://USER:PASS@proxy.example.com:8080",
      "httpsProxy": "http://USER:PASS@proxy.example.com:8080",
      "noProxy": "localhost,127.0.0.1"
    }
  }
}
```

**One build only**:

```bash
docker build --build-arg HTTPS_PROXY=http://USER:PASS@proxy.example.com:8080 .
```

## apt (Debian, Ubuntu)

`/etc/apt/apt.conf.d/95proxy`:

```
Acquire::http::Proxy "http://USER:PASS@proxy.example.com:8080/";
Acquire::https::Proxy "http://USER:PASS@proxy.example.com:8080/";
```

One command: `sudo apt -o Acquire::https::Proxy="http://proxy.example.com:8080/" update`. Note that `sudo` does not pass your `HTTPS_PROXY` variable on by default.

## dnf and yum (Fedora, RHEL)

`/etc/dnf/dnf.conf` (or `/etc/yum.conf`), under `[main]`:

```ini
proxy=http://proxy.example.com:8080
proxy_username=USER
proxy_password=PASS
```

## SSH through a proxy

SSH does not read proxy variables. Use `ProxyCommand` in `~/.ssh/config`:

```
# through an HTTP proxy with a password (ncat comes with nmap)
Host server.example.com
    ProxyCommand ncat --proxy proxy.example.com:8080 --proxy-type http --proxy-auth USER:PASS %h %p

# through a SOCKS5 proxy without a password (OpenBSD netcat)
Host server.example.com
    ProxyCommand nc -X 5 -x proxy.example.com:1080 %h %p
```

The proxy must allow tunnels to port 22. Git over SSH (`git@github.com:`) uses the same config; GitHub also accepts SSH on `ssh.github.com` port 443 if only web ports are allowed.

## Windows

- **System proxy** (browsers and most apps): Settings, Network & internet, Proxy, Manual proxy setup.
- **WinHTTP** (Windows services and some updaters): `netsh winhttp set proxy proxy.example.com:8080 "localhost;127.0.0.1"` and `netsh winhttp reset proxy`.
- **PowerShell**: `Invoke-WebRequest https://ipinfo.io/json -Proxy http://proxy.example.com:8080 -ProxyCredential (Get-Credential)`.

## macOS

```bash
networksetup -setsecurewebproxy "Wi-Fi" proxy.example.com 8080 on USER PASS
networksetup -setwebproxy "Wi-Fi" proxy.example.com 8080 on USER PASS
networksetup -setsecurewebproxystate "Wi-Fi" off      # turn off
networksetup -listallnetworkservices                  # names like "Wi-Fi", "Ethernet"
```

Or System Settings, Network, your connection, Details, Proxies.

## Linux desktop (GNOME)

```bash
gsettings set org.gnome.system.proxy mode 'manual'
gsettings set org.gnome.system.proxy.https host 'proxy.example.com'
gsettings set org.gnome.system.proxy.https port 8080
gsettings set org.gnome.system.proxy mode 'none'      # turn off
```

## VS Code

`settings.json`:

```json
{
  "http.proxy": "http://USER:PASS@proxy.example.com:8080",
  "http.proxySupport": "override"
}
```

## Code

| Language | Guide |
|---|---|
| Python requests | [python-requests-proxy](https://github.com/FusionProxies/python-requests-proxy) |
| Node.js (fetch, axios) | [nodejs-proxy](https://github.com/FusionProxies/nodejs-proxy) |
| Go, PHP, Java, C#, Ruby | [fusionproxy examples](https://github.com/FusionProxies/fusionproxy) |
| Playwright, Puppeteer, Selenium | [playwright-proxy](https://github.com/FusionProxies/playwright-proxy), [puppeteer-proxy](https://github.com/FusionProxies/puppeteer-proxy), [selenium-proxy](https://github.com/FusionProxies/selenium-proxy) |
| Scrapy | [scrapy-proxy-middleware](https://github.com/FusionProxies/scrapy-proxy-middleware) |

## Check that it works

```bash
curl https://ipinfo.io/ip     # with the proxy variables set, this prints the proxy's IP
```

Or open the [what is my IP](https://fusionproxy.net/tools/my-ip?utm_source=github&utm_medium=proxy-cheatsheet) page, or test a proxy with the [online proxy checker](https://fusionproxy.net/tools/proxy-checker?utm_source=github&utm_medium=proxy-cheatsheet) and the [proxy-checker](https://github.com/FusionProxies/proxy-checker) CLI.

The git, npm, pip, Go and cURL commands on this page were run against a live proxy in November 2026. Spotted a tool that is missing or a setting that changed? Open an issue or a pull request.

Need a proxy? [Fusion Proxy](https://fusionproxy.net/?utm_source=github&utm_medium=proxy-cheatsheet) has residential, ISP and datacenter proxies with HTTP(S) and SOCKS5, and a [free trial](https://fusionproxy.net/free-trial?utm_source=github&utm_medium=proxy-cheatsheet).

## License

MIT, see [LICENSE](LICENSE).
