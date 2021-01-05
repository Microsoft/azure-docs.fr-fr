---
title: Autorisations Azure RBAC requises pour utiliser les fonctionnalités
titleSuffix: Azure Network Watcher
description: Découvrez les autorisations de contrôle d’accès en fonction du rôle Azure qui sont nécessaires pour utiliser les fonctionnalités de Network Watcher.
services: network-watcher
documentationcenter: ''
author: damendo
ms.service: network-watcher
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: damendo
ms.openlocfilehash: 548db64bd93dd561f9c69e9f594d01f4c0825db9
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94948338"
---
# <a name="azure-role-based-access-control-permissions-required-to-use-network-watcher-capabilities"></a>Autorisations de contrôle d'accès en fonction du rôle Azure obligatoires pour utiliser les fonctionnalités de Network Watcher

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet d’attribuer aux membres de votre organisation uniquement les actions dont ils ont besoin pour remplir leur fonction. Pour utiliser les fonctionnalités de Network Watcher, le compte avec lequel vous vous connectez à Azure doit être affecté aux rôles intégrés [Propriétaire](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#owner), [Contributeur](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#contributor) ou [Contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#network-contributor), ou à un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) auquel sont affectées les actions listées pour chaque fonctionnalité de Network Watcher dans les sections qui suivent. Pour en savoir plus sur les fonctionnalités de Network Watcher, consultez [Qu’est-ce que Network Watcher](network-watcher-monitoring-overview.md).

## <a name="network-watcher"></a>Network Watcher

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/read                              | Obtenir un observateur réseau                                          |
| Microsoft.Network/networkWatchers/write                             | Créer ou mettre à jour un observateur réseau                             |
| Microsoft.Network/networkWatchers/delete                            | Supprimer un observateur réseau                                       |

## <a name="nsg-flow-logs"></a>Journaux de flux NSG

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/configureFlowLog/action           | Configurer un journal de flux                                           |
| Microsoft.Network/networkWatchers/queryFlowLogStatus/action         | Interroger l’état d’un journal de flux                                    |

## <a name="connection-troubleshoot"></a>Résoudre les problèmes de connexion

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/connectivityCheck/action          | Lancer un test de résolution de problèmes de connexion
| Microsoft.Network/networkWatchers/queryTroubleshootResult/action    | Interroger les résultats d’un test de résolution de problèmes de connexion                |
| Microsoft.Network/networkWatchers/troubleshoot/action               | Exécuter un test de résolution de problèmes de connexion                             |

## <a name="connection-monitor"></a>Moniteur de connexion

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/connectionMonitors/start/action   | Démarrer un moniteur de connexion                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/stop/action    | Arrêter un moniteur de connexion                                      |
| Microsoft.Network/networkWatchers/connectionMonitors/query/action   | Interroger un moniteur de connexion                                     |
| Microsoft.Network/networkWatchers/connectionMonitors/read           | Obtenir un moniteur de connexion                                       |
| Microsoft.Network/networkWatchers/connectionMonitors/write          | Créer un moniteur de connexion                                    |
| Microsoft.Network/networkWatchers/connectionMonitors/delete         | Supprimer un moniteur de connexion                                    |

## <a name="packet-capture"></a>Capture de paquet

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/packetCaptures/queryStatus/action | Interroger l’état d’une capture de paquets                           |
| Microsoft.Network/networkWatchers/packetCaptures/stop/action        | Arrêter une capture de paquets                                          |
| Microsoft.Network/networkWatchers/packetCaptures/read               | Obtenir une capture de paquets                                           |
| Microsoft.Network/networkWatchers/packetCaptures/write              | Créer une capture de paquets                                        |
| Microsoft.Network/networkWatchers/packetCaptures/delete             | Supprimer une capture de paquets                                        |

## <a name="ip-flow-verify"></a>Vérification du flux IP

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/ipFlowVerify/action               | Vérifier un flux IP                                              |

## <a name="next-hop"></a>Tronçon suivant

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/nextHop/action                    | Obtenir le tronçon suivant à partir d’une machine virtuelle                                     |

## <a name="network-security-group-view"></a>Vue du groupe de sécurité réseau

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/securityGroupView/action          | Afficher les groupes de sécurité                                           |

## <a name="topology"></a>Topologie

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/topology/action                   | Obtenir la topologie                                                   |
| Microsoft.Network/networkWatchers/topology/read                     | Identique à ce qui précède                                                  |

## <a name="reachability-report"></a>Rapport d’accessibilité

| Action                                                              | Description                                                           |
| ---------                                                           | -------------                                                  |
| Microsoft.Network/networkWatchers/azureReachabilityReport/action    | Obtenir un rapport d’accessibilité Azure                               |


## <a name="additional-actions"></a>Actions supplémentaires

Les fonctionnalités de Network Watcher nécessitent également les actions suivantes :

| Action(s)                                                           | Description                                                    |
| ---------                                                           | -------------                                                  |
| Microsoft.Authorization/\*/Read                                     | Utilisé pour extraire les attributions de rôle Azure et les définitions de stratégie          |
| Microsoft.Resources/subscriptions/resourceGroups/Read               | Utilisé pour énumérer tous les groupes de ressources dans un abonnement    |
| Microsoft.Storage/storageAccounts/Read                              | Utilisé pour obtenir les propriétés du compte de stockage spécifié   |
| Microsoft.Storage/storageAccounts/listServiceSas/Action, </br> Microsoft.Storage/storageAccounts/listAccountSas/Action, <br> Microsoft.Storage/storageAccounts/listKeys/Action| Utilisé pour récupérer les signatures d’accès partagé permettant l’[accès sécurisé au compte de stockage](../storage/common/storage-sas-overview.md) et l’écriture dans le compte de stockage. |
| Microsoft.Compute/virtualMachines/Read, </br> Microsoft.Compute/virtualMachines/Write| Utilisé pour se connecter à la machine virtuelle, effectuer une capture de paquets et la charger dans le compte de stockage|
| Microsoft.Compute/virtualMachines/extensions/Read </br> Microsoft.Compute/virtualMachines/extensions/Write| Utilisé pour vérifier si l’extension Network Watcher est présente et pour l’installer si nécessaire |
| Microsoft.Compute/virtualMachineScaleSets/Read, </br> Microsoft.Compute/virtualMachineScaleSets/Write| Utilisé pour accéder aux groupes de machines virtuelles identiques, effectuer des captures de paquets et les charger dans le compte de stockage|
| Microsoft.Compute/virtualMachineScaleSets/extensions/Read, </br> Microsoft.Compute/virtualMachineScaleSets/extensions/Write| Utilisé pour vérifier si l’extension Network Watcher est présente et pour l’installer si nécessaire |
| Microsoft.Insights/alertRules/*                                     | Utilisé pour configurer des alertes de métriques                                     |
| Microsoft.Support/*                                                 | Utilisé pour créer et mettre à jour des tickets de support à partir de Network Watcher |