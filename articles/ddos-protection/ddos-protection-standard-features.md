---
title: Fonctionnalités d’Azure DDoS Protection
description: Découvrir les fonctionnalités d’Azure DDoS Protection
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/08/2020
ms.author: yitoh
ms.openlocfilehash: 104c9dcd3b7fd931e4f54841c9de9d17cfd72353
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937319"
---
# <a name="azure-ddos-protection-standard-features"></a>Fonctionnalités Azure DDoS Protection Standard

Les sections suivantes décrivent les principales fonctionnalités du service Azure DDoS Protection Standard.

## <a name="always-on-traffic-monitoring"></a>Analyse permanente du trafic

Le service Protection DDoS Standard surveille l’utilisation du trafic réelle et la compare en permanence aux seuils définis dans la stratégie DDoS. Si le seuil de trafic est dépassé, l’atténuation DDoS est lancée automatiquement. Quand le trafic repasse sous le seuil, l’atténuation est supprimée.

![Atténuation d’Azure DDoS Protection Standard](./media/ddos-protection-overview/mitigation.png)

Pendant l’atténuation, le trafic envoyé vers la ressource protégée est redirigé par le service de protection DDoS et plusieurs vérifications sont effectuées, telles que les suivantes :

- Vérifier que les paquets sont conformes aux spécifications de l’Internet et qu’ils ne sont pas mal formés.
- Interagir avec le client pour déterminer s’il s’agit éventuellement d’un paquet falsifié (par exemple, au moyen des techniques SYN Auth ou SYN Cookie ou en supprimant un paquet afin que la source le retransmette).
- Limiter le débit des paquets si aucune autre méthode de mise en œuvre ne peut être effectuée.

Le service de protection DDoS bloque le trafic d’attaque et transfère le trafic restant vers la destination prévue. Dans les quelques minutes qui suivent la détection d’une attaque, vous êtes informé grâce aux métriques Azure Monitor. Vous pouvez configurer la journalisation des données de télémétrie du service Protection DDos Standard de manière à écrire les journaux d’activité des options disponibles en vue d’une analyse future. Les données des métriques dans Azure Monitor pour le service Protection DDoS Standard sont conservées pendant 30 jours.

## <a name="adaptive-real-time-tuning"></a>Réglage adaptatif en temps réel

Le service Azure DDoS Protection Basic contribue à protéger les clients et évite d’impacter d’autres clients. Par exemple, si un service est provisionné pour un volume de trafic entrant légitime inférieur au *taux de déclenchement* de la stratégie de protection DDoS à l’échelle de l’infrastructure, une attaque DDoS sur les ressources du client peut passer inaperçue. De manière plus générale, la complexité des récentes attaques (comme les attaques DDoS multivectorielles) et les comportements spécifiques aux applications des locataires demandent des stratégies de protection personnalisées par client. Le service effectue cette personnalisation en s’appuyant sur deux éléments :

- L’apprentissage automatique des modèles de trafic par client (par adresse IP) pour les couches 3 et 4.

- La réduction des faux positifs dans la mesure où l’échelle d’Azure permet d’absorber une quantité importante de trafic.

![Schéma du fonctionnement de DDoS Protection Standard, avec la génération de stratégie entourée](./media/ddos-best-practices/image-5.png)

## <a name="ddos-protection-telemetry-monitoring-and-alerting"></a>Protection DDoS : télémétrie, monitoring et génération d’alertes

DDoS Protection Standard expose des données de télémétrie riches par le biais [d’Azure Monitor](../azure-monitor/overview.md) pendant la durée d’une attaque DDoS. Vous pouvez configurer des alertes pour une des mesures d’Azure Monitor utilisée par DDoS Protection. Vous pouvez intégrer la journalisation à Splunk (Azure Event Hubs), Journaux Azure Monitor et Stockage Azure pour l’analyse avancée via l’interface des diagnostics d’Azure Monitor.

### <a name="ddos-mitigation-policies"></a>Stratégies d’atténuation des risques liés à DDoS

