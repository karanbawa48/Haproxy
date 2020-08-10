## VagrantFile config.
```Vagrant.configure("2") do |config|
  config.vm.define "test" do |test|
  test.vm.box = "bento/ubuntu-18.04"
  test.vm.network "private_network", ip: "50.1.1.7"
  test.vm.network :forwarded_port, guest: 10, host: 1010
  end
  config.vm.define "test1" do |test1|  
  test1.vm.box =  "bento/ubuntu-18.04"
test1.vm.network "private_network", ip: "50.1.1.8"  
test1.vm.network :forwarded_port, guest: 40, host: 3306
config.vm.provider "virtualbox" do |test1|
  test1.memory = 500
  test1.cpus = 2  
end
end
end
```
### Install apache server on both boxes.

    sudo apt-get update
    sudo apt-get install -y apache2
    
Change index.html on both 

    vim /var/www/html/index.html
    

### Install HAProxy
`sudo add-apt-repository ppa:vbernat/haproxy-1.8`
`sudo apt-get install haproxy`


### Configure HAProxy Load Balancing

  `  sudo vim /etc/haproxy/haproxy.cfg`
     
   *you need to change listening port of apache for test (server1)  by :*
   
   `sudo vim /etc/apache2/ports.conf ` 
   
  **change port 80 to 100 . because in fut here configuration. I uaed this.**  

## Final HAProxy Configuration File

```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
	stats socket /run/haproxy/admin.sock mode 660 level admin
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

	# Default SSL material locations
	ca-base /etc/ssl/certs
	crt-base /etc/ssl/private

	# Default ciphers to use on SSL-enabled listening sockets.
	# For more information, see ciphers(1SSL). This list is from:
	#  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
	ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256::RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
	ssl-default-bind-options no-sslv3

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
	errorfile 400 /etc/haproxy/errors/400.http
	errorfile 403 /etc/haproxy/errors/403.http
	errorfile 408 /etc/haproxy/errors/408.http
	errorfile 500 /etc/haproxy/errors/500.http
	errorfile 502 /etc/haproxy/errors/502.http
	errorfile 503 /etc/haproxy/errors/503.http
	errorfile 504 /etc/haproxy/errors/504.http

frontend Local_Server
    bind 50.1.1.7:80
    mode http
    default_backend My_Web_Servers

backend My_Web_Servers
    mode http
    balance roundrobin
    option forwardfor
    http-request set-header X-Forwarded-Port %[dst_port]
    http-request add-header X-Forwarded-Proto https if { ssl_fc }
    option httpchk HEAD / HTTP/1.1rnHost:localhost
    server  1:  50.1.1.7:100
    server 2:  50.1.1.8:80 
   ```
### Restart HAProxy

`haproxy -c -f /etc/haproxy/haproxy.cfg `
If above command returned output as configuration file is valid then restart HAProxy service

`sudo service haproxy restart`

# All Done !
Now open your haproxy ip on web browser. in first attempt you'll get output from server1 and on next attempt output from sever2 will shown.
`50.1.1.7:80`
