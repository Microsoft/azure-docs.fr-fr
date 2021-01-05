---
title: Configurer et personnaliser le module de sécurité pour Azure RTOS
description: Découvrez comment configurer et personnaliser votre module de sécurité pour Azure RTOS.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 9106e00fe2146978f97b480e3afd3b7ed58c9130
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929932"
---
# <a name="configure-and-customize-security-module-for-azure-rtos-preview"></a>Configurer et personnaliser le module de sécurité pour Azure RTOS (préversion)

Utilisez le fichier suivant pour configurer le comportement de votre appareil.

## <a name="azure_iot_security_moduleincasc_porth"></a>azure_iot_security_module/inc/asc_port.h

 Le comportement par défaut de chaque configuration est fourni dans les tableaux suivants : 

### <a name="general"></a>Général

| Nom | Type | Default | Détails |
| - | - | - | - |
| ASC_SECURITY_MODULE_ID | Chaîne | --- | Identificateur unique de l’appareil  |
| ASC_SECURITY_MODULE_PENDING_TIME  | Numéro | 300 | Temps d’attente du module de sécurité en secondes. Si le délai est dépassé, l’état passe à Interrompu. |

#### <a name="collection"></a>Collection

| Nom | Type | Default | Détails |
| - | - | - | - |
| ASC_HIGH_PRIORITY_INTERVAL | Numéro | 10 | Intervalle en secondes du groupe à haute priorité des collecteurs. |
| ASC_MEDIUM_PRIORITY_INTERVAL | Numéro | 30 | Intervalle en secondes du groupe de priorité moyenne des collecteurs. |
| ASC_LOW_PRIORITY_INTERVAL | Numéro | 145 440  | Intervalle en secondes du groupe à basse priorité des collecteurs. |

#### <a name="collector-network-activity"></a>Activité du réseau de collecteurs

Pour personnaliser la configuration de l’activité du réseau de collecteurs, utilisez ce qui suit :

| Nom | Type | Default | Détails |
| - | - | - | - |
| ASC_COLLECTOR_NETWORK_ACTIVITY_TCP_DISABLED | Boolean | false | Filtre `TCP` activité de réseau |
| ASC_COLLECTOR_NETWORK_ACTIVITY_UDP_DISABLED | Boolean | false | Filtre `UDP` événements de l’activité de réseau |
| ASC_COLLECTOR_NETWORK_ACTIVITY_ICMP_DISABLED | Boolean | false | Filtre `ICMP` événements de l’activité de réseau |
| ASC_COLLECTOR_NETWORK_ACTIVITY_CAPTURE_UNICAST_ONLY | Boolean | true | Capturer les paquets entrants en monodiffusion uniquement, quand la valeur false capture est également définie sur Diffusion et Multidiffusion |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV4_OBJECTS_IN_CACHE | Numéro | 64 | Nombre maximal d’événements réseau IPv4 à stocker en mémoire |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV6_OBJECTS_IN_CACHE | Numéro | 64  | Nombre maximal d’événements réseau IPv6 à stocker en mémoire |


## <a name="compile-flags"></a>Indicateurs de compilation
Les indicateurs de compilation vous permettent de remplacer les configurations prédéfinies.

### <a name="collectors"></a>Collecteurs
| Nom | Type | Default | Détails |
| - | - | - | - |
| collector_heartbeat_enabled | Boolean | ACTIVÉ | Activer le collecteur de pulsations |
| collector_network_activity_enabled | Boolean | ACTIVÉ | Activer le collecteur d’activité réseau |
| collector_system_information_enabled | Boolean | ACTIVÉ | Activer le collecteur d’informations système |

## <a name="supported-security-alerts-and-recommendations"></a>Alertes et recommandations de sécurité prises en charge

Le module de sécurité pour Azure RTOS prend en charge des alertes de sécurité et des recommandations spécifiques. Veillez à [passer en revue et à personnaliser les valeurs d’alerte et de recommandation pertinentes](concept-rtos-security-alerts-recommendations.md) pour votre service.

## <a name="log-analytics-optional"></a>Analytique des journaux d'activité (facultatif)

Bien que facultatives et non obligatoires, l’activation et la configuration de l’analytique des journaux d’activité peuvent être utiles lorsque vous souhaitez en savoir plus sur les événements et activités des appareils. Découvrez comment configurer et utiliser [l’analytique des journaux d'activité avec le service Defender pour IoT](how-to-security-data-access.md#log-analytics). 

## <a name="next-steps"></a>Étapes suivantes

- Passez en revue et personnalisez [les alertes et les recommandations de sécurité](concept-rtos-security-alerts-recommendations.md) du module de sécurité pour Azure RTO.
- Consultez si nécessaire [le module de sécurité pour l’API Azure RTOS](azure-rtos-security-module-api.md).

