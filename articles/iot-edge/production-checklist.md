---
title: Préparer le déploiement en production d’une solution - Azure IoT Edge
description: Découvrez comment faire passer votre solution Azure IoT Edge du développement à la production, et notamment comment configurer vos appareils avec les certificats nécessaires et élaborer un plan de déploiement des futures mises à jour du code.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/10/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7cabae837656611813d44017ce2e1112f06066ef
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013290"
---
# <a name="prepare-to-deploy-your-iot-edge-solution-in-production"></a>Préparer le déploiement en production d’une solution IoT Edge

Au moment de faire passer votre solution IoT Edge du développement à la production, vérifiez qu’elle est configurée de façon à offrir des performances continues.

Les informations indiquées dans cet article ne se valent pas toutes. Pour clarifier l’ordre de priorité, chaque section commence par une liste qui divise le travail en deux sections : **Important**, donc à effectuer avant de passer en production, ou **Utile**, c’est-à-dire bon à savoir.

## <a name="device-configuration"></a>Configuration de l’appareil

Il existe de nombreux types d’appareils IoT Edge : un Raspberry Pi, un portable, une machine virtuelle sur un serveur, etc. Vous pouvez avoir accès à l’appareil physiquement ou via une connexion virtuelle ; il peut aussi être isolé pendant de longues périodes. Dans les deux cas, l’objectif est de vérifier qu’il est configuré de façon à fonctionner de manière adéquate.

* **Important**
  * Installer les certificats de production
  * Élaborer un plan de gestion des appareils
  * Utiliser Moby comme moteur de conteneur

* **Utile**
  * Choisir un protocole en amont

### <a name="install-production-certificates"></a>Installer les certificats de production

Un certificat d’autorité de certification doit être installé sur chaque appareil IoT Edge en production. Il est ensuite déclaré auprès du runtime IoT Edge dans le fichier config.yaml. Pour faciliter les scénarios de développement et de test, le runtime IoT Edge crée des certificats temporaires si aucun certificat n’est déclaré dans le fichier config.yaml. Toutefois, ces certificats temporaires expirent au bout de trois mois et ne sont pas sécurisés pour les scénarios de production. Dans les scénarios de production, vous devez fournir votre propre certificat d’autorité de certification d’appareil, soit issu d’une autorité de certification auto-signée, soit acheté auprès d’une autorité de certification commerciale.

> [!NOTE]
> Il existe actuellement dans libiothsm une limitation empêchant l’utilisation de certificats qui expirent le 1er janvier 2038 ou après cette date.

Pour comprendre le rôle du certificat d’autorité de certification d’appareil, voir [Comment Azure IoT Edge utilise les certificats](iot-edge-certs.md).

Pour savoir comment installer des certificats sur un appareil IoT Edge et y faire référence dans le fichier config.yaml, consultez [Gestion d’un certificat sur un appareil IoT Edge](how-to-manage-device-certificates.md).

### <a name="have-a-device-management-plan"></a>Élaborer un plan de gestion des appareils

Avant de mettre un appareil en production, il faut savoir comment gérer les mises à jour à venir. Pour un appareil IoT Edge, il peut y avoir plusieurs composants à mettre à jour :

* Microprogramme de l’appareil
* Bibliothèques du système d’exploitation
* Moteur de conteneur, comme Moby
* Démon IoT Edge
* Certificats d’autorité de certification

Pour plus d’informations, consultez [Mettre à jour le runtime IoT Edge](how-to-update-iot-edge.md). Les méthodes actuelles de mise à jour du démon IoT Edge exigent un accès physique ou SSH à l’appareil IoT Edge. Si vous avez de nombreux appareils à mettre à jour, vous pouvez ajouter les étapes de mise à jour à un script ou utiliser un outil d’automatisation comme Ansible.

### <a name="use-moby-as-the-container-engine"></a>Utiliser Moby comme moteur de conteneur

Un moteur de conteneur fait partie des prérequis de tous les appareils IoT Edge. Seul le moteur Moby est pris en charge en production. Les autres moteurs de conteneur, comme Docker, fonctionnent avec IoT Edge et sont utilisables à des fins de développement. Le moteur Moby peut être redistribué s’il est utilisé avec Azure IoT Edge ; par ailleurs, Microsoft en assure la maintenance.

