# Bolt card service installation

## hardware & o/s

1 GHz processor, 2 GB RAM, 10GB storage minimum  
Ubuntu 20.04 LTS server

### install Go

[Go download & install](https://go.dev/doc/install)  
`$ go version` >= 1.18.3

### install Postgres

[Postgres download & install](https://www.postgresql.org/download/linux/ubuntu/)  
`$ psql --version` >= 12.11

### install Caddy

[Caddy download & install](https://caddyserver.com/docs/install)  
`$ caddy version` >= 2.5.2

### download the boltcard repository

`$ git clone https://github.com/boltcard/boltcard`

### get a macaroon and tls.cert from the lightning node

create a macaroon with limited permissions to the lightning node  
```
$ lncli \                                                    
--rpcserver=lightning-node.io:10009 \
--macaroonpath=admin.macaroon \
--tlscertpath="tls.cert" \
bakemacaroon uri:/routerrpc.Router/SendPaymentV2 > SendPaymentV2.macaroon.hex

$ xxd -r -p SendPaymentV2.macaroon.hex SendPaymentV2.macaroon
```

### setup the boltcard server
edit `boltcard.service` in the section named `boltcard service settings`  
edit `Caddyfile` to set the boltcard domain name  
edit `add_card_data.sql` to set up the individual bolt card records

### database creation
`$ ./s_create_db`

### boltcard service install
`$ sudo cp boltcard.service /etc/systemd/system/boltcard.service`  
`$ ./s_build`  
`$ systemctl status boltcard`

### https setup
set up the domain A record to point to the server  
set up the server hosting firewall to allow open access to https (port 443) only  

### caddy setup for https
`$ sudo cp Caddyfile /etc/caddy`  
`$ sudo systemctl stop caddy`  
`$ sudo systemctl start caddy`  
`$ sudo systemctl status caddy`  
you should see 'certificate obtained successfully' in the service log

### service bring-up and testing
#### service log
the service log should be monitored on a separate console while tests are run  
`$ journalctl -u boltcard.service -f`
#### local http
`$ curl http://127.0.0.1:9000/ln?1`  
this should respond with 'bad request' and show up in the service log  
#### remote https
navigate to the service URL from a browser, for example `https://card.yourdomain.com/ln?2`  
this should respond with 'bad request' and show up in the service log  
#### bolt card
[create a bolt card](docs/CARD.md) with the URI pointing to this server  
use a PoS setup to read the bolt card, e.g. [Breez wallet](https://breez.technology/)   
monitor the service log to ensure decryption, authentication, payment rules and lightning payment work as expected  

# Further information and support

[bolt card FAQ](docs/FAQ.md)  
[bolt card telegram group](https://t.me/bolt_card)