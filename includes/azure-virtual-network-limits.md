---
title: Fichier include
description: Fichier include
services: networking
author: anavinahar
ms.service: networking
ms.topic: include
ms.date: 01/14/2020
ms.author: anavin
ms.custom: include file
ms.openlocfilehash: 9751bd082947e66b3fc5ea47448857558301a1e7
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97366382"
---
### <a name="networking-limits---azure-resource-manager"></a><a name="azure-resource-manager-virtual-networking-limits"></a>Limites de mise en réseau - Azure Resource Manager
Les limites suivantes s’appliquent uniquement aux ressources de réseau gérées par le biais d’**Azure Resource Manager** par région et par abonnement. Découvrez comment [afficher l’utilisation actuelle de vos ressources par rapport aux limites de votre abonnement](../articles/networking/check-usage-against-limits.md).

> [!NOTE]
> Récemment, nous avons augmenté toutes les limites par défaut à leur niveau maximal. Si aucune colonne de limite maximale n’est présente, la ressource spécifiée ne possède pas de limites ajustables. Si ces limites ont été augmentées par le support dans le passé et si vous ne voyez pas les limites mises à jour dans les tables suivantes, vous pouvez [ouvrir une demande de support clientèle en ligne sans frais](../articles/azure-resource-manager/templates/error-resource-quota.md)

