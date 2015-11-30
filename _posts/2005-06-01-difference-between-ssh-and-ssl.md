The following is an answer to Linkedin question :

## "What is the difference between SSH and SSL and how they work together?"

SSL (Secured Socket Layer) is an encryption protocol allowing secured communication between two computers. It ensures integrity, authentication and of course privacy. Some others protocols have been made or modified to include SSL support - notably HTTPS, FTPS AND SSH - so they provide the three mentioned benefits. 

SSH is both a secured communication protocol and a program that uses that protocol. SSH is mainly used Unix systems to access shell accounts (like a secured telnet) but it can be used as tunnels for other protocols that do not provide encryption (X11 over SSH, FTP over SSH, etc.). 

Even if their scope are similar, SSL and SSH are not designed to work the same way. 

* SSL is meant as a layer above or in an existing protocol : it provides cryptographic functions 
* SSH is mainly used to connect from/to remote servers 
* SSL does not require an authentication => you may browse anonymously in HTTPS sessions without giving user certificate. 
* SSH requires client authentication 
* If you want SSL to authenticate your users, it will work thanks to certificates exchange (can be a Public Key Infrastructure aka PKI). 
* SSH can do certificates exchange but allows more way to authenticate users.


---------

*Picture by [jasonwryan](http://www.flickr.com/photos/jasonwryan/)*
