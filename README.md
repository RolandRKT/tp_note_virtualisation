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

Ensuite il a fallu utilisé les commandes scp pour partager les fichiers :
scp frontend.tar.gz backend.tar.gz o22203444@172.16.0.35:~/
scp frontend.tar.gz backend.tar.gz o22203444@172.16.0.34:~/

Petite vérification rapide que les .gz sont bien présents :
o22203444@o22203444-35:~$ ls -al
total 384572
drwxr-x--- 3 o22203444 o22203444      4096 Feb  5 11:53 .
drwxr-xr-x 4 root      root           4096 Dec 19 16:20 ..
-rw------- 1 o22203444 o22203444       427 Jan 22 11:31 .bash_history
-rw-r--r-- 1 o22203444 o22203444       220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 o22203444 o22203444      3771 Mar 31  2024 .bashrc
drwx------ 2 o22203444 o22203444      4096 Jan 19 07:22 .cache
-rw-r--r-- 1 o22203444 o22203444       807 Mar 31  2024 .profile
-rw-r--r-- 1 o22203444 o22203444         0 Jan 19 07:37 .sudo_as_admin_successful
-rw-rw-r-- 1 o22203444 o22203444 368488521 Feb  5 11:53 backend.tar.gz
-rw-rw-r-- 1 o22203444 o22203444  25275566 Feb  5 11:41 frontend.tar.gz

Puis sur chaque vm (o22203444@172.16.0.35 & o22203444@172.16.0.34) il n'y avait plus qu'à charger les services :
docker load -i frontend.tar.gz
docker load -i backend.tar.gz

Ensuite je suis passé à la création du docker-stack.yml qui est fidèle au docker-compose.yml de terramino

Puis le déploiement des stacks avec ces commandes depuis le manager :
docker stack deploy -c docker-stack.yml terramino
docker stack ps terramino
docker stack services terramino

La sortie de la dernière commande (comme vérification que tout a bien fonctionné) :
o22203444@o22203444-192:~/tp_note/terramino-go$ docker stack services terramino
ID             NAME                 MODE         REPLICAS   IMAGE                          PORTS
zf89sb5figim   terramino_backend    replicated   2/2        terramino-go-backend:latest    
wgcsyw4l3988   terramino_frontend   replicated   3/3        terramino-go-frontend:latest   *:8081->8081/tcp
wm5eeyzdeh9l   terramino_redis      replicated   1/1        redis:alpine   

Par précaution supplémentaire j'ai fait un curl :
o22203444@o22203444-192:~/tp_note/terramino-go$ curl -4 http://localhost:8081
<!DOCTYPE html>
<html>
  <head>
    <title>Terramino</title>
    <link
      rel="icon"
      href="https://developer.hashicorp.com/favicon.ico"
      type="image/x-icon"
    />

    <link rel="stylesheet" type="text/css" href="terramino.css" />
  </head>

  <body class="dark-mode">
    <div class="game-wrapper">
      <div class="game-header">
        <h1 class="game-title">Terramino</h1>
        <p class="game-instructions">
          Use the arrow keys to move, rotate, and soft drop blocks.<br />
          Press the space bar to hard drop the block.
        </p>
        <div class="hashicorp-logos">
          <img src="hashicorp-logos.png" alt="HashiCorp Logos" />
        </div>
      </div>
      <div class="container">
        <!-- Game Canvas -->
        <div class="game-panel">
          <canvas width="320" height="640" id="game"></canvas>
        </div>

        <!-- Score and Next Block UI -->
        <div class="side-panel">
          <div class="score-board">
            <div class="score-section highscore-section">
              <span class="trophy-icon">
                <svg
                  fill="yellow"
                  width="18px"
                  height="18px"
                  viewBox="0 0 1024 1024"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M735.808 927.872H285.872c-17.68 0-32 14.32-32 32s14.32 32 32 32h449.936c17.68 0 32-14.32 32-32s-14.304-32-32-32zm281.502-806.24c-3.024-14.88-16.16-25.568-31.343-25.568H829.343V64.128c0-17.68-14.32-32-32-32H221.807c-17.68 0-32 14.32-32 32v31.936H38.031c-15.183 0-28.32 10.688-31.344 25.568-.944 4.624-22.4 116.752 39.904 193.152 35.84 43.92 90.607 66.928 162.495 68.976C250.078 504.912 353.15 594.624 477.278 608v222.912H381.5c-17.68 0-32 14.32-32 32s14.32 32 32 32H640.19c17.68 0 32-14.32 32-32s-14.32-32-32-32h-98.912v-222.88c124.336-13.12 227.632-102.8 268.736-224.08 74.336-1.088 130.736-24.24 167.393-69.168 62.304-76.416 40.848-188.528 39.904-193.152zM96.401 274.56c-28.336-34.496-31.184-85.41-29.744-114.497H189.81v108.032c0 17.296 1.6 34.16 3.936 50.769-43.68-4.08-76.447-18.832-97.344-44.304zm668.944-6.465c0 153.088-114.721 277.663-255.713 277.663-141.056 0-255.808-124.56-255.808-277.663V96.127H765.36v171.968h-.015zm162.255 6.463c-21.68 26.432-56.032 41.488-102.272 44.864 2.384-16.784 4.016-33.84 4.016-51.328V160.062h128c1.44 29.12-1.408 80-29.744 114.496z"
                  />
                </svg>
              </span>
              <span id="highscore" class="score-text yellow">00000000</span>
            </div>

            <div class="score-section current-score-section">
              <span class="score-label">SCORE</span>
              <span class="score-text white" id="score">00000000</span>
            </div>
          </div>

          <div class="next-blocks">
            <canvas width="64" height="64" id="next-block-1"></canvas>
            <canvas width="64" height="64" id="next-block-2"></canvas>
          </div>
        </div>
      </div>
    </div>
    <!-- Light/Dark mode toggle -->
    <button id="mode-toggle" class="mode-toggle">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="sun"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="moon"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
    </button>
    
    <!-- Debug Button -->
    <button id="debug-button">Debug</button>

    <!-- Debug Info Modal -->
    <div id="debug-modal">
      <h3>Debug Information</h3>
      <pre id="debug-info"></pre>
      <button id="close-debug">Close</button>
    </div>

    <div class="footer">
      <a href="https://developer.hashicorp.com" target="_blank"
        >developer.hashicorp.com</a
      >
    </div>

    <script src="terramino.js"></script>
  </body>
