# Required files (not in git)

Place these files in this directory before running `nginx-install.yml` / `nginx-ssl-certs.yml`:

| File | Purpose |
|------|---------|
| `nginx-repo.key` | NGINX Plus repository client key |
| `nginx-repo.crt` | NGINX Plus repository client certificate |
| `license.jwt` | NGINX Plus license |
| `STAR.GHC.COM.crt` | Wildcard TLS certificate |
| `ghc.com.key` | TLS private key |

Obtain Plus repo credentials and JWT from your [NGINX Plus subscription](https://www.nginx.com/products/nginx/).
