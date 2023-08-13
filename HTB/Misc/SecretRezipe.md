# SecretRezipe

###### Misc, Easy - X1foideo


## Challenge Description

> We have launched a startup that produces soft drinks. We use special ingredients to make them very tasty, so we have a lot of protections on our files to prevent our competitors from copying our ideas.


## Initial Recon

Analyzing the SecretRezipe web-page we notice that the only thing we are able to do is "Suggesting Some Ingredients" for the Refreshing Drinks they offer.

 <img src="https://user-images.githubusercontent.com/12828790/181280980-c429da09-244b-4c05-8b2e-1c6c4e298d00.png" width="600">

After downloading the *ingredients.zip* we notice that it is protected by a password.


## More than an Initial Recon

There are some files for understanding the whole Challenge.

After downloading them and analysing the files, we find something interesting in *routes.js*

- When *Writing and Zipping* the ingredients on the website, what we are doing is simply posting a request to */ingredients* 

- It creates a tmp folder and a random-16-Bytes subfolder where is stored a *ingredients.txt* where it contains:
  
  - First Line: "Secret: Flag"
  
  - Second Line: "ingredients we inserted in the box"

- The ingredients.txt file is zipped:
  
  ```bash
  zip -P ${PASSWORD} ${tempPath}/ingredients.zip ${tempPath}/ingredients.txt
  ```


## Solution

We look for the password through the files.

Nope.

The password is randomically generated and is a 36-character v4 UUID and... We cannot bruteforce it.


<img src="https://user-images.githubusercontent.com/12828790/181281550-0688d970-771c-4b34-871d-8494150983df.jpg" width="600">


## The Real (Slim Shady) Solution

First of all we try to get some information from the zip.

Leaving the box empty and downloading the file we notice that

```bash
zipdetails 'ingredients.zip'
```

<img width="600" alt="Screenshot 2022-07-27 alle 16 27 32" src="https://user-images.githubusercontent.com/12828790/181281726-172eb698-b6ec-47d2-b1c5-04d8585dfc1b.png">

The *Compression Method* is "Stored".

Looking through the internet we notice that the with this method the file is no compressed and we are able to make a "Known Plain Text Attack" and for this we can use ***bkcrack***.

We need at least 12-bytes of known plaintext and, of course, the encrypted file.


## Ok, Let's Go

- First of all we need to retrieve the ingredients.txt path inside the zip file:
  
  ```bash
  bkcrack -C 'ingredients.zip'
  ```

- Then we can try to begin the attack.
  The assumption is that The first line is (quitely sure):
  
  > Secret: HTB{
  
  More the Flag

- We can create a new file and put this payload

- Let's find the key
  
  ```bash
  bkcrack -C 'ingredients.zip' -c 'tmp/16-RANDOMBYTES/ingredients.txt' -p 'ingredients.txt'
  ```

- TOO SLOW.


## BETTER, FASTER, STRONGER

We NEED a faster way, and more error proof, since we are assuming that *ingredients.txt* begins with that specific payload.

Inserting some "a" (a thousand is enough) and downloading the file we get a really HORRIBLE news.

<img width="600" alt="Screenshot 2022-07-27 alle 16 27 32" src="https://user-images.githubusercontent.com/12828790/181282276-84685f80-5629-4912-850c-f94ad7a9e611.jpg">


DEFLATE METHOD

But what if we bypass this, finding a way for "not compressing" the file?

We need to add something that is not repeated consecutively, Let's say:

> Ingredient: asd
> Ingredient: zxc
> Ingredient: qwe
> Ingredient: rty
> Ingredient: uio
> ...

And... HERE WE ARE.
**We have "Store" compression method back again.**

This time we will put inside the file the exact content put inside the box and then set the offset of where to find it.
We can calculate the offset as follow:

> offset = uncompressed_size - plain_text_len


Let's try again then...

```bash
bkcrack -C 'ingredients.zip' -c 'tmp/16-RANDOMBYTES/ingredients.txt' -p 'ingredients.txt' -o offset
```


Waaaaaaay Faster, and here there's our keys.

Now we need to get the file:

```bash
bkcrack -C 'ingredients.zip' -k 'key' -U unencr_ingredients.zip 'new_password'
```

Unzip and Enjoy


#zip #bkcrack #password