</html>

Cette sortie confirme que tout à bien fonctionné.

Pour répondre à la question 7 : 
Les précautions pour garantir la cohérence des données sont les mêmes
que celles vues lors des TP, à savoir l'utilisation des volumes.
Donc un volume redis partagé avec une seule réplique partagé de Redis
et un placement sur le manager pour être cohérent avec le rôle qu'il
a dans le swarm.

Avant de faire la question 9, encore un petit test :
o22203444@o22203444-192:~/tp_note/terramino-go$ docker stack ps terramino
ID             NAME                   IMAGE                          NODE            DESIRED STATE   CURRENT STATE           ERROR     PORTS
0o2ffnjiis0t   terramino_backend.1    terramino-go-backend:latest    o22203444-34    Running         Running 3 minutes ago             
cb5hrn0btov0   terramino_backend.2    terramino-go-backend:latest    o22203444-35    Running         Running 3 minutes ago             
wg40sbisflc8   terramino_frontend.1   terramino-go-frontend:latest   o22203444-192   Running         Running 3 minutes ago             
p2rdxraq83bm   terramino_frontend.2   terramino-go-frontend:latest   o22203444-35    Running         Running 3 minutes ago             
s95vn7wcin6y   terramino_frontend.3   terramino-go-frontend:latest   o22203444-34    Running         Running 3 minutes ago             
vv5jmqqy7vvm   terramino_redis.1      redis:alpine                   o22203444-192   Running         Running 3 minutes ago

Donc pour le test, je vais stopper un service frontend

docker ps | grep frontend
bae96e640663   terramino-go-frontend:latest      "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes           80/tcp, 8081/tcp   terramino_frontend.1.wg40sbisflc8djdkbqzr5kzbh

-> Je récupère l'id

docker stop bae96e640663

-> nouvelle vérification après quelques secondes d'attentes :
o22203444@o22203444-192:~/tp_note/terramino-go$ docker stack ps terramino | grep frontend
x4vry4zry4cb   terramino_frontend.1   terramino-go-frontend:latest   o22203444-192   Running         Running 3 seconds ago             
p2rdxraq83bm   terramino_frontend.2   terramino-go-frontend:latest   o22203444-35    Running         Running 5 minutes ago             
s95vn7wcin6y   terramino_frontend.3   terramino-go-frontend:latest   o22203444-34    Running         Running 5 minutes ago  

Le swarm a automatiquement recrée la réplique manquante.

Il en est de même pour le backend.

Et à ma surprise il en est de même pour le volume (je ne le savais pas, je pensais que ça allait faire une erreur) :

o22203444@o22203444-192:~/tp_note/terramino-go$ docker ps | grep redis
1ecf650f1f44   redis:alpine                      "docker-entrypoint.s…"   9 minutes ago   Up 9 minutes           6379/tcp           terramino_redis.1.vv5jmqqy7vvmfk794a1gqln8l
30e2936db63d   redis:alpine                      "docker-entrypoint.s…"   2 hours ago     Up 2 hours             6379/tcp           redis-haute-perf.2.csflxs4tgivxyqptbbbnzggxq
46ace2186686   redis:alpine                      "docker-entrypoint.s…"   2 hours ago     Up 2 hours             6379/tcp           redis-haute-perf.1.zlw4amq09pzma8yg5buvcuyr0
o22203444@o22203444-192:~/tp_note/terramino-go$ docker stop 1ecf650f1f44
docker stack ps terramino | grep redis
1ecf650f1f44
o22203444@o22203444-192:~/tp_note/terramino-go$ docker stack ps terramino | grep redis
vv5jmqqy7vvm   terramino_redis.1      redis:alpine                   o22203444-192   Running         Running 10 minutes ago  

Pour bien s'assurer encore une fois, même chose avec un curl, et on voit que tout est là.

