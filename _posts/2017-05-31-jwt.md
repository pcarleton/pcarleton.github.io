---
layout: post
title: "JWT's From Third Party Authentication Providers"
---

##  JWT’s in Google Sign-in
Last time [I wrote about Google Sign-In][last-post], I discussed the details of why you would want google-sign-in vs. using OAuth to grant you access to a person’s account.  I also discussed robot-accounts vs. using individual access tokens.  However, I did not go deep into how the sign-in flow works partially because I didn’t have to; The python google api bindings took care of most things which meant I did not need to think very hard about what was happening to authenticate.  The story is not as nice in Go since the bindings were missing a critical function “validate_token”.  To get that functionality working in Go, I dove into how it worked in python which led me into the details of the JSON Web Token (JWT) specification.  In this post, I’ll go into what I learned about JWT’s and in a follow up post, I’ll show the code I used in Go to validate the tokens.


## JWT’s in the Wild
I had heard about JWT’s with regards to authenticating with [gspread](http://gspread.readthedocs.io/en/latest/oauth2.html) where it talks about SignedJwtAssertionCredentials.  This was one of those scenarios where I said “eww gross” and didn’t try to figure out what that variable name meant because the library handled the details for me.

Reading around about JWT’s, you see they are very divisive.  [Some](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/) [people][jwt-blog] feel very strongly that you should not use a JWT to authenticate.
One recent post compares JWT’s to old train links that caused thousands of people to die! Talk about strong feelings!  I will try to dissect what a JWT is so we can undertand where those feelings come from.

## Okay, what’s a JWT?
A JSON Web Token is a data format which allows its author to be verified.  Basically, you can get a JWT from server A, bring it to server B and server B can validate that it did indeed come from server A and that it’s contents are unmodified.

If you’ve read my post on sessions, this may sound familiar.  It is similar to the HMAC idea I used for making secure cookies.  The difference from the cookie implementation I was discussing in that post is that a JWT can be signed with a private key and be verified with a public key (i.e. RSA encryption)  which means that server A does not have to share any secrets with server B in order for server B to verify the token.  The HMAC cookie I was referring to requires the verifier to be in possession of the secret.

[Jwt.io](https://jwt.io) has a some good resources explaining how they work (It’s hosted by Auth0 which sells a product based on JWT’s).

#### Why is it so divisive?
It seems to mainly stem from the JWT’s spec complexity.  This leads to 1) people using them when a much simpler solution would as well if not much better and 2) people incorrectly implementing the spec leading to security holes which violate the whole point of using a JWT in the first place.  The JWT spec is relatively new (it came out in 2015), so it seems like there was a lot of excitement about the new spec which may have led to over/misusage.

## JWT in practice The Sign-In Flow
Now I’ll look at how JWT’s are relevant to the Google Sign-In flow. First, I click the button and a request is sent to Google.  Simultaneously with this request, a dialog is loaded asking me to authorize the application to sign in.  Once I authorize, the request sent with the initial button press succeeds and I receive a JWT from google.  Here is a crude diagram of what this flow looks like:

![Diagram of Client side sign-in flow][client-side-flow]{:style="margin-right: 7px;margin-top: 7px;"}

The JWT is now is in the JavaScript running in our browser, but our server knows nothing about the authentication that just happened.  The user has proven they own the email associated with the Google account, so can I just tell the server the account email they should be signed in as?

The problem is the server has no proof.  The client received this information from Google, so it trusts it, but the server is receiving it from a client which could easily be co-opted to skip the whole “talking to Google” part and make false claims about what email addresses it owns.

This is where the JWT comes in.  By sending the whole JWT (and not just the identifying info), the server can request Google’s public key and use it to verify that the JWT is legitimate.  Once it has been verified, it can match the Google provided identity with it’s own identity information and consider the user logged in, creating the session like before.

## What’s Inside?

Now it is worth asking, what does a JWT look like? The JWT spec is in [RFC-7519]( https://tools.ietf.org/search/rfc7519#section-3.1)  and also depends on the JWS (JSON Web Signature) spec which is in [RFC-7515](https://tools.ietf.org/html/rfc7515).  Here is a diagram of a JWT. (You can also see an example JWT the original ):


```
+------------------------------------+
|             JWT Claims             |
+------------------------------------+
| Base 64 encoding of:               |
| {"iss":"joe",                      |
|  "exp":1300819380,                 |
|  "http://example.com/is_root":true}|
+-------------+----------------------+
              |
              v
XXXXXXXXXXXXX.YYYYYYYYYYYY.ZZZZZZZZZZZZZZZZZZZZ
   ^                        ^
   |                        |
+-----+------------------+ +---+-----------------------------+
|       JOSE Header      | |Message Authentication Code (MAC)|
+------------------------+ +---------------------------------+
| Base 64 encoding of:   | |Result of putting Header and     |
| {type:"JWT",           | |claims through {alg} and Base64  |
|  alg:"{alg}"}          | |encoding the output.             |
+------------------------+ +---------------------------------+
```


## Go API Client: Missing some pieces
As I mentioned earlier, the Go api client was missing any kind of server-side JWT verification. The pieces needed were (1) fetching public keys from Google (2) parsing the JWT and (3) using the keys to verify the JWT.

Fortunately, there is a nice library for verifying and parsing JWT’s in Go called [jwt-go](https://github.com/dgrijalva/jwt-go), so it was only a matter of fetching the correct keys and calling the correct methods from the library (and making sure I called them correctly! At first, I shot myself in the foot just like the [the blog post above][jwt-blog] warned about by not checking if the token was “Valid” at the end.)

Here’s what the code looks like for parts 2 & 3 (part 1 fetches the `certs` map, and can be found in the [same file](https://github.com/pcarleton/cashcoach/blob/master/auth/auth.go#L13-L53) in my project):

```
func VerifyToken(tokenString string, certs map[string]*rsa.PublicKey) (*jwt.Token, error) {

	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		// Only accept RSA signed JWT's
		if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
			return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
		}

		keyID := token.Header["kid"].(string)
		rsaKey := certs[keyID]
		return rsaKey, nil
	})

	if err != nil {
		return nil, err
	}

	if !token.Valid {
		return nil, fmt.Errorf("Invalid token: %v", token)
	}

	return token, nil
}
```

The call to `Parse` is a little confusing at first.  The two arguments being passed in are the string of the JWT (like the `XX.YY.ZZ` in my previous diagram), and a [`Keyfunc`](https://godoc.org/github.com/dgrijalva/jwt-go#Keyfunc) function.  The `Parse` function parses the JWT into its components (i.e. base64 decodes the relevant pieces) and then uses the `Keyfunc` to decide which key to use to verify the JWT.  This is because often the JWT will supply a `kid` parameter in the JOSE header which specifies which key it was signed with.  When I fetch the public keys from Google, they are provided in a JSON object of the form `{ key_id: public_key }`.  The `Parse` function then uses the provided key to verify the JWT and returns it.

## Conclusion

In this post, I explained what a JWT is, how it is used, and showed how to verify a Google Sign-in JWT in Go.  In a follow-up post, I'll explain how to fit this Go server together with a JavaScript front-end.


[client-side-flow]:{{site.s3_base}}/jwt/client_side_flow.jpg
[jwt-blog]:https://kev.inburke.com/kevin/things-to-use-instead-of-jwt/
