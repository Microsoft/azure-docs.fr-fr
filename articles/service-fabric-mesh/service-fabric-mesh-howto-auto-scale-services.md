---
title: Mettre à l’échelle automatiquement une application s’exécutant dans Azure Service Fabric Mesh
description: Découvrez comment configurer les stratégies de mise à l’échelle automatique pour les services d’une application Azure Service Fabric Mesh.
author: georgewallace
ms.topic: conceptual
ms.date: 12/07/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: f65fcfa76069a3de37fd3a76e38e38fba40e04ac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843054"
---
# <a name="create-autoscale-policies-for-a-service-fabric-mesh-application"></a>Créer des stratégies de mise à l’échelle automatique pour une application Azure Service Fabric Mesh
Le déploiement d’applications vers Service Fabric Mesh permet notamment d’augmenter ou de diminuer la taille de vos services en toute simplicité. Vous pouvez ainsi gérer des charges variables sur vos services ou améliorer la disponibilité. Vous pouvez mettre à l’échelle manuellement vos services ou configurer des stratégies de mise à l’échelle automatique.

La [mise à l’échelle automatique](service-fabric-mesh-scalability.md#autoscaling-service-instances) vous permet d’adapter dynamiquement le nombre d’instances de service dont vous disposez (mise à l’échelle horizontale). La mise à l’échelle automatique offre une grande souplesse et permet d’approvisionner ou de supprimer des instances de service en fonction de l’utilisation du processeur ou de la mémoire.

## <a name="options-for-creating-an-auto-scaling-policy-trigger-and-mechanism"></a>Options relatives à la création d’une stratégie de mise à l’échelle automatique (déclencheur et mécanisme)
Vous devez définir une stratégie de mise à l’échelle automatique pour chaque service que vous souhaitez adapter. La stratégie est définie dans le fichier de ressources de service YAML ou dans le modèle de déploiement JSON. Chaque stratégie de mise à l’échelle se compose de deux parties : un déclencheur et un mécanisme de mise à l’échelle.

Le déclencheur définit le moment auquel la stratégie de mise à l’échelle automatique est appelée.  Vous devez spécifier le type de déclencheur (charge moyenne) et la métrique à surveiller (processeur ou mémoire).  Les seuils de charge supérieur et inférieur sont indiqués sous forme de pourcentage. L’intervalle de mise à l’échelle définit la fréquence de vérification (en secondes) de l’utilisation spécifiée (par exemple, la charge moyenne du processeur) sur toutes les instances de service actuellement déployées.  Le mécanisme est déclenché lorsque la métrique surveillée descend en dessous du seuil inférieur ou dépasse le seuil supérieur.  

Le mécanisme de mise à l’échelle définit l’opération d’adaptation lorsque la stratégie est déclenchée.  Vous devez spécifier le type de mécanisme (ajout/suppression de réplicas) et le nombre minimal et maximal de réplicas (sous forme d’entier).  Le nombre de réplicas de service n’est jamais inférieur au nombre minimal ni supérieur au nombre maximal.  Vous devez également spécifier l’incrément de mise à l’échelle (sous forme d’entier). Il s’agit du nombre de réplicas qui seront ajoutées ou supprimées lors d’une opération de mise à l’échelle.  

## <a name="define-an-auto-scaling-policy-in-a-json-template"></a>Définir une stratégie de mise à l’échelle automatique dans un modèle JSON

L’exemple suivant illustre une stratégie de mise à l’échelle automatique dans un modèle de déploiement JSON.  La stratégie de mise à l’échelle automatique est déclarée dans une propriété du service qui fait l’objet de la mise à l'échelle.  Dans cet exemple, un déclencheur relatif à la charge moyenne du processeur est défini.  Le mécanisme est déclenché si la charge moyenne du processeur pour toutes les instances déployées est inférieure à 0,2 (20 %) ou supérieure à 0,8 (80 %).  La charge du processeur est vérifiée toutes les 60 secondes.  Le mécanisme de mise à l’échelle est défini de sorte que des instances sont ajoutées ou supprimées si la stratégie est déclenchée.  Les instances de service sont ajoutées ou supprimées par incrément de un.  Le nombre minimal d’instances et le nombre maximal d’instances sont définis respectivement sur 1 et 40.

```json
{
"apiVersion": "2018-09-01-preview",
"name": "WorkerApp",
"type": "Microsoft.ServiceFabricMesh/applications",
"location": "[parameters('location')]",
"dependsOn": [
"Microsoft.ServiceFabricMesh/networks/worker-app-network"
],
"properties": {
"services": [   
    { ... },       
    {
    "name": "WorkerService",
    "properties": {
        "description": "Worker Service",
        "osType": "linux",
        "codePackages": [
        {  ...              }
        ],
        "replicaCount": 1,
        "autoScalingPolicies": [
        {
            "name": "AutoScaleWorkerRule",
            "trigger": {
                "kind": "AverageLoad",
                "metric": {
                    "kind": "Resource",
                    "name": "cpu"
                },
                "lowerLoadThreshold": "0.2",
                "upperLoadThreshold": "0.8",
                "scaleIntervalInSeconds": "60"
            },
            "mechanism": {
                "kind": "AddRemoveReplica",
                "minCount": "1",
                "maxCount": "40",
                "scaleIncrement": "1"
            }
        }
        ],        
        ...
    }
    }
]
}
}
```

## <a name="define-an-autoscale-policy-in-a-serviceyaml-resource-file"></a>Définir une stratégie de mise à l’échelle automatique dans un fichier de ressources service.yaml
L’exemple suivant illustre une stratégie de mise à l’échelle automatique dans une ressource de service (YAML).  La stratégie de mise à l’échelle automatique est déclarée sous forme de propriété du service qui fait l’objet de la mise à l'échelle.  Dans cet exemple, un déclencheur relatif à la charge moyenne du processeur est défini.  Le mécanisme est déclenché si la charge moyenne du processeur pour toutes les instances déployées est inférieure à 0,2 (20 %) ou supérieure à 0,8 (80 %).  La charge du processeur est vérifiée toutes les 60 secondes.  Le mécanisme de mise à l’échelle est défini de sorte que des instances sont ajoutées ou supprimées si la stratégie est déclenchée.  Les instances de service sont ajoutées ou supprimées par incrément de un.  Le nombre minimal d’instances et le nombre maximal d’instances sont définis respectivement sur 1 et 40.

```yaml
## Service definition ##
application:
  schemaVersion: 1.0.0-preview2
  name: WorkerApp
  properties:
    services:
      - name: WorkerService
        properties:
          description: WorkerService description.
          osType: Linux
          codePackages:
            ...
          replicaCount: 1
          autoScalingPolicies:
            - name: AutoScaleWorkerRule
              trigger:
                kind: AverageLoad
                metric:
                  kind: Resource
                  name: cpu
                lowerLoadThreshold: 0.2
                upperLoadThreshold: 0.8
                scaleIntervalInSeconds: 60
              mechanism:
                kind: AddRemoveReplica
                minCount: 1
                maxCount: 40
                scaleIncrement: 1
          ...
```

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [mettre à l'échelle un service manuellement](service-fabric-mesh-tutorial-template-scale-services.md)
