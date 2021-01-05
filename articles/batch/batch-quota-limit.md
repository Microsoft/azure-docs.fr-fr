---
title: Quotas et limites du service
description: En savoir plus sur les contraintes, les limites et les quotas par défaut d’Azure Batch, et comment demander une augmentation de quota
ms.topic: conceptual
ms.date: 06/03/2020
ms.custom: seodec18
ms.openlocfilehash: b2039794a0c8a13070c9d81b83869ca4097bd02e
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325964"
---
# <a name="batch-service-quotas-and-limits"></a>Quotas et limites du service Batch

Comme avec d’autres services Azure, il existe des limites concernant certaines ressources associées au service Batch. La plupart de ces limites représentent des quotas par défaut appliqués par Azure au niveau de l’abonnement ou du compte.

Gardez ces quotas à l’esprit quand vous concevez et que vous augmentez vos charges de travail Batch. Par exemple, si votre pool n’atteint pas le nombre cible de nœuds de calcul que vous avez spécifié, vous avez peut-être atteint la limite du quota de cœurs de votre compte Batch.

Vous pouvez exécuter plusieurs charges de travail Batch dans un compte Batch ou répartir vos charges de travail entre plusieurs comptes Batch se trouvant dans le même abonnement mais dans différentes régions Azure.

