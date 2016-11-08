---
layout: post
title: "backdoorctf16 : Lossless"
author: aaron
date: 2016-06-06 16:50:00 -0700
category:
- 2016
- backdoorctf
slug: lossless
---
**Category:** Stego
**Points:** 100
**Description:**

>d4rth used his dirty methods to hide a secret in a png file. He is cleverly trying to divert your focus from challenge, but the force is strong with you. Now extract the flag from these images, my young padawan.
http://hack.bckdr.in/LOSSLESS/original.png
http://hack.bckdr.in/LOSSLESS/encrypted.png
<br />Created by: Arpit Singla

# Investigation
We are provided with 2 image files, original.png and encrypted.png.  When looking at the two images, they appear to be the same.

#### original.png
<img src="{{ site.static }}/2016/backdoorctf/lossless/original.png" class="img-responsive"/>

#### encrypted.png
<img src="{{ site.static }}/2016/backdoorctf/lossless/encrypted.png" class="img-responsive"/>

My first thought was LSB Steganography.  For those unfamiliar, LSB stands for Least Significant Bit.  When applied to images, LSB works by encoding your hidden message amongst all low-order bytes in some innocent cover image.  The reason this works without a user noticing a difference in the image is because the least significant bits are insignificant by definition.  Meaning, if a pixel color component is set to 254, the human eye will be unable to notice the difference from 254 to 255.

<img src="{{ site.static }}/2016/backdoorctf/lossless/lsb.png" class="img-responsive"/>

If the recipient of your message does not have your cover image already, then when you want to encode a 1, you must set the LSB to 1.  However, we call it informed LSB stego when the recipient does have a copy of the cover image.  When that is the case, the encoding works slightly differently.  Instead, to embed a 1, we simply flip the LSB, regardless of it's original value.  If you would like to embed a 0 bit, then you leave the LSB value unchanged, as-is.

To confirm my suspicions about this being an LSB stego challenge, I decided to write some simple Python code to print any changes in pixel values.  This will also serve to reveal implmenentation specific details about the stego techniques used.

```python
from PIL import Image

def img_diff_printer(img_orig, img_enc):
    X,Y = img_orig.size
    for x in range(X):
        for y in range(Y):
            pxl_orig = img_orig.getpixel((x,y))
            pxl_enc = img_enc.getpixel((x,y))
            if pxl_orig != pxl_enc:
                print('[%d, %d] %s != %s' % (x, y, str(pxl_orig), str(pxl_enc)))

img_orig = Image.open('original.png')
img_enc = Image.open('encrypted.png')

img_diff_printer(img_orig, img_enc)
```

And running the script produces the following output

