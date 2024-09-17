

# Sederio TALOS LINUX Omni Dashboard Installation


Perquisite's

1. Cloudflare [DNS TOKEN](https://dash.cloudflare.com/profile/api-tokens) & ZoneID and the email address that it's registered to and with IP
2. Setup an SAML Enterprise Application in MS365 Entra
3. Restrictions added for IPV4 & IPV6 Address belonging to the server.
4. Linux Distro of choice but I have used Hetzner Ubuntu Cloud 22.04 LTS Docker CE
5. Basic Knowledge of Linux commands like cd .. , cp, sudo, etc...... nano, vi, 
6. Docker and Docker compose installed and the user added to the docker group
7. Create a omni directory in your /home/USER/ 
8. Install acme.sh from your home directory.
9. Setup firewall rules Ports can be obtained from the compose.yaml file.
10. Install and run a GPG key gen for the omni.asc file & copy it to the omni directory
11. We need to create a cron Job for copying the renewed SSL certs into the omni directory and restart the container.


---

> [!NOTE]
> FIREWALL RULES ON HETZNER 

[FIREWALL RULES](Documentation/omni-hetzner-firewall-rules.png)
Documentation/omni-hetzner-firewall-rules.png

---





> After you sign in make sure you run the below command to make yourself part of the docker group to make things easier for yourself at a later stage.

```bash
sudo usermod -aG docker $USER && EXIT
```






---



# Install Acme.sh with Email notification

```bash
https://get.acme.sh | sh -s email=patrick.woodlock@rockfieldit.com
```

Go into the acme.sh folder and look for the cf.sh script and insert your TOKEN, not API key and the Zone id along with your email address registered in for the token and domain name.


```
sudo nano ./.acme.sh/dnsapi/dns_cf.sh
```


# Example below

---


```bash
#CF_Key=""

CF_Token="THE TOKEN YOU GENERATED HERE INCLUDING QUOTES"

#CF_Account_ID=""

CF_Zone_ID="ZONE ID HERE INCLUDING THE QUOTES"


CF_Api="https://api.cloudflare.com/client/v4"

########  Public functions #####################
```

## Following that, then you need to go back to your home user directory and run this command, and of course omit the domains to yours.  I have Advanced Cert Subscription with my CF account



> [!NOTE]
> For standard wildard use the below command



```bash
acme.sh --issue --dns dns_cf -d rockfieldit.com -d '*.rockfieldit.com'
```



> [!NOTE]
> For advanced Cloudflare Subscription you can use this type of command.



```bash

acme.sh --issue --dns dns_cf -d talos.1.devsec.ie -d '*.talos.1.devsec.ie'

```


> [!NOTE]
> Your output should look something like this:


```c
[Sat Mar  9 02:45:38 PM UTC 2024] Your cert is in: /home/patrick/.acme.sh/talos.1.devsec.ie_ecc/talos.1.devsec.ie.cer

[Sat Mar  9 02:45:38 PM UTC 2024] Your cert key is in: /home/patrick/.acme.sh/talos.1.devsec.ie_ecc/talos.1.devsec.ie.key

[Sat Mar  9 02:45:38 PM UTC 2024] The intermediate CA cert is in: /home/patrick/.acme.sh/talos.1.devsec.ie_ecc/ca.cer

[Sat Mar  9 02:45:38 PM UTC 2024] And the full chain certs is there: /home/patrick/.acme.sh/talos.1.devsec.ie_ecc/fullchain.cer
```

Once the above has completed correctly, you then need to get the certs the and convert  and copy them into the omni directory with this command.


```bash

acme.sh --install-cert -d rockfieldit.com --cert-file ./omni/tls.crt --key-file ./omni/tls.key --fullchain-file ./omni/chain.crt

```


OR for the normal wildcard mode:


```bash

acme.sh --install-cert -d devsec.ie --cert-file ./omni/tls.crt --key-file ./omni/tls.key --fullchain-file ./omni/chain.crt

```



```
## Update Certificates Script:

Create a script `update-omni-certs.sh` to automatically update the certificates in the `/home/patrick/omni/` directory upon renewal.  Of course you will have to adapt this to your own directory structure
```



---

## Now it's time to create a gpg Key for the Kubernetes ectd folder/database

> [!NOTE]
> NOTE: That you do not password protect the files, so say no to everything!!!!



```bash
gpg --quick-generate-key "patrick.woodlock@rockfieldit.com" rsa4096 cert never && gpg --list-secret-keys

```

Following that please take note of your key from the output. 


```bash
gpg --quick-add-key 24A46215B3F82E6E1B77815DD7A804F9C217289B rsa4096 encr never
```

Then we need to copy the key into the omni folder for use:


```bash
gpg --export-secret-key --armor patrick.woodlock@rockfieldit.com > omni.asc
cp omni.asc omni/omni.asc
```




---


# Generate your UUID Key


> We need this key to assign the container ID instance.
> If we do not complete this the omni instance will change the ID on server restarts etc..




```bash
export OMNI_ACCOUNT_UUID=$(uuidgen)
echo $OMNI_ACCOUNT_UUID
```



Following that we need to insert the result into our docker compose file names compose.yaml

This is our UUID from my current instance on the server:   64c6cf9f-3b8a-4fc8-9209-af60c606e8cc


> [!NOTE]
>       - --account-id=64c6cf9f-3b8a-4fc8-9209-af60c606e8cc






Here is our Docker Compose file which should be transferred in to the omni folder and then run the command:


```bash
# I left the -d at the end of the below command out because you need to watch the logs # for anything wrong!!! at the first start,  If you don't like it, **TUFF!** 


docker compose up
```


Please fill in the required fields after you copy and paste it into new compose.yaml file which should be located in the omni folder.


```C
version: '3.9'


services:

  omni:

    image: ghcr.io/siderolabs/omni:latest

    restart: unless-stopped

    network_mode: host

    cap_add:

      - NET_ADMIN

    volumes:

      - ./etcd:/_out/etcd

      - /var/run/docker.sock:/var/run/docker.sock

      - ./chain.crt:/tls.crt

      - ./tls.key:/tls.key

      - ./omni.asc:/omni.asc

    command:

      - --account-id=64c6cf9f-3b8a-4fc8-9209-af60c606e8cc

      - --name=TALOS-RF

      - --cert=/tls.crt

      - --key=/tls.key

      - --siderolink-api-cert=/tls.crt

      - --siderolink-api-key=/tls.key

      - --private-key-source=file:///omni.asc

      - --event-sink-port=8091

      - --bind-addr=0.0.0.0:443

      - --siderolink-api-bind-addr=0.0.0.0:8090

      - --k8s-proxy-bind-addr=0.0.0.0:8100

      - --advertised-api-url=https://YOUR DOMAIN NAME

      - --siderolink-api-advertised-url=https://YOUR DOMAIN NAME:8090

      - --siderolink-wireguard-advertised-addr=YOUR SERVER IP:50180

      - --advertised-kubernetes-proxy-url=https://YOUR DOMAIN NAME:8100

      - --auth-saml-enabled=true

      - --initial-users=<YOUR REGISTERED EMAIL ADDRESS FOR ALL OF THIS PROCESS>

      - --auth-saml-url=<INSERT YOUR SAML FEDERATION LINK IN HERE>
```

