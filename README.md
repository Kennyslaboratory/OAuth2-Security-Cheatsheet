# Overview
OAuth2 is not a program, service, or coding library.  OAuth2 is simply a framework/standard that was created by the Internet Engineering Task Force to give websites *limited* access to their data/services to other third-party websites using a decenteralized Authorization Server.  

### Key Frameworks
OAuth2 is used for Authorization, not Authentication/Identity.  However, using an extension call OpenID Connect, we can use OAuth2 for Aurhtnetication.
| Frameworks | Description |
| --- | --- |
| [OAuth2](https://tools.ietf.org/html/rfc6749) | Authorization framework that enables a third-party application to obtain limited, short-term access to an HTTP service. |
| [OpenID Connect](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1) | Works ontop of OAuth2, this adds an Identity Layer to the framework.  "Sign In with Google" |

### Main Components
There's typically 5 different actors in the OAuth2 flow:
| Components | Description |
| --- | --- |
| Resource Owner | The subject that is attempting to access a protected resource.  You, me, or some API. |
| User Agent | The local software that the User is interfacing with in order to communicate with the client. This can be a phone app or your web browser. |
| Client | This is the server application _(Web App)_ that runs the OAuth2 logic.  No data or protected resources are located here. |
| Auth Server | The server that is used to authenticate the User.  Tokens are exchanged here for access to protected resources. |
| Resource Server | Server hosting the protected resources. This is the API you want to access. |

### Auth Flow Types
There are *4 types* of ways to use OAuth2, however, most use-cases are "Authorization Code Grant":
| Flow Types | Description |
| --- | --- |
| Implicit Grant | No Auth Code, instead the Client obtains the Access Token directly and no Auth Code is created or exchanged. |
| Authorization Code Grant | **Most Common.** The Client obtains an Authorization Code that the Client can exchange for an Access Token after the User logs in at the Authorization Server. |
| Resource Owner Password Credentials Grant | Basically the Client enters in your Username and Password and signs into the server on your behalf. |
| Client Credentials Grant | The Client is given master credentials that it can use to obtain Access Tokens. |

#### GET Request Parameters
When you're directing a 
| Parameter | Description |
| --- | --- |
| response_type | Specifies what type of Authorization Flow is being used. |
| client_id | This is used by the Auth Server to Identify the client. |
| redirect_uri | Specifies where to redirect the User-Agent. |
| scope _(optional)_ | Declare what resources/permission you want from the Identity Provider. |
| state _(recommended)_ | If not using PKCE to prevent CSRF attack then the state parameter is necessary to protect user accounts fro hijacking via CSRF. |

#### POST Request Parameters
| Parameter | Description |
| --- | --- |
| Host | authorization-endpoint.com |
| grant_type | Specifies the type of Auth Flow you are using.  I.E. `code` |
| code | This is where the Auth Code is after logging into the Auth Server / Identity Provider. |


### Token Types
| Token Types | Description |
| --- | --- |
| Bearer Tokens | An unsigned token that is used by OAuth2.  It's located in the Authorization Header of HTTP Requests and is also considered a predominate access token.  |
| Access Tokens | A short-lived token that grants access to a protected resource.  Normally exchanged with an Authorization Code.  Access Tokens are also encrypted strings that contain user information. |
| Refresh Tokens | A seperate token that is used to renew an expired Access Token. |
| Identity Tokens | A Self-Contained, JSON Web Token used by OpenID Connect for account login purposes. |

**Tip:** Think of Access Tokens like a session that is created once you authenticate to a website. As long as that session is valid, we can interact with that website without needing to login again. Once the session times out, we would need to login again with our username and password. Refresh tokens are like that password, as they allow a Client to create a new session.

### Misc. Token Types
| Token Types | Description |
| --- | --- |
| Handle-Based Tokens | *Reference tokens* that are typically random strings used to retrieve the data associated with that token. Similar to passing a reference to a variable in a programming language. |
| Self-Contained Tokens | *Contain all the data* associated with that token. This is similar to passing a variable by value in a programming language. This is typically expressed as a JWT. |

#### Note: 
  * Refresh tokens are handle-based.
  * Access tokens can be either, handle-based or self-contained.
  * Identity tokens (OpenID/Connect) are self-contained. *(JWTs)*
  
### Client Types
| Clients | Description |
| --- | --- |
| Public Client | A client that is located on the same device as the User-Agent.  This client should not be used to store client secrets because it can't be secured. This category includes mobile applications, single page applications, and desktop clients as an attacker is able to extract the secrets by downloading those applications. |
| Confidential Client | A Web Application that is located on a server seperate to the User-Agent. |

# Common OAuth2 Vulnerabilities
| Attack | Description |
| --- | --- |
| [Pseudo-Auth CSRF Attack](https://security.stackexchange.com/questions/20187/oauth2-cross-site-request-forgery-and-state-parameter) | If OAuth2 is being used as a pseudo-authentication protocol to login, then it is possible to obtain access to a user account by linking your account with their's via CSRF Attack. |
| [Classic CSRF Attack]() | Using a CSRF Attack to forward a User to the Auth Server, having them obtain an Auth Code, then stealing it.  Afterwards, the Hacker exchanges the stolen Auth Code for an Access Token. |
| [Stealing client_secret]() | If the client_secret is embedded in the same local device as the User-Agent then it may be possible to steal the client_secret used to verify the Client with the Auth Server. |
| [Open Redirect]() | Stealing the Authorization Code by hijacking the redirect_url parameter and redirecting the final GET Response to an attacker's server. |
| [Weak state parameter]() | The state parameter is perdictable and not cryptographically secure. |
| [Unchecked state parameter]() | The client does not check or validate the state parameter before submitting the Auth Code for an Access Token. | 
| [state Fixation]() | The state parameter is meant to be calculated from the user session.   |

# Planning your Server Architecture:
## What Authroization Grant Type Should I Use..?
As a general rule of thumb, if you are developing a standard Web Application or Native Mobile Application then you should be using the Authorization Code Grant.  The Implicit Grant was generally how we used to do things before CORS support was widely available but I rarely if ever see a use for the Implicit Grant Type now.
**Example:**
 * If you're building a classic web application: Use the Authorization Code Grant.
 * If you're building a single page application: Use the Authorization Code Grant **without secrets**.
   * Read more about developing OAuth2 Flow for Single Page Apps, [here](https://www.oauth.com/oauth2-servers/single-page-apps/#:~:text=The%20only%20way%20the%20authorization,using%20a%20registered%20redirect%20URL.).
 * If you are building a native mobile application: Use the Authorization Code Grant with [PKCE](https://medium.com/identity-beyond-borders/what-the-heck-is-pkce-40662e801a76).
 * If your client is 100% internal and is authenticating with trusted APIs and not unknown users, then it's probably *absolutely trusted*.  In this can you can use user credentials (i.e. the Facebook app accessing Facebook Auth): In this case, Use the Resource Owner Password Grant.
 * If your client is the sole owner of the data, and there is no user login: Use the Client Credentials Grant.

## Where should I store my Access/Refresh Tokens..?
Design your application so that it can store the Refresh Tokens securely on a server. However, if you are developing on completely RESTful application then this may not be an option for you.  If you do not trust that the client can store tokens securely *(single-page web applications, etc)*, then do **not** issue Refresh Tokens. An attacker with access to long-lived Refresh Tokens will be able to obtain new Access Tokens and use those Access tokens to access the resources of other users. The main downside of not using Refresh Tokens is that users/clients would need to re-authenticate every time the Access Token expires.

**What about Local Storage?**
Access tokens and esspecially refresh tokens should **never** be stored in the local/session storage, because then they can be stolen via Cross-Site Scripting attacks or other attacks that can read local/session storage.  Remember, this area of storage is meant to be public and accessable to the User-Agent. If your client is not being hosted on the same device as the User-Agent then it would be best practice to store the access token in a cookie with [cookie prefixes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) enabled.  At the bare minimun the cookie should use the HttpOnly, Secure, and SameSite cookie prefixes.

