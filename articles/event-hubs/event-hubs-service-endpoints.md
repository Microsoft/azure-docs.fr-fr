---
title: Points de terminaison de service de réseau virtuel - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur l’ajout d’un point de terminaison de service Microsoft.EventHub à un réseau virtuel.
ms.topic: article
ms.date: 07/29/2020
ms.openlocfilehash: 029338e3835d03b1a66ff6629e872c84113b0ff2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015574"
---
# <a name="allow-access-to-azure-event-hubs-namespaces-from-specific-virtual-networks"></a>Autoriser l'accès à un espace de noms Azure Event Hubs à partir de réseaux virtuels spécifiques 

L’intégration d’Event Hubs à des [points de terminaison de service de réseau virtuel][vnet-sep] permet de sécuriser l’accès aux fonctionnalités de messagerie à partir de charges de travail, notamment celles de machines virtuelles liées à des réseaux virtuels. Le chemin du trafic réseau est sécurisé aux deux extrémités.

Une fois configuré pour être lié à au moins un point de terminaison de service de sous-réseau de réseau virtuel, l’espace de noms Event Hubs respectif n’accepte que le trafic provenant de sous-réseaux autorisés dans les réseaux virtuels. Du point de vue du réseau virtuel, la liaison d’un espace de noms Event Hubs à un point de terminaison de service configure un tunnel de mise en réseau isolé allant du sous-réseau de réseau virtuel au service de messagerie. 

Il en résulte une relation privée et isolée entre les charges de travail liées au sous-réseau et l’espace de noms Event Hubs respectif, et ce malgré le fait que l’adresse réseau observable du point de terminaison du service de messagerie figure dans une plage d’adresses IP publique. Toutefois, il existe une exception à ce comportement. Par défaut, l’activation d’un point de terminaison de service active à la règle `denyall` dans le [pare-feu IP](event-hubs-ip-filtering.md) associé au réseau virtuel. Vous pouvez ajouter des adresses IP spécifiques dans le pare-feu IP pour permettre l’accès au point de terminaison public Event Hub. 

