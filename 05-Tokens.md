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

## How orphan tokens work.
when you create a token and create another token with the parent token , something will happens here.
you have a parent and a child and if you revoke the parent the child token will not work anymore and this is the default behavior.
let's try it

parent token 
```
vault token create -ttl=120s \
      -format=json | jq -r ".auth.client_token" > parent_token.txt
```

child_token
```
VAULT_TOKEN=$(cat parent_token.txt) vault token create -ttl=80s \
      -format=json | jq -r ".auth.client_token" > child_token.txt

```
Try running some commands using this child token
`vault token lookup $(cat child_token.txt)`
let's Revoke the parent token.
`vault token revoke $(cat parent_token.txt)`

now let's try some commands using the child token once again 
`vault token lookup $(cat child_token.txt)`

now you see , it's not working any more 

but what is the solution !? 

### orphan tokens
let's repeat the steps and create a parent token 
```
vault token create -ttl=60s \
      -format=json | jq -r ".auth.client_token" > parent_token.txt 

```
>Orphan tokens are not children of their parent; therefore, orphan tokens do not expire when their parent does.

>NOTE: Orphan tokens still expire when their own max TTL is reached.

let's create an Orphan token with the parent token 

```
VAULT_TOKEN=$(cat parent_token.txt) vault token create -ttl=180s -orphan \
      -format=json | jq -r ".auth.client_token" > orphan_token.txt
```

Time to test 
revoke the parent token `vault token revoke $(cat parent_token.txt)`
check the orphan token `vault token lookup $(cat orphan_token.txt)`
it's working.

