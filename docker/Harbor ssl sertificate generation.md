 [[docker]]
 ## Generate a Certificate Authority Certificate

In a production environment, you should obtain a certificate from a CA. In a test or development environment, you can generate your own CA. To generate a CA certficate, run the following commands.

1.  Generate a CA certificate private key.
    
    ```sh
    openssl genrsa -out ca.key 4096
    ```
    
2.  Generate the CA certificate.
    
    Adapt the values in the `-subj` option to reflect your organization. If you use an FQDN to connect your Harbor host, you must specify it as the common name (`CN`) attribute.
    
    ```sh
    openssl req -x509 -new -nodes -sha512 -days 3650 \
     -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=so2harbor.com" \
     -key ca.key \
     -out ca.crt
    ```
    

## Generate a Server Certificate

The certificate usually contains a `.crt` file and a `.key` file, for example, `yourdomain.com.crt` and `yourdomain.com.key`.

1.  Generate a private key.
    
    ```sh
    openssl genrsa -out so2harbor.com.key 4096
    ```
    
2.  Generate a certificate signing request (CSR).
    
    Adapt the values in the `-subj` option to reflect your organization. If you use an FQDN to connect your Harbor host, you must specify it as the common name (`CN`) attribute and use it in the key and CSR filenames.
    
    ```sh
    openssl req -sha512 -new \
        -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=so2harbor.com" \
        -key so2harbor.com.key \
        -out so2harbor.com.csr
    ```
    
3.  Generate an x509 v3 extension file.
    
    Regardless of whether you’re using either an FQDN or an IP address to connect to your Harbor host, you must create this file so that you can generate a certificate for your Harbor host that complies with the Subject Alternative Name (SAN) and x509 v3 extension requirements. Replace the `DNS` entries to reflect your domain.
 ```sh
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=so2harbor.com
DNS.2=so2harbor
DNS.3=so2
EOF
```

4.  Use the `v3.ext` file to generate a certificate for your Harbor host.
    
    Replace the `yourdomain.com` in the CRS and CRT file names with the Harbor host name.
    
    ```sh
    openssl x509 -req -sha512 -days 3650 \
        -extfile v3.ext \
        -CA ca.crt -CAkey ca.key -CAcreateserial \
        -in so2harbor.com.csr \
        -out so2harbor.com.crt
    ```

## Provide the Certificates to Harbor and Docker

After generating the `ca.crt`, `yourdomain.com.crt`, and `yourdomain.com.key` files, you must provide them to Harbor and to Docker, and reconfigure Harbor to use them.

1.  Copy the server certificate and key into the certficates folder on your Harbor host.
    
    ```sh
    cp so2harbor.com.crt /data/cert/
    cp so2harbor.com.key /data/cert/
    ```
    
2.  Convert `yourdomain.com.crt` to `yourdomain.com.cert`, for use by Docker.
    
    The Docker daemon interprets `.crt` files as CA certificates and `.cert` files as client certificates.
    
    ```sh
    openssl x509 -inform PEM -in so2harbor.com.crt -out so2harbor.com.cert
    ```
    
3.  Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host. You must create the appropriate folders first.
    
    ```sh
    cp so2harbor.com.cert /etc/docker/certs.d/so2harbor.com/
    cp so2harbor.com.key /etc/docker/certs.d/so2harbor.com/
    cp ca.crt /etc/docker/certs.d/so2harbor.com/
    ```
    
    If you mapped the default `nginx` port 443 to a different port, create the folder `/etc/docker/certs.d/yourdomain.com:port`, or `/etc/docker/certs.d/harbor_IP:port`.
    
4.  Restart Docker Engine.
    
    ```sh
    systemctl restart docker
    ```
    

You might also need to trust the certificate at the OS level. See [Troubleshooting Harbor Installation](https://goharbor.io/docs/2.5.0/install-config/troubleshoot-installation/#https) for more information.

The following example illustrates a configuration that uses custom certificates.

```fallback
/etc/docker/certs.d/
    └── yourdomain.com:port
       ├── yourdomain.com.cert  <-- Server certificate signed by CA
       ├── yourdomain.com.key   <-- Server key signed by CA
       └── ca.crt               <-- Certificate authority that signed the registry certificate
```

5. also copy certificates to  /usr/local/share/ca-certificates/ 
```bash
cp . /user/local/share/ca-certificates
```

## Deploy or Reconfigure Harbor

If you have not yet deployed Harbor, see [Configure the Harbor YML File](https://goharbor.io/docs/2.5.0/install-config/configure-yml-file/) for information about how to configure Harbor to use the certificates by specifying the `hostname` and `https` attributes in `harbor.yml`.

If you already deployed Harbor with HTTP and want to reconfigure it to use HTTPS, perform the following steps.

1.  Run the `prepare` script to enable HTTPS.
    
    Harbor uses an `nginx` instance as a reverse proxy for all services. You use the `prepare` script to configure `nginx` to use HTTPS. The `prepare` is in the Harbor installer bundle, at the same level as the `install.sh` script.
    
    ```sh
    ./prepare
    ```
    
2.  If Harbor is running, stop and remove the existing instance.
    
    Your image data remains in the file system, so no data is lost.
    
    ```sh
    docker-compose down -v
    ```
    
3.  Restart Harbor:
    
    ```sh
    docker-compose up -d
    ```