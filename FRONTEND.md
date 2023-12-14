# Guide de Déploiement sur AWS (MERN Stack + Angular)

## Section 2: Déployer une application Angular sur une instance EC2 AWS [Partie Frontend]

1. **Configurer l'instance EC2 :**

   - Nous suivrons ces étapes pour installer certaines exigences nécessaires pour exécuter notre application Angular sur notre instance EC2.

      ```bash
      sudo apt-get update
      ```

   - Installez git pour extraire votre code depuis votre dépôt distant :

      ```bash
      sudo apt-get install -y git
      ```

   - Installez npm et nodejs :

      ```bash
      sudo apt-get install -y npm
      sudo apt-get install -y nodejs
      ```

   - Nous utiliserons le serveur Nginx

      ```bash
      sudo apt-get install -y nginx
      ```

   - Pour tester notre serveur nginx en cours d'exécution : Accédez à <public_ip>:8080, vous verrez la page NGINX en cours d'exécution.

   - Ensuite, installez Angular CLI, nécessaire pour exécuter les services Angular.

      ```bash
      sudo npm install -g @angular/cli
      ```

2. **Configurer le serveur Nginx :**

   - Allez dans le répertoire des sites disponibles :

      ```bash
      cd /etc/nginx/sites-available
      ```

   - Créez un fichier avec le nom de votre projet :

      ```bash
      sudo vim <votre-nom-de-projet>
      ```

      Mettez à jour le fichier avec le contenu ci-dessous :

      ```nginx
      server {     
          listen 80;      
          listen [::]:80;      
          server_name http://votre-nom-de-projet.com;      
          root /var/www/my-demo-app/dist;   
          server_tokens off;   
          index index.html index.htm;     
     
          location / {         
              # First attempt to server request as file, then         
              # as directory, then fall back to displaying a 404.          
              try_files $uri $uri/ /index.html =404;      
          }
      }
      ```

      Dans le fichier ci-dessus, changez 'votre-nom-de-projet.co' par le nom de votre site et `root` avec les détails de votre nom d'application.

      Enregistrez ce fichier et liez-le dans un autre répertoire :

      ```bash
      cd /etc/nginx/sites-enabled 
      sudo ln -s ../sites-available/{votre-nom-de-site} 
      ls -l
      sudo rm default //supprime le répertoire par défaut
      ```

3. **Exécutez votre application Angular sur EC2 :**

   - Accédez à un répertoire où nous voulons générer la build et clonez votre code.

      ```bash
      cd /var/www
      sudo git clone https://angular-app
      ```

   - Changez de répertoire vers votre application et construisez-la :

      ```bash
      cd /var/www/my-angular-app
      ng build --prod --build-optimizer
      ```

      En cas d'une erreur Angular, vous pouvez exécuter cette commande :

      ```bash
      sudo npm update
      ```

      Cela générera le dossier /dist à l'intérieur de /var/www/my-demo-app/dist

      Redémarrez maintenant le serveur Nginx :

      ```bash
      service nginx restart
      ```

      Maintenant, votre application devrait fonctionner. Essayez l'adresse IP publique dans votre navigateur.

## Section 3: Créer une base de données RDS et la lier à votre application backend.

Vous pouvez suivre ce lien pour créer un service RDS depuis AWS :
[Creating an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html)

Importez votre ancienne base de données dans le service RDS :
[Migrate Your Application Database to Amazon RDS](https://docs.bitnami.com/aws/how-to/migrate-database-rds/)

Lorsque vous créez le service RDS, AWS vous offre un nom de domaine, copiez-le et collez-le dans le chemin d'URL dans application.properties de Spring Boot avant de générer le fichier WAR.
