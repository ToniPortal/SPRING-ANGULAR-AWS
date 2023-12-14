# Guide de Déploiement sur AWS (MERN Stack + Angular)

## Section 1: Déployer une application Java Spring Boot sur une instance EC2 AWS Amazon Linux 2 [Partie Backend]

1. **Créer votre instance :**

   - Recherchez Ec2 dans votre compte AWS, cliquez sur “Lancer des instances” sur la page principale pour commencer à créer votre nouvelle instance.

   - Choisissez l'AMI Amazon Linux 2 par défaut et sélectionnez le type t2.micro qui fait partie de la Free tier.

   - Le groupe de sécurité décide qui peut accéder à votre instance. Par défaut, le SSH 22 nous aidera à nous connecter à l'instance depuis notre système.
   
   - Ajoutez un nouveau groupe de sécurité pour accéder au port 8080.

      ```plaintext
      Type: Custom TCP Rule
      Protocol: TCP
      Port Range: 8080
      Source: Anywhere
      ```

   - Cliquez sur "Review and Launch" et assurez-vous de télécharger votre Key Pair.

2. **Connectez-vous à votre Instance :**
   - Utilisez le fichier pem pour vous connecter à votre instance.

      ```bash
      ssh -i C:\Users\IhebS\Desktop\AWS\keyPair.pem ec2-user@41.212.10.33
      ```

3. **Configuration de l'instance EC2 :**

   - Java est installé par défaut : `java -version`
   - Installer Java 8 : `sudo yum install java-1.8.0`
   - Créer un utilisateur et un groupe pour Tomcat.

      ```bash
      sudo groupadd --system tomcat 
      sudo useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat
      ```

   - Installer et configurer Apache Tomcat 9.

      ```bash
      export VER="9.0.41"
      wget https://archive.apache.org/dist/tomcat/tomcat-9/v${VER}/bin/apache-tomcat-${VER}.tar.gz
      sudo tar xvf apache-tomcat-${VER}.tar.gz -C /usr/share/
      sudo ln -s /usr/share/apache-tomcat-$VER/ /usr/share/tomcat
      sudo chown -R tomcat:tomcat /usr/share/tomcat
      sudo chown -R tomcat:tomcat /usr/share/apache-tomcat-$VER/
      ```

   - Créer le service SystemD Tomcat.

      ```bash
      sudo tee /etc/systemd/system/tomcat.service<<EOF
      [Unit]
      Description=Tomcat Server
      After=syslog.target network.target

      [Service]
      Type=forking
      User=tomcat
      Group=tomcat

      Environment=JAVA_HOME=/usr/lib/jvm/jre
      Environment='JAVA_OPTS=-Djava.awt.headless=true'
      Environment=CATALINA_HOME=/usr/share/tomcat
      Environment=CATALINA_BASE=/usr/share/tomcat
      Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
      Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
      ExecStart=/usr/share/tomcat/bin/catalina.sh start
      ExecStop=/usr/share/tomcat/bin/catalina.sh stop

      [Install]
      WantedBy=multi-user.target
      EOF

      sudo systemctl daemon-reload
      sudo systemctl start tomcat
      sudo systemctl enable tomcat
      ```

   - Pour accéder à votre Tomcat, allez à l'interface AWS et naviguez vers votre instance EC2 pour obtenir votre DNS IPV4 public.

4. **Utilisateur administrateur Tomcat pour télécharger le fichier WAR :**
   - Ouvrez le fichier de configuration Tomcat.

      ```bash
      sudo vim /usr/share/tomcat/conf/tomcat-users.xml
      ```

   - Mettez à jour les utilisateurs Tomcat.

      ```xml
      <role rolename="admin-gui"/>
      <role rolename="manager-gui"/>
      <user username="admin" password="TomcatP@sSw0rD" fullName="Administrator" roles="admin-gui,manager-gui"/>
      ```

   - Mettez à jour le gestionnaire Webapps.

      ```bash
      sudo su
      vi /usr/share/tomcat/webapps/manager/META-INF/context.xml
      ```

   - Commentez une ligne pour éviter une erreur.

      ```xml
      <!-- <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
      ```

   - Vous pouvez accéder à la page du gestionnaire en utilisant le lien "manager webapp" sur l'interface Tomcat.

5. **Télécharger le fichier WAR :**
   - Faites défiler jusqu'à la section 'WAR file to deploy' et téléchargez votre fichier .war.

