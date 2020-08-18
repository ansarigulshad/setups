# Setup SSL enabled HAProxy Loadbalancer on CentOS/RHEL-7


### Step 1 : Install Required Packages
```
yum update -y
yum install haproxy -y
```

### Step 2 : Enable auto-start on reboot
```
systemctl enable haproxy
```

### Step 3 : Backup default config
```
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.orig
```

### Step 4 : Create self signed SSL Certificates
```
openssl genrsa -out /etc/haproxy/haproxy.key 2048
openssl req -new -key /etc/haproxy/haproxy.key -out /etc/haproxy/haproxy.csr -subj "/C=US/ST=North Carolina/L=Raleigh/O=Cloudera/OU=Support/CN=$(hostname -f)"
openssl x509 -req -days 365 -in /etc/haproxy/haproxy.csr -signkey /etc/haproxy/haproxy.key -out /etc/haproxy/haproxy.crt
cat /etc/haproxy/haproxy.key /etc/haproxy/haproxy.crt > /etc/haproxy/haproxy.pem
```

### Step 5 : Create new `haproxy.cfg` file.
```
cat > haproxy.cfg <<EOFILE
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log         127.0.0.1 local2     #Log configuration
 
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000                
    user        haproxy             #Haproxy running under user and group "haproxy"
    group       haproxy
    daemon
 
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
 
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
 
#---------------------------------------------------------------------
#HAProxy Monitoring Config
#---------------------------------------------------------------------
listen haproxy3-monitoring *:8090                #Haproxy Monitoring run on port 8090
    mode http
    option forwardfor
    option httpclose
    stats enable
    stats show-legends
    stats refresh 5s
    stats uri /stats                             #URL for HAProxy monitoring
    stats realm Haproxy\ Statistics
    stats auth gulshad:gulshad            #User and Password for login to the monitoring dashboard
    stats admin if TRUE
    default_backend app-main                    #This is optionally for monitoring backend
 
#---------------------------------------------------------------------
# FrontEnd Configuration
#---------------------------------------------------------------------
frontend main
    bind *:80
    bind *:443 ssl crt /etc/haproxy/haproxy.pem
    option http-server-close
    option forwardfor
    default_backend app-main
 
#---------------------------------------------------------------------
# BackEnd roundrobin as balance algorithm
#---------------------------------------------------------------------
backend app-main
    mode http
    balance roundrobin
    cookie SERVERID insert indirect nocache
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    server knox1 edge-knox1.cloudera.com:8443 ssl verify none cookie knox1
    server knox2 edge-knox2.cloudera.com:8443 ssl verify none cookie knox2
EOFILE
```

_Kindly update hostname & port number of your service according to your setup_

### Step 6 : Start/Restart HAProxy service
```
systemctl restart haproxy
```

### Step 7 : Validate
```
curl https://`hostname -f`:443
curl -k https://`hostname -f`:443
openssl s_client -connect `hostname -f`:443 -showcerts
```
