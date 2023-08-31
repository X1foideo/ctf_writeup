# I love this world

######  Misc, Easy - x1foideo


## Challenge Description

> Vocaloid is a great software to get your computer sing the flag out to you, but what if you can’t afford it? No worries, there are plenty of other free tools you can use. How about — let’s say — this one?


## Initial Recon 

As exposed in the description the aim is to "sing the flag out" in someway.
Checking the extension of the file we fin out that is an _*.svp_ file, which relates to Synthesizer V, a singing synthesizer.
However, trying to listen to the file, thing get worst as it's completely incomprehensible.
Doing some researches, it's possible to notice that this _*.svp_ is nothing if not a simple _*.json_ file.
Reading the file, at the end of it, there are some interesting voices:

```json
"mainRef"{
  
  ...
  
  "database": {
    "name":"Eleanor Forte (Lite)",
    "language": "english",
    "phoneset": "arpabet"
  }

  ...
  
}
```

We get two important info:
- The file uses a _Eleanor Forte (Lite)_ voice
- The file is using _Arpabet_ as the phoneme standard

Downloading the _Eleanor Forte (Lite)_ voice from the voice Database of Dreamtomics and then importing it and listening to the flag helps us understand more about this one.

#### Listening to the Flag

Listening carefully, the very beginning seems to say **FLAG \[STH_NOT_REALLY_UNDERSTANDABLE]  SEKAI ...**
Not so useful at all but paying more attention we notice that the voice refers not to the japanese words, but to the phonemes defined for each note.


## Solution

First of all we need to extract all the phonemes for each note, simple python script 

```python
import json


with open("ilovethisworld.svp", "r") as f:
    f = json.load(f)


notes = f['library'][0]['notes']
for note in notes:
    phoneme = note['phonemes']
    
	print(phoneme)
```

Getting this list:

```plaintext
eh f
eh l
ey
jh iy
k ow l ax n
eh s
iy
k ey
ey
ay
ow p ax n k er l iy b r ae k ih t
eh s
ow
eh m
iy
w ah n
z iy
eh f
ey
aa r
ey
d ah b ax l y uw
ey
w ay
t iy
eh m
aa r
w ah n
f ay v
eh s
iy
k y uw
y uw
iy
eh l
t iy
ow
ow
y uw
aa r
d iy
aa r
iy
ey
eh m
t iy
d iy
w ay
k l ow s k er l iy b r ae k ih t
```

As exposed before, we are using [ARPABET](https://en.wikipedia.org/wiki/ARPABET) as Phoneme Standard, so referring to the link and getting helped with the "Singed Flag", we can retrieve it.

```plaintext
F = eh f
L = eh l
A = ey
G = jh iy
COLON = k ow l ax n
S = eh s
E = iy
K = k ey
A = ey
I = ay
{ = open curly bracket
S = eh s
O = ow
M = eh m
E = iy
1 = w ah n
Z = z iy
F = eh f
A = ey
R = aa r
A = ey
W = d ah b ax l y uw
A = ey
Y = w ay
T = t iy
M = eh m
R = aa r
1 = w ah n
5 = f ay v
S = eh s
E = iy
Q = k y uw
U = y uw
E = iy
L = eh l
T = t iy
O = ow
O = ow
U = y uw
R = aa r
D = d iy
R = aa r
E = iy
A = ey
M = eh m
T = t iy
D = d iy
} = close curly braket
```

Get the flag

#arpabet #svp #phonemes #json #synthesizerV