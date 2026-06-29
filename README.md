# mTLSauth

[Caddy](https://caddyserver.com/)'s `forward_auth` authorization gateway based on client certificate's serial number :
_Caddy submits each incoming request to mTLSauth, which returns a [200](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/200) (access granted) or [403](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403) (access denied) response depending on client certificate's serial number and requested application._


## Caddy

here is an example configuration for Caddy :
```Caddyfile
{
	admin off
	auto_https off
}
www.applications.net {
	tls /root/pki/server/tls.crt /root/pki/server/tls.key {
		client_auth {
			mode require_and_verify
			trust_pool file /root/pki/root/ca.crt
			verifier revocation {
				mode crl_only
				crl_config {
					work_dir /tmp/
					storage_type memory
					signature_validation_mode verify
					crl_file /root/pki/server/crl.pem
					trusted_signature_cert_file /root/pki/root/ca.crt
				}
			}
		}
	}
	handle {
		forward_auth 127.0.0.1:3000 {
			uri /
			header_up -*
			header_up X-Client-Ip {client_ip}
			header_up X-Client-Serial {http.request.tls.client.serial}
		}
		handle_path /application.one/* {
			reverse_proxy 127.0.0.1:8000
		}
	}
}
```

> client certificate is forced with `mode require_and_verify`.<br/>
> block `forward_auth 127.0.0.1:3000 {…}` enforces authorization through mTLSauth.<br/>
> to streamline processes, most of headers are removed using `header_up -*`.<br/>
> client certificate's serial number is added to headers passed to mTLSauth with `header_up X-Client-Serial…`.<br/>
> client's IP address is also added to the headers with `header_up X-Client-Ip…` for logging purposes.


## mTLSauth configuration

mTLSauth configuration file is `mTLSauth.toml` : it is expected to be located where mTLSauth is started.

for each application that needs to be accessed, it lists in a table serial number - in decimal format - of certificates authorized :
```toml
# alice 243667759311437977940774218371054068666
#   bob 367124548323783656842008786261177525455

"application.one" = [
"243667759311437977940774218371054068666",
"367124548323783656842008786261177525455",
]
```

> [SPPKI](https://github.com/patatetom/SPPKI) can be used to create and manage certificates for a small organization.<br/>
> command `python -c "print(int('$(openssl x509 -in /root/pki/users/alice.crt -noout -serial)'[7:], 16))"` can be used to easily retrieve serial number of client certificate in decimal format.


# mTLSauth

check and install [mTLSauth](mTLSauth) in `/usr/local/sbin/` (for example) and :

```console
~# /usr/local/sbin/mTLSauth
2026-06-28 21:00:05,500 INFO 🟦 mTLSauth listening on 127.0.0.1:3000
2026-06-28 21:00:11,562 WARNING 🟥 not « GET / HTTP/1.1 »
2026-06-28 21:00:29,738 WARNING 🟧 176.174.26.24 243667759311437977940774218371054068666 /bad.application/
2026-06-28 21:00:37,556 WARNING 🟨 176.174.62.42 000000000000000000000000000000000000000 /application.one/
2026-06-28 21:00:51,117 INFO 🟩 176.174.28.176 243667759311437977940774218371054068666 /application.one/
2026-06-28 21:00:56,974 INFO 🟩 176.174.82.16 367124548323783656842008786261177525455 /application.one/
```

## see also

- [Caddy's forward_auth directive](https://caddyserver.com/docs/caddyfile/directives/forward_auth)
- [mTLS: When certificate authentication is done wrong](https://github.blog/security/vulnerability-research/mtls-when-certificate-authentication-is-done-wrong/)
- [TinyAuth](https://tinyauth.app/)
