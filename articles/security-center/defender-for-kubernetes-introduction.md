---
title: 'Azure Defender pour Kubernetes : avantages et fonctionnalités'
description: Découvrez les avantages et les fonctionnalités d’Azure Defender pour Kubernetes.
author: memildin
ms.author: memildin
ms.date: 9/12/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 04427a1eb2760e44086003a06ed4eb0d5cb473bf
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673322"
---
# <a name="introduction-to-azure-defender-for-kubernetes"></a>Présentation d’Azure Defender pour Kubernetes

Azure Kubernetes Service (AKS) est le service managé de Microsoft conçu pour le développement, le déploiement et la gestion d’applications conteneurisées.

Azure Security Center et AKS constituent ensemble la meilleure offre de sécurité Kubernetes native Cloud, qui assure une sécurisation renforcée de l’environnement, une protection de charge de travail et une protection à l’exécution (cf. ci-dessous).

Pour la détection des menaces sur vos clusters Kubernetes, activez **Azure Defender pour Kubernetes**.

La détection des menaces au niveau de l’hôte sur les nœuds AKS Linux est disponible à condition d’activer [Azure Defender pour les serveurs](defender-for-servers-introduction.md).

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|**Azure Defender pour Kubernetes** est facturé conformément aux indications de la [page des tarifs](security-center-pricing.md).|
|Rôles et autorisations obligatoires :|L’**administrateur de sécurité** peut ignorer les alertes.<br>Le **Lecteur de sécurité** peut afficher les résultats.|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Oui](./media/icons/yes-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-kubernetes"></a>Quels sont les avantages d’Azure Defender pour Kubernetes ?

### <a name="run-time-protection"></a>Protection à l’exécution

Par l’analyse continue des sources AKS suivantes, Security Center assure une protection en temps réel contre les menaces des environnements en conteneurs et génère des alertes en cas de menace ou d’activité malveillante détectée au niveau de l’hôte *et* au niveau du cluster AKS. Vous pouvez utiliser ces informations pour remédier rapidement aux problèmes de sécurité et améliorer la sécurité de vos conteneurs.

Security Center offre une protection contre les menaces à différents niveaux : 

- **Niveau de l’hôte (fourni par Azure Defender pour les serveurs)** : à l’aide du même agent Log Analytics que celui utilisé par Security Center sur d’autres machines virtuelles, Azure Defender effectue un monitoring des nœuds AKS Linux pour repérer les activités suspectes, par exemple, la détection de web shell et la connexion avec des adresses IP suspectes connues. L’agent surveille également les analyses propres au conteneur, comme la création de conteneurs privilégiés, l’accès suspect à des serveurs d’API et à des serveurs Secure Shell (SSH) s’exécutant dans un conteneur Docker.

    Pour obtenir la liste des alertes au niveau de l’hôte AKS, consultez la [Table de référence des alertes](alerts-reference.md#alerts-containerhost).

    >[!IMPORTANT]
    > Si vous choisissez de ne pas installer les agents sur vos machines hôtes, vous ne recevrez qu’une partie des avantages et alertes de sécurité relatifs à la protection contre les menaces. Vous recevrez toujours les alertes liées à l’analyse réseau et aux communications avec des serveurs malveillants.

- **Niveau de cluster AKS (fourni par Azure Defender pour Kubernetes)** : au niveau du cluster, la protection contre les menaces est basée sur l’analyse des journaux d’audit Kubernetes. Pour activer cette analyse **sans agent**, activez Azure Defender. Pour générer des alertes à ce niveau, Security Center supervise vos services managés par AKS à l’aide des journaux collectés par AKS. Les tableaux de bord Kubernetes exposés, la création de rôles dotés de privilèges élevés et la création de montages sensibles sont des exemples d’événements à ce niveau.

    Pour obtenir la liste des alertes au niveau du cluster AKS, consultez la [table de référence des alertes](alerts-reference.md#alerts-akscluster).

    >[!NOTE]
    > Security Center génère des alertes de sécurité pour les actions et les déploiements Azure Kubernetes Service, qui se produisent après l’activation de l’option Kubernetes dans les paramètres de l’abonnement. 

De plus, notre équipe mondiale d’experts en sécurité surveille en permanence l’évolution des menaces. Ils ajoutent des alertes et des vulnérabilités propres aux conteneurs au fur et à mesure de leur découverte.

> [!TIP]
> Vous pouvez simuler des alertes de conteneur en suivant les instructions données dans [ce billet de blog](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-demonstrate-the-new-containers-features-in-azure-security/ba-p/1011270).



## <a name="how-does-security-centers-kubernetes-protection-work"></a>Comment fonctionne la protection Kubernetes de Security Center ?

Vous trouverez ci-dessous un diagramme général de l’interaction entre Azure Security Center, Azure Kubernetes Service et Azure Policy.

Comme vous pouvez le constater, les éléments reçus et analysés par Security Center sont les suivants :

- Journaux d’audit provenant du serveur d’API
- Événements de sécurité bruts provenant de l’agent Log Analytics
- Informations de configuration du cluster provenant du cluster AKS
- Configuration des charges de travail provenant d’Azure Policy (par le biais du module complémentaire **Azure Policy pour Kubernetes**). [En savoir plus sur les meilleures pratiques de protection de charge de travail à l’aide du contrôle d’admission Kubernetes](container-security.md#workload-protection-best-practices-using-kubernetes-admission-control)

:::image type="content" source="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png" alt-text="Architecture générale de l’interaction entre Azure Security Center, Azure Kubernetes Service et Azure Policy" lightbox="./media/defender-for-kubernetes-intro/kubernetes-service-security-center-integration-detailed.png":::




## <a name="azure-defender-for-kubernetes---faq"></a>Azure Defender pour Kubernetes – FAQ

### <a name="can-i-still-get-aks-protections-without-the-log-analytics-agent"></a>Puis-je quand même bénéficier des protections AKS sans l’agent Log Analytics ?

Comme indiqué ci-dessus, le plan **Azure Defender pour Kubernetes** facultatif fournit des protections au niveau du cluster, tandis que l’agent Log Analytics **d’Azure Defender pour les serveurs** protège les nœuds. 

Nous vous recommandons de déployer les deux pour bénéficier de la protection la plus complète possible.

Si vous choisissez de ne pas installer l’agent sur vos hôtes, vous ne bénéficiez que d’une partie de la protection contre les menaces et des alertes de sécurité. Vous recevrez toujours les alertes liées à l’analyse réseau et aux communications avec des serveurs malveillants.


### <a name="does-aks-allow-me-to-install-custom-vm-extensions-on-my-aks-nodes"></a>Est-ce qu’AKS m’autorise à installer des extensions de machine virtuelle personnalisées sur mes nœuds AKS ?

Pour qu’Azure Defender supervise vos nœuds AKS, ceux-ci doivent exécuter l’agent Log Analytics. 

AKS est un service managé et, l’agent Log Analytics étant une extension managée par Microsoft, il est également pris en charge sur les clusters AKS.



### <a name="if-my-cluster-is-already-running-an-azure-monitor-for-containers-agent-do-i-need-the-log-analytics-agent-too"></a>Si mon cluster exécute déjà un agent Azure Monitor pour conteneurs, ai-je besoin de l’agent Log Analytics ?

Pour qu’Azure Defender supervise vos nœuds AKS, ceux-ci doivent exécuter l’agent Log Analytics.

Si vos clusters exécutent déjà l’agent Azure Monitor pour conteneurs, vous pouvez également installer l’agent Log Analytics, et les deux agents peuvent fonctionner conjointement sans aucun problème.

[En savoir plus sur l’agent Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-manage-agent.md).


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert la protection Kubernetes de Security Center, notamment Azure Defender pour Kubernetes. 

Pour des informations connexes, consultez les articles suivants : 

- [Activation d’Azure Defender](security-center-pricing.md)
- [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](export-to-siem.md)
- [Table de référence des alertes](alerts-reference.md)