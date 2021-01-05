---
title: Composants d’une stratégie de réponse DDoS
description: Découvrez comment utiliser Azure DDoS Protection Standard pour répondre aux attaques DDoS.
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 0a80f03ee82e8d1216353482dc867402adcf7d09
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992231"
---
# <a name="components-of-a-ddos-response-strategy"></a>Composants d’une stratégie de réponse DDoS

Une attaque DDoS ciblant des ressources Azure nécessite généralement une intervention minimale de l’utilisateur. Néanmoins, le fait d’incorporer l’atténuation des risques liés à DDoS à la stratégie de réponse aux incidents aide à limiter l’impact sur l’activité.

## <a name="microsoft-threat-intelligence"></a>Microsoft Threat Intelligence

Microsoft dispose d’un réseau complet d’informations sur les menaces. Celui-ci s’appuie sur les connaissances d’une communauté de spécialistes de la sécurité prenant en charge les services en ligne de Microsoft, sur les partenaires Microsoft et sur les relations au sein de la communauté dédiée à la sécurité d’Internet. 

En tant que fournisseur d’infrastructures critiques, Microsoft reçoit à l’avance des avertissements sur les menaces. Microsoft collecte des informations sur les menaces à partir de ses services en ligne et de sa base client globale. Microsoft incorpore tous ces renseignements sur les menaces dans les produits Azure DDoS Protection.

En outre, Microsoft Digital Crimes Unit (DCU) met en place des stratégies offensives contre les botnets. Les botnets constituent une source courante de commande et de contrôle pour les attaques DDoS.

## <a name="risk-evaluation-of-your-azure-resources"></a>Évaluation des risques de vos ressources Azure

Il est impératif de comprendre en permanence la portée des risques liés à une attaque DDoS. Demandez-vous régulièrement :

- Quelles nouvelles ressources Azure disponibles publiquement doivent être protégées ?

- Existe-t-il un point de défaillance unique dans le service ? 

- Comment faire pour isoler les services de manière à limiter l’impact d’une attaque tout en les rendant accessibles aux clients valides ?

- Y a-t-il des réseaux virtuels dans lesquels le service DDoS Protection Standard n’est pas activé alors qu’il devrait l’être ? 

- Mes services sont-ils actifs/actifs avec basculement dans plusieurs régions ?

Il est essentiel que vous compreniez le comportement normal d’une application et que vous vous prépariez à agir si l’application ne se comporte pas comme prévu au cours d’une attaque DDoS. Configurez des moniteurs pour vos applications vitales pour l'entreprise qui imitent le comportement du client et vous avertissent en cas de détection d’anomalies pertinentes. Pour obtenir des informations sur l’intégrité de votre application, reportez-vous aux [meilleures pratiques en matière de surveillance et de diagnostic](/azure/architecture/best-practices/monitoring#monitoring-and-diagnostics-scenarios).

[Azure Application Insights](../azure-monitor/app/app-insights-overview.md) est un service extensible de gestion des performances des applications destiné aux développeurs web sur de multiples plateformes. Utilisez Application Insights pour superviser votre application web en direct. Il détecte automatiquement les problèmes de performances. Il intègre des outils d’analyse pour vous aider à diagnostiquer les problèmes et à comprendre ce que les utilisateurs font avec votre application. Il a été conçu pour vous permettre d’améliorer continuellement les performances et la convivialité.

## <a name="customer-ddos-response-team"></a>Équipe de réponse aux attaques DDoS du client

La création d’une équipe de réponse DDoS est essentielle pour garantir une réponse rapide et efficace en cas d’attaque. Dans votre organisation, identifiez plusieurs contacts qui seront en charge de la planification et de l’exécution. Cette équipe de réponse DDoS doit comprendre en détail le service Azure DDoS Protection Standard. Assurez-vous que l’équipe est en mesure d’identifier et d’atténuer une attaque en coordination avec les clients internes et externes, y compris l’équipe de support Microsoft. 

Nous vous recommandons d’utiliser des exercices de simulation comme composant normal de la planification de la continuité et de la disponibilité de votre service. Ces exercices doivent comprendre des tests de mise à l'échelle. Pour savoir comment simuler le trafic de test DDoS sur vos points de terminaison publics Azure, consultez [Tester à l’aide de simulations](test-through-simulations.md).

## <a name="alerts-during-an-attack"></a>Alertes durant une attaque

Azure DDoS Protection Standard identifie et atténue les attaques DDoS sans aucune intervention de l’utilisateur. Pour être averti en cas d’atténuation active pour une adresse IP publique protégée, vous pouvez [configurer une alerte](telemetry-monitoring-alerting.md) sur la métrique **Sous attaque DDoS ou non**. Vous pouvez choisir de créer des alertes pour les autres métriques DDoS afin de comprendre l’échelle des attaques, le trafic abandonné, etc.

### <a name="when-to-contact-microsoft-support"></a>Quand contacter le support Microsoft

Les clients Azure DDoS Protection Standard ont accès à l’équipe de réponse rapide DDoS (DRR), qui peut aider à faire des recherches sur une attaque au cours d’une attaque, ainsi qu’une analyse après l’attaque. Consultez [Réponse rapide DDoS](ddos-rapid-response.md) pour plus d’informations, notamment le moment où vous devez faire appel à l’équipe DRR.

## <a name="post-attack-steps"></a>Étapes à suivre après l’attaque

Il est toujours bon d’effectuer un compte-rendu après une attaque et d’ajuster la stratégie de réponse DDoS au besoin. Points importants à prendre en compte :

- Le manque d’extensibilité de l’architecture a-t-il perturbé le service ou l’expérience utilisateur ?

- Quels sont les services ou les applications qui ont souffert le plus ?

- Quel a été le degré d’efficacité de la stratégie de réponse DDoS et comment peut-il être amélioré ?

Si vous pensez que vous faites l’objet d’une attaque DDoS, transmettez le problème aux canaux de support Azure habituels.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer un plan de protection DDoS](manage-ddos-protection.md).