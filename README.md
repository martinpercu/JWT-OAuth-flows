# JWT-OAuth-flows

## A Deep Dive into OAuth 2.0 and OpenID Connect: Understanding Authentication Flows, Practical Use Cases, and Implementing JSON Web Tokens (JWTs)

------------

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



## 04-authorization-code-flow
## Spotify Auth Flow
- In Spotify developers Create an "App". In .env use the ==>
```
SPOTIFY_CLIENT_ID=8a50f8bdbd2c4afe9947d236b6632801
SPOTIFY_CLIENT_SECRET=#####################
REDIRECT_URI=http://127.0.0.1:3002/api/callback
```
- This is a Next.Js app with a backend. Important as this is a SPA with Backend ==> We can user the Authorization Code Flow.
- The pages/index.js is just a button to /api/login 
- the pages/api/login.js is code in the serve side. We ask in this example 3 scopes ==>
```
const scopes = [
    "user-read-private",
    "user-read-email",
    "playlist-read-private",
  ];
```
- The query is a combination of differents parameters que Authorization server needs.
```
  const query = querystring.stringify({
    response_type: "code",
    client_id: process.env.SPOTIFY_CLIENT_ID,
    scope: scopes.join(" "),
    redirect_uri: process.env.REDIRECT_URI,
  });
```
- With this the user is doing the "Authorization Grant". Now the Authorization Code is going to the /callback
- In the pages/api/callback.js a fetch to SPOTIFY_TOKEN_URL. Something like this ==>
```
const response = await fetch(SPOTIFY_TOKEN_URL, options);
```
- The option is the very important. Also the body. Here we will exchange a code for a token ==>
```
const clientAuth = encode(`${process.env.SPOTIFY_CLIENT_ID}:${process.env.SPOTIFY_CLIENT_SECRET}`)
const options = {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
    Authorization: `Basic ${clientAuth}`,
  },
  body: querystring.stringify({
    code: req.query.code,
    redirect_uri: process.env.REDIRECT_URI,
    grant_type: "authorization_code",
  })
};
```
- With this now should redirect to home BUT still we don't see the date of user. The Authorization server just send the Access Token but we need to send the request with the token to the Resourse Server (Spotify) to get the user data.
- In callback.js with the {data} we will set a cookie==> (important for security HttpOnly to be use only in the backend)
```
res.setHeader("Set-Cookie", `access_token=${data.access_token}; Path=/; HttpOnly`);
```
- In home to use the cookie ==>
```
const SPOTIFY_ME_ENDPOINT = "https://api.spotify.com/v1/me"; // api with user data
const SPOTIFY_PLAYLISTS_ENDPOINT = (userId) =>
  userId ? `https://api.spotify.com/v1/users/${userId}/playlists` : null;

export async function getServerSideProps({ req }) {
  const data = cookie.parse(req.headers.cookie || "");
  return { props: { accessToken: data?.access_token } };
}

export default function Home({ accessToken }) {
  const { response: userProfile } = useFetch(SPOTIFY_ME_ENDPOINT, accessToken);

  const playlistUrl = SPOTIFY_PLAYLISTS_ENDPOINT(userProfile?.id);
  const { response: userPlaylists } = useFetch(playlistUrl, accessToken);

  return (
    <MainContainer>
      <Card userPlaylists={userPlaylists} userProfile={userProfile} />
    </MainContainer>
  );
}
```
- Home() ==> - 1st Fetch to spotify to the userProfile ===>  
- Home() ==> - 2nd with userProfile.id to get the playlistUrl ===>
- Home() ==> - 3rd with playlistUrl Fetch to get the userPlayList ===>
- Home() ==> - 4th to Card with userPlaylists + userProfile



## 05-authorization-code-flow-with-pkce
## X (ex Twitter) Auth Flow
- In https://developer.x.com create an APP (if needed create a dev account)
- In APP setting go to ==> User authentication settings and SET UP.
- In Set UP ==> "App permissions" ==> READ
- In Set UP ==> "Type of App" ==> Native App
- In Set UP ==> "App info" Native App
- In Set UP ==> "App info" Callback ==> "localhost:3003/api/callback"
- In Set UP ==> "App info" Website URL ==> "any real website" (Dont care)
- Save it. Then will show 2 id ==> "Client ID" and "Client Secret". Copy then and keep it .env
- In pages/api/login.js ==>
```
const TWITTER_AUTH_URL = "https://twitter.com/i/oauth2/authorize";
```
- In the function handler set the "scope" and VERY IMPORTANT the "state" is a unique state we keep during the session. (import the randomString to generate the state).
- Also we need for the PKCE the codeVerifier and de codeChallenge (generateCodeChallenge is a typical hashing function, imported from utils)
```
const scopes = ["users.read", "tweet.read"];
const state = randomString.generate(16);

const codeVerifier = randomString.generate(128);
const codeChallenge = generateCodeChallenge(codeVerifier)
```
- In the query we add the scope, the state, the code_challenge and code_challenge_method 
```
const query = querystring.stringify({
    response_type: "code",
    client_id: CLIENT_ID,
    scope: scopes.join(" "),
    redirect_uri: REDIRECT_URI,
    state,
    code_challenge: codeChallenge,
    code_challenge_method: "S256"
  });
```
- VERY IMPORTANT!! Add the cookies to compare after!!!
```
res.setHeader("Set-Cookie", [
  `state=${state}; Path=/; HttpOnly`,
  `verifier=${codeVerifier}; Path=/; HttpOnly`
]);
```
- Now pages/api/callback.js 
- First verify is the state is the same. (Actually if is NOT the same, to throw the error)
```
  if (req.query.state !== cookies.state) {
    res.writeHead(302, { Location: "/#?error=ERROR_STATE_MISMATCH" });
    res.end();
  }
