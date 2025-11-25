# JWT-OAuth-flows


## 01-preview-endpoint
- In folder there are the index.js with 3 endpoints (public, private and token). 
- For each folder there are the environment variable .env
- Step by step I will add features to show how OAuth could be implemented.
- Important the endpoint token ask for "getCredentials" this is in headers.js. This method take header use for authorization and then validate if OK and return { username, password }.
- Then /token with this user = username, password generate the token and will send it.
- Then /private receive the token and make the validations. If OK return "I'm private"
- To test this in Bruno Postman or Insomnia
- POST to /token with "Basic Auth" username and password the same in the .env ==> will return the token
- GET to /private with "Bearer Token" add the token. If token OK return "I'm private"
- This is AUTENTICATION not authorization!

## 02-sign-and-verifying - Symetric
- The Json web tokens have 3 parts joined by 2 dots ==> aaaa.bbbb.cccc aaaa is the header bbbb is the payload and cccc y the signature.
```
example header 
{
  "alg": "HS256",
  "typ": "JWT"
}

example payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
- The signature will be 
```
HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret )
```
- The server use this signature to verify is its real because the client doesn't know the secret. This is the magic
- Install jsonwebtoken 
```
npm install jsonwebtoken
```
- In token.js import it
```
import jwt from "jsonwebtoken";
export const signToken = (user) => {
  const payload = {
    sub: user.id,
    name: user.fullname,
    exp: Date.now() + ONE_MINUTE_IN_MS,
  };

  return jwt.sign(payload, SECRET);
};
```
- The jwt will sign using the SECRET. The .env has the SECRET. 
- In /private const payload = verifyToken(token) use the method in token.js return
```
return jwt.verify(token, SECRET);
```
- Now we can use Postman or Bruno to check this 
- POST to /token with "Basic Auth" username and password the same in the .env ==> will return the token
- GET to /private with "Bearer Token" add the token. If token OK return "I'm private"
- This is AUTENTICATION not authorization!
- This is a symetric signature.
## 02-sign-and-verifying - Asymetric
- Using the script keypair.sh to generate 2 key. Once run the code will be generated private.pem and public.pem. This are the 2 keys paired in PKCS8 (Public Key Cryptographic standard N°8).
- Now in token.js in signToken() method we add:
```
import fs from "node:fs";
const PRIVATE_KEY_PATH = process.env.PRIVATE_KEY_PATH

export const signToken = (user) => {
  const payload = {
    sub: user.id,
    name: user.fullname,
    exp: Date.now() + TWO_MINUTE_IN_MILLISECONDS,
  };

  if(PRIVATE_KEY_PATH) {
    const privateKey = fs.readFileSync(PRIVATE_KEY_PATH, 'utf8');
    return jwt.sign(payload, privateKey, { algorithm: 'RS256' } )
  }

  return jwt.sign(payload, SECRET)
};
```
- Now with at /token we get a new token using RS258 with the private key. When we use this token in /private should return { "error": "invalid algorithm" } because the verifyToken() is not ready to receive this kind of token. So in verifyToken ==>
```
const PUBLIC_KEY_PATH = process.env.PUBLIC_KEY_PATH

export const verifyToken = (token) => {
  if(PUBLIC_KEY_PATH) {
    const publicKey = fs.readFileSync(PUBLIC_KEY_PATH, 'utf8');
    return jwt.verify(token, publicKey);
  }
  return jwt.verify(token, SECRET);
};
```
- Now we can use Postman or Bruno to check this 
- POST to /token with "Basic Auth" username and password the same in the .env ==> will return the token
- GET to /private with "Bearer Token" add the token. If token OK return "I'm private"
- This is a Asymetric signature. Using PUBLIC and PRIVATE KEYS


## 03-resource-owner-password-flow 

## Oauth 2.0 roles
### 1 Client
- Could be any app. Mobile App even a desctop app. 
### 2 Authorization Server
- This is the API that will generate and verified the token and show the permitions for.
### 3 Resource Owner
- This is the user itself. The user is who will click the authorization of permit
### 4 Resourse Server
- Server that will use the tokens (most of time JWT).

## Oauth 2.0 Terms
### Redirect URI
- The address to redirect. 
### Consent Request
- The page the user see to know what permits is asking for.
### Scopes
- The permits we will grant to the user
### Authorizatin Grant
- The code receive by the Authorization server once the user accept the scopes.

## Oauth 2.0 Terms 2 
### Client ID 
- This is like the user and password from the client. In this protocol is important the user to be register by the authentication server.
### Client Secret 
- This is like the user and password from the client. In this protocol is important the user to be register by the authentication server.
### Access Token 
- The token to accest to resources.
### ID Token
- Is a special token that works in Open ID connect flow.

## Authorization Code Flow
- This is the standard

## Authorization Code Flow with PKCE (Proof Key Code Exchange)
- This flow will add a Code Verifier + Code Challenge
#### Code Verifier sdeZEsdfEDF
#### + 
#### Code Challenge method SHA256
#### =
#### Code Challenge sdfueZEGsdfes(...)
- This is usefull when clients cannot keep secret in a safe way. Like single page or native apps etc.
- This is the hashing and is not reversible.
- As example. The client send to server the "Code Challenge Method" with the "Code Challenge". THEN after the client send the "Code Verifier". Now the server will verify the info. With this the server knows that the client is all time the same.  
## Client Credential Flow
- This has no Resource Owner (no user). There is a Client Authentication from the Client to the Authentication server. The Cliente receives the Accest Toke. The Client make the request to Resourse Server with the token.
- This runs in a secure environment.

## Resourse Owner Password Flow
- This is really legacy. Is use when the app cannot make a redirect. This is the same as Client Credential Flow BUT is the Resourse Owner (the user) who ask for connect sending user+password.

## Implicit Flow
- This is not the best practice. The Authorization server will send the Access token just using client ID match with the Authorization request. 

## How to choose the correct FLOW ??
### Use in order to know what is the correct FLOW
- If the Client is Resource Owner ==> Client Credentials Flow
- If the Client is a web app running in the server ==> Authentication Code Flow
- If we trust the Client the user credentials ==> Resourse Owner Password Flow
- If the Client is a Native/Mobile/SPA App ==> Authentication Code Flow with PKCE
- If the Client is a Native/Mobile/SPA App ==> 2nd option Implicit Flow with Form Post (only works with Open Id Connect OIDC) ==> 3rd option Hibrid Flow ==> the last option is Implicit Flow (the secure implementation is not trivial)