```
[0, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[0, 2] (2, 0, 1, 255) != (2, 0, 0, 255)
[0, 4] (1, 1, 1, 255) != (1, 1, 0, 255)
[1, 0] (2, 1, 1, 255) != (2, 1, 0, 255)
[1, 1] (2, 1, 1, 255) != (2, 1, 0, 255)
[1, 3] (2, 1, 1, 255) != (2, 1, 0, 255)
[2, 0] (1, 1, 1, 255) != (1, 1, 0, 255)
[2, 1] (1, 1, 1, 255) != (1, 1, 0, 255)
[2, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[2, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[3, 1] (2, 1, 2, 255) != (2, 1, 3, 255)
[4, 0] (4, 2, 3, 255) != (4, 2, 2, 255)
[4, 1] (3, 2, 3, 255) != (3, 2, 2, 255)
[4, 4] (2, 1, 1, 255) != (2, 1, 0, 255)
[4, 5] (2, 1, 1, 255) != (2, 1, 0, 255)
[5, 0] (4, 2, 3, 255) != (4, 2, 2, 255)
[5, 1] (3, 2, 2, 255) != (3, 2, 3, 255)
[5, 3] (4, 2, 3, 255) != (4, 2, 2, 255)
[5, 4] (4, 2, 3, 255) != (4, 2, 2, 255)
[6, 0] (3, 1, 2, 255) != (3, 1, 3, 255)
[6, 1] (4, 2, 3, 255) != (4, 2, 2, 255)
[6, 6] (4, 2, 3, 255) != (4, 2, 2, 255)
[7, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[7, 1] (2, 1, 1, 255) != (2, 1, 0, 255)
[7, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[7, 5] (3, 1, 2, 255) != (3, 1, 3, 255)
[7, 6] (3, 2, 2, 255) != (3, 2, 3, 255)
[8, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[9, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[9, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[9, 3] (2, 0, 1, 255) != (2, 0, 0, 255)
[9, 6] (2, 0, 1, 255) != (2, 0, 0, 255)
[10, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[10, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[10, 2] (2, 0, 1, 255) != (2, 0, 0, 255)
[10, 5] (2, 0, 1, 255) != (2, 0, 0, 255)
[10, 6] (2, 0, 1, 255) != (2, 0, 0, 255)
[11, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[12, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[12, 2] (2, 0, 1, 255) != (2, 0, 0, 255)
[12, 5] (2, 0, 1, 255) != (2, 0, 0, 255)
[12, 6] (2, 0, 1, 255) != (2, 0, 0, 255)
[13, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[13, 3] (2, 0, 1, 255) != (2, 0, 0, 255)
[14, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[14, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[15, 1] (2, 1, 1, 255) != (2, 1, 0, 255)
[15, 2] (2, 0, 1, 255) != (2, 0, 0, 255)
[15, 5] (2, 0, 1, 255) != (2, 0, 0, 255)
[16, 1] (3, 1, 2, 255) != (3, 1, 3, 255)
[16, 2] (2, 0, 1, 255) != (2, 0, 0, 255)
[16, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[16, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[17, 1] (3, 1, 2, 255) != (3, 1, 3, 255)
[17, 2] (3, 1, 2, 255) != (3, 1, 3, 255)
[17, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[17, 5] (3, 1, 2, 255) != (3, 1, 3, 255)
[18, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[18, 1] (3, 1, 2, 255) != (3, 1, 3, 255)
[18, 2] (3, 1, 2, 255) != (3, 1, 3, 255)
[18, 3] (3, 1, 2, 255) != (3, 1, 3, 255)
[18, 5] (3, 1, 2, 255) != (3, 1, 3, 255)
[18, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[19, 0] (2, 0, 1, 255) != (2, 0, 0, 255)
[19, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[19, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[20, 1] (2, 0, 1, 255) != (2, 0, 0, 255)
[20, 2] (3, 1, 2, 255) != (3, 1, 3, 255)
[20, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[21, 0] (3, 1, 2, 255) != (3, 1, 3, 255)
[21, 1] (3, 1, 2, 255) != (3, 1, 3, 255)
[21, 4] (2, 0, 1, 255) != (2, 0, 0, 255)
[21, 5] (2, 0, 1, 255) != (2, 0, 0, 255)
[22, 0] (3, 1, 2, 255) != (3, 1, 3, 255)
[22, 1] (3, 1, 2, 255) != (3, 1, 3, 255)
[22, 4] (3, 1, 2, 255) != (3, 1, 3, 255)
[22, 5] (3, 1, 2, 255) != (3, 1, 3, 255)
[23, 1] (3, 2, 2, 255) != (3, 2, 3, 255)
[23, 2] (2, 1, 2, 255) != (2, 1, 3, 255)
[23, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[24, 0] (31, 22, 16, 255) != (31, 22, 17, 255)
[24, 1] (25, 17, 13, 255) != (25, 17, 12, 255)
[24, 5] (3, 1, 2, 255) != (3, 1, 3, 255)
[24, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[25, 0] (13, 9, 6, 255) != (13, 9, 7, 255)
[25, 1] (20, 13, 9, 255) != (20, 13, 8, 255)
[25, 2] (31, 22, 14, 255) != (31, 22, 15, 255)
[25, 4] (17, 12, 8, 255) != (17, 12, 9, 255)
[25, 6] (4, 2, 2, 255) != (4, 2, 3, 255)
[26, 0] (30, 20, 13, 255) != (30, 20, 12, 255)
[26, 1] (23, 15, 10, 255) != (23, 15, 11, 255)
[26, 3] (18, 12, 8, 255) != (18, 12, 9, 255)
[26, 4] (25, 17, 11, 255) != (25, 17, 10, 255)
[27, 1] (21, 13, 9, 255) != (21, 13, 8, 255)
[27, 2] (26, 18, 12, 255) != (26, 18, 13, 255)
[27, 4] (15, 10, 6, 255) != (15, 10, 7, 255)
[27, 5] (14, 10, 7, 255) != (14, 10, 6, 255)
[27, 6] (24, 16, 12, 255) != (24, 16, 13, 255)
[28, 0] (6, 3, 2, 255) != (6, 3, 3, 255)
[28, 2] (7, 5, 3, 255) != (7, 5, 2, 255)
[28, 3] (13, 9, 6, 255) != (13, 9, 7, 255)
[28, 4] (21, 14, 9, 255) != (21, 14, 8, 255)
[28, 5] (23, 15, 9, 255) != (23, 15, 8, 255)
[28, 6] (17, 11, 8, 255) != (17, 11, 9, 255)
[29, 0] (7, 5, 3, 255) != (7, 5, 2, 255)
[29, 1] (5, 3, 2, 255) != (5, 3, 3, 255)
[29, 2] (3, 2, 1, 255) != (3, 2, 0, 255)
[29, 4] (11, 6, 4, 255) != (11, 6, 5, 255)
[30, 1] (12, 8, 5, 255) != (12, 8, 4, 255)
[30, 2] (7, 4, 2, 255) != (7, 4, 3, 255)
[31, 0] (6, 4, 2, 255) != (6, 4, 3, 255)
[31, 2] (11, 7, 5, 255) != (11, 7, 4, 255)
[31, 3] (11, 7, 5, 255) != (11, 7, 4, 255)
[31, 4] (7, 4, 2, 255) != (7, 4, 3, 255)
[31, 5] (4, 3, 1, 255) != (4, 3, 0, 255)
[31, 6] (5, 2, 1, 255) != (5, 2, 0, 255)
[32, 0] (6, 4, 2, 255) != (6, 4, 3, 255)
[32, 1] (5, 4, 2, 255) != (5, 4, 3, 255)
[32, 4] (6, 5, 3, 255) != (6, 5, 2, 255)
[32, 5] (7, 3, 2, 255) != (7, 3, 3, 255)
[33, 1] (6, 4, 3, 255) != (6, 4, 2, 255)
[33, 2] (5, 3, 2, 255) != (5, 3, 3, 255)
[34, 0] (7, 4, 3, 255) != (7, 4, 2, 255)
[34, 1] (8, 4, 3, 255) != (8, 4, 2, 255)
[34, 5] (7, 4, 4, 255) != (7, 4, 5, 255)
[34, 6] (5, 2, 2, 255) != (5, 2, 3, 255)
[35, 0] (9, 5, 3, 255) != (9, 5, 2, 255)
[35, 1] (9, 5, 3, 255) != (9, 5, 2, 255)
[35, 2] (10, 5, 3, 255) != (10, 5, 2, 255)
[35, 4] (18, 11, 6, 255) != (18, 11, 7, 255)
[35, 6] (5, 2, 1, 255) != (5, 2, 0, 255)
[36, 1] (6, 4, 3, 255) != (6, 4, 2, 255)
[36, 2] (7, 4, 3, 255) != (7, 4, 2, 255)
[36, 4] (23, 14, 7, 255) != (23, 14, 6, 255)
[36, 6] (7, 3, 2, 255) != (7, 3, 3, 255)
[37, 1] (8, 4, 2, 255) != (8, 4, 3, 255)
[37, 3] (15, 9, 5, 255) != (15, 9, 4, 255)
[37, 4] (21, 13, 7, 255) != (21, 13, 6, 255)
[38, 0] (4, 3, 1, 255) != (4, 3, 0, 255)
[38, 1] (4, 2, 1, 255) != (4, 2, 0, 255)
[38, 2] (4, 2, 1, 255) != (4, 2, 0, 255)
[38, 4] (6, 4, 3, 255) != (6, 4, 2, 255)
[38, 5] (5, 3, 3, 255) != (5, 3, 2, 255)
[38, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[39, 0] (4, 3, 1, 255) != (4, 3, 0, 255)
[39, 1] (4, 3, 1, 255) != (4, 3, 0, 255)
[39, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[40, 1] (4, 3, 1, 255) != (4, 3, 0, 255)
[40, 2] (9, 5, 3, 255) != (9, 5, 2, 255)
[40, 4] (5, 3, 2, 255) != (5, 3, 3, 255)
[40, 6] (3, 1, 2, 255) != (3, 1, 3, 255)
[41, 0] (3, 2, 1, 255) != (3, 2, 0, 255)
[41, 1] (4, 3, 1, 255) != (4, 3, 0, 255)
[41, 3] (12, 7, 4, 255) != (12, 7, 5, 255)
[41, 4] (5, 3, 1, 255) != (5, 3, 0, 255)
[41, 5] (4, 2, 2, 255) != (4, 2, 3, 255)
[42, 1] (6, 4, 2, 255) != (6, 4, 3, 255)
[42, 4] (3, 2, 0, 255) != (3, 2, 1, 255)
[42, 5] (4, 3, 1, 255) != (4, 3, 0, 255)
[42, 6] (6, 4, 2, 255) != (6, 4, 3, 255)
[43, 0] (11, 6, 4, 255) != (11, 6, 5, 255)
[43, 1] (10, 6, 4, 255) != (10, 6, 5, 255)
[43, 2] (4, 2, 1, 255) != (4, 2, 0, 255)
[43, 4] (3, 2, 0, 255) != (3, 2, 1, 255)
[44, 0] (12, 7, 4, 255) != (12, 7, 5, 255)
[44, 2] (4, 3, 1, 255) != (4, 3, 0, 255)
[44, 3] (4, 3, 1, 255) != (4, 3, 0, 255)
[44, 4] (4, 3, 1, 255) != (4, 3, 0, 255)
[44, 5] (4, 3, 1, 255) != (4, 3, 0, 255)
[44, 6] (6, 3, 2, 255) != (6, 3, 3, 255)
[45, 0] (7, 4, 3, 255) != (7, 4, 2, 255)
[45, 1] (5, 3, 1, 255) != (5, 3, 0, 255)
[45, 3] (3, 2, 1, 255) != (3, 2, 0, 255)
[45, 6] (4, 3, 1, 255) != (4, 3, 0, 255)
[46, 1] (5, 2, 1, 255) != (5, 2, 0, 255)
[46, 2] (5, 2, 1, 255) != (5, 2, 0, 255)
[46, 4] (5, 2, 1, 255) != (5, 2, 0, 255)
[46, 5] (5, 3, 1, 255) != (5, 3, 0, 255)
[46, 6] (5, 2, 1, 255) != (5, 2, 0, 255)
[47, 1] (6, 2, 1, 255) != (6, 2, 0, 255)
[47, 2] (6, 2, 1, 255) != (6, 2, 0, 255)
[47, 3] (6, 2, 1, 255) != (6, 2, 0, 255)
[47, 4] (6, 2, 1, 255) != (6, 2, 0, 255)
[47, 5] (6, 2, 1, 255) != (6, 2, 0, 255)
[47, 6] (6, 2, 1, 255) != (6, 2, 0, 255)
[48, 0] (3, 2, 0, 255) != (3, 2, 1, 255)
[48, 1] (3, 2, 0, 255) != (3, 2, 1, 255)
[48, 2] (4, 3, 1, 255) != (4, 3, 0, 255)
[48, 3] (4, 3, 1, 255) != (4, 3, 0, 255)
[48, 4] (4, 3, 1, 255) != (4, 3, 0, 255)
[48, 6] (5, 2, 1, 255) != (5, 2, 0, 255)
```

