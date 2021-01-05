---
title: Contrôle de sécurité Azure - Sécurité réseau
description: Contrôle de sécurité Azure – Sécurité réseau
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 3a232f8e8c35e265a8243ac79e465c03f6b9650e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487862"
---
# <a name="security-control-network-security"></a>Contrôle de sécurité : Sécurité réseau

Les recommandations en matière de sécurité réseau se concentrent sur la spécification des protocoles réseau, des ports TCP/UDP et des services connectés au réseau auxquels l'accès aux services Azure est autorisé ou refusé.

## <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.1 | 9.2, 9.4, 14.1, 14.2, 14.3 | Customer |

Assurez-vous qu'un groupe de sécurité réseau a été appliqué à tous les déploiements de sous-réseaux du réseau virtuel avec des contrôles d'accès réseau spécifiques aux ports et sources approuvés de votre application. Lorsque c’est possible, utilisez des points de terminaison privés avec une liaison privée pour sécuriser vos ressources de service Azure sur votre réseau virtuel en étendant l’identité du réseau virtuel à ce service. Si les points de terminaison privés et la liaison privée ne sont pas disponibles, utilisez des points de terminaison de service. Pour les exigences spécifiques au service, reportez-vous à la recommandation de sécurité relative à ce service. 

Pour les cas d’usage spécifiques, vous pouvez également satisfaire aux exigences en implémentant le Pare-feu Azure.

- [Présentation des points de terminaison de service de réseau virtuel](../../virtual-network/virtual-network-service-endpoints-overview.md)

- [Présentation d’Azure Private Link](../../private-link/private-link-overview.md)

- [Guide pratique pour créer un réseau virtuel](../../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../../virtual-network/tutorial-filter-network-traffic.md)

- [Guide pratique pour déployer et configurer le Pare-feu Azure](../../firewall/tutorial-firewall-deploy-portal.md)

## <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.2 | 9.3, 12.2, 12.8 | Customer |

Utilisez Azure Security Center et suivez les recommandations de protection du réseau pour sécuriser vos ressources réseau dans Azure. Activez les journaux de flux NSG et transférez-les vers un compte de stockage pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux dans un espace de travail Log Analytics et utiliser Traffic Analytics pour obtenir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

- [Guide pratique pour activer les journaux de flux NSG](../../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../../network-watcher/traffic-analytics.md)

- [Présentation de la sécurité réseau assurée par Azure Security Center](../../security-center/security-center-network-recommendations.md)

## <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.3 | 9.5 | Customer |

Déployez un pare-feu d'applications web (WAF) Azure devant les applications web critiques pour bénéficier d'un contrôle supplémentaire du trafic entrant. Activez le paramètre de diagnostic du WAF et ingérez les journaux dans un compte de stockage, un hub d'événements ou un espace de travail Log Analytics.

- [Guide pratique pour déployer Azure WAF](../../web-application-firewall/ag/create-waf-policy-ag.md)

## <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.4 | 12.3 | Customer |

Activez la protection DDoS Standard sur vos réseaux virtuels Azure pour vous protéger des attaques DDoS. Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications présentant des adresses IP connues comme étant malveillantes.

Déployez le Pare-feu Azure à chaque extrémité du réseau de l'organisation en activant la fonctionnalité de renseignement sur les menaces et en la configurant sur « Alerter et refuser » pour vous protéger de tout trafic réseau malveillant.

Utilisez la fonctionnalité d'accès réseau juste-à-temps d'Azure Security Center pour configurer les groupes de sécurité réseau (NSG) et limiter l'exposition des points de terminaison aux adresses IP approuvées pendant une période limitée.

Utilisez la fonctionnalité de renforcement du réseau adaptatif d'Azure Security Center pour recommander des configurations NSG qui limitent les ports et les adresses IP sources en fonction du trafic réel et du renseignement sur les menaces.

- [Guide pratique pour configurer la protection DDoS](../../ddos-protection/manage-ddos-protection.md)

