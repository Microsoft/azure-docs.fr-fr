---
title: Exécuter OPC Publisher - Azure | Microsoft Docs
description: Cet article explique comment exécuter et déboguer OPC Publisher. Il traite également de considérations relatives aux performances et à la mémoire.
author: dominicbetts
ms.author: dobett
ms.date: 06/10/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: f7d6581a1892ebd74a1adba5c09c0af9d3cf9d43
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92079001"
---
# <a name="run-opc-publisher"></a>Exécuter OPC Publisher

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Cet article explique comment exécuter et déboguer OPC Publisher. Il traite également de considérations relatives aux performances et à la mémoire.

## <a name="command-line-options"></a>Options de ligne de commande

L’utilisation des applications est indiquée à l’aide de l’option de ligne de commande `--help`, comme suit :

```sh/cmd
Current directory is: /appdata
Log file is: <hostname>-publisher.log
Log level is: info

OPC Publisher V2.3.0
Informational version: V2.3.0+Branch.develop_hans_methodlog.Sha.0985e54f01a0b0d7f143b1248936022ea5d749f9

Usage: opcpublisher.exe <applicationname> [<IoT Hubconnectionstring>] [<options>]

OPC Edge Publisher to subscribe to configured OPC UA servers and send telemetry to Azure IoT Hub.
To exit the application, just press CTRL-C while it is running.

applicationname: the OPC UA application name to use, required
                  The application name is also used to register the publisher under this name in the
                  IoT Hub device registry.

IoT Hubconnectionstring: the IoT Hub owner connectionstring, optional

There are a couple of environment variables which can be used to control the application:
_HUB_CS: sets the IoT Hub owner connectionstring
_GW_LOGP: sets the filename of the log file to use
_TPC_SP: sets the path to store certificates of trusted stations
_GW_PNFP: sets the filename of the publishing configuration file

Command line arguments overrule environment variable settings.

Options:
      --pf, --publishfile=VALUE
                              the filename to configure the nodes to publish.
                                Default: '/appdata/publishednodes.json'
      --tc, --telemetryconfigfile=VALUE
                              the filename to configure the ingested telemetry
                                Default: ''
  -s, --site=VALUE           the site OPC Publisher is working in. if specified
                                this domain is appended (delimited by a ':' to
                                the 'ApplicationURI' property when telemetry is
                                sent to IoT Hub.
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --ic, --iotcentral     publisher will send OPC UA data in IoTCentral
                                compatible format (DisplayName of a node is used
                                as key, this key is the Field name in IoTCentral)
                                . you need to ensure that all DisplayName's are
                                unique. (Auto enables fetch display name)
                                Default: False
      --sw, --sessionconnectwait=VALUE
                              specify the wait time in seconds publisher is
                                trying to connect to disconnected endpoints and
                                starts monitoring unmonitored items
                                Min: 10
                                Default: 10
      --mq, --monitoreditemqueuecapacity=VALUE
                              specify how many notifications of monitored items
                                can be stored in the internal queue, if the data
                                can not be sent quick enough to IoT Hub
                                Min: 1024
                                Default: 8192
      --di, --diagnosticsinterval=VALUE
                              shows publisher diagnostic info at the specified
                                interval in seconds (need log level info).
                                -1 disables remote diagnostic log and diagnostic
                                output
                                0 disables diagnostic output
                                Default: 0
      --ns, --noshutdown=VALUE
                              same as runforever.
                                Default: False
      --rf, --runforever     publisher can not be stopped by pressing a key on
                                the console, but will run forever.
                                Default: False
      --lf, --logfile=VALUE  the filename of the logfile to use.
                                Default: './<hostname>-publisher.log'
      --lt, --logflushtimespan=VALUE
                              the timespan in seconds when the logfile should be
                                flushed.
                                Default: 00:00:30 sec
      --ll, --loglevel=VALUE the loglevel to use (allowed: fatal, error, warn,
                                info, debug, verbose).
                                Default: info
        --ih, --IoT Hubprotocol=VALUE
                              the protocol to use for communication with IoT Hub (
                                allowed values: Amqp, Http1, Amqp_WebSocket_Only,
                                  Amqp_Tcp_Only, Mqtt, Mqtt_WebSocket_Only, Mqtt_
                                Tcp_Only) or IoT EdgeHub (allowed values: Mqtt_
                                Tcp_Only, Amqp_Tcp_Only).
                                Default for IoT Hub: Mqtt_WebSocket_Only
                                Default for IoT EdgeHub: Amqp_Tcp_Only
      --ms, --IoT Hubmessagesize=VALUE
                              the max size of a message which can be send to
                                IoT Hub. when telemetry of this size is available
                                it will be sent.
                                0 will enforce immediate send when telemetry is
                                available
                                Min: 0
                                Max: 262144
                                Default: 262144
      --si, --IoT Hubsendinterval=VALUE
                              the interval in seconds when telemetry should be
                                send to IoT Hub. If 0, then only the
                                IoT Hubmessagesize parameter controls when
                                telemetry is sent.
                                Default: '10'
      --dc, --deviceconnectionstring=VALUE
                              if publisher is not able to register itself with
                                IoT Hub, you can create a device with name <
                                applicationname> manually and pass in the
                                connectionstring of this device.
                                Default: none
  -c, --connectionstring=VALUE
                              the IoT Hub owner connectionstring.
                                Default: none
      --hb, --heartbeatinterval=VALUE
                              the publisher is using this as default value in
                                seconds for the heartbeat interval setting of
                                nodes without
                                a heartbeat interval setting.
                                Default: 0
      --sf, --skipfirstevent=VALUE
                              the publisher is using this as default value for
                                the skip first event setting of nodes without
                                a skip first event setting.
                                Default: False
      --pn, --portnum=VALUE  the server port of the publisher OPC server
                                endpoint.
                                Default: 62222
      --pa, --path=VALUE     the enpoint URL path part of the publisher OPC
                                server endpoint.
                                Default: '/UA/Publisher'
      --lr, --ldsreginterval=VALUE
                              the LDS(-ME) registration interval in ms. If 0,
                                then the registration is disabled.
                                Default: 0
      --ol, --opcmaxstringlen=VALUE
                              the max length of a string opc can transmit/
                                receive.
                                Default: 131072
      --ot, --operationtimeout=VALUE
                              the operation timeout of the publisher OPC UA
                                client in ms.
                                Default: 120000
      --oi, --opcsamplinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds to request the servers to sample
                                the nodes with this interval
                                this value might be revised by the OPC UA
                                servers to a supported sampling interval.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a negative value will set the sampling interval
                                to the publishing interval of the subscription
                                this node is on.
                                0 will configure the OPC UA server to sample in
                                the highest possible resolution and should be
                                taken with care.
                                Default: 1000
      --op, --opcpublishinginterval=VALUE
                              the publisher is using this as default value in
                                milliseconds for the publishing interval setting
                                of the subscriptions established to the OPC UA
                                servers.
                                please check the OPC UA specification for
                                details how this is handled by the OPC UA stack.
                                a value less than or equal zero will let the
                                server revise the publishing interval.
                                Default: 0
      --ct, --createsessiontimeout=VALUE
                              specify the timeout in seconds used when creating
                                a session to an endpoint. On unsuccessful
                                connection attemps a backoff up to 5 times the
                                specified timeout value is used.
                                Min: 1
                                Default: 10
      --ki, --keepaliveinterval=VALUE
                              specify the interval in seconds the publisher is
                                sending keep alive messages to the OPC servers
                                on the endpoints it is connected to.
                                Min: 2
                                Default: 2
      --kt, --keepalivethreshold=VALUE
                              specify the number of keep alive packets a server
                                can miss, before the session is disconneced
                                Min: 1
                                Default: 5
      --aa, --autoaccept     the publisher trusts all servers it is
                                establishing a connection to.
                                Default: False
      --tm, --trustmyself=VALUE
                              same as trustowncert.
                                Default: False
      --to, --trustowncert   the publisher certificate is put into the trusted
                                certificate store automatically.
                                Default: False
      --fd, --fetchdisplayname=VALUE
                              same as fetchname.
                                Default: False
      --fn, --fetchname      enable to read the display name of a published
                                node from the server. this will increase the
                                runtime.
                                Default: False
      --ss, --suppressedopcstatuscodes=VALUE
                              specifies the OPC UA status codes for which no
                                events should be generated.
                                Default: BadNoCommunication,
                                BadWaitingForInitialData
      --at, --appcertstoretype=VALUE
                              the own application cert store type.
                                (allowed values: Directory, X509Store)
                                Default: 'Directory'
      --ap, --appcertstorepath=VALUE
                              the path where the own application cert should be
                                stored
                                Default (depends on store type):
                                X509Store: 'CurrentUser\UA_MachineDefault'
                                Directory: 'pki/own'
      --tp, --trustedcertstorepath=VALUE
                              the path of the trusted cert store
                                Default: 'pki/trusted'
      --rp, --rejectedcertstorepath=VALUE
                              the path of the rejected cert store
                                Default 'pki/rejected'
      --ip, --issuercertstorepath=VALUE
                              the path of the trusted issuer cert store
                                Default 'pki/issuer'
      --csr                  show data to create a certificate signing request
                                Default 'False'
      --ab, --applicationcertbase64=VALUE
                              update/set this applications certificate with the
                                certificate passed in as bas64 string
      --af, --applicationcertfile=VALUE
                              update/set this applications certificate with the
                                certificate file specified
      --pb, --privatekeybase64=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as base64 string
      --pk, --privatekeyfile=VALUE
                              initial provisioning of the application
                                certificate (with a PEM or PFX fomat) requires a
                                private key passed in as file
      --cp, --certpassword=VALUE
                              the optional password for the PEM or PFX or the
                                installed application certificate
      --tb, --addtrustedcertbase64=VALUE
                              adds the certificate to the applications trusted
                                cert store passed in as base64 string (multiple
                                strings supported)
      --tf, --addtrustedcertfile=VALUE
                              adds the certificate file(s) to the applications
                                trusted cert store passed in as base64 string (
                                multiple filenames supported)
      --ib, --addissuercertbase64=VALUE
                              adds the specified issuer certificate to the
                                applications trusted issuer cert store passed in
                                as base64 string (multiple strings supported)
      --if, --addissuercertfile=VALUE
                              adds the specified issuer certificate file(s) to
                                the applications trusted issuer cert store (
                                multiple filenames supported)
      --rb, --updatecrlbase64=VALUE
                              update the CRL passed in as base64 string to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --uc, --updatecrlfile=VALUE
                              update the CRL passed in as file to the
                                corresponding cert store (trusted or trusted
                                issuer)
      --rc, --removecert=VALUE
                              remove cert(s) with the given thumbprint(s) (
                                multiple thumbprints supported)
      --dt, --devicecertstoretype=VALUE
                              the IoT Hub device cert store type.
                                (allowed values: Directory, X509Store)
                                Default: X509Store
      --dp, --devicecertstorepath=VALUE
                              the path of the iot device cert store
                                Default Default (depends on store type):
                                X509Store: 'My'
                                Directory: 'CertificateStores/IoT Hub'
  -i, --install              register OPC Publisher with IoT Hub and then exits.
                                Default:  False
  -h, --help                 show this message and exit
      --st, --opcstacktracemask=VALUE
                              ignored, only supported for backward comaptibility.
      --sd, --shopfloordomain=VALUE
                              same as site option, only there for backward
                                compatibility
                                The value must follow the syntactical rules of a
                                DNS hostname.
                                Default: not set
      --vc, --verboseconsole=VALUE
                              ignored, only supported for backward comaptibility.
      --as, --autotrustservercerts=VALUE
                              same as autoaccept, only supported for backward
                                cmpatibility.
                                Default: False
      --tt, --trustedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted cert store will always reside in a
                                directory.
      --rt, --rejectedcertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the rejected cert store will always reside in a
                                directory.
      --it, --issuercertstoretype=VALUE
                              ignored, only supported for backward compatibility.
                                the trusted issuer cert store will always
                                reside in a directory.
```

