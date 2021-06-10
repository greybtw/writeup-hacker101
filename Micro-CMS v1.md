## flag2
Okay yah start new challenge, we dont have so much link to discovery. Oh we have a button to create a new page, I created a new page but nothing happens, really?
I got attention to this test `Markdown is supported, but scripts are not`, script? I think we should xss this form. So, I inject into tile like this `</h1><script>alert(1)<script>` and create a new page. But nothing happens againnn, I back to hompage and boomp an alert was appeared with first flag
flag2 is : ^FLAG^02ea874476054e9d854502177096c60b48087d41c823f37565f82f7e2896d5ed$FLAG$

## flag0
Okay next, I got a hint is "How are pages indexed?". Oh I realized when I created a new page, the new index is jumped from 1-2 to 12? So, where is 3-11? I check out every index and got the same response `Not Found` but index=5 is `Forbidden`. We need to access this page, it maybe contains flag.
Other pages also have the button edit and redirect to `page/edit/{index}`, we'll use this point to access page `index=5` by access the link `page/edit/5` and we get the flag
flag0 is : ^FLAG^5b6b3473352fad3136e24955849489cd04938ea0d8f0e498b1663d2ef8a8036c$FLAG

## flag1
I got a hint `Make sure you tamper with every input` but I was XSS into amost every input, what did I miss? Hmmm, look about the link I think it can be a SQL
I add a quote `'` after `http://35.227.24.107/122612c0e9/page/1` but I got the response error 😶 Okay I'm fine.
And I'm really missing the edit page, oops! Next try, I added a quote after link edit page like the following link:
`http://35.227.24.107/122612c0e9/page/edit/1'`
The flag was appeared! It's SQL injection
flag1 is : ^FLAG^a8d92cd5b416d8e2e471e7e836f7e1d150f2817653fecab19b2c98694e8df88e$FLAG$


## flag3
How about XSS into another block? Now we try to XSS into block body with button
```
<button onclick=alert()>1</button>
```
Clicked the button but we didn't get any flag. View-source page with `Ctrl U`, now we got the flag
flag3 is : ^FLAG^670c93d2ffcfbbb30d3beae35ec2b77b4408a5e705a0d2a884b122c928ae6fe8$FLAG$