---
title: Sécurité des données Azure Security Center| Microsoft Docs
description: Ce document explique comment les données sont gérées et protégées dans le Centre de sécurité Azure.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 33f2c9f4-21aa-4f0c-9e5e-4cd1223e39d7
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/13/2020
ms.author: memildin
ms.openlocfilehash: 49533947ff01aea07eaacd9d761b6414fb672a1c
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92339473"
---
# <a name="azure-security-center-data-security"></a>Sécurité des données Azure Security Center

Pour aider les clients à prévenir, détecter et contrer les menaces, Azure Security Center collecte et traite des données de sécurité, notamment des informations de configuration, des métadonnées et des journaux d’événements. Microsoft adhère aux recommandations strictes de conformité et de sécurité, du codage jusqu'à l'exploitation d'un service.

Cet article explique comment les données sont gérées et protégées dans le Centre de sécurité.

## <a name="data-sources"></a>Sources de données
Pour assurer une visibilité de l’état de la sécurité, identifier les vulnérabilités, recommander des mesures d’atténuation et détecter des menaces actives, le Centre de sécurité analyse les données provenant des sources suivantes :

- **Services Azure** : utilise les informations relatives à la configuration des services Azure que vous avez déployés en communiquant avec le fournisseur de ressources de ce service.
- **Trafic réseau** : tire parti des métadonnées de trafic réseau échantillonnées provenant de l'infrastructure de Microsoft, telles que l'IP/le port source/de destination, la taille de paquet et le protocole réseau.
- **Solutions de partenaires** : collecte les alertes de sécurité des solutions de partenaires intégrées, telles que les solutions de pare-feu et anti-programme malveillant.
- **Vos machines** : utilise les détails de configuration et les informations relatives aux événements de sécurité, telles que les événements Windows, les journaux d’audit et les messages syslog de vos machines.


## <a name="data-protection"></a>Protection des données

### <a name="data-segregation"></a>Ségrégation des données
les données client sont maintenues séparées logiquement sur chaque composant, dans l'ensemble du service. Toutes les données sont balisées en fonction de l'organisation. Ce balisage est conservé tout au long du cycle de vie des données, et il est appliqué dans chaque couche du service.

### <a name="data-access"></a>Accès aux données
Pour fournir des recommandations en matière de sécurité et enquêter sur les éventuelles menaces de sécurité, le personnel de Microsoft peut accéder aux informations collectées ou analysées par les services Azure, notamment les événements de création de processus et artefacts de disque de machine virtuelle, qui peuvent involontairement exposer des données client ou personnelles provenant de vos machines. 

