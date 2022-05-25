# Vault Encryption as a Service
consider you have a web application and a mysql server and you want to encrypt your info.
instead of encrypting the database, you can encrypt and decrypt all the transitions. so Vault can play as EaaS for you.

[https://www.vaultproject.io/docs/secrets/transit/](https://www.vaultproject.io/docs/secrets/transit/)

## Enable the Transit Secrets Engine
all secrets engine must be enabled before they can be used. so let's check them all
`vault secrets list`


you can enable Transit secret engin with the command below
`vault secrets enable -path=lob_a/workshop/transit transit`
or you can have it with this command with the default path transit/
```
vault secrets enable transit
```

you can verify it with `vault secrets list` and it's obvious

## Create a Key for the Transit Secrets Engine
Now we need to create and encryption key that the transit engine you created in the previous command will use to encrypt and decrypt sensitive data in mysql database running on vault server.for example SSN or Birthday or address. 
more info [https://www.vaultproject.io/docs/secrets/transit/#key-types](https://www.vaultproject.io/docs/secrets/transit/#key-types)

you need at least one key. its name is customer-key
`vault write -f lob_a/workshop/transit/keys/customer-key`
or if you are using default path you acn add keys
```
vault write -f transit/keys/my-key
```

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



### Encrypting Data

if you want to encrypt your data . you need to convert it to base64 and use the command below 
```sh
vault write transit/encrypt/demo-key plaintext=IlRoaXMgaXMgb3VyIGVuY29kZWQgdGV4dCI=
```

also you can use this command too 
```sh
vault write transit/encrypt/my-key plaintext=$(base64 <<< "my secret data")
```
I'm pretty sure that you know we had created my-key in the line 29 for transit secrets engine

| you need to store the ciphertext in your database and you can use it whenever you decided to decrypt your data.

### Decrypting Data
```sh
vault write transit/decrypt/demo-key ciphertext=YOUR-CIPHERTEXT-HERE

```
it will give you the base64 info and you can use it with this command ` base64 --decode <<< "bXkgc2VjcmV0IGRhdGEK"` I've used the base64 that the top command is created for use
### Website Used During Video:

https://www.base64decode.org/



## you can rotate your key 
Rotate the underlying encryption key. This will generate a new encryption key and add it to the keyring for the
`vault write -f transit/keys/my-key/rotate`