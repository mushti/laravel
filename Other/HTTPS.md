#Setup HTTPS on Localhost

1. Install OpenSSL
https://slproweb.com/products/Win32OpenSSL.html

2. Generate RSA-2048 Key
You will be prompted for a pass phrase which you’ll need to enter each time you use this particular key to generate a certificate.
```
openssl genrsa -des3 -out rootCA.key 2048
```

3. Root SSL Certificate
This certificate will have a validity of 1,024 days. Feel free to change it to any number of days you want.
```
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem
```

4. Create Certificate Key
```
openssl req -new -sha256 -nodes -out localhost.csr -newkey rsa:2048 -keyout localhost.key -subj "/C=IN/ST=State/L=City/O=Organization/OU=OrganizationUnit/CN=localhost/emailAddress=demo@example.com"
```

5. Create v3.ext File
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
```

6. 
```
openssl x509 -req -in localhost.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out localhost.crt -days 500 -sha256 -extfile v3.ext
```

7. Configure httpd.conf
```
LoadModule ssl_module modules/mod_ssl.so
```