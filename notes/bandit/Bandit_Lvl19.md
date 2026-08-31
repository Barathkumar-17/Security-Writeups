The password is in /etc/bandit_pass/bandit20
but can only be opened using a user bandit20.

We are given a setuid binary , when we execute it as it is , we end up with a reply called 

run a command as another user.

so this implies that using this i can run a command as user bandit20 .

./bandit20-do cat /etc/bandit_pass/bandit20

will display the password .

to identify such files look out for the tag 's' in permissions for a file 
