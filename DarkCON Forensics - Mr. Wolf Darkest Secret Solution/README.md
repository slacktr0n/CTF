The challenge:

Mr.Wolf is hiding something from us .Can you find Mr.Wolf's Deepest Secret and he was trying to access some other system and we need to gather more information about Mr.Wolf activities

The solution:

The first step is to mount the E01 file in a way that we can read the contents.  An E01 file is a general container used for forensic images, and in this case it contains a somewhat limited logical image of our target filesystem.  There are many ways to do this, however I chose FTK Imager as that’s what I’m the most familiar with.  This can be done under File > Image Mounting in the GUI.  Once you have the image mounted and a drive letter assigned, it’s time to take a look at our file system.

![dirlisting](https://raw.githubusercontent.com/slacktr0n/CTF/main/screenshots/dir1.png)

As you can see there aren’t many typical artifacts present, which helps to narrow our focus to the available data.  The challenge text mentions that Mr. Wolf was accessing other systems so the Teamviewer directory seems like an obvious starting point, however there isn’t much useful data present aside from some brief connection logs.  Digging deeper into the file structure reveals several RDP Bitmap cache files under \Users\Mr.Wolf\local\Microsoft\Microsoft\Terminal Server Client\Cache, indicating that the Microsoft Remote Desktop Protocol was also used on this system.

In this case the binary files (Cache0000.bin, Cache0001.bin, etc) seem to contain the most usable data.  During the course of an RDP session, Microsoft caches partial sections of the remote desktop interface as they’re updated and the resulting images are stored in these files.  To parse them we’ll need a tool, for which I chose bmc-tools (https://github.com/ANSSI-FR/bmc-tools/).

![collage](https://raw.githubusercontent.com/slacktr0n/CTF/main/screenshots/collage.png)

Once installed, you can use bmc-tools to parse the files with the following command: ./bmc-tools.py -s Cache0000.bin -d Cache0001/ -b (The -b flag provides a collage which can be easier to sort through, as shown above).  Repeat this for each .bin file and directory to dump all of the images into a viewable format, although the first (0000) cache will have the most useful data.  You can look at the individual pictures or the collage, but either way this ends up being the digital equivalent of putting a shredded document back together.  Despite all of this we’re able to recover some interesting URLs:

https://defuse.ca/b/OphiXIBtXMpo5j1l9DfXQn

https://defuse.ca/b/4wpwCy52raDjsqLWvj5f5s

https://pastebin.com/pDQGmA6R

The first defuse.ca link contains encrypted text, which won’t appear without the right decryption key.  To find this we’ll need to turn back to the source file system.  As you may have noticed from the first screenshot, there’s a hidden $RECYCLE.BIN folder that contains two deleted text files, of which the $RIYT1YS.txt file contains our password (youwillknowwhenyouhavetouseit).  To be able to read the file you'll need to change permissions on the directory, for which either the image will need to be opened with read/write permissions or the folder will need to be otherwise copied off.  When the correct password is entered in the defuse.ca page we get a large block of base64 encoded text, when dumped to a binary file we can see this is a zip archive by using the file command on Linux.

This is where things start to get tricky, as we rename the file and attempt to extract it only to find that the archive is password protected.  It appears to contain our flag, in addition to another file named "Hackers, Heroes of the Computer Revolution. Chapters 1 and 2 by Steven Levy.txt".  My initial thought was that the password had to be somewhere in the forensic image, however my attempts to find it turned up empty.  I also considered that it might be an easier brute-force challenge, and even went as far as to converting the text of the referenced book into a wordlist to use with John the Ripper, but had no luck there either.
The only remaining option at this point is to exploit the zip file another way, through a known plaintext attack.  Zip files encrypted with legacy encryption are vulnerable to this type of attack if we know some of the contents in plain text, which thankfully we have a pretty good idea of from the file name.  Finding the correct source file takes a bit of guesswork as we need to make sure that the beginning of the text files are exactly the same, however based on the title used in the file name it appears that this is the free text version hosted on Gutenberg:

https://www.gutenberg.org/ebooks/729

I chose bkcrack (https://github.com/kimci86/bkcrack) to run this attack, however there are several other options out there as well.  For this attack to work, we have to make sure that we put the known text file into an archive in the same format as the target.  Once that’s done, we can attack the file:

![bkcrack1](https://raw.githubusercontent.com/slacktr0n/CTF/main/screenshots/bkcrack1.png)

Once the process completes successfully we are given the decryption key for the archive, which we can apply to extract the flag.txt file from the original archive:

![bkcrack2](https://raw.githubusercontent.com/slacktr0n/CTF/main/screenshots/bkcrack2.png)

With that, we now know Mr. Wolf's darkest secret, and the challenge is complete.
