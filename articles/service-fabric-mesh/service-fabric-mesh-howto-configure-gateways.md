---
title: Configurer une passerelle pour router les demandes
description: Découvrez comment configurer la passerelle qui gère le trafic entrant de votre ou vos applications exécutées sur Service Fabric Mesh.
author: georgewallace
ms.topic: conceptual
ms.date: 11/28/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: aa3ac9d8835cd17387346bb29b3e7c30f286cd1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91839722"
---
# <a name="configure-a-gateway-resource-to-route-requests"></a>Configurer une ressource Passerelle pour router les demandes

Une ressource Passerelle est utilisée pour router le trafic entrant vers le réseau qui héberge votre application. Configurez-la pour spécifier des règles selon lesquelles les demandes sont dirigées vers des services ou points de terminaison spécifiques en fonction de leur structure. Consultez [Introduction aux réseaux dans Service Fabric Mesh](service-fabric-mesh-networks-and-gateways.md) pour plus d’informations sur les réseaux et les passerelles dans Mesh. 

Les ressources Passerelle doivent être déclarées dans le cadre de votre modèle de déploiement (JSON ou yaml) et sont dépendantes d’une ressource Réseau. Ce document décrit les différentes propriétés qui peuvent être définies pour votre passerelle et propose un exemple de configuration de passerelle.

## <a name="options-for-configuring-your-gateway-resource"></a>Options de configuration de votre ressource Passerelle

Étant donné que la ressource Passerelle fait office de pont entre le réseau de votre application et le réseau de l’infrastructure sous-jacente (le réseau `open`), vous n’avez besoin d’en configurer qu’un seul (la préversion de Mesh limite à une passerelle par application). La déclaration pour la ressource se compose de deux parties principales : les métadonnées et les propriétés de la ressource. 

### <a name="gateway-resource-metadata"></a>Métadonnées de la ressource Passerelle

Une passerelle est déclarée avec les métadonnées suivantes :
* `apiVersion` - doit être défini avec "2018-09-01-preview" (ou ultérieur, pour plus tard)
* `name` - nom de chaîne de cette passerelle
* `type` - "Microsoft.ServiceFabricMesh/gateways"
* `location` - doit être défini avec l’emplacement de votre application / réseau ; est généralement une référence au paramètre d’emplacement dans votre déploiement
* `dependsOn` - réseau pour lequel cette passerelle servira de point d’entrée

Voici à quoi cela ressemble dans un modèle de déploiement Azure Resource Manager (JSON) : 

```json
{
  "apiVersion": "2018-09-01-preview",
  "name": "myGateway",
  "type": "Microsoft.ServiceFabricMesh/gateways",
  "location": "[parameters('location')]",
  "dependsOn": [
    "Microsoft.ServiceFabricMesh/networks/myNetwork"
  ],
  "properties": {
    [...]
  }
}
```

### <a name="gateway-properties"></a>Propriétés de la passerelle

La section des propriétés est utilisée pour définir les réseaux entre lesquels la passerelle se trouve et les règles de routage des demandes. 

#### <a name="source-and-destination-network"></a>Réseau source et de destination 

Chaque passerelle nécessite un `sourceNetwork` et un `destinationNetwork`. Le réseau source est défini comme réseau à partir duquel votre application reçoit les demandes entrantes. Sa propriété de nom doit toujours être définie avec la valeur « Open ». Le réseau de destination est le réseau que ciblent les demandes. Sa valeur de nom doit être définie avec le nom de ressource du réseau local de votre application (doit inclure la référence complète à la ressource). Voici un exemple de configuration pour un déploiement sur un réseau appelé « myNetwork ».

```json 
"properties": {
  "description": "Service Fabric Mesh Gateway",
  "sourceNetwork": {
    "name": "Open"
  },
  "destinationNetwork": {
    "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'myNetwork')]"
  },
  [...]
}
```

#### <a name="rules"></a>Règles 

Une passerelle peut avoir plusieurs règles de routage spécifiant comment va être géré le trafic entrant. Une règle de routage définit la relation entre le port d’écoute et le point de terminaison de destination pour une application donnée. Pour les règles de routage TCP, il existe un mappage 1 à 1 entre Port :Point de terminaison. Pour les règles de routage HTTP, vous pouvez définir des règles de routage plus complexes qui examinent le chemin de la demande et éventuellement les en-têtes pour décider comment la requête va être routée. 

Les règles de routage sont spécifiées pour chaque port. Chaque port d’entrée a son propre groupe de règles dans la section des propriétés de la configuration de votre passerelle. 

#### <a name="tcp-routing-rules"></a>Règles de routage TCP 

Une règle de routage TCP se compose des propriétés suivantes : 
* `name` : référence à la règle qui peut être n’importe quelle chaîne de votre choix 
* `port` : port sur lequel écouter pour les demandes entrantes 
* `destination` : spécification de point de terminaison qui inclut `applicationName`, `serviceName` et `endpointName`, vers où les demandes doivent être routées

Voici un exemple de règle de routage TCP :

```json
"properties": {
  [...]
  "tcp": [
    {
      "name": "web",
      "port": 80,
      "destination": {
        "applicationName": "myApp",
        "serviceName": "myService",
        "endpointName": "myListener"
      }
    }
  ]
}
```


