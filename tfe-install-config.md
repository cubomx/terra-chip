# Install & Config

## Post Install
- Initial Admins
- User management: SS0/SAML, IdP Config
- Admin account recovery

## IACT
Initial admin user must be created with Initial Admin Creation Token (IACT). 
Retrieve it with:
- Replicated `replicated admin retrieve-iact`
- CIDR mask to allow clients in that range to fecth the token via api. For
example, `tfe.mycompany.com` at `https://tfe.mycompany.com/admin/retrieve-iact`.
Can be limited the time for the requests.

## SAML SSO
Authentication & authorization. Also, Business tier. Requires 2 parties:
- Identity Provider (IdP)
- Service Provider (SP) or Relying Party (RP) -> TFE (don't create teams, match exactly teams -> SAML assertions)

## SAML Attributes
- Username Attribute Name: TFE username for a user logging (SSO)
- Site Admin Attribute Name: if it has site-admin permissions
- Team Attribute Name: team membership

## Tips
- Disable SAML SSO
- Create a non-SSO admin account for recovery (always available, can be created in Rails console, check who has shell access to the TFE host)
- Reconfigure SAML SSO with SAML Debugging enabled

----------------
# Automated TFE Installation
- Configure replicated
- Configure TFE itself

## Prerequisites
- App settings file (TFE). JSON.
- /etc/replicated.conf, (Replicated installer settings). (Paths to files, and other upper settings)
- License file to the instance
- [Airgapped] download `.airgap` bundle to the instance

Additional flags (IP addresses) to avoid being prompted.

List
- Operational Mode
- Credentials
- Data Storage (prepare them)
- Linux Instance (for running TFE)
- Net req (allow traffic)
- SSL Certificate (host & integrations)
- Pre-define App config (TFE settings)
- Set config files
- License file
- Download software

## Pre-configure
SSH your settings:
```sh
tfe$ replciatedctl app-config export > settings.json
```

Create your config file.

## Install
Online
```sh
curl -o install.sh https://install.terraform.io/ptfe/stable
bash ./install.sh \
    no-proxy \
    private-address=1.2.3.4
    public-address=5.6.7.8
```

For airgapped you must have the installer bootstrapper already on the instance.
Commonly unarchieved in `/tmp`.
```sh
cd /tmp
./install.sh \
    airgap \
    no-proxy \
    private-address=1.2.3.4
    public-address=5.6.7.8
```

## Waiting for TFE
Do a poll to the `/_health_check` endpoint until a 200 is returned

```sh
while ! curl -ksfS --connect-timeout 5 \
    https://tfe.example.com/_health_check: do
        sleep 5
done
```