---
title: Présentation de la vue Règles de sécurité en vigueur dans Azure Network Watcher | Microsoft Docs
description: Cette page fournit une vue d’ensemble de la capacité d’affichage Règles de sécurité en vigueur de Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/26/2017
ms.author: damendo
ms.openlocfilehash: d172ba7353b7ac4c490584cd7c84871c6244f214
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961819"
---
# <a name="introduction-to-effective-security-rules-view-in-azure-network-watcher"></a>Présentation de la vue Obtenir les règles de sécurité efficaces dans Azure Network Watcher

Les groupes de sécurité réseau sont associés à un niveau de sous-réseau ou à un niveau de carte réseau. Lorsqu’il est associé à un niveau de sous-réseau, il s’applique à toutes les instances de machine virtuelle du sous-réseau. La vue Obtenir les règles de sécurité efficaces retourne l’ensemble des règles et des groupes de sécurité réseau configurés qui sont associés à un niveau de sous-réseau et de carte réseau pour une machine virtuelle fournissant des informations sur la configuration. En outre, les règles de sécurité effectives sont renvoyées pour chacune des cartes réseau d’une machine virtuelle. La vue Obtenir les règles de sécurité efficaces vous permet d’évaluer les vulnérabilités réseau d’une machine virtuelle comme les ports ouverts. Vous pouvez également vérifier si votre Groupe de sécurité réseau fonctionne comme prévu en [comparant les règles de sécurité approuvées et configurées](network-watcher-nsg-auditing-powershell.md).

Un cas d’utilisation plus poussée concerne l’audit et la conformité de la sécurité. Vous pouvez définir un ensemble normatif de règles de sécurité comme modèle pour la gouvernance de la sécurité de votre organisation. Un audit de conformité périodique peut être implémenté de façon programmatique en comparant les règles normatives avec les règles effectives pour toutes les machines virtuelles de votre réseau.

Sur le portail, à chaque interface réseau correspondent des règles qui sont regroupées selon qu’elles s’appliquent au trafic entrant ou au trafic sortant. Cela permet de savoir facilement quelles sont les règles appliquées à une machine virtuelle. Un bouton de téléchargement est fourni pour télécharger simplement toutes les règles de sécurité, quel que soit l’onglet, dans un fichier CSV.

![affichage des groupes de sécurité][1]

Les règles peuvent être sélectionnées et un nouveau panneau s’ouvre pour afficher le groupe de sécurité réseau et les préfixes source et de destination. Dans ce panneau, vous pouvez naviguer directement vers la ressource de groupe de sécurité réseau.

![exploration][2]

### <a name="next-steps"></a>Étapes suivantes

Vous pouvez également utiliser la fonctionnalité *Groupes de sécurité en vigueur* par le biais d’autres méthodes listées ci-dessous :
* [REST API](/rest/api/virtualnetwork/NetworkInterfaces/ListEffectiveNetworkSecurityGroups)
* [PowerShell](/powershell/module/az.network/get-azeffectivenetworksecuritygroup?view=azps-4.4.0)
* [Azure CLI](/cli/azure/network/nic?view=azure-cli-latest#az-network-nic-list-effective-nsg)

Découvrez comment effectuer un audit de vos paramètres de groupe de sécurité réseau en consultant [Automate NSG auditing with Azure Network Watcher Security group view](network-watcher-nsg-auditing-powershell.md) (Automatiser l’audit des groupes de sécurité réseau avec l’affichage des groupes de sécurité réseau Azure Network Watcher)

[1]: ./media/network-watcher-security-group-view-overview/securitygroupview.png
[2]: ./media/network-watcher-security-group-view-overview/figure1.png