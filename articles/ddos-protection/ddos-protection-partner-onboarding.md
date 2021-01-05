---
title: Partenariat avec Azure DDoS Protection Standard
description: Découvrez les opportunités de partenariat offertes par Azure DDoS Protection Standard.
ms.service: ddos-protection
documentationcenter: na
author: yitoh
mms.devlang: na
ms.topic: how-to
ms.date: 08/28/2020
ms.author: kumud
ms.openlocfilehash: 4cc0fd970f9adc26f5fdb024f501dcb0edf77bde
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992418"
---
# <a name="partnering-with-azure-ddos-protection-standard"></a>Partenariat avec Azure DDoS Protection Standard
Cet article décrit les opportunités de partenariat offertes par Azure DDoS Protection Standard. Il est conçu pour aider les chefs de produits et les personnes en charge du développement commercial à comprendre les parcours d’investissement et à fournir des insights sur les propositions de valeur de partenariat.

## <a name="background"></a>Arrière-plan
Les attaques par déni de service distribué (DDoS) représentent l’un des problèmes de disponibilité et de sécurité majeurs soulevés par les clients qui déplacent leurs applications vers le cloud. L’extorsion et le « hacktivisme » sont des motivations courantes d’attaques DDoS, et elles présentent une augmentation constante en termes de type, d’échelle et de fréquence des occurrences, car elles sont relativement faciles et peu onéreuses à lancer.

Azure DDoS Protection propose des mesures contre les menaces DDoS les plus sophistiquées, en tirant parti de l’échelle mondiale des réseaux Azure. Le service fournit des fonctionnalités améliorées d’atténuation des attaques DDoS pour les applications et les ressources déployées sur des réseaux virtuels.

Les partenaires informatiques peuvent protéger les ressources de leurs clients de manière native avec Azure DDoS Protection Standard afin de répondre aux problèmes de disponibilité et de fiabilité dus aux attaques DDoS.

## <a name="introduction-to-azure-ddos-protection-standard"></a>Présentation d’Azure DDoS Protection Standard
Azure DDoS Protection Standard offre des fonctionnalités améliorées d’atténuation DDoS contre les attaques DDoS de couche 3 et de couche 4. Voici les principales fonctionnalités du service DDoS Protection Standard :

### <a name="adaptive-real-time-tuning"></a>Réglage adaptatif en temps réel
Pour chaque application protégée, Azure DDoS Protection Standard ajuste automatiquement les seuils de stratégie d’atténuation DDoS en fonction des modèles de profil de trafic de l’application. Le service effectue cette personnalisation en s’appuyant sur deux éléments :

- L’apprentissage automatique des modèles de trafic par client (par adresse IP) pour les couches 3 et 4.
- La réduction des faux positifs dans la mesure où l’échelle d’Azure permet d’absorber une quantité importante de trafic.

![Réglage adaptatif en temps réel](./media/ddos-protection-partner-onboarding/real-time-tuning.png)

### <a name="attack-analytics-telemetry-monitoring-and-alerting"></a>Analytique des attaques, télémétrie, supervision et alertes
Azure DDoS Protection identifie et atténue les attaques DDoS sans aucune intervention de l’utilisateur.

