---
title: Utiliser des points de terminaison privés avec des comptes Azure Batch
description: Découvrez comment vous connecter en privé à un compte Azure Batch à l’aide de points de terminaison privés.
ms.topic: how-to
ms.date: 09/28/2020
ms.custom: references_regions
ms.openlocfilehash: 38d92d787a8d01dd3f87e1cdcacd336982c8c910
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94579553"
---
# <a name="use-private-endpoints-with-azure-batch-accounts"></a>Utiliser des points de terminaison privés avec des comptes Azure Batch

Par défaut, les [comptes Azure Batch](accounts.md) possèdent un point de terminaison public et sont accessibles publiquement. Le service Batch permet de créer des comptes Batch privés, empêchant l’accès au réseau public.

À l’aide d’[Azure Private Link](../private-link/private-link-overview.md), vous pouvez vous connecter à un compte Azure Batch via un [point de terminaison privé](../private-link/private-endpoint-overview.md). Le point de terminaison privé est un ensemble d’adresses IP privées dans un sous-réseau au sein de votre réseau virtuel. Vous pouvez alors limiter l’accès à un compte Azure Batch via des adresses IP privées.

Private Link permet aux utilisateurs d’accéder à un compte Azure Batch à partir du réseau virtuel ou de tout réseau virtuel appairé. Les ressources mappées à Private Link sont également accessibles localement via un Peering privé par le biais d’un VPN ou d’[Azure ExpressRoute](../expressroute/expressroute-introduction.md). Vous pouvez vous connecter à un compte Azure Batch configuré avec Private Link en utilisant les [méthodes d’approbation automatique ou manuelle](../private-link/private-endpoint-overview.md#access-to-a-private-link-resource-using-approval-workflow).

> [!IMPORTANT]
> La prise en charge de la connectivité privée dans Azure Batch est actuellement disponible pour toutes les régions publiques à l’exception d’Allemagne Centre et d’Allemagne Nord-Est.

Cet article décrit les étapes permettant de créer un compte Batch privé et d’y accéder à l’aide d’un point de terminaison privé.

## <a name="azure-portal"></a>Portail Azure

Procédez comme suit pour créer un compte Batch privé à l’aide du portail Azure :

1. Dans le volet **Créer une ressource**, choisissez **Service Batch**, puis sélectionnez **Créer**.
2. Entrez l’abonnement, le groupe de ressources, la région et le nom du compte Batch sous l’onglet **De base**, puis sélectionnez **Suivant : Avancé**.
3. Sous l’onglet **Avancé**, définissez **Accès réseau public** sur **Désactivé**.
4. Dans **Paramètres**, sélectionnez **Connexions des points de terminaison privés**, puis **+ Point de terminaison privé**.
   :::image type="content" source="media/private-connectivity/private-endpoint-connections.png" alt-text="Connexions des points de terminaison privés":::
5. Dans le volet **De base**, entrez ou sélectionnez l’abonnement, le groupe de ressources, le nom de la ressource de point de terminaison privé et les détails de la région, puis sélectionnez **suivant : Ressource**.
6. Dans le volet **Ressource**, définissez le **type de ressource** sur **Microsoft.Batch/batchAccounts**. Sélectionnez le compte Batch privé auquel vous souhaitez accéder, puis choisissez **Suivant : Configuration**.
   :::image type="content" source="media/private-connectivity/create-private-endpoint.png" alt-text="Créer un point de terminaison privé - volet Ressource":::
7. Dans le volet **Configuration**, entrez ou sélectionnez les informations suivantes :
   - **Réseau virtuel** : Sélectionnez votre réseau virtuel.
   - **Sous-réseau** : Sélectionnez votre sous-réseau.
   - **Intégrer à une zone DNS privée** :   Sélectionnez **Oui**. Pour vous connecter en privé à votre point de terminaison privé, vous avez besoin d’un enregistrement DNS. Nous vous recommandons d’intégrer votre point de terminaison privé à une zone DNS privée. Vous pouvez également utiliser vos propres serveurs DNS ou créer des enregistrements DNS à l’aide des fichiers hôtes sur vos machines virtuelles.
   - **Zone DNS privée** :  Sélectionnez privatelink.\<region\>.batch.azure.com. La zone DNS privée est déterminée automatiquement. Vous ne pouvez pas la modifier à l’aide du Portail Azure.
8. Sélectionnez **Vérifier + créer**, puis attendez qu’Azure valide votre configuration.
9. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

Une fois le point de terminaison privé approvisionné, vous pouvez accéder au compte Batch à partir de machines virtuelles dans le même réseau virtuel à l’aide du point de terminaison privé.

> [!IMPORTANT]
> L’exécution d’opérations en dehors du réseau virtuel sur lequel le point de terminaison privé est approvisionné entraîne un message « AuthorizationFailure » dans le portail Azure.

Pour afficher l’adresse IP à partir du portail Azure :

1. Sélectionnez **Toutes les ressources**.
2. Recherchez le point de terminaison privé que vous avez créé précédemment.
3. Sélectionnez l’onglet **Vue d’ensemble** pour afficher les paramètres DNS et les adresses IP.

:::image type="content" source="media/private-connectivity/access-private.png" alt-text="Paramètres DNS du point de terminaison privé et adresses IP":::

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Lorsque vous [créez un compte Batch à l’aide d’un modèle Azure Resource Manager](quick-create-template.md), modifiez le modèle pour définir **publicNetworkAccess** sur **Disabled** (désactivé), comme indiqué ci-dessous.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "batchAccountName": {
      "type": "string",
      },
    "location": {
      "type": "string",
    }
  },
  "resources": [
    {
      "name": "[parameters('batchAccountName')]",
      "type": "Microsoft.Batch/batchAccounts",
      "apiVersion": "2020-03-01-preview",
      "location": "[parameters('location')]",
      "dependsOn": []
      "properties": {
        "poolAllocationMode": "BatchService"
        "publicNetworkAccess": "Disabled"
      }
    }
  ]
}
```

## <a name="configure-dns-zones"></a>Configurer des zones DNS

Utilisez une [zone DNS privée](../dns/private-dns-privatednszone.md) dans le sous-réseau où vous avez créé le point de terminaison privé. Configurez les points de terminaison de façon à ce que chaque adresse IP privée soit mappée à une entrée DNS.

Lorsque vous créez le point de terminaison privé, vous pouvez intégrer celui-ci à une [zone DNS privée](../dns/private-dns-privatednszone.md) dans Azure. Si vous choisissez plutôt d’utiliser un [domaine personnalisé](../dns/dns-custom-domain.md), vous devez la configurer afin d’ajouter des enregistrements DNS pour toutes les adresses IP privées réservées au point de terminaison privé.

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les coûts liés aux points de terminaison privés, voir [Prix d’Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).

## <a name="current-limitations-and-best-practices"></a>Limites et meilleures pratiques actuelles

Lorsque vous créez votre compte Batch privé, gardez à l’esprit les points suivants :

- Les ressources de points de terminaison privés doivent être créées dans le même abonnement que le compte Batch.
- Pour supprimer la connexion privée, vous devez supprimer la ressource de point de terminaison privé.
- Une fois qu’un compte Batch a été créé avec un accès réseau public, vous ne pouvez pas le modifier en accès privé uniquement.
- Les enregistrements DNS dans la zone DNS privée ne sont pas supprimés automatiquement lorsque vous supprimez un point de terminaison privé ou une région du compte Batch. Vous devez supprimer manuellement les enregistrements DNS avant d’ajouter un nouveau point de terminaison privé lié à cette zone DNS privée. Si vous ne nettoyez pas les enregistrements DNS, des problèmes de plan de données inattendus peuvent se produire, tels que l’indisponibilité des données dans les régions ajoutées après la suppression du point de terminaison privé ou le retrait de la région.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer des pools Batch dans des réseaux virtuels](batch-virtual-network.md).
- Découvrir comment [créer des pools Batch avec des adresses IP publiques](batch-pool-no-public-ip-address.md)
- Découvrez comment [créer des pools Batch avec des adresses IP publiques spécifiées](create-pool-public-ip.md).
- En savoir plus sur [Azure Private Link](../private-link/private-link-overview.md).
