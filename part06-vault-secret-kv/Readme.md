### let's how check all the secrets engine that we have 
```sh
 vault secrets list
```
### shall we create a new path for KV ?
yes you can start another kv secret engine with a different path

```sh
vault secrets enable -path=demopath -version=2 kv

```
run the command below one more time and check your secrets engine
`vault secrets list`

if you want to destroy it just go for this command `vault secrets disable demopath`
[check this documentation](https://www.vaultproject.io/docs/secrets)
### to see your current KV secret keys
```sh
vault kv list secret
```

### Create multiple versions of secret
```sh
vault kv put secret/second-secret user=admin01
vault kv put secret/second-secret user=admin02
vault kv put secret/second-secret user=admin03
```
### Read Secret:
```sh
vault kv get secret/second-secret
```
### Read specific version of secret
```sh
vault kv get -version=2 secret/second-secret
```

### put more than one version for a secret 
if you want to go with that you need to write the command couple of times and create new version of the file

```sh
vault kv put secret/test3 password=test3version2
vault kv put secret/test3 password=test3version3

```

### how check all version of the secret 
```sh
vault kv metadata get secret/test3
```

### how check specific version of a secret 
```sh
vault kv get -version=3 secret/test3
```

### Delete specific version of secret
```sh
vault kv delete -versions=2 secret/second-secret
```
### Undelete version of secret
```sh
vault kv undelete -versions=2 secret/second-secret
```
### Permanently delete version of secret:
```sh
vault kv destroy -versions=2 secret/second-secret
```
### Delete the secret
```sh
vault kv metadata delete secret/second-secret
```

### lease management 

```sh
vault lease renew -increment=36000 demopath/creds/admini-role/wT7Nuleh1RUW5XTKUuemxwZ2
vault lease revoke -h
```