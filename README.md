# OAuth2-Security-Cheatsheet
Security Cheatsheet for OAuth2

# Overview
OAuth2 is not a program, service, or coding library.  OAuth2 is simply a framework/standard that was created by the Internet Engineering Task Force to give websites *limited* access to their data/services to other third-party websites using a decenteralized Authorization Server.  

## Key Frameworks
| Frameworks | Description |
| --- | --- |
| OAuth2 | Authorization framework that enables a third-party application to obtain limited, short-term access to an HTTP service. |
| OpenID Connect | Works ontop of OAuth2, this adds an Identity Layer to the framework.  "Sign In with Google" |

## Main Components
| Components | Description |
| --- | --- |
| Resource Owner (User) | The subject that is attempting to access a protected resource.  You, me, or a API. |
| User Agent (Browser) | The local software that the User is interfacing with in order to communicate with the client. This can be a phone app or your web browser. |
| Client (Web App) | This is the server application that runs the logic.  No data or protected resources are located here. |
| Auth Server (IdP) | The server that is used to authenticate the User.  This is where  |
| Resource Server | Server hosting the protected resources. This is the API you want to access. |

## Auth Flow Types
| Flow Types | Description |
| --- | --- |
| Implicit Grant | *Less Secure.*  No Auth Code, instead the Client obtains the Access Token directly and no Auth Code is created or exchanged. |
| Authorization Code Grant | **Most Common.** The Client obtains an Authorization Code that the Client can exchange for an Access Token after the User logs in at the Authorization Server. |
| Resource Owner Password Credentials Grant | *Not Secure* Basically the Client enters in your Username and Password and signs into the server on your behalf. |
| Client Credentials Grant | *Not Secure* The Client is given master credentials that it can use to obtain Access Tokens. |

## Token Types
| Token Types | Description |
| --- | --- |
| Bearer Tokens | An unsigned token that is used by OAuth2.  It's located in the Authorization Header of HTTP Requests and is also considered a predominate access token.  |
| Access Tokens | A short-lived token that grants access to a protected resource.  Normally exchanged with an Authorization Code.  Access Tokens are also encrypted strings that contain user information. |
| Refresh Tokens | A seperate token that is used to renew an expired Access Token. |
| Identity Tokens | A Self-Contained, JSON Web Token used by OpenID Connect for account login purposes. |

**Tip:** Think of Access Tokens like a session that is created once you authenticate to a website. As long as that session is valid, we can interact with that website without needing to login again. Once the session times out, we would need to login again with our username and password. Refresh tokens are like that password, as they allow a Client to create a new session.

## Misc. Token Types
| Token Types | Description |
| --- | --- |
| Handle-Based Tokens | *Reference tokens* that are typically random strings used to retrieve the data associated with that token. Similar to passing a reference to a variable in a programming language. |
| Self-Contained Tokens | *Contain all the data* associated with that token. This is similar to passing a variable by value in a programming language. This is typically expressed as a JWT. |

### Note: 
  * Refresh tokens are handle-based.
  * Access tokens can be either, handle-based or self-contained.
  * Identity tokens (OpenID/Connect) are self-contained. *(JWTs)*
  
  

