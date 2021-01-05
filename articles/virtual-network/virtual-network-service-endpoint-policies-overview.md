---
title: Stratégies de point de terminaison de service de réseau virtuel Azure | Microsoft Docs
description: Découvrez comment filtrer le trafic de réseau virtuel vers les ressources de service Azure à l’aide de stratégies de point de terminaison de service
services: virtual-network
documentationcenter: na
author: RDhillon
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/21/2020
ms.author: rdhillon
ms.openlocfilehash: 14ecb30af11bf750c90e45c3fb6b443d861a1445
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89400753"
---
# <a name="virtual-network-service-endpoint-policies-for-azure-storage"></a>Stratégies de points de terminaison de service de réseau virtuel pour le stockage Azure

Les stratégies de points de terminaison de service de réseau virtuel permettent de filtrer le trafic de réseau virtuel sortant vers les comptes de stockage Azure sur le point de terminaison de service et d’exfiltrer des données vers des comptes de stockage Azure spécifiques uniquement. Les stratégies de points de terminaison fournissent un contrôle d’accès précis pour le trafic de réseau virtuel vers Stockage Azure lors de la connexion via le point de terminaison de service.

![Sécurisation du trafic sortant du réseau virtuel vers les comptes de stockage Azure](./media/virtual-network-service-endpoint-policies-overview/vnet-service-endpoint-policies-overview.png)

Cette fonctionnalité est généralement disponible pour le __stockage Azure__ dans __toutes les régions Azure à l’échelle internationale__.

## <a name="key-benefits"></a>Principaux avantages

Les stratégies de point de terminaison de service de réseau virtuel offrent les avantages suivants :

