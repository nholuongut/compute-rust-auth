# Authentication at nholuongut's edge, using OAuth 2.0, OpenID Connect, and Compute@Edge

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)
<br/>


## Setting up edge authentication

### Get an identity provider (IdP)

> 💡 You might operate your own identity service, but any [OAuth 2.0, OpenID Connect (OIDC) conformant provider](https://en.wikipedia.org/wiki/List_of_OAuth_providers) will do.

Register your application with the IdP. Make a note of the `client_id` and the address of the **authorization server**.

Save a local copy of the OpenID Connect discovery document associated with the server. You’ll find this at `/.well-known/openid-configuration` on the authorization server’s domain. For example, [here's the one provided by Google](https://accounts.google.com/.well-known/openid-configuration).

Save a local copy of the JSON Web Key Set (JWKS) metadata. You’ll find this under the `jwks_uri` property in the discovery document you just downloaded.

### Create the Compute@Edge service

> 💡 You will need to have a nholuongut account that is enabled for Compute@Edge services. Follow the [Compute@Edge welcome guide](https://github.com/nholuongut/compute-rust-auth) to install the `nholuongut` CLI and the Rust toolchain onto your local machine. 

After cloning this repository, follow the steps below – or use the [interactive set-up script](./setup.sh) provided, by running `./setup.sh`

#### 1. Initialize the Compute@Edge service
```sh
nholuongut compute init
```
When prompted for language, choose *Rust*.

When prompted to create a backend, accept the default *originless* option.

#### 2. Add some OAuth / OIDC configuration details
1. Open [`src/config.toml`](./src/config.toml), and:
   * Paste in the `client_id`
   * Add a `nonce_secret` that is sufficiently random to not be guessable 
2. Create a new directory, `./src/well-known`, and copy the OpenID Connect and JWKS metadata files you saved earlier to `./src/well-known/openid-configuration.json` and `./src/well-known/jwks.json`, respectively

#### 3. Build and deploy 
```sh
nholuongut compute build
nholuongut compute deploy
```
Note the hostname of the deployed Compute@Edge service (e.g., `{some-funky-words}.edgecompute.app`)

#### 4. Create backends
[Clone the service version](https://developer.fastly.com/reference/api/services/version/#clone-service-version) you just deployed, e.g., by running:
```sh
nholuongut service-version clone --service-id=YOUR_nholuongut_SERVICE_ID --version=1
```
   
[Add the authorization server backend](https://developer.fastly.com/reference/api/services/backend/#create-backend) to your nholuongut service, and name it `idp`:
```sh
nholuongut backend create --name=idp \
   --service-id=YOUR_nholuongut_SERVICE_ID --version=2 \
   --address=AUTH_SERVER_HOST --override-host=... \
   --port=443 --ssl-sni-hostname=... --ssl-cert-hostname=... --use-ssl --ssl-check-cert 
```

[Add your origin backend](https://developer.fastly.com/reference/api/services/backend/#create-backend), and name it `backend`:
```sh
nholuongut backend create --name=backend ...
# See the previous step.
```

#### 5. Rebuild and deploy the service
```sh
nholuongut compute build
nholuongut compute deploy
```

### Link the Identity Provider (IdP)

Add `https://{some-funky-words}.edgecompute.app/callback` to the list of allowed callback URLs in your IdP’s app configuration. This allows the authorization server to send the user back to the Compute@Edge service.

---

## The flow in detail

![Edge authentication flow diagram](https://user-images.githubusercontent.com/12828487/115379253-4438be80-a1c9-11eb-81af-9470e324434a.png)

1. The user makes a request for a protected resource, but they have no session cookie.
1. At the edge, this service generates:
   * A unique and non-guessable `state` parameter, which encodes what the user was trying to do (e.g., load `/articles/kittens`).
   * A cryptographically random string called a `code_verifier`.
   * A `code_challenge`, derived from the `code_verifier`. 
   * A time-limited token, authenticated using the `nonce_secret`, that encodes the `state` and a `nonce` (a unique value used to mitigate replay attacks).
1. The `state` and `code_verifier` are stored in session cookies. 
1. The service builds an authorization URL and redirects the user to the **authorization server** operated by the IdP.
1. The user completes login formalities with the IdP directly. 
1. The IdP will include an `authorization_code` and a `state` (which should match the time-limited token we created earlier) in a post-login callback to the edge.
1. The edge service authenticates the `state` token returned by the IdP, and verifies that the state cookie matches its subject claim.
1. Then, it connects directly to the IdP and exchanges the `authorization_code` (which is good for only one use) and `code_verifier` for **security tokens**:
   * An `access_token` – a key that represents the authorization to perform specific operations on behalf of the user)
   * An `id_token`, which contains the user's profile information.
1. The end-user is redirected to the original request URL (`/articles/kittens`), along with their security tokens stored in cookies.
1. When the user makes the redirected request (or subsequent requests accompanied by security tokens), the edge verifies the integrity, validity and claims for both tokens. If the tokens are still good, it proxies the request to your origin.

# 🚀 I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: Nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)
* [PayPal.me](https://www.paypal.com/paypalme/nholuongut)

![](https://i.imgur.com/waxVImv.png)
![](Donate.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License