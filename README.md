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
			uri /mTLSauth
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
> to streamline the processes, most of the headers are removed using `header_up -*`.<br/>
> client certificate's serial number is added to the headers passed to mTLSauth with `header_up X-Client-Serial…`.<br/>
> client's IP address is also added to the headers with `header_up X-Client-Ip…` for logging purposes.


## see also

- [Caddy's forward_auth directive](https://caddyserver.com/docs/caddyfile/directives/forward_auth)
- [mTLS: When certificate authentication is done wrong](https://github.blog/security/vulnerability-research/mtls-when-certificate-authentication-is-done-wrong/)
