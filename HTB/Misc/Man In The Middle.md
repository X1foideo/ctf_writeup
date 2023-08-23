# Man In The Middle

######  Misc, Easy - x1foideo


## Challenge Description

> Help! One of our red teamers has captured something between a user and their computer, but we've got no idea what we're looking at! Can you take a look?


## Initial Recon 

Analyzing the _log_ file we got, it results in being a _BTSnoop File_
```bash
file mitm.log 
mitm.log: BTSnoop version 1
```
Loking for this type of file on the internet, it is possible to find out that is a Bluetooth Capture file, which can be opened with Wireshark.

Open the file with _Wireshark_ and **Decode As...** "_BT HID_".
It is possible to see that most of the packets are 15 in Length, while a minority is 17.

<img src="https://raw.githubusercontent.com/x1foideo/CTFs-Writeups/cc5b5ab41fbd31138c25f28177735a7284b0e7fe/img/maninthemiddle/wiresharkcapture.png" width="600">

Analyzing the _Bluetooth HID Profile_ it's possible to notice that:
- 15 Len Packets are Mouse Captures
- 17 Len Packets are Keyboard Captures

Paying more attention to the 17 Len Packets, we find out that Packet No.
- 548 is a capture for letter 'h'
- 750 is a capture for letter 't'
- 982 is a capture for letter 'b'
- 1285 is a capture for '\['

This last one is quite strange since usually htb-flags begin with '_htb{_...}'.
Keeping in mind this information we notice that 
- _Modifier: LEFT SHIFT: True_

<img src="https://github.com/x1foideo/CTFs-Writeups/blob/main/img/maninthemiddle/wiresharkshifttrue.png?raw=true" width="600">

Implying that it's a '{' and not a '\['

## Solution

Extract all the Keyboards chars, and if they are pressed simultaneously with 'Shift'.
```bash
tshark -r "mitm.log" -d 'btl2cap.cid==65,bthid' -Y 'bthid.data.protocol_code == 0x01' -Y 'usbhid.boot_report.keyboard.keycode_1 != 0x00' -T fields -e 'usbhid.boot_report.keyboard.modifier.left_shift' -e 'usbhid.boot_report.keyboard.keycode_1' | tr '\t' ',' > payload
```

Obtaining
```plaintext
0,0x28
0,0x28
1,0x0b
1,0x17
1,0x05
1,0x2f
1,0x0e
0,0x20
0,0x1c
1,0x16
0,0x17
1,0x15
0,0x27
0,0x0e
0,0x08
1,0x16
1,0x2d
1,0x06
0,0x27
0,0x10
1,0x13
0,0x15
0,0x27
0,0x10
0,0x1e
0,0x16
0,0x20
0,0x07
1,0x30
```

We need to convert from HEX to ASCII char, a simple python script is needed 

```python
# HEX to char
KEY_CODES = {
    '0x04':['a', 'A'],
    '0x05':['b', 'B'],
    '0x06':['c', 'C'],
    '0x07':['d', 'D'],
    '0x08':['e', 'E'],
    '0x09':['f', 'F'],
    '0x0a':['g', 'G'],
    '0x0b':['h', 'H'],
    '0x0c':['i', 'I'],
    '0x0d':['j', 'J'],
    '0x0e':['k', 'K'],
    '0x0f':['l', 'L'],
    '0x10':['m', 'M'],
    '0x11':['n', 'N'],
    '0x12':['o', 'O'],
    '0x13':['p', 'P'],
    '0x14':['q', 'Q'],
    '0x15':['r', 'R'],
    '0x16':['s', 'S'],
    '0x17':['t', 'T'],
    '0x18':['u', 'U'],
    '0x19':['v', 'V'],
    '0x1a':['w', 'W'],
    '0x1b':['x', 'X'],
    '0x1c':['y', 'Y'],
    '0x1d':['z', 'Z'],
    '0x1e':['1', '!'],
    '0x1f':['2', '@'],
    '0x20':['3', '#'],
    '0x21':['4', '$'],
    '0x22':['5', '%'],
    '0x23':['6', '^'],
    '0x24':['7', '&'],
    '0x25':['8', '*'],
    '0x26':['9', '('],
    '0x27':['0', ')'],
    '0x28':['\n','\n'],
    '0x29':['␛','␛'],
    '0x2a':['⌫', '⌫'],
    '0x2b':['\t','\t'],
    '0x2c':[' ', ' '],
    '0x2d':['-', '_'],
    '0x2e':['=', '+'],
    '0x2f':['[', '{'],
    '0x30':[']', '}'],
    '0x32':['#','~'],
    '0x33':[';', ':'],
    '0x34':['\'', '"'],
    '0x36':[',', '<'],
    '0x37':['.', '>'],
    '0x38':['/', '?'],
    '0x39':['⇪','⇪'],
    '0x4f':[u'→',u'→'],
    '0x50':[u'←',u'←'],
    '0x51':[u'↓',u'↓'],
    '0x52':[u'↑',u'↑']
}

# Open payload file
with open('./payload', 'r') as f:
    payload = f.readlines()

# Iterate through every line 
flag = []
for l in payload:
    l = l.strip()
    sh, ch = l.split(',')
    
    #  Check if Shift is pressed or not, and choose the right char
    if sh == "1":
        ch = KEY_CODES[ch][1]
    else:
        ch = KEY_CODES[ch][0]

    flag.append(ch)

flag = "".join(flag)
print(flag)
```

Get The Flag

#keyboard #wireshark #btsnoop #bt #bluetooth #tshark #HID
