# Vault Policies 
use the command below to get help for the syntax of Vault policies

`vault policy write -h`

to see the current policies 

`vault policy list`

you can create policy in two different ways
### first way
 create a .hcl policy file and then write it into vault 

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

then you need to create the policy with the below command
`vault policy write <user_2> /vault/policies/user-2-policy.hcl`

### Second way to create a policy
you can write all the policy when you are trying to write it on Vault 
```
vault policy write my-policy - << EOF
# Dev servers have version 2 of KV secrets engine mounted by default, so will
# need these paths to grant permissions:
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF

```

# Verify your policy
I've used second way to create a policy and following commands are based that configuration.


create a token with the new policy the

`export VAULT_TOKEN="$(vault token create -field token -policy=my-policy)"`
you can verify it with `vault token lookup`

then create a new kv 

`vault kv put secret/creds password="my-long-password"`

try the command below and you will see you don't have enough permission to create on the pass foo
but you can do create and update on other paths

`vault kv put secret/foo robot=beepboop`


# Associate Policies to Auth Methods
first we need to check the current authentication methods list

`vault auth list `

let's enable approle auth methods 

`vault auth enable approle`

then we need to create a role 

```
vault write auth/approle/role/my-role \
   secret_id_ttl=10m \
   token_num_uses=10 \
   token_ttl=20m \
   token_max_ttl=30m \
   secret_id_num_uses=40 \
   token_policies=my-policy

```
To authenticate with AppRole, first fetch the role ID, and capture its value in a ROLE_ID environment variable.

‍‍`export ROLE_ID="$(vault read -field=role_id auth/approle/role/my-role/role-id)"`

Next, get a secret ID (which is similar to a password for applications to use for AppRole authentication), and capture its value in the SECRET_ID environment variable.

`export SECRET_ID="$(vault write -f -field=secret_id auth/approle/role/my-role/secret-id)"`

Finally, authenticate to AppRole with vault write by specifying the role path and passing the role ID and secret ID values with the respective options.
`vault write auth/approle/login role_id="$ROLE_ID" secret_id="$SECRET_ID"`