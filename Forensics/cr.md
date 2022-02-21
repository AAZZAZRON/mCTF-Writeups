## Cr Writeup

This is our writeup for [cr](https://ctf.mcpt.ca/problem/cr), a problem found on LyonCTF 2022.

### 0. Password Protected ZIP
Upon downloading `cr.zip`, we immediately realize what we need to do. The ZIP file is protected by a password, so the goal would be to crack the password to access the contents of the file.

![](https://cdn.discordapp.com/attachments/937537794870485082/945357818683351071/unknown.png)

### 1. Googling
How do we accomplish this task? A quick google search on how ZIP files work brings us to [this wikipedia page](https://en.wikipedia.org/wiki/ZIP_(file_format)). If we read the page, specifically the section on encryption, we realize that the encryption system on ZIP files known as `ZipCrypto` (now legacy), and that it is "seriously flawed".

![https://en.wikipedia.org/wiki/ZIP_(file_format)#Encryption](https://cdn.discordapp.com/attachments/937537794870485082/945358472394989638/unknown.png)

After realizing the encryption method is flawed, we realize that the goal of this problem must be to crack ZipCrypto. Doing another google search on how to crack ZipCrypto brings us to blog posts like [this](https://anter.dev/posts/plaintext-attack-zipcrypto/) and [this](https://blog.devolutions.net/2020/08/why-you-should-never-use-zipcrypto/), which both redirect us to a ZipCrypto cracking script called [`bkcrack`](https://github.com/kimci86/bkcrack).

### 2. Understanding bkcrack
Reading the README on the bkcrack git repo, we realize that we have reduced the problem to just a few commands:
- recover the internal ZIP keys
- use the keys to change the ZIP password to a password of our choice
- use that password to access the ZIP files

We can also use the [tutorial](https://github.com/kimci86/bkcrack/blob/master/example/tutorial.md) provided by bkcrack to further understand each command.

### 3. Recover the Internal ZIP Keys
To recover the internal keys, we must provide bkcrack with at least 12 bytes of known `.webp` plaintext. We realize that we can provide 12 bytes of known plaintext by simply providing the `webp header`.

![](https://cdn.discordapp.com/attachments/937537794870485082/945361188739514378/unknown.png)

There are many ways to do this, but the most efficient way would be to use:
```
$ bkcrack -C "cr.zip" -c 1.webp -x 0 52494646 -x 6 000057454250565038
```
To understand how this command works, we strongly recommend you read the bkcrack README on your own time.

![](https://cdn.discordapp.com/attachments/937537794870485082/945364579393892422/unknown.png)

Note that we are very lucky that one of our zipped files (`1.webp`) is stored uncompressed in the zip file. If it were to be compressed, we would first have to figure out how the file was compressed, before we are able to provide the correct plaintext.

### 4. Change ZIP Password
Executing the previous command (which would have taken a while) gives us 3 keys:

```
4d849e19 9e6953b9 ccfdc777
```

We can use these keys to change the ZIP password to something we choose, say `LyonCTF`

```
bkcrack -C cr.zip -k 4d849e19 9e6953b9 ccfdc777 -U cr_unlocked.zip LyonCTF
```

### 5. Flag
From there, we can simply use our password to access the flag.

Flag: `CTF{cryptography_is_my_passion}`
