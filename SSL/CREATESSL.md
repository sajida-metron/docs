## Create SSL Certificate using openssl

For development purposes, you can use the following tools and websites to generate SSL certificates:

OpenSSL (Command-line tool)

Best for: Developers comfortable with the command line.

OpenSSL is a powerful tool for generating SSL certificates, including self-signed certificates, which are perfect for local development.

How to use:

Install OpenSSL if you don't have it already.

Run commands to generate a private key and a self-signed certificate for development purposes.

### Example
```bash
    openssl genpkey -algorithm RSA -out private.key
    openssl req -new -key private.key -out csr.csr
    openssl x509 -req -in csr.csr -signkey private.key -out certificate.crt
```
To generate an SSL certificate using OpenSSL on Ubuntu, follow these steps:
### Step 1: Install OpenSSL
 If you don’t have OpenSSL installed, install it using the following command:
 ```bash
    sudo apt update
    sudo apt install openssl
```
### Step 2: Generate a Private Key
A private key is required to generate the SSL certificate. Run the following command to generate a private key:
```bash
  openssl genpkey -algorithm RSA -out private.key
```
This will create a private key file (private.key) in the current directory.
### Step 3: Create a Certificate Signing Request (CSR)
Next, create a Certificate Signing Request (CSR) using the private key. A CSR is required if you're requesting an SSL certificate from a Certificate Authority, but for self-signed certificates, it’s still necessary for the process.

Run the following command:
```bash
  openssl req -new -key private.key -out csr.csr
```
You'll be prompted to enter some details about your organization and the domain name for which the certificate is being generated. For development, you can enter anything (e.g., localhost or your local domain).
Example:
```psql
  Country Name (2 letter code) [AU]: US
  State or Province Name (full name) [Some-State]: California
  Locality Name (eg, city) []: San Francisco
  Organization Name (eg, company) [Internet Widgits Pty Ltd]: MyCompany
  Organizational Unit Name (eg, section) []: IT
  Common Name (e.g. server FQDN or YOUR name) []: localhost
  Email Address []: admin@mycompany.com
```
### Step 4: Generate a Self-Signed Certificate
Now that you have a private key and CSR, you can generate a self-signed SSL certificate. This certificate will be valid for testing and development purposes, but browsers will treat it as insecure.

Run the following command:
```bash
  openssl x509 -req -in csr.csr -signkey private.key -out certificate.crt -days 365
```
The -days 365 option specifies that the certificate will be valid for 365 days. You can adjust the duration as needed.
This command will generate the SSL certificate file (certificate.crt).
### Step 5: Verify the Certificate
You can verify the contents of the certificate using this command:
```bash
  openssl x509 -in certificate.crt -text -noout
```
## Convert your certificate and private key to .pem format
 If your certificate and private key are not in .pem format (they may be in .crt and .key formats), you can convert them using OpenSSL.

### Convert your certificate to .pem
If your certificate is in .crt format, you can convert it to .pem using this command:
```bash
  openssl x509 -in certificate.crt -out certificate.pem -outform PEM
```
### Convert your private key to .pem
If your private key is in .key format, you can convert it to .pem using this command:
```bash
  openssl rsa -in private.key -out private.pem
```
Now you should have two .pem files:

* certificate.pem (SSL certificate)
* private.pem (Private key)

## Flask Application to Use .pem Files
Once you have the .pem files, you can modify your Flask application to use them for SSL.

Here's how you can modify your Flask app to use the .pem files:
```bash
   from flask import Flask

  app = Flask(__name__)
  
  @app.route('/')
  def hello_world():
      return "Hello, SSL World!"
  
  if __name__ == '__main__':
      # Use the .pem files for SSL
      app.run(debug=True, host='0.0.0.0', port=443, ssl_context=('certificate.pem', 'private.pem'))
```
Run your Flask application with the following command:
Your Flask app will now be served over HTTPS using the .pem files.

### Notes:
PEM format: .pem is just a container format that can store both the certificate and the private key. It’s often used for certificates in web servers, including Apache and NGINX.

Security: If your private key is encrypted (protected with a password), Flask won’t be able to use it directly without the password, as mentioned earlier. You would either need to decrypt the private key or handle the password manually via ssl.create_default_context as described before.

## Use .pem in JAVA(Spring Boot Service)
To enable SSL/TLS for your Spring Boot application using a .pem certificate file, you'll need to follow these steps:
### Step 1: Convert .pem files into Java Keystore (.jks) format
Spring Boot uses a Java Keystore (JKS) or PKCS12 format for SSL/TLS certificates. You can't directly use a .pem file in Spring Boot without converting it into a format that Java can read. You can use the keytool utility (which comes with Java) to convert your .pem files into a .jks (Java Keystore) or .p12 (PKCS12) file.
### Example: Convert .pem to .p12 or .jks
1. Combine the certificate and private key into a .p12 (PKCS12) file:
     If you have a certificate (certificate.pem) and private key (private.pem), you can combine them into a .p12 file with OpenSSL, and then import it into a Java keystore.
```bash
     openssl pkcs12 -export -in certificate.pem -inkey private.pem -out keystore.p12
```
  * certificate.pem: The certificate file in PEM format.
  * private.pem: The private key file in PEM format.
  * keystore.p12: The output file in PKCS12 format.
You will be asked to set an export password. Keep it in mind, as you will need it later.
2. Convert the .p12 file to a .jks keystore (if needed):
   While Spring Boot can work with PKCS12, some configurations may require a JKS keystore format. If you need a .jks file, you can convert it with the keytool utility:
 ```bash
    keytool -importkeystore -srckeystore keystore.p12 -srcstoretype PKCS12 -destkeystore keystore.jks -deststoretype JKS
```
* keystore.p12: The PKCS12 file you generated earlier.
* keystore.jks: The Java keystore file you want to create.
  You'll be asked for a password for the keystore during this process.

### Step 2: Configure Spring Boot to Use the Keystore
Once you've generated the .jks or .p12 keystore, you need to configure Spring Boot to use it for SSL.

Edit application.properties (or application.yml):
You’ll need to configure your application.properties or application.yml to point to your keystore file and set the password.

### For application.properties:
```bash
  server.port=443
  server.ssl.key-store-type=PKCS12  # Or JKS if using Java Keystore
  server.ssl.key-store=classpath:keystore.p12  # Path to your keystore file
  server.ssl.key-store-password=your_keystore_password
  server.ssl.key-alias=your_alias   # The alias for your key (if applicable)
```
* server.ssl.key-store: The path to your keystore file. If the keystore is in the resources directory, you can use classpath:.
* server.ssl.key-store-password: The password you set for the keystore when generating the .p12 or .jks file.
* server.ssl.key-alias: The alias of the key inside the keystore (this is optional depending on your keystore setup).
### For application.yml:
 ```bash
  server:
  port: 443
  ssl:
    key-store-type: PKCS12  # Or JKS
    key-store: classpath:keystore.p12
    key-store-password: your_keystore_password
    key-alias: your_alias  # Optional
```
### Step 3: Ensure Spring Boot Runs with SSL
Once you’ve updated the application.properties or application.yml file, Spring Boot will automatically use the provided keystore for SSL/TLS configuration.
### Step 4: Run Your Spring Boot Application
```bash
  mvn spring-boot:run
```

### Step 5: Test Your HTTPS Service
To test, open a browser and navigate to https://localhost (or the appropriate URL for your application). Your browser should now establish a secure connection using the certificate you provided. If you used a self-signed certificate, the browser will likely show a warning, but you can proceed to the site.


