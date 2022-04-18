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