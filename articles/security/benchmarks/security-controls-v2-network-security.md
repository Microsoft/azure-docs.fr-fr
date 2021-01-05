---
title: Azure Security Benchmark V2 - Sécurité réseau
description: Sécurité réseau dans Azure Security Benchmark V2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 38ab16fbadee245810a4486170dd54922c0c38f9
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558910"
---
# <a name="security-control-v2-network-security"></a>Contrôle de sécurité V2 : Sécurité réseau

La sécurité réseau couvre les contrôles permettant de sécuriser et de protéger les réseaux Azure. Cela comprend la sécurisation des réseaux virtuels, l’établissement de connexions privées, la prévention et l’atténuation des attaques externes, ainsi que la sécurisation de DNS.

## <a name="ns-1-implement-security-for-internal-traffic"></a>NS-1 : Implémenter la sécurité pour le trafic interne

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-1 | 9.2, 9.4, 14.1, 14.2, 14.3 | AC-4, CA-3, SC-7 |

Vérifiez que tous les réseaux virtuels Azure suivent un principe de segmentation d’entreprise qui s’aligne sur les risques métier. Tout système qui peut entraîner des risques plus élevés pour l’organisation doit être isolé sur son propre réseau virtuel et être suffisamment sécurisé à l’aide d’un groupe de sécurité réseau (NSG) et/ou du Pare-feu Azure. 

En fonction de vos applications et de votre stratégie de segmentation d’entreprise, limitez ou autorisez le trafic entre des ressources internes suivant des règles de groupe de sécurité réseau. Pour des applications bien définies (par exemple, une application à 3 niveaux), il peut s’agir d’une approche « Refuser par défaut, Autoriser par exception », hautement sécurisée. Cela peut ne pas bien évoluer si vous avez un grand nombre d’applications et de points de terminaison qui interagissent les uns avec les autres. Vous pouvez également utiliser le Pare-feu Azure dans les cas où la gestion centrale est exigée sur un grand nombre de segments d’entreprise ou de spokes (dans une topologie hub/spoke). 

Utilisez la fonctionnalité de renforcement du réseau adaptatif d’Azure Security Center pour recommander des configurations de groupe de sécurité réseau qui limitent les ports et les adresses IP sources en fonction de la référence à des règles de trafic réseau externe.

Utilisez Azure Sentinel pour découvrir l’utilisation de protocoles non sécurisés hérités, tels que SSL/TLSv1, SMBv1, LM/NTLMv1, wDigest, liaisons LDAP non signées et chiffrements faibles dans Kerberos.

- [Comment créer un groupe de sécurité réseau avec des règles de sécurité](../../virtual-network/tutorial-filter-network-traffic.md)