Yup, looks like LSB stego to me!

Furthermore, I gleaned some details about the embedding techniques.  I notice that only the 3rd color component changes.  The other 3 color components remain unchanged.  Next I noticed that only the first 7 pixels of every row of pixels ever changes.  So at this point, I assumed that each byte is represented by the first 8 pixels in every row and that the 8th bit (MSB, most significant) is always 0.  This makes sense if we consider that the embedded data is likely ASCII-printable characters.

Oh, and it looks like only the first 49 rows of pixels contain embedded data.  So I will ignore all other rows.

# Solution
Finally, I wrote some code to extract the hidden message bytes.

```python
#!/usr/bin/env python
from PIL import Image

def lsb_destego(img_orig, img_enc):
    decode = ''
    for x in range(50):
        byte = ''
        for y in range(7):
            pxl_orig = img_orig.getpixel((x,y))
            pxl_enc = img_enc.getpixel((x,y))
            if pxl_orig != pxl_enc:
                byte += '1'
            else:
                byte += '0'
        decode += chr(int(byte, 2))
    return decode

img_orig = Image.open('original.png')
img_enc = Image.open('encrypted.png')

decode = lsb_destego(img_orig, img_enc)
print(decode)
```

# Flag
This challenge is hosted permanently at [Backdoor](https://backdoor.sdslabs.co/challenges/LOSSLESS), so go find the flag yourself!
