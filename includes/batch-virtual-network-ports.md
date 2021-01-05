---
title: Fichier include
description: Fichier include
services: batch
documentationcenter: ''
author: JnHs
manager: evansma
editor: ''
ms.service: batch
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.date: 06/16/2020
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: e4f17fbfad1e7e550b3a1e95c93e4b061d0f1c3c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993418"
---
### <a name="general-requirements"></a>Conditions générales

* Le réseau virtuel doit se trouver dans la même région et le même abonnement que le compte Batch utilisé pour créer le pool.

* Le pool utilisant le réseau virtuel peut avoir un maximum de 4096 nœuds.

* Le sous-réseau spécifié pour le pool doit avoir suffisamment d’adresses IP non attribuées pour contenir le nombre de machines virtuelles ciblées pour le pool, autrement dit, la somme des propriétés `targetDedicatedNodes` et `targetLowPriorityNodes` du pool. Si le sous-réseau ne dispose pas de suffisamment d’adresses IP non attribuées, le pool alloue partiellement les nœuds de calcul, et une erreur de redimensionnement se produit.

* Votre point de terminaison de Stockage Azure doit être résolu par tous les serveurs DNS personnalisés qui traitent votre réseau virtuel. Plus précisément, les URL sous la forme `<account>.table.core.windows.net`, `<account>.queue.core.windows.net` et `<account>.blob.core.windows.net` doivent être résolvables.

Les exigences de réseau virtuel supplémentaires diffèrent si le pool Batch est dans la configuration de machine virtuelle ou dans la configuration des services cloud. Pour les nouveaux déploiements de pool dans un réseau virtuel, la configuration de machine virtuelle est recommandée.

### <a name="pools-in-the-virtual-machine-configuration"></a>Pools dans la configuration de machine virtuelle

**Prise en charge des réseaux virtuels** : uniquement les réseaux virtuels basés sur Azure Resource Manager

**ID de sous-réseau** : lorsque vous spécifiez le sous-réseau à l’aide de l’API Batch, utilisez l’*identificateur de ressource* du sous-réseau. L’identificateur du sous-réseau est au format :

`/subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.Network/virtualNetworks/{network}/subnets/{subnet}`

**Autorisations** : vérifiez si vos stratégies ou verrous de sécurité l’abonnement ou le groupe de ressources du réseau virtuel restreignent les autorisations d’un utilisateur pour gérer le réseau virtuel.

**Ressources de mise en réseau supplémentaires** : Batch alloue automatiquement des ressources de mise en réseau supplémentaires dans le groupe de ressources contenant le réseau virtuel.

> [!IMPORTANT]
> Pour chaque 100 nœuds dédiés ou de faible priorité, Batch alloue : un groupe de sécurité réseau (NSG), une adresse IP publique et un équilibreur de charge. Ces ressources sont limitées par les [quotas de ressources](../articles/azure-resource-manager/management/azure-subscription-service-limits.md) de l’abonnement. Pour les grands pools, vous devrez peut-être demander une augmentation du quota pour une ou plusieurs de ces ressources.

#### <a name="network-security-groups-batch-default"></a>Groupes de sécurité réseau : configuration Batch par défaut

Le sous-réseau doit autoriser les communications entrantes issues du service Batch pour pouvoir planifier des tâches sur les nœuds de calcul, et également les communications sortantes pour communiquer avec le stockage Azure ou d’autres ressources en fonction des besoins de votre charge de travail. Pour les pools dans la configuration de machine virtuelle, Batch ajoute des groupes de sécurité réseau au niveau des interfaces réseau (NIC) attaché aux nœuds de calcul. Ces groupes de sécurité réseau sont configurés avec les règles supplémentaires suivantes :

* Le trafic TCP entrant sur les ports 29876 et 29877 à partir d’adresses IP de service Batch qui correspondent à l’étiquette de service `BatchNodeManagement`.
* Le trafic TCP entrant sur le port 22 (nœuds Linux) ou le port 3389 (nœuds Windows) pour autoriser l’accès à distance. Pour certains types de tâches multi-instances sur Linux (par exemple, MPI), vous devez également autoriser le trafic sur le port 22 SSH pour les adresses IP dans le sous-réseau contenant les nœuds de calcul Batch. Ce trafic peut être bloqué par des règles de groupe de sécurité réseau au niveau du sous-réseau (voir ci-dessous).
* Le trafic sortant sur n’importe quel port vers le réseau virtuel. Ce trafic peut être modifié par des règles de groupe de sécurité réseau au niveau du sous-réseau (voir ci-dessous).
* Le trafic sortant sur n’importe quel port vers internet. Ce trafic peut être modifié par des règles de groupe de sécurité réseau au niveau du sous-réseau (voir ci-dessous).

> [!IMPORTANT]
> Soyez prudent si vous modifiez ou ajoutez des règles de trafic entrant ou sortant dans des groupes de sécurité réseau configurés par Batch. Si la communication vers les nœuds de calcul dans le sous-réseau spécifié est refusée par un groupe de sécurité réseau, le service Batch définit l’état des nœuds de calcul sur **inutilisable**. En outre, aucun verrou de ressource ne doit être appliqué à une ressource créée par Batch, puisque cela peut empêcher le nettoyage des ressources à la suite d’actions lancées par l’utilisateur, telles que la suppression d’un pool.

