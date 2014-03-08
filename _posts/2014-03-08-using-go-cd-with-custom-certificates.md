---
title: Using ThoughtWorks Go with custom certificates
layout: default
---

# Using \#gocd (ThoughtWorks Go) with custom certificates

The following post will help you setting up [Go](http://www.go.cd) to use your custom certificate instead of the self-signed certificate that Go ships with.

> Assumption: You have the certificate key (*.key) and certificate (*.crt)

## Step 1: Converting your certificate (*.crt) into PKCS12 format

If you have the key and certificate, you should export them to the pkcs12 format by running

```
$ openssl pkcs12 -inkey goserver.key -in goserver.crt -export -out goserver.pkcs12
```

## Step 2: Importing the PKCS12 store into the Java Keystore

Once you have the goserver.pkcs12 file, you would need to import this keystore into the java keystore that Go uses. We will use the **keytool** utility that ships with Java.

> Note: Destination keystore password **must** be set to **serverKeystorepa55w0rd**

```
$ keytool -importkeystore -srckeystore goserver.pkcs12 -srcstoretype PKCS12 -destkeystore keystore
Enter destination keystore password: serverKeystorepa55w0rd
Re-enter new password: serverKeystorepa55w0rd
Enter source keystore password:
Entry for alias 1 successfully imported.
Import command completed:  1 entries successfully imported, 0 entries failed or cancelled
```

## Step 3: Replacing the current Go keystore with the newly generated one

Now that the **keystore** (/tmp/keystore) is created, we'll replace the one that Go uses with this new one.

- Stop go-server

```
sudo /etc/init.d/go-server stop
```

- Change user to **go**

```
sudo su - go
```

- Change directory

```
cd /etc/go
```

- Backup the current keystore

```
go@/etc/go$ mv keystore keystore.original
```

- Copy over the new keystore

```
cp /tmp/keystore /etc/go
```

- Start go-server

```
sudo /etc/init.d/go-server start
```


## References

- [#gocd](http://www.go.cd)
- [Setting up self-signed SSL certificates for your Jetty instance (experiments with Noir and Clojure)](http://sharetheconversation.blogspot.in/2012/01/setting-up-self-signed-ssl-certificates.html)