- __Sécurité renforcée pour votre trafic de réseau virtuel vers Stockage Azure__

  Les [étiquettes de service Azure pour les groupes de sécurité réseau](https://aka.ms/servicetags) vous permettent de restreindre le trafic sortant du réseau virtuel vers des régions de stockage Azure spécifiques. Toutefois, cela autorise le trafic vers n’importe quel compte au sein de la région de stockage Azure sélectionnée.
  
  Les stratégies de points de terminaison vous permettent de spécifier les comptes de stockage Azure qui bénéficient d’un accès sortant du réseau virtuel et limitent l’accès à tous les autres comptes de stockage. Cela apporte un contrôle de sécurité beaucoup plus précis pour la protection de l’exfiltration de données à partir de votre réseau virtuel.

- __Stratégies évolutives et hautement disponibles pour filtrer le trafic de service Azure__

   Les stratégies de point de terminaison fournissent une solution évolutive horizontalement et hautement disponible pour filtrer le trafic de service Azure à partir de réseaux virtuels, sur les points de terminaison de service. Aucun temps système supplémentaire n’est nécessaire afin de maintenir les appliances réseau centrales pour ce trafic sur vos réseaux virtuels.

## <a name="json-object-for-service-endpoint-policies"></a>Objet JSON pour les stratégies de points de terminaison de service
Examinons rapidement l’objet de stratégie de point de terminaison de service.

```json
"serviceEndpointPolicyDefinitions": [
    {
            "description": null,
            "name": "MySEP-Definition",
            "resourceGroup": "MySEPDeployment",
            "service": "Microsoft.Storage",
            "serviceResources": [ 
                    "/subscriptions/subscriptionID/resourceGroups/MySEPDeployment/providers/Microsoft.Storage/storageAccounts/mystgacc"
            ],
            "type": "Microsoft.Network/serviceEndpointPolicies/serviceEndpointPolicyDefinitions"
    }
]
```

## <a name="configuration"></a>Configuration

-   Vous pouvez configurer les stratégies de points de terminaison pour limiter le trafic de réseau virtuel à des comptes de stockage Azure spécifiques.
-   La stratégie de point de terminaison de service est configurée sur un sous-réseau dans un réseau virtuel. Les points de terminaison de service pour le stockage Azure doivent être activés sur le sous-réseau afin d’appliquer la stratégie.
-   La stratégie de point de terminaison vous permet d’ajouter des comptes de stockage Azure spécifiques à la liste verte, en utilisant le format resourceID. Vous pouvez restreindre l’accès à
    - tous les comptes de stockage dans un abonnement<br>
      `E.g. /subscriptions/subscriptionId`

    - tous les comptes de stockage dans un groupe de ressources<br>
      `E.g. subscriptions/subscriptionId/resourceGroups/resourceGroupName`
     
    - un compte de stockage individuel en listant le resourceId Azure Resource Manager correspondant. Cela concerne le trafic vers les objets blob, les tables, les files d’attente, les fichiers et Azure Data Lake Storage Gen2. <br>
    `E.g. /subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.Storage/storageAccounts/storageAccountName`
-   Par défaut, si aucune stratégie n’est attachée à un sous-réseau avec des points de terminaison, vous pouvez accéder à tous les comptes de stockage du service. Une fois qu’une stratégie est configurée sur ce sous-réseau, seules les ressources spécifiées dans la stratégie sont accessibles à partir d’instances de calcul dans ce sous-réseau. L’accès à tous les autres comptes de stockage sera refusé.
-   Lorsque vous appliquez des stratégies de points de terminaison de service sur un sous-réseau, l’*étendue du point de terminaison de service* de stockage Azure est mise à niveau de régionale vers **globale**. Cela signifie que tout le trafic vers le stockage Azure est sécurisé par la suite sur le point de terminaison de service. Les stratégies de points de terminaison de service sont également applicables globalement, de sorte que tous les comptes de stockage qui ne sont pas explicitement autorisés se verront refuser l’accès. 
-   Vous pouvez appliquer plusieurs stratégies à un sous-réseau. Lorsque plusieurs stratégies sont associées au sous-réseau, le trafic de réseau virtuel vers les ressources spécifiées dans toutes ces stratégies est autorisé. L’accès à toutes les autres ressources de service, qui ne sont pas spécifiées dans ces stratégies, est refusé.

    > [!NOTE]  
    > Les stratégies de points de terminaison de service étant des **stratégies d’autorisation**, à part les ressources spécifiées, toutes les autres ressources sont limitées. Vérifiez que toutes les dépendances de ressource de service pour vos applications sont identifiées et listées dans la stratégie.

- Seuls les comptes de stockage utilisant le modèle de ressource Azure peuvent être spécifiés dans la stratégie de point de terminaison. Vos comptes de stockage Azure classiques ne prennent pas en charge les stratégies de points de terminaison de service Azure.
- L’accès secondaire RA-GRS est automatiquement autorisé si le compte principal est répertorié.
- Les comptes de stockage peuvent se trouver dans le même abonnement, dans un autre abonnement ou dans le locataire Azure Active Directory en tant que réseau virtuel.

## <a name="scenarios"></a>Scénarios

- **Réseaux virtuels appairés, connectés ou multiples** : Pour filtrer le trafic dans des réseaux virtuels appairés, les stratégies de point de terminaison doivent leur être appliquées individuellement.
- **Filtrage du trafic Internet avec des appliances réseau ou le Pare-feu Azure** : Filtrez le trafic du service Azure avec des stratégies, sur les points de terminaison de service et filtrez le reste du trafic Internet ou Azure par le biais d’appliances ou du pare-feu Azure.
- **Filtrage du trafic sur les services Azure déployés dans des réseaux virtuels** : À ce stade, les stratégies de points de terminaison de service Azure ne sont pas prises en charge pour les services Azure managés qui sont déployés dans votre réseau virtuel. 
- **Filtrage du trafic local vers les services Azure** : Les stratégies de point de terminaison de service s’appliquent uniquement au trafic en provenance des sous-réseaux associés aux stratégies. Pour autoriser l’accès à des ressources de service Azure spécifiques en local, le trafic doit être filtré à l’aide d’appliances de réseau virtuel ou de pare-feu.

## <a name="logging-and-troubleshooting"></a>Journalisation et résolution des problèmes
Aucun enregistrement centralisé n’est disponible pour les stratégies de point de terminaison de service. Pour les journaux de ressources de service, consultez [Journalisation des points de terminaison de service](virtual-network-service-endpoints-overview.md#logging-and-troubleshooting).

### <a name="troubleshooting-scenarios"></a>Scénarios de résolution des problèmes
- Accès refusé aux comptes de stockage qui fonctionnaient dans la préversion (et non dans la région géographiquement associée)
  - Avec la mise à niveau du stockage Azure pour utiliser des balises de service global, l’étendue du point de terminaison de service et, par conséquent, les stratégies de points de terminaison de service sont désormais globales. Ainsi, tout le trafic vers le stockage Azure est chiffré sur les points de terminaison de service et l’accès est autorisé uniquement aux comptes de stockage qui sont explicitement listés dans la stratégie.
  - Établissez explicitement une liste verte de tous les comptes de stockage requis pour restaurer l’accès.  
  - Contactez le support Azure.
- L’accès est refusé aux comptes répertoriés dans les stratégies de point de terminaison
  - Le filtrage des groupes de sécurité réseau ou du pare-feu peut bloquer l’accès
  - Si la suppression/nouvelle application de la stratégie entraîne la perte de la connectivité :
    - Vérifiez que le service Azure est configuré pour autoriser l’accès à partir du réseau virtuel sur les points de terminaison, ou que la stratégie par défaut pour la ressource est définie sur *Allow All* (Tout autoriser).
    - Vérifiez que les diagnostics de service affichent le trafic sur les points de terminaison.
    - Vérifiez si les journaux de flux de groupe de sécurité réseau affichent l’accès et si les journaux d’activité de stockage affichent l’accès, comme prévu, sur les points de terminaison de service.
    - Contactez le support Azure.
- L’accès est refusé aux comptes non répertoriés dans les stratégies de point de terminaison de service
  - Vérifiez que le stockage Azure est configuré pour autoriser l’accès à partir du réseau virtuel sur les points de terminaison, ou que la stratégie par défaut pour la ressource est définie sur *Allow All* (Tout autoriser).
  - Vérifiez que les comptes ne sont pas des **comptes de stockage classiques** avec les stratégies de points de terminaison de service sur le sous-réseau.
- Un service Azure managé a cessé de fonctionner après l’application d’une stratégie de point de terminaison de service sur le sous-réseau
  - Les services managés ne sont pas pris en charge avec les stratégies de points de terminaison de service pour l’instant. *Surveillez cette page pour obtenir des mises à jour*.

## <a name="provisioning"></a>Approvisionnement

Les stratégies de points de terminaison de service peuvent être configurées sur les sous-réseaux par un utilisateur avec accès en écriture à un réseau virtuel. En savoir plus sur les [rôles intégrés](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) Azure et l’assignation d’autorisations spécifiques aux [rôles personnalisés](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Les réseaux virtuels et les comptes de stockage Azure peuvent se trouver dans des abonnements identiques ou différents, ou dans des locataires Azure Active Directory.

## <a name="limitations"></a>Limites

- Vous pouvez uniquement déployer des stratégies de point de terminaison de service sur des réseaux virtuels déployés à l’aide du modèle de déploiement Azure Resource Manager.
- Les réseaux virtuels doivent se trouver dans la même région que la stratégie de point de terminaison de service.
- Vous pouvez uniquement appliquer la stratégie de point de terminaison de service sur un sous-réseau si les points de terminaison de service sont configurés pour les services Azure répertoriés dans la stratégie.
- Vous ne pouvez pas utiliser les stratégies de points de terminaison de service pour le trafic allant de votre réseau local aux services Azure.
- Les services managés Azure ne prennent actuellement pas en charge les stratégies de points de terminaison. Cela inclut les services managés déployés dans les sous-réseaux partagés (par exemple, *Azure HDInsight, Azure Batch, Azure ADDS, Azure Application Gateway, Azure VPN Gateway, Azure Firewall*) ou dans les sous-réseaux dédiés (par exemple, *Azure App Service Environment, Azure Cache pour Redis, Azure API Management, Azure SQL Managed Instance, les services managés classiques*).

 > [!WARNING]
 > Les services Azure déployés dans votre réseau virtuel, comme Azure HDInsight, accèdent aux autres services Azure, comme Stockage Azure, pour les besoins de l’infrastructure. La restriction de stratégie de point de terminaison à des ressources spécifiques peut bloquer l’accès à ces ressources d’infrastructure pour les services Azure déployés dans votre réseau virtuel.

- Les comptes de stockage classiques ne sont pas pris en charge dans les stratégies de point de terminaison. Par défaut, les stratégies refusent l’accès à tous les comptes de stockage classiques. Si votre application doit accéder à Azure Resource Manager et à des comptes de stockage classiques, les stratégies de point de terminaison ne doivent pas être utilisées pour ce trafic.

## <a name="pricing-and-limits"></a>Tarification et limites

L’utilisation de stratégies de point de terminaison de service n’engendre pas de frais supplémentaires. Le modèle de tarification actuel pour les services Azure (comme Stockage Azure) reste le même que celui utilisé actuellement, sur les points de terminaison de service.

Les limites suivantes sont appliquées sur les stratégies de point de terminaison de service : 

 |Ressource | Limite par défaut |
 |---------|---------------|
 |ServiceEndpointPoliciesPerSubscription |500 |
 |ServiceEndpointPoliciesPerSubnet|100 |
 |ServiceResourcesPerServiceEndpointPolicyDefinition|200 |

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [comment configurer des stratégies de point de terminaison de service de réseau virtuel](virtual-network-service-endpoint-policies-portal.md)
- En savoir plus sur les [points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md)
