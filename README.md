# passgit

a password manager app design using git repository  
inspired by this video : `https://youtu.be/FhwsfH2TpFA`

## requirements for usage

- create a git repo
- create a RSA key pair
- store secret key in home dir or other dir
- put public key into local repo dir

## drawbacks
- if the secret key goes, the passwords wont be accesible 
- if the secret key is taken by someone else, the passwords will be accesible by someone else
- for this design, there cannot be a mobile app 

## apps
- a local api server 
- cli tool
- browser extension

### local api server

#### endpoints

- POST /add for adding a new credential

```json
{
  "host": "abc.com",
  "username": "said@yeter.com",
  "password": "123"
}
```
the post request to /add will create a dir called 'abc.com' in the repo  
then will create a file called '1'  which represents the nth credential for 'abc.com'  
then will encrypt the username and password with the public key  
and will put the encrypted username and password into a file  
then will run 'git add abc.com/1 && git push origin'
with that, the username/password will be stored on the git server as encrypted

- GET /getusernames/{hostname} to get all usernames related to the host
this request will go to {hostname} dir in repo
then will loop over on all files and will open the file and decrypt the first line which is the username using the secret key on the home dir
the response will look like this 
```json
[
    {
        "id": 1,
        "username": "said@yeter.com"
    },
    {
        "id": 2,
        "username": "said@gmail.com"
    }
]
```

- GET /getpassword/{hostname}/{id}
this request will go to {hostname} dir in repo
then will open the file called {id} and decrypt the second line which is the password using the secret key on the home dir
the response will look like this 
```json
{
  "password": "123"
}
```

- for the requirements, an endpoint like /init can be designed


### cli tool

#### commands

- passgit add  
it will ask hostname
after entering hostname
it will ask username
after entering username
it will ask password
after entering password
it will ask 'are you sure'
after approval, it will make a request to the local api server to add new credentials

- passgit get
it will ask hostname
after entering the hostname it will make a GET request to the local api server /getusernames/{hostname} to get all usernames related to the host
then will show usernames a list
user will be able to select a particular username all across the list
after selecting a username it will make a GET request to the local api server /getpassword/{hostname}/{id} to get a password related to the host and username and will copy it to the clipboard
then will return a message to the user like this: 'The password is copied to the clipboard'

### browser extension

- the extension will be active when it is opened,  
then it will check the local server by making a request  
if the server is not available then it will be disabled

- when a user goes to a site, the extension will get the hostname from URL and will make a GET request to the local api server /getusernames/{hostname} to get all usernames related to the host  
if there are any accounts, on the extension icon the extension will show a number as a badge that represents how many accounts are available related to the host  
if the user clicks on the extension icon it will create a popup and show the user name list. every row will show username/usericon/password icon  
if the user clicks on the user icon, the username will be copied to the clipboard, and if the user clicks on the password icon, the password will be copied to the clipboard  
for the user experience, parsing dom and putting username/password into related fields can be implemented 

- when the user clicks on the extension icon, whether there is an account or not, the popup will show an add button  
then as you may guess,  when the user clicks on the add button, a host/username/password form will be opened as the host is filled

## possible ways to make money 
- host a server to store secret keys for users to bring back with some security steps
- an app to change RSA key pairs and change all encrypted data
- show change history using git commands