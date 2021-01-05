---
title: Préparation à la production et bonnes pratiques - Azure
description: Cet article fournit des conseils sur la configuration et le déploiement du module Live Video Analytics sur IoT Edge dans des environnements de production.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 56982d84b7ffac718072683076657d56a2691d6c
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400554"
---
# <a name="production-readiness-and-best-practices"></a>Préparation à la production et bonnes pratiques

Cet article fournit des conseils sur la configuration et le déploiement du module Live Video Analytics sur IoT Edge dans des environnements de production. Consultez également l'article [Préparer le déploiement en production d'une solution IoT Edge](../../iot-edge/production-checklist.md) consacré à la préparation d'une solution IoT Edge. 

> [!NOTE]
> Pour les aspects liés à la sécurité, contactez les services informatiques de votre organisation.

## <a name="running-the-module-as-a-local-user"></a>Exécution du module en tant qu'utilisateur local

Lorsque vous déployez le module Live Video Analytics sur IoT Edge sur un périphérique, celui-ci s'exécute par défaut avec des privilèges élevés. Si vous consultez alors les journaux du module (`sudo iotedge logs {name-of-module}`), vous verrez l'avertissement suivant :

```
!! production readiness: user accounts – Warning
       LOCAL_USER_ID and LOCAL_GROUP_ID environment variables are not set. The program will run as root!
       For optimum security, make sure to set LOCAL_USER_ID and LOCAL_GROUP_ID environment variables to a non-root user and group.
       See https://aka.ms/lva-iot-edge-prod-checklists-user-accounts for more information.
```

Les sections ci-dessous expliquent comment répondre à cet avertissement.

### <a name="creating-and-using-a-local-user-account"></a>Création et utilisation d'un compte d'utilisateur local

Vous pouvez et devez exécuter le module Live Video Analytics sur IoT Edge en production à l'aide d'un compte doté du moins de privilèges possible. Les commandes suivantes, par exemple, montrent comment créer un compte d'utilisateur local sur une machine virtuelle Linux :

```
sudo groupadd -g 1010 localuser
sudo adduser --home /home/edgeuser --uid 1010 -gid 1010 edgeuser
```

Ensuite, dans le manifeste de déploiement, vous pouvez définir les variables d'environnement LOCAL_USER_ID et LOCAL_GROUP_ID pour l'utilisateur et le groupe non racine correspondants :

```
"lvaEdge": {
"version": "1.0",
…
"env": {
    "LOCAL_USER_ID": 
    {
        "value": "1010"
    },
    "LOCAL_GROUP_ID": {
        "value": "1010"
    }
}
},
…
```

### <a name="granting-permissions-to-device-storage"></a>Octroi d'autorisations au stockage du périphérique

Le module Live Video Analytics sur IoT Edge doit avoir la possibilité d'écrire des fichiers dans le système de fichiers local dans les cas suivants :

