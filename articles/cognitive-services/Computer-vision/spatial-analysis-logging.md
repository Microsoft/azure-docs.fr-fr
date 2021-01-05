---
title: Données de télémétrie et de journalisation pour les conteneurs d’analyse spatiale
titleSuffix: Azure Cognitive Services
description: L’analyse spatiale fournit à chaque conteneur des insights sur l’infrastructure de configuration commune, la journalisation et les paramètres de sécurité communs.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/11/2020
ms.author: aahi
ms.openlocfilehash: dd1b6d216f6225a13d86aa2435b5b1c807547ec3
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95014575"
---
# <a name="telemetry-and-troubleshooting"></a>Télémétrie et résolution des problèmes

L’analyse spatiale comprend un ensemble de fonctionnalités permettant de surveiller l’intégrité du système et d’aider à diagnostiquer les problèmes.

## <a name="enable-visualizations"></a>Activer les visualisations

Pour activer une visualisation des événements d’insights d’IA dans une image vidéo, vous devez utiliser la version `.debug` d’une [opération d’analyse spatiale](spatial-analysis-operations.md) sur un ordinateur de bureau. La visualisation n’est pas possible sur les appareils Azure Stack Edge. Quatre opérations de débogage sont disponibles.

Si votre appareil n’est pas un appareil Azure Stack Edge, modifiez le fichier de manifeste de déploiement pour [ordinateurs de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json) pour utiliser la valeur correcte pour la variable d’environnement `DISPLAY`. Elle doit correspondre à la variable `$DISPLAY` sur l’ordinateur hôte. Après la mise à jour du manifeste de déploiement, redéployez le conteneur.

Une fois le déploiement terminé, vous devrez peut-être copier le fichier `.Xauthority` de l’ordinateur hôte vers le conteneur, puis le redémarrer. Dans l’exemple ci-dessous, `peopleanalytics` est le nom du conteneur sur l’ordinateur hôte.

```bash
sudo docker cp $XAUTHORITY peopleanalytics:/root/.Xauthority
sudo docker stop peopleanalytics
sudo docker start peopleanalytics
xhost +
```


## <a name="collect-system-health-telemetry"></a>Collecter les données de télémétrie d’intégrité du système

Telegraf est une image open source qui fonctionne avec l’analyse spatiale, et est disponible dans le registre de conteneurs Microsoft. Elle prend les entrées suivantes et les envoie à Azure Monitor. Le module Telegraf peut être généré avec les entrées et sorties personnalisées souhaitées. La configuration du module Telegraf dans l’analyse spatiale fait partie du manifeste de déploiement (lien ci-dessous). Ce module est facultatif et peut être supprimé du manifeste si vous n’en avez pas besoin. 

Entrées : 
1. Métriques d’analyse spatiale
2. Métriques de disque
3. Métriques de l’UC
4. Métriques Docker
5. Métriques du GPU

Sorties :
1. Azure Monitor

Le module Telegraf d’analyse spatiale fourni publie toutes les données de télémétrie émises par le conteneur d’analyse spatiale dans Azure Monitor. Pour plus d’informations sur l’ajout d’Azure Monitor à votre abonnement, [cliquez ici](../../azure-monitor/overview.md).

Après avoir configuré Azure Monitor, vous devrez créer des informations d’identification qui permettent au module d’envoyer des données de télémétrie. Vous pouvez utiliser le portail Azure pour créer un principal de service ou utiliser la commande Azure CLI ci-dessous pour en créer un.

> [!NOTE] 
> Cette commande nécessite que vous disposiez des privilèges de propriétaire sur l’abonnement. 

```bash
# Find your Azure IoT Hub resource ID by running this command. The resource ID  should start with something like 
# "/subscriptions/b60d6458-1234-4be4-9885-c7e73af9ced8/resourceGroups/...”
az iot hub list

# Create a Service Principal with `Monitoring Metrics Publisher` role in the IoTHub resource:
# Save the output from this command. The values will be used in the deployment manifest. The password won't be shown again so make sure to write it down
az ad sp create-for-rbac --role="Monitoring Metrics Publisher" --name "<principal name>" --scopes="<resource ID of IoT Hub>"
```

