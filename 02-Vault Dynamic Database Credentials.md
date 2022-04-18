
# Vault Dynamic Database Credentials
Secrets engines are vault plugins that store, generate , or encrypt data. secrets engines are incredibly flexible, so it is easiest to think about them in terms of their function.
Vault's Database secrets engine dynamically generates credentials for many databases.

To learn more, see these links:
[ https://www.vaultproject.io/docs/secrets/databases/](https://www.vaultproject.io/docs/secrets/databases/)
 [https://www.vaultproject.io/docs/secrets/databases/mysql-maria/](https://www.vaultproject.io/docs/secrets/databases/mysql-maria/)


 ## Enable the Database Secrets Engine

 first check the current secrets list 
 `vault secrets list`

let's enable database secrets engine 
`vault secrets enable -path=lob_a/workshop/database database`


## Configure the Database Secrets Engine


we need to configure database secrets engine to use mysql database plugin. 
We are configuring a database connection called "wsmysqldatabase" that is allowed to use two roles that we will create below.

```
vault write lob_a/workshop/database/config/wsmysqldatabase \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(localhost:3306)/" \
  allowed_roles="workshop-app","workshop-app-long" \
  username="babak" \
  password="Password123"

``` 
to verify the configuration you can use the command below
`vault read lob_a/workshop/database/config/wsmysqldatabase`

if you want to secure the password you can use the command below

`vault write -force lob_a/workshop/database/rotate-root/wsmysqldatabase`

We can make the configuration of the database secrets engine even more secure by rotating the root credentials (actually just the password) that we passed into the configuration.

### create a role 1
```
vault write lob_a/workshop/database/roles/workshop-app-long \
  db_name=wsmysqldatabase \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT ALL ON my_app.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"
  
  ```

### create a role 2
  ```
vault write lob_a/workshop/database/roles/workshop-app \
  db_name=wsmysqldatabase \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT ALL ON my_app.* TO '{{name}}'@'%';" \
  default_ttl="3m" \
  max_ttl="6m"
  
  ```


  ok now you have two roles. you can use the dynamic credentials with the command below
  `curl --header "X-Vault-Token: root" "http://localhost:8200/v1/lob_a/workshop/database/creds/workshop-app-long" | jq `

  you will get a json and in the data index you will find a username and password. sounds great!

  you can get user pass based on the same role that you did curd. with the command below
  `vault read lob_a/workshop/database/creds/workshop-app-long`

## Renew and Revoke Database Credentials

first create a credentials with the following command
`vault read lob_a/workshop/database/creds/workshop-app`

if you want to increment the lease time you can go with the command below
`vault write sys/leases/renew lease_id="<lease_id>" increment="120"`
be sure that your increments needs to be lower than max_ttl 

to examine the current lease with a command like this:
`vault write sys/leases/lookup lease_id="<lease_id>"`



### revoke the credentials
to revoke the credential you can use the command below
`vault write sys/leases/revoke lease_id="<lease_id>" `


## configure Vault and nodejs backend 
[https://dzone.com/articles/managing-secrets-in-nodejs-with-hashicorp-vault](https://dzone.com/articles/managing-secrets-in-nodejs-with-hashicorp-vault)