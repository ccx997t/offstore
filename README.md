# Edge Node Deployment

Edge nodes construct the underlying P2P messaging network. They play the role of relaying messages, providing verifiable storage, supporting video streaming, and serving as network entries for client applications.

The edge node communication meets the requirements of the _libp2p_ specification, supporting protocols like _TCP_, _UDP_, _WS_, _WSS_, etc. We offer diverse options for configuring your node.

### System Requirements

* OS: Linux
* CPU: Intel Core i3 or equivalent
* Storage: 200 GB internal storage drive
* Memory: 8 GB RAM
* Internet Speed: 10 Mbps
* A public IP address is strongly recommended

### Costs of Hosting

Edge nodes can be hosted on any cloud server or local device that meets the system requirements. Depending on the network traffic, the monthly hosting expense could vary. Under the typical situation, running the node on Amazon Web services costs around 200 USD per month.

### Preparation

Edge Node uses a local postsql database. For setting up an edge node for the first time, you need to install the database and configure the login credentials. Take Ubuntu as an example:

```bash
sudo apt-get install postgresql postgresql-client
sudo su postgres
psql
\password postgres # replace "password" with your password
```

If the database didn't start up automatically after installation, do it manually:

```bash
sudo pg_ctlcluster 12 main start # 12 is the version number
```

Now we have set the default username _postgres_, password _a_, and database name _postgres_.

Create a new user credentials (you may omit the following steps if keep using _postgres_):

```sql
CREATE USER storenode WITH PASSWORD '123456';
```

Create database for new user:

```sql
CREATE DATABASE db_storenode OWNER storenode;
```

Grant full access to the database:

```sql
GRANT ALL PRIVILEGES ON DATABASE  db_storenode to storenode;
```