En général, vous spécifiez la chaîne de connexion du propriétaire IoT Hub uniquement lors de la première exécution de l’application. La chaîne de connexion est chiffrée et stockée dans le magasin de certificats de la plateforme. Sur les exécutions ultérieures, l’application lit la chaîne de connexion à partir du magasin de certificats. Si vous spécifiez la chaîne de connexion à chaque exécution, l’appareil qui est créé pour l’application dans le registre des appareils IoT Hub est supprimé et recréé.

## <a name="run-natively-on-windows"></a>Exécuter en mode natif sur Windows

Ouvrez le projet **opcpublisher.sln** avec Visual Studio, générez la solution, puis publiez-la. Vous pouvez démarrer l’application dans le **répertoire cible** dans lequel vous avez effectué la publication, comme suit :

```cmd
dotnet opcpublisher.dll <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-self-built-container"></a>Utiliser un conteneur généré automatiquement

Générez votre propre conteneur et démarrez-le comme suit :

```sh/cmd
docker run <your-container-name> <applicationname> [<IoT Hubconnectionstring>] [options]
```

## <a name="use-a-container-from-microsoft-container-registry"></a>Utiliser un conteneur à partir de Microsoft Container Registry

Un conteneur prédéfini est disponible dans Microsoft Container Registry. Commencez comme suit :

```sh/cmd
docker run mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

