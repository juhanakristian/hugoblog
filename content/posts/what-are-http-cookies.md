---
title: "What are HTTP cookies"
draft: false
date: "2021-10-28T00:00:00+00:00"
author: juhana.jauhiainen
tags: ["webdev", "cookies", "nodejs", "expressjs"]
description: "A HTTP cookie is a small piece of data that a server sends to a user's web browser. The browser may then store the cookie and send it back to the same server with later requests."
---

A HTTP cookie is a small piece of data that a server sends to a user's web browser. The browser may then store the cookie and send it back to the same server with later requests.

Cookies are used typically to tell if two HTTP requests come from the same browser/user. So for **session management**, **tracking** and to a lesser extent **personalization**

For general data storage on the client, there are more modern APIs.

### How cookies are sent between server and client

When client(browser) and the server communicate with HTTP calls (GET, POST etc) the calls include content, or a [body](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages#body) and [headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages#headers)

The headers contain a lot of information on the request or response currently being transported. Things like [content type](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type), [cache control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) and [user agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent). Headers are present both when client mades a request to the server, and when the server responds.

**Cookies are also set and transported using HTTP headers.** Setting cookies on the server is done by adding a [Set-Cookie header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) to a response. The browser will then attach a [Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie) header, with the value sent by the server in the `Set-Cookie` header, to every request it will make to a url with the same domain.

`Set-Cookie` has a number of optional attributes which can be used to configure how the the browser treats the cookie. These include for example `Max-Age`, which can be used to set the lifetime of the cookie. 

### Using cookies in Node.js and Express

Setting cookies in Express.js is done by calling the [cookie](https://expressjs.com/en/5x/api.html#res.cookie) method on a [Response object](https://expressjs.com/en/5x/api.html#res). 

```javascript
app.get('/', (req, res) => {
	const value = "something";
	res.cookie("somecookie", value, { maxAge: 1000 * 60 * 15 })
	res.send('Hello World!')
});
```

Reading cookies can be done with the [cookies](https://expressjs.com/en/5x/api.html#req.cookies) property of the [Request object](https://expressjs.com/en/5x/api.html#req) When using the [cookie-parser](https://www.npmjs.com/package/cookie-parser) middleware, `cookies` becomes a object that contains the cookies senty by the browser.

```javascript
var app = express()

app.use(cookieParser())
app.get('/', (req, res) => {
	const value = req.cookies.somecookie;
	res.send('Hello World!');
});
```


### Cookie security

Cookies are automatically sent by the browser when a request is made to the domain which set the cookie originally. This is really nice, but can result in security issues. You don't want the browser to allow accessing the cookie from JavaScript or sending the cookies over an insecure connection.

To make your cookies secure, you should use the optional attributes passed to `Set-Cookie`.

**HttpOnly**  
Forbids JavaScript access to cookie using `Document.cookie`

**Secure**  
Sends the cookie only over **HTTPS**

**SameSite=Strict | Lax | None**  
Strict: Only send the cookie for same-site requests  
Lax: Don't send the cookie on cross-site requests but send it when navigating to our server using a link  
None: Always sends the cookie  

**Expires=date**  
Sets the maximum lifetime of the cookie as a HTTP-date timestamp

It's important to set **HttpOnly** for any cookies containing authentication tokens or sensitive information to prevent JavaScript access to them in the client. 

**SameSite** can be used to prevent some [Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf) attacks. Setting it to **Strict** might not be ideal, because then a user navigating from a link to your site, will have to login even if they have a valid session. Setting it to **Lax** will only send the cookies [when a user is navigating to your site](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite#lax) SameSite defaults to **Lax** if not set.

If **Expires** is not set, the cookie will be deleted when the browser session ends.

With Express.js you can set these using the options object passed to `cookie` method of the Response object.

```javascript
app.get('/', (req, res) => {
	const value = "something";
	res.cookie("somecookie", value, { 
		httpOnly: true,
		secure: true,
		sameSite: "strict",
		expires: date
	})
	res.send('Hello World!')
});
```

If you're using cookies to create a user session, consider using a middleware like [express-session](https://www.npmjs.com/package/express-session)


## References
[MDN on Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
