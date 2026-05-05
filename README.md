# EDtunnel
Use Cloudflare pages and worker serverless to implement VLESS protocol.

<br>

## Deploy in pages.dev
1. See YouTube Video: [https://www.youtube.com/watch?v=8I-yTNHB0aw](https://www.youtube.com/watch?v=8I-yTNHB0aw)
2. Clone this repository deploy in cloudflare pages.

## Deploy in worker.dev
1. Copy `_worker.js` code from [here](https://github.com/Vauth/vless-cf/blob/main/_worker.js).
2. Alternatively, you can click the button below to deploy directly.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/Vauth/vless-cf)


## DNS Relay (fixes `dns: exchange failed … EOF` errors)

### What causes the EOF error?
When a VLESS client (e.g. v2rayN) uses a built-in DNS module to resolve domain names
through the proxy, it typically sends UDP DNS queries (port 53) inside the VLESS/WebSocket
tunnel. If the Worker cannot forward those queries — for example because the configured
DNS-over-HTTPS (DoH) endpoint is blocked — the client gets an immediate connection close
(`EOF`) and cannot browse any website even though the tunnel itself is active.

### How the DNS relay works
This Worker intercepts those UDP port-53 queries inside the tunnel and forwards each DNS
message to a configurable upstream DNS server using a **DNS-over-TCP** connection (TCP
connects to port 53 on the DNS server). This avoids dependence on DoH endpoints and
typically resolves the EOF error.

### DNS relay environment variables

| Variable | Default | Description |
|---|---|---|
| `ENABLE_DNS_RELAY` | `true` | Set to `false` to fall back to DoH instead of direct relay. |
| `DNS_SERVER_ADDRESS` | `1.1.1.1` | IP/hostname of the upstream DNS server for direct relay. |
| `DNS_SERVER_PORT` | `53` | Port of the upstream DNS server. |
| `DNS_RESOLVER_URL` | `https://freedns.controld.com/p0` | DoH URL used only when `ENABLE_DNS_RELAY` is `false`. |

#### How to set env vars in Cloudflare

**Workers (dashboard)**:
1. Open your Worker → **Settings** → **Variables**.
2. Under *Environment Variables*, click **Add variable**.
3. Add `DNS_SERVER_ADDRESS`, `DNS_SERVER_PORT`, and/or `ENABLE_DNS_RELAY` with the values
   you want, then click **Save and deploy**.

**Workers (wrangler.toml)**:
```toml
[vars]
DNS_SERVER_ADDRESS = "1.1.1.1"
DNS_SERVER_PORT    = "53"
ENABLE_DNS_RELAY   = "true"
```

**Pages**:
1. Open your Pages project → **Settings** → **Environment variables**.
2. Add the variables under *Production* (and optionally *Preview*) and click **Save**.

### Recommended v2rayN DNS settings

Open **Settings → DNS Settings** and configure:

| Setting | Recommended value |
|---|---|
| Bootstrap DNS | `1.1.1.1` (or `8.8.8.8`) |
| Remote DNS (DoH) | `https://cloudflare-dns.com/dns-query` |
| Domestic DNS | `https://cloudflare-dns.com/dns-query` |

If you still get EOF errors, try **disabling custom DNS** in v2rayN and let the operating
system handle DNS resolution (select *Use system DNS* or clear the custom DNS fields).

## WebSocket path

The Worker accepts WebSocket upgrade requests on **any** path. The subscription generator
(`/sub/<uuid>`) and the config page (`/<uuid>`) both output `/ws` as the WebSocket path,
which is the value to use in client configuration:

```
WS Path: /ws
```

Make sure your v2rayN / Clash profile uses `/ws` (with the leading slash) and **not** `ws`
(without a slash) or `/?ed=2048`.

## DoH with Cloudflare
1. Follow the https://github.com/serverless-dns/serverless-dns .
2. Replace the dns url with `dohURL` value in `_worker.js` .

## UUID Setting (Optional)

1. When deploy in cloudflare pages, you can set uuid in `wrangler.toml` file. variable name is `UUID`. `wrangler.toml` file is also supported. (recommended) in case deploy in webpages, you can not set uuid in `wrangler.toml` file.

2. When deploy in worker.dev, you can set uuid in `_worker.js` file. variable name is `userID`. `wrangler.toml` file is also supported. (recommended) in case deploy in webpages, you can not set uuid in `wrangler.toml` file. in this case, you can also set uuid in `UUID` enviroment variable.

Note: `UUID` is the uuid you want to set. pages.dev and worker.dev all of them method supported, but depend on your deploy method.

### UUID Setting Example

1. single uuid environment variable

   ```.environment
   UUID = "uuid here your want to set"
   ```

2. multiple uuid environment variable

   ```.environment
   UUID = "uuid1,uuid2,uuid3"
   ```

   note: uuid1, uuid2, uuid3 are separated by commas`,`.
   when you set multiple uuid, you can use `https://edtunnel.pages.dev/uuid1` to get the clash config and vless:// link.

## subscribe vless:// link (Optional)

1. visit `https://edtunnel.pages.dev/uuid your set` to get the subscribe link.

2. visit `https://edtunnel.pages.dev/sub/uuid your set` to get the subscribe content with `uuid your set` path.

   note: `uuid your set` is the uuid you set in UUID enviroment or `wrangler.toml`, `_worker.js` file.
   when you set multiple uuid, you can use `https://edtunnel.pages.dev/sub/uuid1` to get the subscribe content with `uuid1` path.(only support first uuid in multiple uuid set)

3. visit `https://edtunnel.pages.dev/sub/uuid your set/?format=clash` to get the subscribe content with `uuid your set` path and `clash` format. content will return with base64 encode.

   note: `uuid your set` is the uuid you set in UUID enviroment or `wrangler.toml`, `_worker.js` file.
   when you set multiple uuid, you can will use `https://edtunnel.pages.dev/sub/uuid1/?format=clash` to get the subscribe content with `uuid1` path and `clash` format.(only support first uuid in multiple uuid set)

## subscribe Cloudflare bestip(pure ip) link

1. visit `https://edtunnel.pages.dev/bestip/uuid your set` to get subscribe info.

2. cpoy subscribe url link `https://edtunnel.pages.dev/bestip/uuid your set` to any clients(clash/v2rayN/v2rayNG) you want to use.

3. done. if have any questions please join [@edtunnel](https://t.me/edtunnel)

## multiple port support (Optional)

   <!-- let portArray_http = [80, 8080, 8880, 2052, 2086, 2095];
	let portArray_https = [443, 8443, 2053, 2096, 2087, 2083]; -->

For a list of Cloudflare supported ports, please refer to the [official documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/ports).

By default, the port is 80 and 443. If you want to add more ports, you can use the following ports:

```text
80, 8080, 8880, 2052, 2086, 2095, 443, 8443, 2053, 2096, 2087, 2083
http port: 80, 8080, 8880, 2052, 2086, 2095
https port: 443, 8443, 2053, 2096, 2087, 2083
```

if you deploy in cloudflare pages, https port is not supported. Simply add multiple ports node drictly use subscribe link, subscribe content will return all Cloudflare supported ports.

## proxyIP (Optional)

1. When deploy in cloudflare pages, you can set proxyIP in `wrangler.toml` file. variable name is `PROXYIP`.

2. When deploy in worker.dev, you can set proxyIP in `_worker.js` file. variable name is `proxyIP`.

note: `proxyIP` is the ip or domain you want to set. this means that the proxyIP is used to route traffic through a proxy rather than directly to a website that is using Cloudflare's (CDN). if you don't set this variable, connection to the Cloudflare IP will be cancelled (or blocked)...

resons: Outbound TCP sockets to Cloudflare IP ranges are temporarily blocked, please refer to the [tcp-sockets documentation](https://developers.cloudflare.com/workers/runtime-apis/tcp-sockets/#considerations)

## Usage

frist, open your pages.dev domain `https://edtunnel.pages.dev/` in your browser, then you can see the following page:
The path `/uuid your seetting` to get the clash config and vless:// link.

## Star History

<a href="https://www.star-history.com/#vauth/vless-cf&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=vauth/vless-cf&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=vauth/vless-cf&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=vauth/vless-cf&type=Date" />
 </picture>
</a>
