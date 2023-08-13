# BabyEncryption

###### Crypto, Very Easy - X1foideo

## Challenge Description

> You are after an organised crime group which is responsible for the illegal weapon market in your country. As a secret agent, you have infiltrated the group enough to be included in meetings with clients. During the last negotiation, you found one of the confidential messages for the customer. It contains crucial information about the delivery. Do you think you can decrypt it?


## Understanding Code

```python
import string
from secret import MSG

def encryption(msg):
    ct = []
    for char in msg:
        ct.append((123 * char + 18) % 256)
    return bytes(ct)

ct = encryption(MSG)
f = open('./msg.enc','w')
f.write(ct.hex())
f.close()
```

Following the code Bottom-Top, the TO-DOs are:
- Decode frome Hex
- Overcome Modulo
- Reverse to original Char

First and third step aren't a problem, while the second one could be a little bit tedious.
What we have here is the result of *Modulo Operator*, so our original value can be considered as:

> ov = q * d + m
> 
> *q = quotient (unknown)
> d = divisor (256)
> m = modulo (the value we have)*

Since we are dealing with ASCII, the maximum value "char" can have is 127 and so "q" must be lesser than:

> (12 • 123 + 18)/256 = ~62

Now we know that:
> 0 < q < 62

And last but not least we need an integer, not a float, since we are dealing with int values for ASCII


## Solving

``` python
# Get Hexxed Flag
enc_flag = bytes.fromhex(HEXXED_FLAG)

d = 256
flag = []
# Iterate through every modulo
for m in enc_flag:
    # Iterate through all the possible values of q
    for q in range(65):
	# Reverse the Encryption Algorithm
        ov = (q*d+m-18)/123
	
	# Check if value is INT for ASCII
        if f.is_integer():
            f = chr(int(f))
            flag.append(f)

            break
        else:
            pass

print("".join(flag))


```

Get the flag