---
title: Benchmark de sécurité Azure v2 – Sécurité des points de terminaison
description: 'Benchmark de sécurité Azure v2 : Sécurité des points de terminaison'
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 937d0b379c4f669e5b57e5053d5e3bffeb091e78
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368951"
---
# <a name="security-control-v2-endpoint-security"></a>Contrôle de sécurité V2 : Sécurité des points de terminaison

La fonctionnalité Sécurité des points de terminaison couvre les contrôles de détection de point de terminaison et de réponse. Cela comprend l’utilisation de la détection de point de terminaison et réponse (EDR) et du service anti-programme malveillant pour les points de terminaison dans les environnements Azure.

## <a name="es-1-use-endpoint-detection-and-response-edr"></a>ES-1 : Utiliser la détection de point de terminaison et réponse (EDR)

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| ES-1 | 8.1 | SI-2, SI-3, SC-3 |

Activez les capacités de détection de point de terminaison et réponse (EDR) pour les serveurs et les clients, et intégrez-les aux processus SIEM et d’opérations de sécurité.

Microsoft Defender Advanced Threat Protection propose la capacité EDR dans le cadre d’une plateforme de sécurité des points de terminaison d’entreprise pour prévenir et détecter les menaces avancées, enquêter sur elles et y répondre. 

- [Vue d’ensemble de Microsoft Defender Advanced Threat Protection](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)

- [Service Microsoft Defender ATP pour les serveurs Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)

- [Service Microsoft Defender ATP pour les serveurs non Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-endpoints-non-windows)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security)

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-2-use-centrally-managed-modern-anti-malware-software"></a>ES-2 : Utiliser un logiciel anti-programme malveillant moderne géré de manière centralisée

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| ES-2 | 8.1 | SI-2, SI-3, SC-3 |

Utilisez une solution anti-programme malveillant pour les points de terminaison qui est gérée de manière centralisée et capable d’effectuer des analyses en temps réel et périodiques.

Azure Security Center peut identifier automatiquement l’utilisation d’un certain nombre de solutions anti-programmes malveillants populaires pour vos machines virtuelles, signaler l’état de fonctionnement de la protection des points de terminaison et formuler des recommandations. 

Microsoft Antimalware pour Azure Cloud Services est le logiciel anti-programme malveillant par défaut pour les machines virtuelles Windows. Pour les machines virtuelles Linux, utilisez une solution anti-programme malveillant tierce.  En outre, vous pouvez utiliser la détection des menaces pour les services de données d’Azure Security Center afin de détecter les programmes malveillants chargés sur des comptes de stockage Azure. 

- [Guide pratique pour configurer Microsoft Antimalware pour les services cloud et les machines virtuelles](../fundamentals/antimalware.md)

- [Solutions de protection des points de terminaison prises en charge](../../security-center/security-center-services.md?tabs=features-windows#supported-endpoint-protection-solutions-)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security)

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

## <a name="es-3-ensure-anti-malware-software-and-signatures-are-updated"></a>ES-3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| ES-3 | 8,2 | SI-2, SI-3 |

Vérifiez que les signatures anti-programme malveillant sont mises à jour rapidement et de manière cohérente. 

Suivez les recommandations faites dans Azure Security Center : « Calcul et applications » pour mettre à jour les signatures de tous les points de terminaison. Par défaut, Microsoft Antimalware installe automatiquement les dernières signatures et mises à jour du moteur. Pour Linux, utilisez une solution logicielle anti-programme malveillant tierce.

- [Guide pratique pour déployer Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../fundamentals/antimalware.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security)

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

- [Gestion de la conformité de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-compliance-management)

- [Évaluation de la protection de point de terminaison et recommandations dans Azure Security Center](../../security-center/security-center-endpoint-protection.md)