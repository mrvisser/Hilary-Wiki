All authentication relies on [PassportJS](http://passportjs.org). Passport supports a lot of strategies out-of-the-box which can technically all be used.

So far the following strategies have been tested and are supported by the OAE team. All of them can be enabled or disabled at runtime:

## Local authentication

A user can authenticate by submitting a form that contains a username and password.

## CAS authentication

Users can login via CAS (Central Authentication Service). They will be redirected to the CAS login endpoint and when they return their ticket will be validated.

**Configuration options:** 

* Name
  * The name that regular users would recognize as their institutional Single Sign On service
  * e.g., GT Login
* Service
  * The full URI of the CAS server. This should include the protocol (https)
  * e.g., https://login.gatech.edu
* Base path (optional)
  * The base path where the CAS servlet handlers are tied to
  * e.g., /cas
* Login path
  * The path where the login servlet handler is registered on
  * The combination of the URI, Base path and Login path will be used to redirect users to
  * e.g., /login
  * The above configuration result in `https://login.gatech.edu/cas/login`
* Logout path
  * The path where the logout servlet handler is registered on
  * e.g., /logout
* Service validation path
  * The path that can be used to validate tickets
  * e.g., /serviceValidate

## Shibboleth authentication

Users can login on a Shibboleth Identity Provider

### Requirements

Although most of the other strategies require no additional work, Shibboleth requires some initial preparation.

* Java runtime
* The SAMLParser JAR
* You will need to specify the path to the JAR in the main config.js file
* an X509 certificate

You can generate an X509 certificate with the following commands:

```
cat > cert.cnf <<EOF
[req]
prompt=no
default_bits=2048
encrypt_key=no
default_md=sha1
distinguished_name=dn
string_mask=MASK:0002
x509_extensions=ext
[dn]
CN=oae.cam.ac.uk
[ext]
subjectAltName=DNS:oae.cam.ac.uk
subjectKeyIdentifier=hash
EOF
touch key.pem
chmod 600 key.pem
openssl req -config cert.cnf -new -x509 -days 365  -keyout key.pem -out cert.pem
```
Replace oae.cam.ac.uk with your domain.
 
**Configuration options (in the admin UI):**

* Name
  * The name that regular users would recognize as their institutional Single Sign On service.
  * e.g., Testshib
* IdP URL
  * The URL where users can be redirected to in order to start the SSO process.
  * This should be the endpoint where shibboleth supports the HTTP-REDIRECT format.
  * e.g., https://idp.testshib.org/idp/profile/SAML2/Redirect/SSO
* IdP Public key
  * The key that the IdP uses to sign SAML messages.
  * e.g.,

```
MIIEDjCCAvagAwIBAgIBADANBgkqhkiG9w0BAQUFADBnMQswCQYDVQQGEwJVUzEVMBMGA1UECBMMUGVubnN5bHZhbmlhMRMwEQYDVQQHEwpQaXR0c2J1cmdoMREwDwYDVQQKEwhUZXN0U2hpYjEZMBcGA1UEAxMQaWRwLnRlc3RzaGliLm9yZzAeFw0wNjA4MzAyMTEyMjVaFw0xNjA4MjcyMTEyMjVaMGcxCzAJBgNVBAYTAlVTMRUwEwYDVQQIEwxQZW5uc3lsdmFuaWExEzARBgNVBAcTClBpdHRzYnVyZ2gxETAPBgNVBAoTCFRlc3RTaGliMRkwFwYDVQQDExBpZHAudGVzdHNoaWIub3JnMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArYkCGuTmJp9eAOSGHwRJo1SNatB5ZOKqDM9ysg7CyVTDClcpu93gSP10nH4gkCZOlnESNgttg0r+MqL8tfJC6ybddEFB3YBo8PZajKSe3OQ01Ow3yT4I+Wdg1tsTpSge9gEz7SrC07EkYmHuPtd71CHiUaCWDv+xVfUQX0aTNPFmDixzUjoYzbGDrtAyCqA8f9CN2txIfJnpHE6q6CmKcoLADS4UrNPlhHSzd614kR/JYiks0K4kbRqCQF0Dv0P5Di+rEfefC6glV8ysC8dB5/9nb0yh/ojRuJGmgMWHgWk6h0ihjihqiu4jACovUZ7vVOCgSE5Ipn7OIwqd93zp2wIDAQABo4HEMIHBMB0GA1UdDgQWBBSsBQ869nh83KqZr5jArr4/7b+QazCBkQYDVR0jBIGJMIGGgBSsBQ869nh83KqZr5jArr4/7b+Qa6FrpGkwZzELMAkGA1UEBhMCVVMxFTATBgNVBAgTDFBlbm5zeWx2YW5pYTETMBEGA1UEBxMKUGl0dHNidXJnaDERMA8GA1UEChMIVGVzdFNoaWIxGTAXBgNVBAMTEGlkcC50ZXN0c2hpYi5vcmeCAQAwDAYDVR0TBAUwAwEB/zANBgkqhkiG9w0BAQUFAAOCAQEAjR29PhrCbk8qLN5MFfSVk98t3CT9jHZoYxd8QMRLI4j7iYQxXiGJTT1FXs1nd4Rha9un+LqTfeMMYqISdDDI6tv8iNpkOAvZZUosVkUo93pv1T0RPz35hcHHYq2yee59HJOco2bFlcsH8JBXRSRrJ3Q7Eut+z9uo80JdGNJ4/SJy5UorZ8KazGj16lfJhOBXldgrhppQBb0Nq6HKHguqmwRfJ+WkxemZXzhediAjGeka8nz8JjwxpUjAiSWYKLtJhGEaTqCYxCCX2Dw+dOTqUzHOZ7WKv4JXPK5G/Uhr8K/qhmFT2nIQi538n6rVYLeWj8Bbnl+ev0peYzxFyF5sQA==
```

* Service Provider Entity ID
  * The Entity ID that will be used to register OAE as a Service Provider with the IdP
  * It's usually a good idea to take the hostname of where the SP will be running and append /shibboleth to it
  * e.g., https://oae.cam.ac.uk/shibboleth (Assuming we're running OAE at oae.cam.ac.uk)
* Service Provider Certificate
  * An X509  public key (the base64 string from cert.pem without any spaces or line-breaks)
* Service Provider Certificate subject name
  * The subject name you used to create the certificate (the value you replaced oae.cam.ac.uk with in the above script)
* Service Provider Private Key
  * The corresponding X509 private key (the base64 string from key.pem without any spaces or line-breaks)
 
Once you've configured Shibboleth, you should be able to retrieve the necessary metadata (in XML format) at /api/auth/shibboleth/metadata. You can use this to register OAE as a service provider with your IdP.
(In case of testshib.org, it's this file that you should upload.) 
 
**Note:** Although some Shibboleth IdPs pass a bunch of attributes about the logged in users. These will not be used

## Google authentication

Let users authenticate with their Google account.

You will need to register a web application on the [Google API console](https://code.google.com/apis/console) (under API Access). Use the tenant host name to register your application with, you will need to create a key per tenant (e.g., "http://oae.oae-qa0.oaeproject.org").

After creating the web application make sure the path in the redirect url is set to "/api/auth/google/callback" (e.g., "http://oae.oae-qa0.oaeproject.org/api/auth/google/callback").

Once you've created your web application in the Google API Console, you can switch back to the OAE global administration panel and you'll be able to enable Google authentication. Go to  the configuration for your tenant, click modules, select the 'OAE Authentication module' and perform the following tasks in the form:

* Check 'Allow Google authentication for tenant' 
* Fill in the 'Google Client ID' (you can find it in the Google Console)
* Fill in the 'Google Client Secret' (you can find it in the Google Console)
* Click the Save configuration button.

Google authentication should now be enabled for your tenant.

## Twitter authentication

Let users authenticate with their Twitter account. You will need to register an application on the [Twitter Developer Website](https://dev.twitter.com/apps) and provide the consumer key and secret in the admin UI.

## Facebook authentication

Let users authenticate with their Facebook account. You will need to:

* register an application on the [Facebook Developer Website](https://developers.facebook.com/apps)
* Set the app domains to something like 'oae-qa0.oaeproject.org'
* Enable the 'Website with Facebook Login' check
* Fill in the tenant base URL (e.g., https://oae.oae-qa0.oaeproject.org)
* Save the changes

Fill in the Facebook App ID and Secret on the tenant administration panel under authentication.