Dans le portail Azure, cliquez sur **Surveiller** > **Métriques**. Dans le volet **Métriques**, sélectionnez le groupe de ressources, le type de ressource **Adresse IP publique** et votre adresse IP publique Azure. Les métriques DDoS sont visibles dans le volet **Métriques disponibles**.

DDoS Protection Standard applique trois stratégies de prévention réglées automatiquement (TCP SYN, TCP et UDP) pour chaque adresse IP publique de la ressource protégée, dans le réseau virtuel sur lequel la protection DDoS est activée. Vous pouvez afficher les seuils de stratégie en sélectionnant la métrique **Paquets entrants pour déclencher l’atténuation DDoS**.

![Métriques disponibles et graphique des métriques](./media/ddos-best-practices/image-7.png)

Les seuils de stratégie sont configurés automatiquement par le biais du système de profilage du trafic réseau basé sur l’apprentissage automatique. L’atténuation des risques liés à DDoS pour une adresse IP n’a lieu que si le seuil de stratégie est dépassé.

### <a name="metric-for-an-ip-address-under-ddos-attack"></a>Métrique d’une adresse IP faisant l’objet d’une attaque DDoS

Si l’adresse IP publique est attaquée, la valeur de la métrique **Sous attaque DDoS ou non** passe à 1 à mesure que DDoS Protection effectue les opérations d’atténuation sur le trafic de l’attaque.

![Métrique « Sous attaque DDoS ou non » et graphique](./media/ddos-best-practices/image-8.png)

Nous vous recommandons de configurer une alerte sur cette métrique. Vous serez alors averti en cas d’atténuation DDoS active sur votre adresse IP publique.

Pour plus d’informations, consultez [Gérer Azure DDoS Protection Standard à l’aide du portail Azure](manage-ddos-protection.md).

## <a name="web-application-firewall-for-resource-attacks"></a>Pare-feu d’applications web pour les attaques sur les ressources

Pour lutter spécifiquement contre les attaques sur les ressources au niveau de la couche application et renforcer la sécurité des applications web, vous devez configurer le pare-feu d’applications web (WAF). WAF inspecte le trafic web entrant pour bloquer les injections SQL, les scripts intersites, les attaques DDoS et autres attaques au niveau de la couche 7. [WAF est une fonctionnalité d’Application Gateway](../web-application-firewall/ag/ag-overview.md) fournie par Azure pour protéger de manière centralisée vos applications web contre les vulnérabilités et exploits courants. Les partenaires Azure proposent d’autres offres WAF. Pour trouver celle la mieux adaptée à vos besoins, visitez la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps?search=WAF&page=1).

Même les pare-feu d’application web sont vulnérables aux attaques volumétriques et d’épuisement d’état. Nous vous recommandons fortement d’activer DDoS Protection Standard sur le réseau virtuel WAF pour contribuer à vous protéger contre les attaques volumétriques et de protocole. Pour plus d’informations, consultez la section sur [les architectures de référence DDoS Protection](ddos-protection-reference-architectures.md).

## <a name="protection-planning"></a>Planification de la protection

La planification et la préparation sont des étapes cruciales pour comprendre comment un système se comporte en cas d’attaque DDoS. La conception d’un plan de réponse de gestion des incidents fait partie de cet effort.

Si vous disposez de DDoS Protection Standard, assurez-vous qu’il est activé sur le réseau virtuel des points de terminaison accessibles sur Internet. La configuration d’alertes DDoS vous permet d’être en permanence attentif aux attaques potentielles sur votre infrastructure. 

Surveillez vos applications de manière indépendante. Vous devez comprendre le comportement normal d’une application. Préparez-vous à intervenir si l’application ne se comporte pas comme prévu durant une attaque DDoS. 

Découvrez comment vos services répondent à une attaque en effectuant [des tests par le biais de simulations](test-through-simulations.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer un plan de protection DDoS](manage-ddos-protection.md).