Si vous envisagez d’exécuter des charges de travail de production dans Batch, vous devrez peut-être affecter à un ou plusieurs des quotas une valeur supérieure à la valeur par défaut. Si vous souhaitez augmenter un quota, vous pouvez ouvrir une [demande de service clientèle](#increase-a-quota) en ligne gratuitement.

## <a name="resource-quotas"></a>Quotas de ressources

Un quota est une limite, pas une garantie de capacité. Si vous avez des besoins de capacité à grande échelle, contactez le support Azure.

Notez également que les quotas ne sont pas des valeurs garanties. Les quotas peuvent varier en fonction des modifications du service Batch ou d’une demande d’utilisateur visant à modifier une valeur de quota.

[!INCLUDE [azure-batch-limits](../../includes/azure-batch-limits.md)]

## <a name="core-quotas"></a>Quota de cœurs

### <a name="cores-quotas-in-batch-service-mode"></a>Quotas de cœurs en mode de service Batch

L’application de quotas de cœurs dédiés est améliorée, les modifications étant mises à disposition par étapes et finalisées pour tous les comptes Batch d’ici à la fin décembre 2020.

Des quotas de cœurs existent pour chaque série de machines virtuelles prises en charge par Batch et s’affichent sur la page **Quotas** dans le portail. Les limites de quota de la série de machines virtuelles peuvent être mises à jour avec une demande de support, comme indiqué ci-dessous.

Une fois le mécanisme existant supprimé, les limites de quota pour les séries de machines virtuelles ne sont pas vérifiées, et seule la limite totale du quota pour le compte est appliquée. Cela signifie qu’il est possible d’allouer plus de cœurs pour une série de machines virtuelles que ce qui est indiqué par le quota de la série de machines virtuelles, jusqu’à la limite totale de quota du compte.

Le mécanisme mis à jour appliquera les quotas de la série de machines virtuelles, en plus du quota total du compte. Dans le cadre de la transition vers le nouveau mécanisme, les valeurs de quota de la série de machines virtuelles peuvent être mises à jour pour éviter les échecs d’allocation : les séries de machines virtuelles utilisées au cours des derniers mois afficheront donc un quota de série de machines virtuelles mis à jour pour correspondre au quota total du compte. Cette modification n’autorise pas l’utilisation d’une capacité supérieure à celle déjà disponible.

Il est possible de déterminer si l’application de quota de la série de machines virtuelles a été activée pour un compte Batch en vérifiant les éléments suivants :

* La propriété d’API [dedicatedCoreQuotaPerVMFamilyEnforced](/rest/api/batchmanagement/batchaccount/get#batchaccount) du compte Batch.

* Texte sur la page **Quotas** du compte Batch dans le portail.

### <a name="cores-quotas-in-user-subscription-mode"></a>Quotas de cœurs en mode Abonnement utilisateur

Si vous avez créé un [compte Batch](accounts.md) en mode d’allocation de pool défini sur **abonnement utilisateur**, les machines virtuelles Batch et les autres ressources, telles que les comptes de stockage, sont créées directement dans votre abonnement lors de la création d’un pool. Les quotas de cœurs Azure Batch ne s’appliquent pas et les quotas de votre abonnement pour les cœurs de calcul régionaux, les cœurs de calcul par série et d’autres ressources sont utilisés et appliqués.

Pour en savoir plus sur ces quotas, consultez [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="pool-size-limits"></a>Limites de taille de pool

Des limites de taille de pool sont définies par le service Batch. Contrairement aux [quotas des ressources](#resource-quotas), ces valeurs ne peuvent pas être modifiées. Seuls les pools présentant une communication entre nœuds et des images personnalisées possèdent des restrictions différentes du quota standard.

| **Ressource** | **Limite maximale** |
| --- | --- |
| **Nœuds de calcul dans un [pool prenant en charge la communication entre nœuds](batch-mpi.md)**  ||
| Mode d’allocation de pool du service Batch | 100 |
| Mode d’allocation de pool de l'abonnement Batch | 80 |
| **Nœuds de calcul dans un [pool créé avec une ressource d’image managée](batch-custom-images.md)** <sup>1</sup> ||
| Nœuds dédiés | 2000 |
| Nœuds de faible priorité | 1 000 |

<sup>1</sup> Pour les pools ne prenant pas en charge la communication entre nœuds.

## <a name="other-limits"></a>Autres limites

Limites supplémentaires définies par le service Batch. Contrairement aux [quotas des ressources](#resource-quotas), ces valeurs ne peuvent pas être modifiées.

| **Ressource** | **Limite maximale** |
| --- | --- |
| [Tâches simultanées](batch-parallel-node-tasks.md) par nœud de calcul | 4 x nombre de cœurs de nœud |
| [Applications](batch-application-packages.md) par compte Batch | 20 |
| Packages d’applications par application | 40 |
| Packages d’applications par pool | 10 |
| Durée de vie maximale de la tâche | 180 jours<sup>1</sup> |
| [Montages](virtual-file-mount.md) par nœud de calcul | 10 |

<sup>1</sup> La durée de vie maximale d’une tâche, entre le moment où elle est ajoutée au travail et la fin de son exécution, est de 180 jours. Les tâches terminées sont conservées pendant sept jours. Les données de tâches non terminées pendant la durée de vie maximale ne sont pas accessibles.

## <a name="view-batch-quotas"></a>Afficher les quotas Batch

Pour voir vos quotas de compte Batch dans le [portail Azure](https://portal.azure.com) :

1. Sélectionnez **Comptes Batch**, puis le compte Batch qui vous intéresse.
1. Sélectionnez **Quotas** dans le menu du compte Batch.
1. Affichez les quotas actuellement appliqués au compte Batch.

:::image type="content" source="./media/batch-quota-limit/account-quota-portal.png" alt-text="Quotas de compte Batch":::

## <a name="increase-a-quota"></a>Augmenter un quota

Vous pouvez demander une augmentation de quota pour votre compte Batch ou votre abonnement à l’aide du [portail Azure](https://portal.azure.com). Le type d’augmentation de quota varie selon le mode d’allocation de pool de votre compte Batch. Pour demander une augmentation de quota, vous devez inclure la série de machines virtuelles dont vous souhaitez augmenter le quota. Lorsque l’augmentation de quota est appliquée, elle l’est pour toutes les séries de machines virtuelles.

1. Sélectionnez la mosaïque **Aide + Support** dans le tableau de bord du portail, ou le point d’interrogation ( **?** ) dans le coin supérieur droit du portail.
1. Sélectionnez **Nouvelle demande de support** > **De base**.
1. Dans **De base** :
   
    1. **Type de problème** > **Limites du service et des abonnements (quotas)**
   
    1. Sélectionnez votre abonnement.
   
    1. **Type de quota** > **Batch**
      
       Sélectionnez **Suivant**.
    
1. Dans **Détails** :
      
    1. Dans **Fournir des détails**, spécifiez l’emplacement, le type de quota et le compte Batch.
    
       ![Augmentation de quota Batch][quota_increase]

       Types de quotas :

       * **Par compte Batch**  
         Valeurs propres à un seul et même compte Batch, incluant les cœurs dédiés et basse priorité ainsi que le nombre de travaux et de pools.
        
       * **Par région**  
         Valeurs qui s’appliquent à tous les comptes Batch d’une région et qui incluent le nombre de comptes Batch par région et par abonnement.

       Un quota basse priorité est une valeur unique dans toutes les séries de machines virtuelles. Si vous avez besoin de références SKU avec contraintes, vous devez sélectionner **Cœurs basse priorité** et inclure les familles de machines virtuelles à la demande.

    1. Sélectionnez un **niveau de gravité** en fonction de [l’impact sur votre activité](https://aka.ms/supportseverity).

       Sélectionnez **Suivant**.

1. Dans **Informations de contact** :
   
    1. Sélectionnez une **méthode de contact préférée**.
   
    1. Vérifiez et entrez les informations de contact requises.
   
       Sélectionnez **Créer** pour envoyer la demande de support.

Une fois que vous avez envoyé votre demande de support, le support Azure vous contactera. Les demandes de quota peuvent être effectuées en quelques minutes ou jusqu’à deux jours ouvrables.

## <a name="related-quotas-for-vm-pools"></a>Quotas associés pour les pools de machines virtuelles

Les pools Batch dans la configuration de machine virtuelle déployés dans un réseau virtuel Azure allouent automatiquement des ressources de mise en réseau Azure supplémentaires. Les ressources suivantes sont nécessaires pour chacun des 50 nœuds de pools dans un réseau virtuel :

- Un seul [groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md#network-security-groups)
- Une seule [adresse IP publique](../virtual-network/public-ip-addresses.md)
- Un seul [équilibreur de charge](../load-balancer/load-balancer-overview.md)

Ces ressources sont allouées dans l’abonnement contenant le réseau virtuel fourni lors de la création du pool Batch. Ces ressources sont limitées par les [quotas de ressources](../azure-resource-manager/management/azure-subscription-service-limits.md) de l’abonnement. Si vous planifiez des déploiements de pools de grande taille dans un réseau virtuel, vérifiez les quotas de l’abonnement pour ces ressources. Si nécessaire, demandez une augmentation sur le portail Azure en sélectionnant **Aide et support**.

## <a name="next-steps"></a>Étapes suivantes

* [Créer un compte Azure Batch à l’aide du portail Azure](batch-account-create-portal.md).
* Apprenez-en davantage sur le [flux de travail et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
* En savoir plus sur [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md).

[account_quotas]: ./media/batch-quota-limit/accountquota_portal.png
[quota_increase]: ./media/batch-quota-limit/quota-increase.png