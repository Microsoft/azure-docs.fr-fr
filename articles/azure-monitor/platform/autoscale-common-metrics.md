---
title: Métriques courantes pour la mise à l’échelle automatique
description: Découvrez les métriques utilisées pour la mise à l’échelle automatique de vos instances Cloud Services, Virtual Machines et Web Apps.
ms.topic: conceptual
ms.date: 12/6/2016
ms.subservice: autoscale
ms.openlocfilehash: e1a77fc1b40faca0a339c5e1aaceb71dec8de8bd
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004634"
---
# <a name="azure-monitor-autoscaling-common-metrics"></a>Métriques courantes pour la mise à l’échelle automatique d’Azure Monitor

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

La fonction de mise à l’échelle automatique d’Azure Monitor vous permet de diminuer ou d’augmenter la taille des instances en fonction des données de télémétrie (métriques). Ce document décrit les métriques courantes que vous pouvez utiliser. Dans le portail Azure, vous pouvez choisir les métriques de la ressource à mettre à l’échelle. Toutefois, vous pouvez également choisir des métriques à partir d’une autre ressource à mettre à l’échelle.

La mise à l’échelle automatique Azure Monitor s’applique uniquement aux [groupes de machines virtuelles identiques](https://azure.microsoft.com/services/virtual-machine-scale-sets/), aux [services cloud](https://azure.microsoft.com/services/cloud-services/), à [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) et aux [services de gestion des API](../../api-management/api-management-key-concepts.md). Les autres services Azure utilisent des méthodes de mise à l’échelle différentes.

## <a name="compute-metrics-for-resource-manager-based-vms"></a>Calcul des métriques pour les machines virtuelles basées sur Resource Manager
Par défaut, les machines virtuelles et jeux de mise à l’échelle de machine virtuelle basés sur Resource Manager émettent des métriques de base (niveau hôte). En outre, lorsque vous configurez la collecte de données de diagnostic pour une machine virtuelle ou des jeux de mise à l’échelle de machine virtuelle Azure, l’extension de diagnostic Azure émet également les compteurs de performances du système d’exploitation invité (communément appelés « Métriques de système d’exploitation invité »).  Vous utilisez toutes ces métriques dans les règles de mise à l’échelle automatique.

Vous pouvez utiliser l’API/PoSH/CLI `Get MetricDefinitions` pour afficher les métriques disponibles pour votre ressource VMSS.

Si vous utilisez des jeux de mise à l’échelle de machine virtuelle et qu’une métrique que vous cherchez n’est pas répertoriée, il est probable qu’elle soit *désactivée* dans votre extension de diagnostics.

Si une métrique particulière n’est pas en cours d’échantillonnage ou de transfert vers la fréquence souhaitée, vous pouvez mettre à jour la configuration des diagnostics.

Si l’un des deux cas ci-dessus s’applique, consultez la page [Utiliser PowerShell pour activer Diagnostics Azure sur une machine virtuelle exécutant Windows](../../virtual-machines/extensions/diagnostics-windows.md) pour savoir comment utiliser PowerShell pour configurer et mettre à jour votre extension de diagnostics de machine virtuelle Windows Azure afin d’activer la métrique. Cet article inclut également un exemple de fichier de configuration de diagnostics.

### <a name="host-metrics-for-resource-manager-based-windows-and-linux-vms"></a>Métriques de l’hôte pour les machines virtuelles Windows et Linux basées sur Resource Manager
Les métriques de niveau hôte suivantes sont émies par défaut pour les machines virtuelles et jeux de mise à l’échelle de machine virtuelle Azure dans les instances Windows et Linux. Ces métriques décrivent votre machine virtuelle Azure, mais sont collectées à partir de l’hôte de la machine Virtuelle Azure plutôt que via l’agent installé sur la machine virtuelle invitée. Vous pouvez utiliser ces métriques dans les règles de mise à l’échelle automatique.

- [Métriques de l’hôte pour les machines virtuelles Windows et Linux basées sur Resource Manager](./metrics-supported.md#microsoftcomputevirtualmachines)
- [Métriques de l’hôte pour les jeux de mise à l’échelle de machine virtuelle Windows et Linux basées sur Resource Manager](./metrics-supported.md#microsoftcomputevirtualmachinescalesets)

### <a name="guest-os-metrics-for-resource-manager-based-windows-vms"></a>Métriques de système d’exploitation invité pour les machines virtuelles Windows basées sur Resource Manager
Lorsque vous créez une machine virtuelle dans Azure, les diagnostics sont effectués grâce à l’extension Diagnostics. L’extension de diagnostics émet un ensemble de métriques extraites de la machine virtuelle. Cela signifie que vous pouvez automatiser la mise à l’échelle des métriques qui ne sont pas émises par défaut.

Vous pouvez utiliser la commande suivante dans PowerShell pour générer une liste des métriques.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

Vous pouvez créer une alerte pour les métriques suivantes :

| Nom de métrique | Unité |
| --- | --- |
| \Processor(_Total)\% temps processeur |Pourcentage |
| \Processor(_Total)\\% temps privilégié |Pourcentage |
| \Processor(_Total)\\% temps utilisateur |Pourcentage |
| \Processor Information(_Total)\Fréquence du processeur |Count |
| \System\Processus |Count |
| \Process(_Total)\Nombre de threads |Count |
| \Process(_Total)\Nombre de handles |Count |
| \Memory\\% octets validés en cours d’utilisation |Pourcentage |
| \Memory\Octets disponibles |Octets |
| \Memory\Octets validés |Octets |
| \Memory\Limite de mémoire dédiée |Octets |
| \Memory\Octets de réserve paginée |Octets |
| \Memory\Octets de réserve non paginée |Octets |
| \PhysicalDisk(_Total)\\% temps disque |Pourcentage |
| \PhysicalDisk(_Total)\\% temps de lecture du disque |Pourcentage |
| \PhysicalDisk(_Total)\\% temps écriture du disque |Pourcentage |
| \PhysicalDisk(_Total)\Disk Transfers/sec |CountPerSecond |
| \PhysicalDisk(_Total)\Lectures disque/s |CountPerSecond |
| \PhysicalDisk(_Total)\Écritures disque/s |CountPerSecond |
| \PhysicalDisk(_Total)\Octets disque/s |BytesPerSecond |
| \PhysicalDisk(_Total)\Lectures disque, octets/s |BytesPerSecond |
| \PhysicalDisk(_Total)\Écritures disque, octets/s |BytesPerSecond |
| \PhysicalDisk(_Total)\Longueur moyenne Longueur de file d'attente de disque |Count |
| \PhysicalDisk(_Total)\Longueur moyenne de file d’attente lecture disque |Count |
| \PhysicalDisk(_Total)\Longueur moyenne de file d’attente écriture disque |Count |
| \LogicalDisk(_Total)\\% espace libre |Pourcentage |
| \LogicalDisk(_Total)\Mégaoctets libres |Count |

### <a name="guest-os-metrics-linux-vms"></a>Métriques de SE invité pour les machines virtuelles Linux
Lorsque vous créez une machine virtuelle dans Azure, les diagnostics sont activés par défaut grâce à l’extension Diagnostics.

Vous pouvez utiliser la commande suivante dans PowerShell pour générer une liste des métriques.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

 Vous pouvez créer une alerte pour les métriques suivantes :

| Nom de métrique | Unité |
| --- | --- |
| \Mémoire\Mémoire disponible |Octets |
| \Mémoire\Pourcentage de mémoire disponible |Pourcentage |
| \Mémoire\Mémoire utilisée |Octets |
| \Mémoire\Pourcentage de mémoire utilisée |Pourcentage |
| \Mémoire\Pourcentage de mémoire utilisée par le cache |Pourcentage |
| \Mémoire\Pages par seconde |CountPerSecond |
| \Mémoire\Pages lues par seconde |CountPerSecond |
| \Mémoire\Pages écrites par seconde |CountPerSecond |
| \Mémoire\Échanges disponibles |Octets |
| \Mémoire\Pourcentage d’échanges disponibles |Pourcentage |
| \Mémoire\Échanges utilisés |Octets |
| \Mémoire\Pourcentage d’échanges utilisés |Pourcentage |
| \Processeur\Pourcentage de temps d’inactivité |Pourcentage |
| \Processeur\Pourcentage de temps d’utilisateur |Pourcentage |
| \Processeur\Pourcentage de temps d’efficacité |Pourcentage |
| \Processeur\Pourcentage de temps privilégié |Pourcentage |
| \Processeur\Pourcentage de temps d’interruption |Pourcentage |
| \Processeur\Pourcentage de temps d’appels de procédure différés (DPC) |Pourcentage |
| \Processeur\Pourcentage de temps de processeur |Pourcentage |
| \Processeur\Pourcentage de temps d’attente |Pourcentage |
| \Disque physique\Octets par seconde |BytesPerSecond |
| \Disque physique\Octets lus par seconde |BytesPerSecond |
| \Disque physique\Octets écrits par seconde |BytesPerSecond |
| \Disque physique\Transferts par seconde |CountPerSecond |
| \Disque physique\Lectures par seconde |CountPerSecond |
| \Disque physique\Écritures par seconde |CountPerSecond |
| \Disque physique\Temps de lecture moyen |Secondes |
| \Disque physique\Temps d’écriture moyen |Secondes |
| \Disque physique\Temps de transfert moyen |Secondes |
| \Disque physique\Longueur moyenne de la file d’attente du disque |Count |
| \Interface réseau\Octets transmis |Octets |
| \Interface réseau\Octets reçus |Octets |
| \Interface réseau\Paquets transmis |Count |
| \Interface réseau\Paquets reçus |Count |
| \Interface réseau\Total des octets |Octets |
| \Interface réseau\Total des erreurs Rx |Count |
| \Interface réseau\Total des erreurs Tx |Count |
| \Interface réseau\Total des collisions |Count |

## <a name="commonly-used-app-service-server-farm-metrics"></a>Métriques App Service (batterie de serveurs) couramment utilisées
Vous pouvez également effectuer la mise à l’échelle en fonction des métriques de serveur web courantes, telles que la longueur de file d’attente HTTP. Son nom de métrique est **HttpQueueLength**.  La section suivante liste les métriques de batterie de serveurs (App Service) disponibles.

### <a name="web-apps-metrics"></a>Métriques Web Apps
Vous pouvez utiliser la commande suivante dans PowerShell pour générer une liste des métriques Web Apps.

```
Get-AzMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

Ces métriques permettent d’émettre une alerte ou de procéder à un mise à l’échelle.

| Nom de métrique | Unité |
| --- | --- |
| CpuPercentage |Pourcentage |
| MemoryPercentage |Pourcentage |
| DiskQueueLength |Count |
| HttpQueueLength |Count |
| BytesReceived |Octets |
| BytesSent |Octets |

## <a name="commonly-used-storage-metrics"></a>Métriques couramment utilisées dans Azure Storage
Vous pouvez procéder à une mise à l’échelle en fonction de la métrique Longueur de file d’attente, qui correspond au nombre de messages dans la file d’attente de stockage. La longueur de file d’attente de stockage est une métrique spéciale et le seuil appliqué sera le nombre de messages par instance. Par exemple, si vous avez deux instances et que le seuil est défini sur 100, la mise à l’échelle aura lieu lorsque la file d’attente contiendra 200 messages. Cela peut être 100 messages par instance, 120 et 80 ou toute autre combinaison qui correspond à 200 ou plus.

Configurez de paramètre dans le panneau **Paramètres** du Portail Azure. Pour les jeux de mise à l’échelle de machine virtuelle, vous pouvez mettre à jour le paramètre Mise à l’échelle automatique dans le modèle Resource Manager afin d’utiliser *metricName* avec la valeur *ApproximateMessageCount*, puis transmettre l’ID de la file d’attente de stockage avec la valeur *metricResourceUri*.

Par exemple, avec un compte de stockage classique, le paramètre de mise à l’échelle automatique metricTrigger peut inclure :

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ClassicStorage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
 ```

Pour un compte de stockage (non classique), le paramètre metricTrigger peut inclure :

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.Storage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
```

## <a name="commonly-used-service-bus-metrics"></a>Métriques Service Bus généralement utilisées
Vous pouvez procéder à une mise à l’échelle en fonction de la longueur de la file d’attente Service Bus, autrement dit en fonction du nombre de messages présents dans cette file d’attente. La longueur de la file d’attente Service Bus est une métrique particulière. Le seuil correspond au nombre de messages par instance. Par exemple, si vous avez deux instances et que le seuil est défini sur 100, la mise à l’échelle aura lieu lorsque la file d’attente contiendra 200 messages. Cela peut être 100 messages par instance, 120 et 80 ou toute autre combinaison qui correspond à 200 ou plus.

Pour les jeux de mise à l’échelle de machine virtuelle, vous pouvez mettre à jour le paramètre Mise à l’échelle automatique dans le modèle Resource Manager afin d’utiliser *metricName* avec la valeur *ApproximateMessageCount*, puis transmettre l’ID de la file d’attente de stockage avec la valeur *metricResourceUri*.

```
"metricName": "ApproximateMessageCount",
 "metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ServiceBus/namespaces/SB_NAMESPACE/queues/QUEUE_NAME"
```

> [!NOTE]
> Le concept de groupe de ressources n’existe pas pour Service Bus, mais Azure Resource Manager crée un groupe de ressources par défaut par région. Le groupe de ressources est généralement affiché au format « [Région] Service Bus par défaut ». Par exemple, « Est des États-Unis Service Bus par défaut », « Ouest des États-Unis Service Bus par défaut », « Est de l’Australie Service Bus par défaut », etc.
>
>

