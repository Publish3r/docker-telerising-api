# docker-telerising-api
A docker container for running the Telerising API

### Prerequisites
You will need to have `docker` installed on your system and the user you want to run it needs to be in the `docker` group.

> **Note:** The image is a multi-arch build providing variants for amd64, arm32v7 and arm64v8 - the correct variant for your Architecture needs to be tagged eg :amd64 :arm32v7 :arm64v8


## Technical info for Docker
To learn how to manually start the container or about available parameters (you might need for your GUI used) see the following example:

```
docker run \
  -d \
  -e USER_ID="99" \
  -e GROUP_ID="100" \
  -e TIMEZONE="Europe/Berlin" \
  -e UPDATE="yes" \
  -p 5000:5000 \
  -v {TELERISING_STORAGE}:/telerising \
  --name=new-telerising-api \
  --restart unless-stopped \
  --tmpfs /tmp \
  --tmpfs /var/log \
  --net="bridge" \
  takealug/new-telerising-api:tag
```

The available parameters in detail:

| Parameter | Optional | Values/Type | Default | Description |
| ---- | --- | --- | --- | --- |
| `USER_ID` | yes | [integer] | 99 | UID to run telerising as |
| `GROUP_ID` | yes | [integer] | 100 | GID to run telerising as |
| `TIMEZONE` | yes | [string] | Europe/Berlin | Timezone for the container |
| `UPDATE` | yes | yes/no | yes | Updates Telerising Script inside This Container each restart |

Network Settings:

| Parameter | Optional | Values/Type | Default | Description |
| ---- | --- | --- | --- | --- |
| `-p` | yes | [integer] | 5000:5000 | Map Container Listenport to Host Device Listen Port (Bridge Mode)|
| `--net` | yes | bridge/host | bridge | The Network Mode to run The Container in|

> **Note:** If you plan to use VLC / tvheadend ect outside of your Docker network (e.g. 172.17.0.X) to use, you should run the container in --net = "host" .

Frequently used volumes:
 
| Volume | Optional | Description |
| ---- | --- | --- |
| `TELERISING_STORAGE` | no | The directory to persist telerising to |


When passing volumes please replace the name including the surrounding curly brackets with existing absolute paths with correct permissions.

## Support my work
If you like my Work, please [![Paypal Donation Page](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](https://paypal.me/DeBaschdi) - thank you! :-)

## Unraid Template
> **Note:** An Template for Unraid can be found here : https://raw.githubusercontent.com/DeBaschdi/docker.new-telerising-api/master/Templates/Unraid/my-NEW-Telerising-API.xml
> Please safe it to into /boot/config/plugins/dockerMan/templates-user/my-NEW-Telerising-API.xml, after that you can use this Template in Unraids Webui. Docker > Add Container > Select Template and choose NEW-Telerising-API

<img src="https://raw.githubusercontent.com/DeBaschdi/docker.new-telerising-api/master/Templates/Unraid/Screenshot.png" height="325" width="265">

## Run over OpenVPN
Create OpenVPN folder and ovpn file:

```
mkdir /opt/ovpn \
  && cd /opt/ovpn \
  && touch vpn.ovpn \
  && nano vpn.ovpn \
```

Start OpenVPN docker container:

```  
sudo docker run \
  -d \
  -e TZ="Europe/Berlin" \
  -p 5000:5000 \
  -v /opt/ovpn:/vpn \
  --name=vpn \
  --restart=always \
  --cap-add=NET_ADMIN \
  --dns="1.1.1.1" \
  --dns="1.0.0.1" \
  --net="bridge" \
  --privileged \
  --device /dev/net/tun \
  dperson/openvpn-client:latest
```
  
Start Telerising docker container:
  
``` 
sudo docker run \
  -d \
  -e USER_ID="0" \
  -e GROUP_ID="0" \
  -e TIMEZONE=Europe/Berlin \
  -e UPDATE=yes \  
  -v /opt/telerising:/telerising \
  -p 5000:5000 \
  --name=telerising \
  --restart=always \
  --net=container:vpn \
  takealug/new-telerising-api:latest
```

## Run over Proxy Server

Start Telerising docker container over Proxy Server:
  
``` 
sudo docker run \
  -d \
  -e USER_ID="0" \
  -e GROUP_ID="0" \
  -e TIMEZONE=Europe/Berlin \
  -e UPDATE=yes \  
  -e HTTP_PROXY=http://username:password@proxy2.domain.com \
  -e HTTPS_PROXY=http://username:password@proxy2.domain.com \
  -v /opt/telerising:/telerising \
  -p 5000:5000 \
  --name=telerising \
  --restart=unless-stopped \
  --net=bridge \
  takealug/new-telerising-api:latest
```