>[!WARNING]
> L’activation de réseaux virtuels pour votre espace de noms Event Hubs bloque vos demandes entrantes par défaut, sauf si les demandes proviennent d’un service opérant à partir de réseaux virtuels autorisés. Les demandes qui sont bloquées comprennent les demandes émanant d’autres services Azure, du portail Azure, des services de journalisation et de métriques, etc. En guise d’exception, vous pouvez autoriser l’accès aux ressources Event Hubs à partir de certains services approuvés, même lorsque les réseaux virtuels sont activés. Pour obtenir la liste des services approuvés, consultez [Services approuvés](#trusted-microsoft-services).

> [!NOTE]
> Les réseaux virtuels sont pris en charge dans les niveaux **standard** et **dédié** d’Event Hubs. Il ne sont pas pris en charge dans le niveau **De base**.

## <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>Scénarios de sécurité avancés pris en charge par l’intégration à VNet 

En général, les solutions nécessitant une sécurité étroite et compartimentée et dans lesquelles les sous-réseaux de réseau virtuel assurent la segmentation entre les services compartimentés ont encore besoin de chemins de communication entre les services résidant dans ces compartiments.

Toute route IP immédiate entre les compartiments, notamment celles acheminant le trafic HTTPS sur TCP/IP, comporte un risque d’exploitation des vulnérabilités émanant de la couche réseau. Les services de messagerie fournissent des chemins de communication isolés, les messages étant même écrits sur disque à mesure qu’ils passent d’une partie à une autre. Les charges de travail dans deux réseaux virtuels distincts qui sont tous les deux liés à la même instance d’Event Hubs peuvent communiquer de manière efficace et fiable au moyen de messages, sans aucun impact sur l’intégrité des limites d’isolement de chaque réseau.
 
Vos solutions cloud sensibles en matière de sécurité accèdent donc aux fonctionnalités de messagerie asynchrones de pointe d’Azure qui allient fiabilité et scalabilité. Vos solutions peuvent désormais utiliser la messagerie pour créer des chemins de communication entre des compartiments de solution sécurisés, ce qui offre intrinsèquement une meilleure sécurité que n’importe quel mode de communication pair à pair (notamment HTTPS et d’autres protocoles de sockets sécurisés par TLS).

## <a name="bind-event-hubs-to-virtual-networks"></a>Lier des Event Hubs à des réseaux virtuels

Les **règles de réseau virtuel** sont une fonctionnalité de sécurité de pare-feu. Elles permettent de contrôler si votre espace de noms Azure Event Hubs doit accepter ou non les connexions d’un sous-réseau de réseau virtuel particulier.

La liaison d’un espace de noms Event Hubs à un réseau virtuel est un processus en deux étapes. Vous devez d’abord créer un **point de terminaison de service de réseau virtuel** sur un sous-réseau de réseau virtuel, puis l’activer pour **Microsoft.EventHub**, comme expliqué dans l’article [Vue d’ensemble des points de terminaison de service][vnet-sep]. Après avoir ajouté le point de terminaison de service, liez-le à l’espace de noms Event Hubs au moyen d’une **règle de réseau virtuel**.

La règle de réseau virtuel est une association de l’espace de noms Event Hubs et d’un sous-réseau de réseau virtuel. Une fois la règle en place, toutes les charges de travail liées au sous-réseau sont autorisées à accéder à l’espace de noms Event Hubs. Event Hubs n’établit jamais de connexions sortantes, ne nécessite aucun accès et ne se voit donc jamais accorder l’accès à votre sous-réseau quand cette règle est activée.

## <a name="use-azure-portal"></a>Utiliser le portail Azure
Cette section montre comment utiliser le portail Azure pour ajouter un point de terminaison de service de réseau virtuel. Pour limiter l’accès, vous devez intégrer le point de terminaison de service de réseau virtuel pour cet espace de noms Event Hubs.

1. Accédez à votre **espace de noms Event Hubs** sur le [Portail Azure](https://portal.azure.com).
4. Sous **Paramètres** sur le menu de gauche, sélectionnez **Mise en réseau**. L’onglet **Réseau** s’affiche uniquement pour les espaces de noms **standard** ou **dédiés**. 

    > [!NOTE]
    > Par défaut, l’option **Réseaux sélectionnés** est sélectionnée, comme indiqué dans l’image suivante. Si vous ne spécifiez pas de règle de pare-feu IP ou n’ajoutez pas de réseau virtuel sur cette page, l’espace de noms est accessible via l’**Internet public** (à l’aide de la clé d’accès). 

    :::image type="content" source="./media/event-hubs-firewall/selected-networks.png" alt-text="Onglet Réseaux - Option Réseaux sélectionnée" lightbox="./media/event-hubs-firewall/selected-networks.png":::    

    Si vous sélectionnez l'option **Tous les réseaux**, le hub d'événements accepte les connexions de toutes les adresses IP (à l'aide de la clé d'accès). Ce paramètre est équivalent à une règle qui accepte la plage d’adresses IP 0.0.0.0/0. 

    ![Option Pare-feu – Tous les réseaux sélectionnée](./media/event-hubs-firewall/firewall-all-networks-selected.png)
1. Pour restreindre l'accès à des réseaux spécifiques, sélectionnez l'option **Réseaux sélectionnés** en haut de la page, si ce n'est déjà fait.
2. Dans la section **Réseau virtuel** de la page, sélectionnez **+Ajouter un réseau virtuel existant** _. Sélectionnez _ *+ Créer un réseau virtuel** si vous souhaitez créer un réseau virtuel. 

    ![ajouter un réseau virtuel existant](./media/event-hubs-tutorial-vnet-and-firewalls/add-vnet-menu.png)
3. Sélectionnez le réseau virtuel dans la liste des réseaux virtuels, puis choisissez le **sous-réseau**. Vous devez activer le point de terminaison de service avant d’ajouter le réseau virtuel à la liste. Si le point de terminaison de service n’est pas activé, le portail vous invite à l’activer.
   
   ![sélectionner un sous-réseau](./media/event-hubs-tutorial-vnet-and-firewalls/select-subnet.png)

4. Une fois le point de terminaison de service pour le sous-réseau activé pour **Microsoft.EventHub**, le message suivant doit s’afficher. Sélectionnez **Ajouter** au bas de la page pour ajouter le réseau. 

    ![sélectionner le sous-réseau et activer le point de terminaison](./media/event-hubs-tutorial-vnet-and-firewalls/subnet-service-endpoint-enabled.png)

    > [!NOTE]
    > Si vous ne pouvez pas activer le point de terminaison de service, vous pouvez ignorer le point de terminaison de service de réseau virtuel manquant en utilisant le modèle Resource Manager. Cette fonctionnalité n’est pas disponible dans le portail.
5. Spécifiez si vous voulez **Autoriser les services Microsoft approuvés à contourner ce pare-feu**. Pour plus d’informations, consultez [Services Microsoft approuvés](#trusted-microsoft-services). 
6. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer les paramètres. Patientez quelques minutes jusqu’à ce que la confirmation s’affiche dans la zone de notifications du portail.

    ![Enregistrer un réseau](./media/event-hubs-tutorial-vnet-and-firewalls/save-vnet.png)

    > [!NOTE]
    > Pour restreindre l'accès à des adresses ou plages d'adresses IP spécifiques, consultez [Autoriser l'accès à partir d'adresses ou de plages d'adresses IP spécifiques](event-hubs-ip-filtering.md).

[!INCLUDE [event-hubs-trusted-services](../../includes/event-hubs-trusted-services.md)]

## <a name="use-resource-manager-template"></a>Utilisation d’un modèle Resource Manager

Le modèle Resource Manager suivant permet d’ajouter une règle de réseau virtuel à un espace de noms Event Hubs.

Paramètres du modèle :

* `namespaceName` : espace de noms Event Hubs.
* `vnetRuleName` : nom de la règle de réseau virtuel à créer.
* `virtualNetworkingSubnetId` : chemin complet de Resource Manager pour le sous-réseau de réseau virtuel, par exemple `/subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/default` pour le sous-réseau par défaut d’un réseau virtuel.

> [!NOTE]
> Bien qu’il n’existe aucune règle de refus possible, l’action par défaut du modèle Azure Resource Manager est **Autoriser**, ce qui ne restreint pas les connexions.
> Lorsque vous élaborez des règles de réseau virtuel ou de pare-feu, vous devez modifier **_« defaultAction »_**
> 
> de
> ```json
> "defaultAction": "Allow"
> ```
> to
> ```json
> "defaultAction": "Deny"
> ```
>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "eventhubNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Event Hubs namespace"
        }
      },
      "virtualNetworkName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Virtual Network Rule"
        }
      },
      "subnetName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Virtual Network Sub Net"
        }
      },
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location for Namespace"
        }
      }
    },
    "variables": {
      "namespaceNetworkRuleSetName": "[concat(parameters('eventhubNamespaceName'), concat('/', 'default'))]",
      "subNetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets/', parameters('virtualNetworkName'), parameters('subnetName'))]"
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('eventhubNamespaceName')]",
        "type": "Microsoft.EventHub/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
        },
        "properties": { }
      },
      {
        "apiVersion": "2017-09-01",
        "name": "[parameters('virtualNetworkName')]",
        "location": "[parameters('location')]",
        "type": "Microsoft.Network/virtualNetworks",
        "properties": {
          "addressSpace": {
            "addressPrefixes": [
              "10.0.0.0/23"
            ]
          },
          "subnets": [
            {
              "name": "[parameters('subnetName')]",
              "properties": {
                "addressPrefix": "10.0.0.0/23",
                "serviceEndpoints": [
                  {
                    "service": "Microsoft.EventHub"
                  }
                ]
              }
            }
          ]
        }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.EventHub/namespaces/networkruleset",
        "dependsOn": [
          "[concat('Microsoft.EventHub/namespaces/', parameters('eventhubNamespaceName'))]"
        ],
        "properties": {
          "virtualNetworkRules": 
          [
            {
              "subnet": {
                "id": "[variables('subNetId')]"
              },
              "ignoreMissingVnetServiceEndpoint": false
            }
          ],
          "ipRules":[<YOUR EXISTING IP RULES>],
          "trustedServiceAccessEnabled": false,
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

Pour déployer le modèle, suivez les instructions pour [Azure Resource Manager][lnk-deploy].

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réseaux virtuels, consultez les liens suivants :

- [Points de terminaison de service de réseau virtuel Azure][vnet-sep]
- [Filtrage IP Azure Event Hubs][ip-filtering]

[vnet-sep]: ../virtual-network/virtual-network-service-endpoints-overview.md
[lnk-deploy]: ../azure-resource-manager/templates/deploy-powershell.md
[ip-filtering]: event-hubs-ip-filtering.md
