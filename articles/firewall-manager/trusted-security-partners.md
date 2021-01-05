---
title: Que sont les fournisseurs de partenaire de sécurité Azure Firewall Manager ?
description: Découvrir les fournisseurs de partenaire de sécurité Azure Firewall Manager
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: victorh
ms.openlocfilehash: 923c6b685d20ff68788e7d9cfcb45ebaecb535e3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96490106"
---
# <a name="what-are-security-partner-providers"></a>Que sont les fournisseurs de partenaire de sécurité ?

Dans Azure Firewall Manager, les *fournisseurs de partenaire de sécurité* vous permettent d’utiliser les meilleures offres SECaaS (sécurité en tant que service) tierces afin de protéger l’accès à Internet de vos utilisateurs.

Au moyen d’une configuration rapide, vous pouvez sécuriser un hub avec un partenaire de sécurité pris en charge, et router et filtrer le trafic Internet à partir de vos réseaux virtuels ou positions de filiale au sein d’une région. Pour cela, vous pouvez utiliser la gestion automatisée des routes, sans configuration ni gestion des routes définies par l’utilisateur (UDR).

Vous pouvez déployer des hubs sécurisés configurés avec le partenaire de sécurité de votre choix dans plusieurs régions Azure pour que vos utilisateurs bénéficient de la connectivité et de la sécurité n’importe où dans le monde entier dans ces régions. Avec la possibilité d’utiliser l’offre du partenaire de sécurité pour le trafic des applications SaaS/Internet et Pare-feu Azure pour le trafic privé dans les hubs sécurisés, vous pouvez maintenant commencer à créer sur Azure un périmètre de sécurité proche de vos utilisateurs et applications répartis dans le monde entier.

Les partenaires de sécurité pris en charge sont **Zscaler**, **[Check Point](check-point-overview.md)** et **iboss**.

![Fournisseurs de partenaire de sécurité](media/trusted-security-partners/trusted-security-partners.png)

## <a name="key-scenarios"></a>Principaux scénarios

Vous pouvez utiliser les partenaires de sécurité pour filtrer le trafic Internet dans les scénarios suivants :

- Réseau virtuel vers Internet

   Tirez parti de la protection Internet avancée prenant en charge l’utilisateur pour vos charges de travail cloud s’exécutant sur Azure.

- Filiale vers Internet

   Tirez parti de votre connectivité Azure et de votre distribution mondiale pour ajouter facilement un filtrage NSaaS tiers dans le cadre des scénarios de type « filiale vers Internet ». Vous pouvez créer votre réseau de transit mondial et votre périmètre de sécurité à l’aide d’Azure Virtual WAN.

Les scénarios suivants sont pris en charge :
- Réseau virtuel/branche à Internet par le biais d’un fournisseur de partenaire de sécurité et l’autre trafic (Spoke à Spoke, Spoke à branche, branche à Spoke) par le biais du Pare-feu Azure.
- Réseau virtuel/branche à Internet par le biais d’un fournisseur de partenaire de sécurité

## <a name="best-practices-for-internet-traffic-filtering-in-secured-virtual-hubs"></a>Bonnes pratiques pour le filtrage du trafic Internet dans les hubs virtuels sécurisés

Le trafic Internet comprend généralement le trafic web. Toutefois, il comprend également le trafic destiné aux applications SaaS comme Microsoft 365 et aux services PaaS publics Azure tels que Stockage Azure et Azure SQL. Voici les bonnes pratiques recommandées pour gérer le trafic vers ces services :

### <a name="handling-azure-paas-traffic"></a>Gestion du trafic PaaS Azure
 
- Utilisez Pare-feu Azure pour la protection si votre trafic se compose principalement de données Azure PaaS et que l’accès aux ressources pour vos applications peut être filtré à l’aide d’adresses IP, de noms de domaine complets, d’étiquettes de service ou d’étiquettes de nom de domaine complet.

- Utilisez une solution de partenaire tierce dans vos hubs si votre trafic est constitué d’un accès aux applications SaaS, si vous avez besoin d’un filtrage prenant en charge l’utilisateur (par exemple, pour vos charges de travail d’infrastructure de bureau virtuel) ou si vous avez besoin de fonctionnalités de filtrage Internet avancées.

![Tous les scénarios pour Azure Firewall Manager](media/trusted-security-partners/all-scenarios.png)

## <a name="handling-microsoft-365-traffic"></a>Gestion du trafic Microsoft 365

Dans les scénarios de positions de filiales à l’échelle mondiale, vous devez rediriger le trafic Microsoft 365 directement au niveau de la filiale avant d’envoyer le reste du trafic Internet à votre hub sécurisé Azure.

Pour Microsoft 365, la latence et les performances du réseau sont essentielles pour une expérience utilisateur réussie. Pour que les performances et l’expérience utilisateur soient optimales, les clients doivent implémenter un dispositif de déchargement direct et local pour Microsoft 365 avant d’envisager de router le reste du trafic Internet via Azure.

Selon les [principes de connectivité réseau Microsoft 365](/microsoft-365/enterprise/microsoft-365-network-connectivity-principles), les principales connexions réseau Microsoft 365 doivent être routées localement à partir de l’appareil mobile ou de la filiale où se trouvent les utilisateurs et directement via Internet vers le point de présence réseau Microsoft le plus proche.

De plus, les connexions Microsoft 365 sont chiffrées pour protéger la confidentialité et utilisent des protocoles propriétaires efficaces pour des raisons de performances. C’est pourquoi il est peu pratique et trop risqué de soumettre ces connexions à des solutions de sécurité de niveau réseau traditionnelles. Pour ces raisons, nous recommandons vivement aux clients d’envoyer le trafic Microsoft 365 directement à partir des filiales, avant d’envoyer le reste du trafic via Azure. Microsoft s’est associé à plusieurs fournisseurs de solutions SD-WAN, qui s’intègrent avec Azure et Microsoft 365, et permettent aux clients d’activer le déchargement direct et local du trafic Internet pour Microsoft 365. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Virtual WAN ?](../virtual-wan/virtual-wan-about.md)

## <a name="next-steps"></a>Étapes suivantes

[Déployez une offre de partenaire de sécurité dans un hub sécurisé à l’aide d’Azure Firewall Manager](deploy-trusted-security-partner.md).
