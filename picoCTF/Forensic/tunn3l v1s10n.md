# tunn3l v1s10n

###### Forensic, 40 pts - x1foideo

## Challenge Description

>We found this [file](https://mercury.picoctf.net/static/09a86202e72dbdb5bf4d1b5d2c6a5b86/tunn3l_v1s10n). Recover the flag.


## Recognition

Check the file type:
```bash
file tunn3l_vision

tunn3l_v1s10n: data
```

Not really useful at all...

Try to grab as much Info as possible:

```bash
exiftool tunn3l_vision

ExifTool Version Number         : 12.16
File Name                       : tunn3l_v1s10n (1)
Directory                       : /media/psf/CTF
File Size                       : 2.8 MiB
File Modification Date/Time     : 2023:05:19 11:08:25+02:00
File Access Date/Time           : 2023:05:19 11:11:59+02:00
File Inode Change Date/Time     : 2023:05:19 11:11:10+02:00
File Permissions                : rw-r--r--
File Type                       : BMP
File Type Extension             : bmp
MIME Type                       : image/bmp
BMP Version                     : Unknown (53434)
Image Width                     : 1134
Image Height                    : 306
Planes                          : 1
Bit Depth                       : 24
Compression                     : None
Image Length                    : 2893400
Pixels Per Meter X              : 5669
Pixels Per Meter Y              : 5669
Num Colors                      : Use BitDepth
Num Important Colors            : All
Red Mask                        : 0x27171a23
Green Mask                      : 0x20291b1e
Blue Mask                       : 0x1e212a1d
Alpha Mask                      : 0x311a1d26
Color Space                     : Unknown (,5%()
Rendering Intent                : Unknown (826103054)
Image Size                      : 1134x306
Megapixels                      : 0.347
```

As it is possible to see it's an image/BMP, renaming the file as *.bmp* is not enough, so probably we need to do some additional work.

As usual Google is friend (sometimes) and it's possible to find how a BMP header should be structured:

> https://en.wikipedia.org/wiki/BMP_file_format

And then compare it with the hex of the file:

```bash
hexyl -v -n 100 tunn3l_vision | head

┌────────┬─────────────────────────┬─────────────────────────┬────────┬────────┐
│00000000│ 42 4d 8e 26 2c 00 00 00 ┊ 00 00 ba d0 00 00 28 00 │BM×&,000┊00××00(0│
│00000010│ 00 00 6e 04 00 00 6e 04 ┊ 00 00 01 00 18 00 00 00 │00n•00n•┊00•0•000│
│00000020│ 00 00 58 26 2c 00 25 16 ┊ 00 00 25 16 00 00 00 00 │00X&,0%•┊00%•0000│
│00000030│ 00 00 00 00 00 00 23 1a ┊ 17 27 1e 1b 29 20 1d 2a │000000#•┊•'••) •*│
│00000040│ 21 1e 26 1d 1a 31 28 25 ┊ 35 2c 29 33 2a 27 38 2f │!•&••1(%┊5,)3*'8/│
│00000050│ 2c 2f 26 23 33 2a 26 2d ┊ 24 20 3b 32 2e 32 29 25 │,/&#3*&-┊$ ;2.2)%│
│00000060│ 30 27 23 33             ┊                         │0'#3    ┊        │
└────────┴─────────────────────────┴─────────────────────────┴────────┴────────┘
```


## Solution

It's possible to notice that there are some errors and it's mandatory to correct them.
Let's edit the file with a hex_editor:

```shell
ghex tunn3l_vision
```

- 0E: ba d0 --> 28 00 (in bytes 40)

However, when opening the image: 

![[secretrezipe.png]]

Quite obvious that's not the flag.

Noticing the exif_data, it's possible to find that it's a 3MiB file for a 1134x306 size, "quite enormous" for such dimensions.

Changing the height at offset 16 with a 1134 value (little_endian HEX: 6E 04) and...

Get the flag

#bmp #image #forensic #hex #height #width