### <a name="choose-upstream-protocol"></a>Choisir un protocole en amont

Vous pouvez configurer le protocole (qui détermine le port utilisé) pour les communications en amont vers IoT Hub, tant pour l’agent IoT Edge que pour le hub IoT Edge. Le protocole par défaut, AMQP, est modifiable en fonction de la configuration réseau.

Les modules runtime ont tous les deux une variable d’environnement **UpstreamProtocol**, dont les valeurs valides sont les suivantes :

* MQTT
* AMQP
* MQTTWS
* AMQPWS

Configurez la variable UpstreamProtocol pour l’agent IoT Edge dans le fichier config.yaml sur l’appareil. Par exemple, si l’appareil IoT Edge se trouve derrière un serveur proxy qui bloque les ports AMQP, il peut se révéler nécessaire de configurer l’agent IoT Edge de façon à ce qu’il utilise AMQP sur WebSocket (AMQPWS) pour établir la connexion initiale à IoT Hub.

Une fois l’appareil IoT Edge connecté, poursuivez la configuration de la variable UpstreamProtocol pour les deux modules de runtime dans les déploiements à venir. Vous trouverez un exemple de ce processus dans [Configurer un appareil IoT Edge pour communiquer via un serveur proxy](how-to-configure-proxy-support.md).

## <a name="deployment"></a>Déploiement

* **Utile**
  * Rester cohérent avec le protocole en amont
  * Configurer le stockage hôte pour les modules système
  * Réduire l’espace mémoire utilisé par le hub IoT Edge
  * Ne pas utiliser les versions de débogage des images de module

### <a name="be-consistent-with-upstream-protocol"></a>Rester cohérent avec le protocole en amont

Si vous avez configuré l’agent IoT Edge sur votre appareil IoT Edge de façon à ce qu’il utilise un protocole autre que le protocole par défaut (AMQP), déclarez le même protocole dans tous les déploiements futurs. Par exemple, si votre appareil IoT Edge se trouve derrière un serveur proxy qui bloque les ports AMQP, vous avez probablement configuré l’appareil se sorte qu’il se connecte via AMQP sur WebSocket (AMQPWS). Lorsque vous déployez des modules sur l’appareil, configurez le même protocole AMQPWS pour l’agent IoT Edge et le hub IoT Edge. Sinon, le protocole par défaut, AMQP, remplacera les paramètres et vous empêchera de vous reconnecter.

Il suffit de configurer la variable d’environnement UpstreamProtocol pour les deux modules : l’agent IoT Edge et le hub IoT Edge. Tous les modules supplémentaires adoptent le protocole défini dans les modules de runtime.

Vous trouverez un exemple de ce processus dans [Configurer un appareil IoT Edge pour communiquer via un serveur proxy](how-to-configure-proxy-support.md).

### <a name="set-up-host-storage-for-system-modules"></a>Configurer le stockage hôte pour les modules système

Les modules de hub et d’agent IoT Edge utilisent le stockage local pour maintenir l’état et activer la messagerie entre les modules, les appareils et le cloud. Pour une fiabilité et des performances optimales, configurez les modules système pour qu’ils utilisent le stockage sur le système de fichiers hôte.

Pour plus d’informations, consultez [Stockage hôte pour les modules système](how-to-access-host-storage-from-module.md).

### <a name="reduce-memory-space-used-by-iot-edge-hub"></a>Réduire l’espace mémoire utilisé par le hub IoT Edge

Si vous déployez des appareils contraints avec une quantité de mémoire disponible limitée, vous pouvez configurer le hub IoT Edge de façon à ce qu’il s’exécute avec une capacité rationalisée et utilise moins d’espace disque. Ces configurations limitent les performances du hub IoT Edge. Trouvez l’équilibre adapté à votre solution.

#### <a name="dont-optimize-for-performance-on-constrained-devices"></a>Ne pas optimiser les performances sur les appareils contraints

Le hub IoT Edge, optimisé par défaut du point de vue des performances, tente d’allouer de grands blocs de mémoire. Cette configuration risque provoquer des problèmes de stabilité sur les petits appareils, comme le Raspberry Pi. Si vous déployez des appareils offrant des ressources limitées, vous pouvez si vous le souhaitez définir la variable d’environnement **OptimizeForPerformance** sur **false** sur le hub IoT Edge.

