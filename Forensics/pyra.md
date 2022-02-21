## Pyra Writeup

This is our writeup for https://ctf.mcpt.ca/problem/pyra.

### 0. Tool Bashing
Before we do anything with our `fishy.png`, let's first bash our forensics tools on it to see if we find anything useful.

Tools like `steghide`, `stegsolve`, `zsteg`, and `pngcheck` result in no new information, but running `exiftool` on the file does...

![Output from Exiftool](https://cdn.discordapp.com/attachments/937537794870485082/945323818547810304/Untitled_drawing_2.png)

`the ^ password is fishy`. What could this possibly mean?

We also notice that the problem statement is encrypted with what looks to be a substitution cipher. Maybe this will come in handy later...

### 1. Noticing the Colours
When we first open `fishy.png`, we notice that there is a strip of pixels on the bottom of the image.

![](https://cdn.discordapp.com/attachments/937537794870485082/945321234202247168/Untitled_drawing.png)

Upon further inspection (i.e. zooming in), we notice that this strip of pixels is exactly `8` high and `321` across.


### 2. What do we do?
What do we do with the colours we just found? The most obvious thing would be to `lsb` or `msb` the RGBA values of the pixels. If we try both of these options, they result in no new information.

At this point, we should remember our `exiftool` output, and realize that it might be a clue to `XOR` to RGBA values with the password `fishy`. Let's try that.

### 3. Extracting the Pixels
To extract the RGB values, we use `Pillow` module, which allows us to manipulate images, to write a Python script:

```py
from PIL import Image


def extractRGB(src):
    out = []
    with Image.open(src) as img:
        width, height = img.size
        for j in range(height - 8, height):
            for i in range(width):
                pixel = list(img.getpixel((i, j)))
                out.extend(pixel)
    return out


t = extractRGB('fishy.png')
print(t)
```
Notice that we use `j` as the outer for loop and `i` as the inner for loop. This is because coordinates in coding are read as `[y, x]`.

### 4. XOR the Pixels
Taking the pixels, we can through them into `Cyberchef` to XOR them.

Almost immediately, we are greeted with the Cyberchef magic wand, which states that there is a `.ttf` file. Let's extract this, and see what we find.

![](https://cdn.discordapp.com/attachments/937537794870485082/945326770184405022/unknown.png)

### 5. Fonts?
We quickly realize that the `ttf` file is actually a font file. What can we do with this?

![](https://cdn.discordapp.com/attachments/937537794870485082/945327547938398248/unknown.png)

After playing around with the font, you should realize that there is an alphabet at the very top of the file (as seen in the image). What if that's our encoded alphabet? Throwing the problem statement and substitution cipher into Cyberchef gives us this:

![](https://cdn.discordapp.com/attachments/937537794870485082/945328821308096522/unknown.png)

### 6. Guess CTF
It worked! We are now able to read the problem statment.

`RONA IS AT THE GROCERY STORE. AS HE WAS PUSHING HIS CART FILLED WITH FISH AND A PLUSH SPARROW, HE NOTICED SOMETHING SPECIAL.`

This gives us no new information. We still don't know where the flag is. At this point, the smartest thing to do would be to bash forensics tools on the `ttf`, hoping to find something. In particular, try opening the file using a hex editor like `Imhex`, `HxD`, or `Hex Fiend`.

When we open the hex editor and scroll to the very bottom, we realize that the problem statement is embedded into the font. If we look closely, it seems like this is a longer version of the problem statement...

![](https://cdn.discordapp.com/attachments/937537794870485082/945329758982504498/unknown.png)

Taking the extended problem statement, and applying the same substitution cipher to it gives us the contents of the flag.

`RONA IS AT THE GROCERY STORE. AS HE WAS PUSHING HIS CART FILLED WITH FISH AND A PLUSH SPARROW, HE NOTICED SOMETHING SPECIAL. YOUR FLAG IS: DISTRACTED. WRAP IT WITH THE FLAG WRAPPER BEFORE SUBMITTING IT....`

Thus, the flag is `CTF{DISTRACTED}`

### 7. Closing Remarks
We thoroughly enjoyed solving this problem, and it illustrated just how fun CTF can be when you interpret the clues correctly. We hope this writeup is helpful.

\- Aaron, Daniel, James, Shane
