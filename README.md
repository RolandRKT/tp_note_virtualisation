# tp_note_virtualisation

Dans un premier temps j'ai initialisé un dépôt git afin de pouvoir travailler chez moi au besoin.

On commence tout de suite avec la prise en main. Donc j'ai lu le docker-compose.yml
Afin de voir les services nécessaires.

Par la suite j'ai commencé à build le frontend et le backend

Et enfin pour tester le tout, j'ai lancé la commande docker compose -d

Les questions :

Pour quel services est-il nécessaire de construire l’image plutôt que de la télécharger telle quelle
depuis hub docker?
-> backend et frontend

Quelles commandes permettent de construire chacune des images qui ne sont pas téléchargées?
-> docker compose build frontend
docker compose build backend

Le démarrage des services peut être compromis si d’autres processus de la machine virtuelle
sont déjà en écoute sur certains ports. Quels sont le ou les ports concernés?
-> - 8081 (frontend) - 8080 (backend) - 6379 (port redis par défaut)
-> Par exemple le port 8081 était bloqué. La solution que j'ai utilisé a été de supprimé tous les containers avec ces 2 commandes :
docker stop $(docker ps -q)
sudo systemctl restart docker

Avant de passer à la partie 2, j'ai vérifié que tout fonctionnait.

La première commande :
docker images | grep -E "(frontend|backend)"

Sortie :
WARNING: This output is designed for human readability. For machine-readable output, please use --format.
terramino-go-backend:latest             e22c792ab240        1.1GB             0B   U    
terramino-go-frontend:latest            2eff3230ccb0       62.1MB             0B   U 

Pour la partie swarm (question 4).

J'ai utilisé les commandes suivantes pour sauvegarder les bonnes images : 
docker save terramino-go-frontend:latest | gzip > frontend.tar.gz
docker save terramino-go-backend:latest  | gzip > backend.tar.gz

