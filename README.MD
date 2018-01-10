# First Steps in Ubuntu 16.04 / Hardening and Config With Docker 

#### Init

    dpkg-reconfigure tzdata
    dpkg-reconfigure locales
    apt update
    apt upgrade
    apt install -y git vim ufw wget curl

#### Install BashIt

    git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && ~/.bash_it/install.sh \
    && mkdir -p ~/.bash_it/plugins/enabled/ \
    && ln -s ~/.bash_it/plugins/available/history.plugin.bash ~/.bash_it/plugins/enabled/250---history.plugin.bash \
    && sed -i 's/bobby/sexy/' ~/.bashrc

#### Hardening SSH

    ssh-keygen -A

    adduser nasatome \
    && usermod -aG sudo nasatome

En Maquina Local:

    ssh-keygen -t ed25519
    ssh-add ~/.ssh/id_ed25519
    cat ~/.ssh/id_ed25519.pub

En Maquina Remota Pegar Salida en:
(Iniciar Sesion Como Nuevo Usuario Creado)

    nasatome $: mkdir ~/.ssh
    nasatome $: chmod 700 ~/.ssh
    nasatome $: vim ~/.ssh/authorized_keys
    nasatome $: chmod 600 ~/.ssh/authorized_keys
    example to paste: ssh-ed25519 AAAAC3Nzavrg545t64t34rferfdrferodwEv1MMI2+Nh9QxpgrNxb2Is1Cc6 test1

exit console and test....

test root login:

    nasatome $: sudo -E su root

exit and test

    cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak \
    && sed -i 's/Port 22/Port 4449/' /etc/ssh/sshd_config \
    && sed -i 's/X11Forwarding yes/X11Forwarding no/' /etc/ssh/sshd_config \
    && sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config \
    && sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config \
    && sed -zi 's/DebianBanner yes\|$/DebianBanner no\n/' /etc/ssh/sshd_config \
    && sed -zi 's/ClientAliveInterval 60\|$/ClientAliveInterval 60\n/' /etc/ssh/sshd_config \
    && sed -zi 's/ClientAliveCountMax 600\|$/ClientAliveCountMax 600\n/' /etc/ssh/sshd_config \
    && sed -zi 's/AllowAgentForwarding no\|$/AllowAgentForwarding yes/' /etc/ssh/sshd_config

and 

    systemctl reload sshd

 in local machine
    
    echo "Host *" >> ~/.ssh/config
    echo "ServerAliveInterval 60" >> ~/.ssh/config
    echo "ServerAliveCountMax 600" >> ~/.ssh/config
    echo "ForwardAgent yes" >> ~/.ssh/config

and test:

    cat ~/.ssh/config
    <<-----------------------
    <<Host *
    <<ServerAliveInterval 60
    <<ServerAliveCountMax 600
    <<ForwardAgent yes
    <<-----------------------



in remote host: test

    sudo -E su root
    echo "$SSH_AUTH_SOCK"
    ssh-add -L
    ssh -T git@github.com
    
#### Firewall UFW And IPTABLES Persistent

    ufw allow 4449 && ufw allow http && ufw allow https && ufw default deny incoming && ufw enable && ufw logging on
      
    apt install iptables-persistent 
    
    iptables -A INPUT -j LOG \
    && iptables -A FORWARD -j LOG \
    && ip6tables -A INPUT -j LOG \
    && ip6tables -A FORWARD -j LOG
    
    netfilter-persistent save

