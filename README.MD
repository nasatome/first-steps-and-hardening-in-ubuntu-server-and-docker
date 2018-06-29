# First Steps in Ubuntu Server 16.04 / Hardening and Config With Docker

#### Init

    dpkg-reconfigure tzdata
    dpkg-reconfigure locales
    apt update
    apt upgrade
    apt install -y git vim ufw wget curl

#### Install BashIt (only if you do not use zsh or some similar)

    git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && ~/.bash_it/install.sh \
    && mkdir -p ~/.bash_it/plugins/enabled/ \
    && ln -s ~/.bash_it/plugins/available/history.plugin.bash ~/.bash_it/plugins/enabled/250---history.plugin.bash \
    && sed -i 's/bobby/sexy/' ~/.bashrc
    
> You can change the last line "sexy" by any of the available themes, exit the console and re-enter to apply https://github.com/Bash-it/bash-it/tree/master/themes
    
#### Unattended Upgrades
> (Change "YOUREMAIL@TOCHANGE.COM")

> the second command it's an interactive dialog (respond YES and OK) which will create /etc/apt/apt.conf.d/20auto-upgrades 
    
    apt install update-notifier-common && \
    dpkg-reconfigure --priority=low unattended-upgrades && \
    sed -i 's#//Unattended-Upgrade::Mail "root";#Unattended-Upgrade::Mail "YOUREMAIL@TOCHANGE.COM";#' /etc/apt/apt.conf.d/50unattended-upgrades &&\
    sed -i 's#//Unattended-Upgrade::MailOnlyOnError#Unattended-Upgrade::MailOnlyOnError#' /etc/apt/apt.conf.d/50unattended-upgrades &&\
    sed -i 's#//Unattended-Upgrade::Automatic-Reboot "false";#Unattended-Upgrade::Automatic-Reboot "true";#' /etc/apt/apt.conf.d/50unattended-upgrades &&\
    sed -i 's#//Unattended-Upgrade::Automatic-Reboot-Time "02:00";#Unattended-Upgrade::Automatic-Reboot-Time "04:00";#' /etc/apt/apt.conf.d/50unattended-upgrades

#### Hardening SSH
> (Change "YOURUSERNAME")

    ssh-keygen -A
    --
    adduser YOURUSERNAME \
    && usermod -aG sudo YOURUSERNAME

> In Your Local Machine:

    ssh-keygen -t ed25519
    ssh-add ~/.ssh/id_ed25519
    cat ~/.ssh/id_ed25519.pub

> In Remote Machine Paste Output in:
> (Start Session as a New User Created)

    YOURUSERNAME $: mkdir ~/.ssh
    YOURUSERNAME $: chmod 700 ~/.ssh
    YOURUSERNAME $: vim ~/.ssh/authorized_keys
    YOURUSERNAME $: chmod 600 ~/.ssh/authorized_keys
    
    Example to paste: ssh-ed25519 AAAAC3Nzavrg545t64t34rferfdrferodwEv1MMI2+Nh9QxpgrNxb2Is1Cc6 YOURUSERNAME

> Exit console and reconect....

> Test root login:

    YOURUSERNAME $: sudo -E su root

> Now exec this command in root mode (after executing -> sudo -E su root):

    cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak \
    && sed -i 's/Port 22/Port 4449/' /etc/ssh/sshd_config \
    && sed -i 's/X11Forwarding yes/X11Forwarding no/' /etc/ssh/sshd_config \
    && sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config \
    && sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config \
    && sed -zi 's/DebianBanner yes\|$/DebianBanner no\n/' /etc/ssh/sshd_config \
    && sed -zi 's/ClientAliveInterval 60\|$/ClientAliveInterval 60\n/' /etc/ssh/sshd_config \
    && sed -zi 's/ClientAliveCountMax 600\|$/ClientAliveCountMax 600\n/' /etc/ssh/sshd_config \
    && sed -zi 's/AllowAgentForwarding no\|$/AllowAgentForwarding yes/' /etc/ssh/sshd_config

> And now reload ssh service

    systemctl reload sshd

 > To avoid disconnections *In local machine*
    
    echo "Host *" >> ~/.ssh/config
    echo "ServerAliveInterval 60" >> ~/.ssh/config
    echo "ServerAliveCountMax 600" >> ~/.ssh/config
    echo "ForwardAgent yes" >> ~/.ssh/config

> And test:

    cat ~/.ssh/config
    
> You should see this exit

    <<-----------------------
    <<Host *
    <<ServerAliveInterval 60
    <<ServerAliveCountMax 600
    <<ForwardAgent yes
    <<-----------------------



> In remote host (Server): exec this commands

    sudo -E su root
    echo "$SSH_AUTH_SOCK"
    ssh-add -L
    ssh -T git@github.com

> Everyone should give you positive outcomes

```Hi YOURGITUSERNAME! You've successfully authenticated, but GitHub does not provide shell access.```

> Remember that you must add your SSH key to your GITHUB account

   `https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/`

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
    
> Reload With

    sysctl -p
    
> Now

    sed -i 's/order hosts,bind/order bind,hosts/' /etc/host.conf \
    && sed -i 's/multi on/nospoof on/' /etc/host.conf
    
#### Fail2Ban and PSAD

    apt install -y fail2ban psad
    
> Now exec (Change MAIL@YOURMAIL.COM and root@YOURSERVERDOMAIN.COM)

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

> Now restart
    
    systemctl restart fail2ban

> For psad ( Change: EMAIL@YOUREMAILTOCHANGE.COM and SERVER_NAME_TO_CHANGE)
    
    sed -i 's/EMAIL_ADDRESSES             root@localhost;/EMAIL_ADDRESSES             EMAIL@YOUREMAILTOCHANGE.COM;/' /etc/psad/psad.conf \
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
    

> Now Update
    
    psad --sig-update 
    
> Check

    service rsyslog restart
    psad --fw-analyze
    psad -H
    psad -R   
    
### Protect/Block TOR IPs 

    ´https://github.com/nasatome/block_tor_net_by_iptables´

### Install Docker

> Review Script Docker:

´https://get.docker.com/´

> And Exec

    curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
> (for ubuntu 18.04 in not production mode:  today there are still no stable packages 2018-05-06)   
    curl -fsSL test.docker.com -o get-docker.sh && sh get-docker.sh 
    
> Can verify
    
    docker info
    docker version
    
> Now test Docker

    docker run hello-world

#### Install Docker Compose 

> Review last stable release:
> https://github.com/docker/compose/releases
> and change the number version in the command:

    COMPOSE_VERSION=`git ls-remote https://github.com/docker/compose | grep refs/tags | grep -oP "[0-9]+\.[0-9][0-9]+\.[0-9]+$" | tail -n 1`
    curl -L https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose && docker-compose --version
    
    
### Check for rootkits

    apt install rkhunter chkrootkit
    
> test

    chkrootkit
    
> test 2

    rkhunter --update
    rkhunter --propupd
    rkhunter --check
    
    
    
    
> todo:

* add to cron: check rootkits with mail
* Tiger and Tripwire
* https://www.digitalocean.com/community/questions/best-practices-for-hardening-new-sever-in-2017
