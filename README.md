# ELK
https://medium.com/@lopchannabeen138/deploying-elk-inside-docker-container-docker-compose-4a88682c7643
# Creation docker nginx + filebeat
https://medium.com/@knoldus/how-to-run-filebeat-in-a-docker-container-c5293942e070

# ELK:
'cd ELK && docker-compose up -d'

# NGINX_Filebeat
```bash

# Afficher ses ip lisible facilement : ip a | awk '/inet / {split($2, a, "/"); print a[1]}'
# Definition de l'IP d'ELK pour NGINX_Filebeat
IP_ELK="172.26.35.23"

# Se placer dans le bon repertoire :
cd NGINX_Filebeat/

# Creation d'un rep pour stocker les logs NGINX
[ ! -d nginx_logs ] && mkdir nginx_logs

# Lancer conteneur nginx 
docker run -d -p 8080:80 -v $PWD/nginx_logs:/var/log/nginx --name nginx nginx

# Lancer le conteneur fileabeat en initialisant et configure la connexion vers elastic et kibana
docker run \
docker.elastic.co/beats/filebeat:7.9.2 \
setup -E setup.kibana.host=${IP_ELK}:5601 \
-E output.elasticsearch.hosts=["${IP_ELK}:9200"]

# Effectue le remplacement de IP dans filebeat_docker.yml
sed "s/\${IP}/${IP_ELK}/g" template_filebeat_docker.yml > filebeat_docker.yml
# Donner les droits pour que user root puisse les lire dans le conteneur
chown root:root filebeat_docker.yml
chmod go-w filebeat_docker.yml

# Lancement du conteneur et configuration 
docker run -d \
  --name=filebeat \
  --user=root \
  --volume="$(pwd)/filebeat_docker.yml:/usr/share/filebeat/filebeat.yml" \
  --volume="$(pwd)/nginx.yml:/usr/share/filebeat/modules.d/nginx.yml" \
  --volume="/var/lib/docker/containers:/var/lib/docker/containers:ro" \
  --volume="/var/run/docker.sock:/var/run/docker.sock:ro" \
  --volume="$(pwd)/nginx_logs:/var/log/nginx:ro" \
  docker.elastic.co/beats/filebeat:7.9.2 filebeat -e --strict.perms=false


#Connexion au conteneur :
docker exec -ti filebeat /bin/bash 
#Configuration du dashboard
./filebeat setup --dashboards 
#Vérification de la présence des fichiers de logs dans le conteneur
ls -la /var/log/nginx/
#Activation du module Nginx 
ls -la /usr/share/filebeat/modules.d | grep nginx
./filebeat modules list | grep nginx
./filebeat modules enable nginx
#Test de configuration
./filebeat test config -e
#Démarrage de Filebeat
./filebeat setup -e
```