| Ressource | Limite | 
| --- | --- |
| Réseaux virtuels |1 000 |
| Nombre de sous-réseaux par réseau virtuel |3 000 |
| Peerings VNet par réseau virtuel |500 |
| [Passerelles de réseau virtuel (passerelles VPN) par réseau virtuel](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md#gwsku) |1 |
| [Passerelles de réseau virtuel (passerelles ExpressRoute) par réseau virtuel](../articles/expressroute/expressroute-about-virtual-network-gateways.md#gwsku) |1 |
| Serveurs DNS par réseau virtuel |20 |
| Adresses IP privées par réseau virtuel |65 536 |
| Adresses IP privées par interface réseau |256 |
| Adresses IP privées par machine virtuelle |256 |
| Adresses IP publiques par interface réseau |256 |
| Adresses IP publiques par machine virtuelle |256 |
| [Flux TCP ou UDP simultanés par carte réseau de machine virtuelle ou instance de rôle](../articles/virtual-network/virtual-machine-network-throughput.md#flow-limits-and-recommendations) |500 000 |
| Cartes d’interface réseau |65 536 |
| Network Security Group |5 000 |
| Règles de groupe de sécurité réseau par groupe de sécurité réseau |1 000 |
| Adresses IP et plages spécifiées pour la source et la destination dans un groupe de sécurité |4 000 |
| Groupes de sécurité d’application |3 000 |
| Groupes de sécurité d’application par configuration IP, par carte réseau |20 |
| Configurations IP par groupe de sécurité d’application |4 000 |
| Groupes de sécurité d’application qui peuvent être spécifiés dans toutes les règles de sécurité d’un groupe de sécurité réseau |100 |
| Tables d’itinéraires définies par l’utilisateur |200 |
| Itinéraires définis par l’utilisateur par table d’itinéraire |400 |
| Certificats racines point à site pour chaque passerelle VPN Azure |20 |
| TAP de réseau virtuel |100 |
| Configurations TAP d’interface réseau de chaque réseau virtuel TAP |100 |

#### <a name="public-ip-address-limits"></a><a name="publicip-address"></a>Limites de l’adresse IP publique
| Ressource | Limite par défaut | Limite maximale |
| --- | --- | --- |
| Adresse IP publique<sup>1</sup> | 10 pour De base. | Contactez le support technique. |
| Adresses IP publiques statiques<sup>1</sup> | 10 pour De base. | Contactez le support technique. |
| Adresses IP publiques standard<sup>1</sup> | 10 | Contactez le support technique. |
| [Adresses IP publiques par groupe de ressources](../articles/azure-resource-manager/management/resources-without-resource-group-limit.md#microsoftnetwork) | 800 | Contactez le support technique. | 
| Préfixes d’adresses IP publiques | Limités par le nombre d’adresses IP publiques standard dans un abonnement | Contactez le support technique. |
| Taille de préfixe d’adresse IP publique | /28 | Contactez le support technique. |

<sup>1</sup>Les limites par défaut pour les adresses IP publiques varient selon le type de catégorie d’offre, comme Essai gratuit, Paiement à l’utilisation, CSP. Par exemple, la valeur par défaut pour les abonnements Accord Entreprise est 1 000.

#### <a name="load-balancer-limits"></a><a name="load-balancer"></a>Limites de l’équilibreur de charge
Les limites suivantes s’appliquent uniquement aux ressources de réseau gérées par le biais d’Azure Resource Manager par région et par abonnement. Découvrez comment [afficher l’utilisation actuelle de vos ressources par rapport aux limites de votre abonnement](../articles/networking/check-usage-against-limits.md).

**Standard Load Balancer**

| Ressource                                | Limite         |
|-----------------------------------------|-------------------------------|
| Équilibreurs de charge                          | 1 000                         |
| Règles par ressource                      | 1 500                         |
| Règles par groupe de sécurité réseau (sur l’ensemble des adresses IP sur une carte réseau) | 300                           |
| Configurations d’adresses IP frontales              | 600                           |
| Taille de pool de serveur principal                       | 1 000 configurations d’adresses IP, réseau virtuel unique |
| Ressources back-end par équilibreur de charge <sup>1<sup> | 250                   |
| Ports de haute disponibilité                 | 1 par serveur frontal interne       |
| Règles de trafic sortant par Load Balancer        | 600                           |
| Équilibreurs de charge par machine virtuelle                   | 2 (1 public et 1 interne)   |

<sup>1</sup>La limite est fixée à 150 ressources, toute combinaison de ressources de machines virtuelles autonomes, ressources de groupes à haute disponibilité et groupes de placement de groupes de machines virtuelles identiques.

**Load Balancer de base**

| Ressource                                | Limite        |
|-----------------------------------------|------------------------------|
| Équilibreurs de charge                          | 1 000                        |
| Règles par ressource                      | 250                          |
| Règles par groupe de sécurité réseau (sur l’ensemble des adresses IP sur une carte réseau) | 300                          |
| Configurations d’adresses IP frontales              | 200                          |
| Taille de pool de serveur principal                       | 300 configurations d’adresses IP, groupe à haute disponibilité |
| Groupes à haute disponibilité par Load Balancer     | 1                            |
| Équilibreurs de charge par machine virtuelle                   | 2 (1 public et 1 interne)  |

<a name="virtual-networking-limits-classic"></a>Les limites suivantes s’appliquent uniquement aux ressources de réseau gérées par le biais du modèle de déploiement **classique** par abonnement. Découvrez comment [afficher l’utilisation actuelle de vos ressources par rapport aux limites de votre abonnement](../articles/networking/check-usage-against-limits.md).

| Ressource | Limite par défaut | Limite maximale |
| --- | --- | --- |
| Réseaux virtuels |100 |100 |
| Sites de réseau local |20 |50 |
| Serveurs DNS par réseau virtuel |20 |20 |
| Adresses IP privées par réseau virtuel |4 096 |4 096 |
| Flux TCP ou UDP simultanés par carte réseau de machine virtuelle ou instance de rôle |500 000, jusqu’à 1 000 000 pour deux cartes réseau ou plus. |500 000, jusqu’à 1 000 000 pour deux cartes réseau ou plus. |
| Groupes de sécurité réseau (NSG) |200 |200 |
| Règles de groupe de sécurité réseau par groupe de sécurité réseau |200 |1 000 |
| Tables d’itinéraires définies par l’utilisateur |200 |200 |
| Itinéraires définis par l’utilisateur par table d’itinéraire |400 |400 |
| Adresses IP publiques (dynamiques) |500 |500 |
| Adresses IP publiques réservées |500 |500 |
| IP publique par déploiement |5 |Contacter le support technique |
| IP privée (équilibrage de charge interne) par déploiement |1 |1 |
| Listes de contrôle d’accès (ACL) par point de terminaison |50 |50 |
