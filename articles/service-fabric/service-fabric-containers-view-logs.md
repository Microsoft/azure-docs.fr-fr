---
title: Afficher des journaux d’activité de conteneurs dans Azure Service Fabric
description: Décrit comment afficher des fichiers journaux d’activité de conteneurs de services Service Fabric en cours d’exécution à l’aide de Service Fabric Explorer.
ms.topic: conceptual
ms.date: 05/15/2018
ms.openlocfilehash: c47a408b272f95dbfcf3d791c644bfeb52254a72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75458186"
---
# <a name="view-logs-for-a-service-fabric-container-service"></a>Afficher les fichiers journaux d’activité d’un service de conteneurs Service Fabric
Azure Service Fabric, qui est un orchestrateur de conteneurs, prend en charge les [conteneurs Linux et Windows](service-fabric-containers-overview.md).  Cet article décrit comment afficher les fichiers journaux d’activité de conteneurs d’un service de conteneurs en cours d’exécution ou d’un conteneur mort, de manière à pouvoir diagnostiquer et résoudre les problèmes.

## <a name="access-the-logs-of-a-running-container"></a>Accéder aux journaux d’activité d’un conteneur en cours d’exécution
Les fichiers journaux d’activité de conteneurs sont accessibles via [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).  Dans un navigateur web, ouvrez Service Fabric Explorer à partir du point de terminaison de gestion du cluster, en accédant à `http://mycluster.region.cloudapp.azure.com:19080/Explorer`.  

Les fichiers journaux d’activité de conteneurs sont hébergés sur le nœud de cluster sur lequel est exécutée l’instance du service de conteneurs. Par exemple, récupérez les journaux d’activité du conteneur web frontal de l’[exemple d’application Linux Voting](service-fabric-quickstart-containers-linux.md). Dans l’arborescence, développez **Cluster**>**Applications**>**VotingType**>**fabric:/Voting/azurevotefront**.  Ensuite, développez la partition (d1aa737e-f22a-e347-be16-eec90be24bc1, dans cet exemple) et vérifiez que le conteneur est exécuté sur le nœud de cluster *_lnxvm_0*.

Dans l’arborescence, recherchez le package de code sur le nœud *_lnxvm_0*, en développant **Nœuds**> **_lnxvm_0**>**fabric:/Voting**>**azurevotfrontPkg**>**Packages de code**>**code**.  Ensuite, sélectionnez **Journaux d’activité de conteneur** afin d’afficher les fichiers journaux d’activité de conteneurs.

![Plateforme Service Fabric][Image1]

## <a name="access-the-logs-of-a-dead-or-crashed-container"></a>Accéder aux journaux d’activité d’un conteneur planté ou mort
À compter de v6.2, vous pouvez également extraire les journaux d’activité pour un conteneur mort ou planté à l’aide d’[API REST](/rest/api/servicefabric/sfclient-index) ou de commandes [SFCTL (Service Fabric CLI)](service-fabric-cli.md).

### <a name="set-container-retention-policy"></a>Définir la stratégie de rétention de conteneur
Pour aider à diagnostiquer les échecs de démarrage du conteneur, Service Fabric (version 6.1 ou supérieure) prend en charge les conteneurs conservés qui ont terminé ou échoué leur démarrage. Cette stratégie peut être définie dans le fichier **ApplicationManifest.xml** comme indiqué dans l’extrait de code suivant :
```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
 ```

Le paramètre **ContainersRetentionCount** spécifie le nombre de conteneurs à conserver en cas d’échec. Si une valeur négative est spécifiée, tous les conteneurs en échec sont conservés. Quand l’attribut **ContainersRetentionCount** n’est pas spécifié, aucun conteneur n’est conservé. L’attribut **ContainersRetentionCount** prend également en charge les paramètres de l’application. Les utilisateurs peuvent ainsi spécifier des valeurs différentes pour les clusters de test et de production. Utilisez des contraintes de placement pour cibler le service conteneur sur un nœud particulier lors de l’utilisation de cette fonctionnalité pour empêcher le service conteneur de se déplacer vers d’autres nœuds. Tous les conteneurs conservés à l’aide de cette fonctionnalité doivent être supprimés manuellement.

Le paramètre **RunInteractive** correspond aux [indicateurs](https://docs.docker.com/engine/reference/commandline/run/#options)`--interactive` et `tty` de Docker. Quand ce paramètre a la valeur true dans le fichier manifeste, ces indicateurs sont utilisés pour démarrer le conteneur.  

### <a name="rest"></a>REST
Utilisez l’opération [Obtenir les journaux d’activité de conteneur déployés sur le nœud](/rest/api/servicefabric/sfclient-api-getcontainerlogsdeployedonnode) afin d’obtenir les journaux d’activité pour un conteneur planté. Spécifiez le nom du nœud sur lequel le conteneur s’exécutait, le nom de l’application, le nom du manifeste de service et le nom du package de code.  Spécifiez `&Previous=true`. La réponse contiendra les journaux d’activité du conteneur mort de l’instance de package de code.

L’URI de requête a le format suivant :

```
/Nodes/{nodeName}/$/GetApplications/{applicationId}/$/GetCodePackages/$/ContainerLogs?api-version=6.2&ServiceManifestName={ServiceManifestName}&CodePackageName={CodePackageName}&Previous={Previous}
```

Exemple de requête :
```
GET http://localhost:19080/Nodes/_Node_0/$/GetApplications/SimpleHttpServerApp/$/GetCodePackages/$/ContainerLogs?api-version=6.2&ServiceManifestName=SimpleHttpServerSvcPkg&CodePackageName=Code&Previous=true  
```

Corps de réponse 200 :
```json
{   "Content": "Exception encountered: System.Net.Http.HttpRequestException: Response status code does not indicate success: 500 (Internal Server Error).\r\n\tat System.Net.Http.HttpResponseMessage.EnsureSuccessStatusCode()\r\n" } 
```

### <a name="service-fabric-sfctl"></a>Service Fabric (SFCTL)
Utilisez la commande [sfctl service get-container-logs](service-fabric-sfctl-service.md) afin d’extraire les journaux d’activité pour un conteneur planté.  Spécifiez le nom du nœud sur lequel le conteneur s’exécutait, le nom de l’application, le nom du manifeste de service et le nom du package de code. Spécifiez l’indicateur `--previous`.  La réponse contiendra les journaux d’activité du conteneur mort de l’instance de package de code.

```
sfctl service get-container-logs --node-name _Node_0 --application-id SimpleHttpServerApp --service-manifest-name SimpleHttpServerSvcPkg --code-package-name Code –-previous
```
Réponse :
```json
{   "content": "Exception encountered: System.Net.Http.HttpRequestException: Response status code does not indicate success: 500 (Internal Server Error).\r\n\tat System.Net.Http.HttpResponseMessage.EnsureSuccessStatusCode()\r\n" }
```

## <a name="next-steps"></a>Étapes suivantes
- Exécutez le [didacticiel sur la création d’une application conteneur Linux](service-fabric-tutorial-create-container-images.md).
- En savoir plus sur [Service Fabric et les conteneurs](service-fabric-containers-overview.md)

[Image1]: media/service-fabric-containers-view-logs/view-container-logs-sfx.png
