Almost same as before only thing is we need to find the port which is open and available for ssl

for this we use nmap 

nmap -p range target

this will return all open ports but not what service they do , we want one which has ssl and some service which will check and return something.

so we add -sV which will run with the nmap and output with the services in it.

nmap -sV -p 31000-32000 localhost

we need the one with service : SSL/unkown 
not echo cause it will give whatever we send so it is not what we want .

next is we do the same openssl as before.

but entering password will result in a issue called keyupdate ,

this is because the prev lvl(16) pass starts with k which is apparently a command , which is parsed automatically by openssl 

to solve this we add another sub command in openssl called -quiet which will stop this parsing

