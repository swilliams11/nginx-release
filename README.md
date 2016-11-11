# BOSH-deployed nginx Server for Edgemicro

This BOSH release deploys nginx server.  This repo is explicitly for the [apigee-cloudfoundry-edgemicro](https://github.com/swilliams11/apigee-cloudfoundry-edgemicro).
It already has a self-signed public certificate and private key included so you don't have to create it.

### Prerequisites
1. Install [Bosh Lite](https://github.com/cloudfoundry/bosh-lite)
2. Install [Bosh CLI](https://bosh.io/docs/bosh-cli.html)
3. Set your bosh target

```
bosh target 192.168.50.4 lite
```
username/password should be admin/admin

4. Add the following routes.
`10.244.0.0` is for Bosh/containers
`10.244.5.0/24` is for Nginx.

```
sudo route add -net 10.244.0.0/19 192.168.50.4
sudo route add -net 10.244.5.0/24 192.168.50.4

```

Linux users
```
sudo ip route add 10.244.0.0/19 via 192.168.50.4
sudo ip route add 10.244.5.0/24 via 192.168.50.4
```

### 1. Clone this Repository

```
git clone git@github.com:swilliams11/nginx-release.git
```

### 2. Update the Director UUID
* Execute the following command to get the Bosh Director uuid
```
bosh status
```

Response
```
Director
  Name       Bosh Lite Director
  URL        https://hostname:25555
  Version    1.3262.3.0 (00000000)
  User       admin
  UUID       fb70f16a-4d6f-41f8-bc2e-0181a03dff18
  CPI        warden_cpi
  dns        disabled
```


* Open the `nginx_manifest.yml` file and change the `director_uuid` to the director uuid of your Bosh Director shown above.

### 3. Upload cloud-config
The manifest.yml file was updated to V2 so that it is cloud provider agnostic.  
[Manifest.yml V2](http://bosh.io/docs/manifest-v2.html)


Execute the following commands.

```
cd nginx-release

bosh update cloud-config cloud.yml
```

View cloud config.
```
bosh cloud-config
```

### 4. Deploy the Release
Make sure to execute these commands from the `nginx-release` directory.

```bash
bosh create release --force

bosh upload release

bosh deployment nginx_manifest.yml

bosh deploy
```

Once Nginx is deployed you can open a browser and enter the following IP.
`http://10.244.5.2`

You should be automatically redirected to `https`. Now that you have confirmed that Nginx is running, you can proceed to creating the [apigee-cloudfoundry-edgemicro release](https://github.com/swilliams11/apigee-cloudfoundry-edgemicro).

### 5. Optionally Configure the Domain Name in /etc/hosts
You can modify your `/etc/hosts` file to include the following entry.  

```
10.244.5.2      springhello.io
```

Then you can use the domain name in the browser `https://springhello.io` and when sending the curl commands to the Microgateway.  


## Notes
The original documentation for this is in [README2.md](README2.md). See this file if you want to know who `nginx.conf` is configured.   


#### 1. nginx Job Properties
* HTTPS is configured with a self-signed certificate. The certificate and key is located in the `mycerts` directory.  

* nginx is configured to have a static IP address of `10.244.5.2`
* nginx is confirgured to have an upstream server (microgateway) with the IP `10.244.0.2:8000`
  View the `nginx_manifest.yml` file to see the nginx configuration.
