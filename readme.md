### python

Set up python 3.10.1

```
brew install pyenv

pyenv install 3.10.1
```


### Poetry

there are a few tools to manage the dependencies, I’m using poetry,

to use poetry first install poetry on your machine 

`~ $ brew install poetry`

### OS package dependency

- `xmlsec1` - a requirement for [`pysaml2`](https://pysaml2.readthedocs.io/en/latest/install.html#install-pysaml2)
  ```
  brew install libxmlsec1
  ```

### Setting up

1. Clone the repository and go into its folder:

   ```sh
   git clone git@github.com:inbalzelinger/sample_sp_app.git
   cd sample_sp_app
   ```

2. Set the local Python interpreter: `pyenv local 3.10.1`.


It is always a good practice to create a separate  python virtual env for each project and menage the dependencies there.

 to create a new virtual env:

```
~/workspace/sample_sp_app $ python3 -m venv .venv

~/workspace/sample_sp_app $ source .venv/bin/activate
```

3. Install dependencies: `(venv) ~/workspace/sample_sp $ poetry install`.

### run the application:

`python3 manage.py runserver`

### define SSO app on jumpCloud:

1. log in to Jump Cloud as an Administrator.
2. Go to SSO
3. press + to add an application
4. Chose custom SAML App

General Info:

  Display label: `Sample_app`

  Chose Logo and write your description

SSO:

  Idp Entity Id:  `jumpcloud/twingate/sample-sp`

  Sp Entity Id: `http://localhost:8000/sample_sp`

  **note**: Those entity ids could be any string, but usually we will set then to a url.

  Acs Url: `http://127.0.0.1:8000/saml2/acs/`
    
  SAMLSubject's NameID: `email`
    
  SAMLSubject's NameID Format: `urn:oasis:names:tc:SAML:1.0:nameid-format-unspecified`
    
  Signature Algorithem: `RSA-SHA256`
    
  sign assertion: keep unmarked

  Default RelayState: this is the url that the user will actually land in after a successful response verification. 
    For an SP initiate SSO  this url is already known, meaning this is the URL that the user is trying to access on the app. 
    **So let’s keep empty.**
    
  Login Url: `http://127.0.0.1:8000/saml2/login` 
    
  Declare Redirect Endpoint: keep unmarked
    
  Idp Url: `https://sso.jumpcloud.com/saml2/saml2`

  Click **activate** to save and activate the connector. After the application is activated, a public certificate and private key pair are generated for the application.
    Download the public certificate and private key pair 
    back to our service provider app, in the [saml.py](http://saml.py), remember this? `x509_cert="<add this later>"`
    
  open the `.pem` file that we just downloaded from JumpCloud, and copy its content. 
  We need the certificate in string format, and for that we can use [onelogin SAML tools](https://www.samltool.com/format_x509cert.php)
    
  now copy the cert in string format to the `x509_cert` key as a value


### check the app

ready to check our app. let's run our Django service provider 
`(venv) ~/workspace/sample_sp $ python3 [manage.py](http://manage.py/) runserver`

lets access the login page of our app in the browser:

[http://localhost:8000/saml2/login](http://localhost:8000/saml2/login)

and, what a nice magic, we got redirected to the Jumpcloud login page, 

now let's login into jump cloud with the user that has access to our app

after the successful login to Jumpcloud we will immediately redirect to our SP acs url with a SAML request.

