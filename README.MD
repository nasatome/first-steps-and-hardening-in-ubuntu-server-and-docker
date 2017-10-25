####Init
    * dpkg-reconfigure tzdata
    * dpkg-reconfigure locales
    * apt update
    * apt upgrade
    * apt install -y git vim ufw wget curl

#### Install BashIt
    git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && ~/.bash_it/install.sh \
    && mkdir -p /root/.bash_it/plugins/enabled/ \
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