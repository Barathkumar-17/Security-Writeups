For this level it uses  SSL/TLS encryption ,

Transport Layer Security , for the server,server to handshake they must agree on encryption rules then they will exchange information as required 

for this we use openssl

so it does the handshake and stuff then everything else same as openssl which will act as TLS client and do the handshake.

for this handshake we use the command 

openssl s_client (client connecting to localhost server)

for specifying host , port we use the sub commands
-host localhost 
-port 30001

so final command

openssl s_client -host localhost -port 30001
