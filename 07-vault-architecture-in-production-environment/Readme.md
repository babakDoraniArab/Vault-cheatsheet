### Documentation 
https://www.vaultproject.io/docs/configuration

https://www.vaultproject.io/docs/configuration/storage/filesystem

## What will happen to our information?

in the dev env we was storing info in-memory but for production , we need better storages 
- Filesystem
- S3
- Consul
- Database (Mysql, DynamoDB , PostgreSQL)
## 01-lets start 
first we need a configuration file. it can be HCL or JSON, and it's obvious that I prefer HCL 

```sh 
storage "file" {
  path  = "/root/vault-data"
}

listener "tcp" {
 address     = "127.0.0.1:8200"
 tls_disable = 1
}

```

##   02 - Start Vault Server with Configuration File
```sh
vault server - config demo.hcl
```
##   03 - Initialize the Vault

we only initialize vault when the server is started against a new backend that has never been used with vault before.
 during initialization the encryption keys are generated , unseal keys are created , and the initial root token is setup.
```sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```
##   04 - Unseal the vault

every initialized vault server starts in the sealed state. from the configuration, vault can access the physical storage, but it can't read any of it because it doesn't know how to decrypt it.
the process of teaching vault how to decrypt the data is know as unsealing the vault .

```sh
vault operator unseal
```