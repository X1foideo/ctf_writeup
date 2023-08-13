# Flip

######  Crypto, Easy - x1foideo


## Challenge Description

> Hey, do a flip!


## Initial Recon 

We are given a python source file showing how the service is working, and how the encryption/decryption works.
Analyzing the source code we get that it's using AES-CBC and that for getting the flag:
- access_username: _admin_
- password: _sUp3rPaSs1_

_**Function 1**_
```python
def start(server):

	...
	
    if "admin&password=sUp3rPaSs1" in message:
		send_message(server, 'Not that easy :)\nGoodbye!\n')
    else:
		setup(server,username,password,key,iv)

```

_**Function 2**_
``` python
def setup(server,username,password,key,iv):

	...
	
	if check:
		send_message(server, 'No way! You got it!\nA nice flag for you: '+ flag)
		server.close()
    else:
        send_message(server, 'Flip off!')
        server.close()
```

_**Function 3**_
```python
def decrypt_data(encryptedParams,key,iv):

	...
	
    if b'admin&password=sUp3rPaSs1' in unpad(paddedParams,16,style='pkcs7'):
        return 1
    else:
        return 0
```

However it's not possible to generate a token with the given credentials as shown in _Function 1_, but for getting the flag we need to send it in some way.

#### AES-CBC

In the Cipher Block Chaining (CBC) mode of operation, each plaintext block is XORed with the previous ciphertext block before being encrypted.

_CT = E(PT^IV)_

<img src="/img/flip/encrypt.png" width="600" display="block">

While considering decryption:

_PT = D(CT) ^ IV_

<img src="/img/flip/decrypt.png" width="600">
So the plaintext of each block depends not only on the corresponding block of ciphertext, but also on the previous block of ciphertext.


## Solution

Since we need to generate a Token having:
- access_username: _admin_
- password: _sUp3rPaSs1_

and since we can't pass these data when asked for them, we need to generate it in a certain way.

Since we are lazy, it's possible to send as data:
- access_username: _bdmin_
- password: _sUp3rPaSs1_

so that the whole token would be "correct" except for the first encrypted byte, then we will need to change just that one.

#### Bit-Flipping Attack

<img src="/img/flip/bitflipping.png" width="600">

Suppose an attacker wishes to manipulate block _i_ of the plaintext.
He then XORs the previous ciphertext block with _x_:
<img src="/img/flip/math_bitflipping.png" width="600">

#### Flag

After sending
- username: bdmin
- bdmin's password: sUp3rPaSs1

We get a leaked 96 hex chars ciphertext, meaning that it's 48 chars (3x16).
Keeping in mind that _access_username=admin&password=sUp3rPaSs1_ is 41 chars and that
- [0:16] are '_access_username=_' 
- [16:32] are '_bdmin&password=s_'
it's possible to change 'bdmin' with 'admin' assuming 'a' in access xors with 'b' in bdmin.

In general terms as seen before:
- _d_(CT<sub>i</sub>) = hex(_PT_) ^ CT<sub>-1, i</sub>
- CT<sub>(-1, i)'</sub> = hex(_PT'_) ^ _d_(CT<sub>i</sub>)

We know that:
- hex(a) = 61
- hex(b) = 62

Scripting:
```python
d = hex(0x62 ^ int(CT[:2], base=16))
r = hex(0x61 ^ int(d, base=16))
CT = r + CT[2:]
```

Send and get the Flag


#crypto #aes #cbc #bitflippingattack
