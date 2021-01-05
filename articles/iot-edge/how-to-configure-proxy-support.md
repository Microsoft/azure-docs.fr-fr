---
title: Configurer des appareils pour des proxys réseau - Azure IoT Edge | Microsoft Docs
description: Découvrez comment configurer le runtime Azure IoT Edge et les modules IoT Edge accessibles sur Internet pour communiquer via un serveur proxy.
author: kgremban
ms.author: kgremban
ms.date: 09/03/2020
ms.topic: how-to
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- contperf-fy21q1
ms.openlocfilehash: fb7cb0638ca86ea736749e6fb35e2295128162aa
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032981"
---
# <a name="configure-an-iot-edge-device-to-communicate-through-a-proxy-server"></a>Configurer un appareil IoT Edge pour communiquer via un serveur proxy

Les appareils IoT Edge envoient des requêtes HTTPS pour communiquer avec IoT Hub. Si votre appareil est connecté à un réseau qui utilise un serveur proxy, vous devez configurer le runtime IoT Edge pour communiquer via le serveur. Les serveurs proxy peuvent également impacter des modules IoT Edge individuels s’ils effectuent des requêtes HTTP ou HTTPS qui ne sont pas routées via le hub IoT Edge.

Cet article vous montre la procédure en quatre étapes à suivre pour configurer et gérer un appareil IoT Edge derrière un serveur proxy :