#### <a name="http-routing-rules"></a>Règles de routage HTTP 

Une règle de routage HTTP se compose des propriétés suivantes : 
* `name` : référence à la règle qui peut être n’importe quelle chaîne de votre choix 
* `port` : port sur lequel écouter pour les demandes entrantes 
* `hosts` : groupe de stratégies qui s’appliquent aux demandes provenant des différents « hôtes » sur le port spécifié ci-dessus. Les hôtes sont un ensemble d’applications et de services qui peuvent s’exécuter sur le réseau et prendre en charge les demandes entrantes, par exemple une application web. Les stratégies d’hôte sont interprétées dans l’ordre, donc vous devez créer les éléments suivants dans l’ordre décroissant des niveaux de spécificité.
    * `name`  : nom DNS de l’hôte pour lequel les règles de routage suivantes sont spécifiées. L’utilisation de « * » créerait ici des règles de routage pour tous les hôtes.
    * `routes` : groupe de stratégies pour cet hôte spécifique
        * `match` : spécification de la structure de demande entrante pour cette règle à appliquer, basée sur un `path`
            * `path` : contient une `value` (URI entrant), un `rewrite` (mode de transfert souhaité pour la demande) et un `type` (peut être uniquement « Prefix » pour l’instant)
            * `header` : groupe facultatif de valeurs d’en-tête à faire correspondre dans l’en-tête de la demande si la demande correspond à la spécification de chemin (ci-dessus).
              * Chaque entrée contient un `name` (nom de la chaîne de l’en-tête à faire correspondre), une `value` (valeur de chaîne de l’en-tête de la demande) et un `type` (peut être uniquement « Exact » pour l’instant).
        * `destination` : si la demande correspond, elle est routée vers cette destination, qui est spécifiée avec un `applicationName`, un `serviceName` et un `endpointName`

Voici un exemple de règle de routage HTTP, qui s’applique à des demandes arrivant sur le port 80, pour tous les hôtes pris en charge par les applications de ce réseau. Si l’URL de demande a une structure qui correspond à la spécification de chemin, par exemple `<IPAddress>:80/pickme/<requestContent>`, elle dirige vers le point de terminaison `myListener`.  

```json
"properties": {
  [...]
  "http": [
    {
      "name": "web",
      "port": 80,
      "hosts": [
        {
          "name": "*",
          "routes": [
            {
              "match": {
                "path": {
                  "value": "/pickme",
                  "rewrite": "/",
                  "type": "Prefix"
                }
              },
              "destination": {
                "applicationName": "meshApp",
                "serviceName": "myService",
                "endpointName": "myListener"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="sample-config-for-a-gateway-resource"></a>Exemple de configuration d’une ressource Passerelle 

Voici à quoi ressemble une configuration complète d’une ressource Passerelle (adaptée de l’exemple d’entrée disponible dans le [dépôt d’exemples Mesh](https://github.com/Azure-Samples/service-fabric-mesh/blob/2018-09-01-preview/templates/ingress/meshingress.linux.json)) :

```json
{
  "apiVersion": "2018-09-01-preview",
  "name": "ingressGatewayLinux",
  "type": "Microsoft.ServiceFabricMesh/gateways",
  "location": "[parameters('location')]",
  "dependsOn": [
    "Microsoft.ServiceFabricMesh/networks/meshNetworkLinux"
  ],
  "properties": {
    "description": "Service Fabric Mesh Gateway for Linux mesh samples.",
    "sourceNetwork": {
      "name": "Open"
    },
    "destinationNetwork": {
      "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'meshNetworkLinux')]"
    },
    "http": [
      {
        "name": "web",
        "port": 80,
        "hosts": [
          {
            "name": "*",
            "routes": [
              {
                "match": {
                  "path": {
                    "value": "/hello",
                    "rewrite": "/",
                    "type": "Prefix"
                  }
                },
                "destination": {
                  "applicationName": "meshAppLinux",
                  "serviceName": "helloWorldService",
                  "endpointName": "helloWorldListener"
                }
              },
              {
                "match": {
                  "path": {
                    "value": "/counter",
                    "rewrite": "/",
                    "type": "Prefix"
                  }
                },
                "destination": {
                  "applicationName": "meshAppLinux",
                  "serviceName": "counterService",
                  "endpointName": "counterServiceListener"
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Cette passerelle est configurée pour une application Linux, "meshAppLinux", qui se compose d’au moins deux services, "helloWorldService" et "counterService", qui écoute sur le port 80. En fonction de la structure de l’URL de la demande entrante, elle route la demande vers l’un de ces services. 
* « \<IPAddress>:80/helloWorld/\<request\> » dirigerait une demande vers le « helloWorldListener » du helloWorldService. 
* « \<IPAddress>:80/counter/\<request\> » dirigerait une demande vers le « counterListener » du counterService. 

## <a name="next-steps"></a>Étapes suivantes
* Déployer l’[exemple d’entrée](https://github.com/Azure-Samples/service-fabric-mesh/tree/2018-09-01-preview/templates/ingress) pour voir les passerelles en action
