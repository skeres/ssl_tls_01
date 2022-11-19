# Goals : Create self-signed certificates using OpenSSL in 2 steps
- Step 1 : Create our own root CA certificate Authority
- Step 2 : Create our Self-Signed Certificate with our root Authority

**Inspired from tutorial**  
[How to Create Self-Signed Certificates using OpenSSL ](https://devopscube.com/create-self-signed-certificates-openssl/)  

**Prerequisites :**
- Openssl installed on your host
      
**TL;DR**  
## > Context
You want to set up SSL/TLS communication between servers.  
The trick is here : we can be our own certificate authority (CA) by creating a self-signed root CA certificate,  
and then installing it as a trusted certificate in our the local browser.  
Dude, how are you gonna that ?  
Man, it's quiet simple, we will just generate 5 files.
- rootCA.crt  rootCA.key : encrypted, our root Authority to signe certificates  
- server.csr  server.key : encrypted, our web server information for example.   
- cert.conf : a temporary file, containing information non encrypted information about our web server  
- server.crt : last, but not least, our final SSL certificate four our web serveur.  

After that, we will use our server.crt file to configure our web server and set SSL, and rootCA.crt in our browser  
to inform him that our server can be trusted.  

## >> STEP 1 : Create our own root CA certificate Authority
We will use our rootCA.key and rootCA.crt to sign later the SSL certificate.  

### Create a dedicated directory for our work
```
mkdir openssl && cd openssl
```

### Create the rootCA.keyand rootCA.crt
```
openssl req -x509 \
            -sha256 -days 3650 \
            -nodes \
            -newkey rsa:2048 \
            -subj "/CN=my.root.authority.com/C=FR/L=PARIS" \
            -keyout rootCA.key -out rootCA.crt 
```
**result :**  
`cat rootCA.key`  
*-----BEGIN PRIVATE KEY-----*  
*MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCgvw/aU3Ais6Rh*  
*r/9cndaqMpqhVIVSgBgxoOh9ndzzf6Zs3Gq4U/LadhmeFjDWiF/CUYwKr+68OPZz*  
*dxn5KkFg6i4f43Bs1FwtusZAcGU6JiUNeuTbbS8PDXEWaNLhrgrbO3goxXBnzAt8*  
*... ( truncated )*  
*-----END PRIVATE KEY-----*  

`cat rootCA.crt`  
*----------BEGIN CERTIFICATE----------*  
*p4nYD0Ti/XrRPbPXGKuU9dsf4qacM82LRdSbTnzrN5TCYfsthA4Rem1yhppbmDaI*  
*5de4GkIjpdXgbVJFumo1NU+CQ9nmQ8nXiYiOcFKhN9af+N14oKnXvVxReBZXHAta*  
*RZZlEihgajoSFghvXyZr6NlLcTWmvifoDtwi/xJGnb8NetP4wyPiny5a5SvedW9G*  
*... ( truncated )*  
*----------END CERTIFICATE----------*  

All done ! End of step 1, we have our own CA certificate Authority






## >> Step 2 : Create our Self-Signed Certificate
3 things to do :   
- Generate our server private key  
- Generate a CSR Certificate Signing Request file : if we want our certificate to be signed by our root authority,  
we need a certificate signing request (CSR)  
- Generate SSL self signed certificate file with our CSR and root authority informations.  

### Generate server private key
```
openssl genrsa -out server.key 2048 && cat server.key  
```
**result :**  
*-----BEGIN PRIVATE KEY-----*  
*/P1ISBC64Mu6JfJJm+JCIFrwzvaJTCJgmSDZj/V5qQKBgQDfw32G5UeIm6AHt39t*  
*rRZ/WEQZ91cIcrqj1KRJ67pg/2RwQsFPEubhB97RJWTHnjYeT53XI5S4XGCBS5N2*  
*8ohcRBZXpBq6RVuP4N4y3d6M*  
*... ( truncated )*  
*-----END PRIVATE KEY-----

### Generate a CSR Certificate Signing Request
To create our signing request, we use the private server key generated above.  
You will be prompted to answer few questions. An important field is “Common Name,” which  
should be ***the exact*** Fully Qualified Domain Name (FQDN) of our domain.  
Example : www.yourdomain.com  
```
openssl req -new -key server.key -out server.csr
```
**result :**  
*You are about to be asked to enter information that will be incorporated into your certificate request.*  
*What you are about to enter is what is called a Distinguished Name or a DN.*  
*There are quite a few fields but you can leave some blank. For some fields there will be a default value,*  
*If you enter '.', the field will be left blank.*  
**  
*Country Name (2 letter code) [AU]:FR*  
*State or Province Name (full name) [Some-State]:FRANCE*  
*Locality Name (eg, city) []:PARIS*  
*Organization Name (eg, company) [Internet Widgits Pty Ltd]:myCie*  
*Organizational Unit Name (eg, section) []:section3*  
***Common Name (e.g. server FQDN or YOUR name) []:www.my-little-compagny.com** *  
*Email Address []:*  
**  
*Please enter the following 'extra' attributes*  
*to be sent with your certificate request*  
*A challenge password []:*  
*An optional company name []:*  

### Generate SSL self signed certificate
Create cert.conf for the SSL certificate. 
**Again, an important field is "alt_names"**. Past the content in the file, and replace value "www.my-little-compagny.com" 
with your own domain name or IP address.  
 
`touch cert.conf`  

```

authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = www.my-little-compagny.com

```


### Generate self signed SSL certificate.
Execute the following command to generate the SSL certificate that is signed by the rootCA.crt and rootCA.key  
created as part of our own Certificate Authority  
```
openssl x509 -req \
    -in server.csr \
    -CA rootCA.crt -CAkey rootCA.key \
    -CAcreateserial -out server.crt \
    -days 365 \
    -sha256 -extfile cert.conf
```
**result :**  
*Certificate request self-signature ok*  
*subject=C = FR, ST = FRANCE, L = PARIS, O = myCie, OU = section3, CN = www.my-little-compagny.com*  



### Before closing your computer 
Congrats ! You're good man !
 
Enjoy !! :sunglasses: :tropical_drink: :tropical_drink:
















