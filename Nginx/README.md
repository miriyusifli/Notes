# Nginx

### rewrite directive
Enclose the rewrite directive in a server or location context that defines the URLs to be rewritten. 
```
rewrite regex URL [flag];
```
But the first argument, regex, means that NGINX Plus and NGINX rewrite the URL only if it matches the specified regular expression (in addition to matching the server or location directive). 
rewrite directive can return only code 301 or 302. To return other codes, you need to include a return directive after the rewrite directive (see the example below). 
And finally the rewrite directive does not necessarily halt NGINX’s processing of the request as return does, and it doesn’t necessarily send a redirect to the client. 
Example:
```
location /test {
   rewrite /test/(.*) /$1 last;
   return 301;
}

location /hello {
   return 501;
}
```
In the example above, `www.domain.com/test` returns `301` as HTTP response status code while `www.domain.com/test/hello` returns `501`. Firstly, nginx finds the corresonding
location for the `/test` URL, then it checks regex `/test/(.*)`  of the rewrite rule. If it matches, rule is applied. 

##### What does this rule mean?
As mentioned above, it regex is matched, rule is applied. It means, that, URL is rewritten. `$1` indicates path element (first paranthesis) `(.*)` that is not changing.
Let's look at some examples how this rule will be applied to the following URLs.
```
/test/hello -> /hello
/test/bye -> /bye
/test/login -> /login
```

##### flag
3 flags can be used with rewrite directive.
1. last -  means if the rewrite condition equals true then rewrite the requested uri and jump to the location block that fits the newly rewritten uri.
2. break -  means if the rewrite condition equals true then rewrite the requested uri **BUT DO NOT JUMP**, instead stay in the same location block and continue to the next line inside the block
3. permanent - It is similar to last, but changes URL in the browser permanently and returns a permanent redirect with the 301 code.



