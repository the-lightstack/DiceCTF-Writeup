# DiceCTF-Writeup
# Babier CSP Challenge by notdeghost

When you visit the [challenge website](https://babier-csp.dicec.tf/) you are welcomed by a link with the name 'View Fruit'.
When clicked, it adds the get paramater ?name to the url with the value of a random Fruit.

The `name` value is just echoed back into the sites html. The challenge description also tells you, that your objective is to steal the admins
cookies which greatly hints towards a cross site scripting attack (XSS) -- So let's do that!

When you try to access `<site>?name=<script>alert(1)</script>` , nothing happens though. If you have a look at the main sites html using burp to intercept to
the request, you see that, the "View Fruit" link works with javascript and it has a nonce value set!

![Burp request intercept](https://github.com/the-lightstack/DiceCTF-Writeup/blob/main/burp_babier_csp.png)
