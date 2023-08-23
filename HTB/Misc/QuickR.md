# QuickR

######  Misc, Easy - X1foideo


## Challenge Description

> Let's see if you're a QuickR soldier as you pretend to be


## Initial Recon 

Starting the Instance, we get a Host.
First idea is to check it with nc, so let's give it a try:

<img width="600" alt="Screenshot 2022-12-06 alle 15 59 30" src="https://user-images.githubusercontent.com/12828790/205954894-b6fbece3-7b31-4204-a91d-5a948aea9936.png">
Scanning the QR we get some calculations to do, in 3 seconds.
I'm not a supergenius and not so fast (at least not my phone) for scanning, reading and solving the problem.
What a news, we need to automatize it.


##  It's Showtime!

### Terminal Interaction

First of all we need to "interact" with Terminal:

```python
from pwn import *


nc = remote(host, port)
nc.recvuntil(b"seconds!")

# Getting rid of spaces and unwanted extra-data
my_qr = nc.recvuntil(b"[+]")
my_qr = my_qr.strip()

# Splitting Lines
my_qr = my_qr.split(b"\n")
my_qr = my_qr[:-3]

for line in my_qr():
	print(line)
```

We notice the output is made of 2 *ANSI Escape Codes*:

 - **\x1b[0m\x1b[7m**: Reset and Reverse
 -  **\x1b[0m\x1b[41m**: Reset and BGRed

So we can assume this represents the QRCode (White and Red).

### The problem

The real problem here is that as we can see we don't get an image, but ANSI and checking for Decoding QRs, the only way we find is interacting with images.
So what we need, is to find a way for creating a QR-Image from what we get, scan It and then solve the calculation (of course then we need to send It and get the flag).

### Creating QRCanva

Since I don't like colors, let's say that the 2 ANSIs represents Black and White

```python
from PIL import Image, ImageColor


BLACK = b'\x1b[0m\x1b[7m'
WHITE = b'\x1b[0m\x1b[41m'

# Create a Canva
img = Image.new("RGB", (200, 200))

for j, line in enumerate(my_qr):
    # Converting QR to B/W 
    line = line.split()
    line = [b'B' if BLACK in e else b'W' for e in line]
	
    # Print the QR on the Canva
    for k, color in enumerate(line):
        if color == b'B':
            img.putpixel((j, k), (155,155,155))
        else:
            img.putpixel((j, k), (0,0,0))
``` 

### Decoding QR

```python
from pyzbar.pyzbar import decode

# Decode QR and get Data
qr_decoded = decode(img)
qr_decoded = qr_decoded[0].data.decode()

# Some Adjustments
qr_decoded = qr_decoded.replace("=", "").strip()
if "x" in qr_decoded:
    qr_decoded = qr_decoded.replace("x", "*")

# Solving Math
qr_data = eval(qr_decoded)
```


## Solution

```python
from pwn import *
from PIL import Image, ImageColor
from pyzbar.pyzbar import decode


BLACK = b'\x1b[0m\x1b[7m'
WHITE = b'\x1b[0m\x1b[41m'


nc = remote("HOST", port)
nc.recvuntil(b"seconds!")
my_qr = nc.recvuntil(b"[+]")

my_qr = my_qr.strip()
my_qr = my_qr.split(b"\n")
my_qr = my_qr[:-3]

img = Image.new("RGB", (200, 200))
for j, line in enumerate(my_qr):
    line = line.split()
    line = [b'B' if BLACK in e else b'W' for e in line]

    for k, color in enumerate(line):
        if color == b'B':
            img.putpixel((j, k), (155,155,155))
        else:
            img.putpixel((j, k), (0,0,0))

qr_decoded = decode(img)
qr_decoded = qr_decoded[0].data.decode()

qr_decoded = qr_decoded.replace("=", "").strip()
if "x" in qr_decoded:
    qr_decoded = qr_decoded.replace("x", "*")

qr_data = eval(qr_decoded)

nc.recv()

nc.sendline(str(qr_data).encode())

result = nc.recv()
print(result)
```

And we got the Flag.
