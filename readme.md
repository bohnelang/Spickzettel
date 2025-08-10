
# IP
## Externe IP Adresse
* curl -4 ifconfig.co
* curl -4 ifconfig.co/ip

## Geodaten:
### Eigene 
* curl -4 ifconfig.co/country
* Alles: curl -4 ifconfig.co/json
### Fremde
* COUNTRY=$(curl -s https://reallyfreegeoip.org/csv/$IP |  grep "CountryName" | sed s/"<CountryName>"//g  | sed s/"<\/CountryName>"//g | awk '{$1=$1};1' )




# Linux 
## Grub
* GRUB_CMDLINE_LINUX="text nomodeset  ipv6.disable=1"
+ update-grub

## SSHD
### Geo-Blocking
```
/etc/hosts.allow
sshd : LOCAL
sshd : 127.
sshd : 192.168.
sshd : .de
sshd : aclexec /etc/host_allow_geoip.sh %a
```

/etc/host_allow_geoip.sh
```
#!/bin/bash




if [ $# -ne 1 ]; then
    echo "Usage: $(basename $0) <ip_address>" 1>&2
    exit 0 # return true in case of config issue
fi

IP=$1

COUNTRY=$(curl -s -4 ifconfig.co/country)

# Handle errors in geoip lookup
if [[ $COUNTRY == Error* || -z $COUNTRY ]]; then
    RESPONSE="DENY"
else
    if [[ " Germany " =~ " $COUNTRY " ]]; then
        RESPONSE="ALLOW"
    else
        RESPONSE="DENY"
    fi
fi

# Log and exit appropriately
logger -p auth.log "sshd $RESPONSE connection from $IP ($COUNTRY)"
if [ "$RESPONSE" = "ALLOW" ]; then
    exit 0
else
    exit 1
fi

```
