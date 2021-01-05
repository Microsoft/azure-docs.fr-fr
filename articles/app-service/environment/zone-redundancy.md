---
title: Prise en charge des zones de disponibilité pour les environnements App Service
description: Apprenez à déployer vos environnements App Service pour que vos applications soient redondantes dans une zone.
author: ccompy
ms.assetid: 24e3e7eb-c160-49ff-8d46-e947818ef186
ms.topic: article
ms.date: 07/15/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 1e88aac4209f7960b2589cf43f59ead4bd129134
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90605071"
---
# <a name="availability-zone-support-for-app-service-environments"></a>Prise en charge des zones de disponibilité pour les environnements App Service

Les environnements App Service Environment (ASE) peuvent être déployés dans des zones de disponibilité (AZ).  Les clients peuvent déployer un ASE d’équilibreur de charge interne (ILB) dans une zone de disponibilité spécifique au sein d’une région Azure. Si vous épinglez votre ASE ILB à une zone de disponibilité spécifique, les ressources utilisées par un ASE ILB seront épinglées à la zone de disponibilité spécifiée ou déployées de manière redondante dans une zone.  

Un ASE ILB déployé explicitement dans une zone de disponibilité est considéré comme une ressource zonale, car l’ASE ILB est épinglé à une zone spécifique. Les dépendances ASE ILB suivantes seront épinglées à la zone spécifiée :

- l’adresse IP de l’équilibreur de charge interne de l’ASE
- les ressources de calcul utilisées par l’ASE pour gérer et exécuter des applications web

Le stockage de fichiers à distance pour les applications web déployées sur un ASE ILB utilise le stockage redondant interzone (ZRS).

Si les étapes décrites dans cet article ne sont pas suivies, les ASE ILB ne sont pas déployés automatiquement de manière zonale. Vous ne pouvez pas épingler un ASE externe avec une adresse IP publique à une zone de disponibilité spécifique. 

Les ASE ILB zonaux peuvent être créés dans l’une des régions suivantes :

- Australie Est
- Brésil Sud
- Centre du Canada
- USA Centre
- USA Est
- USA Est 2
- USA Est 2 (EUAP)
- France Centre 
- Allemagne Centre-Ouest
- Japon Est
- Europe Nord
- Europe Ouest
- Asie Sud-Est
- Sud du Royaume-Uni
- USA Ouest 2

Les applications déployées sur un ASE ILB zonal continuent à s’exécuter et à traiter le trafic sur cet ASE, même si d’autres zones de la même région subissent une panne.  Il est possible que les comportements hors runtime, dont la mise à l’échelle du plan de service d’application, la création de l’application, la configuration de l’application et la publication de l’application soient affectés par une panne dans d’autres zones de disponibilité. Le déploiement épinglé à une zone d’un ASE ILB zonal assure uniquement la continuité de service des applications déjà déployées.

## <a name="how-to-deploy-an-app-service-environment-in-an-availability-zone"></a>Comment déployer App Service Environment dans une zone de disponibilité ##

Les ASE ILB zonaux doivent être créés à l’aide de modèles ARM. Une fois qu’un ASE ILB zonal est créé via un modèle ARM, vous pouvez l’afficher et interagir avec via le portail Azure et l’interface CLI.  Un modèle ARM n’est nécessaire que pour la création initiale d’un ASE ILB zonal.

La seule modification nécessaire dans un modèle ARM pour spécifier un ASE ILB zonal est la nouvelle propriété ***zones***. La propriété ***zone*** doit être définie sur la valeur « 1 », « 2 » ou « 3 » selon la zone de disponibilité logique dans laquelle l’ASE ILB doit être épinglé.

L’exemple d’extrait de modèle ARM ci-dessous montre la nouvelle propriété ***zones*** spécifiant que l’ASE ILB doit être épinglé à la zone 2.

```
   "resources": [
      {
         "type": "Microsoft.Web/hostingEnvironments",
         "kind": "ASEV2",
         "name": "yourASENameHere",
         "apiVersion": "2015-08-01",
         "location": "your location here",
         "zones": [
            "2"
         ],
         "properties": {
         "name": "yourASENameHere",
         "location": "your location here",
         "ipSslAddressCount": 0,
         "internalLoadBalancingMode": "3",
         "dnsSuffix": "contoso-internal.com",
         "virtualNetwork": {
             "Id": "/subscriptions/your-subscription-id-here/resourceGroups/your-resource-group-here/providers/Microsoft.Network/virtualNetworks/your-vnet-name-here",
             "Subnet": "yourSubnetNameHere"
          }
         }
      }
    ]
```

Pour que votre zone d’applications soit redondante, vous devez déployer deux ASE ILB zonaux. Les deux ASE ILB zonaux doivent se trouver dans des zones de disponibilité distinctes. Vous devez ensuite déployer vos applications dans chacun des ASE ILB. Une fois vos applications créées, vous devez configurer une solution d’équilibrage de charge. La solution recommandée consiste à déployer une [passerelle applicative avec redondance interzone](../../application-gateway/application-gateway-autoscaling-zone-redundant.md) en amont de l’ASE ILB zonal. 

## <a name="in-region-data-residency"></a>Résidence des données dans la région ##

Les ASE ILB déployés dans une zone de disponibilité stockent uniquement les données client dans la région où l’ASE ILB zonal a été déployé. Le contenu du fichier de site web ainsi que les paramètres et secrets fournis par le client stockés dans App Service restent dans la région où l’ASE ILB zonal est déployé.

Les clients garantissent la résidence des données dans une région unique en suivant les étapes décrites plus haut dans la section « Comment déployer App Service Environment dans une zone de disponibilité ». Si vous configurez un environnement App Service Environment selon ces étapes, un environnement App Service Environment déployé dans une zone de disponibilité répond aux exigences de la résidence des données de la région, y compris celles spécifiées dans le [Centre de confidentialité Azure](https://azuredatacentermap.azurewebsites.net/).

Les clients peuvent valider qu’un App Service Environment est correctement configuré pour stocker des données dans une seule région en procédant comme suit : 

1. Dans l’[Explorateur de ressources](https://resources.azure.com), accédez à la ressource ARM pour App Service Environment.  Les ASE sont répertoriés sous *providers/Microsoft.Web/hostingEnvironments*.
2. Si une propriété *zones* existe dans la vue de la syntaxe JSON ARM et qu’elle contient un tableau JSON à valeur unique avec la valeur « 1 », « 2 » ou « 3 », l’ASE est déployé par zone et les données client restent dans la même région.
2. Si aucune propriété *zones* n’est présente ou si la propriété n’a pas de valeur de zone valide comme indiqué précédemment, l’ASE n’est pas déployé de manière zonale et les données client ne sont pas exclusivement stockées dans la même région.
