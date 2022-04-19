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

## How to create periodic tokens and short-lived tokens

### Periodic Tokens
Periodic tokens have a TTL (validity period), but no max TTL; therefore, they may live for an infinite duration of time so long as they are renewed within their TTL. This is useful for long-running services that cannot handle regenerating a token.
> Root or sudo users have the ability to generate periodic tokens.


lets create a 24 hours ttl token 
```
vault token create -policy="default" -period=24h -format=json \
   | jq -r ".auth.client_token" > periodic-token.txt
```
we can review it with the following command
`vault token lookup $(cat periodic-token.txt)`

you can renew the generated token indefinitely for as long as it does not expire. if you dont renew, the token expires after 24 hours

### Short-lived Tokens

let's create a token with TTL of 40 seconds and save it a file named short-lived-token.txt
```
vault token create -ttl=40s  -format=json \
   | jq -r ".auth.client_token" > short-lived-token.txt

```
let's check it `vault token lookup $(cat short-lived-token.txt)`

run the command below one more time after 40 seconds and you will see the token is not valid any more . 

`vault token lookup $(cat short-lived-token.txt)`
you will get the code 403 

