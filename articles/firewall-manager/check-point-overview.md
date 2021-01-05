---
title: Sécuriser des hubs virtuels Azure avec Check Point CloudGuard Connect
description: En savoir plus sur l’utilisation de Check Point CloudGuard Connect pour sécuriser des hubs virtuels Azure
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: conceptual
ms.date: 10/30/2020
ms.author: victorh
ms.openlocfilehash: 3c61dc689d19e1a7d6f9b6dbefae846e9458d750
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146856"
---
# <a name="secure-virtual-hubs-using-check-point-cloudguard-connect"></a>Sécuriser des hubs virtuels avec Check Point CloudGuard Connect

La solution Check Point CloudGuard Connect est un partenaire de sécurité de confiance dans Azure Firewall Manager. Elle protège les connexions globalement distribuées de filiale à Internet (B2I) ou de réseau virtuel à Internet (V2I) avec une protection avancée contre les menaces. 

Une simple configuration dans Azure Firewall Manager vous permet d’acheminer des connexions de filiale et de réseau virtuel vers Internet en bénéficiant de la sécurité en tant que service de CloudGuard Connect Security (SECaaS). Le trafic est protégé en transit entre votre hub et le service cloud Check Point dans des tunnels VPN IPsec.

Lorsque vous activez la synchronisation automatique dans le portail Check Point, toutes les ressources marquées comme *sécurisées* dans le portail Azure sont automatiquement sécurisées. Vous n’avez pas besoin de gérer vos ressources deux fois. Il vous suffit de choisir de les sécuriser une fois dans le portail Azure.

Check Point unifie plusieurs services de sécurité sous un même parapluie. Le trafic de sécurité intégré est déchiffré et inspecté en une seule passe. Le contrôle d’application, le filtrage d’URL et la sensibilisation au contenu (DLP) garantissent une utilisation du web sécurisée et protègent vos données. Les applications IPS et Antivirus protègent les utilisateurs contre les attaques réseau connues. L’application Anti-Bot bloque les connexions aux serveurs de commande et de contrôle, et vous avertit en cas d’infection d’un ordinateur hôte.

L’application Threat Emulation (sandboxing) protège les utilisateurs contre les menaces inconnues et Zero-Day. Check Point SandBlast Zero-Day Protection est une technologie de mise en bac à sable hébergée dans le cloud qui permet une mise en quarantaine et une inspection rapides des fichiers. Elle opère dans un bac à sable virtuel pour détecter les comportements malveillants avant qu’ils pénètrent dans votre réseau. Elle bloque les menaces avant qu’elles occasionnent des dommages afin de réduire le temps de réponse aux menaces. 

## <a name="deployment-example"></a>Exemple de déploiement

Regardez la vidéo suivante pour voir comment déployer Check Point CloudGuard Connect en tant que partenaire de sécurité Azure de confiance.

> [!VIDEO https://www.youtube.com/embed/C8AuN76DEmU]

## <a name="next-steps"></a>Étapes suivantes

- [Déployer un fournisseur de partenaire de sécurité](deploy-trusted-security-partner.md)