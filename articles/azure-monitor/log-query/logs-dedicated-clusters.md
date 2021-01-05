---
title: Clusters dédiés pour les journaux Azure Monitor
description: Les clients qui ingèrent plus de 1 To de données d’analyse peuvent utiliser des clusters dédiés plutôt que des clusters partagés.
ms.subservice: logs
ms.topic: conceptual
author: rboucher
ms.author: robb
ms.date: 09/16/2020
ms.openlocfilehash: d2446e866c0e12d50a0759373682f4f62bc4bba0
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512220"
---
# <a name="azure-monitor-logs-dedicated-clusters"></a>Clusters dédiés pour les journaux Azure Monitor

Les Clusters dédiés pour les journaux Azure Monitor sont une option de déploiement qui permet de bénéficier de fonctionnalités avancées pour les clients des journaux Azure Monitor. Les clients ayant des clusters dédiés peuvent choisir les espaces de travail à héberger sur ces clusters.

Les fonctionnalités qui nécessitent des clusters dédiés sont les suivantes :

- **[Clés gérées par le client](../platform/customer-managed-keys.md)**  : chiffrez les données du cluster à l’aide de clés fournies et contrôlées par le client.
- **[Lockbox](../platform/customer-managed-keys.md#customer-lockbox-preview)**  : les clients peuvent contrôler les demandes d’accès aux données par les ingénieurs du support technique Microsoft.
- Le **[double chiffrement](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption)** permet d’éviter un scénario impliquant une possible compromission d’un algorithme ou d’une clé de chiffrement. Dans ce cas, la couche de chiffrement supplémentaire continue de protéger vos données.
- **[Espaces de travail multiples](../log-query/cross-workspace-query.md)**  : si un client utilise plusieurs espaces de travail pour la production, il peut être judicieux d’utiliser un cluster dédié. Les requêtes entre espaces de travail s’exécuteront plus rapidement si tous les espaces de travail se trouvent sur le même cluster. Il peut être plus rentable d’utiliser un cluster dédié, car les niveaux de réservation de capacité affectés prennent en compte toutes les ingestions de cluster et s’appliquent à tous ses espaces de travail, même si certains d’entre eux sont petits et ne peuvent bénéficier de la remise de réservation de capacité.

Les clusters dédiés exigent des clients qu’ils s’engagent à utiliser une capacité d’au moins 1 To d’ingestion des données par jour. La migration vers un cluster dédié est simple. Il n’y a pas de perte de données ni d’interruption de service. 

> [!IMPORTANT]
> Les clusters dédiés sont approuvés et entièrement pris en charge pour les déploiements de production. Toutefois, en raison de contraintes temporaires de capacité, nous vous demandons de vous inscrire à l’avance pour pouvoir utiliser cette fonction. Utilisez vos contacts au sein de Microsoft pour fournir vos ID d’abonnement.

## <a name="management"></a>Gestion 

Les clusters dédiés sont gérés via une ressource Azure qui représente les clusters de journaux Azure Monitor. Toutes les opérations sont effectuées sur cette ressource à l’aide de PowerShell ou de l’API REST.

Une fois le cluster créé, vous pouvez le configurer et des espaces de travail peuvent lui être associés. Lorsqu’un espace de travail est lié à un cluster, les nouvelles données envoyées à l’espace de travail résident sur le cluster. Seuls les espaces de travail qui se trouvent dans la même région que le cluster peuvent être liés à celui-ci. Les espaces de travail peuvent être dissociés d’un cluster avec certaines limitations. Cet article contient plus de détails sur ces limitations. 

Les données ingérées dans les clusters dédiés sont chiffrées deux fois : une fois au niveau du service à l’aide de clés gérées par Microsoft ou d’[une clé gérée par le client](../platform/customer-managed-keys.md), et une fois au niveau de l’infrastructure à l’aide de deux algorithmes de chiffrement différents et de deux clés différentes. Le [double chiffrement](../../storage/common/storage-service-encryption.md#doubly-encrypt-data-with-infrastructure-encryption) permet d’éviter un scénario impliquant une possible compromission d’un algorithme ou d’une clé de chiffrement. Dans ce cas, la couche de chiffrement supplémentaire continue de protéger vos données. Le cluster dédié vous permet également de protéger vos données à l’aide du contrôle [Lockbox](../platform/customer-managed-keys.md#customer-lockbox-preview).

Toutes les opérations au niveau du cluster requièrent l’autorisation de l’action `Microsoft.OperationalInsights/clusters/write` sur le cluster. Cette autorisation peut être accordée via le propriétaire ou le contributeur qui contient l’action `*/write` ou via le rôle Contributeur Log Analytics qui contient l’action `Microsoft.OperationalInsights/*`. Pour plus d’informations sur les autorisations de Log Analytics, consultez [Gérer l’accès aux données de journal et aux espaces de travail dans Azure Monitor](../platform/manage-access.md). 


## <a name="cluster-pricing-model"></a>Modèle de tarification des clusters

Les clusters dédiés Log Analytics utilisent un modèle de tarification de réservation de capacité d’au moins 1 000 Go/jour. Toute utilisation au-dessus du niveau de réservation sera facturée au tarif de paiement à l’utilisation.  Les informations sur la tarification de réservation de capacité sont disponibles sur la [page de tarification Azure Monitor]( https://azure.microsoft.com/pricing/details/monitor/).  

Le niveau de réservation de la capacité du cluster est configuré par programmation avec Azure Resource Manager à l’aide du paramètre `Capacity` sous `Sku`. La `Capacity` est spécifiée en unités de Go et peut avoir des valeurs de 1 000 Go/jour ou plus par incréments de 100 Go/jour.

Il existe deux modes de facturation pour l’utilisation sur un cluster. Ils peuvent être spécifiés par le paramètre `billingType` lors de la configuration de votre cluster. 

1. **Cluster** : dans ce cas (valeur par défaut), la facturation des données ingérées est effectuée au niveau du cluster. Les quantités de données ingérées de chaque espace de travail associé à un cluster sont agrégées pour calculer la facture quotidienne du cluster. 

2. **Espaces de travail** : les coûts de réservation de capacité pour votre cluster sont attribués proportionnellement aux espaces de travail du cluster (après prise en compte des allocations par nœud depuis [Azure Security Center](../../security-center/index.yml) pour chaque espace de travail).

Si votre espace de travail utilise le niveau tarifaire Par nœud hérité, lorsqu’il est lié à un cluster, il sera facturé en fonction des données ingérées par rapport à la réservation de capacité du cluster, et non plus par nœud. Les allocations de données par nœud d’Azure Security Center continuent à être appliquées.

Vous trouverez plus d’informations sur la facturation des clusters dédiés Log Analytics [ici]( https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#log-analytics-dedicated-clusters).


## <a name="creating-a-cluster"></a>Création d’un cluster

Vous devez d’abord créer des ressources de cluster pour commencer à créer un cluster dédié.

Les propriétés suivantes doivent être spécifiées :

- **ClusterName** : Utilisée à des fins d’administration. Les utilisateurs ne sont pas exposés à ce nom.
- **ResourceGroupName** : Comme pour toutes les ressources Azure, les clusters appartiennent à un groupe de ressources. Nous vous recommandons d’utiliser un groupe central de ressources informatiques, car les clusters sont généralement partagés par de nombreuses équipes dans l’organisation. Pour plus d’informations sur les considérations de conception, consultez [Conception de votre déploiement de journaux Azure Monitor](../platform/design-logs-deployment.md).
- **Emplacement** : Un cluster se trouve dans une région Azure spécifique. Seuls les espaces de travail situés dans cette région peuvent être liés à ce cluster.
- **SkuCapacity** : Vous devez spécifier le niveau (SKU) de *réservation de capacité* lors de la création d’une ressource de *cluster*. Le niveau de *réservation de capacité* peut aller de 1 000 à 3 000 Go par jour. Le cas échéant, vous pouvez ultérieurement le mettre à jour par paliers de 100. Si vous avez besoin d’un niveau de réservation de capacité supérieur à 3 000 Go par jour, contactez-nous à l’adresse LAIngestionRate@microsoft.com. Pour plus d’informations sur les coûts de cluster, consultez [Gérer les coûts des clusters Log Analytics](../platform/manage-cost-storage.md#log-analytics-dedicated-clusters).

Après avoir créé votre ressource de *cluster*, vous pouvez modifier des propriétés supplémentaires telles que *SKU*, *keyVaultProperties ou *billingType*. Pour plus d’informations, voir ci-dessous.

> [!WARNING]
> La création du cluster déclenche l’allocation et l’approvisionnement de la ressource. Cette opération peut prendre une heure. Il est recommandé de l’exécuter de manière asynchrone.

Le compte d’utilisateur qui crée les clusters doit disposer de l’autorisation standard de création de ressources Azure `Microsoft.Resources/deployments/*` et de l’autorisation d’écriture sur le cluster `(Microsoft.OperationalInsights/clusters/write)`.

### <a name="create"></a>Créer 

**PowerShell**

```powershell
New-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} -ClusterName {cluster-name} -Location {region-name} -SkuCapacity {daily-ingestion-gigabyte} -AsJob

# Check when the job is done
Get-Job -Command "New-AzOperationalInsightsCluster*" | Format-List -Property *
```

**REST**

*Appeler* 
```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-03-01-preview
Authorization: Bearer <token>
Content-type: application/json

{
  "identity": {
    "type": "systemAssigned"
    },
  "sku": {
    "name": "capacityReservation",
    "Capacity": 1000
    },
  "properties": {
    "billingType": "cluster",
    },
  "location": "<region-name>",
}
```

*Réponse*

Doit être 200 OK et un en-tête.

### <a name="check-provisioning-status"></a>Vérifier l’état du provisionnement

L’approvisionnement du cluster Log Analytics prend un certain temps. Vous pouvez vérifier l’état d’approvisionnement de plusieurs façons :

- Exécutez la commande PowerShell Get-AzOperationalInsightsCluster avec le nom du groupe de ressources et vérifiez la propriété ProvisioningState. La valeur indique *ProvisioningAccount* pendant l’approvisionnement et *Succeeded* une fois celui-ci terminé.
  ```powershell
  New-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} 
  ```

- Copiez la valeur de l’URL Azure-AsyncOperation à partir de la réponse et suivez les instructions decontrôle de l’état des opérations asynchrones.

- Envoyez une requête GET sur la ressource *cluster*, puis examinez la valeur *provisioningState*. La valeur indique *ProvisioningAccount* pendant l’approvisionnement et *Succeeded* une fois celui-ci terminé.

   ```rst
   GET https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-03-01-preview
   Authorization: Bearer <token>
   ```

   **Réponse**

   ```json
   {
     "identity": {
       "type": "SystemAssigned",
       "tenantId": "tenant-id",
       "principalId": "principal-id"
       },
     "sku": {
       "name": "capacityReservation",
       "capacity": 1000,
       "lastSkuUpdate": "Sun, 22 Mar 2020 15:39:29 GMT"
       },
     "properties": {
       "provisioningState": "ProvisioningAccount",
       "billingType": "cluster",
       "clusterId": "cluster-id"
       },
     "id": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name",
     "name": "cluster-name",
     "type": "Microsoft.OperationalInsights/clusters",
     "location": "region-name"
   }
   ```

Le GUID *principalId* est généré par le service d’identité managée pour la ressource de *cluster*.

## <a name="change-cluster-properties"></a>Modifier les propriétés du cluster

Une fois que vous avez créé votre ressource de *cluster* et qu’elle est entièrement approvisionnée, vous pouvez modifier des propriétés supplémentaires au niveau du cluster à l’aide de PowerShell ou de l’API REST. Outre les propriétés disponibles lors de la création du cluster, d’autres propriétés ne peuvent être définies qu’après l’approvisionnement du cluster :

- **keyVaultProperties** : Utilisée pour configurer le compte Azure Key Vault utilisé pour approvisionner une [clé gérée par le client Azure Monitor](../platform/customer-managed-keys.md#customer-managed-key-provisioning-procedure). Il contient les paramètres suivants :  *KeyVaultUri*, *KeyName*, *KeyVersion*. 
- **billingType** : La propriété *billingType* détermine l’attribution de facturation pour la ressource de *cluster* et ses données :
  - **Cluster** (par défaut) : Les coûts de la réservation de capacité pour votre cluster sont attribués à la ressource de *cluster*.
  - **Espaces de travail** : Les coûts de la réservation de capacité pour votre cluster sont attribués proportionnellement aux espaces de travail du cluster. Une partie de l’utilisation est facturée à la ressource de *cluster* si le total des données ingérées pour la journée est inférieur à la réservation de capacité. Pour en savoir plus sur le modèle de tarification du cluster, consultez [Clusters dédiés Log Analytics](../platform/manage-cost-storage.md#log-analytics-dedicated-clusters). 

> [!NOTE]
> La propriété *billingType* n’est pas prise en charge dans PowerShell.
> Les mises à jour des propriétés du cluster peuvent être exécutées de façon asynchrone et peuvent prendre un certain temps.

**PowerShell**

```powershell
Update-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} -ClusterName {cluster-name} -KeyVaultUri {key-uri} -KeyName {key-name} -KeyVersion {key-version}
```

**REST**

> [!NOTE]
> Vous pouvez mettre à jour les valeurs *sku*, *keyVaultProperties* ou *billingType* de la ressource *Cluster* à l’aide de l’instruction PATCH.

Par exemple : 

*Appeler*

```rst
PATCH https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-03-01-preview
Authorization: Bearer <token>
Content-type: application/json

{
   "sku": {
     "name": "capacityReservation",
     "capacity": <capacity-reservation-amount-in-GB>
     },
   "properties": {
    "billingType": "cluster",
     "KeyVaultProperties": {
       "KeyVaultUri": "https://<key-vault-name>.vault.azure.net",
       "KeyName": "<key-name>",
       "KeyVersion": "<current-version>"
       }
   },
   "location":"<region-name>"
}
```

« KeyVaultProperties » contient les détails de l'identificateur de clé Key Vault.

*Réponse*

200 OK et en-tête

### <a name="check-cluster-update-status"></a>Vérifier l’état de mise à jour du cluster

La propagation de l’identificateur de clé prend quelques minutes. Vous pouvez vérifier l’état de la mise à jour de deux manières :

- Copiez la valeur de l’URL Azure-AsyncOperation à partir de la réponse et suivez les instructions decontrôle de l’état des opérations asynchrones. 

   OR

- Envoyez une requête GET sur la ressource *cluster*, puis examinez les propriétés *KeyVaultProperties*. Les détails de l’identificateur de clé récemment mis à jour doivent être retournés dans la réponse.

   Une réponse à la demande de récupération sur la ressource de *cluster* doit ressembler à ceci lorsque la mise à jour de l’identificateur de clé est terminée :

   ```json
   {
     "identity": {
       "type": "SystemAssigned",
       "tenantId": "tenant-id",
       "principalId": "principle-id"
       },
     "sku": {
       "name": "capacityReservation",
       "capacity": 1000,
       "lastSkuUpdate": "Sun, 22 Mar 2020 15:39:29 GMT"
       },
     "properties": {
       "keyVaultProperties": {
         "keyVaultUri": "https://key-vault-name.vault.azure.net",
         "kyName": "key-name",
         "keyVersion": "current-version"
         },
        "provisioningState": "Succeeded",
       "billingType": "cluster",
       "clusterId": "cluster-id"
     },
     "id": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name",
     "name": "cluster-name",
     "type": "Microsoft.OperationalInsights/clusters",
     "location": "region-name"
  }
  ```

## <a name="link-a-workspace-to-the-cluster"></a>Lier un espace de travail au cluster

Lorsqu’un espace de travail est lié à un cluster dédié, les nouvelles données qui sont ingérées dans l’espace de travail sont acheminées vers le nouveau cluster alors que les données existantes restent sur le cluster existant. Si le cluster dédié est chiffré à l’aide de clés gérées par le client (CMK), seules les nouvelles données sont chiffrées avec la clé. Le système extrait cette différence des utilisateurs et ces derniers n’ont qu’à interroger l’espace de travail comme d’habitude, tandis que le système exécute des requêtes entre clusters sur le serveur principal.

Un cluster peut être lié à jusqu’à 100 espaces de travail. Les espaces de travail liés se trouvent dans la même région que le cluster. Pour protéger le serveur principal système et éviter la fragmentation des données, un espace de travail ne peut pas être lié à un cluster plus de deux fois par mois.

Pour effectuer l’opération de liaison, vous devez disposer des autorisations « d’écriture » pour l’espace de travail et la ressource de *cluster* :

- Dans l’espace de travail : *Microsoft.OperationalInsights/workspaces/write*
- Dans la ressource de *cluster* : *Microsoft.OperationalInsights/clusters/write*

Outre les aspects de facturation, l’espace de travail lié conserve ses propres paramètres, tels que la durée de conservation des données.
L’espace de travail et le cluster peuvent se trouver dans des abonnements différents. L’espace de travail et le cluster peuvent se trouver dans différents locataires si Azure Lighthouse est utilisé pour les mapper à un seul locataire.

Comme n’importe quelle opération de cluster, la liaison d’un espace de travail ne peut être accomplie qu’une fois l’approvisionnement du cluster Log Analytics terminé.

> [!WARNING]
> La liaison d’un espace de travail à un cluster requiert la synchronisation de plusieurs composants principaux et la l’alimentation du cache. Cette opération peut prendre jusqu’à deux heures. Nous vous recommandons de l’exécuter de manière asynchrone.


**PowerShell**

Utilisez la commande PowerShell suivante pour le relier à un cluster :

```powershell
# Find cluster resource ID
$clusterResourceId = (Get-AzOperationalInsightsCluster -ResourceGroupName {resource-group-name} -ClusterName {cluster-name}).id

# Link the workspace to the cluster
Set-AzOperationalInsightsLinkedService -ResourceGroupName {resource-group-name} -WorkspaceName {workspace-name} -LinkedServiceName cluster -WriteAccessResourceId $clusterResourceId -AsJob

# Check when the job is done
Get-Job -Command "Set-AzOperationalInsightsLinkedService" | Format-List -Property *
```


**REST**

Utilisez l’appel REST suivant pour le relier à un cluster :

*Envoi*

```rst
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>/linkedservices/cluster?api-version=2020-03-01-preview 
Authorization: Bearer <token>
Content-type: application/json

{
  "properties": {
    "WriteAccessResourceId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/clusters/<cluster-name>"
    }
}
```

*Réponse*

200 OK et en-tête.

### <a name="using-customer-managed-keys-with-linking"></a>Utilisation de clés gérées par le client avec la liaison

Si vous utilisez des clés gérées par le client, les données ingérées sont stockées chiffrées avec votre clé gérée après une opération d’association qui peut prendre jusqu’à 90 minutes. 

Vous pouvez vérifier l’état de l’association d’espace de travail de deux manières :

- Copiez la valeur de l’URL Azure-AsyncOperation à partir de la réponse et suivez les instructions decontrôle de l’état des opérations asynchrones.

- Envoyez une requête [Workspaces – Get](/rest/api/loganalytics/workspaces/get) et observez la réponse. L’espace de travail associé possède un clusterResourceId sous « fonctionnalités ».

Une requête d’envoi ressemble à ceci :

*Envoi*

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalInsights/workspaces/<workspace-name>?api-version=2020-03-01-preview
Authorization: Bearer <token>
```

*Réponse*

```json
{
  "properties": {
    "source": "Azure",
    "customerId": "workspace-name",
    "provisioningState": "Succeeded",
    "sku": {
      "name": "pricing-tier-name",
      "lastSkuUpdate": "Tue, 28 Jan 2020 12:26:30 GMT"
    },
    "retentionInDays": 31,
    "features": {
      "legacy": 0,
      "searchVersion": 1,
      "enableLogAccessUsingOnlyResourcePermissions": true,
      "clusterResourceId": "/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.OperationalInsights/clusters/cluster-name"
    },
    "workspaceCapping": {
      "dailyQuotaGb": -1.0,
      "quotaNextResetTime": "Tue, 28 Jan 2020 14:00:00 GMT",
      "dataIngestionStatus": "RespectQuota"
    }
  },
  "id": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/microsoft.operationalinsights/workspaces/workspace-name",
  "name": "workspace-name",
  "type": "Microsoft.OperationalInsights/workspaces",
  "location": "region-name"
}
```

## <a name="unlink-a-workspace-from-a-dedicated-cluster"></a>Dissocier un espace de travail d’un cluster dédié

Vous pouvez dissocier un espace de travail d’un cluster. Après la dissociation d’un espace de travail du cluster, les nouvelles données associées à cet espace de travail ne sont pas envoyées au cluster dédié. En outre, la facturation de l’espace de travail ne se fait plus via le cluster. Les anciennes données de l’espace de travail dissocié peuvent être conservées sur le cluster. Si ces données sont chiffrées à l’aide de clés gérées par le client (CMK), les secrets Key Vault sont conservés. Le système extrait cette modification des utilisateurs de Log Analytics. Les utilisateurs peuvent simplement interroger l’espace de travail comme d’habitude. Le système exécute des requêtes entre clusters sur le serveur principal le cas échéant, sans aucune indication aux utilisateurs.  

> [!WARNING] 
> Il existe une limite de deux opérations de liaison par espace de travail dans un délai d’un mois. Prenez le temps de prendre en compte et de planifier les actions de dissociation en conséquence. 

## <a name="delete-a-dedicated-cluster"></a>Supprimer un cluster dédié

Une ressource de cluster dédiée peut être supprimée. Vous devez dissocier tous les espaces de travail du cluster avant de le supprimer. Pour effectuer cette opération, vous devez disposer des autorisations d’écriture sur la ressource *cluster*. 

Une fois la ressource de cluster supprimée, le cluster physique entre dans un processus de vidage et de suppression. La suppression d’un cluster entraîne la suppression de toutes les données qui ont été stockées sur le cluster. Les données peuvent provenir d’espaces de travail qui ont été reliés au cluster par le passé.

Une ressource *cluster* supprimée au cours des 14 derniers jours est dans un état de suppression réversible. Il est possible de la récupérer avec ses données. Étant donné que tous les espaces de travail ont été dissociés de la ressource de *cluster* suite à la suppression de celle-ci, vous devez réassocier vos espaces de travail après la récupération. L’opération de récupération ne peut pas être effectuée par l’utilisateur. Contactez votre canal Microsoft ou le support technique pour toute demande de récupération.

Dans les 14 jours qui suivent la suppression, le nom de la ressource de cluster est réservé et ne peut pas être utilisé par d’autres ressources.

**PowerShell**

Utilisez la commande PowerShell suivante pour supprimer un cluster :

  ```powershell
  Remove-AzOperationalInsightsCluster -ResourceGroupName "resource-group-name" -ClusterName "cluster-name"
  ```

**REST**

Utilisez l’appel REST suivant pour supprimer un cluster :

  ```rst
  DELETE https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/clusters/<cluster-name>?api-version=2020-08-01
  Authorization: Bearer <token>
  ```

  **Réponse**

  200 OK



## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [facturation des clusters dédiés Log Analytics](../platform/manage-cost-storage.md#log-analytics-dedicated-clusters)
- En savoir plus sur la [conception appropriée des espaces de travail Log Analytics](../platform/design-logs-deployment.md)
