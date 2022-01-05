# certbot-dns-desec: Get Let's Encrypt Certificates for Domains Hosted at deSEC

![main branch CI test status](https://github.com/desec-io/certbot-dns-desec/workflows/Tests/badge.svg?branch=main)
[![pypi badge](https://img.shields.io/pypi/v/certbot-dns-desec.svg)](https://pypi.org/project/certbot-dns-desec/)

Certbot plugin to obtain TLS certificates from Let's Encrypt for domains hosted with deSEC.io, using the DNS challenge
challenge mechanism.


## Installation

To get certificates from Let's Encrypt, install certbot and this plugin.
There are many ways to install certbot, this guide uses Python's `pip`:

```
python3 -m pip install certbot certbot-dns-desec
```

## Prerequisites

To get a Let's Encrypt certificate for your domain,
you need an access token for the deSEC.io account that holds the domain you need the certificate for.
Also make sure that your domain name has been delegated to deSEC according to deSEC's instructions.


## Issue Certificate

To issue and renew certificates using `certbot-dns-desec`, an access token to your deSEC account is required.
To store such a token in a secure location, use, e.g.:

```
DOMAIN=example.com
TOKEN=your-desec-access-token
sudo mkdir /etc/letsencrypt/secrets/
sudo chmod 700 /etc/letsencrypt/secrets/
echo "dns_desec_token = $TOKEN" | sudo tee /etc/letsencrypt/secrets/$DOMAIN.ini
sudo chmod 600 /etc/letsencrypt/.secrets/$DOMAIN.ini
```

Adjust `$DOMAIN` and `$TOKEN` according to your domain and deSEC access token, respectively.
The file location is just a suggestion and can be changed.

With the credentials stored, you can apply for a wildcard certificate for your domain by using, e.g.,

```
certbot certonly \
     --authenticator dns-desec \
     --dns-desec-credentials /etc/letsencrypt/secrets/$DOMAIN.ini \
     -d "$DOMAIN" \
     -d "*.$DOMAIN"
```

In this command, `--authenticator dns-desec` activates the `certbot-dns-desec` plugin;
the `--dns-desec-credentials` argument provides the deSEC access token location to the plugin.
These flags can be combined with more sophisticated usages of certbot, e.g. to automatically reload servers after the
renewal process.


## CLI Interface

This plugin is activated by passing the ``--authenticator dns-desec`` argument to certbot.
It accepts the following command line arguments:

1. ``--dns-desec-credentials <file>`` Specifies the file holding the deSEC API credentials (required, see below).
1. ``--dns-desec-propagation-seconds`` Waiting time for DNS to propagate before asking the ACME server to verify the
    DNS record (default 80).


## Credentials File Format

The credentials file only holds the deSEC API access token:

    dns_desec_token = token


## Development and Testing

To test certbot-dns-desec, create a virtual environment at `venv/` for this repository and activate it.
Register a domain `$DOMAIN` with desec.io, and obtain a DNS management token `$TOKEN`. Then run

```
python3 -m pip install .
TOKEN=token-you-obtained-from-desec-io
DOMAIN=domain-you-registered-at-desec-io
EMAIL=youremail@example.com
echo "dns_desec_token = $TOKEN" > desec-secret.ini
chmod 600 desec-secret.ini
./venv/bin/certbot \
    --config-dir tmp/certbot/config \
    --logs-dir tmp/certbot/logs \
    --work-dir tmp/certbot/work \
    --test-cert \
    -d $DOMAIN -d "*.$DOMAIN" \
    --authenticator dns-desec \
    --dns-desec-credentials desec-secret.ini \
    --non-interactive --agree-tos \
    --email $EMAIL \
    certonly
```


## Maintenance: Prepare New Release

1. Make sure tests are okay (see GitHub actions)
1. Commit all changes
1. Clean up `dist/` folder
1. Set up new release version: `RELEASE=x.y.z`
1. Update version to `x.y.z` in `setup.py`
1. Commit with message "Release Version vx.y.z": `git commit -p -m "Release Version v$RELEASE"`
1. Tag commit using `git tag -as v$RELEASE -m "Release Version v$RELEASE"`
1. Push
    1. branch: `git push`
    1. tag: `git push origin v$RELEASE`
1. Set environment variables `GITHUB_TOKEN` to a GitHub token, `TWINE_USERNAME` and `TWINE_PASSWORD` to PyPi
    credentials.
1. Publish using `python3 -m publish desec-io certbot-dns-desec`
