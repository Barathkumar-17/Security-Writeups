This time the setuid connects to a localhost on some port we specify and then we must enter the curr levle password from that side , then it(suconnect) will verify and return the password to the connection.

For this we need another terminal in which we will host a listening local server using nc , with port greater than 1000 ( anything below is standard ports specified for other purposes)

command:

nc -lvp 4545

for creating the listening server in another terminal with port 4545

when u connect using suconnect and port it doesnt work and just exits.

THis is because the suconnect wants the password as soon as connection and doesnt wait , 

so for this we pipe the password into the server hosting 

ie echo the password

echo password | nc -lp 4545

-l for creating listening server
p for port number 

-v will show status 

so use -lvp
