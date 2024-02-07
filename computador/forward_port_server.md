# Configurar os roteadores para apontar o IP Fixo para o servidor
https://portforward.com/help/doublerouterportforwarding.htm

External IP: buscalucro.giize.com
Internal IP R1:192.168.0.1

External IP R2: 192.168.0.45   
Internal IP R2: 192.168.5.1

Computer IP: 192.168.5.213

## Configurar ip estático na Nova
   Tenda app > Configurações > Configurações da Internet > 
      Endereço IP: 192.168.0.45
      Máscara de Sub-rede: 255.255.255.0
      Gateway padrão: 192.168.0.1
      Servidor DNS principal: 8.8.8.8
   > Salvar
   
## Abrir as configurações do modem ARRIS da Net
1) Logar na rede net virtua
2) acessar o site 192.168.0.1
3) Colocar o usuário e senha
   3.1) Usuário: NET_DF406F
   3.2) Senha: BC644BDF406F
   3.3) Clicar em Apply
4) Firewall > Virtual Servers > Port Forwarding:
   Description: www
   Inbound Port: 1-65535
   Type: Both
   Private Ip Address: 192.168.0.45       #IP do roteador Nova Mesh
   Local Port: 1-65535 

## Configure to remote access
 1) First, log in to the MariaDB shell with the following command:
     $ mysql -u root -p

 2) Create a database and user with the following command:
     MariaDB [(none)]> CREATE DATABASE prod;
     MariaDB [(none)]> CREATE USER  'tafranco'@'localhost' IDENTIFIED BY 'password';

 3) Grant permissions to the remote system with IP address
     MariaDB [(none)]> GRANT ALL ON prod.* to 'tafranco'@'192.168.5.213' IDENTIFIED BY 'ocB15soA' WITH GRANT OPTION;  

 4) Next, flush the privileges and exit
     MariaDB [(none)]> FLUSH PRIVILEGES;
     MariaDB [(none)]> EXIT;

## Configuração vscode
Host Notebook
  HostName buscalucro.giize.com
  User root
  #ServerAliveInterval 60
  #ServerAliveCountMax 30
  ForwardAgent yes
  #ForwardX11 yes
  #ForwardX11Trusted yes
  Port 22
  IdentityFile C:\\Users\\desig.thiago\\.ssh\\id_rsa.

## Configuração DBEAVER
SSH:
   Usar SSH Tunnel
   Host/Ip: buscalucro.giize.com   Port:22
   User Name: root
   Password: ocB15soA

Principal:
   Server Host: 192.168.5.213 Port:3306
   Database: prod
   Nome de Usuário: tafranco
   Senha: ocB15soA
## Configuração Gunicorn
$ nano /etc/systemd/system/gunicorn.service

[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/var/www/html/django/buscalucro
ExecStart=/usr/local/bin/gunicorn  --worker-connections=1000 --workers=3 --bind 192.168.5.213:8000 buscalucro.wsgi

[Install]
WantedBy=multi-user.target

$ systemctl daemon-reload
$ systemctl restart gunicorn

## Configuração nginx
$ nano /etc/nginx/sites-available/buscalucro

server {
    listen 80;
    server_name 192.168.5.213;           

    location = /favicon.ico { access_log off; log_not_found off; }
}

$ systemctl restart nginx

   

