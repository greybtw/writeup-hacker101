# Flag0 
Totally, we have 2 functions are `add cart` and `check out` 
In `check out`, inspect element in nearly button. I got a hidden input hold a json object which defines price of a pet
```html
<input type="hidden" name="cart" value="[[1, {"logo": "puppy.jpg", "price": 7.95, "name": "Puppy", "desc": "8\"x10\" color glossy photograph of a puppy."}], [0, {"logo": "kitten.jpg", "price": 8.95, "name": "Kitten", "desc": "8\"x10\" color glossy photograph of a kitten."}], [1, {"logo": "puppy.jpg", "price": 7.95, "name": "Puppy", "desc": "8\"x10\" color glossy photograph of a puppy."}]]">
```
Adjust the price to lower price (`0`) and `check out`
```python
Flag is : ^FLAG^03c74c0654e76c9eacb2c8e0133aa8401e1b8b58990c0f2a78cbb312a483e9e3$FLAG$
```

# Flag1
I got some hints
```
There must be a way to administer the app
Tools may help you find the entrypoint
```
Therefore, tool `dirb` is a great idea to find endpoint
```http
+ http://35.227.24.107/40e4f9edbc/edit (CODE:400|SIZE:192)                          
+ http://35.227.24.107/40e4f9edbc/login (CODE:200|SIZE:329) 
```
`Tools are also great for finding credentials` is another hint. I should use Burp Suite to brute force `username` and `password`
--> username=`cayla`
Guessing password -> password=`felicia`
```python
Flag is : ^FLAG^fcac5ff6b846ddc65d3bce1a27e368a4025b30acb06a9fdde18d00e48005a8bd$FLAG$
```

# Flag2
After log in, we can edit the price and detail information of a pet. 
Try to simple XSS and get the flag (click in shopping cart)
```python
Flag is : ^FLAG^55e4ae35db4076dd69e9400c1d5c406bba1cb81bde9ac54a11ccc9de4ffe72f4$FLAG$
```