#### <a name="network-security-groups-specifying-subnet-level-rules"></a>Groupes de sécurité réseau : spécification de règles au niveau du sous-réseau

Vous n’avez pas besoin de spécifier des groupes de sécurité réseau au niveau du sous-réseau du réseau virtuel, car Batch configure ses propres groupes de sécurité réseau (voir ci-dessus). Si vous avez un groupe de sécurité réseau associé au sous-réseau dans lequel les nœuds de calcul Batch sont déployés ou si vous souhaitez appliquer des règles de groupe de sécurité réseau personnalisées pour remplacer celles appliquées par défaut, vous devez configurer ce groupe de sécurité réseau avec au moins les règles de sécurité entrantes et sortantes montrées dans les tableaux suivants.

Configurez le trafic entrant sur le port 3389 (Windows) ou 22 (Linux) uniquement si vous avez besoin d’autoriser l’accès à distance aux nœuds de calcul depuis des sources externes. Vous devrez peut-être activer les règles du port 22 sur Linux si vous avez besoin de la prise en charge des tâches multi-instances avec certains runtimes MPI. Le fait d’autoriser le trafic sur ces ports n’est pas strictement nécessaire pour que les nœuds de calcul du pool soient utilisables.

**Règles de sécurité entrantes**

| Adresses IP sources | Balise du service source | Ports source | Destination | Ports de destination | Protocol | Action |
| --- | --- | --- | --- | --- | --- | --- |
| N/A | [Étiquette de service](../articles/virtual-network/network-security-groups-overview.md#service-tags) `BatchNodeManagement` (si vous utilisez une variante régionale, dans la même région que votre compte Batch) | * | Quelconque | 29876-29877 | TCP | Allow |
| Adresses IP sources utilisateurs pour accéder à distance à des nœuds de calcul et/ou à un sous-réseau de nœuds de calcul pour les tâches multi-instances de Linux, si nécessaire. | N/A | * | Quelconque | 3389 (Windows), 22 (Linux) | TCP | Allow |

> [!WARNING]
> Les adresses IP du service Batch peuvent changer au fil du temps. Par conséquent, il est fortement recommandé d’utiliser l’étiquette de service `BatchNodeManagement` (ou une variante régionale) pour les règles de groupe de sécurité réseau. Évitez de renseigner les règles NSG avec des adresses IP spécifiques du service Batch.

**Règles de sécurité de trafic entrant**

| Source | Ports source | Destination | Identification de destination | Ports de destination | Protocol | Action |
| --- | --- | --- | --- | --- | --- | --- |
| Quelconque | * | [Balise du service](../articles/virtual-network/network-security-groups-overview.md#service-tags) | `Storage` (si vous utilisez une variante régionale, dans la même région que votre compte Batch) | 443 | TCP | Allow |

### <a name="pools-in-the-cloud-services-configuration"></a>Pools dans la configuration des services cloud

**Réseaux virtuels pris en charge** : réseaux virtuels classiques uniquement

**ID de sous-réseau** : lorsque vous spécifiez le sous-réseau à l’aide de l’API Batch, utilisez l’*identificateur de ressource* du sous-réseau. L’identificateur du sous-réseau est au format :

`/subscriptions/{subscription}/resourceGroups/{group}/providers/Microsoft.ClassicNetwork /virtualNetworks/{network}/subnets/{subnet}`

**Autorisations** : le `Microsoft Azure Batch`principal du service doit avoir le rôle Azure `Classic Virtual Machine Contributor` pour le réseau virtuel spécifié.

#### <a name="network-security-groups"></a>Groupes de sécurité réseau

Le sous-réseau doit autoriser les communications entrantes issues du service Batch pour pouvoir planifier des tâches sur les nœuds de calcul, et également les communications sortantes pour communiquer avec le stockage Azure ou d’autres ressources.

Il est inutile de spécifier un groupe de sécurité réseau, étant donné que Batch configure les communications entrantes uniquement à partir d’adresses IP Batch pour les nœuds du pool. Cependant, si le sous-réseau spécifié comporte des groupes de sécurité réseau associés et/ou un pare-feu, configurez les règles de sécurité du trafic entrant et sortant comme indiqué dans les tableaux suivants. Si la communication vers les nœuds de calcul dans le sous-réseau spécifié est refusée par un groupe de sécurité réseau, le service Batch définit l’état des nœuds de calcul sur **inutilisable**.

Configurez le trafic entrant sur le port 3389 (Windows) si vous avez besoin d’autoriser l’accès RDP aux nœuds du pool. Ce n’est pas nécessaire pour que les nœuds du pool soient utilisables.

**Règles de sécurité entrantes**

| Adresses IP sources | Ports source | Destination | Ports de destination | Protocol | Action |
| --- | --- | --- | --- | --- | --- |
Quelconque <br /><br />Bien que cela nécessite réellement de « tout autoriser », le service Batch applique une règle ACL au niveau de chaque nœud, excluant toutes les adresses IP des services autres que Batch. | * | Quelconque | 10100, 20100, 30100 | TCP | Allow |
| Facultatif, pour autoriser l’accès RDP aux nœuds de calcul. | * | Quelconque | 3389 | TCP | Allow |

**Règles de sécurité de trafic entrant**

| Source | Ports source | Destination | Ports de destination | Protocol | Action |
| --- | --- | --- | --- | --- | --- |
| Quelconque | * | Quelconque | 443  | Quelconque | Allow |