```
- Now we must make the change with the code received from server to get the token. Now we add the code_verifier (remember in cookie)
```
const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: querystring.stringify({
      code: req.query.code,
      redirect_uri: REDIRECT_URI,
      grant_type: "authorization_code",
      client_id: CLIENT_ID,
      code_verifier: cookies.verifier,
    }),
  };
```
- Then the request to Twitter URL asking the token ==> const TWITTER_TOKEN_URL = "https://api.twitter.com/2/oauth2/token"; ===> this will return the token. This tokek we save in a cookie as acces_token. The got to /home
```
export default async function handler(req, res) {
  const cookies = cookie.parse(req.headers.cookie);

  if (req.query.state !== cookies.state) {
    res.writeHead(302, { Location: "/#?error=ERROR_STATE_MISMATCH" });
    res.end();
  }

  const options = {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: querystring.stringify({
      code: req.query.code,
      redirect_uri: REDIRECT_URI,
      grant_type: "authorization_code",
      client_id: CLIENT_ID,
      code_verifier: cookies.verifier,
    }),
  };

  try {
    const response = await fetch(TWITTER_TOKEN_URL, options);
    const data = await response.json();

    res.setHeader(
      "Set-Cookie",
      `access_token=${data.access_token}; Path=/; HttpOnly`
    );

    res.writeHead(302, { Location: "/home" });
    res.end();
  } catch (error) {
    console.error(error);
  }
```
- VERY IMPORTANT. Twitter us CORS so not possible to make request from the client. To avoid this I have and endpoint cors.js. This will receive the url directly (of course I'm sending the access_token). Is like a middleware to "bridge" the Cors policy. and be able to make request from localhost.
- Now pages/api/home.js I use the cors endpoint to get user info to show in the card.



## 06-implicit-flow
## Twitch
- In https://dev.twitch.tv/console create an APP
- In Register Your Application ==> "Name" ==> "any name you liki it"
- In Register Your Application ==> "OAuth Redirect URLs" ==> "localhost:3004/callback"
- In Register Your Application ==> "Category" ==> "Website integration"
- In Register Your Application ==> "Client Type" ==> "Confidential"
- Now from manage copy the Client ID to use in .env
- In src/App.jsx form here will be done the request. In the MainContainer the button will got to "authUrl"
- This URL we must to "build" =="const authUrl = useAuthUrl();"
- The useAuthUrl() is a React hook. This is +or- the same as before (AUTH_URL+CLIENTE_ID+SCOPES etc etc.)
- IMPORTANT in the state (not safe cookie from server). So we will persist in the sessionStorage. 
```
  const state = useSession("state")
```
- IMPORTANT ==> The authorization server when receive the request will redirect with the access token in the fragment. So we must take it from there. So as this is a React app we will the the access tomken from the hash fragment. In App.jsx use the utility getHashParams (src/utils/getHashParams.js) ==>
```
  const queryAccessToken = getHashParams("access_token");
```
- VERY IMPORTANT Avoid attacks Cross-Site Request Forgery. ==>
```
  const queryState = getHashParams("state");
  const sessionState = sessionStorage.getItem("state");
```
- Check if they are the same and also we have the queryAccessToken This validation should be done when we have the access token. Is not throw error ==>
```
  if (queryState !== sessionState && queryAccessToken) {
    console.error(new Error("ERROR_STATE_MISMATCH"));
    cleanUrlHash();
    return setAccessToken(null);
  }
```
- Then if all OK the access token will be set in the state ==>
```
  if (queryAccessToken) {
    setAccessToken(queryAccessToken);
    cleanUrlHash();
  }
```
- Now we have what we need to call Twitch endpoints
```
  const { response: userProfile } = useFetch(TWITCH_USERS_URL, accessToken);
  const userChannelsUrl = TWITCH_CHANNELS_URL(userProfile?.data[0].id);
```
- Conclusion & Important this flow is not reccomended for SPA. Is important at least delet the hash in the redirect from server. NEVER NEVER keep the access token in localStorage or this kind of implementations.








## 07-client-credentials-flow
## Discord
- In https://discord.com/developers create an APP
- GO To tab ==> OAuth2 
- In General Information of Application ==> "Copy Client ID" ==> "save in .env"
- In General Information of Application ==> "Copy Client Secret" ==> "save in .env"
- This is a basic node.js with express server.
- In the index.js the url will be consumed before and after get token ==> 
```
const DISCORD_TOKEN_URL = "https://discord.com/api/oauth2/token";
const DISCORD_USER_URL = "https://discord.com/api/users/@me";
const DISCORD_GUILDS_URL = "https://discord.com/api/users/@me/guilds";
```
- The Scopes ==>   const scopes = ["identify", "guilds"];
- In the body the new is just the "client_secret"
```
const options = {
  method: "POST",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
  },
  body: querystring.stringify({
    grant_type: "client_credentials",
    client_id: CLIENT_ID,
    scope: scopes.join(" "),
    client_secret: CLIENT_SECRET,
  }),
};
```
- Then with the access token we make the fetch to the userData and guildData.
- The fetchWithToken() is just a fetch adding headers Authorization Bearer with the accesToken ==>
```
export const fetchWithToken = async (url, accessToken) => {
  try {
    const response = await fetch(url, {
      headers: {
        Authorization: `Bearer ${accessToken}`,
      },
    });
    return await response.json();
  } catch (error) {
    console.error(error);
  }
};
```
- Now check the http://localhost:3005 you shoul see the json of Discord user.
- This is just the basic connesion used machine to machine.

- 

- 

 In General Information of Application ==> "Client Type" ==> "Confidential"
- Now from manage copy the Client ID to use in .env