#### Network Hardening

    sed -i 's/#net.ipv4.conf.default.rp_filter=1/net.ipv4.conf.default.rp_filter=1/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.conf.all.rp_filter=1/net.ipv4.conf.all.rp_filter=1/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.conf.all.accept_source_route = 0/net.ipv4.conf.all.accept_source_route = 0/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv6.conf.all.accept_source_route = 0/net.ipv6.conf.all.accept_source_route = 0/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.conf.all.send_redirects = 0/net.ipv4.conf.all.send_redirects = 0/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.tcp_syncookies=1/net.ipv4.tcp_syncookies=1/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.conf.all.log_martians = 1/net.ipv4.conf.all.log_martians = 1/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv4.conf.all.accept_redirects = 0/net.ipv4.conf.all.accept_redirects = 0/' /etc/sysctl.conf \
    && sed -i 's/#net.ipv6.conf.all.accept_redirects = 0/net.ipv6.conf.all.accept_redirects = 0/' /etc/sysctl.conf
    
Reload With

    sysctl -p
    
now

    sed -i 's/order hosts,bind/order bind,hosts/' /etc/host.conf \
    && sed -i 's/multi on/nospoof on/' /etc/host.conf
    
#### Fail2Ban and PSAD

    apt install -y fail2ban psad
    
now 

    
    sed -i 's/destemail = root@localhost/destemail = MAIL@YOURMAIL.COM/' /etc/fail2ban/jail.conf \
    && sed -i 's/sender = root@localhost/sender = root@YOURSERVERDOMAIN.COM/' /etc/fail2ban/jail.conf \
    && sed -i 's/action = %(action_)s/action = %(action_mwl)s/' /etc/fail2ban/jail.conf \
    && echo "[nginx-http-auth]" > /etc/fail2ban/jail.local \
    && echo "enabled  = false" >> /etc/fail2ban/jail.local \
    && echo "filter   = nginx-http-auth" >> /etc/fail2ban/jail.local \
    && echo "port     = http,https" >> /etc/fail2ban/jail.local \
    && echo "logpath  = %(nginx_error_log)s" >> /etc/fail2ban/jail.local \
    && echo "#logpath  = /opt/prj/dockavel/data/log/nginx/error.log" >> /etc/fail2ban/jail.local \
    && echo "" >> /etc/fail2ban/jail.local \
    && echo "[nginx-noscript]" >> /etc/fail2ban/jail.local \
    && echo "enabled  = false" >> /etc/fail2ban/jail.local \
    && echo "port     = http,https" >> /etc/fail2ban/jail.local \
    && echo "filter   = nginx-noscript" >> /etc/fail2ban/jail.local \
    && echo "logpath  = %(nginx_error_log)s" >> /etc/fail2ban/jail.local \
    && echo "#logpath  = /opt/prj/dockavel/data/log/nginx/error.log" >> /etc/fail2ban/jail.local \
    && echo "maxretry = 6" >> /etc/fail2ban/jail.local \
    && echo "" >> /etc/fail2ban/jail.local \
    && echo "[ssh]" >> /etc/fail2ban/jail.local \
    && echo "enabled = true" >> /etc/fail2ban/jail.local \
    && echo "port    = 4449" >> /etc/fail2ban/jail.local \
    && echo "filter  = sshd" >> /etc/fail2ban/jail.local \
    && echo "logpath = /var/log/auth.log" >> /etc/fail2ban/jail.local \
    && echo "maxretry = 4" >> /etc/fail2ban/jail.local \
    && echo "" >> /etc/fail2ban/jail.local \
    && echo "[ssh-ddos]" >> /etc/fail2ban/jail.local \
    && echo "enabled = true" >> /etc/fail2ban/jail.local \
    && echo "port    = 4449" >> /etc/fail2ban/jail.local \
    && echo "filter  = sshd-ddos" >> /etc/fail2ban/jail.local \
    && echo "logpath = /var/log/auth.log" >> /etc/fail2ban/jail.local \
    && echo "" >> /etc/fail2ban/jail.local \
    && echo "[Definition]" > /etc/fail2ban/filter.d/nginx-http-auth.conf \
    && echo 'failregex = ^ \[error\] \d+#\d+: \*\d+ user "\S+":? (password mismatch|was not found in ".*"), client: <HOST>, server: \S+, request: "\S+ \S+ HTTP/\d+\.\d+", host: "\S+"\s*$' >> /etc/fail2ban/filter.d/nginx-http-auth.conf \
    && echo "ignoreregex =" >> /etc/fail2ban/filter.d/nginx-http-auth.conf \
    && echo "[Definition]" > /etc/fail2ban/filter.d/nginx-noscript.conf \
    && echo "failregex = ^<HOST> -.*GET.*(\.asp|\.exe|\.pl|\.cgi|\.scgi)" >> /etc/fail2ban/filter.d/nginx-noscript.conf \
    && echo "ignoreregex =" >> /etc/fail2ban/filter.d/nginx-noscript.conf