o22203444@o22203444-192:~/tp_note/terramino-go$ curl -4 http://localhost:8081 | head -5
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4528  100  4528    0     0   196k      0 --:--:-- --:--:-- --:--:--  210k
<!DOCTYPE html>
<html>
  <head>
    <title>Terramino</title>
    <link

De ce que j'ai pu remarquer, la valeur du record est préservé grâce
au volume persistant, aux répliques qui se déploient automatiquement et au
placement stable du volume sur le manager avec la contrainte ajouté (node role doit être manager).

Passer de 3 à 5 répliques du frontend est relativement simple (l'avantage du fichier .yml)
il a suffit de mettre 5 à la place de 3 répliques.

Puis redéployer avec la commande suivante : docker stack deploy -c docker-stack.yml terramino

Comme d'habitude j'ai effectué quelques vérifications : 

o22203444@o22203444-192:~/tp_note/terramino-go$ docker service ls | grep frontend
341lfshebzni   terramino_frontend   replicated   5/5        terramino-go-frontend:latest      *:8081->8081/tcp

Q.11 Dans la version du compose déjà donné au départ, on remarque des ports exposés inutilement.

- Frontend : 8081 exposé
- Backend : 8080 exposé
- Redis : 6379 exposé

Et dans le docker-stack.yml crée :
- Frontend : 8081 exposé
- Backend : pas exposé (communication interne via réseau overlay)
- Redis : pas exposé (communication interne via réseau overlay)

Donc ce qui nous fait un total de seulement 1 port exposé. Car il est le seul ayant véritablement besoin de 
communiquer avec le client (l'extérieur).

Pour la question 12 (juste suivre/recopier à peu près ce qui a déjà été fait dans le TP Cluster avec Swarm).


Donc l'objectif est de répliquer le service Redis avec NFS

Mais comme on l'avait vu le problème initial c'est que le volume local ne peut pas être partagé entre répliques Redis sur différents nœuds.

alors faut appliquer le volume NFS partagé exactement comme le TP.

Mise en place serveur NFS

Sur le manager je crée un nouveau dossier
mkdir -p ./nfs_redis

Puis exécute :
docker run -d \
  --name nfs-server-redis \
  --privileged \
  -v $(pwd)/nfs_redis:/nfsshare \
  -e SHARED_DIRECTORY=/nfsshare \
  -p 2049:2049 \
  itsthenetwork/nfs-server-alpine

Ensuite, la création du volume NFS :
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=172.16.3.192,nfsvers=4,rw,nolock,soft \
  --opt device=:/ \
  redis_nfs_volume

Pour garder une trace de mes fichier yml, je n'ai pas modifier le docker-stack.yml. J'en ai simplement recrée un nommé : docker-stack-redis.yml

Avec dedans la ligne à changé, à savoir 3 répliques pour le serveur redis avec le bon volume.

Puis le déploiement: 

docker stack deploy -c docker-stack-redis-replicated.yml terramino

On peut ensuite remarqué le résultat :

o22203444@o22203444-192:~/tp_note/terramino-go$ docker service ls
ID             NAME                 MODE         REPLICAS   IMAGE                             PORTS
ta18v1gnjch1   mon-app_visualizer   replicated   1/1        dockersamples/visualizer:latest   *:8080->8080/tcp
op9wn868so9o   mon-app_web          replicated   4/4        nginx:latest                      
zze9il3gwjpk   redis-haute-perf     replicated   2/2        redis:alpine                      
oy87nly5grtw   terramino_backend    replicated   2/2        terramino-go-backend:latest       
ksovyo5xjjrl   terramino_frontend   replicated   5/5        terramino-go-frontend:latest      *:8081->8081/tcp
ufsavii1zsq7   terramino_redis      replicated   3/3        redis:alpine                      
ilvpjw8a988x   web-app              replicated   10/10      nginx:1.20         

Q13. c'est encore ce qu'on a fait lors du dernier TP.

Pour faire le comparatif, on regarde d'abord la taille que prend l'image avant opti.

o22203444@o22203444-192:~/tp_note/terramino-go$ docker images | grep "terramino-go-backend.*latest"
WARNING: This output is designed for human readability. For machine-readable output, please use --format.
terramino-go-backend:latest              e22c792ab240        1.1GB             0B 

Je sauvegarde l'ancien backend en le renommant avec un .old

Voici les changements finaux en copiant ce qui a été fait en TP :


FROM golang:1.22 AS builder
FROM ubuntu:latest AS runner
COPY --from=builder

Après le build avec docker compose build backend

On obtient maintenant :

o22203444@o22203444-192:~/tp_note/terramino-go$ docker images | grep terramino
WARNING: This output is designed for human readability. For machine-readable output, please use --format.
terramino-go-backend:latest              b4b0429c5428       93.2MB             0B        
terramino-go-frontend:latest             2eff3230ccb0       62.1MB             0B   U    

On est passé de 1.1GB à 93.2MB

Comme je n'avais déjà pas très bien compris pendant le TP j'ai du tout reprendre.
Mais finalement en suivant le TP et en faisant à côté en même temps c'est passé.

