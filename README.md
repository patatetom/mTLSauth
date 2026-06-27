# mTLSauth

[Caddy](https://caddyserver.com/)'s `forward_auth` authorization gateway based on the client certificate's serial number :
_Caddy submits each incoming request to mTLSauth, which returns a [200](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/200) (access granted) or [403](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403) (access denied) response depending on the client certificate's serial number and the requested application._


## see also

- [mTLS: When certificate authentication is done wrong](https://github.blog/security/vulnerability-research/mtls-when-certificate-authentication-is-done-wrong/)
