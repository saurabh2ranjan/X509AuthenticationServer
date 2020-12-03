# X509AuthenticationServer
This module contains articles about X.509 authentication with Spring Security

Relevant Articles:
X.509 Authentication in Spring Security
https://www.baeldung.com/x-509-authentication-in-spring-security

You can follow the article step by step and generate all the needed files by yourself.
Certificates can be created as below 

set openssl path in the environment variable
=============================================
openssl path = "D:\softwares\openssl-0.9.8k_X64\bin\openssl.exe"

Create CA Root.It will create root CA key and certificate.
==========================================================
openssl req -x509 -config "D:\softwares\openssl-0.9.8k_X64\openssl.cnf" -sha256 -days 3650 -newkey rsa:4096 -keyout rootCA.key -out rootCA.crt
(enter CN as "sr.com)

Create server certificate.It will create server key and an unsigned certificate.
=================================================================================
openssl req  -config "D:\softwares\openssl-0.9.8k_X64\openssl.cnf" -new -newkey rsa:4096 -keyout localhost.key –out localhost.csr	

sign the server certificate with our rootCA.crt certificate and its private key.
================================================================================
openssl  x509 -req  -CA rootCA.crt -CAkey rootCA.key -in localhost.csr -out localhost.crt -days 365 -CAcreateserial -extfile localhost.ext


package our server’s private key together with the above signed certificate.
============================================================================
openssl pkcs12 -export -out localhost.p12 -name “localhost” -inkey localhost.key -in localhost.crt

(enter pass phrase for localhost.key and set export password)

import the above package file to keystore.jks
==============================================
keytool -importkeystore -srckeystore localhost.p12 -srcstoretype PKCS12 -destkeystore keystore.jks -deststoretype JKS
(set destination keystore password)
(enter source keystore password  i.e. localhost.p12)


create TrustStore and add our root CA certificate in that
==========================================================
keytool -import -trustcacerts -noprompt -alias ca -ext san=dns:localhost,ip:127.0.0.1 -file rootCA.crt -keystore truststore.jks
(set keystore password)


Create Client certificate. It will create client private key and unsigned certificate
======================================================================================
openssl req  -config "D:\softwares\openssl-0.9.8k_X64\openssl.cnf" -new -newkey rsa:4096 -nodes -keyout clientBob.key –out clientBob.csr
Important: Give CN name as "Bob".

sign the client certificate with our CA certificate and CA key
===============================================================
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in clientBob.csr -out clientBob.crt -days 365 -CAcreateserial

package the client’s private key + signed certificate
======================================================
openssl pkcs12 -export -out clientBob.p12 -name "clientBob" -inkey clientBob.key -in clientBob.crt




