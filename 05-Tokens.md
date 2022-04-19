# Tokens
to get help for tokens use this command `vault token create -h`


Create a token with TTL of 1 hour and a use limit of 2. Attach the default policy.
`vault token create -ttl=1h -use-limit=2 -policy=default`

if you want to validate it and store it in a separate file you can use the command below:
it store only the client token to the corresponding txt file.
```
vault token create -ttl=1h -use-limit=2 -policy=default \
   -format=json | jq -r ".auth.client_token" > use-limit-token.txt
```

but if you want to store all the info go with the following command
```
vault token create -ttl=1h -use-limit=2 -policy=default \
   -format=json | jq -r  > use-limit-token.txt
```
Use the token to retrieve its detail.
`VAULT_TOKEN=$(cat use-limit-token.txt) vault token lookup`

