# Vault Cheat Sheet by Babak Doraniarab
## The Vault CLI
`vault version`
`vault` use it to see the list of Vault Cli commands

## vault on dev
you can run vault on a dev mode with this command and it will listen on all ips and with your token id.
`vault server -dev -dev-listen-address=0.0.0.0:8200 -dev-root-token-id=root`

`vault kv put secret/my-first-secret age=32`

## vault API 
`curl http://localhost:8200/v1/sys/health | jq`


to retrieve your 'my-first-secret' that you created before you can use the following command
`curl --header "X-Vault-Token: root" http://localhost:8200/v1/secret/data/my-first-secret | jq`

## run a production server
first you need to run the server with the corresponding config file 
`vault server -config=/vault/config/vault-config.hcl`
then you have to initialize the server in another terminal 
`vault operator init -key-shares=1 -key-threshold=1`

you need to save your root token and create an environment variable
`export VAULT_TOKEN=<root_token>`

being sure to use your own root token instead of <root_token>.
Please also add this to your ".profile" file with this command:

`echo "export VAULT_TOKEN=$VAULT_TOKEN" >> /root/.profile`

You next need to unseal your Vault server, providing the unseal key that the "init" command returned:
`vault operator unseal`

`vault status`

## Use the KV V2 Secrets Engine


to enable KV v2 secrets engine you need to run the following command

`vault secrets enable -version=2 kv`

Next, write a secret to your new secrets engine:

`vault kv put kv/a-secret value=1234`


## basic authentication

now you have a vault on production and you have KV v2 secret engine mounted ,it's time to enable `userpass` for authentication

`vault auth enable userpass`
you can verify it with the following command

`vault auth list`

Next, add yourself as a Vault user without any policies:

`vault write auth/userpass/users/babak password=babakpassword`

now go to UI and you can login with babak and babakpassword.

also, if you want to go and login in CLI with userpass, you need to use the following command

`vault login -method=userpass username=<name> password=<pwd>`

for me it would be like this
`vault login -method=userpass username=babak password=babakpassword`

when you use this command you will get a warning and it says, you have got a new token for this username and password and you need to unset current token or set it to new token. 
I'm going to set new token on VAULT_TOKEN 

to unset a VAULT_TOKEN use following command 
` unset VAULT_TOKEN `
and set new token to VAULT_TOKEN
` export VAULT_TOKEN=<new token > `


### tip 
if you want to confirm that your new token is being used, run this command 

` vault token lookup ` 
You will see that the display_name of the current token is "userpass-<name>" where <name> is your username and that the only policy listed for the token is the "default" policy.

Try to read the secret you wrote to the KV v2 secrets engine in the last challenge:


# Vault Policies

now we have userpass enabled and we can add policies to give different users access to different secrets.

let's create another userpass 

` vault write auth/userpass/users/user2 password=password2 `

you can list all the users in the userpass path with the following command
`vault list auth/userpass/users`

now we need to create policy files for each user.

### user-1-policy.hcl 
```
path "kv/data/<babak>/*" {
  capabilities = ["create", "update", "read", "delete"]
}
path "kv/delete/<babak>/*" {
  capabilities = ["update"]
}
path "kv/metadata/<babak>/*" {
  capabilities = ["list", "read", "delete"]
}
path "kv/destroy/<babak>/*" {
  capabilities = ["update"]
}

# Additional access for UI
path "kv/metadata" {
  capabilities = ["list"]
}
```

### user-2-policy.hcl 
```
path "kv/data/<user2>/*" {
  capabilities = ["create", "update", "read", "delete"]
}
path "kv/delete/<user2>/*" {
  capabilities = ["update"]
}
path "kv/metadata/<user2>/*" {
  capabilities = ["list", "read", "delete"]
}
path "kv/destroy/<user2>/*" {
  capabilities = ["update"]
}

# Additional access for UI
path "kv/metadata" {
  capabilities = ["list"]
}
```


Now we need to create this two policies 
```
vault policy write <user_1> /vault/policies/user-1-policy.hcl
vault policy write <user_2> /vault/policies/user-2-policy.hcl

#for me it would be : 
vault policy write babak /vault/policies/user-1-policy.hcl
vault policy write user2 /vault/policies/user-2-policy.hcl

```
ok we have two policies right now, babak and user2 and both of them are connect to their hcl policy file.

Now, you can assign the new policies to the users by updating the policies assigned to the users:

```
vault write auth/userpass/users/babak/policies policies=babak
vault write auth/userpass/users/user2/policies policies=user2


```
