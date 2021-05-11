Setup
Docker
Installer docker https://docs.docker.com/get-docker/

git clone https://github.com/Kinamori/projet-poei.git
cd projet-poei
mkdir jenkins-data/ nexus-data/ sonarqube/

docker-compose up -d

docker ps devrait afficher CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
e07b354b0302 jenkinsci/blueocean "/sbin/tini -- /usr/…" 3 seconds ago Up 2 seconds 0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 50000/tcp jenkins
a85779621db3 sonarqube "bin/run.sh bin/sona…" 4 seconds ago Up 3 seconds 0.0.0.0:9000->9000/tcp, :::9000->9000/tcp sonarqube
7a6a21b6f89c sonatype/nexus3 "sh -c ${SONATYPE_DI…" 4 seconds ago Up 3 seconds 0.0.0.0:8081->8081/tcp, :::8081->8081/tcp nexus

SI CA FONCTIONNE ?
ifconfig ou ipconfig
en0 ou enp0 ou eth0
Copier son addresse ip
ADRESSE IP UNIQUE POUR LES 2 CONFIGURATIONS \

SI CA FONCTIONNE PAS (virtual machine ?)
copier l'ip du conteneur jenkins
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' jenkins
copier l'ip du conteneur sonarqube
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' sonarqube
copier l'ip du conteneur nexus
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nexus

Jenkins
Connexion
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
Rentrer le mot de passe dans localhost:8080
Installer les plugins suggérés
Créer un administrateur admin admin
Laisser l'URL d'instance par défaut (localhost:8080)
Start using Jenkins
Manage Jenkins -> Manage Plugin -> Available -> SonarQube Scanner -> install without restart
docker restart jenkins
Actualiser la page, se connecter avec le compte admin créer plus tôt

Plugin Sonarqube
Docker jenkins cd /var/jenkins_home
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
unzip sonar-scanner-cli-3.3.0.1492-linux.zip
Manage Jenkins -> Configure Global Tool -> Sonarqube Scanner
Name : Sonarqube scanner
Uncheck install automatically
PATH : /var/jenkins_home/sonar-scanner-3.3.0.1492-linux
Save

Manage Jenkins -> Configure System -> SonarQube servers
Name : Sonarqube
server URL : http://:9000
Server authentication token -> add
Kind : Secret text
Secret : le token copier lors de la configuration de sonarqube
id: token sonarqube
Save

Plugin Git Integration
(Ne fonctionne pas si on n'a pas accès aux urls avec l'adresse ip)
Pour build après chaque commit
Jenkins -> Manage Jenkins -> Manage Plugin -> Github Integration -> install without restart
Restart jenkins

Creation d'un item
Jenkins
New Item -> Pipeline project
Name : projet_pipeline
Pipeline -> definition : Pipeline Script from SCM
SCM : Git
Repository URL : https://github.com/Kinamori/projet-poei.git
Branches to build -> Branch Specifier : */main
Script Path : flask-pytest-example-master/Jenkinsfile
SAVE

Sonarqube
localhost:9000 admin admin par défaut
Changer de mot de passe

Webhook
Administration -> Configuration -> webhook
Creer un nouveau webhook
name : Sonarqube
URL : http://:8080/sonarqube-webhook
Create

Token
Administrator -> my account -> security
jenkins puis generate token
Copier le token créé
Dans mon cas 933f34d817d7ef4b8b87944525c17b02c74d3743

Sonarqube -> create new project
project key : projet
OK
Provide token -> use existing token -> le nom du token créé plus tôt
OK
Other -> Linux

Nexus
docker exec -it nexus cat /nexus-data/admin.password
Copier le mot de passe affiché 7e5325aa-959b-4961-8568-17d9d3285721
Sign in admin le mot de passe récupéré plus haut
new password root
Enable anonymous access
FINISH

Icone paramètre -> Repositories -> create repository -> raw (hosted)
Name : projet_pipeline
CREATE REPOSITORY

Github
Dans le projet github -> settings -> webhooks -> add new webhook
Payload URL : http://:8080/github-webhook
Content type : json
Check Just push event
Check Active
