# Vault Encryption as a Service
consider you have a web application and a mysql server and you want to encrypt your info.
instead of encrypting the database, you can encrypt and decrypt all the transitions. so Vault can play as EaaS for you.

[https://www.vaultproject.io/docs/secrets/transit/](https://www.vaultproject.io/docs/secrets/transit/)

## Enable the Transit Secrets Engine
all secrets engine must be enabled before they can be used. so let's check them all
`vault secrets list`

you can enable Transit secret engin with the command below
`vault secrets enable -path=lob_a/workshop/transit transit`

you can verify it with `vault secrets list` and it's obvious

## Create a Key for the Transit Secrets Engine
Now we need to create and encryption key that the transit engine you created in the previous command will use to encrypt and decrypt sensitive data in mysql database running on vault server.for example SSN or Birthday or address. 
more info [https://www.vaultproject.io/docs/secrets/transit/#key-types](https://www.vaultproject.io/docs/secrets/transit/#key-types)

you need at least one key. its name is customer-key
`vault write -f lob_a/workshop/transit/keys/customer-key`

you need to configure you application to use the encryption, for example for the python web app it can be like this 
```
[DEFAULT]
LogLevel = INFO
[DATABASE]
Address=localhost
Port=3306
User=root
Password=sJ2w*8NX
Database=my_app
[VAULT]
Enabled = True
DynamicDBCreds = True
DynamicDBCredsPath = lob_a/workshop/database/creds/workshop-app-long
ProtectRecords=True
Address=http://localhost:8200
#Address=vault.service.consul
Token=root
KeyPath=lob_a/workshop/transit
KeyName=customer-key



```