- Si la ressource protégée se trouve dans l’abonnement couvert par Azure Security Center, DDoS Protection Standard envoie automatiquement une alerte à Security Center chaque fois qu’une attaque DDoS est détectée et atténuée sur l’application protégée.
- En guise d’alternative, pour être notifié en cas d’atténuation active pour une adresse IP publique protégée, vous pouvez [configurer une alerte](telemetry-monitoring-alerting.md#configure-alerts-for-ddos-protection-metrics) sur la métrique Sous attaque DDoS ou non.
- Vous pouvez également choisir de créer des alertes pour les autres métriques DDoS et [configurer l’analytique des attaques](telemetry-monitoring-alerting.md) afin de comprendre l’ampleur de l’attaque, le trafic abandonné, les vecteurs d’attaque, les principaux contributeurs et d’autres détails.

![Métriques DDoS](./media/ddos-protection-partner-onboarding/ddos-metrics.png)

### <a name="ddos-rapid-response-drr"></a>DDoS Rapid Response (DRR)
Les clients DDoS Protection Standard peuvent contacter l’[équipe du service Rapid Response](ddos-rapid-response.md) pendant une attaque active. Le service DRR peut aider en enquêtant sur l’attaque lorsqu’elle survient et en publiant une analyse de cette dernière.

### <a name="sla-guarantee-and-cost-protection"></a>Garantie du contrat SLA et protection des coûts
Le service DDoS Protection Standard est couvert par un contrat SLA de 99,99 %, et la protection des coûts fournit des crédits de ressources pour le scale-out lors d’une attaque documentée. Pour plus d’informations, consultez [SLA pour Protection DDoS Azure](https://azure.microsoft.com/support/legal/sla/ddos-protection/v1_0/).

## <a name="featured-partner-scenarios"></a>Scénarios de partenaires
Voici les principaux avantages dont vous pouvez bénéficier par l’intégration à Azure DDoS Protection Standard :

- Les services proposés par les partenaires (équilibreur de charge, pare-feu d’applications web, pare-feu, etc.) à leurs clients sont protégés automatiquement (marqués en blanc) par Azure DDoS Protection Standard dans le back-end.
- Les partenaires ont accès à l’analytique des attaques et à la télémétrie Azure DDoS Protection Standard qu’ils peuvent intégrer à leurs propres produits, offrant ainsi une expérience client unifiée.  
- Les partenaires ont accès au support de réponse rapide DDoS, même en l’absence de réponse rapide Azure, pour les problèmes liés à DDoS.
- Les applications protégées des partenaires bénéficient d’une garantie de contrat SLA DDoS et d’une protection des coûts en cas d’attaques DDoS.

## <a name="technical-integration-overview"></a>Présentation de l’intégration technique
Les opportunités de partenariat Azure DDoS Protection Standard sont mises à disposition par le biais du portail Azure, des API et de l’interface CLI/PS.

### <a name="integrate-with-ddos-protection-standard"></a>Intégrer à DDoS Protection Standard
La configuration de l’intégration avec Azure DDoS Protection Standard par les partenaires nécessite les étapes suivantes :
1. Créez un plan de protection DDoS dans l’abonnement (partenaire) souhaité. Pour obtenir des instructions pas à pas, consultez [Créer un plan de protection DDoS Standard](manage-ddos-protection.md#create-a-ddos-protection-plan).
   > [!NOTE]
   > Un seul plan de protection DDoS doit être créé pour un locataire donné. 
2. Déployez un service avec un point de terminaison public dans vos abonnements (partenaires), par exemple équilibreur de charge, pare-feu et pare-feu d’applications web. 
3. Activez Azure DDoS Protection Standard sur le réseau virtuel du service qui a des points de terminaison publics à l’aide du plan de protection DDoS créé à la première étape. Pour obtenir des instructions pas à pas, consultez [Activer un plan de protection DDoS Standard](manage-ddos-protection.md#enable-ddos-protection-for-an-existing-virtual-network).
   > [!IMPORTANT] 
   > Une fois qu’Azure DDoS Protection Standard est activé sur un réseau virtuel, toutes les adresses IP publiques de ce réseau virtuel sont protégées automatiquement. L’origine de ces adresses IP publiques peut être dans Azure (abonnement client) ou en dehors d’Azure. 
4. Si vous le souhaitez, vous pouvez intégrer l’analytique des attaques et la télémétrie Azure DDoS Protection Standard dans votre tableau de bord client propre à l’application. Pour plus d’informations sur l’utilisation de la télémétrie, reportez-vous à [Consulter et configurer la télémétrie de la protection DDoS](telemetry-monitoring-alerting.md). 

### <a name="onboarding-guides-and-technical-documentation"></a>Guides d’intégration et documentation technique

- [Azure DDoS Protection : page produit](https://azure.microsoft.com/services/ddos-protection/)
- [Documentation sur Azure DDoS Protection](ddos-protection-overview.md)
- [Informations de référence sur l’API Azure DDoS Protection](/rest/api/virtualnetwork/ddosprotectionplans)
- [Informations de référence sur l’API de Réseau virtuel Azure](/rest/api/virtualnetwork/virtualnetworks)

### <a name="get-help"></a>Obtenir de l’aide

- Si vous avez des questions sur l’intégration des applications, des services ou des produits avec Azure DDoS Protection Standard, contactez la [communauté de sécurité Azure](https://techcommunity.microsoft.com/t5/security-identity/bd-p/Azure-Security).
- Suivez les discussions sur [Stack Overflow](https://stackoverflow.com/tags/azure-ddos/).

### <a name="get-to-market"></a>Accès au marché

- Le principal programme de partenariat avec Microsoft est le [Microsoft Partner Network](https://partner.microsoft.com/). – Les intégrations de Microsoft Graph Security sont comprises dans le suivi [MPN Independent Software Vendor (ISV)](https://partner.microsoft.com/saas-solution-guide).
- [Microsoft Intelligent Security Association](https://www.microsoft.com/security/business/intelligent-security-association?rtc=1) est un programme spécifiquement destiné aux partenaires de sécurité Microsoft afin de les aider à enrichir leurs produits de sécurité et à améliorer la découvertibilité par les clients des intégrations aux produits de sécurité Microsoft.

## <a name="next-steps"></a>Étapes suivantes
Consultez les intégrations partenaires existantes :

- [Barracuda WAF-as-a-Service](https://www.barracuda.com/waf-as-a-service)
- [Azure Cloud WAF de Radware](https://www.radware.com/resources/microsoft-azure/)