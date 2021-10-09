# Flag0 
The first hint is `The person with username "user" has a very easy password...` so I use the Burp Suite to brute force with payload `Passwords` 
Credentials:
```
username=user&password=password
```
And the flag
```ruby
Flag is : ^FLAG^ff6b9cfbbf363e2ec617b2026cdbc94d491ee93e202711f9ee711c8d1143c2b1$FLAG$
```

# Flag1
Access `My profile` and I attend to URL contain `id` is a predictable character
`URL=http://35.227.24.107/f87e69ba92/index.php?page=profile.php&id=c` 
I change the `id` to `a, b` and access successful the admin's profile with `id=b` in page `SECRET: Dear diary...`
```ruby
Flag is : ^FLAG^e710214c1c678bab0671e077205db5f386b455ad095e81ffa36e5477bba1a3c1$FLAG$
```

# Flag2
Access `Settings`, I have 2 inputs are `Username, Password` and a button `Submit`. 
How about change password of admin?
After submit `username=admin&password=admin` I receive 2 notifications `Username updated!` and `Password updated!`
Log out and log in with admin's submitted credentials. But I didn't get anything :)))
*because I just change the username and password of `user` (not `admin`). Maybe the application use the cookie `id` to identify user

Continue, I try to XSS the function `Create new post` but I cannot escape string to exploit XSS
When I inspect element the form, I saw a hidden input `name=user_id` 😗 Change `value`, submit new post and capture the flag

```ruby
Flag is : ^FLAG^48fe2c5696d5888ba3494ef940debbd132a82f43a95580592de11d790ed60d91$FLAG$
```

# Flag4
Continue with URL contains `id` 
I can edit other's post with different id sush as id of admin'post `http://35.227.24.107/f87e69ba92/index.php?page=edit.php&id=1`

```ruby
Flag is : ^FLAG^48fe2c5696d5888ba3494ef940debbd132a82f43a95580592de11d790ed60d91$FLAG$
```

# Flag5
The cookie `id` lookly easy to crash, I think it is `md5` hash. Use the online tool to crack 
`c81e728d9d4c2f636f067f89cc14862c` -> `2`
Easy to change to `id` of admin :)) `md5` hash value `1`
`1` -> `c4ca4238a0b923820dcc509a6f75849b`

# Flag6
Inspect the post, I detect URL to delete post with given id hashed (`md5`).
`href="index.php?page=delete.php&amp;id=8f14e45fceea167a5a36dedd4bea2543`
Crack `8f14e45fceea167a5a36dedd4bea2543` -> `7`
So I can delete other's post by inspect the URL or hash `id`
```ruby
Flag is : ^FLAG^0a156387ba924706220d8471e14b84be7535cc4220496b1db9f306f16f9437b1$FLAG$
```

# Flag3
Hint is `189 * 5` ( ╯□╰ )  
It seems a large id, access `http://35.227.24.107/f87e69ba92/index.php?page=view.php&id=945` and get the flag
```ruby
Flag is : ^FLAG^e0a91d3489a9d0cd3095ded0ce1cdc425192b50d55acb38bdc1cb513eb700f86$FLAG$
```