- [Guide pratique pour déployer le Pare-feu Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [Présentation de la fonctionnalité Threat Intelligence intégrée à Azure Security Center](../../security-center/azure-defender.md)

- [Présentation de la fonctionnalité de renforcement du réseau adaptatif d’Azure Security Center](../../security-center/security-center-adaptive-network-hardening.md)

- [Présentation de la fonctionnalité de contrôle d’accès réseau juste-à-temps d’Azure Security Center](../../security-center/security-center-just-in-time.md)

## <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.5 | 12.5 | Customer |

Activez la capture de paquets Network Watcher pour analyser les activités anormales.

- [Guide pratique pour activer Network Watcher](../../network-watcher/network-watcher-create.md)

## <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.6 | 12.6, 12.7 | Customer |

Sélectionnez une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile.  Si la détection et/ou la prévention des intrusions basées sur l’inspection de la charge utile ne font pas partie des exigences, vous pouvez utiliser le Pare-feu Azure avec Threat Intelligence. Le filtrage basé sur le renseignement sur les menaces du Pare-feu Azure peut générer des alertes et refuser le trafic depuis ou vers des adresses IP et des domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence.

Déployez la solution de pare-feu de votre choix dans les limites réseau de votre organisation pour détecter et/ou refuser le trafic malveillant.

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall)

- [Guide pratique pour déployer le Pare-feu Azure](../../firewall/tutorial-firewall-deploy-portal.md)

- [Guide pratique pour configurer des alertes avec le Pare-feu Azure](../../firewall/threat-intel.md)

## <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.7 | 12.9, 12.10 | Customer |

Déployez Azure Application Gateway pour les applications web en activant le protocole HTTPS/TLS pour les certificats approuvés.

- [Guide pratique pour déployer Application Gateway](../../application-gateway/quick-create-portal.md)

- [Guide pratique pour configurer Application Gateway de façon à utiliser le protocole HTTPS](../../application-gateway/create-ssl-portal.md)

- [Présentation de l’équilibrage de charge de niveau 7 avec les passerelles d’applications web Azure](../../application-gateway/overview.md)

## <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.8 | 1.5 | Customer |

Utilisez des balises de service de réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou le Pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Vous pouvez également utiliser des groupes de sécurité d’application pour simplifier les configurations de sécurité complexes. Les groupes de sécurité d’application permettent de configurer la sécurité réseau comme un prolongement naturel de la structure de l’application, et donc de regrouper les machines virtuelles et définir des stratégies de sécurité réseau basés sur ces groupes.

- [Présentation et utilisation des balises de service](../../virtual-network/service-tags-overview.md)

- [Présentation et utilisation des groupes de sécurité d’application](../../virtual-network/network-security-groups-overview.md#application-security-groups)

## <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.9 | 11,1 | Customer |

Définissez et implémentez des configurations de sécurité standard pour les ressources réseau à l'aide d'Azure Policy.

Vous pouvez également utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les principaux artefacts d’environnement (par exemple, les modèles Resource Manager, les contrôles Azure RBAC et les stratégies) dans une même définition de blueprint. Vous pouvez appliquer le blueprint aux nouveaux abonnements et ajuster le contrôle et la gestion grâce au contrôle de version.

- [Guide pratique pour configurer et gérer Azure Policy](../../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../../governance/policy/samples/built-in-policies.md#network)

- [Guide pratique pour créer un blueprint Azure](../../governance/blueprints/create-blueprint-portal.md)

## <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1,10 | 11.2 | Customer |

Utilisez des balises pour les groupes de sécurité réseau (NSG) et autres ressources liées à la sécurité réseau et au trafic. Concernant les règles NSG individuelles, utilisez le champ « Description » afin de spécifier le besoin métier et/ou la durée (etc.) pour toutes les règles qui autorisent le trafic vers/depuis un réseau.

Utilisez l’une des définitions de stratégie Azure intégrée en lien avec l’étiquetage comme « Exiger une étiquette et sa valeur » pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Vous pouvez utiliser Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../../azure-resource-manager/management/tag-resources.md)

- [Guide pratique pour créer un réseau virtuel](../../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../../virtual-network/tutorial-filter-network-traffic.md)

## <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

| Identifiant Azure | Identifiants CIS | Responsabilité |
|--|--|--|
| 1.11 | 11.3 | Customer |

Utilisez le journal d’activité Azure pour effectuer le monitoring des configurations de ressources et détecter les changements apportés à vos ressources Azure. Créez des alertes dans Azure Monitor, qui se déclenchent en cas de modification des ressources critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../../azure-monitor/platform/alerts-activity-log.md)

## <a name="next-steps"></a>Étapes suivantes

- Consultez le contrôle de sécurité suivant : [Journalisation et supervision](security-control-logging-monitoring.md)