Lorsque la valeur d’**OptimizeForPerformance** est **true**, l’en-tête du protocole MQTT utilise PooledByteBufferAllocator, qui offre de meilleures performances, mais alloue plus de mémoire. L’allocateur ne fonctionne pas correctement sur des systèmes d’exploitation 32 bits ou sur des appareils ne disposant pas de suffisamment de mémoire. En outre, en cas d’optimisation des performances, RocksDB alloue plus de mémoire pour son rôle de fournisseur de stockage local.

Pour plus d’informations, consultez [Problèmes de stabilité sur les appareils plus petits](troubleshoot-common-errors.md#stability-issues-on-smaller-devices).

#### <a name="disable-unused-protocols"></a>Désactiver les protocoles inutilisés

Il existe un autre moyen d’optimiser les performances du hub IoT Edge et de réduire son utilisation de la mémoire : désactiver les têtes de tous les protocoles non utilisés dans la solution.

Les têtes de protocole se configurent en définissant des variables d’environnement booléennes pour le module du hub IoT Edge dans les manifestes de déploiement. Voici les trois variables en question :

* **amqpSettings__enabled**
* **mqttSettings__enabled**
* **httpSettings__enabled**

Ces variables comportent toutes les trois *deux traits de soulignement*. Elles peuvent être définies sur true ou false.

#### <a name="reduce-storage-time-for-messages"></a>Réduire le temps de stockage des messages

Le module du hub IoT Edge stocke temporairement les messages si, pour une raison ou pour une autre, il n’est pas possible de les remettre à IoT Hub. Vous pouvez configurer la durée pendant laquelle le hub IoT Edge conserve les messages non remis avant qu’ils n’expirent. Si vous avez des problèmes de mémoire sur votre appareil, vous pouvez diminuer la valeur **timeToLiveSecs** dans le jumeau de module du hub IoT Edge.

La valeur par défaut du paramètre timeToLiveSecs est de 7 200 secondes, soit deux heures.

### <a name="do-not-use-debug-versions-of-module-images"></a>Ne pas utiliser les versions de débogage des images de module

Lors du passage de scénarios de test à des scénarios de production, pensez à supprimer les configurations de débogage des manifestes de déploiement. Vérifiez qu’aucune des images de modules des manifestes de déploiement ne comporte le suffixe **\.debug**. Si vous avez ajouté des options de création pour exposer les ports dans les modules à des fins de débogage, supprimez-les également.

## <a name="container-management"></a>Gestion de conteneur

* **Important**
  * Gérer l’accès au registre de conteneurs
  * Utiliser des balises pour gérer les versions
* **Utile**
  * Stocker les conteneurs Runtime dans votre registre privé

### <a name="manage-access-to-your-container-registry"></a>Gérer l’accès au registre de conteneurs

Avant de déployer des modules sur des appareils IoT Edge en production, veillez à contrôler l’accès à votre registre de conteneurs pour éviter que des intrus ne puissent accéder à vos images conteneur ou les modifier. Utilisez un registre de conteneurs privé, et non public, pour gérer les images conteneur.

Dans les tutoriels et autres documents, nous prescrivons d’utiliser les mêmes informations d’identification de registre de conteneur sur l’appareil IoT Edge que sur l’ordinateur de développement. Ces instructions, qui ne sont destinées qu’à aider à configurer plus facilement les environnements de test et de développement, ne doivent pas être suivies dans un scénario de production.

Pour un accès plus sécurisé à votre Registre, vous avez le choix entre plusieurs [options d’authentification](../container-registry/container-registry-authentication.md). Une authentification populaire et recommandée consiste à utiliser un principal de service Active Directory adapté aux applications ou aux services pour extraire des images de conteneur de manière automatisée ou sans assistance (headless/sans périphérique de contrôle), comme le font les appareils IoT Edge.

Pour créer un principal de service, exécutez les deux scripts comme décrit dans [Créer un principal de service](../container-registry/container-registry-auth-service-principal.md#create-a-service-principal). Ces scripts effectuent les tâches suivantes :

* Le premier script crée le principal du service. Il génère l’ID du principal de service et le mot de passe du principal de service. Conservez ces valeurs en lieu sûr dans vos dossiers.

* Le deuxième script crée des attributions de rôles à accorder au principal de service, qui peuvent être exécutées ultérieurement si nécessaire. Nous vous recommandons d’appliquer le rôle d’utilisateur **acrPull** pour le paramètre `role`. Pour obtenir la liste des rôles, consultez [Autorisations et rôles Azure Container Registry](../container-registry/container-registry-roles.md).

Pour vous authentifier à l’aide d’un principal de service, fournissez l’ID et le mot de passe du principal de service que vous avez obtenus grâce au premier script. Spécifiez ces informations d’identification dans le manifeste de déploiement.

* Pour le nom d’utilisateur ou l’ID client, spécifiez l’ID du principal de service.

* Pour le mot de passe ou la clé secrète client, spécifiez le mot de passe du principal de service.

> [!NOTE]
> Après avoir implémenté une authentification de sécurité renforcée, désactivez le paramètre **Utilisateur administrateur** afin que l’accès par défaut avec le nom d’utilisateur/le mot de passe ne soit plus possible. Dans le Registre de conteneurs du portail Azure, dans le menu du volet gauche sous **Paramètres**, sélectionnez **Clés d’accès**.

### <a name="use-tags-to-manage-versions"></a>Utiliser des balises pour gérer les versions

Une balise est un concept Docker servant à faire la distinction entre les versions des conteneurs Docker. Ce sont des suffixes comme **1.0**, ajoutés à la fin d’un référentiel de conteneur. Exemple : **mcr.microsoft.com/azureiotedge-agent:1.0**. Les balises étant mutables et modifiables pour pointer vers un autre conteneur à tout moment, il est essentiel que votre équipe se mette d’accord sur une convention à suivre pour mettre à jour vos images de module par la suite.

Les balises aident également à appliquer des mises à jour sur les appareils IoT Edge. Lorsque vous envoyez une version mise à jour d’un module à votre registre de conteneurs, incrémentez la balise. Ensuite, transmettez un nouveau déploiement sur vos appareils avec la balise incrémentée. Le moteur de conteneur la reconnaîtra comme une nouvelle version et extraira la dernière version du module sur l’appareil.

Pour un exemple de convention de balise, voir [Mettre à jour le runtime IoT Edge](how-to-update-iot-edge.md#understand-iot-edge-tags) : vous découvrirez comment IoT Edge utilise des balises propagées et des balises spécifiques pour effectuer le suivi des versions.

### <a name="store-runtime-containers-in-your-private-registry"></a>Stocker les conteneurs Runtime dans votre registre privé

Vous savez comment stocker vos images de conteneur pour les modules de code personnalisés dans votre registre privé Azure, mais vous pouvez également l’utiliser pour stocker des images de conteneur publiques, comme pour les modules runtime edgeAgent et edgeHub. Cela peut être nécessaire si vous avez des restrictions de pare-feu très strictes, car ces conteneurs de runtime sont stockés dans le registre de conteneurs Microsoft (MCR).

Obtenez les images avec la commande Docker pull pour les placer dans votre registre privé. N’oubliez pas que vous devez mettre à jour les images avec chaque nouvelle version du runtime IoT Edge.

| Conteneur de runtime IoT Edge | Commande Docker pull |
| --- | --- |
| [Agent Azure IoT Edge](https://hub.docker.com/_/microsoft-azureiotedge-agent) | `docker pull mcr.microsoft.com/azureiotedge-agent` |
| [Hub Azure IoT Edge](https://hub.docker.com/_/microsoft-azureiotedge-hub) | `docker pull mcr.microsoft.com/azureiotedge-hub` |

Ensuite, veillez à mettre à jour les références d’image dans le fichier deployment.template.json pour les modules système edgeAgent et edgeHub. Remplacez `mcr.microsoft.com` par le nom et le serveur de votre registre pour les deux modules.

* edgeAgent :

    `"image": "<registry name and server>/azureiotedge-agent:1.0",`

* edgeHub :

    `"image": "<registry name and server>/azureiotedge-hub:1.0",`

## <a name="networking"></a>Mise en réseau

* **Utile**
  * Vérifier la configuration sortante/entrante
  * Autoriser les connexions à partir d’appareils IoT Edge
  * Configurer la communication via un proxy

### <a name="review-outboundinbound-configuration"></a>Vérifier la configuration sortante/entrante

Les canaux de communication entre Azure IoT Hub et IoT Edge sont toujours configurés pour être sortants. Dans la plupart des scénarios IoT Edge, seules trois connexions sont nécessaires. Une connexion doit être établie entre le moteur de conteneur et le ou les registres de conteneurs qui contiennent les images de module. Le runtime IoT Edge doit être connecté à IoT Hub pour récupérer des informations de configuration des appareils et envoyer des messages et des données de télémétrie. Enfin, si vous utilisez l’approvisionnement automatique, le démon IoT Edge doit se connecter au service Device Provisioning. Pour plus d’informations, voir [Règles de configuration du pare-feu et des ports](troubleshoot.md#check-your-firewall-and-port-configuration-rules).

### <a name="allow-connections-from-iot-edge-devices"></a>Autoriser les connexions à partir d’appareils IoT Edge

Si votre configuration réseau exige d’autoriser explicitement les connexions effectuées à partir d’appareils IoT Edge, passez en revue la liste suivante de composants IoT Edge :

* **L’agent IoT Edge** ouvre une connexion AMQP/MQTT persistante à IoT Hub, éventuellement sur WebSockets.
* Le **hub IoT Edge** ouvre une seule connexion AMQP persistante ou plusieurs connexions MQTT à IoT Hub, éventuellement sur WebSockets.
* Le **démon IoT Edge** effectue des appels HTTPS intermittents à IoT Hub.

Dans les trois cas, le nom DNS respecte le modèle \*.azure-devices.net.

Par ailleurs, le **Moteur de conteneur** effectue des appels aux registres de conteneurs via HTTPS. Pour récupérer les images de conteneur du runtime IoT Edge, le nom DNS est mcr.microsoft.com. Le moteur de conteneur se connecte aux autres registres configurés dans le déploiement.

Cette liste de vérification est un point de départ pour les règles de pare-feu :

   | URL (\* = caractère générique) | Ports TCP sortants | Usage |
   | ----- | ----- | ----- |
   | mcr.microsoft.com  | 443 | Registre de conteneurs Microsoft |
   | global.azure-devices-provisioning.net  | 443 | Accès DPS (facultatif) |
   | \*.azurecr.io | 443 | Registres de conteneurs personnels et tiers |
   | \*.blob.core.windows.net | 443 | Télécharger les deltas d’image Azure Container Registry à partir du stockage Blob |
   | \*.azure-devices.net | 5671, 8883, 443 | Accès IoT Hub |
   | \*.docker.io  | 443 | Accès Docker Hub (facultatif) |

Certaines de ces règles de pare-feu sont héritées d’Azure Container Registry. Pour plus d’informations, consultez [Configurer des règles pour accéder à un registre de conteneurs Azure derrière un pare-feu](../container-registry/container-registry-firewall-access-rules.md).

> [!NOTE]
> Pour fournir un nom de domaine complet (FQDN) cohérent entre les points de terminaison REST et de données, à partir du **15 juin 2020**, le point de terminaison de données Registre de conteneurs Microsoft passe de `*.cdn.mscr.io` à `*.data.mcr.microsoft.com`.  
> Pour plus d’informations, consultez [Configuration des règles de pare-feu de client du Registre de conteneurs Microsoft](https://github.com/microsoft/containerregistry/blob/master/client-firewall-rules.md).

Si vous ne souhaitez pas configurer votre pare-feu pour autoriser l’accès aux registres de conteneurs publics, vous pouvez stocker les images dans votre registre de conteneurs privé, comme décrit dans [Stocker les conteneurs Runtime dans votre registre privé](#store-runtime-containers-in-your-private-registry).

### <a name="configure-communication-through-a-proxy"></a>Configurer la communication via un proxy

Si vos appareils sont destinés à être déployés sur un réseau qui utilise un serveur proxy, ils doivent être en mesure de communiquer via le proxy pour accéder à IoT Hub et aux registres de conteneurs. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md).

## <a name="solution-management"></a>Gestion de solution

* **Utile**
  * Configurer les journaux d’activité et les diagnostics
  * Prendre en compte les tests et les pipelines CI/CD

### <a name="set-up-logs-and-diagnostics"></a>Configurer les journaux d’activité et les diagnostics

Sous Linux, le démon IoT Edge utilise des journaux comme pilote de journalisation par défaut. Vous pouvez vous servir de l’outil en ligne de commande `journalctl` pour interroger les journaux d’activité du démon. Sous Windows, le démon IoT Edge utilise les diagnostics PowerShell. Interrogez les journaux d’activité du démon avec `Get-IoTEdgeLog`. Les modules IoT Edge utilisent le pilote par défaut JSON pour la journalisation.  

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Lorsque vous testez un déploiement IoT Edge, vous pouvez généralement accéder à vos appareils pour récupérer les journaux d’activité et résoudre les problèmes. Dans un scénario de déploiement, vous n’avez pas forcément cette option. Réfléchissez à la façon dont vous allez collecter des informations sur vos appareils en production. Il est possible d’utiliser un module de journalisation, par exemple, [logspout-loganalytics](https://github.com/veyalla/logspout-loganalytics), qui recueille des informations auprès des autres modules et les envoie vers le cloud. Vous pouvez également concevoir votre propre module de journalisation.

### <a name="place-limits-on-log-size"></a>Placer les limites de taille de journal

Par défaut, le moteur de conteneur Moby ne définit pas de limites de taille pour le journal de conteneur. Au fil du temps, cela peut amener l’appareil à se remplir de journaux et à manquer d’espace disque. Envisagez les options suivantes pour éviter cet écueil :

#### <a name="option-set-global-limits-that-apply-to-all-container-modules"></a>Option : Définir des limites globales qui s’appliquent à tous les modules de conteneur

Vous pouvez limiter la taille de tous les fichiers de journaux de conteneur dans les options de journal de moteur de conteneur. L’exemple suivant définit le pilote de journal `json-file` (recommandé) avec des limites de taille et de nombre de fichiers :

```JSON
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    }
}
```

Ajoutez ces informations dans un fichier nommé `daemon.json` et placez-le à l’emplacement approprié pour votre plateforme d’appareil.

| Plateforme | Emplacement |
| -------- | -------- |
| Linux | `/etc/docker/` |
| Windows | `C:\ProgramData\iotedge-moby\config\` |

Vous devez redémarrer le moteur de conteneur pour que les modifications entrent en vigueur.

#### <a name="option-adjust-log-settings-for-each-container-module"></a>Option : Ajuster les paramètres de journal pour chaque module de conteneur

Vous pouvez le faire dans **createOptions** au sein de chaque module. Par exemple :

```yml
"createOptions": {
    "HostConfig": {
        "LogConfig": {
            "Type": "json-file",
            "Config": {
                "max-size": "10m",
                "max-file": "3"
            }
        }
    }
}
```

#### <a name="additional-options-on-linux-systems"></a>Options supplémentaires sur les systèmes Linux

* Configurez le moteur de conteneur pour envoyer des journaux au [journal](https://docs.docker.com/config/containers/logging/journald/) `systemd` en définissant `journald` comme pilote de journalisation par défaut.

* Supprimez régulièrement les anciens journaux d’activité de votre appareil en installant un outil logrotate. Utilisez la spécification de fichier suivante :

   ```txt
   /var/lib/docker/containers/*/*-json.log{
       copytruncate
       daily
       rotate7
       delaycompress
       compress
       notifempty
       missingok
   }
   ```

### <a name="consider-tests-and-cicd-pipelines"></a>Prendre en compte les tests et les pipelines CI/CD

Pour maximiser l’efficacité du scénario de déploiement IoT Edge, intégrez votre déploiement de production dans vos pipelines de tests et CI/CD. Azure IoT Edge prend en charge plusieurs plateformes CI/CD, notamment Azure DevOps. Pour plus d’informations, voir [Intégration continue et déploiement continu sur Azure IoT Edge](how-to-continuous-integration-continuous-deployment.md).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [déploiement automatique IoT Edge](module-deployment-monitoring.md).
* Découvrez comment IoT Edge prend en charge [l’intégration continue et le déploiement continu](how-to-continuous-integration-continuous-deployment.md).
