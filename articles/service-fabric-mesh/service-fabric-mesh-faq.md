---
title: Questions courantes sur Azure Service Fabric Mesh
description: Découvrez les questions et réponses fréquentes concernant Azure Service Fabric mesh.
ms.author: pepogors
ms.date: 4/23/2019
ms.topic: troubleshooting
ms.openlocfilehash: 64eeb43d743d71d5acd456409445a4fadfe91aeb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260121"
---
# <a name="commonly-asked-service-fabric-mesh-questions"></a>Questions fréquemment posées sur Service Fabric mesh

Azure Service Fabric mesh est un service entièrement géré qui permet aux développeurs de déployer des applications de microservices sans gestion de machines virtuelles, de stockage ou de mise en réseau. Cet article contient des réponses aux questions fréquemment posées.

## <a name="how-do-i-report-an-issue-or-ask-a-question"></a>Comment faire pour signaler un problème ou poser une question ?

Posez des questions, obtenez des réponses d’ingénieurs Microsoft, et signalez des problèmes dans le [dépôt GitHub service-fabric-mesh-preview](https://aka.ms/sfmeshissues).

## <a name="quota-and-cost"></a>Quota et coût

### <a name="what-is-the-cost-of-participating-in-the-preview"></a>Quel est le coût de participation à la préversion ?

Le déploiement d’applications ou de conteneurs sur la préversion de Mesh n’occasionne actuellement pas de frais. Veuillez consulter les mises à jour de mai pour activer la facturation. En revanche, nous vous invitons à supprimer les ressources que vous déployez et à ne pas les laisser s’exécuter, sauf si vous les testez de façon active.

### <a name="is-there-a-quota-limit-of-the-number-of-cores-and-ram"></a>Y a-t-il une limite de quota concernant le nombre de cœurs et la RAM ?

Oui. Les quotas pour chaque abonnement sont les suivants :

- Nombre d’applications : 5
- Cœurs par application : 12
- RAM totale par application : 48 Go
- Points de terminaison réseau et d’entrée : 5
- Volumes Azure que vous pouvez attacher : 10
- Nombre de réplicas de service : 3
- Le plus grand conteneur que vous pouvez déployer est limité à 4 cœurs et 16 Go de RAM.
- Vous pouvez allouer des cœurs partiels à vos conteneurs, par incréments de 0,5 cœurs, jusqu’à 6 cœurs maximum.

### <a name="how-long-can-i-leave-my-application-deployed"></a>Combien de temps puis-je laisser mon application déployée ?

Actuellement, nous avons réduit la durée de vie des applications à deux jours, ceci afin d’optimiser l’utilisation des cœurs libres alloués à la préversion. Par conséquent, vous êtes autorisé à exécuter un déploiement en continu pendant 48 heures au maximum, après quoi il est arrêté.

Si ce phénomène se produit, vous pouvez vérifier que le système est à l’origine de l’interruption en exécutant la commande Azure CLI `az mesh app show`. Cette commande doit renvoyer le résultat `"status": "Failed", "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue."` 

Par exemple : 

```azurecli
az mesh app show --resource-group myResourceGroup --name helloWorldApp
```

```output
{
  "debugParams": null,
  "description": "Service Fabric Mesh HelloWorld Application!",
  "diagnostics": null,
  "healthState": "Ok",
  "id": "/subscriptions/1134234-b756-4979-84re-09d671c0c345/resourcegroups/myResourceGroup/providers/Microsoft.ServiceFabricMesh/applications/helloWorldApp",
  "location": "eastus",
  "name": "helloWorldApp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "serviceNames": [
    "helloWorldService"
  ],
  "services": null,
  "status": "Failed",
  "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue.",
  "tags": {},
  "type": "Microsoft.ServiceFabricMesh/applications",
  "unhealthyEvaluation": null
}
```

Pour supprimer le groupe de ressources, utilisez la commande `az group delete <nameOfResourceGroup>`.

## <a name="deployments"></a>Déploiements

### <a name="what-container-images-are-supported"></a>Quelles sont les images conteneur prises en charge ?

Si vous réalisez votre développement sur un ordinateur doté de Windows Fall Creators Update (version 1709), vous pouvez uniquement utiliser des images de la version 1709 de Windows.

Si vous réalisez votre développement sur un ordinateur doté de la mise à jour d'avril 2018 de Windows 10 (version 1803), vous pouvez utiliser des images Docker de la version 1709 de Windows et de la version 1803 de Windows.

Lors du déploiement des services, vous pouvez utiliser les images de système d’exploitation du conteneur suivantes :
- Windows : windowsservercore et nanoserver
    - Windows Server 1709
    - Windows Server 1803
    - Windows Server 1809
    - Windows Server 2019 LTSC
- Linux
    - Aucune limitation connue

> [!NOTE]
> Les outils Visual Studio pour Mesh ne prennent pas encore en charge le déploiement dans les conteneurs Windows Server 2019 et 1809.

### <a name="what-types-of-applications-can-i-deploy"></a>Quels types d’applications puis-je déployer ? 

Vous pouvez déployer toutes les applications qui s’exécutent dans des conteneurs qui respectent les restrictions placées sur une ressource d’application (voir ci-dessus pour plus d’informations sur les quotas). Si nous détectons que vous utilisez Mesh pour l’exécution de charges de travail illégales ou que vous utilisez de façon abusive le système (par exemple, exploration de données), nous nous réservons le droit d’interrompre vos déploiements et d’empêcher votre abonnement de s’exécuter sur le service. Veuillez nous contacter si vous avez des questions sur l’exécution d’une charge de travail spécifique. 

## <a name="developer-experience-issues"></a>Problèmes relatifs à l’expérience de développement

### <a name="dns-resolution-from-a-container-doesnt-work"></a>Un résolution DNS d’un conteneur ne fonctionne pas

Les requêtes DNS sortantes d’un conteneur vers le service DNS Service Fabric peut échouer dans certaines circonstances. Ce problème est en cours d’examen. Solution de contournement :

- Utilisez Windows Fall Creators Update (version 1709) ou une version ultérieure en tant qu’image de conteneur de base.
- Si l’utilisation du nom du service seul ne fonctionne pas, essayez avec le nom qualifié complet : ServiceName.ApplicationName.
- Dans le fichier Docker pour votre service, ajoutez `EXPOSE <port>`, où « port » correspond au port d’exposition de votre service. Par exemple :

```Dockerfile
EXPOSE 80
```

### <a name="dns-does-not-work-the-same-as-it-does-for-service-fabric-development-clusters-and-in-mesh"></a>Le DNS ne fonctionne pas de la même façon dans mes clusters de développement Service Fabric et dans Mesh

Il se peut que vous deviez référencer les services différemment dans votre cluster de développement local et dans Azure Mesh.

Dans votre cluster de développement local, utilisez `{serviceName}.{applicationName}`. Dans Azure Service Fabric Mesh, utilisez `{servicename}`. 

À l’heure actuelle, Azure Mesh ne prend pas en charge la résolution DNS entre applications.

Pour d’autres problèmes de DNS connus en lien avec l’exécution d’un cluster de développement Service Fabric sur Windows 10, consultez les pages : [Déboguer les conteneurs Windows](../service-fabric/service-fabric-how-to-debug-windows-containers.md) et [Problèmes DNS connus](../service-fabric/service-fabric-dnsservice.md#known-issues).

### <a name="networking"></a>Mise en réseau

Il se peut que le NAT du réseau ServiceFabric disparaisse lors de l’exécution de votre application sur votre ordinateur local. Pour savoir si ce phénomène s’est produit, exécutez la commande suivante à partir d’une invite de commandes :

`docker network ls`, et déterminez si la valeur `servicefabric_nat` est présente.  Si elle est absente, exécutez la commande suivante : `docker network create -d=nat --subnet 10.128.0.0/24 --gateway 10.128.0.1 servicefabric_nat`

Cette commande permet de résoudre le problème, même si l’application est déjà déployée localement en état défectueux.

### <a name="issues-running-multiple-apps"></a>Problèmes relatifs à l’exécution d’applications multiples

Il se peut que la disponibilité du processeur et les limites fixées pour toutes les applications posent problème. Solution de contournement :
- Créez un cluster à cinq nœuds.
- Réduisez l’utilisation du processeur par les services sur l’application déployée. Par exemple, dans le fichier service.yaml de votre service, remplacez `cpu: 1.0` par `cpu: 0.5`

Il est impossible de déployer plusieurs applications sur un cluster à un nœud. Solution de contournement :
- Utilisez un cluster à cinq nœuds lors du déploiement de plusieurs applications sur un cluster local.
- Supprimez les applications qui ne sont pas actuellement en test.

### <a name="vs-tooling-has-limited-support-for-windows-containers"></a>Les outils VS ont une prise en charge limitée pour les conteneurs Windows

Les outils Visual Studio prennent uniquement en charge le déploiement de conteneurs Windows avec une version de système d’exploitation de base de Windows Server 1709 et 1803 aujourd’hui. 

## <a name="feature-gaps-and-other-known-issues"></a>Lacunes relatives aux fonctionnalités et autres problèmes connus

### <a name="after-deploying-my-application-the-network-resource-associated-with-it-does-not-have-an-ip-address"></a>Après avoir déployé mon application, je vois que la ressource réseau qui y est associée ne dispose pas d’adresse IP

Dans certains cas, l’adresse IP n’est pas immédiatement disponible. Ce problème est connu. Vérifiez l’état de la ressource réseau dans quelques minutes pour voir l’adresse IP associée.

### <a name="my-application-fails-to-access-the-right-networkvolume-resource"></a>Mon application ne parvient pas à accéder à la ressource réseau/de volume adéquate

Dans votre modèle d’application, utilisez l’ID de ressource complet pour que les réseaux et volumes puissent accéder à la ressource associée. Voici un exemple tiré du guide de démarrage rapide :

```json
"networkRefs": [
    {
    "name":  "[resourceId('Microsoft.ServiceFabric/networks', 'SbzVotingNetwork')]" 
    }
]
```

### <a name="when-i-scale-out-all-of-my-containers-are-affected-including-running-ones"></a>Lorsque j’effectue un scale-out, tous mes conteneurs sont affectés, y compris ceux qui sont en cours d’exécution

Il s’agit d’un bogue. Un correctif est en cours d’implémentation.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Service Fabric mesh, voir la [Vue d’ensemble](service-fabric-mesh-overview.md).
