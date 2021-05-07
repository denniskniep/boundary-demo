# Boundary Demo

## Quickstart
Add /etc/hosts entry
```
127.0.0.1 keycloak
```

Start Components
```
sudo docker-compose -f docker-compose.fromImage.yaml -f docker-compose.env.yaml up --build
```

Boundary: `http://localhost:9200`
admin:admin123456

Keycloak: `http://localhost:8080`
admin:admin

Keycloak Realm Boundary: `http://localhost:8080/auth/realms/Boundary/account/#/` 
Demo1:demo1


Install boundary CLI
https://github.com/hashicorp/boundary/releases


## Use Boundary with Password

Authenticate
```
export BOUNDARY_TOKEN=`boundary authenticate password -auth-method-id=ampw_1234567890 -login-name=admin -password=admin123456 -keyring-type=none -format=json | jq -r '.item.token'`
```

Connect to SSH Server
```
boundary connect -target-scope-name="CoreInfra" -target-name="backend_servers_ssh" -host-id="hst_BEoIGODMHp" -listen-port=1234
```


## Use Boundary with IdentityProvider via OpenIdConnect

Authenticate
```
export BOUNDARY_TOKEN=`boundary authenticate password -auth-method-id=ampw_1234567890 -login-name=admin -password=admin123456 -keyring-type=none -format=json | jq -r '.item.token'`

export AUTH_METHOD_KEYCLOAK=`boundary auth-methods list -format=json | jq -r '.items[] | select( .name == "keycloak" ) | .id'`

export BOUNDARY_TOKEN=`boundary authenticate oidc -auth-method-id ${AUTH_METHOD_KEYCLOAK} -format=json | jq -r '.item.token'`
```

Connect to SSH Server
```
export HOST_CATALOG_NAME="backend_servers"
export HOST_NAME="backend_server_service_ssh-testserver-01"

export HOST_CATALOG_ID=`boundary host-catalogs list -recursive -format json | jq -r ".items[] | select( .name == \"${HOST_CATALOG_NAME}\" ) | .id"`

export HOST_ID=`boundary hosts list -host-catalog-id=${HOST_CATALOG_ID} -format=json | jq -r ".items[] | select( .name == \"${HOST_NAME}\" ) | .id"`

boundary connect -target-scope-name="CoreInfra" -target-name="backend_servers_ssh" -host-id="${HOST_ID}" -listen-port=1234
```

Connect to ssh with `admin:password`
```
ssh admin@127.0.0.1 -p 1234
```

## Export Keycloak realm settings
Execute following in running Keycloak container

```
/opt/jboss/keycloak/bin/standalone.sh \
-Djboss.socket.binding.port-offset=100 -Dkeycloak.migration.action=export \
-Dkeycloak.migration.provider=singleFile \
-Dkeycloak.migration.realmName=Boundary \
-Dkeycloak.migration.usersExportStrategy=REALM_FILE \
-Dkeycloak.migration.file=/tmp/realm.json
```

Copy `/tmp/realm.json`
```
sudo docker cp <containerId>:/tmp/realm.json /tmp/realm.json
```


## Run Boundary compiled sources 
Add /etc/hosts entry
```
127.0.0.1 keycloak
```

Clone Boundary (https://github.com/hashicorp/boundary) next to Boundary_demo

In Boundary Directory excecute:
```
make dev
```

Start Components
```
sudo docker-compose -f docker-compose.fromLocal.yaml -f docker-compose.env.yaml up --build
```