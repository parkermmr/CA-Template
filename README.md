#### Generating a Root CA

```bash
openssl req -x509 -config openssl-ca.cnf -newkey rsa:4096 -sha256 -nodes -out root-ca.cert -keyout root-ca.key -days 3650
```

#### Generate server certificate:

```bash
# Create server private key and CSR
openssl req -new -newkey rsa:4096 -keyout < server >.key -out < server >.csr -config openssl-ca.cnf

# Sign the server CSR with CA
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions server_cert -extfile < server {required} >.ext -out < server >.crt -infiles < server >.csr
```

#### Generate client certificate:

```bash
# Create client private key and CSR
openssl req -new -newkey rsa:4096 -nodes -keyout < client >.key -out < client >.csr -config openssl-ca.cnf

# Sign the client CSR with CA
openssl ca -config openssl-ca.cnf -policy signing_policy -extensions client_cert -extfile < client {optional} >.ext -out client.crt -infiles client.csr
```

#### Adding Keys to Keystore (Generally Only Server)

```bash
# Convert PEM certificate and key to PKCS12 format
openssl pkcs12 -export -out < server >.p12 \
  -inkey < server >.key \
  -in < server >.crt \
  -certfile root-ca.cert \
  -name "< server >" \
  -password pass:< password >

# Import PKCS12 into Java keystore (JKS format)
keytool -importkeystore \
  -srckeystore < server >.p12 \
  -srcstoretype PKCS12 \
  -srcstorepass < password > \
  -destkeystore < server >.keystore.jks \
  -deststoretype JKS \
  -deststorepass < password > \
  -alias < server >
```

#### Adding Certs to Trustore

Creating a truststore.

```bash
# Import CA certificate into truststore (JKS)
keytool -import -trustcacerts \
  -alias < cert > \
  -file < cert >.cert \
  -keystore < name >.truststore.jks \
  -storepass < password > \
  -noprompt

# Or as PKCS12
keytool -import -trustcacerts \
  -alias < cert > \
  -file < cert >.cert \
  -keystore < name >.truststore.p12 \
  -storetype PKCS12 \
  -storepass < password > \
  -noprompt
```

Importing a certificate to a truststore.

```bash
# Import CA
keytool -import -trustcacerts -alias < cert > -file < cert >.cert \
  -keystore < name >.truststore.jks -storepass changeit -noprompt
```

### Verifying Truststore

```bash
# List entries in keystore
keytool -list -v -keystore keystore.jks -storepass changeit

# List entries in PKCS12 keystore
keytool -list -v -keystore keystore.p12 -storetype PKCS12 -storepass changeit

# List trusted certificates
keytool -list -v -keystore truststore.jks -storepass changeit
```

### Configuring Java Applications

```bash
# For server (needs both keystore and truststore)
java -Djavax.net.ssl.keyStore=keystore.p12 \
     -Djavax.net.ssl.keyStorePassword= \
     -Djavax.net.ssl.keyStoreType=PKCS12 \
     -Djavax.net.ssl.trustStore=truststore.jks \
     -Djavax.net.ssl.trustStorePassword= \
     -jar your-app.jar

# For client (usually just needs truststore)
java -Djavax.net.ssl.trustStore=truststore.jks \
     -Djavax.net.ssl.trustStorePassword= \
     -jar your-client-app.jar
```