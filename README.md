# Netflix Self-Hosted (NSH), Docker version

# Démarrage des conteneurs
Une fois le fichier `.env` configuré avec le contenu de celui de [`.env-example`](/.env-example), lancer la commande `docker compose up`. 

Liste des conteneurs :
- [Jellyfin](http://localhost:8096/)
- [Transmission](http://localhost:9091)
- [Prowlarr](http://localhost:9696/)
- [Radarr](http://localhost:7878/)
- [Sonarr](http://localhost:8989/)

# Jellyfin
Accédez à [http://localhost:8096](http://localhost:8096) et créez le serveur ainsi qu'un compte utilisateur.\
Configurez les médiathèques.\
Choisissez un type de contenu et choisissez le dossier de destination où Jellyfin devra regarder.\
Pour les films, choisissez `/movies` et pour les séries `/tv`.\
Puis terminez la configuration du serveur Jellyfin.

# Compte Trakt
Créez un compte Trakt sur [https://app.trakt.tv/](https://app.trakt.tv/).\
Ce compte vous permettra d'ajouter des films et séries à votre _Watchlist_, afin qu'ils soient téléchargés automatiquement plus tard.\
Une fois le compte créé, ajoutez un film et une série, déjà sortis de préférence, à votre Watchlist.

# Transmission
Accédez à [http://localhost:9091](http://localhost:9091).\
Dans les paramètres sous la section `Seeding`, cochez les deux cases et renseignez la valeur `0` si vous ne souhaitez pas faire de l'upload de torrent une fois les votres téléchargés.\
Attention : ce n'est pas recommandé pour les indexeurs privés qui requiert en général d'avoir un ratio d'upload/download en général.

# Prowlarr
Accédez à [http://localhost:9696/](http://localhost:9696/)
## Lien avec Transmission
Dans `Paramètres` puis `Clients de téléchargement`, ajouter Transmission comme client de téléchargement.\
Seul le paramètre `Hôte` est nécessaire à renseigner.

## Ajouter un indexeur de téléchargement
Allez dans `Indexeurs` et cliquez sur le bouton `Ajouter un nouvel indexeur`.\
Choisissez votre indexeur, notamment son URL de base et validez votre choix.\
N'oubliez pas de tester si l'URL fonctionne correctement et n'est pas bloquée par Cloudflare ou tout autre système de sécurité.

# Radarr et Sonarr
Les explications suivantes sont identiques pour Radarr et Sonarr, bien que certaines options diffèrent légèrement dans les paramètres.\
Pour accéder à Radarr, accéder à [http://localhost:7878/](http://localhost:7878/).\
Pour accéder à Sonarr, accéder à [http://localhost:8989/](http://localhost:8989/).

## Dossier racine
Dans `Paramètres` puis `Gestion des médias`, choisissez `Ajouter un dossier racine`.\
Pour Radarr, on renseignera `/movies` et `/tv` pour Sonarr.\
Activez les paramètres avancés et cocher la case `Utiliser les liens durs au lieu de copier`.

## Lien avec Jellyfin
Dans `Paramètres` puis `Connexions`, choisissez `Emby / Jellyfin` pour ajouter Jellyfin.\
Seuls les champs `Hôtes` et `Clef API` sont à renseigner.
La clef API est à générer sur [le serveur Jellyfin](http://localhost:8096/web/#/dashboard/keys).

## Lien avec Trakt
Dans `Paramètres` puis `Listes`, choisissez `Trakt User` pour lier votre compte Trakt.\
Cochez `Activer`, choisissez le dossier racine précédemment renseigné, renseigner votre utilisateur et utilisez `Start OAuth` pour établir le lien avec Trakt.

## Lien avec Transmission
Dans `Paramètres` puis `Clients de téléchargement`, ajouter Transmission comme client de téléchargement.\
Seul le champ `Hôte` est nécessaire à renseigner.

## Lien avec Prowlarr
Dans `Paramètres` et `Général`, copiez la clef API.\
Sur Prowlarr, dans `Paramètres` puis `Applications`, sélectionnez `Radarr` et `Sonarr`.\
Renseignez le champ `Hôte` et la clef API.\
Retournez sur Radarr. Allez dans `Paramètres` puis `Indexeurs`. Vous devriez retrouver le ou les indexeurs que vous aviez ajoutés sur Prowlarr précédement. Si vous ne les voyez pas, vérifiez que les indexeurs choisis comportent le bon tag associé.

# Q&A
**Question :** Pourquoi mettre l'IP de la machine hôte plutôt que mettre `localhost` lorsqu'il faut remplir le champ `Hôte` ?\
**Réponse :** `localhost` désigne l'environnement où est le service. Or ces derniers sont dans des conteneurs. Indiquer `localhost` avec un port dans un conteneur pour accéder à un autre service ne fonctionnerait pas. En alternative, on pourrait mettre `http://` suivi du nom du service renseigné dans le fichier `docker-compose.yaml` puisque les conteneurs se repèrent entre eux grâce à cela.

**Question :** Pourquoi créer un compte sur Trakt et non pas un autre site tel que TMDb, IMDb ou Sens Critique ?\
**Réponse :** Sens Critique n'est pas compatible avec Radarr et Sonarr afin de surveiller des listes.\
TMDb n'est comptatible uniquement avec Radarr, tandis que IMDb l'est avec les deux. Cependant, ils ne sont exploitables qu'avec des listes.\
Trakt est comptatible avec Radarr et Sonarr et permet de connecter un utilisateur plutôt qu'une liste. Ainsi, il suffira de renseigner plus tard un utilisateur et cela sera reconnu automatiquement par les deux outils.\
Une fois le compte créé, ajoutez un film et une série, déjà sortis de préférence, à votre Watchlist.

**Question :** J'ai le message suivant dans l'onglet `Système` sur Radarr ou Sonarr :
```text
Vous utilisez docker ; Transmission enregistre les téléchargements dans /downloads/complete/radarr mais ce dossier n'est pas présent dans ce conteneur. Vérifiez vos paramètres de dossier distant et les paramètres de votre conteneur docker.
```
Que faire ?\
**Réponse :** Il est probable qu'il manque un dossier. Il suffit juste de créer les dossiers `${CONTENT_PATH}/transmission/complete/radarr` et `${CONTENT_PATH}/transmission/complete/tv-sonarr` puis de redémarrer vos conteneurs.

**Question :** Une fois ma configuration terminée, j'ai bien mes films qui s'affichent et l'ajout d'un nouveau déclenche automatiquement son téléchargement. Cependant, ça n'est pas le cas pour ceux qui étaient déjà surveillés. Que faire ?\
**Réponse :** Il suffit juste d'appuyer sur le bouton `Tout rechercher` pour commencer la recherche des films sur les différents indexeurs.

**Question :** Je ne vois pas comment faire pour déclencher manuellement la recherche de mes séries sur les indexeurs sur Sonarr. Comment faire ?\
**Réponse :** Il faut aller dans `Recherché` et appuyer sur `Tout rechercher`.

**Question :** Je n'arrive pas à ajouter un indexeur sur Prowlarr. Il me dit qu'il est bloqué par la protection Cloudflare. Que faire ?\
**Réponse :** Changer le DNS de la machine par `1.1.1.1`. Si cela ne fonctionne toujours pas, changer d'indexeur.