---
title: Présentation des diagnostics de configuration réseau dans Azure Network Watcher | Microsoft Docs
description: Cette page fournit une vue d’ensemble du service Network Watcher – Diagnostics de configuration réseau
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/15/2020
ms.author: damendo
ms.openlocfilehash: ad60b0be8a9a5341c71d760f98c0db84a72763c2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94951517"
---
# <a name="introduction-to-network-configuration-diagnostics-in-azure-network-watcher"></a>Présentation des diagnostics de configuration réseau dans Azure Network Watcher

L’outil de diagnostic de configuration réseau aide les clients à comprendre quels flux de trafic seront autorisés ou refusés dans votre réseau virtuel Azure, et fournit des informations détaillées pour le débogage. Cela peut vous aider à comprendre si vos règles de groupe de sécurité réseau (NSG) sont correctement configurées. 

## <a name="pre-requisites"></a>Conditions préalables
Pour que vous puissiez utiliser les diagnostics de configuration réseau, le service Network Watcher doit être activé dans votre abonnement. Pour l’activer Voir [Créer une instance d’Azure Network Watcher](./network-watcher-create.md).

## <a name="background"></a>Arrière-plan

- Vos ressources dans Azure sont connectées via des réseaux virtuels et des sous-réseaux. Vous pouvez gérer la sécurité de ces réseaux virtuels et sous-réseaux à l’aide d’un groupe de sécurité réseau (NSG).
- Un NSG contient une liste des règles de sécurité qui autorisent ou rejettent le trafic réseau à destination des ressources auxquelles il est connecté. Vous pouvez associer des NSG à des sous-réseaux, à des machines virtuelles ou à des cartes réseau (NIC) attachées à des machines virtuelles. 
- Tous les flux de trafic au sein de votre réseau sont évalués à l’aide des règles dans le groupe de sécurité réseau applicable.
- Les règles sont évaluées en fonction de leur priorité, de la plus basse à la plus élevée. 

## <a name="how-does-network-configuration-diagnostic-work"></a>Fonctionnement du diagnostic de configuration réseau 

Pour un flux donné, l’outil NCD exécute une simulation du flux et retourne une valeur indiquant si le flux est autorisé (ou refusé), ainsi que des informations détaillées sur les règles autorisant/refusant le flux.  Les clients doivent fournir les détails d’un flux, tels que sa source, sa destination, son protocole, etc. L’outil indique si le trafic a été autorisé ou refusé, les règles de NSG qui ont été évaluées pour le flux spécifié, et les résultats de l’évaluation de chaque règle.

## <a name="next-steps"></a>Étapes suivantes

Utiliser un diagnostic de la configuration réseau via d’autres interfaces
 - [REST API](/rest/api/network-watcher/networkwatchers/getnetworkconfigurationdiagnostic)
 - [PowerShell](/powershell/module/az.network/invoke-aznetworkwatchernetworkconfigurationdiagnostic?view=azps-4.6.1)
 - [Azure CLI](/cli/azure/network/watcher?view=azure-cli-latest#az_network_watcher_run_configuration_diagnostic)