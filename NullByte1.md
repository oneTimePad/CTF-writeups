### NullByte 1 boot2root writeup:
    (sorry for how this looks)
    After doing a couple of the boot2roots on vulnhub.com, I decided to start writing up some of my work.
    
    So to start off with the usual:
        - nmap -sT -v -A -p 1-65535 [vm ip]
    
    The only interesting port opened seemed to port 80 which contained the webserver.
    
    To see if there was anything else interesting on the server, I did some directory busting:
    
        - dirb http://[vm ip]
    
    nothing seemed to stick out, except the /uploads folder which just contained a message... so i moved on
    
    when visiting the index, I was confronted with a page just containg a gif. It seemed quite obvious what to do.
    So I downloaded the image, and used exiftool to inspect the metadata, and there it was under the comment:
        - Comment P-): kzMb5nVYJw
    Then using an educated guess I visited: http://[vm ip]/kzMb5nVYJw, upon which I was confronted with a login that just
    asked for a key.
    
    I then busted out burpsuite to inspect the server response. There were no tricks in there, so I decided to let hydra
    do some work.
    
        - hydra [vm ip] http-form-post "/kzMb5nVYJw/index.php:key=^USER^:invalid key" -L /usr/share/wordlists/rockyou.txt
    
    And after a few minutes, I found the correct key: elite.
    
    Then I was presented with a strange page that just asked for a username. Confused, I fuzzed around a little bit, 
    and I was excited to see that " resulted in a MySQL error. So there was obviosuly an Error-based SQLi vuln.
    
    I first tried to see if I could construct a valid query that the db saw as the same as a normal query for a usename
       
        - user"--+
        
    Since that worked, I now know what the correct format is to break the query and create a new one, and the follow query,
    gave me an interesting result:
    
        -- user" union select user,pass,2 from users--+
        
    I saw a user named `ramses` with a base64 encoded password. After decoding the password, I saw it was a password hash.
    After cracking the hash with online tools, I found the password was `omega`.
    
    Performing another educated guess and doing:
        ssh ramse@[vm ip]
    with password omega, I was able to log in.
    
    I then started doing the usual of looking for local privilage escalation exploits:
          cat /proc/version
          sudo -l
          
    However, I wasn't able to find anything interesting. After a while, I came across an executable called `procwatch`.
    I ran it, and saw that it was spawning a root shell to run the `ps` command. I knew this was the answer but couldn't
    figure out what to do, so I moved on.
    
    After running my favorite linuxprivchecker.py, I saw that the machine was vulnerable to the OpenSSL/SSH predictable 
    PRNG exploit. So I found some common keys https://github.com/g0tmi1k/debian-ssh . However, this all led me down 
    a hopeless road. I finally looked at the walkthrough, and was disappointed that the answer was quite obvious.
    
    The answer was to creat a symlink named `ps` that would be called by procwatch, and append the location of that symlink to the 
    front of the PATH. Thus procwatch would run the chosen command as root, revealing the flag.
    
    Overall, I enjoyed this boot2root, but was dissapointed that couldn't figure out the last step. Thanks!
