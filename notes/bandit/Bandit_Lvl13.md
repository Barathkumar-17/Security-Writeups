We are given ssh passkey using which we can login to next level , the password for level 14 is stored in a dir which can be access from level14 only , so we login using passkey then move on with next level ./

Issues:
cannot ssh directly from lvl13 to lvl14 so we need to copy paste the content from lvl13 to linux txt and then pass it and login 

for loggin using passkey:

ssh -i passkey.private(file) username@host -p ...

For transfering files from one user to another in linux server using command is scp 

The passkey must not be open ie only RW for owner no one else so it must be changed using chmod

SCP:

scp -p port username@server:file-path where to save

doing so will ask the password for that level (13) which we already have and now by this we can securely transfer the file .

Initially i created a temp dir in /tmp called band1928
in which i copied the passkey then transfered from there to VM 

then did the ssh using passkey 
