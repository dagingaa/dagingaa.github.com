---
layout: post
authors: [dagingaa, havardge]
title: Introduction to OAuth 2.0
category: OAuth
---
_This post was originally posted on [Comoyo Engineering](http://comoyo.github.com/blog/2012/07/11/introduction_to_oauth/)._

For our summer project we were tasked with building a hyper efficient, minimal implementation of an OAuth 2.0 Authorization server. This would be of great use internally because we previously had no method of ensuring that a user’s protected resources were protected by anything but a user's credentials, which poses some limitations on access to for example external developers or other Telenor affiliates that wanted to use our APIs and data to create their own services. 

In this post, we wish to explain a little bit about what OAuth 2.0 is, what it isn’t, and the problem it solves compared to the traditional way of doing protected resource authorization.

#The traditional model#
![The traditional authorization method](/assets/img/posts/oauth/traditional_model.png)

In the traditional model an end-user, let’s call him Bob, would give a third-party application access to his protected resources by letting the application have access to and use his private credentials to do actions on his behalf. If say, an application wanted access to Bobs previously watched movies on [Comoyo Film](https://www.comoyo.com/no/film), that application had to obtain the end-user’s credentials and store them. However, this poses a problem for Bob. How does Bob know that the application stores his credentials in a safe manner? [Many](http://www.wired.com/threatlevel/2010/12/gawker-hacked/) big [corporations](http://it.slashdot.org/story/12/06/06/1335228/linkedin-password-hashes-leaked-online) [fail](http://www.bgr.com/2012/06/07/last-fm-passwords-hacked-leaked/) at this [time](http://www.scmagazine.com.au/News/304694,10000-twitter-passwords-leaked.aspx) and time again. How can Bob trust this single developer to have the best intentions about using his account, and keep others with malicious intent from accessing these credentials?

What if Bob had signed up for a lot of these services, and one of them had malicious intent for Bob’s list of watched movies? Bob would then have to change his credentials with Comoyo, effectively deauthorizing all his other services. In addition, Bob has no specific control over who has access to what. Bob doesn’t want a movie recommender to have access to his payment details, or be able to make purchases on his behalf. Bob finds these disadvantages unacceptable, but what are his options?

#What makes OAuth 2 better?#
<img src="/assets/img/posts/oauth/maskot.png" class="pull-right"/>

With OAuth 2.0 you can forget about giving third-party applications your private credentials. You can revoke their access at any time, or adjust their access to your data. OAuth 2.0 works by inserting an authorization server between you, the application and the resource server, which handles authorization requests, user authentication and user authorization. 

The authorization server will, after authenticating both the user and the application (hereby client), issue an access token to the client which grants specific access to a user’s resources over a given time. After this, the client would have to ask permission again, or use a refresh token to refresh its access token. This gives Bob the ability to revoke the access of the misbehaving client, without deauthorizing all his other services. 

The access token works in many ways as a valet key for your expensive luxury car. The valet key only allows the driver to drive a certain distance while blocking access to the trunk, glove box  and on-board cellphone. This way the car owner (resource owner) can grant a parking attendant (client) limited access to his car by issuing the valet key (access token).

It is also possible for an application to achieve [pseudo-authentication](http://en.wikipedia.org/wiki/OAuth#OpenID_vs._pseudo-authentication_using_OAuth) using OAuth 2.0. Because only the true resource owner (in this case, the end user) can authorize the issuing of a access token to an application, the application can be certain that if the access token obtained is valid, then the user must also be valid. This is based on trust. The application trusts the API provider to handle user authentication, and it trusts the issuing of access tokens. However, for this scheme to work, the application needs to verify the authenticity of the access token by making a request to the API providers resource server and obtaining some personally identifiable information about the user. This is the mechanism behind services such as Facebook Connect. 

However, OAuth is not a complete authentication solution, and does not state how user authentication should happen on the authorization server, only that it must happen. OAuth is a service that is complementary to, but distinct from, [OpenID](http://openid.net/).

#Some terminology#

While the original OAuth technology is referred to as a protocol, the OAuth 2.0 is more of an authorization framework that generalizes OAuth. An important aspect of this framework is the notion of roles and how they interact.
To make things a bit easier to explain, we will go through some terminology related to OAuth 2.0. OAuth 2.0 defines four roles: Resource Owner, Client, Authorization Server and Resource Server. 
<dl>
<dt>Resource owner</dt>
 <dd>an entity capable of granting access to a protected resource. When the resource owner is a person, it is referred to as an end-user.</dd>
<dt>Resource server</dt>
<dd>the server hosting the protected resources. It is capable of responding to protected resource requests using access tokens. In other words, when a client requests a protected resource, the resource server validates the access token, and if valid, serves the request.</dd>
<dt>Client</dt>
<dd>an application making protected resource requests on behalf of the resource owner and with its authorization.</dd>
<dt>Authorization server</dt>
<dd>the server issuing access tokens to the client. Before issuing tokens, it must authenticate the resource owner and obtain authorization to do so.</dd>
</dl>

OAuth 2.0 also defines several authorization flows, which we will explain a bit more in detail below.

#How can you authorize a client#
The abstract flow illustrated below describes how these four roles interact.

![Interaction model](/assets/img/posts/oauth/general_flow.png)

1. The client requests authorization from the resource owner.
2. The client receives an authorization grant which is a credential representing the resource owner’s authorization.
3. The client requests an access token by authenticating with the authorization server and presenting the authorization grant.
4. The authorization server authenticates the client and validates the authorization grant, and if valid issues an access token.
5. The client requests the protected resource from the resource server and authenticates by presenting the access token.
6. The resource server validates the token, and if valid, serves the request.

There are also many other ways of authorizing a client. The specification defines four grant types: authorization code, implicit, resource owner resource owner password credentials, and client credentials, as well as an extensibility mechanism for defining additional types. The preferred method is to use the authorization code grant, where the authorization server is used as an intermediary between the client and the resource owner.

Next time, we will look into the different authorization flows in detail.

#Resources#
1. [The IETF OAuth 2.0 draft](http://tools.ietf.org/html/draft-ietf-oauth-v2-28)
2. [Introduction to OAuth 2.0 by hueniverse](http://hueniverse.com/2010/05/introducing-oauth-2-0/)
3. [OAuth Wikipedia entry](http://en.wikipedia.org/wiki/OAuth)