restart
    
    systemctl restart fail2ban

psad
    
    sed -i 's/EMAIL_ADDRESSES             root@localhost;/EMAIL_ADDRESSES             EMAIL@YOUREMAIL.COM;/' /etc/psad/psad.conf \
    && sed -i 's/HOSTNAME                    _CHANGEME_;/HOSTNAME                    SERVER_NAME_TO_CHANGE;/' /etc/psad/psad.conf \
    && sed -i 's/ENABLE_AUTO_IDS             N;/ENABLE_AUTO_IDS             Y;/' /etc/psad/psad.conf \
    && sed -i 's/DANGER_LEVEL2               15;/DANGER_LEVEL2               15;/' /etc/psad/psad.conf \
    && sed -i 's/DANGER_LEVEL3               150;/DANGER_LEVEL3               30;/' /etc/psad/psad.conf \
    && sed -i 's/DANGER_LEVEL4               1500;/DANGER_LEVEL4               100;/' /etc/psad/psad.conf \
    && sed -i 's/DANGER_LEVEL5               10000;/DANGER_LEVEL5               300;/' /etc/psad/psad.conf \
    && sed -i 's/EMAIL_ALERT_DANGER_LEVEL    1;/EMAIL_ALERT_DANGER_LEVEL    5;/' /etc/psad/psad.conf \
    && sed -i 's/EMAIL_LIMIT                 0;/EMAIL_LIMIT                 1;/' /etc/psad/psad.conf \
    && sed -i 's/EMAIL_LIMIT_STATUS_MSG      Y;/EMAIL_LIMIT_STATUS_MSG      N;/' /etc/psad/psad.conf \
    && sed -i 's/ENABLE_AUTO_IDS_EMAILS      Y;/ENABLE_AUTO_IDS_EMAILS      N;/' /etc/psad/psad.conf \
    && sed -i 's/AUTO_IDS_DANGER_LEVEL       5;/AUTO_IDS_DANGER_LEVEL       1;/' /etc/psad/psad.conf \
    && sed -i 's#IPT_SYSLOG_FILE             /var/log/messages;#IPT_SYSLOG_FILE             /var/log/syslog;#' /etc/psad/psad.conf
    

update
    
    psad --sig-update 
    
   

check

    service rsyslog restart
    psad --fw-analyze
    psad -H
    psad -R   
    
### Protect/Block TOR IPs 
https://github.com/nasatome/block_tor_net_by_iptables

### Install Docker
Review Script Docker:

> https://get.docker.com/

And Exec

    curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
    
verify
    
    docker info
    docker version
    
test

    docker run hello-world

#### Install Docker Compose 
Review last stable release:
> https://github.com/docker/compose/releases
and change the number version in the command:

    curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose && docker-compose --version
    
    
### Check for rootkits

    apt install rkhunter chkrootkit
    
test

    chkrootkit
    
test 2

    rkhunter --update
    rkhunter --propupd
    rkhunter --check
    
    
    
    
>todo:
* add https://github.com/nasatome/block_tor_net_by_iptables
* add to cron: check rootkits with mail
* Tiger and Tripwire
* https://www.digitalocean.com/community/questions/best-practices-for-hardening-new-sever-in-2017
