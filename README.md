# Installation-Odoo15-Ubuntu22.04
Ce dépôt contient les fichiers de configuration et les instructions pour installer et configurer Odoo 15 sur Ubuntu 22.04.

## Prérequis
- Ubuntu 22.04
- Accès root ou utilisateur avec privilèges sudo

## Étapes d'installation
1. Mettre à jour le système :
   ```bash
   sudo apt-get update
   sudo apt upgrade -y

##Installer les dépendances :
   ```bash
sudo apt install -y git python3-pip python3-dev python3-venv libxml2-dev libxslt1-dev zlib1g-dev libsasl2-dev libldap2-dev build-essential libssl-dev libffi-dev libmysqlclient-dev libjpeg-dev libpq-dev libjpeg8-dev liblcms2-dev libblas-dev libatlas-base-dev

##Installer PostgreSQL :
   ```bash
sudo apt install -y postgresql postgresql-contrib

##Créer un utilisateur PostgreSQL :
   ```bash
sudo -u postgres createuser --createdb --username postgres --no-createrole --no-superuser --pwprompt calebasse

##Créer un utilisateur système pour Odoo :
   ```bash
sudo adduser --system --home=/opt/calebasse --group calebasse

##Installer Odoo 15 :
   ```bash
sudo git clone https://www.github.com/odoo/odoo --depth 1 --branch 15.0 /opt/calebasse/odoo
sudo python3 -m venv /opt/calebasse/venv
sudo /opt/calebasse/venv/bin/pip3 install wheel
sudo /opt/calebasse/venv/bin/pip3 install -r /opt/calebasse/odoo/requirements.txt

##Configurer Odoo : Créez le fichier /opt/calebasse/odoo.conf avec le contenu suivant :
[options]
admin_passwd = public
db_host = False
db_port = False
db_user = calebasse
db_password = public
addons_path = /opt/calebasse/odoo/addons

##Créer un service systemd pour Odoo : Créez le fichier /etc/systemd/system/calebasse.service avec le contenu suivant :
[Unit]
Description=calebasse
After=postgresql.service

[Service]
Type=simple
User=calebasse
Group=calebasse
ExecStart=/opt/calebasse/venv/bin/python3 /opt/calebasse/odoo/odoo-bin -c /opt/calebasse/odoo.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target

##Configurer Nginx comme reverse proxy : Créez le fichier /etc/nginx/sites-available/calebasse avec le contenu suivant :
server {
    listen 80;
    server_name ip_serveur;

    location / {
        proxy_pass http://127.0.0.1:8069;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /longpolling {
        proxy_pass http://127.0.0.1:8072;
    }

    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering on;
        expires 864000;
        proxy_pass http://127.0.0.1:8069;
    }
}

##Redémarrer les services :
sudo systemctl daemon-reload
sudo systemctl start calebasse
sudo systemctl enable calebasse
sudo systemctl restart nginx

##Accès à Odoo: Ouvrez votre navigateur et accédez à :
http://ip_serveur


Auteur

Vebama S Souleymane


