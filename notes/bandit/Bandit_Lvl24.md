So this level requires script writing again.

The password for next level is obtained by sending the current level password + 4 digit pin combination to a localhost at 30002 listening .

SO we use nc for the server listening and sending password part , but we cant try all 10^4 combinations manually each connection , for this what we can do is write a script which will take each password combination give it as input to the server opened using | and echo then store output in a file ..

Script:

for i  in  {0..9}{0..9}{0..9}{0..9};
	do  echo "hVQMk3lJNsmQ7VF3ubyrNNBom7BOgVXv $i"; 
done | 
nc localhost 30002 > pass.txt

