---
title: Enabling SSL-only access on postgreSQL server
layout: default
---

# Enabling SSL-only access on postgreSQL server

The following post will help you setup a postgreSQL server with SSL-only access.

> Assumption: We will be using a self-signed certificate for this demo.

## Generating a self-signed certificate

We'll be following the steps outlined in the [postgresql documentation](http://www.postgresql.org/docs/9.2/static/ssl-tcp.html) to generate a self-signed certificate.

```bash
openssl req -new -text -out server.req
```

While running the above command, all fields except CN can be set to blank. Common Name (CN) should be set to the hostname of the postgreSQL server.

Create a new key by running the below command. Also, remove the private key after successfully creating a key file.

```bash
openssl rsa -in privkey.pem -out server.key
rm privkey.pem
```

Unlock the newly created key by running the below command. Also change permission of the key file.

```bash
openssl req -x509 -in server.req -text -key server.key -out server.crt
chmod og-rwx server.key
```

Copy the server key and certificate to $PGDATA directory

```
cp server.key $PGDATA
cp server.crt $PGDATA
```

## Configuring postgreSQL to enable SSL

Edit ```postgresql.conf``` in ```$PGDATA``` directory and uncomment the below lines

```
ssl = on
ssl_ciphers = 'ALL:!ADH:!LOW:!EXP:!MD5:@STRENGTH'
ssl_renegotiation_limit = 512MB
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
```

## Configuring postgreSQL to disallow non-SSL communication

Edit ```pg_hba.conf``` in ```$PGDATA``` directory and add the below line.

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD

hostssl	all		        all		        127.0.0.1/32		    md5
```

**IMPORTANT**: Comment every other line in ```pg_hba.conf``` to mandate SSL-only communication.

```
# "local" is for Unix domain socket connections only
#local   all             all                                     md5
# IPv4 local connections:
#host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
#host    all             all             ::1/128                 md5
```

## Verification

Restart postgreSQL server after all the above changes are completed successfully.

```bash
PGDATA=`pwd` ../bin/pg_ctl restart
```

When you attempt to establish a connection using the postgreSQL client (psql), you should see a message **SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)**, confirming that an SSL connection has been established.

```bash
psql -h 127.0.0.1 -U user -d testing
Password for user user: 
psql (9.2.4)
SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
Type "help" for help.

testing=> 
```

## References

- [Secure TCP/IP Connections with SSL](http://www.postgresql.org/docs/9.2/static/ssl-tcp.html)
