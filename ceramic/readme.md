# Comment lancer un noeud Ceramic ?
## Si vous etes intéréssé par un docker je vous invite à aller dans le dossier [docker_compose](https://github.com/Aornn/howtolaunch/tree/main/ceramic/docker_compose)

Tutoriel en francais pour lancer un noeud pas à pas. Nous partirons du principe que vous etes sur linux ou ubuntu.
L'interface de ligne de commande Ceramic offre un moyen simple de démarrer un nœud JS Ceramic dans un environnement Node.js local. Il s'agit d'un excellent moyen de commencer à développer avec Ceramic avant de passer à un nœud hébergé dans le cloud pour les cas d'utilisation en production.

# Installation

Vous devez avoir installé au minimum node.js v14 et npm v6.
Si les commandes ne passent pas dans certains cas il faudra les lancer en tant que root

## Node et npm

### Ubuntu

    sudo apt install nodejs npm
### Mac os 

    brew install node
## Installation CLI ceramic

    npm install -g @ceramicnetwork/cli
Bien que npm v7 ne soit pas officiellement pris en charge, il se peut que vous puissiez encore le faire fonctionner. Vous devrez installer le paquet node-pre-gyp de manière globale. Ceci est nécessaire jusqu'à ce que node-webrtc, dont IPFS dépend, soit mis à jour.

    npm install -g node-pre-gyp

# Lancement du noeud Ceramic

Utilisez la commande ceramic daemon pour démarrer un nœud local JS Ceramic connecté au Clay Testnet, votre noeud sera disponible à l'adresse https://localhost:7007.
La configuration localhost vous permet de charger des flux à partir d'autres nœuds connectés au même réseau.

Cependant, les écritures sur votre nœud local ne seront disponibles que sur votre nœud et sur les autres nœuds trouvés sur la [peerlist](https://github.com/ceramicnetwork/peerlist/blob/main/testnet-clay.json).

Elles ne seront pas disponibles pour tous les nœuds du réseau.

Pour une meilleure connectivité, vous pouvez connecter votre CLI à un nœud long-lived Ceramic node.

## Lancement du noeud

    ceramic daemon
## Lancement du noeud en spécifiant un network

    ceramic daemon –network [name]

> Nom des différents réseau Ceramic possible. "mainnet", "testnet-clay",
> "dev-unstable", "local", ou "inmemory". Par défaut à "testnet-clay".  
> Remplacer [name] par la valeure voulue.

# Accessibilité du noeud depuis l'exterieur avec Ngrok

Ngrok est un service qui permet d’exposer sur internet un port qui tourne sur votre machine localement sans ouvrir les ports de votre box. Donc si vous avez un serveur qui tourne et que vous voulez le partager au monde entier il est le service idéal.

## Installation

### Ubuntu

    sudo snap install ngrok
### Mac OS

    brew install --cask ngrok

## Lancement Ngrok

    ngrok http 7007
   
## Partager l'adresse 
L’adresse se trouvera en face de la ligne : 
**Forwarding https://96d5-2a01-e0a-b16-16d0-eca9-796f-94ef-39cc.ngrok.io**

Dans mon cas l’adresse de mon noeud à partager sera : **https://96d5-2a01-e0a-b16-16d0-eca9-796f-94ef-39cc.ngrok.io**

> Si vous ne voulez pas partager votre noeud et uniquement le tester sur votre réseau local cette étape ne sert à rien.

# Questions ?
Si vous avez des questions ou des remarques ou suggestions n’hésitez pas à ouvrir une issue avec pour titre le model suivant :  *[nom_du_projet] votre question*
Et d'ajouter le label *question* sur votre issue.
Vous pouvez aussi me contacter sur twitter à l'adresse suivante @Ronan_Dtb
