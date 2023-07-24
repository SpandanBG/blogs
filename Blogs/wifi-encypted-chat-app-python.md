# WiFi Encypted Chat App (Python) 

---

Python WiFi Encrypted Chat CLI Application to communicate with peers in the network; encrypted with 2048 bit RSA.

---

Exam season; and like in all exam seasons where the wall looks way too interesting than anything else, I have stared at the wall for a little too long, and the boredom created the idea of an application with which I can have encrypted conversation with others using the hostel WiFi locally.

To do this during my exams I had to cut out the idea of a GUI app and stick with the old CLI interfaced application. The code is written in Python 2.7, cause no time for missing ;s.

The app required python's socket programming, synchronized multithreading and the PyCrypto library for RSA (Rivest–Shamir–Adleman) encryption. The idea was to create two threads:

- Client thread with which the user will have direct interactions, like passing commands or sending messages
- Server thread whose sole responsiblity is to send the app's RSA public key to a newly connected users and to receive messages from them.

The only dependency in the app is the PyCrypto which you can easily install using pip. Pip is python package manager system to easily install python packages. To get them you can run the following commands in linux:

```bash
apt-get install pip
pip install pycrypto
```

The name I gave to the application is WCanon, acronym for WiFi Chat Anonymous; github link to which is [here](https://github.com/SpandanBG/WCanon). The application allows three commands:

- *list* => To list all the users having the app under the X.X.X.1-254 IP
- *connect X.X.X.X *=> To connect to the user at X.X.X.X IP
- *exit *=> To exit the application (CTRL+Z will keep the server thread running in the background)

To send a message to the connected user simply type in the message and hit **ENTER**. If you're initially not connected with anyone, the message goes to your own server. The length of the message shouldn't be more than 256 characters since we're using 2048 bit RSA encryption and 2048 bit is equivalent to 256 bytes.

## How it works?

Let's divide the working of the application into four parts: Server thread, Client thread, User search thread and RSA Handler.

## Server Thread

The server thread has a pretty simple job. Keep the socket open for incoming messages and send RSA public keys to newly connected users. To do this a thread class is created that when ran opens a port and listens to it for incoming connections. If a new connection is made it sends the public key to the client, otherwise it decrypts the message received using the private key and displays the text to the user. The following is the current core code to the server thread:

```python
clientList = {}             # List tracking the uses who have already connected once
rsa = RSAHandler()          # This class comes later in the blog

# Opening the port with TCP IPv4 
server = socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)
server.bind((host, port))   # host='X.X.X.X' and port=1234

# 5 defines how many no. of concurrent connections 
# not accept()-ed by the process is allowed
server.listen(5)

while(True):
    c, addr = server.accept()   # waiting to accept connections
    msg = c.recv(4096)  # 4096 bytes of buffer space
    # If user already have connected once and non empty message
    if(len(msg)>0 and clientList.has\_key(addr\[0\])):
        msg = rsa.decrypt(msg)
        print '\\n',addr\[0\],' says: ',msg,'\\n'
    # If user connected for the first time
    else:
        c.send(rsa.getPublicKey())  # send the RSA public key
        clientList\[addr\[0\]\] = True  # add the user to clientList{}
    c.close()   # close connection to client
```

The socket created is binded to the IP address and port number on which the app is ready to receive connections. The <code>socket.AF_INET</code> implies IPv4 socket family and <code>socket.SOCK_STREAM</code> implies TCP socket type. The process keeps looping and waiting for connections. When a user connects, it checks if the user has previously connected by referencing to the <code>clientList</code> dictionary. If user has previously connected it follows on to decrypt the message and displaying it, otherwise it sends the public key to the client and adds it to the <code>clientList</code> dictionary.

## Client Thread

The client thread has a series of tasks. First it has to identify if the user has passed a command to the process or a message to be sent to the connected server. This can be easily done with a simple <code>if-elif</code> structure, so lets look into the functionalities of each commands:

```python
serverList = {}     # List tracking of already connected servers
rsa = RSAHandler()  # later in the blog

# Create a TCP IPv4 socket to connect to the server
client = socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)

# host = X.X.X.X => IP of the server to be connected to
# port = 1234 => the port on which the app will work
client.connect((host, port))  # Perform initial connection

# If connection is made for the first time
if(serverList.has\_key(host) == False):
    key = client.recv(4096)     # receive key
    serverList\[host\] = key      # push the host and key to the list

rsa.setOtherPublicKey(serverList\[host\]) # set the public key of the host
print 'Connected!\\n'    # notify successful setup
client.close()  # close connection
```

```python
rsa = RSAHandler() # later in the blog

# Create a TCP IPv4 socket to connect to the server
client = socket.socket(socket.AF\_INET, socket.SOCK\_STREAM)

# host = X.X.X.X => IP of the server to be connected to
# port = 1234 => the port on which the app will work
client.connect((host, port))

msg = rsa.encrypt(msg)  # RSA encrypt the message with private key
client.send(msg)    # send the message
client.close()  # close connection
```

```python
# we'll see this thread after this part
# there is a reason why it should be in a seperate thread
userList = UsersThead()     # create the thread class object
userList.start()            # run the thread
```

The client thread contains three primary methods:

- *List method*: To list users using the app under a specific domain range.
- *Connect method*: To test and setup initial connection to the server.
- *Send message method*: To encrypt messages and send it to the server.

## User search thread
You remember the List Method in the Client thread we just did above? The reason why this is a seperate thread is because searching for open socket in the app port takes time. Making it a seperate thread allows users to keep interacting with the app till it finds all users under that domain range. The following is the working:

```python
user = []   # the list of users

# myIP = X.X.X.X => the wlo1 IPv4 address
# appPort = 1234 => the port on which the app works
srcUnit = myIP[ : myIP.rfind('.')+1]    # extracting X.X.X. from myIP

# creating loop for domain range
for i in range(1,254):
    try:
        # creating TCP IPv4 socket
        search = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

        search.settimeout(0.3)      # setting 300ms of timeout time
        testIP = srcUnit+str(i)     # getting the new test IP
        search.connect((testIP, appPort))   # attempt at connecting to testIP on appPort
        search.settimeout(None)     # removing timeout

        # if no exception occured, connection success
        # add to user list
        self.user.append(srcUnit+str(i))
    except Exception as e:
        pass

# display users after found
for u in user:
    print u
```

The idea is simple. It uses the old hit-and-trial method to find the list of users using the app in the specified domain range. The domain range can be increased by simply creating nested for loops such that X.X.i.j range is obtained. This will obviously take time. The approx. time estimated for 300ms timeout, for the X.X.X.i range search, is 1.27 minutes.

## RSA Handler Class
The RSA handler is pretty simple. It creates a RSA private key and generates the public key with it. The class also stores the public key of the currently connected server. The following is the class code:

```python
import Crypto
from Crypto.PublicKey import RSA
from Crypto import Random
import ast

class RSAHandler:
    def __init__(self):
        self.rand_gen = Random.new().read
        self.privatekey = RSA.generate(2048, self.rand_gen)
        self.publickey = self.privatekey.publickey()
        self.otherPublicKey = None

    def setOtherPublicKey(self, key):
        try:
            self.otherPublicKey = RSA.importKey(key)
        except Exception as e:
            pass

    def getPublicKey(self):
        return self.publickey.exportKey()

    def encrypt(self, msg):
        try:
            return str(self.otherPublicKey.encrypt(msg,32))
        except Exception as e:
            return msg

    def decrypt(self, msg):
        try:
            return self.privatekey.decrypt(ast.literal_eval(str(msg)))
        except Exception as e:
            return msg
```

Fitting all these together created WCanon. The full code is avaible in my [github](https://github.com/SpandanBG/WCanon), feel free to check it out.

---

Spandan Buragohain,
2017-11-25 22:08:44