Dans le manifeste de déploiement pour votre [appareil Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou un autre [ordinateur de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json), recherchez le module *telegraf* et remplacez les valeurs suivantes par les informations du principal de service de l’étape précédente, puis redéployez.

```json

"telegraf": { 
  "settings": {
  "image":   "mcr.microsoft.com/azure-cognitive-services/vision/spatial-analysis/telegraf:1.0",
  "createOptions":   "{\"HostConfig\":{\"Runtime\":\"nvidia\",\"NetworkMode\":\"azure-iot-edge\",\"Memory\":33554432,\"Binds\":[\"/var/run/docker.sock:/var/run/docker.sock\"]}}"
},
"type": "docker",
"env": {
    "AZURE_TENANT_ID": {
        "value": "<Tenant Id>"
    },
    "AZURE_CLIENT_ID": {
        "value": "Application Id"
    },
    "AZURE_CLIENT_SECRET": {
        "value": "<Password>"
    },
    "region": {
        "value": "<Region>"
    },
    "resource_id": {
        "value": "/subscriptions/{subscriptionId}/resourceGroups/{resoureGroupName}/providers/Microsoft.Devices/IotHubs/{IotHub}"
    },
...
```

Une fois le module Telegraf déployé, les métriques signalées sont accessibles par le biais du service Azure Monitor ou en sélectionnant **Surveillance** dans l’IoT Hub du portail Azure.

:::image type="content" source="./media/spatial-analysis/iot-hub-telemetry.png" alt-text="Rapport de télémétrie Azure Monitor":::

### <a name="system-health-events"></a>Événements d’intégrité du système

| Nom de l'événement | Description|
|------|---------|
|archon_exit    |Envoyé lorsqu’un utilisateur modifie l’état du module d’analyse spatiale de la valeur *en cours d’exécution* à *arrêté*.  |
|archon_error   |Envoyé lorsque l’un des processus à l’intérieur du conteneur plante. Il s'agit d'une erreur critique.  |
|InputRate  |Rythme auquel le graphique traite l’entrée vidéo. Signalé toutes les 5 minutes. | 
|OutputRate     |Rythme auquel le graphique génère des insights d’IA. Signalé toutes les 5 minutes. |
|archon_allGraphsStarted | Envoyé lorsque tous les graphiques ont fini de démarrer. |
|archon_configchange    | Envoyé lorsqu’une configuration de graphique a changé. |
|archon_graphCreationFailed     |Envoyé lorsque le graphique avec le `graphId` signalé ne parvient pas à démarrer. |
|archon_graphCreationSuccess    |Envoyé lorsque le graphique avec le `graphId` signalé démarre avec succès. |
|archon_graphCleanup    | Envoyé lorsque le graphique avec le `graphId` signalé se nettoie et se termine. |
|archon_graphHeartbeat  |Pulsation envoyée toutes les minutes pour chaque graphique d’une compétence. |
|archon_apiKeyAuthFail |Envoyé lorsque la clé de ressource Vision par ordinateur ne parvient pas à authentifier le conteneur pendant plus de 24 heures, pour les raisons suivantes : Hors quota, non valide, hors connexion. |
|VideoIngesterHeartbeat     |Envoyé toutes les heures pour indiquer que la vidéo est diffusée à partir de la source vidéo, avec le nombre d’erreurs au cours de cette heure. Signalé pour chaque graphique. |
|VideoIngesterState | Les rapports *arrêtés* ou *démarrés* pour la diffusion vidéo.  Signalé pour chaque graphique. |

##  <a name="troubleshooting-an-iot-edge-device"></a>Résolution des problèmes d’un appareil IoT Edge

Vous pouvez utiliser l’outil en ligne de commande `iotedge` pour vérifier l’état et les journaux des modules en cours d’exécution. Exemple :
* `iotedge list`: Signale la liste des modules en cours d’exécution. 
  Vous pouvez rechercher d’autres erreurs avec `iotedge logs edgeAgent`. Si `iotedge` est bloqué, vous pouvez essayer de le redémarrer avec `iotedge restart edgeAgent`
* `iotedge logs <module-name>`
* `iotedge restart <module-name>` pour redémarrer un module spécifique 

## <a name="collect-log-files-with-the-diagnostics-container"></a>Collecter les fichiers journaux avec le conteneur de diagnostics

L’analyse spatiale génère des journaux de débogage Docker que vous pouvez utiliser pour diagnostiquer les problèmes d’exécution ou inclure dans les tickets de support. Le module de diagnostics d’analyse spatiale est disponible au téléchargement dans le Registre de conteneurs Microsoft. Dans le fichier de déploiement du manifeste de votre [appareil Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou autre [ordinateur de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json), recherchez le module *diagnostics*.

Dans la section « env », ajoutez la configuration suivante :

```json
"diagnostics": {  
  "settings": {
  "image":   "mcr.microsoft.com/azure-cognitive-services/vision/spatial-analysis/diagnostics:1.0",
  "createOptions":   "{\"HostConfig\":{\"Mounts\":[{\"Target\":\"/usr/bin/docker\",\"Source\":\"/home/data/docker\",\"Type\":\"bind\"},{\"Target\":\"/var/run\",\"Source\":\"/run\",\"Type\":\"bind\"}],\"LogConfig\":{\"Config\":{\"max-size\":\"500m\"}}}}"
  }
```    

Pour optimiser les journaux chargés sur un point de terminaison distant, comme le stockage d’objets Blob Azure, nous vous recommandons de maintenir une petite taille de fichier. Consultez l’exemple ci-dessous pour connaître la configuration recommandée des journaux Docker.

```json
{
    "HostConfig": {
        "LogConfig": {
            "Config": {
                "max-size": "500m",
                "max-file": "1000"
            }
        }
    }
}
```

### <a name="configure-the-log-level"></a>Configurer le niveau de journalisation

La configuration du niveau de journalisation vous permet de contrôler le niveau de détail des journaux générés. Les niveaux de journalisation pris en charge sont les suivants : `none`, `verbose`, `info`, `warning` et `error`. Le niveau de détail de journalisation par défaut pour les nœuds et la plateforme est `info`. 

Les niveaux de journalisation peuvent être modifiés globalement en affectant l’une des valeurs autorisées à la variable d’environnement `ARCHON_LOG_LEVEL`.
Vous pouvez également les définir par le biais du document de jumeau de module IoT Edge à l’échelle globale, pour toutes les compétences déployées ou pour chaque compétence spécifique en définissant les valeurs pour `platformLogLevel` et `nodeLogLevel` comme indiqué ci-dessous.

```json
{
    "version": 1,
    "properties": {
        "desired": {
            "globalSettings": {
                "platformLogLevel": "verbose"
            },
            "graphs": {
                "samplegraph": {
                    "nodeLogLevel": "verbose",
                    "platformLogLevel": "verbose"
                }
            }
        }
    }
}
```

### <a name="collecting-logs"></a>collecte des journaux d’activité

> [!NOTE]
> Le module `diagnostics` n’affecte pas le contenu de la journalisation. Il sert uniquement pour la collecte, le filtrage et le téléchargement des journaux existants.
> Vous devez disposer de l’API Docker version 1.40 ou ultérieure pour utiliser ce module.

L’exemple de fichier manifeste de déploiement pour votre [appareil Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou autre [ordinateur de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json) comprend un module nommé `diagnostics` qui collecte et charge les journaux. Ce module est désactivé par défaut et doit être activé via la configuration du module IoT Edge lorsque vous avez besoin d’accéder aux journaux. 

La collecte de `diagnostics` est effectuée à la demande et contrôlée via une méthode directe IoT Edge, et elle peut envoyer des journaux à un stockage d’objets Blob Azure.

### <a name="configure-diagnostics-upload-targets"></a>Configurer les cibles de chargement des diagnostics

Dans le portail IoT Edge, sélectionnez votre appareil, puis le module **Diagnostics**. Dans l’exemple de fichier manifeste de déploiement pour votre [appareil Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou autre [ordinateur de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json), recherchez la section **Variables d’environnement** pour diagnostics, nommée `env`, puis ajoutez les informations suivantes :

**Configurer le chargement vers le stockage d’objets Blob Azure**

1. Créez votre propre compte de stockage d’objets Blob Azure si vous ne l’avez pas déjà fait.
2. Récupérez la **chaîne de connexion** pour votre compte de stockage à partir du portail Azure. Elle se trouve dans **Clés d’accès**.
3. Les journaux d’analyse spatiale sont téléchargés automatiquement dans un conteneur de stockage d’objets Blob nommé *rtcvlogs* avec le format de nom de fichier suivant : `{CONTAINER_NAME}/{START_TIME}-{END_TIME}-{QUERY_TIME}.log`.

```json
"env":{
    "IOTEDGE_WORKLOADURI":"fd://iotedge.socket",
    "AZURE_STORAGE_CONNECTION_STRING":"XXXXXX",   //from the Azure Blob Storage account
    "ARCHON_LOG_LEVEL":"info"
}
```

### <a name="uploading-spatial-analysis-logs"></a>Téléchargement des journaux d’analyse spatiale

Les journaux sont téléchargés à la demande à l’aide de la méthode IoT Edge `getRTCVLogs`, dans le module `diagnostics`. 


1. Accédez à la page du portail IoT Hub, sélectionnez **Appareils Edge**, puis sélectionnez votre appareil et votre module de diagnostic. 
2. Accédez à la page de détails du module et cliquez sur l’onglet **_Méthode directe_* _.
3. Entrez `getRTCVLogs` pour Nom de la méthode et une chaîne de format json dans la charge utile. Vous pouvez entrer `{}`, qui est une charge utile vide. 
4. Définissez les délais d’expiration de connexion et de méthode, puis cliquez sur _*Appeler la méthode**.
5. Sélectionnez votre conteneur cible, puis créez une chaîne json de charge utile à l’aide des paramètres décrits dans la section **Syntaxe de journalisation**. Cliquez sur **Appeler la méthode** pour effectuer la requête.

>[!NOTE]
> L’appel de la méthode `getRTCVLogs` avec une charge utile vide retourne une liste de tous les conteneurs déployés sur l’appareil. Le nom de la méthode respecte la casse. Vous obtiendrez une erreur 501 si un nom de méthode incorrect est spécifié.

:::image type="content" source="./media/spatial-analysis/direct-log-collection.png" alt-text="Appel de la méthode getRTCVLogs":::
![Page Méthode directe getRTCVLogs](./media/spatial-analysis/direct-log-collection.png)

 
### <a name="logging-syntax"></a>Syntaxe de journalisation

Le tableau ci-dessous répertorie les paramètres que vous pouvez utiliser lors de l’interrogation des journaux.

| Mot clé | Description | Valeur par défaut |
|--|--|--|
| StartTime | Heure de début des journaux souhaités, en temps universel coordonné (UTC). | `-1`, le début de l’exécution du conteneur. Lorsque vous utilisez `[-1.-1]` comme intervalle de temps, l’API retourne les journaux de la dernière heure.|
| EndTime | Heure de fin des journaux souhaités, en temps universel coordonné (UTC). | `-1`, l’heure actuelle. Lorsque vous utilisez l’intervalle de temps `[-1.-1]`, l’API retourne les journaux de la dernière heure. |
| ContainerId | Conteneur cible pour l’extraction des journaux.| `null`, lorsqu’il n’y a pas d’ID de conteneur. L’API retourne toutes les informations sur les conteneurs disponibles avec les ID.|
| DoPost | Effectuez l’opération de chargement. Quand cette propriété a la valeur `false`, elle effectue l’opération demandée et retourne la taille du chargement sans effectuer le chargement. Lorsqu’elle est définie sur `true`, elle lance le chargement asynchrone des journaux sélectionnés | `false`, ne pas télécharger.|
| Limitation | Indiquez le nombre de lignes de journaux à charger par lot | `1000`, utilisez ce paramètre pour ajuster la vitesse de publication. |
| Filtres | Filtrez les journaux à charger | `null`, les filtres peuvent être spécifiés en tant que paires clé/valeur en fonction de la structure des journaux d’analyse spatiale : `[UTC, LocalTime, LOGLEVEL,PID, CLASS, DATA]`. Par exemple : `{"TimeFilter":[-1,1573255761112]}, {"TimeFilter":[-1,1573255761112]}, {"CLASS":["myNode"]`|

Le tableau suivant répertorie les attributs de la réponse à la requête.

| Mot clé | Description|
|--|--|
|DoPost| *true* ou *false*. Indique si les journaux ont été chargés ou non. Lorsque vous choisissez de ne pas charger les journaux, l’API retourne des informations ***de façon synchrone** _. Lorsque vous choisissez de charger les journaux, l’API retourne 200 si la demande est valide, et démarre le chargement des journaux de façon _*_asynchrone_*_.|
|TimeFilter| Filtre de temps appliqué aux journaux.|
|ValueFilters| Filtres de mots clés appliqués aux journaux. |
|TimeStamp| Heure de début de l’exécution de la méthode. |
|ContainerId| ID du conteneur cible. |
|FetchCounter| Nombre total de lignes de journal. |
|FetchSizeInByte| Quantité totale de données de journal en octets. |
|MatchCounter| Nombre valide de lignes de journal. |
|MatchSizeInByte| Quantité valide de données de journal en octets. |
|FilterCount| Nombre total de lignes de journal après application du filtre. |
|FilterSizeInByte| Quantité totale de données du journal en octets après application du filtre. |
|FetchLogsDurationInMiliSec| Durée de l’opération de récupération. |
|PaseLogsDurationInMiliSec| Durée de l’opération de filtrage. |
|PostLogsDurationInMiliSec| Durée post-opération. |

#### <a name="example-request"></a>Exemple de requête 

```json
{
    "StartTime": -1,
    "EndTime": -1,
    "ContainerId": "5fa17e4d8056e8d16a5a998318716a77becc01b36fde25b3de9fde98a64bf29b",
    "DoPost": false,
    "Filters": null
}
```

#### <a name="example-response"></a>Exemple de réponse 

```json
{
    "status": 200,
    "payload": {
        "DoPost": false,
        "TimeFilter": [-1, 1581310339411],
        "ValueFilters": {},
        "Metas": {
            "TimeStamp": "2020-02-10T04:52:19.4365389+00:00",
            "ContainerId": "5fa17e4d8056e8d16a5a998318716a77becc01b36fde25b3de9fde98a64bf29b",
            "FetchCounter": 61,
            "FetchSizeInByte": 20470,
            "MatchCounter": 61,
            "MatchSizeInByte": 20470,
            "FilterCount": 61,
            "FilterSizeInByte": 20470,
            "FetchLogsDurationInMiliSec": 0,
            "PaseLogsDurationInMiliSec": 0,
            "PostLogsDurationInMiliSec": 0
        }
    }
}
```

Vérifiez les lignes, les heures et les tailles du journal extrait. Si ces paramètres semblent corrects, remplacez _*_DoPost_*_ par `true` pour envoyer les journaux avec les mêmes filtres aux destinations. 

Vous pouvez exporter les journaux à partir du stockage d’objets Blob Azure lors de la résolution des problèmes. 

## <a name="common-issues"></a>Problèmes courants

Si vous voyez le message suivant dans les journaux de module, cela peut signifier que votre abonnement Azure doit être approuvé : 

«Le conteneur n’est pas dans un état valide. La validation de l’abonnement a échoué avec l’état « Incompatibilité ». La clé API n’est pas destinée au type de conteneur donné. »

Pour plus d’informations, consultez [Demande d’approbation pour l’exécution du conteneur](spatial-analysis-container.md#request-approval-to-run-the-container).

## <a name="troubleshooting-the-azure-stack-edge-device"></a>Résolution des problèmes avec les appareils Azure Stack Edge

La section suivante est fournie pour vous aider à déboguer et vérifier l’état de votre appareil Azure Stack Edge.

### <a name="access-the-kubernetes-api-endpoint"></a>Accédez au point de terminaison de l’API Kubernetes. 

1. Dans l’interface utilisateur locale de votre appareil, accédez à la page _ *Appareils**. 
2. Sous **Points de terminaison de l’appareil**, copiez le point de terminaison du service d’API Kubernetes. Ce point de terminaison est une chaîne au format suivant : `https://compute..[device-IP-address]`.
3. Enregistrez la chaîne de point de terminaison. Vous allez l’utiliser ultérieurement lors de la configuration de `kubectl` pour accéder au cluster Kubernetes.

### <a name="connect-to-powershell-interface"></a>Se connecter à l’interface PowerShell

Connexion à distance depuis un client Windows. Une fois le cluster Kubernetes créé, vous pouvez gérer les applications à l’aide de ce cluster. Vous devez vous connecter à l’interface PowerShell de l’appareil. Selon le système d’exploitation du client, les procédures permettant de se connecter à distance à l’appareil peuvent être différentes. Les étapes suivantes portent sur un client Windows exécutant PowerShell.

> [!TIP]
> * Avant de commencer, assurez-vous que votre client Windows exécute Windows PowerShell 5.0 ou une version ultérieure.
> * PowerShell est également [disponible sur Linux](/powershell/scripting/install/installing-powershell-core-on-linux).

1. Démarrez une session Windows PowerShell en tant qu’administrateur. 
    1. Assurez-vous que le service Windows Remote Management est en cours d’exécution sur votre client. À l’invite de commandes, tapez `winrm quickconfig`.

2. Attribuez une variable à l’adresse IP de l’appareil. Par exemple : `$ip = "<device-ip-address>"`.

3. Utilisez la commande suivante pour ajouter l’adresse IP de l’appareil à la liste des hôtes approuvés du client. 

    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force
    ```
 
4. Démarrez une session Windows PowerShell sur l’appareil. 

    ```powershell
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell
    ```

5. Indiquez le mot de passe lorsque vous y êtes invité. Utilisez le mot de passe vous permettant de vous connecter à l’interface web locale. Le mot de passe de l’interface web locale par défaut est `Password1`. 

### <a name="access-the-kubernetes-cluster"></a>Accéder au cluster Kubernetes

Une fois le cluster Kubernetes créé, vous pouvez utiliser l’outil de ligne de commande `kubectl` pour accéder au cluster.

1. Créez un espace de noms. 

    ```powershell
    New-HcsKubernetesNamespace -Namespace
    ```

2. Créez un utilisateur et obtenez un fichier config. Cette commande génère des informations de configuration pour le cluster Kubernetes. Copiez ces informations et enregistrez-les dans un fichier nommé *config*. N’enregistrez pas le fichier avec une extension de fichier.
    
    ```powershell
    New-HcsKubernetesUser -UserName
    ```

3. Ajoutez le fichier *config* au dossier *.kube* dans votre profil utilisateur sur l’ordinateur local.   

4. Associez l’espace de noms à l’utilisateur que vous avez créé.

    ```powershell
    Grant-HcsKubernetesNamespaceAccess -Namespace -UserName
    ```

5. Installez `kubectl` sur votre client Windows à l’aide de la commande suivante :
    
    ```powershell
    curl https://storage.googleapis.com/kubernetesrelease/release/v1.15.2/bin/windows/amd64/kubectl.exe -O kubectl.exe
    ```

6. Ajoutez une entrée DNS au fichier hosts sur votre système. 
    1. Exécutez le Bloc-notes en tant qu’administrateur et ouvrez le fichier *hosts* situé à l’emplacement `C:\windows\system32\drivers\etc\hosts`. 
    2. Créez une entrée dans le fichier hosts avec l’adresse IP et le domaine DNS que vous avez obtenus à partir de la page **Appareil** dans l’interface utilisateur locale. Le point de terminaison que vous devez utiliser ressemble à ce qui suit : `https://compute.asedevice.microsoftdatabox.com/10.100.10.10`.

7. Vérifiez que vous pouvez vous connecter aux pods Kubernetes.

    ```powershell
    kubectl get pods -n "iotedge"
    ```

Pour accéder aux journaux de conteneur, exécutez la commande suivante :

```powershell
kubectl logs <pod-name> -n <namespace> --all-containers
```

### <a name="useful-commands"></a>Commandes utiles

|Commande  |Description  |
|---------|---------|
|`Get-HcsKubernetesUserConfig -AseUser`     | Génère un fichier de configuration Kubernetes. Lors de l’utilisation de la commande, copiez les informations dans un fichier nommé *config*. N’enregistrez pas le fichier avec une extension.        |
| `Get-HcsApplianceInfo` | Retourne des informations sur votre appareil. |
| `Enable-HcsSupportAccess` | Génère des informations d’identification d’accès pour démarrer une session de support. |

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application web de comptage de personnes](spatial-analysis-web-app.md)
* [Configurer les opérations d’analyse spatiale](./spatial-analysis-operations.md)
* [Guide de positionnement de la caméra](spatial-analysis-camera-placement.md)
* [Guide de positionnement de zone et de ligne](spatial-analysis-zone-line-placement.md)