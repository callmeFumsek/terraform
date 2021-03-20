# Terraform du SiMDE

## Disclaimer :
⚠️Ce document est en cours de rédaction, il peut contenir des incohérences, vous pouvez les relever via un commentaire sur la PR, une issue ou un message sur le Discord ! (ou un petit MP salé à l'auteur, ca marche aussi 🦄)

## Structure

### 1. / (racine)

*	**backend.tf** : Définition de la connexion au backend (le système qui sauvegarde et partage le state terraform)
*	**main.tf** : Point d'entrée principal, chargant les modules "discord_simde" et "github"
*	**variables.tf** : Définition des variables d'entrée pour terraform (les tokens d'auth etc)

### 2. .github/ (GitHub actions)

*	**workflows/workflow.yaml** : Définition des jobs de la CI (ajout automatique de labels et validation d'un *terraform plan*
*	**labeler.yml** : Définition des labels mis automatiquement sur les PR en fonction des fichiers modifiés

### 3. discord_simde/ (terraform du Discord)

*	**channels.tf** : Définition des canaux en suivant le module "channel" (voir 3.1)
*	**custom_channels.tf** : Définition des canaux spéciaux, ne suivant pas le modèle du module "channel"
*	**custom_permissions.tf** : Définition des permissions particulières
*	**invites.tf** : Définition des invitations discords aux différents canaux
*	**logo.png** : Logo du serveur
*	**members.tf** : Définition des membres du Discord
*	**outputs.tf** : Définition des valeurs a exposer hors du module pour être utilisées ailleurs (codes d'invitation)
*	**permissions.tf** : Définition des permissions liées aux roles
*	**provider.tf** : Définition du provider utilisé pour ce module
*	**roles.tf** : Définition des roles
*	**server.tf** : Définition de la resource représentant le serveur Discord
*	**variables.tf** :  Définition des variables d'entrée pour terraform (token api)

#### 3.1. channels/ (module représentant les channels d'un projet)

Chaque projet aura une catégorie contenant (au minimum) un channel textuel et un channel vocal. Cette catégorie pourra etre cachée, privée ou publique selon le cas. Le module prendra la forme suivante dans */discord_simde/channels.tf*, les valeurs à modifier son commentées :

```terraform
module private-project {							#nom de la ressource
  source      = "./channels"
  name        = "private-project" 					#nom des canaux, mettre le meme que le nom de la ressource
  api_token   = var.api_token
  server_id   = discord_server.server.id
  position    = module.autre-canal.position + 1 	#position de la catégorie, on utilise ici les positions relatives
  restricted  = true 								#catégorie publique / cachée
  permissions = local.permissions
  roles       = local.roles
}
```


*	**main.tf** : Représente les resources du module (canaux et permissions)
*	**outputs.tf** : Définition des valeurs a exposer hors du module pour être utilisées ailleurs (ID des canaux, position, etc)
*	**provider.tf** : Définition du provider utilisé pour ce module
*	**variables.tf** : Définition des variables d'entrée pour terraform (permission, token api, nom des canaux, ...)

#### 3.2. member/ (Module représentant les members du Discord)
Chaque membre du Discord SiMDE sera représenté par une ressource issue de ce module pour assurer un suivi et une maintenance simplifiée. Chaque instance prendra la forme suivante dans */discord_simde/members.tf*, les valeurs à modifier son commentées :
```terraform
module cesar_richard {							#nom de la ressource, mettre le prenom_nom reel de l'utilisateur
  source        = "./member"
  api_token     = var.api_token
  server_id     = discord_server.server.id
  username      = "licorne"						#username
  discriminator = 2476							#discriminant
  roles = {										#liste des roles à completer grâce aux roles de /discord_simde/roles.tf
    (discord_role.admin.id)  = true
    (discord_role.payutc.id) = true
    (discord_role.member.id) = false			#il est possible de retirer un role automatiquement attribuer grâce à ce booléen
  }
}
```

*	**main.tf** : Représente les resources du module (membre au sens Discord et rôles attachés)
*	**provider.tf** : Définition du provider utilisé pour ce module
*	**variables.tf** : Définition des variables d'entrée pour terraform (username, discriminant, token api, liste de roles, ...)

### 4. github/ (terraform du GitHub)

*	**outputs.tf** : Définition des valeurs a exposer hors du module pour être utilisées ailleurs
*	**providers.tf** : Définition du provider utilisé pour ce module
*	**repo_flairsou.tf** : Définition des répo du projet flairsou (forme à retravailler)
*	**repositories.tf** : Définition des répo du SiMDE (forme à retravailler)
*	**teams.tf** : Définition des Teams GitHub
*	**users.tf** : Définition des Utilisateurs GitHub en suivant le module "users" (voir 4.1)
*	**variables.tf** : Définition des variables d'entrée pour terraform (token API, organisation, ...)

#### 4.1. users/ (Module représentant les utilisateurs GitHub)

Chaque instance prendra la forme suivante dans */discord_simde/members.tf*, les valeurs à modifier son commentées :

```terraform
module "cesar_richard" {				#nom de la ressource, mettre le nom réel de la personne
  source   = "./users"
  username = "cesar-richard"			#username GitHub
  role     = "admin"					#role global dans l'organisation GitHub SiMDE (defaults to "member")
  teams = {								#liste des Teams attachées, voir /discord_simde/teams.tf
    "jeffrey" = {						#nom de la team
      team = github_team.jeffrey.id,	#id de la team
      role = "member"					#role au sein de la team
    },
    "simde" = {
      team = github_team.simde.id,
      role = "maintainer"
    }
  }
}
``` 

*	**main.tf** : Fichier représentant la resource complète. Vu sa taille et sa complexité il n'a pas été necessaire de le scinder comme les autres modules

