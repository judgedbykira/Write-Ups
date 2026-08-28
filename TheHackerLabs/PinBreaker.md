# Objective

>Your objective is simple: unlock this app.
- Examine the APK, look for clues within the code and find the correct PIN.
- Once you have the PIN, calculate its SHA256 hash – that will be the value of the flag.

>Good luck!

# Reverse Engineering the APK

>First, we will extract the APK file and insert the .dex files on Ghidra to start reversing them:

<p align="center">
  <img width="705" height="533" alt="image" src="https://github.com/user-attachments/assets/281b6fa6-eb69-4405-9fc5-6322db19cdd8"/>
</p>

>After some analysis, we found in `classes3.dex` a function called **checkPin** where we can see a hardcoded PIN:

<p align="center">
  <img width="337" height="150" alt="image" src="https://github.com/user-attachments/assets/7df6e27a-95aa-4e2e-9ca2-9f4e2d451713" />
</p>

>The CTF says to convert the PIN to SHA256 to get the flag, we will do it with the following command, this is used as the user and root flags, meaning we completed the CTF:

```bash
┌──(kali㉿jbkira)-[~]
└─$ echo "8524947156" | sha256sum 
2a4be6606b9490b9955c7aac8e856c8e3098f9b15e98a8985ce5c192049c96ef
```

<p align="center">
  <img width="818" height="586" alt="image" src="https://github.com/user-attachments/assets/198532d1-bf54-41bb-9e49-d48087bcf7e6" />
</p>
