# Vault Policies 
use the command below to get help for the syntax of Vault policies
`vault policy write -h`

to see the current policies 
`vault policy list`

you can create policy in two different ways
## first way
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

## Second way to create a policy
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