* Utilisation d’une propriété de jumeau de module [`applicationDataDirectory`](module-twin-configuration-schema.md#module-twin-properties), sachant que vous devez spécifier un répertoire du système de fichiers local pour le stockage des données de configuration
* Utilisation d'un graphique multimédia pour l'enregistrement de vidéo dans le cloud, sachant que le module requiert l'utilisation d'un répertoire du périphérique comme cache (voir l'article [Enregistrement vidéo continu](continuous-video-recording-concept.md) pour plus d'informations)
* [Enregistrement dans un fichier local](event-based-video-recording-concept.md#video-recording-based-on-events-from-other-sources), sachant que vous devez spécifier le chemin de la vidéo enregistrée.

Dans tous les cas, vous devez vous assurer que le compte d'utilisateur ci-dessus a accès au répertoire qui convient. Prenons l'exemple de applicationDataDirectory. Vous pouvez créer un répertoire sur le périphérique et lier le stockage du périphérique au stockage du module. 

```
sudo mkdir /var/local/mediaservices
sudo chown -R edgeuser /var/local/mediaservices
```

Ensuite, à partir des options de création du module de périphérie dans le manifeste de déploiement, vous pouvez ajouter un paramètre de liaison mappant le répertoire (« var/local/mediaservices/ ») ci-dessus à un répertoire du module (par exemple, « /var/lib/azuremediaservices/ »). Et vous devez utiliser ce dernier répertoire comme valeur pour applicationDataDirectory.

```
"lvaEdge": {
    "version": "1.0",
    "type": "docker",
    "status": "running",
    "restartPolicy": "always",
    "settings": {
        "image": "mcr.microsoft.com/media/live-video-analytics:1.0",
        "createOptions": "{\"HostConfig\":{\"LogConfig\":{\"Type\":\"\",\"Config\":{\"max-size\":\"10m\",\"max-file\":\"10\"}},\"Binds\":[\"/var/local/mediaservices/:/var/lib/azuremediaservices/\"]}}"
    },
    "env": {
        "LOCAL_USER_ID": 
        {
            "value": "1010"
        },
        "LOCAL_GROUP_ID": {
            "value": "1010"
        }
    }
    },
    …
    
    "lvaEdge": {
    "properties.desired": {
    "applicationDataDirectory": "/var/lib/azuremediaservices",
    …
    }
}
```

Si vous examinez les exemples de graphiques multimédias du guide de démarrage rapide et des tutoriels, comme l'[enregistrement vidéo continu](continuous-video-recording-tutorial.md), vous remarquerez que le répertoire du cache multimédia (localMediaCachePath) utilise un sous-répertoire sous applicationDataDirectory. Il s'agit de l'approche recommandée, car le cache contient des données temporaires.

### <a name="naming-video-assets-or-files"></a>Attribution d'un nom aux fichiers et ressources vidéo

Les graphiques multimédias permettent de créer des ressources dans le cloud ou des fichiers mp4 en périphérie. Les ressources multimédias peuvent être générées par [enregistrement vidéo continu](continuous-video-recording-tutorial.md) ou par [enregistrement vidéo basé sur des événements](event-based-video-recording-tutorial.md). Bien que ces ressources et fichiers puissent être nommés comme vous le souhaitez, la structure d'attribution de nom recommandée pour les ressources multimédias basées sur l'enregistrement vidéo continu est « &lt;texte&gt;-${System.GraphTopologyName}-${System.GraphInstanceName} ».   

Le modèle de substitution est défini par le signe $ suivi d’accolades : **${variableName}** .  

Par exemple, vous pouvez définir assetNamePattern sur le récepteur de ressources comme suit :

```
"assetNamePattern": "sampleAsset-${System.GraphTopologyName}-${System.GraphInstanceName}
```

Pour les ressources générées par enregistrement vidéo basé sur des événements, le modèle d'attribution de nom recommandé est « &lt;texte&gt;-${System.DateTime} ». La variable système garantit que les ressources ne seront pas remplacées si des événements se produisent en même temps. Par exemple, vous pouvez définir assetNamePattern sur le récepteur de ressources comme suit :

```
"assetNamePattern": "sampleAssetFromEVR-LVAEdge-${System.DateTime}"
```

Si vous exécutez plusieurs instances du même graphique, vous pouvez utiliser le nom de la topologie du graphique et le nom de l'instance pour les différencier. Par exemple, vous pouvez définir assetNamePattern sur le récepteur de ressources comme suit :

```
"assetNamePattern": "sampleAssetFromEVR-${System.GraphTopologyName}-${System.GraphInstanceName}-${System.DateTime}"
```

Pour les clips vidéo mp4 générés en périphérie par enregistrement vidéo basé sur des événements, le modèle d'attribution de nom recommandé doit inclure DateHeure et, en présence de plusieurs instances du même graphique, il est recommandé d'utiliser les variables système GraphTopologyName et GraphInstanceName. Par exemple, vous pouvez définir filePathPattern sur le récepteur de fichiers comme suit : 

```
"fileNamePattern": "/var/media/sampleFilesFromEVR-${fileSinkOutputName}-${System.DateTime}"
```

ou 

```
"fileNamePattern": "/var/media/sampleFilesFromEVR-${fileSinkOutputName}--${System.GraphTopologyName}-${System.GraphInstanceName} ${System.DateTime}"
```
>[!NOTE]
> Dans l’exemple ci-dessus, la variable **fileSinkOutputName** est un exemple de nom de variable que vous définissez dans la topologie de graphe. Il **ne s’agit pas** d’une variable système. 

#### <a name="system-variables"></a>Variables système
Variables définies par le système que vous pouvez utiliser :

|Variable système|Description|Exemple|
|-----------|-----------|-----------|
|System.DateTime|Date/heure UTC au format conforme de fichier ISO8601 (représentation de base AAAAMMJJThhmmss).|20200222T173200Z|
|System.PreciseDateTime|Date/heure UTC au format conforme de fichier ISO8601 avec les millisecondes (représentation de base AAAAMMJJThhmmss.sss).|20200222T173200.123Z|
|System.GraphTopologyName|Nom fourni par l’utilisateur de la topologie de graphe en cours d’exécution.|IngestAndRecord|
|System.GraphInstanceName|Nom fourni par l’utilisateur de l’instance de graphe en cours d’exécution.|camera001|

>[!TIP]
> Impossible d’utiliser System.PreciseDateTime lors du nommage des ressources en raison du point « . » dans le nom
### <a name="keeping-your-vm-clean"></a>Nettoyage de votre machine virtuelle

La machine virtuelle Linux que vous utilisez en tant que périphérique peut cesser de répondre en l'absence de gestion régulière. Il est essentiel de nettoyer les caches, d'éliminer les packages inutiles et de supprimer les conteneurs inutilisés de la machine virtuelle. Pour ce faire, voici un ensemble de commandes recommandées que vous pouvez utiliser sur votre machine virtuelle de périphérie.

1. `sudo apt-get clean`

    La commande apt-get clean efface du référentiel local les fichiers de package récupérés qui sont conservés dans/var/cache. Les répertoires qu'elle nettoie sont /var/cache/apt/archives/ et /var/cache/apt/archives/partial/. Les seuls fichiers qu'elle laisse dans/var/cache/apt/archives sont le fichier de verrouillage et le sous-répertoire partiel. La commande apt-get clean est généralement utilisée pour libérer de l'espace disque en fonction des besoins, notamment dans le cadre d'une maintenance planifiée régulière. Pour plus d'informations, reportez-vous à [Nettoyer avec apt-get](https://www.networkworld.com/article/3453032/cleaning-up-with-apt-get.html).
1. `sudo apt-get autoclean`

    L'option apt-get autoclean, comme apt-get clean, efface du référentiel local les fichiers de package récupérés, mais elle supprime uniquement les fichiers qui ne peuvent plus être téléchargés et qui sont inutiles. Cela permet d'éviter que votre cache ne devienne trop volumineux.
1. `sudo apt-get autoremove1`

    L'option auto remove supprime les packages qui ont été installés automatiquement parce que d'autres packages en avaient besoin, mais qui ne sont plus nécessaires suite à la suppression de ces autres packages.
1. `sudo docker image ls` : fournit une liste d'images Docker sur votre système de périphérie
1. `sudo docker system prune`

    Docker adopte une approche conservatrice en matière de nettoyage des objets inutilisés (souvent appelés « garbage collection »), tels que les images, les conteneurs, les volumes et les réseaux : ces objets ne sont généralement pas supprimés, sauf si vous le demandez explicitement à Docker. Cela peut amener Docker à utiliser de l'espace disque supplémentaire. Pour chaque type d'objet, Docker fournit une commande d'élagage. En outre, vous pouvez utiliser la commande d'élagage du système Docker pour nettoyer plusieurs types d'objets à la fois. Pour plus d'informations, reportez-vous à [Élagage des objets Docker inutilisés](https://docs.docker.com/config/pruning/).
1. `sudo docker rmi REPOSITORY:TAG`

    Au fil des mises à jour du module de périphérie, votre instance de Docker peut contenir des versions anciennes de celui-ci. Dans ce cas, il est conseillé d'utiliser la commande Docker rmi pour supprimer des images spécifiques identifiées par leur balise de version.

## <a name="next-steps"></a>Étapes suivantes

[Démarrage rapide : Bien démarrer – Live Video Analytics sur IoT Edge](get-started-detect-motion-emit-events-quickstart.md)
