
It was a great honor to participate in **Arab Cyber Wargames championship 2019 in Egypt**.

Honestly, my team did a great work, especially in the last hour.

The style of challenges, the first 4 hours was **Jeopardy** challenges, then they proceeded to lanuch **attack-defence** machines.

This's a write-up of **IOT** challenge called "*Flash*".

After downloading the "tar.gz" file , which it's a ***firmware*** for a Netgear Router, I proceeded to explore the content of this firmware. An interesting file called "***rootfs.squashfs***" was found. "Squashfs" is a some sort of linux file system used in embeded linux systems.

![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-07-45.png)

I extracted this file in regard to find the whole system files, and of course, Gnu/linux OS directories were found.

![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-14-08.png)


![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-15-12.png)


The first place came to mind, was to explore the "/home" directory, and search for interesting files in each user. A user name called "Allen" existed, which was suspicious somewhat.

![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-15-26.png)

And, oh these files appeared, so I had to swap to my little cryptanalysis background, let's look on "encryption.py".

Here's a simple AES encryption procedures, let's dive into these lines of codes.

![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-17-18.png)


The "***encrypt***" function takes the plaintext_from the use, which is definitely is the flag, then getting the MD5 hash of the key. After that, initialization vector is obtained randomly as long as the length of AES standard block length.
Depending on AES CBC mode, and after encrypting and doing proper padding, it proceeds to print the ciphertext as a ***base64*** encoding.

Let's try to recover the plaintext, but it seems impossible without having the encryption secret key !
I had no choice, but to explore the other files, hence the ciphertext was found in "***mysecret.txt***" file. And a phrase in the "***23f88ac14feead92f4fdf8e640507d3c.txt***", which it seemed to have no interesting hints.


![](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-15-39.png)


![mysecret.txt](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-15-51.png)


![23f88ac14feead92f4fdf8e640507d3c.txt](/Arab_Security_Wargames_2019/flash/Screenshot_from_2019-09-24_14-16-07.png)


But wait, the name of this file was suspicious to me; "23f88ac14feead92f4fdf8e640507d3c.txt", I tried to identify it, whether it would be one of the known hashes. Guess what, it's ***md5*** hash. And after looking up on an online database, I recover the hash, producing the "nora" text.

Surely, this's a hint_from the challenge author, to narrow players' searching area.
The most interested thing that, "encrypt" function get the md5 of the key before submitting it on AES encryption mechanism.

So, using the recover hash as key , would be a nice try, but after encoding it as "utf-8" and getting it's "md5" hash.

Here's the decryption function, now we have all what we need to recover the secret message.

    from hashlib import md5
    from base64  import b64decode
    from Crypto.Cipher import AES
    from Crypto.Random import get_random_bytes
    
    def decrypt():
	    # ciphertext
	    c = 'tDSZhqegpwWb3ZLOvsewA+KpMPez6ATY2+mYuMy4+2xi4LILHkZj1qG2rVJCyIu5'
	    key = 'nora'
	    key = md5(key.encode('utf-8')).digest()
	    iv = get_random_bytes(AES.block_size)
	    cipher = AES.new(key, AES.MODE_CBC, iv)
	    print cipher.decrypt( b64decode(c))
	    
    decrypt()

In "encrypt" function, you can find the the expected ciphertext consists of iv - as a first portion ,and the plainttext.

Let's have a try! Wow, it works, the flag is
**ASCWG{H0W_I_Miss_Y0U}**

