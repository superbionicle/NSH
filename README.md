# Netflix Self-Hosted (NSH), Docker version

# Sommaire
- [Pré-requis](#pré-requis)
- [Configuration réseau](#configuration-réseau)
- [Création du répertoire projet](#création-du-répertoire-projet)
- [Fichier `docker-compose` initial](#fichier-docker-compose-initial)
- [Compte Trakt](#compte-trakt)
- [Transmission](#transmission)
- [Prowlarr](#prowlarr)
  - [Bloc du `docker-compose`](#bloc-du-docker-compose)
  - [Lien avec Transmission](#lien-avec-transmission)
  - [Indexeurs](#indexeurs)
- [Radarr](#radarr)
  - [Bloc du `docker-compose`](#bloc-du-docker-compose-1)
  - [Dossier racine](#dossier-racine)
  - [Lien avec le serveur Plex](#lien-avec-le-serveur-plex)
  - [Lien avec Trakt](#lien-avec-trakt)
  - [Lien avec Prowlarr](#lien-avec-prowlarr)
  - [Lien avec Transmission](#lien-avec-transmission-1)
  - [Téléchargements](#téléchargements)
- [Sonarr](#sonarr)
- [Fichier `docker-compose.yml` final](#fichier-docker-composeyml-final)

# Pré-requis
- connaissance de Docker
- connaissance en bash
- docker et docker compose installés
- terminal de commandes accessible
- accès aux paramètres réseaux

__Remarque pour docker compose :__ dans la suite du README.md, la commande `docker compose up` sera appelée maintes fois. Une fois lancée, le terminal n'est plus accessible à moins d'utiliser `Ctrl+C` afin de tuer le processus. Au lieu de cela, vous pouvez utiliser à la place `docker compose up -d` afin de lancer les conteneurs en détaché afin de garder la main sur le terminal.

__Remarque pour le fichier `docker-compose.yml` :__ dans les explications, vous trouverez `<plex-dir>` dans les `volumes` du fichier `docker-compose.yml`. Il s'agit du chemin où est situé votre répertoire de travail. Dans mon cas à moi, il s'agit de `/Users/arthur/Downloads/plex`.

# Configuration réseau
Afin de faciliter la configuration du NSH, il est recommandé de ne pas utiliser l'adresse IP donnée via le DHCP de la box mais d'en utiliser une fixe. Pour ma part, et pour le reste des explications, mon adresse IP sera `192.168.1.200`.

Il sera intéressant aussi de changer le DNS de la machine hébergeant le NSH, afin de contourner certains sécurités Cloudflare, en mettant le DNS `1.1.1.1`, c'est-à-dire celui de Cloudflare lui-même.

__Remarque :__ attention, en changeant le DNS par défaut par celui de Cloudflare, certains sites peuvent ne plus être accessible. C'est notamment le cas pour accéder à l'interface de votre routeur Livebox par exemple.
# Création du répertoire projet
Lancez la commande `mkdir plex` et placez-vous dedans avec `cd plex`.
# Fichier `docker-compose` initial
Lancez `touch docker-compose.yml`et remplissez le fichier créé avec le contenu ci-desssus :
```yaml
services:
  plex:
    image: plexinc/pms-docker:latest
    container_name: plex
    environment:
        - PUID=1000
        - PGID=1000
        - TZ=Etc/UTC
        - VERSION=docker
        #- PLEX_CLAIM=
    volumes:
        - <plex-dir>:/config
        - <plex-dir>/movies:/movies
        - <plex-dir>/tv:/tv
    ports:
        - 32400:32400
        - 32469:32469
    restart: unless-stopped
```
Une fois fait, lancez `docker compose up` et rendez-vous sur http://localhost:32400
Connectez vous à Plex avec votre compte, ou créez vous un compte Plex.
![capture écran accueil Plex](/images/1-plex-welcome.png)
Une fois fait, vous devriez arriver sur une page similaire à celle ci-dessous.
![capture écran accueil Plex](/images/2-plex-welcome.png)
Une fois fait, accédez à [account.plex.tv/fr/claim](https://account.plex.tv/fr/claim) afin de récupérer votre code Plex. Collez-le à la suite du champ `#- PLEX_CLAIM=` dans le `docker-compose.yml` et décommentez la ligne.
Exécutez `docker compose down` puis `docker compose up`.
Choisissez le nom de votre NSH et créer une bibliothèque de films, et une de séries.
![plex name](/images/2-plex-name.png)
![biblio plex](/images/2-plex-libs.png)
# Compte Trakt
Créez un compte Trakt. Ce compte vous permettra d'ajouter des films et séries à votre _Watchlist_, afin qu'ils soient téléchargés automatiquement plus tard.

__Remarque :__ pourquoi créer un compte sur Trakt et non pas un autre site tel que TMDb, IMDb ou Sens Critique ?
Sens Critique n'est pas compatible avec Radarr et Sonarr afin de surveiller des listes.
TMDb n'est comptatible uniquement avec Radarr, tandis que IMDb l'est avec les deux. Cependant, ils ne sont exploitables qu'avec des listes.
Trakt est comptatible avec Radarr et Sonarr et permet de connecter un utilisateur plutôt qu'une liste. Ainsi, il suffira de renseigner plus tard un utilisateur et cela sera reconnu automatiquement par les deux outils.
Une fois le compte créé, ajoutez un film et une série, déjà sortis de préférence, à votre Watchlist.
![ watchlist](/images/3-watchlist.png)
# Transmission
Ajoutez au `docker-compose.yml` le bloc suivant :
```yaml
transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
        - PUID=1000
        - PGID=1000
        - TZ=Etc/UTC
    volumes:
        - <plex-dir>/transmission:/config
        - <plex-dir>/transmission/downloads/complete:/downloads/complete
        - <plex-dir>/transmission/downloads/incomplete:/downloads/incomplete
    ports:
        - 9091:9091
        - 51413:51413
        - 51413:51413/udp
    restart: unless-stopped
```
Lancez ensuite `docker compose down` et `docker compose up`.
Rendez-vous sur [localhost:9091](http://localhost:9091) pour vérifier le bon fonctionnement du conteneur.
# Prowlarr
## Bloc du `docker-compose`
Ajoutez au `docker-compose.yml` le bloc suivant :
```yaml
prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```
Lancez ensuite `docker compose down` et `docker compose up`.
Rendez-vous ensuite sur [localhost:9696](http://localhost:9696). Un pop-up apparait pour vous demander de renseigner la méthode de connexion à Prowlarr par la suite.
![pop up prowlarr](/images/4-pop-up-prowlarr.png)
Une fois connecté à Radarr, allez dans `Settings` puis `UI` afin de modifier l'interface, et notamment le format des dates et heures ou la langue.
__Remarque :__ pour la suite des explications, je basculerai l'interface en "Français" afin de faciliter les explications.
## Lien avec Transmission
Allez dans `Paramètres` et `Clients de téléchargement`. Cliquez sur le bouton `+` et sur `Transmission`.
![+](/images/4-+.png)
![tranmission](/images/4-transmission.png)
Remplissez uniquement le champ "Hôte" avec l'adresse IP de la machine et validez.
![settings transmission](/images/4-settings.png)
![transmission add](/images/4-ok.png)
## Indexeurs
__Remarque :__ par soucis de légalité, je ne montrerai ni recommanderai aucun indexeur.

Allez dans `Indexeurs` et cliquez sur le bouton `Ajouter un nouvel indexeur`.
![indexeurs](/images/5-indexeurs.png)
Choisissez votre indexeur, notamment son URL de base et validez votre choix. N'oubliez pas de tester si l'URL fonctionne correctement et n'est pas bloquée par Cloudflare ou tout autre système de sécurité.
# Radarr
## Bloc du `docker-compose`
Ajoutez au `docker-compose.yml` le bloc suivant :
```yaml
radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
        - PUID=1000
        - PGID=1000
        - TZ=Etc/UTC
    volumes:
        - <plex-dir>/radarr:/config
        - <plex-dir>/movies:/movies
        - <plex-dir>/transmission/downloads:/downloads
    ports:
        - 7878:7878
    restart: unless-stopped
```
Lancez ensuite `docker compose down` et `docker compose up`.
Rendez-vous ensuite sur [localhost:7878](http://localhost:7878). De la même manière que pour Prowlarr, un pop-up apparait pour vous demander de renseigner la méthode de connexion à Radarr par la suite.
Une fois connecté à Radarr, allez dans `Settings` puis `UI` afin de modifier l'interface, et notamment le format des dates et heures ou la langue.
__Remarque :__ pour la suite des explications, je basculerai l'interface en "Français" afin de faciliter les explications.
## Dossier racine
Allez dans `Paramètres` et `Gestion des médias`. Cliquez sur le bouton `Ajouter un dossier racine`.
![+](/images/6-+.png)
Dans notre cas, il s'agit de `/movies`.
![movies](/images/6-movies.png)
![ok](/images/6-ok.png)
Le dossier racine permet à Radarr de déplacer les films téléchargés dans ce dossier, afin qu'ils soient détectés par Plex.
## Lien avec le serveur Plex
Allez dans `Paramètres` et `Connexions`. Cliquez sur le bouton `+` et sur `Plex Media Server`.
![+](/images/7-+.png)
![PMS](/images/7-pms.png)
Remplissez les informations nécessaires, utilisez `Start OAuth` pour établir le lien avec votre serveur Plex et laissez cocher `Mettre à jour la bibliothèque` (cela permet à Radarr de mettre à jour automatiquement votre bibliothèque Plex une fois le téléchargement d'un nouveau film).
![settings](/images/7-settings.png)

__Remarque :__ attention, si vous suivez ce guide afin de tout utiliser via des conteneurs, il ne faudra pas mettre `localhost` dans le champ `Hôte` mais bien l'adresse IP de votre machine. Si au contraire tout est installé en dur sur la machine, vous pouvez mettre `localhost`.
![add plex](/images/7-ok.png)
## Lien avec Trakt
Allez dans `Paramètres` et `Listes`. Cliquez sur le bouton `+` et sur `Trakt User`.
![+](/images/8-+.png)
![trakt user](/images/8-user.png)
Cochez `Activer`, choisissez le dossier racine, remplissez l'utilisateur et utilisez `Start OAuth` pour établir le lien avec votre compte Trakt.
![settings](/images/8-settings.png)
Une fois cela fait, votre liste devrait apparait.
![trakt user](/images/8-ok.png)

__Remarque :__ depuis une certaine version, Radarr (et par conséquent Sonarr) ne font un refresh des listes que toutes les 12h. Il est donc recommander d'anticiper l'ajout de films dans la liste la veille pour le lendemain.

Pour vérifier le bon fonctionnement de notre manipulation, allez dans `Films`. Vous devriez voir normalement apparaitre les films que vous aviez au préalable ajoutés dans votre Watchlist sur Trakt. 
![films](/images/8-films.png)
## Lien avec Prowlarr
Allez dans `Paramètres` et `Général`.
Copiez la clef API.
![clef API](/images/9-api.png)
Allez sur Prowlarr. Dans `Paramètres` puis `Applications`, cliquez sur le bouton `+` et sélectionnez `Radarr`.
![+](/images/9-+.png)
![settings](/images/9-settings.png)
Remplacez les `localhost` dans les champs par l'adresse IP de la machine et renseignez la clef API, puis validez.
![ok](/images/9-ok.png)
Retournez sur Radarr. Allez dans `Paramètres` puis `Indexeurs`. Vous devriez retrouver le ou les indexeurs que vous aviez ajoutés sur Prowlarr précédement. Si vous ne les voyez pas, vérifiez que les indexeurs choisis comportent le tag `Movies`.
## Lien avec Transmission
Allez dans `Paramètres` et `Clients de téléchargement`. Cliquez sur le bouton `+`et sélectionnez `Transmission`.
![+](/images/10-+.png)
Renseignez l'adresse IP de la machine et laissez cochées les deux dernières cases, puis validez.
![settings](/images/10-settings-1.png)
![settings](/images/10-settings-2.png)

__Remarque :__ si le message `Vous utilisez docker ; Transmission enregistre les téléchargements dans /downloads/complete/radarr mais ce dossier n'est pas présent dans ce conteneur. Vérifiez vos paramètres de dossier distant et les paramètres de votre conteneur docker.` apparait dans l'onglet `Système`, il est probable qu'il manque un dossier. Il suffit juste de créer le dossier `<plex-dir>/transmission/complete/radarr` et de redémarrer vos conteneurs.
## Téléchargements
Une fois que tout est configuré, l'ajout de films devrait déclencher automatiquement leur téléchargement. Cependant, on remarque que ce n'est pas le cas pour le film qui est déjà surveillé. C'est normal car nous avons fait la configuration de l'ensemble des composants. Il suffit juste d'appuyer sur le bouton `Tout rechercher` pour commencer la recherche des films sur les différents indexeurs.
![film](/images/11-film.png)
![film surveillé](/images/11-film-watch.png)
Allez dans l'onglet `Activité` pour voir que le téléchargement a bien débuté.
![activité](/images/11-activité.png)
![dl](/images/11-dl.png)
Vous pouvez également retourner sur Transmission pour observer que le téléchargement s'effectue correctement de ce côté là.
![dl transmission](/images/11-dl-trans.png)
Une fois le téléchargement fini, Radarr commence l'état d'import pour déplacer le film dans le dossier racine.
![import](/images/11-import.png)
![ok](/images/11-film-ok.png)
Une fois l'import fini, Radarr devrait avoir mis à jour Plex et ce dernier devrait reconnaitre le film dans son répertoire.
![plex](/images/11-film-plex.png)
Si ce n'est pas le cas, il suffit de faire un scan de la bibliothèque de fichiers.
![scan plex](/images/11-scan.png)

__Remarque :__ malgré la sélection des cases pour supprimer les films téléchargés dans le répertoire `<plex-dir>/transmission/downloads/complete`, ceux si persistent. Attention donc si vous utilisez ce système sur votre ordinateur personnel, le stockage pourrait vite saturer.
![remarque](/images/11-rq.png)
_TODO: trouver un moyen de supprimer les torrents une fois ceux-là terminés, et supprimer les films en doublon dans le répertoire `<plex-dir>/transmission/downloads/complete`._
# Sonarr
Ajoutez au `docker-compose.yml` le bloc suivant :
```yaml
sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/sonarr/data:/config
      - <plex-dir>/tv:/tv
      - <plex-dir>/transmission/downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped
```
Lancez ensuite `docker compose down` et `docker compose up`. Rendez-vous ensuite sur [localhost:8989](http://localhost:8989).
La configuration de Sonarr est similaire à Radarr. Vous pouvez reprendre les points précédents pour configurer le téléchargement de séries avec Sonarr.
![]()

__Remarque :__ le dossier racine n'est pas `/movies` mais `/tv` ici.

__Remarque :__ pour déclencher manuellement la recherche sur les indexeurs, il faut aller dans `Recherché` et appuyer sur `Tout rechercher`.

![recherché](/images/12-recherché.png)
![activité](/images/12-activité.png)
# Fichier `docker-compose.yml` final
Contenu du fichier [`docker-compose.yml`](/docker-compose.yml) :
```yaml
services:
  plex:
    image: plexinc/pms-docker:latest
    container_name: plex
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - VERSION=docker
      #- PLEX_CLAIM=claim-
    volumes:
      - <plex-dir>:/config
      - <plex-dir>/movies:/movies
    ports:
      - 32400:32400
      - 32469:32469
    restart: unless-stopped

  transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/transmission:/config
      - <plex-dir>/transmission/downloads/complete:/downloads/complete
      - <plex-dir>/transmission/downloads/incomplete:/downloads/incomplete
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/radarr:/config
      - <plex-dir>/movies:/movies
      - <plex-dir>/transmission/downloads:/downloads
    ports:
      - 7878:7878
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - <plex-dir>/sonarr:/config
      - <plex-dir>/tv:/tv
      - <plex-dir>/transmission/downloads:/downloads
    ports:
      - 8989:8989
    restart: unless-stopped
```