Has your place of employment rolled out Zscaler and now network requests mysteriously fail? That's the power of zero trust security. A system that is inoperable can't be compromised!

# How to fix npm and pip SSL errors on a *nix machine

If npm is throwing `UNABLE_TO_GET_ISSUER_CERT_LOCALLY` or pip ain't working, download [Zscaler's public key](./zscaler_root_ca.pem) and define some [important environment variables](https://help.zscaler.com/zia/adding-custom-certificate-application-specific-trust-store) for npm and pip by doing the following:

```
mkdir -p ~/Documents/certs/
curl https://raw.githubusercontent.com/contolini/zscaler-woes/refs/heads/main/ca_bundle_with_zscaler.pem -o ~/Documents/certs/ca_bundle_with_zscaler.pem
curl https://raw.githubusercontent.com/contolini/zscaler-woes/refs/heads/main/zscaler_root_ca.pem -o ~/Documents/certs/zscaler_root_ca.pem
curl https://raw.githubusercontent.com/contolini/zscaler-woes/refs/heads/main/.env >> ~/.zshenv
source ~/.zshenv
```

This assumes you use zsh and have a `~/.zshenv` file. If you don't, put the environment variables wherever is appropriate.

## Docker and Zscaler

If you're using Docker in a project, add this to your `Dockerfile`:

```
# Copy the zscaler root CA certificate
# This cert can also be included in your project's repo if you don't want to remotely request it
ARG ZSCALER_CERT_LOCATION=https://raw.githubusercontent.com/contolini/zscaler-woes/refs/heads/main/zscaler_root_ca.pem
ADD ${ZSCALER_CERT_LOCATION} /usr/local/share/ca-certificates/zscaler_root_ca.pem

# Regenerate /etc/ssl/certs/ca-certificates.crt, the file that lists all system CA certs
RUN update-ca-certificates

# Tell pip to use the generated bundle of CA certs
RUN pip config set global.cert /etc/ssl/certs/ca-certificates.crt
```

Here's how it looks in consumerfinance.gov's [`Dockerfile`](https://github.com/cfpb/consumerfinance.gov/compare/ZsCaLeR).

## What is `ca_bundle_with_zscaler.pem`?

I took [Mozilla's CA certificate store](https://curl.se/docs/caextract.html) and added the Zscaler root CA signature to the bottom of it. Some projects might want a full list of certificate authorities.
