---
title: Préversion du déploiement Docker Compose dans Azure Service Fabric
description: Azure Service Fabric accepte le format Docker Compose pour vous permettre d’orchestrer plus facilement des conteneurs existants à l’aide de Service Fabric. Cette prise en charge est actuellement en mode préliminaire.
ms.topic: conceptual
ms.date: 2/23/2018
ms.openlocfilehash: f84dd0ecb7a4002182c8455bfd86354d794a6f7c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84691286"
---
# <a name="docker-compose-deployment-support-in-azure-service-fabric-preview"></a>Prise en charge du déploiement Docker Compose dans Azure Service Fabric (préversion)

Docker utilise le fichier [docker-compose.yml](https://docs.docker.com/compose) pour la définition d’applications à conteneurs multiples. Pour permettre aux clients familiarisés avec Docker d’orchestrer facilement des applications de conteneur existantes sur Azure Service Fabric, nous avons inclus la prise en charge en préversion du déploiement Docker Compose en mode natif dans la plateforme. Service Fabric peut accepter des fichiers `docker-compose.yml` version 3 ou ultérieure. 

Étant donné que cette prise en charge est disponible en préversion, seul un sous-ensemble des directives de Compose est reconnu.

Pour utiliser cette préversion, créez votre cluster en utilisant la version 5.7 ou plus du runtime Service Fabric via le portail Azure, ainsi que le Kit de développement logiciel (SDK) correspondant. 

> [!NOTE]
> Cette fonctionnalité est disponible en préversion et n’est pas prise en charge dans les environnements de production.
> Les exemples ci-dessous sont basés sur le runtime version 6.0 et le SDK version 2.8.

## <a name="deploy-a-docker-compose-file-on-service-fabric"></a>Déployer un fichier Docker Compose sur Service Fabric

Les commandes ci-après créent une application Service Fabric (nommée `fabric:/TestContainerApp`), que vous pouvez surveiller et gérer comme n’importe quelle autre application Service Fabric. Vous pouvez utiliser le nom de l’application spécifiée pour des requêtes d’intégrité.
Service Fabric reconnaît « DeploymentName » en tant qu’identificateur du déploiement Compose.

### <a name="use-powershell"></a>Utiliser PowerShell

Créez un déploiement Compose Service Fabric à partir d’un fichier docker-compose.yml en exécutant la commande suivante dans PowerShell :

```powershell
New-ServiceFabricComposeDeployment -DeploymentName TestContainerApp -Compose docker-compose.yml [-RegistryUserName <>] [-RegistryPassword <>] [-PasswordEncrypted]
```

`RegistryUserName` et `RegistryPassword` font référence au nom d’utilisateur et au mot de passe du registre de conteneurs. Une fois que vous avez créé le déploiement, vous pouvez en vérifier l’état à l’aide de la commande suivante :

```powershell
Get-ServiceFabricComposeDeploymentStatus -DeploymentName TestContainerApp
```

Pour supprimer le déploiement Compose, utilisez la commande suivante dans PowerShell :

```powershell
Remove-ServiceFabricComposeDeployment  -DeploymentName TestContainerApp
```

Pour démarrer une mise à niveau du déploiement Compose, utilisez la commande suivante dans PowerShell :

```powershell
Start-ServiceFabricComposeDeploymentUpgrade -DeploymentName TestContainerApp -Compose docker-compose-v2.yml -Monitored -FailureAction Rollback
```

Pour restaurer la mise à niveau du déploiement Compose, utilisez la commande suivante dans PowerShell :

```powershell
Start-ServiceFabricComposeDeploymentRollback -DeploymentName TestContainerApp
```

Une fois la mise à niveau acceptée, vous pouvez suivre sa progression à l’aide de la commande suivante :

```powershell
Get-ServiceFabricComposeDeploymentUpgrade -DeploymentName TestContainerApp
```

### <a name="use-azure-service-fabric-cli-sfctl"></a>Utiliser l’interface CLI Azure Service Fabric (sfctl)

Vous pouvez également utiliser la commande CLI Service Fabric ci-après :

```shell
sfctl compose create --deployment-name TestContainerApp --file-path docker-compose.yml [ [ --user --encrypted-pass ] | [ --user --has-pass ] ] [ --timeout ]
```

Une fois que vous avez créé le déploiement, vous pouvez en vérifier l’état à l’aide de la commande suivante :

```shell
sfctl compose status --deployment-name TestContainerApp [ --timeout ]
```

Pour supprimer le déploiement Compose, utilisez la commande ci-après :

```shell
sfctl compose remove  --deployment-name TestContainerApp [ --timeout ]
```

Pour démarrer une mise à niveau du déploiement Compose, utilisez la commande ci-après :

```shell
sfctl compose upgrade --deployment-name TestContainerApp --file-path docker-compose-v2.yml [ [ --user --encrypted-pass ] | [ --user --has-pass ] ] [--upgrade-mode Monitored] [--failure-action Rollback] [ --timeout ]
```

Pour restaurer la mise à niveau du déploiement Compose, utilisez la commande suivante :

```shell
sfctl compose upgrade-rollback --deployment-name TestContainerApp [ --timeout ]
```

Une fois la mise à niveau acceptée, vous pouvez suivre sa progression à l’aide de la commande suivante :

```shell
sfctl compose upgrade-status --deployment-name TestContainerApp
```

## <a name="supported-compose-directives"></a>Directives Compose prises en charge

Cette préversion prend en charge un sous-ensemble des options de configuration du format Compose version 3, incluant les primitives suivantes :

* Services > Déploiement > Réplicas
* Services > Déploiement > Placement > Contraintes
* Services > Déploiement > Ressources > Limites
    * -cpu-shares
    * -memory
    * -memory-swap
* Services > Commandes
* Services > Environnement
* Services > Ports
* Services > Image
* Services > Isolation (uniquement pour Windows)
* Services > Journalisation > Pilote
* Services > Journalisation > Pilote > Options
* Volume & déploiement > Volume

Configurez le cluster pour qu’il applique des limites de ressources, comme décrit dans l’article [Gouvernance des ressources Azure Service Fabric](service-fabric-resource-governance.md). Aucune des autres directives Docker Compose n’est prise en charge pour cette préversion.

### <a name="ports-section"></a>Section Ports

Spécifiez le protocole HTTP ou HTTPS dans la section Ports qui sera utilisée par l’écouteur du service Service Fabric. Ceci garantit que le protocole du point de terminaison est publié correctement avec le service de nommage pour autoriser le proxy inverse à transférer les requêtes :
* Pour router vers des services Compose de Service Fabric non sécurisés, spécifiez **/http**. Par exemple, **"80:80/http"** .
* Pour router vers des services Compose de Service Fabric sécurisés, spécifiez **/https**. Par exemple, **"443:443/https"** .

> [!NOTE]
> La syntaxe de la section Ports /http et /https est spécifique à Service Fabric pour inscrire l’URL correcte de l’écouteur Service Fabric.  Si la syntaxe du fichier Docker Compose est validée par programmation, elle peut provoquer une erreur de validation.

## <a name="servicednsname-computation"></a>Calcul de ServiceDnsName

Si le nom de service que vous spécifiez dans un fichier Compose est un nom de domaine complet (autrement dit, s’il contient un point [.]), le nom DNS inscrit par Service Fabric est `<ServiceName>` (y compris le point). Dans le cas contraire, chaque segment de chemin d’accès dans le nom de l’application devient une étiquette de domaine dans le nom DNS du service, le premier segment de chemin devenant l’étiquette du domaine de premier niveau.

Par exemple, si le nom de l’application spécifié est `fabric:/SampleApp/MyComposeApp`, `<ServiceName>.MyComposeApp.SampleApp` est le nom DNS inscrit.

## <a name="compose-deployment-instance-definition-versus-service-fabric-app-model-type-definition"></a>Déploiement Compose (définition de l’instance) et modèle d’application Service Fabric (définition de type)

Un fichier docker-compose.yml décrit un ensemble déployable de conteneurs, y compris leurs propriétés et leurs configurations.
Par exemple, le fichier peut contenir des variables d’environnement et des ports. Dans le fichier docker-compose.yml, vous pouvez également spécifier des paramètres de déploiement, comme les contraintes de positionnement, les limites de ressources et les noms DNS.

Le [modèle d’application Service Fabric](service-fabric-application-model.md) utilise des types de service et des types d’application. Il peut y avoir de nombreuses instances d’application du même type. Par exemple, vous pouvez avoir une seule instance d’application par client. Ce modèle basé sur le type prend en charge plusieurs versions du même type d’application inscrit auprès du runtime.

Par exemple, le client A peut utiliser une application instanciée avec le type 1.0 d’AppTypeA, tandis que le client B peut disposer d’une autre application instanciée avec le même type et la même version. Vous définissez les types d’applications dans les manifestes d’application, et vous spécifiez le nom de l’application et les paramètres de déploiement au moment où vous créez l’application.

Bien que ce modèle offre une certaine flexibilité, nous prévoyons également de prendre en charge un modèle de déploiement plus simple, basé sur les instances, dans lequel les types sont implicites à partir du fichier manifeste. Dans ce modèle, chaque application obtient son propre manifeste indépendant. Nous proposons cela en version préliminaire, en ajoutant la prise en charge du format docker-compose.yml, qui est un format de déploiement basé sur les instances.

## <a name="next-steps"></a>Étapes suivantes

* Lisez les informations sur le [Modèle d’application Service Fabric](service-fabric-application-model.md)
* [Prise en main de l’interface de ligne de commande Service Fabric](service-fabric-cli.md)
