This level needs to find human readable content in a text file with binary content and the password is preceded by few = 

The issue is even though we can use grep with binary , it ends up returning almost most of the file 

so we convert it to a string ie human readable using 

string data.txt

we pipe this output to grep with "=" , which will give the answer .

Since they kept ===* sequences for the , password , is , actual passowrd , you could by luck find it in binary grep , but by converting to string we can be sure
