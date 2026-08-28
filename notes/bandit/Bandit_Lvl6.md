Bandit Level 6 again requires find command but this time we need to search in home or root dir of the current one , so basically it is content of full server 

We need to find a file with group bandit6 and user bandit7 and size 33 bytes in / dir 

running this command 
"find / -user bandit7 -group bandit6 -size 33c"

will output all the permission denied with the one where it is found , but will be buried in it , for this we route all the permission denied ( error 2) to a null dir inbuilt in linux server (/dev/null)

so we add 2>/dev/null at the end 

so command :

ind / -type f -user bandit7 -group bandit6 -size 33c 2>/dev/null


