---
title: Configurer le pare-feu IP pour l’espace de noms Azure Relay
description: Cet article explique comment utiliser des règles de pare-feu pour autoriser les connexions d’adresses IP spécifiques à des espaces de noms Azure Relay.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: ad8feed5df49dcc4503226a5fae50195bb9d48aa
ms.sourcegitcommit: 090ea6e8811663941827d1104b4593e29774fa19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91999500"
---
# <a name="configure-ip-firewall-for-an-azure-relay-namespace"></a>Configurer le pare-feu IP pour un espace de noms Azure Relay
Par défaut, les espaces de noms Azure Relay sont accessibles sur Internet tant que la demande s’accompagne d’une authentification et d’une autorisation valides. Avec le pare-feu IP, vous pouvez les limiter à un ensemble d’adresses IPv4 ou de plages d’adresses IPv4 dans la notation [CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Cette fonctionnalité est utile dans les scénarios où Azure Relay ne doit être accessible qu’à partir de certains sites bien connus. Les règles de pare-feu permettent de configurer des règles pour accepter le trafic provenant d’adresses IPv4 spécifiques. Par exemple, si vous utilisez Azure Relay avec [Azure ExpressRoute](../expressroute/expressroute-faqs.md#supported-services), vous pouvez créer une **règle de pare-feu** pour autoriser uniquement le trafic provenant des adresses IP de votre infrastructure locale. 


> [!IMPORTANT]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. 


## <a name="enable-ip-firewall-rules"></a>Activer les règles de pare-feu IP
Les règles de pare-feu IP sont appliquées au niveau de l’espace de noms. Par conséquent, les règles s’appliquent à toutes les connexions de clients utilisant un protocole pris en charge. Toute tentative de connexion à partir d’une adresse IP qui ne correspond pas à une règle IP autorisée dans l’espace de noms est rejetée comme étant non autorisée. La réponse ne mentionne pas la règle IP. Les règles de filtre IP sont appliquées dans l’ordre et la première règle qui correspond à l’adresse IP détermine l’action d’acceptation ou de rejet.

### <a name="use-azure-portal"></a>Utiliser le portail Azure
Cette section explique comment utiliser le portail Azure afin de créer des règles de pare-feu IP pour un espace de noms. 

1. Accédez à votre **espace de noms Azure Relay** sur le [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, sélectionnez l’option **Réseaux**. Si vous sélectionnez l’option **Tous les réseaux** dans la section **Autoriser l’accès à partir de**, l’espace de noms Azure Relay accepte les connexions depuis n’importe quelle adresse IP. Ce paramètre est équivalent à une règle qui accepte la plage d’adresses IP 0.0.0.0/0. 

    ![La capture d’écran montre la page Mise en réseau avec l’option Tous les réseaux sélectionnée.](./media/ip-firewall/all-networks-selected.png)
1. Pour restreindre l’accès à des réseaux et des adresses IP spécifiques, sélectionnez l’option **Réseaux sélectionnés**. Dans la section **Pare-feu**, suivez ces étapes :
    1. Sélectionnez l’option **Ajouter l’adresse IP de votre client** pour permettre à l’adresse IP de votre client actuel d’accéder à l’espace de noms. 
    2. Dans **Plage d’adresses**, entrez une adresse IPv4 ou une plage d’adresses IPv4 spécifique en notation CIDR. 

        ![Option Pare-feu – Tous les réseaux sélectionnée](./media/ip-firewall/selected-networks-trusted-access-disabled.png)
3. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer les paramètres. Patientez quelques minutes jusqu’à ce que la confirmation s’affiche dans les notifications du portail.


### <a name="use-resource-manager-template"></a>Utilisation d’un modèle Resource Manager
Le modèle Resource Manager suivant permet d’ajouter une règle de filtre IP à un espace de noms Azure Relay.

Le modèle prend un paramètre : **ipMask**, qui est une adresse IPv4 unique ou un bloc d’adresses IP en notation CIDR. Par exemple, dans la notation CIDR, 70.37.104.0/24 représente les 256 adresses IPv4 comprises entre 70.37.104.0 et 70.37.104.255, 24 indiquant le nombre de bits de préfixe significatifs pour la plage.

> [!NOTE]
> Bien qu’il n’existe aucune règle de refus possible, l’action par défaut du modèle Azure Resource Manager est **Autoriser**, ce qui ne restreint pas les connexions.
> Lorsque vous élaborez des règles de réseau virtuel ou de pare-feu, vous devez modifier ***defaultAction***
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
      "relayNamespaceName": {
        "type": "string",
        "metadata": {
          "description": "Name of the Relay namespace"
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
      "namespaceNetworkRuleSetName": "[concat(parameters('relayNamespaceName'), concat('/', 'default'))]"
    },
    "resources": [
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[parameters('relayNamespaceName')]",
        "type": "Microsoft.Relay/namespaces",
        "location": "[parameters('location')]",
        "sku": {
          "name": "Standard",
          "tier": "Standard"
        },
        "properties": { }
      },
      {
        "apiVersion": "2018-01-01-preview",
        "name": "[variables('namespaceNetworkRuleSetName')]",
        "type": "Microsoft.Relay/namespaces/networkrulesets",
        "dependsOn": [
          "[concat('Microsoft.Relay/namespaces/', parameters('relayNamespaceName'))]"
        ],
        "properties": {
          "ipRules": 
          [
            {
                "ipMask":"10.1.1.1",
                "action":"Allow"
            },
            {
                "ipMask":"11.0.0.0/24",
                "action":"Allow"
            }
          ],
          "virtualNetworkRules": [],
          "trustedServiceAccessEnabled": false,
          "defaultAction": "Deny"
        }
      }
    ],
    "outputs": { }
  }
```

Pour déployer le modèle, suivez les instructions pour [Azure Resource Manager](../azure-resource-manager/templates/deploy-powershell.md).



## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les autres fonctionnalités relatives à la sécurité réseau, consultez [Sécurité du réseau](network-security.md).


<!-- Links -->

[express-route]:  ../expressroute/expressroute-faqs.md#supported-services