1. [**Installer le runtime IoT Edge sur votre appareil**](#install-the-runtime-through-a-proxy)

   Les scripts d’installation d’IoT Edge extraient les packages et les fichiers à partir d’Internet. Votre appareil doit donc communiquer via le serveur proxy pour envoyer ces demandes. Pour les appareils Windows, le script d’installation fournit également une option d’installation hors connexion.

   Vous n’effectuerez la configuration décrite dans cette étape qu’une seule fois lors de la première installation de votre appareil IoT Edge. Vous devez disposer des mêmes connexions lorsque vous mettez à jour le runtime IoT Edge.

2. [**Configurer le démon Docker et le démon IoT Edge sur votre appareil**](#configure-the-daemons)

   IoT Edge utilise deux démons sur l’appareil et les deux doivent envoyer des requêtes web via le serveur proxy. Le démon IoT Edge est responsable des communications avec IoT Hub. Le démon Moby est responsable de la gestion des conteneurs. Il communique donc avec les registres de conteneur.

   Vous n’effectuerez la configuration décrite dans cette étape qu’une seule fois lors de la première installation de votre appareil IoT Edge.

3. [**Configurer les propriétés de l’agent IoT Edge dans le fichier config.yaml de votre appareil**](#configure-the-iot-edge-agent)

   Le démon IoT Edge démarre initialement le module edgeAgent. Ensuite, le module edgeAgent récupère le manifeste de déploiement à partir de IoT Hub et démarre tous les autres modules. Afin que l’agent IoT Edge puisse établir la connexion initiale à IoT Hub, configurez manuellement les variables d’environnement du module edgeAgent directement sur l’appareil. Une fois la première connexion effectuée, vous pouvez configurer le module edgeAgent à distance.

   Vous n’effectuerez la configuration décrite dans cette étape qu’une seule fois lors de la première installation de votre appareil IoT Edge.

4. [**Pour tous vos futurs déploiements de module, définissez des variables d’environnement pour chaque module communiquant via le proxy**](#configure-deployment-manifests)

   Une fois votre appareil IoT Edge configuré et connecté à IoT Hub via le serveur proxy, vous devez maintenir la connexion de tous les futurs déploiements de module.

   Cette étape est un processus continu à effectuer à distance afin que chaque nouvelle mise à jour de module ou de déploiement conserve la capacité de l’appareil à communiquer via le serveur proxy.

## <a name="know-your-proxy-url"></a>Identifier l’URL de votre proxy

Avant de commencer l’une des étapes de cet article, vous devez connaître l’URL de votre proxy.

Les URL de proxy se présentent sous ce format : **protocol**://**proxy_host**:**proxy_port**.

* Le paramètre **protocol** a la valeur HTTP ou HTTPS. Le démon Docker peut utiliser chacun de ces protocoles, selon vos paramètres de registre de conteneurs, mais les conteneurs de runtime et le démon IoT Edge doivent toujours utiliser HTTP pour se connecter au proxy.

* Le paramètre **proxy_host** est une adresse du serveur proxy. Si votre serveur proxy requiert une authentification, vous pouvez fournir vos informations d’identification à l’aide de l’hôte de proxy au format suivant : **user**:**password**\@**proxy_host**.

* **proxy_port** représente le port réseau à partir duquel le proxy répond au trafic réseau.

## <a name="install-the-runtime-through-a-proxy"></a>Installer le runtime via un proxy

Si vous exécutez votre appareil IoT Edge sur Windows ou Linux, vous devez accéder aux packages d’installation via le serveur proxy. Pour installer le runtime IoT Edge via un serveur proxy, suivez les étapes correspondant à votre système d’exploitation.

### <a name="linux-devices"></a>Appareils Linux

Si vous installez le runtime IoT Edge sur un appareil Linux, configurez le Gestionnaire de package de sorte à passer par votre serveur proxy pour accéder au package d’installation. Par exemple, [Configurez apt-get pour utiliser un proxy http](https://help.ubuntu.com/community/AptGet/Howto/#Setting_up_apt-get_to_use_a_http-proxy). Une fois que votre gestionnaire de package est configuré, suivez les instructions dans [Installer le runtime Azure IoT Edge](how-to-install-iot-edge.md) comme d’habitude.

### <a name="windows-devices"></a>Appareils Windows

Si vous installez le runtime IoT Edge sur un appareil Windows, vous devez passer deux fois par le serveur proxy. La première connexion télécharge le fichier de script du programme d’installation et la seconde survient pendant l’installation afin de télécharger les composants nécessaires. Vous pouvez configurer les informations de proxy dans les paramètres Windows ou inclure vos informations de proxy directement dans les commandes PowerShell.

Les étapes suivantes illustrent un exemple d’installation sous Windows qui utilise l’argument `-proxy` :

1. La commande Invoke-WebRequest a besoin des informations de proxy pour accéder au script du programme d’installation. Ensuite, la commande Deploy-IoTEdge a besoin des informations de proxy pour télécharger les fichiers d’installation.

   ```powershell
   . {Invoke-WebRequest -proxy <proxy URL> -useb aka.ms/iotedge-win} | Invoke-Expression; Deploy-IoTEdge -proxy <proxy URL>
   ```

2. La commande Initialize-IoTEdge n’a pas besoin de transiter par le serveur proxy, alors seule la commande Invoke-WebRequest doit disposer des informations de proxy lors de la seconde étape.

   ```powershell
   . {Invoke-WebRequest -proxy <proxy URL> -useb aka.ms/iotedge-win} | Invoke-Expression; Initialize-IoTEdge
   ```

Si vous avez des informations d’identification complexes pour le serveur proxy qui ne peuvent pas être incluses dans l’URL, utilisez le paramètre `-ProxyCredential` dans `-InvokeWebRequestParameters`. Par exemple,

```powershell
$proxyCredential = (Get-Credential).GetNetworkCredential()
. {Invoke-WebRequest -proxy <proxy URL> -ProxyCredential $proxyCredential -useb aka.ms/iotedge-win} | Invoke-Expression; `
Deploy-IoTEdge -InvokeWebRequestParameters @{ '-Proxy' = '<proxy URL>'; '-ProxyCredential' = $proxyCredential }
```

Pour plus d’informations sur les paramètres de proxy, consultez [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest). Pour plus d’informations sur les paramètres d’installation Windows, consultez [Scripts PowerShell pour IoT Edge sous Windows](reference-windows-scripts.md).

## <a name="configure-the-daemons"></a>Configurer les démons

IoT Edge s’appuie sur deux démons s’exécutant sur l’appareil IoT Edge. Le démon Moby effectue des requêtes web pour extraire les images conteneur des registres de conteneurs. Le démon IoT Edge effectue des requêtes web pour communiquer avec IoT Hub.

Les démons Moby et IoT Edge doivent être configurés afin d’utiliser le serveur proxy pour les fonctionnalités en continu de l’appareil. Cette étape s’effectue sur l’appareil IoT Edge pendant la première installation de l’appareil.

### <a name="moby-daemon"></a>Démon Moby

Étant donné que Moby est basé sur Docker, consultez la documentation sur Docker pour savoir comment configurer le démon Moby avec des variables d’environnement. La plupart des registres de conteneurs (y compris les registres de conteneurs DockerHub et Azure) prennent en charge les requêtes HTTPS. Vous devez donc définir le paramètre **HTTPS_PROXY**. Si vous tirez des images d’un registre qui ne prend pas en charge le protocole TLS, vous devez définir le paramètre **HTTP_PROXY**.

Choisissez l’article correspondant au système d’exploitation de votre appareil IoT Edge :

* [Configurez le démon Docker sur Linux](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) Le démon Mobyx garde le nom Docker sur les appareils Linux.
* [Configurez le démon Docker sur Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon#proxy-configuration) Le démon Moby s’appelle iotedge-moby sur les appareils Windows. Les noms sont différents, car il est possible d’exécuter simultanément Docker Desktop et Moby sur un appareil Windows.

### <a name="iot-edge-daemon"></a>Démon IoT Edge

Le démon IoT Edge est configuré de manière similaire au démon Moby. Suivez les étapes ci-dessous afin de définir une variable d’environnement pour le service, selon votre système d’exploitation.

Le démon IoT Edge utilise toujours le protocole HTTPS pour envoyer des demandes à IoT Hub.

#### <a name="linux"></a>Linux

Ouvrez un éditeur dans le terminal pour configurer le démon IoT Edge.

```bash
sudo systemctl edit iotedge
```

Entrez le texte suivant, en remplaçant **\<proxy URL>** par l’adresse et le port de votre serveur proxy. Ensuite, enregistrez et quittez.

```ini
[Service]
Environment="https_proxy=<proxy URL>"
```

Actualisez Service Manager afin de charger la nouvelle configuration pour IoT Edge.

```bash
sudo systemctl daemon-reload
```

Redémarrez IoT Edge pour que les modifications soient prises en compte.

```bash
sudo systemctl restart iotedge
```

Vérifiez que votre variable d’environnement a été créée et que la nouvelle configuration a été chargée.

```bash
systemctl show --property=Environment iotedge
```

#### <a name="windows"></a>Windows

Ouvrez une fenêtre PowerShell en tant qu’administrateur et exécutez la commande suivante pour modifier le registre avec la nouvelle variable d’environnement. Remplacez **\<proxy url>** par l’adresse et le port de votre serveur proxy.

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Services\iotedge /v Environment /t REG_MULTI_SZ /d https_proxy=<proxy URL>
```

Redémarrez IoT Edge pour que les modifications soient prises en compte.

```powershell
Restart-Service iotedge
```

## <a name="configure-the-iot-edge-agent"></a>Configurer l’agent IoT Edge

L’agent IoT Edge est le premier module démarré sur un appareil IoT Edge. Il est démarré pour la première fois en fonction des informations contenues dans le fichier config.yaml sur IoT Edge. L’agent IoT Edge se connecte ensuite à IoT Hub pour récupérer les manifestes de déploiement, qui déclarent les autres modules devant être déployés sur l’appareil.

Cette étape s’effectue une seule fois sur l’appareil IoT Edge pendant la première installation de l’appareil.

1. Ouvrez le fichier config.yaml sur votre appareil IoT Edge. Sur les systèmes Linux, ce fichier se trouve dans **/etc/iotedge/config.yaml**. Sur les systèmes Windows, ce fichier se trouve dans **C:\ProgramData\iotedge\config.yaml**. Le fichier de configuration étant protégé, vous devez avoir des privilèges Administrateur pour y accéder. Sur les systèmes Linux, utilisez la commande `sudo` avant d’ouvrir le fichier dans votre éditeur de texte habituel. Sur Windows, ouvrez un éditeur de texte, tel que le bloc-notes, en tant qu’administrateur, puis ouvrir le fichier.

2. Dans le fichier config.yaml, recherchez la section **Edge Agent module spec**. La définition de l’agent IoT Edge inclut un paramètre **env** où vous pouvez ajouter des variables d’environnement.

3. Supprimez les accolades servant d’espaces réservés pour le paramètre env, puis ajoutez la nouvelle variable sur une nouvelle ligne. N’oubliez pas que les mises en retrait dans YAML sont de deux espaces.

   ```yaml
   https_proxy: "<proxy URL>"
   ```

4. Le runtime IoT Edge utilise AMQP par défaut pour communiquer avec IoT Hub. Certains serveurs proxy bloquent les ports AMQP. Si c’est le cas, vous devez également configurer edgeAgent pour utiliser AMQP sur WebSocket. Ajoutez une deuxième variable d’environnement.

   ```yaml
   UpstreamProtocol: "AmqpWs"
   ```

   ![Définition d’edgeAgent avec les variables d’environnement](./media/how-to-configure-proxy-support/edgeagent-edited.png)

5. Enregistrez les modifications dans config.yaml et fermez l’éditeur. Redémarrez IoT Edge pour que les modifications soient prises en compte.

   * Linux :

      ```bash
      sudo systemctl restart iotedge
      ```

   * Windows :

      ```powershell
      Restart-Service iotedge
      ```

## <a name="configure-deployment-manifests"></a>Configurer les manifestes de déploiement  

Une fois que vous avez configuré votre appareil IoT Edge pour communiquer via votre serveur proxy, vous devez également déclarer la variable d’environnement HTTPS_PROXY dans les manifestes de déploiement que vous allez utiliser. Vous pouvez modifier les manifestes de déploiement à l’aide de l’Assistant du portail Azure ou en modifiant un fichier JSON de déploiement de manifeste.

Vous devez toujours configurer les deux modules de runtime, edgeAgent et edgeHub, pour communiquer via le serveur proxy afin qu’ils puissent conserver une connexion avec IoT Hub. Si vous supprimez les informations de proxy du module edgeAgent, vous ne pouvez rétablir la connexion qu’en modifiant le fichier config.yaml sur l’appareil, comme décrit dans la section précédente.

Outre les modules edgeAgent et edgeHub, d’autres modules peuvent nécessiter la configuration du proxy. Les modules qui doivent accéder aux ressources Azure en plus d’IoT Hub, comme le stockage Blob, et pour lesquels la variable HTTPS_PROXY doit être spécifiée dans le fichier manifeste de déploiement.

La procédure suivante s’applique tout au long de la durée de vie de l’appareil IoT Edge.

### <a name="azure-portal"></a>Portail Azure

Quand vous créez des déploiements sur des appareils IoT Edge à l’aide de l’Assistant **Définir des modules**, chaque module a une section **Variables d’environnement** dans laquelle vous pouvez configurer des connexions au serveur proxy.

Pour configurer les modules de l’agent IoT Edge et du hub IoT Edge, sélectionnez **Paramètres d'exécution** à la première étape de l’Assistant.

![Configurer les paramètres avancés du runtime Edge](./media/how-to-configure-proxy-support/configure-runtime.png)

Ajoutez la variable d’environnement **https_proxy** aux définitions du module Agent IoT Edge et du module Hub IoT Edge. Si vous avez inclus la variable d’environnement **UpstreamProtocol** dans le fichier config.yaml sur votre appareil IoT Edge, ajoutez-la également à la définition du module de l’agent IoT Edge.

![Définir la variable d’environnement https_proxy](./media/how-to-configure-proxy-support/edgehub-environmentvar.png)

Tous les autres modules que vous ajoutez à un manifeste de déploiement suivent le même modèle.

### <a name="json-deployment-manifest-files"></a>Fichiers de manifeste de déploiement JSON

Quand vous créez des déploiements sur des appareils IoT Edge en utilisant des modèles dans Visual Studio Code ou en créant manuellement des fichiers JSON, vous pouvez ajouter les variables d’environnement directement à chaque définition de module.

Utilisez le format JSON suivant :

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    }
}
```

Après l’ajout des variables d’environnement, votre définition de module doit ressembler à l’exemple edgeHub suivant :

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": ""
    },
    "env": {
        "https_proxy": {
            "value": "http://proxy.example.com:3128"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

Si vous avez inclus la variable d’environnement **UpstreamProtocol** dans le fichier config.yaml sur votre appareil IoT Edge, ajoutez-la également à la définition du module de l’agent IoT Edge.

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    },
    "UpstreamProtocol": {
        "value": "AmqpWs"
    }
}
```

## <a name="working-with-traffic-inspecting-proxies"></a>Utilisation de proxys d’inspection du trafic

Si le proxy que vous essayez d’utiliser effectue l’inspection du trafic sur les connexions sécurisées par TLS, il est important de noter que l’authentification par certificats X.509 ne fonctionne pas. IoT Edge établit un canal TLS chiffré de bout en bout avec le certificat et la clé fournis. Si ce canal est interrompu pour l’inspection du trafic, le proxy ne peut pas rétablir le canal avec les informations d’identification appropriées, et IoT Hub et le service de provisionnement des appareils IoT Hub retournent une erreur `Unauthorized`.

Pour utiliser un proxy qui effectue l’inspection du trafic, vous devez utiliser l’authentification par signature d’accès partagé ou faire ajouter IoT Hub et le service de provisionnement d’appareils IoT Hub à une liste d’autorisation pour éviter l’inspection.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les rôles du [runtime IoT Edge](iot-edge-runtime.md)

Résoudre les erreurs d’installation et de configuration avec [Problèmes courants et résolutions pour Azure IoT Edge](troubleshoot.md)
