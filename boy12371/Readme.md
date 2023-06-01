### /etc/init.d/cloudflared
```
##-----------------Add cloudflared-------------------------
curl -O -sL -m 30 --retry 20 https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64 -o /usr/bin/cloudflared
chmod +x /usr/bin/cloudflared
touch /etc/init.d/cloudflared
chmod +x /etc/init.d/cloudflared
cat << EOF >> /etc/init.d/cloudflared
#!/bin/sh /etc/rc.common

USE_PROCD=1
START=95
STOP=01

cfd_init="/etc/init.d/cloudflared"
cfd_token="<yourtoken>"

boot()
{
    ubus -t 30 wait_for network.interface network.loopback 2>/dev/null
	rc_procd start_service
}

start_service()
{
    if [ $("${cfd_init}" enabled; printf "%u" ${?}) -eq 0 ]
	then
        procd_open_instance
		procd_set_param command /usr/bin/cloudflared --no-autoupdate tunnel run --token ${cfd_token}
		procd_set_param stdout 1
		procd_set_param stderr 1
		procd_set_param respawn ${respawn_threshold:-3600} ${respawn_timeout:-5} ${respawn_retry:-5}
		procd_close_instance
	fi
}

stop_service()
{
    pidof cloudflared && kill -SIGINT `pidof cloudflared`
}
EOF
##---------------------------------------------------------
```