- [Guide pratique pour déployer et configurer le Pare-feu Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [Renforcement du réseau adaptatif dans Azure Security Center](../../security-center/security-center-adaptive-network-hardening.md)

- [Classeur des protocoles non sécurisés Azure Sentinel](../../sentinel/quickstart-get-visibility.md#use-built-in-workbooks)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)     

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-2-connect-private-networks-together"></a>NS-2 : Interconnecter des réseaux privés

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-2 | N/A | CA-3, AC-17, MA-4 |

Utilisez Azure ExpressRoute ou un réseau privé virtuel (VPN) Azure pour créer des connexions privées entre les centres de données Azure et une infrastructure locale dans un environnement de colocation. Les connexions ExpressRoute ne transitent pas par l’Internet public et offrent davantage de fiabilité, des vitesses supérieures et des latences inférieures par rapport aux connexions Internet classiques. Pour un VPN point à site et un VPN site à site, vous pouvez connecter des appareils ou des réseaux locaux à un réseau virtuel à l’aide de n’importe quelle combinaison de ces options VPN et d’Azure ExpressRoute.

Pour interconnecter deux réseaux virtuels ou plus dans Azure, utilisez un appairage de réseaux virtuels ou une liaison privée. Le trafic réseau entre les réseaux virtuels appairés est privé et conservé sur le réseau principal Azure. 

- [Quels sont les modèles de connectivité ExpressRoute ?](../../expressroute/expressroute-connectivity-models.md) 

- [Vue d’ensemble des VPN Azure](../../vpn-gateway/vpn-gateway-about-vpngateways.md)

- [Peering de réseau virtuel](../../virtual-network/virtual-network-peering-overview.md)

- [Azure Private Link](../../private-link/private-link-service-overview.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)    

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

## <a name="ns-3-establish-private-network-access-to-azure-services"></a>NS-3 : Établir un accès réseau privé aux services Azure

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-3 | 14,1 | AC-4, CA-3, SC-7 |

Utilisez Azure Private Link pour activer l’accès privé aux services Azure à partir de vos réseaux virtuels, sans passer par Internet. Dans les cas où Azure Private Link n’est pas encore disponible, utilisez des points de terminaison de service Réseau virtuel Azure.  Les points de terminaison de service Réseau virtuel Azure offrent un accès sécurisé aux services par le biais d’une route optimisée sur le réseau principal Azure.  

L’accès privé est une mesure de défense renforcée supplémentaire en plus de la sécurité de l’authentification et du trafic proposée par les services Azure. 

- [Présentation d’Azure Private Link](../../private-link/private-link-overview.md)

- [Présentation des points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

## <a name="ns-4-protect-applications-and-services-from-external-network-attacks"></a>NS-4 : Protégez les applications et les services contre les attaques réseau externes

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-4 | 9.5, 12.3, 12.9 | SC-5, SC-7 |

Protégez les ressources Azure contre les attaques de réseaux externes, notamment les attaques par déni de service distribué (DDoS), les attaques spécifiques aux applications et le trafic Internet non sollicité et potentiellement malveillant. Pour ce faire, Azure intègre des fonctionnalités natives :
-   Utilisez le Pare-feu Azure pour protéger les applications et les services contre le trafic potentiellement malveillant provenant d’Internet et d’autres emplacements externes. 

-   Utilisez les fonctionnalités Web Application Firewall (WAF) dans Azure Application Gateway, Azure Front Door et Azure Content Delivery Network (CDN) pour protéger vos applications, services et API contre les attaques de la couche Application. 

-   Protégez vos ressources contre les attaques DDoS en activant la protection DDoS Standard sur vos réseaux virtuels Azure. 
-   Utilisez Azure Security Center pour détecter les risques de configuration incorrecte liés à ce qui précède. 

- [Documentation sur le pare-feu Azure](../../firewall/index.yml)

- [Guide pratique pour déployer Azure WAF](../../web-application-firewall/overview.md)

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](../../ddos-protection/manage-ddos-protection.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)  

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS)

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-5 | 12.6, 12.7 | SI-4 |

Utilisez le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure pour générer des alertes et/ou bloquer le trafic en provenance ou à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence. Quand l’inspection de la charge utile est obligatoire, vous pouvez déployer un système tiers de détection et de prévention d’intrusion (IDS/IPS) à partir de la Place de marché Azure avec des fonctionnalités d’inspection de charge utile. Vous pouvez également utiliser un système IDS/IPS basé sur un hôte ou une solution de détection de point de terminaison et réponse (EDR) basée sur un hôte avec ou au lieu d’un système IDS/IPS basé sur le réseau.  

Remarque : Si vous avez une exigence réglementaire ou autre pour l’utilisation des systèmes IDS/IPS, vérifiez qu’elle est toujours adaptée pour fournir des alertes de haute qualité à votre solution SIEM. 

- [Guide pratique pour déployer le Pare-feu Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [La Place de marché Azure comprend des fonctionnalités de système de détection d’intrusion (IDS) tierces](https://azuremarketplace.microsoft.com/marketplace?search=IDS)

- [Fonctionnalité de détection de point de terminaison et réponse (EDR) de Microsoft Defender ATP](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

## <a name="ns-6-simplify-network-security-rules"></a>NS-6 : Simplifier les règles de sécurité réseau

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-6 | 1.5 | IA-4 |

Simplifiez les règles de sécurité réseau en tirant parti des étiquettes de service et des groupes de sécurité d’application (ASG). 

Utilisez des étiquettes de service de réseau virtuel pour définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou sur le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de l’étiquette de service dans le champ source ou de destination d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Vous pouvez également utiliser des groupes de sécurité d’application pour simplifier les configurations de sécurité complexes. Au lieu de définir une stratégie en fonction d’adresses IP explicites dans des groupes de sécurité réseau, les groupes de sécurité d’application permettent de configurer la sécurité réseau comme un prolongement naturel de la structure d’une application, et donc de regrouper les machines virtuelles et définir des stratégies de sécurité réseau en fonction de ces groupes.

- [Présentation et utilisation des balises de service](../../virtual-network/service-tags-overview.md)

- [Présentation et utilisation des groupes de sécurité d’application](../../virtual-network/network-security-groups-overview.md#application-security-groups)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)    

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="ns-7-secure-domain-name-service-dns"></a>NS-7 : Système DNS (Domain Name System) sécurisé

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| NS-7 | N/A | SC-20, SC-21 |

Suivez les bonnes pratiques concernant la sécurité DNS afin d’atténuer les attaque courantes comme les entrées DNS non résolues, les attaques d’amplifications DNS, l’empoisonnement et l’usurpation d’identité DNS, etc. 

Quand Azure DNS est utilisé comme votre service DNS faisant autorité, vérifiez que les zones et les enregistrements DNS sont protégés contre toute modification accidentelle ou malveillante à l’aide du contrôle d’accès en fonction du rôle (RBAC) Azure et de verrous de ressources. 

- [Vue d’ensemble d’Azure DNS](../../dns/dns-overview.md)

- [Guide de déploiement d’un système DNS (Domain Name System) sécurisé](https://csrc.nist.gov/publications/detail/sp/800-81/2/final)

- [Prévention des entrées DNS non résolues et de l’acquisition de sous-domaine](../fundamentals/subdomain-takeover.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)   

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)