Vérifiez [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher) pour voir les systèmes d’exploitation et les architectures de processeur pris en charge. Si l’architecture de votre système d’exploitation et de votre processeur est prise en charge, Docker sélectionne automatiquement le conteneur approprié.

## <a name="run-as-an-azure-iot-edge-module"></a>Exécuter en tant que module Azure IoT Edge

OPC Publisher est prêt à être utilisé en tant que module [Azure IoT Edge](../iot-edge/index.yml). Quand vous utilisez OPC Publisher en tant que module IoT Edge, les seuls protocoles de transport pris en charge sont **Amqp_Tcp_Only** et **Mqtt_Tcp_Only**.

Pour ajouter OPC Publisher comme module à votre déploiement IoT Edge, accédez à vos paramètres IoT Hub dans le portail Azure, puis effectuez les étapes suivantes :

1. Accédez à **IoT Edge**, puis créez ou sélectionnez votre appareil IoT Edge.
1. Sélectionnez **Définir modules**.
1. Sélectionnez **Ajouter** sous **Modules de déploiement**, puis **Module IoT Edge**.
1. Dans le champ **Nom**, entrez **publisher**.
1. Dans le champ **URI de l’image**, entrez `mcr.microsoft.com/iotedge/opc-publisher:<tag>`
1. Vous trouverez les balises disponibles sur [Docker Hub](https://hub.docker.com/_/microsoft-iotedge-opc-publisher)
1. Collez le code JSON suivant dans le champ **Options de création de conteneur** :

    ```json
    {
        "Hostname": "publisher",
        "Cmd": [
            "--aa"
        ]
    }
    ```

    Cette configuration configure IoT Edge pour démarrer un conteneur appelé **publisher** à l’aide de l’image OPC Publisher. Le nom d’hôte du système du conteneur est défini sur **publisher**. OPC Publisher est appelé avec l’argument de ligne de commande suivant : `--aa`. Avec cette option, OPC Publisher approuve les certificats des serveurs OPC UA auxquels il se connecte. Vous pouvez utiliser n’importe quelle option de ligne de commande OPC Publisher. La seule limite concerne la taille des **Options de création de conteneur** prises en charge par IoT Edge.

1. Laissez les autres paramètres inchangés et sélectionnez **Enregistrer**.
1. Si vous voulez traiter la sortie d’OPC Publisher localement à l’aide d’un autre module IoT Edge, revenez à la page **Définir des modules**. Accédez ensuite à l’onglet **Spécifier des routes**, puis ajoutez une nouvelle route qui ressemble au code JSON suivant :

    ```json
    {
      "routes": {
        "processingModuleToIoT Hub": "FROM /messages/modules/processingModule/outputs/* INTO $upstream",
        "opcPublisherToProcessingModule": "FROM /messages/modules/publisher INTO BrokeredEndpoint(\"/modules/processingModule/inputs/input1\")"
      }
    }
    ```

1. De retour dans la page **Définir des modules**, sélectionnez **Suivant** jusqu’à ce que vous atteigniez la dernière page de la configuration.
1. Sélectionnez **Soumettre** pour envoyer votre configuration à IoT Edge.
1. Quand vous avez démarré IoT Edge sur votre appareil de périphérie et que le conteneur Docker **publisher** est en cours d’exécution, vous pouvez consulter la sortie du journal d’OPC Publisher à l’aide de `docker logs -f publisher` ou en consultant le fichier journal. Dans l’exemple précédent, le fichier journal est au-dessus de `d:\iiotegde\publisher-publisher.log`. Vous pouvez également utiliser l’[outil iot-edge-opc-publisher-diagnostics](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics).

### <a name="make-the-configuration-files-accessible-on-the-host"></a>Rendre les fichiers de configuration accessibles sur l’hôte

Pour rendre les fichiers de configuration du module IoT Edge accessibles dans le système de fichiers hôte, utilisez les **Options de création de conteneur** suivants. L’exemple suivant est celui d’un déploiement à l’aide de conteneurs Linux pour Windows :

```json
{
    "Hostname": "publisher",
    "Cmd": [
        "--pf=./pn.json",
        "--aa"
    ],
    "HostConfig": {
        "Binds": [
            "d:/iiotedge:/appdata"
        ]
    }
}
```

Avec ces options, OPC Publisher lit les nœuds qu’il doit publier à partir du fichier `./pn.json` et le répertoire de travail du conteneur est défini sur `/appdata` au démarrage. Avec ces paramètres, OPC Publisher lit le fichier `/appdata/pn.json` à partir du conteneur pour obtenir sa configuration. Sans l’option `--pf`, OPC Publisher tente de lire le fichier de configuration par défaut `./publishednodes.json`.

Le fichier journal, qui utilise le nom par défaut `publisher-publisher.log`, est écrit dans `/appdata` et le répertoire `CertificateStores` est également créé dans ce répertoire.

Pour rendre tous ces fichiers disponibles dans le système de fichiers hôte, la configuration du conteneur nécessite un volume de montage de liaison. La liaison `d://iiotedge:/appdata` mappe le répertoire `/appdata`, qui est le répertoire de travail actuel au démarrage du conteneur, au répertoire hôte `d://iiotedge`. Sans cette option, aucune donnée de fichier n’est conservée quand le conteneur démarre ensuite.

Si vous exécutez des conteneurs Windows, la syntaxe du paramètre `Binds` est différente. Au démarrage du conteneur, le répertoire de travail est `c:\appdata`. Pour placer le fichier de configuration dans le répertoire `d:\iiotedge` sur l’hôte, spécifiez le mappage suivant dans la section `HostConfig` :

```json
"HostConfig": {
    "Binds": [
        "d:/iiotedge:c:/appdata"
    ]
}
```

Si vous exécutez des conteneurs Linux sur Linux, la syntaxe du paramètre `Binds` est de nouveau différente. Au démarrage du conteneur, le répertoire de travail est `/appdata`. Pour placer le fichier de configuration dans le répertoire `/iiotedge` sur l’hôte, spécifiez le mappage suivant dans la section `HostConfig` :

```json
"HostConfig": {
    "Binds": [
        "/iiotedge:/appdata"
    ]
}
```

## <a name="considerations-when-using-a-container"></a>Considérations relatives à l’utilisation d’un conteneur

Les sections suivantes listent quelques informations à garder à l’esprit quand vous utilisez un conteneur :

### <a name="access-to-the-opc-publisher-opc-ua-server"></a>Accéder au serveur OPC UA d’OPC Publisher

Par défaut, le serveur OPC UA d’OPC Publisher écoute sur le port 62222. Pour exposer ce port d’entrée dans un conteneur, utilisez la commande suivante :

```sh/cmd
docker run -p 62222:62222 mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="enable-intercontainer-name-resolution"></a>Activer la résolution de noms interconteneurs

Pour activer la résolution de noms à partir du conteneur sur d’autres conteneurs, créez un réseau de pont docker défini par l’utilisateur, puis connectez le conteneur à ce réseau à l’aide de l’option `--network`. Affectez également un nom au conteneur à l’aide de l’option `--name`, comme suit :

```sh/cmd
docker network create -d bridge iot_edge
docker run --network iot_edge --name publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

Le conteneur est désormais accessible par d’autres conteneurs sur le même réseau avec le nom `publisher`.

### <a name="access-other-systems-from-within-the-container"></a>Accéder à d’autres systèmes à partir du conteneur

D’autres conteneurs sont accessibles avec les paramètres décrits dans la section précédente. Si le système d’exploitation sur lequel Docker est hébergé prend en charge DNS, l’accès à tous les systèmes qui sont connus de DNS fonctionne.

Dans les réseaux qui utilisent la résolution de noms NetBIOS, activez l’accès à d’autres systèmes en démarrant votre conteneur avec l’option `--add-host`. En fait, cette option ajoute en réalité une entrée au fichier hôte du conteneur :

```cmd/sh
docker run --add-host mydevbox:192.168.178.23  mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="assign-a-hostname"></a>Attribuer un nom d’hôte

OPC Publisher utilise le nom d’hôte de l’ordinateur sur lequel il s’exécute pour la génération de certificat et de point de terminaison. Docker choisit un nom d’hôte aléatoire si aucun n’est défini par l’option `-h`. L’exemple suivant montre comment affecter `publisher` comme nom d’hôte interne du conteneur :

```sh/cmd
docker run -h publisher mcr.microsoft.com/iotedge/opc-publisher <applicationname> [<IoT Hubconnectionstring>] [options]
```

### <a name="use-bind-mounts-shared-filesystem"></a>Utiliser des montages de liaisons (système de fichiers partagé)

Au lieu d’utiliser le système de fichiers du conteneur, vous pouvez choisir le système de fichiers hôte pour stocker les informations de configuration et les fichiers journaux. Pour configurer cette option, utilisez l’option `-v` de `docker run` en mode de montage de liaison.

## <a name="opc-ua-x509-certificates"></a>Certificats X.509 OPC UA

OPC UA utilise des certificats X.509 pour authentifier le client et le serveur OPC UA quand ils établissent une connexion et pour chiffrer la communication entre eux. OPC Publisher utilise des magasins de certificats maintenus par la pile OPC UA pour gérer tous les certificats. Au démarrage, OPC Publisher vérifie s’il existe un certificat pour lui-même. S’il n’y a pas de certificat dans le magasin de certificats et qu’aucun n’est passé sur la ligne de commande, OPC Publisher crée un certificat auto-signé. Pour plus d’informations, consultez la méthode **InitApplicationSecurityAsync** dans `OpcApplicationConfigurationSecurity.cs`.

Les certificats auto-signés ne fournissent aucune sécurité, car ils ne sont pas signés par une autorité de certification approuvée.

OPC Publisher fournit des options de ligne de commande pour :

- Récupérer les informations CSR du certificat d’application actuel utilisé par OPC Publisher.
- Provisionner OPC Publisher avec un certificat signé par une autorité de certification.
- Provisionner OPC Publisher avec une nouvelle paire de clés et un certificat signé par une autorité de certification adéquat.
- Ajouter des certificats à un pair approuvé ou à un magasin de certificats d’émetteurs approuvés.
- Ajouter une liste de révocation de certificats.
- Supprimer un certificat d’un pair approuvé ou à un magasin de certificats d’émetteurs approuvés.

Toutes ces options vous permettent de passer des paramètres à l’aide de fichiers ou de chaînes encodées en base64.

Le type de magasin par défaut pour tous les magasins de certificats est le système de fichiers, que vous pouvez changer à l’aide d’options de ligne de commande. Étant donné que le conteneur ne fournit pas de stockage persistant dans son système de fichiers, vous devez choisir un autre type de magasin. Utilisez l’option Docker `-v` pour rendre les magasins de certificats persistants dans le système de fichiers hôte ou sur un volume Docker. Si vous utilisez un volume Docker, vous pouvez passer des certificats à l’aide de chaînes encodées en base64.

L’environnement d’exécution affecte la façon dont les certificats sont rendus persistants. Évitez de créer des magasins de certificats chaque fois que vous exécutez l’application :

- En cas d’exécution en mode natif sur Windows, vous ne pouvez pas utiliser un magasin de certificats d’application de type `Directory`, car l’accès à la clé privée échoue. Dans ce cas, utilisez l’option `--at X509Store`.
- En cas d’exécution en tant que conteneurs Docker Linux, vous pouvez mapper les magasins de certificats au système de fichiers hôte avec l’option d’exécution Docker `-v <hostdirectory>:/appdata`. Cette option rend le certificat persistant entre les exécutions d’application.
- En cas d’exécution en tant que conteneur Linux Docker, si vous voulez utiliser un magasin X509 pour le certificat d’application, utilisez l’option d’exécution Docker `-v x509certstores:/root/.dotnet/corefx/cryptography/x509stores` et l’option d’application `--at X509Store`

## <a name="performance-and-memory-considerations"></a>Considérations relatives aux performances et à la mémoire

Cette section présente les options de gestion de la mémoire et des performances :

### <a name="command-line-parameters-to-control-performance-and-memory"></a>Paramètres de ligne de commande pour contrôler les performances et la mémoire

Quand vous exécutez OPC Publisher, vous devez connaître vos besoins en termes de performances et les ressources mémoire disponibles sur votre hôte.

La mémoire et les performances sont interdépendantes et l’un comme l’autre dépendent de la configuration du nombre de nœuds que vous configurez pour la publication. Vérifiez que les paramètres suivants répondent à vos besoins :

- Intervalle d’envoi à IoT Hub : `--si`
- Taille des messages IoT Hub (par défaut `1`) : `--ms`
- Capacité de la file d’attente des éléments supervisés : `--mq`

Le paramètre `--mq` contrôle la limite supérieure de la capacité de la file d’attente interne qui met en mémoire tampon toutes les notifications de changement de valeur de nœud OPC. Si OPC Publisher ne peut pas envoyer assez rapidement des messages à IoT Hub, cette file d’attente met les notifications en mémoire tampon. Le paramètre définit le nombre de notifications qui peuvent être mises en mémoire tampon. Si vous constatez que le nombre d’éléments de cette file d’attente augmente dans vos séries de tests, pour éviter de perdre des messages, vous devez :

- Réduire l’intervalle d’envoi à IoT Hub
- Augmenter la taille des messages IoT Hub

Le paramètre `--si` force OPC Publisher à envoyer des messages à IoT Hub à l’intervalle spécifié. OPC Publisher envoie un message dès que la taille de messages spécifiée par le paramètre `--ms` est atteinte, ou dès que l’intervalle spécifié par le paramètre `--si` est atteint. Pour désactiver l’option de taille des messages, utilisez `--ms 0`. Dans le cas présent, OPC Publisher utilise la plus grande taille de messages IoT Hub possible de 256 Ko pour traiter les données par lot.

Le paramètre `--ms` vous permet de traiter par lot les messages envoyés à IoT Hub. Le protocole que vous utilisez détermine si la surcharge résultant de l’envoi d’un message à IoT Hub est élevée par rapport au moment d’envoi réel de la charge utile. Si votre scénario permet une latence quand des données sont ingérées par IoT Hub, configurez OPC Publisher pour utiliser la taille de messages maximale de 256 Ko.

Avant d’utiliser OPC Publisher dans des scénarios de production, testez les performances et l’utilisation de la mémoire dans des conditions de production. Vous pouvez utiliser le paramètre `--di` pour spécifier l’intervalle, en secondes, auquel OPC Publisher écrit des informations de diagnostic.

### <a name="test-measurements"></a>Mesures de test

Les exemples de diagnostic suivants montrent des mesures avec différentes valeurs pour les paramètres `--si` et `--ms` publiant 500 nœuds avec un intervalle de publication OPC de 1 seconde.  Le test a utilisé une build Debug OPC Publisher sur Windows 10 en mode natif pendant 120 secondes. Le protocole IoT Hub était le protocole MQTT par défaut.

#### <a name="default-configuration---si-10---ms-262144"></a>Configuration par défaut (--si 10 --ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:33:05 (started @ 26.10.2017 15:31:09)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54363
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54363
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:33:04
bytes sent to IoT Hub: 12709429
avg msg size: 116600
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 10
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

La configuration par défaut envoie des données à IoT Hub toutes les 10 secondes, ou quand 256 Ko de données sont disponibles pour être ingérées par IoT Hub. Cette configuration ajoute une latence modérée d’environ 10 secondes, mais elle a la probabilité la plus faible de perdre des données en raison de la grande taille de messages. La sortie de diagnostic montre qu’il n’y a pas de mise à jour de nœuds OPC perdue : `monitored item notifications enqueue failure: 0`.

#### <a name="constant-send-interval---si-1---ms-0"></a>Intervalle d’envoi constant (--si 1--ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:35:59 (started @ 26.10.2017 15:34:03)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54243
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54243
---------------------------------
messages sent to IoT Hub: 109
last successful msg sent @: 26.10.2017 15:35:59
bytes sent to IoT Hub: 12683836
avg msg size: 116365
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 1
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

Quand la taille de messages est définie sur 0, OPC Publisher traite les données par lot en interne avec la plus grande taille de messages IoT Hub prise en charge, qui est de 256 Ko. La sortie de diagnostic montre que la taille de messages moyenne est de 115 019 octets. Dans cette configuration, OPC Publisher ne perd aucune mise à jour de valeur de nœud OPC et il bénéficie d’une latence plus faible par rapport à la valeur par défaut.

### <a name="send-each-opc-node-value-update---si-0---ms-0"></a>Envoyer chaque mise à jour de valeur de nœud OPC (--si 0 --ms 0)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:39:33 (started @ 26.10.2017 15:37:37)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 8184
monitored item notifications enqueued: 54232
monitored item notifications enqueue failure: 44624
monitored item notifications dequeued: 1424
---------------------------------
messages sent to IoT Hub: 1423
last successful msg sent @: 26.10.2017 15:39:33
bytes sent to IoT Hub: 333046
avg msg size: 234
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 96
--si setting: 0
--ms setting: 0
--ih setting: Mqtt
==========================================================================
```

Cette configuration envoie un message à IoT Hub pour chaque changement de valeur de nœud OPC. Les diagnostics montrent que la taille moyenne des messages est de 234 d’octets, ce qui est faible. L’avantage de cette configuration est qu’OPC Publisher n’ajoute aucune latence. Le nombre de mises à jour de valeur de nœud OPC perdues (`monitored item notifications enqueue failure: 44624`) est élevé, ce qui rend cette configuration inadaptée pour les scénarios où des volumes élevés de données de télémétrie doivent être publiés.

### <a name="maximum-batching---si-0---ms-262144"></a>Traitement par lot maximal (--si 0--ms 262144)

```log
==========================================================================
OpcPublisher status @ 26.10.2017 15:42:55 (started @ 26.10.2017 15:41:00)
---------------------------------
OPC sessions: 1
connected OPC sessions: 1
connected OPC subscriptions: 5
OPC monitored items: 500
---------------------------------
monitored items queue bounded capacity: 8192
monitored items queue current items: 0
monitored item notifications enqueued: 54137
monitored item notifications enqueue failure: 0
monitored item notifications dequeued: 54137
---------------------------------
messages sent to IoT Hub: 48
last successful msg sent @: 26.10.2017 15:42:55
bytes sent to IoT Hub: 12565544
avg msg size: 261782
msg send failures: 0
messages too large to sent to IoT Hub: 0
times we missed send interval: 0
---------------------------------
current working set in MB: 90
--si setting: 0
--ms setting: 262144
--ih setting: Mqtt
==========================================================================
```

Cette configuration traite par lot autant de mises à jour de valeur de nœud OPC que possible. La taille maximale des messages IoT Hub est de 256 Ko, ce qui est configuré ici. Aucun intervalle d’envoi n’est demandé, ce qui signifie que la quantité de données à ingérer par IoT Hub détermine la latence. Cette configuration a la probabilité de perte de valeur de nœud OPC la plus faible et convient pour la publication d’un grand nombre de nœuds. Quand vous utilisez cette configuration, vérifiez que votre scénario ne comporte pas de conditions qui introduisent une latence élevée si la taille de messages de 256 Ko n’est pas atteinte.

## <a name="debug-the-application"></a>Déboguer l’application

Pour déboguer l’application, ouvrez le fichier solution **opcpublisher.sln** avec Visual Studio et utilisez les outils de débogage Visual Studio.

Si vous avez besoin d’accéder au serveur OPC UA dans OPC Publisher, vérifiez que votre pare-feu permet d’accéder au port sur lequel le serveur écoute. Le port par défaut est : 62222.

## <a name="control-the-application-remotely"></a>Contrôler l’application à distance

La configuration des nœuds à publier peut être effectuée à l’aide de méthodes directes IoT Hub.

OPC Publisher implémente quelques appels de méthode directe IoT Hub supplémentaires pour lire les éléments suivants :

- Informations générales.
- Des informations de diagnostic sur les sessions, les abonnements et les éléments supervisés OPC.
- Des informations de diagnostic sur les événements et messages IoT Hub.
- Le journal de démarrage.
- Les 100 dernières lignes du journal.
- Les informations d’arrêt de l’application.

Les dépôts GitHub suivants contiennent des outils pour [configurer les nœuds à publier](https://github.com/Azure-Samples/iot-edge-opc-publisher-nodeconfiguration) et [lire les informations de diagnostic](https://github.com/Azure-Samples/iot-edge-opc-publisher-diagnostics). Ces deux outils sont également disponibles en tant que conteneurs dans Docker Hub.

## <a name="use-a-sample-opc-ua-server"></a>Utiliser un exemple de serveur OPC UA

Si vous n’avez pas de serveur OPC UA réel, vous pouvez utiliser l’[exemple de contrôleur PLC OPC UA](https://github.com/Azure-Samples/iot-edge-opc-plc) pour commencer. Cet exemple de contrôleur PLC est également disponible sur Docker Hub.

Il implémente un certain nombre de balises, lesquelles génèrent des données aléatoires et des balises avec anomalies. Vous pouvez étendre l’exemple si vous avez besoin simuler des valeurs de balises supplémentaires.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris comment exécuter OPC Publisher, les étapes suivantes recommandées servent à en savoir plus sur [OPC Twin](overview-opc-twin.md) et [OPC Vault](overview-opc-vault.md).