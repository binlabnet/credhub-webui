# CredHub Web Interface

A web interface for performing basic functions with CredHub.

## Log in
You need to log in with valid Credentials
Currently supports:
* `client_id`
* `client_secret`

## Features
Currently supports:
* View
* Delete
* Generate
* Search

# Screenshots
Sign in
![sign in](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/01-sign_in.png)
List all credentials in CredHub
![list creds](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/02-list_creds.png)
Search credentials by using a search term, it uses `name-like` from CredHub
![search creds](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/03-search-creds.png)
View a selected credential
![view cred](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/04-view_cred.png)
Generate a credential
![generate cred](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/05-generate_cred.png)
View a generated credential
![view generated](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/06-view_generated.png)
Set a credential
![set credential](https://github.com/shreddedbacon/credhub-webui/blob/master/screenshots/07-set_credential.png)

# Deploy
Use BOSH to deploy this, see [this](https://github.com/shreddedbacon/credhub-webui-boshrelease) BOSH release

# Build
```
go get -v .
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -o credhub-webui .
```

# Run
```
./credhub-webui -ui-ssl-cert server.crt \
  -ui-ssl-key server.key \
  -cookie-key "super-secret-key" \
  -cookie-name "auth-cookie" \
  -credhub-server "https://192.168.50.6:8844" \
  -ui-url "https://localhost:8443" \
  -client-id "credhub" \
  -client-secret "credhubsecret"
```

# Docker
## Build
```
docker build -t shreddedbacon/credhub-webui .
```
## Run
```
docker run -p 8443:8443 \
  -e CLIENT_ID=credhub \
  -e CLIENT_SECRET=credhubsecret \
  -e UI_URL=https://localhost:8443 \
  -e CREDHUB_SERVER=https://<ip>:<port> \
  shreddedbacon/credhub-webui
```
E.g
```
docker run -p 8443:8443 \
  -e CLIENT_ID=credhub \
  -e CLIENT_SECRET=credhubsecret \
  -e UI_URL=https://localhost:8443 \
  -e CREDHUB_SERVER=https://192.168.50.6:8844 \
  shreddedbacon/credhub-webui
```

# Access

```
https://localhost:8443
```



# UAA Ops
```
# add a credhub user to bosh deployed uaa
- type: replace
  path: /instance_groups/name=bosh/jobs/name=uaa/properties/uaa/scim/users/-
  value:
    name: credhub
    groups: [credhub.write, credhub.read]
    password: credhub

# add a credhub webui client and button to uaa portal
- type: replace
  path: /instance_groups/name=bosh/jobs/name=uaa/properties/uaa/clients/credhub_webui?
  value:
    override: true
    authorized-grant-types: authorization_code,refresh_token
    scope: credhub.read,credhub.write
    authorities: ""
    access-token-validity: 300
    refresh-token-validity: 1800
    secret: "credhubsecret"
    autoapprove: true
    override: true
    redirect-uri: https://localhost:8443/login/callback
    show-on-homepage: true
    app-launch-url: https://localhost:8443
    app-icon: "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"
```