For detailed database installation info, please refer to this [blog](https://www.cherryservers.com/blog/how-to-install-and-setup-postgresql-server-on-ubuntu-20-04).

### Installation

**Step 1. Download the edge node package** [**here**](https://p2p-alpha.sending.me/dendrite-libp2p.zip)

**Step 2. Extract the binary to the installation location**

```
unzip dendrite-libp2p.zip
```

**Step 3. Run edge node application**

To support client applications using _TCP_, _UDP_ and _WS_ protocols, e.g., applications running on iOS, Android, etc.:

```shell
./dendrite-libp2p -offPgUser=postgres -offPgPasswd=123456 -offPgDb=postgres -port=-1 -listen=-1 -listenWs=9085 -ipfs=true -public=true -offMode=true -offserver=true -offProKv=true -disableFileLog=false -announce=/ip4/111.200.195.194/tcp/9085/ws,/dns4/p2p-test.analyst.ai/tcp/9010/ws  -peer=/ip4/10.15.10.43/tcp/8082/ws/p2p/12D3KooWEbqSC2vSudddmPARBK1nbypStxvo9wWGoqhzrBRc7N2e,/ip4/10.15.10.44/tcp/8082/ws/p2p/12D3KooWPUNBE3CmsgTkVwXxwm7ch87BCg1Kspa5SGd997qWxb32 -disableMdns=true
```

To support web client applications that use _WSS_ protocol, specify the _WSS_ address with `-announce` and `-peer`. Also, you need to configure the domain name, _Nginx_ proxy, and certificate. Please refer to Step 4 for detailed information.

```shell
./dendrite-libp2p -port=-1 -listen=-1 -listenWs=9085 -ipfs=true -public=true -offMode=true -offserver=true -offProKv=true -disableFileLog=false -announce=/ip4/111.200.195.194/tcp/9085/ws,/dns4/p2p-test.analyst.ai/tcp/443/ws,/dns4/p2p-test.analyst.ai/tcp/9010/wss  -peer=/ip4/10.15.10.43/tcp/8082/ws/p2p/12D3KooWEbqSC2vSudddmPARBK1nbypStxvo9wWGoqhzrBRc7N2e,/ip4/10.15.10.44/tcp/8082/ws/p2p/12D3KooWPUNBE3CmsgTkVwXxwm7ch87BCg1Kspa5SGd997qWxb32，/dns4/p2p-test.boostrap/tcp/443/wss/12D3pooWPUNBE3CmsgTkVwXxwmXkk87BCg1Kspa5SGd997qeXIKx -disableMdns=true
```

_**Description**_

`-listenWs` Specify the WebSocket listen port if you want to use the _WS_ protocol; otherwise, set it to -1. Make sure you set at least one of the `-listen` or `-listen`.

`-listen` Specify the port used for _TCP_ and _UDP_ if these protocols are in use; otherwise, set it to -1. Make sure you set at least one of the `-listenWs` or `-listen`.

`-port` Specify the port for receiving API requests. Set to -1 in most cases since the edge node is seldom used as a client.

`-ipfs` Set `true` to support the IPFS feature.

`-public` Set `true` if the server has a public IP address and port; otherwise, set `false`.

`-offMode` Set `true` to provide services as an edge node.

`-offserver` Set `true` to support offline message caching.

`-offProKv` Set `true` to support DID document storage; otherwise, set `false`.

`-disableFileLog` Set `true` to disable log recording; otherwise, set `false`.

`-announce` Specify the public IP addresses and ports explicitly if applicable, separated by commas. If `-public` is set to `false` and no `-announce` value is provided, the node can only be reached by relays, leading to reduced performance and connectivity.

`-peer` Specify the bootstrapping node(s) addresses, separated with commas.

`-disableMdns` Disable MDNS service. MDNS is used to find and connect to edge nodes inside the intranet.

`-offPgUser` Specify the database login user name.

`-offPgPasswd` Specify the database login password.

`-offPgDb` Specify the database name.

**Step 4. Web client applications related (optional)**

To support web client applications, you need to specify the `-announce` and `-peer` with _WSS_ protocol address with a domain name.

1. Prepare a domain name.
2.  Configure a proxy to verify the certificate, convert the _WSS_ protocol to _WS_ and forward it to the edge node listen port.

    You can find more about _Nginx_ proxy configurations [here](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04).

    You can apply for a certificate with _Cerbot_, detailed configuration can be found [here](https://certbot.eff.org/instructions?ws=nginx\&os=ubuntufocal).

Below is a sample _Nginx_ configuration:

```nginx
server {
    server_name p2p-test.sending.network;
     
    listen 9010 ssl http2;
    listen [::]:9010 ssl http2;
 
    access_log /var/log/nginx/p2p-test.sending.network_access.log;
    error_log /var/log/nginx/p2p-test.sending.network_error.log;
 
    #listen [::]:443 ssl ipv6only=on; # managed by Certbot
    #listen 443 ssl; # managed by Certbot
    # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/p2p-test.sending.network/fullchain.pem;
    # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/p2p-test.sending.network/privkey.pem;
    # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf;
    # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
 
    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
 
    location /p2p/12D3KooWLPZrPkxSbnx6bcmkFyYTeJVApModhhu7b16YtPGXQAA5 {
        proxy_pass http://127.0.0.1:9085;
    proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
 
}
```

* The external port (_Nginx_ proxy port) is 9010, and the service listen port is 9085.
* The encryption key generated by _Cerbot_ locates at `/etc/letsencrypt/`.
*   You can get the `peerID` by searching in the log file:

    `grep 'Listening on' -r ./log | grep "peer id"`

    The output looks like:

    ```
    Dec 08 03:05:09 ip-172-31-34-109 storenode[8932]: time="2022-12-08T03:05:09.053437868Z" level=info msg="Listening on 9d163f247d73d28c2dfaa0f2bce1a0e9e7785316c580f7a2279a1c244cb2b02a, peer id: 12D3KooWLPZrPkxSbnx6bcmkFyYTeJVApModhhu7b16YtPGXQAA5"
    ```

    In this case, `12D3KooWLPZrPkxSbnx6bcmkFyYTeJVApModhhu7b16YtPGXQAA5` is the `peerID`.