Nous respectons l’[Addendum relatif à la protection de Microsoft Online Services](https://www.microsoftvolumelicensing.com/Downloader.aspx?DocumentId=17880), qui indique que Microsoft n’utilisera pas les données client ou ne tirera pas d’informations de ces dernières à des fins commerciales, publicitaires ou similaires. Nous utilisons uniquement les données client en fonction des besoins pour vous proposer des services Azure, notamment des utilisations compatibles avec la fourniture de ces services. Vous conservez tous les droits sur les données client.

### <a name="data-use"></a>Utilisation des données
Microsoft utilise des modèles et des informations sur les menaces observées auprès de multiples locataires pour améliorer ses fonctionnalités de prévention et de détection. Cette utilisation s'effectue en accord avec les engagements de confidentialité décrits dans la [Déclaration de confidentialité](https://privacy.microsoft.com/privacystatement) de Microsoft.

## <a name="manage-data-collection-from-machines"></a>Gérer la collecte de données à partir de machines
Lorsque vous activez Security Center dans Azure, la collecte de données est activée pour chacun de vos abonnements Azure. Vous pouvez également activer la collecte de données pour vos abonnements dans le Centre de sécurité. Lorsque la collecte de données est activée, le Centre de sécurité approvisionne l’agent Log Analytics sur toutes les machines virtuelles Azure prises en charge préexistantes et nouvellement créées.

L'agent Log Analytics analyse diverses configurations de sécurité et crée des événements sous la forme de traces de [Suivi d’événements pour Windows (ETW)](/windows/win32/etw/event-tracing-portal). En outre, le système d’exploitation déclenche des événements du Journal des événements au cours de l’exécution de la machine. Il peut s’agir des données suivantes : type et version de système d’exploitation, journaux d’activité de système d’exploitation (journaux d’événements Windows), processus en cours d’exécution, nom de machine, adresses IP, utilisateur connecté et ID de locataire. L'agent Log Analytics lit les entrées du journal des événements et les traces ETW, puis les copie dans vos espaces de travail à des fins d’analyse. L’agent Log Analytics active également les événements de création de processus et l’audit de ligne de commande.

Si vous n’utilisez pas Azure Defender, vous pouvez également désactiver la collecte de données à partir des machines virtuelles dans la stratégie de sécurité. La collecte des données est obligatoire pour les abonnements protégés par Azure Defender. La collecte des artefacts et des captures instantanées des disques de machine virtuelle reste activée, même si la collecte de données est désactivée.

Vous pouvez spécifier l’espace de travail et la région où sont stockées les données collectées à partir de vos machines. La valeur par défaut consiste à stocker les données collectées à partir de vos machines dans l’espace de travail le plus proche, comme indiqué dans le tableau suivant :

| Zone géographique de machine virtuelle                                      | Zone géographique d’espace de travail  |
|---------------------------------------------|----------------|
| États-Unis, Brésil, Afrique du Sud         | États-Unis  |
| Canada                                      | Canada         |
| Europe (à l’exclusion du Royaume-Uni)           | Europe         |
| Royaume-Uni                              | Royaume-Uni |
| Asie (à l’exception de l’Inde, du Japon, de la Corée, de la Chine) | Asie-Pacifique   |
| Corée du Sud                                       | Asie-Pacifique   |
| Inde                                       | Inde          |
| Japon                                       | Japon          |
| Chine                                       | Chine          |
| Australie                                   | Australie      |
|                                             |                |

> [!NOTE]
> **Azure Defender pour le stockage** stocke les artefacts de façon régionale en fonction de l’emplacement de la ressource Azure associée. Pour en savoir plus, consultez [Présentation d’Azure Defender pour le stockage](defender-for-storage-introduction.md).


## <a name="data-consumption"></a>Consommation des données

Les clients peuvent accéder à des données associées du Centre de sécurité à partir des flux de données suivants :


| STREAM                                                                                | Types de données                                                                                                                                                                                                          |
|---------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Journal d’activité Azure](../azure-monitor/platform/activity-log.md)                       | Toutes les alertes de sécurité, les requêtes approuvées d’accès [juste-à-temps](security-center-just-in-time.md) au Centre de sécurité, et toutes les alertes générées par les [contrôles d’application adaptatifs](security-center-adaptive-application.md).|
| [Journaux d’activité Azure Monitor](../azure-monitor/platform/data-platform.md)                      | Toutes les alertes de sécurité.                                                                                                                                                                                                |
| [Azure Resource Graph](../governance/resource-graph/overview.md)                      | Alertes de sécurité, recommandations de sécurité, résultats d’évaluation des vulnérabilités, informations sur le degré de sécurisation, état des vérifications de conformité, et bien plus encore.                                                                       |
| [API REST Azure Security Center](/rest/api/securitycenter/) | Alertes de sécurité, recommandations de sécurité et bien plus encore.                                                                                                                                                                |
|                                                                                       |                                                                                                                                                                                                                     |

## <a name="next-steps"></a>Étapes suivantes

Ce document explique comment les données sont gérées et protégées dans le Centre de sécurité Azure. 

Pour obtenir des informations générales sur le Centre de sécurité Azure, consultez [Qu’est-ce que le Centre de sécurité Azure ?](security-center-introduction.md).