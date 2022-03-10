# Comment lancer un noeud Ceramic avec Docker ?

Avant de commencer il est important de comprendre comment fonctionne Ceramic. <br>
Il y a 2 dockers à lancer, le premier servira au serveur [IPFS](https://academy.bit2me.com/fr/qu%27est-ce-que-ipfs/)  le second quand à lui servira au noeud. Le noeud se connectera donc au serveur IPFS. Puis pour interagir avec votre noeuds il y a plusieurs solutions mais la solution recommandé est la suivante  : https://developers.ceramic.network/reference/core-clients/ceramic-http/ 
Nous partirons du principe que docker est installé sinon voici un tuto : https://docs.docker.com/get-docker/

# Sans docker compose
## Serveur IPFS
Le premier docker à lancer est celui de l'ipfs :

    docker run -d -p 4011:4011 -p 5011:5011 -e CERAMIC_NETWORK=testnet-clay --name ipfs-daemon ceramicnetwork/ipfs-daemon:latest
Ici rien de compliqué, on expose deux port le **4011** et le **5011** pour les rendre public, on détermine sur quel réseau on se connecte ici celui de **test** et on donne un nom à notre image ici **ipfs-daemon**.
On n'oublie pas de récupérer son adresse IP avec la commande  : 

    docker inspect -f \                                                                                                         
      '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
      ipfs-daemon

##  Node

    docker run -d \                                                                                                             
    -p 7007:7007 \
    -v chemin_du_fichier_de_conf:/root/.ceramic/daemon.config.json \
    -v chemin_ou_les_logs_seront:/root/.ceramic/logs \
    -v chemin_ou_le_statestore_sera:/root/.ceramic/statestore \
    --name js-ceramic \
    ceramicnetwork/js-ceramic:latest
  Ici c'est un peu compliqué on précise 3 volumes que l'on doit monter. Le premier est celui du fichier de configuration. C'est dedans que l'on devra définir toutes les variables nécessaire pour que notre nœud puisse tourner. Et les deux autres ils s'agit juste de chemin ou seront stocké des logs. <br>
Voici un fichier de conf :

    {
        "anchor": {
            "ethereum-rpc-url": "https://ropsten.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161"
        },
        "http-api": {
            "cors-allowed-origins": [
                ".*"
            ]
        },
        "ipfs": {
            "mode": "remote",
            "host": "http://mettre_ici_lip_de_votre_noeud_ipfs:5011"
        },
        "logger": {
            "log-level": 0,
            "log-to-files": true
        },
        "network": {
            "name": "testnet-clay" // si vous avez lancé votre serveur ipfs sur un autre réseau, il faudra mettre la meme valeur.
        },
        "node": {},
        "state-store": {
            "mode": "fs",
            "local-directory": "/root/.ceramic/statestore"
        }
    }

## Vérification si tout c'est bien passé
Si vous avez effectué toutes les commandes vous pouvez regarder les logs de votre nœud pour vérifier si tout fonctionne bien.
### Lister tous les noeuds

       > docker ps -a 
           CONTAINER ID   IMAGE                                             COMMAND                  CREATED              STATUS                    PORTS                                                                                            NAMES
    31f12de1c784   ceramicnetwork/js-ceramic:latest                  "./packages/cli/bin/…"   About a minute ago   Up About a minute         0.0.0.0:7007->7007/tcp, :::7007->7007/tcp                                                        js-ceramic
    f2b994478b4a   ceramicnetwork/ipfs-daemon:latest                 "./packages/ipfs-dae…"   2 minutes ago        Up 2 minutes              0.0.0.0:4011->4011/tcp, :::4011->4011/tcp, 0.0.0.0:5011->5011/tcp, :::5011->5011/tcp, 9011/tcp   ipfs-daemon

Si on regarde dans la colonne **STATUS**, on voit bien que nos deux docker tourne correctement. On a plus qu'a afficher les logs de notre nœud pour vérifier. La commande est la suivante `docker logs CONTAINER_ID`
Donc dans mon cas :

       > docker logs 31f12de1c784
       'Configuring pubsub to rate limit query messages to 10 per second'
    'Starting Ceramic Daemon at version 1.11.1 with config: \n' +
      '{\n' +
      '  "anchor": {\n' +
      '    "ethereum-rpc-url": "https://ropsten.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161"\n' +
      '  },\n' +
      '  "http-api": {\n' +
      '    "cors-allowed-origins": [\n' +
      '      ".*"\n' +
      '    ]\n' +
      '  },\n' +
      '  "ipfs": {\n' +
      '    "mode": "remote",\n' +
      '    "host": "http://172.17.0.2:5011"\n' +
      '  },\n' +
      '  "logger": {\n' +
      '    "log-level": 0,\n' +
      '    "log-to-files": true\n' +
      '  },\n' +
      '  "network": {\n' +
      '    "name": "testnet-clay"\n' +
      '  },\n' +
      '  "node": {},\n' +
      '  "state-store": {\n' +
      '    "mode": "fs",\n' +
      '    "local-directory": "/root/.ceramic/statestore"\n' +
      '  }\n' +
      '}'
    "Connecting to ceramic network 'testnet-clay' using pubsub topic '/ceramic/testnet-clay'"
    "Connecting to peers found in 'https://raw.githubusercontent.com/ceramicnetwork/peerlist/main/testnet-clay.json'"
    [Thu, 10 Mar 2022 15:26:07 GMT] service=pubsub peer=Qmb6hqnqt2XWffHpFfUBWgDzPSEu4b5o9WXmoNTnfA8116 event=subscribed topic=/ceramic/testnet-clay
    'This node with peerId Qmb6hqnqt2XWffHpFfUBWgDzPSEu4b5o9WXmoNTnfA8116 is not included in the peer list for Ceramic network testnet-clay. It will not be discoverable by other nodes in the network, and so data created against this node will not be available to the rest of the network.'
    'Connected to peers: /ip4/209.250.224.39/tcp/4011/p2p/QmahYokVGMAAQzKkSCA3GiiHQT2PEqkBvRHoyYJuff9Ega, /dns4/ipfs-ceramic-public-clay-external.3boxlabs.com/tcp/4012/wss/p2p/QmWiY3CbNawZjWnHXx3p3DXsg21pZYTj4CRY1iwMkhP8r3, /dns4/ipfs-ceramic-public-clay-external.ceramic.network/tcp/4012/wss/p2p/QmSqeKpCYW89XrHHxtEQEWXmznp6o336jzwvdodbrGeLTk, /dns4/ipfs-ceramic-private-clay-external.3boxlabs.com/tcp/4012/wss/p2p/QmQotCKxiMWt935TyCBFTN23jaivxwrZ3uD58wNxeg5npi, /dns4/ipfs-ceramic-private-cas-clay-external.3boxlabs.com/tcp/4012/wss/p2p/QmbeBTzSccH8xYottaYeyVX8QsKyox1ExfRx7T1iBqRyCd, /dns4/ipfs-clay-1.ceramic.geoweb.network/tcp/4012/ws/p2p/QmbDGaByZoomn3NQQjZzwaPWH6ei3ptzWK7a7ECtS35DKL'
    [Thu, 10 Mar 2022 15:26:08 GMT] service=pubsub peer=Qmb6hqnqt2XWffHpFfUBWgDzPSEu4b5o9WXmoNTnfA8116 event=received topic=/ceramic/testnet-clay message.from=QmWiY3CbNawZjWnHXx3p3DXsg21pZYTj4CRY1iwMkhP8r3 message.seqno=07945296dcffcaae message.topicIDs.0=/ceramic/testnet-clay message.typ=0 message.doc=k2t6wyse1ukyeqrxni7tchxxyduvr9l1v78buwt8fe4wp1fxbpy2qks15b4dg8 message.stream=k2t6wyse1ukyeqrxni7tchxxyduvr9l1v78buwt8fe4wp1fxbpy2qks15b4dg8 message.tip=bafyreids2h3okviznjs2clulcc77ophj24ev6nr3mwy4hdibrpfc5hnh54
    [Thu, 10 Mar 2022 15:26:08 GMT] service=pubsub peer=Qmb6hqnqt2XWffHpFfUBWgDzPSEu4b5o9WXmoNTnfA8116 event=received topic=/ceramic/testnet-clay message.from=QmVq3ShqaGbptEMGaeHJhZJtzg8cZjYyiETm4w2MdqMGmr message.seqno=982a1fa8463db62d message.topicIDs.0=/ceramic/testnet-clay message.typ=3 message.ts=1646925968077
    "Connected to anchor service 'https://cas-clay.3boxlabs.com' with supported anchor chains ['eip155:3']"
    [Thu, 10 Mar 2022 15:26:08 GMT] service=pubsub peer=Qmb6hqnqt2XWffHpFfUBWgDzPSEu4b5o9WXmoNTnfA8116 event=received topic=/ceramic/testnet-clay message.from=QmQotCKxiMWt935TyCBFTN23jaivxwrZ3uD58wNxeg5npi message.seqno=2a041d6dd5523d61 message.topicIDs.0=/ceramic/testnet-clay message.typ=0 message.doc=k2t6wyfsu4pg0nijesiin1w1up8jtxfr66b1gzcz4np2wzcm3vb0n72g46p8w8 message.stream=k2t6wyfsu4pg0nijesiin1w1up8jtxfr66b1gzcz4np2wzcm3vb0n72g46p8w8 message.tip=bagcqceraw2wd356zav2t2wqdfcmojwbbtjza3c3gjeh45hcf7765m77zt4ba

Si vous ne voyez pas d'erreur c'est que tout est bon ! Bravo votre nœud tourne ! <br>
# Avec docker compose
Si par contre ce qui vous intéresse est de déployer votre noeuds sur un service de cloud, il vous faudra un docker compose, et ca tombe bien que je vous en propose un !

    version: '3'
    services:
      ipfs-daemon:
        container_name: ipfs-daemon
        image: ceramicnetwork/ipfs-daemon:latest
        healthcheck:
          test: [ "CMD", "curl", "http://0.0.0.0:5011" ]
          timeout: 20s
          interval: 10s
          retries: 10
        ports:
          - "4011:4011"
          - "5011:5011"
        environment:
          - CERAMIC_NETWORK=testnet-clay
        networks:
          - internal
      js-ceramic:
        container_name: js-ceramic
        image: ceramicnetwork/js-ceramic:latest
        volumes:
          - chemin_du_fichier_de_conf:/root/.ceramic/daemon.config.json
          - chemin_vers_les_logs:/root/.ceramic/logs
          - chemin_du_statestore:/root/.ceramic/statestore
        ports:
          - "7007:7007"
        networks:
          - internal
        depends_on:
          ipfs-daemon:
            condition: service_healthy
    
    networks:
      internal:
        name: internal
Si jamais vous voulez faire tourner ce docker compose en local vous avez juste à le copier, copier le fichier de configuration et modifier les valeurs des volumes qu'il vous convient.
Pour le lancer rien de bien compliqué :

    docker-compose up --build --force-recreate --remove-orphans
Si vous ne voyez pas d'erreur c'est que votre docker-compose s'est bien lancé !
Et voila vous savez maintenant monter un noeud avec deux docker et avec un docker compose !
Si vous voulez savoir commencer lancer ce docker compose depuis un heroku ou aws faites le moi savoir et je completerai ce document

# Questions ?
Si vous avez des questions ou des remarques ou suggestions n’hésitez pas à ouvrir une issue avec pour titre le model suivant :  *[nom_du_projet] votre question*
Et d'ajouter le label *question* sur votre issue.
Vous pouvez aussi me contacter sur twitter à l'adresse suivante @Ronan_Dtb
