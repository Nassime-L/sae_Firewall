Utilisation du pare feu local UFW

*Activer/désactiver le pare-feu :*

sudo ufw enable/disable 

*Vérifier le status et les règles :*

sudo ufw status (numbered pouvoir supprimer une règle avec le numéro) (verbose permet de voir la totalité des règles)

*Appliquer règles par défaut en entrée/sortie :*

Sudo ufw default allow/reject/deny incoming/outgoing

*Appliquer une règle :*

sudo ufw allow/deny/reject protocole/ports/ip (plages possibles) 

sudo ufw allow/deny/reject {proto tcp/udp/..} from ip/reseaux to any/ip/reseaux/ports

Problématiques rencontrées avec UFW :

Lorsque l’on bloque tous les flux entrant vers le serveur, le ping lui n’est pas inclus. Je pouvais ping mon serveur depuis ma machine cliente. Pour bloquer les flux icmp, il faut modifier le fichier /etc/ufw/before.rules et modifie sur la partie icmp INPUT : ACCEPT en DROP ! (et reload le firewall)

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.001.png)

J’ai aussi rajouté le même bloc mais en OUTPUT (ping depuis la machine vers une IP)

Sudo ufw allow proto tcp from 192.168.0.1 to 192.168.0.2 : autorise les flux tcp venant du client vers le serveur

Insert NUM : permet d’organiser l’ordre des règles par exemple ici :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.002.png)

Je refuse tous les flux entrant dans mon serveur mais j’autorise les fllux tcp utilisant le port 80 de mon client.

Résultat : 

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.003.png)

Il n’a pas accès.

Si l’on change l’ordre comme ceci avec (insert 2) : 

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.004.png)

Résultat sur mon client :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.005.png)



NGINX :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.006.png)

\- Décommenter la ligne « server\_tokens off », permet de ne pas spécifier la version du serveur nginx. 

\- J’ai ajouté deux lignes sur le fichier de configuration nginx :

More\_set\_headers `Serveur : apache` : permet de modifier le header serveur (apache pour tromper un attaquant par exemple) 

More clear headers `Server` : permet de supprimer complètement l’entête.

Bloquer les requêtes utilisant curl et wget :

Dans le fichier /etc/nginx/nginx.conf :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.007.png)

Résultats depuis la machine client : 

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.008.png)






Limiter le taux de connexion au serveur Nginx : 

Sur le fichier nginx.conf :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.009.png)

La ligne « **limit\_req\_zone $binary\_remote\_addr zone=limitreqsbyaddr:20m rate=10r/s;** »  permet la limitation de débit. Les paramètres requis sont une clé pour identifier les clients, une zone de mémoire partagée qui stockera l'état de la clé et la fréquence à laquelle elle a accédé à une URL restreinte aux requêtes, ainsi que le taux.

La ligne « **limit\_req\_status 430;** » sert à définir un code d’état de réponse renvoyé aux demandes rejetées.

Enfin pour appliquer cette limitation il faut indiquer la zone au serveur avec « **limit\_req zone=limitreqsbyaddr;** »

L'exemple de configuration suivant montre la limitation du taux de requêtes au serveur Web (page index). La taille de la mémoire partagée est de 10 Mo et la limite du taux de requêtes est de 1 requêtes par seconde. 
**


**Test de la limite avec Hey :**

Pour tester cette limitation, nous utiliseront la commande hey, qui se chargera d’envoyer des requête http au serveur web.

Par exemple, lorsque la limitation est appliquée, voilà le résultat pour 500000 requêtes envoyées :

![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.010.png)

On peut voir que sur les 500 000 requêtes 147 sont passées, et cela à pris 147 secondes, la limitation est donc bien appliquée.









HTTPS : 

J’ai fait ce fichier de conf de serveur Nginx qui va écouter les connexions en HTTPS sur le port 443 et utilise le protocole SSL/TLS pour sécuriser les connexions. Pour cela j’utilise un certificat SSl pour sécuriser les connexions. Les requêtes entrantes sont dans le répertoire « /var/www/html/ » pour le fichier « index.nginx-debian.html ».



![Une image contenant texte, logiciel, Page web, Site web

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.011.png)

J’autorise le port 443 (HTTPS) et refuse le 80 (HTTP), par la suite je fais une requête en https et la page est bien sécurisé.

![Une image contenant texte, capture d’écran, Police

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.012.png)

![Une image contenant texte, capture d’écran, Police

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.013.png)

On peut aussi retrouver le certificat de sécurité.

![Une image contenant texte, capture d’écran, Police, nombre

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.014.png)

SSH

Dans un premier temps je bloque le port 22 sur le serveur.

![Une image contenant texte, capture d’écran, Police

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.015.png)

Puis j’essaye de me connecter au serveur en ssh depuis la machine cliente. Sans surprise la requête n’a pas abouti.

`	`![](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.016.png)

Ensuite j’autorise le port 22, qui va me permettre de me connecter au serveur depuis le client avec la commande « ssh 192.168.0.2 ».

![Une image contenant texte, capture d’écran, Police, ligne

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.017.png)

![Une image contenant texte, Police, capture d’écran, Graphique

Description générée automatiquement](Aspose.Words.91d15f1a-4f26-4f50-bfad-e325802f9e8a.018.png)
