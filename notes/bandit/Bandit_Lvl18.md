Logging in to the server using ssh automatically logs you out ,.

So need to get the readme file on home dir of lvl 18 , any connection to lvl 18 using ssh is logged out using .bashrc

there are two ways to solve 
1. run command directly when ssh (only this command will be run during the login)
2. use scp to pull the file

for 1.
ssh username@server -p port command



