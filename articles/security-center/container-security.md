---
title: Sécurité des conteneurs dans Azure Security Center | Microsoft Docs
description: Découvrez les fonctionnalités de sécurité des conteneurs d’Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/22/2020
ms.author: memildin
ms.openlocfilehash: 7265195f9614928a2150a56a780ea7b36bc2e266
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96030864"
---
# <a name="container-security-in-security-center"></a>Sécurité des conteneurs dans Security Center

Azure Security Center est la solution Azure native pour la sécurisation de vos conteneurs.

Security Center peut protéger les types de ressources de conteneur suivants :

| Type de ressource | Protections offertes par Security Center |
|:--------------------:|-----------|
| ![Service Kubernetes](./media/security-center-virtual-machine-recommendations/icon-kubernetes-service-rec.png)<br>**Clusters Azure Kubernetes Service (AKS)** | - L’évaluation continue de vos configurations de clusters AKS pour fournir une visibilité sur les erreurs de configuration et des instructions pour vous aider à résoudre les problèmes détectés.<br>[En savoir plus sur la sécurisation renforcée de l’environnement via les recommandations de sécurité](#environment-hardening).<br><br>- Protection contre les menaces pour les clusters AKS et les nœuds Linux. Les alertes pour les activités suspectes sont fournies par [Azure Defender pour Kubernetes](defender-for-kubernetes-introduction.md), qui est facultatif.<br>[En savoir plus sur la protection au moment de l’exécution pour les nœuds et clusters AKS](#run-time-protection-for-aks-nodes-and-clusters).|
| ![Hôte de conteneur](./media/security-center-virtual-machine-recommendations/icon-container-host-rec.png)<br>**Hôtes de conteneur**<br>(Machines virtuelles exécutant Docker) | - L’évaluation continue de vos configurations Docker pour fournir une visibilité sur les erreurs de configuration et des instructions pour vous aider à résoudre les problèmes détectés avec le [Azure Defender pour les serveurs](defender-for-servers-introduction.md).<br>[En savoir plus sur la sécurisation renforcée de l’environnement via les recommandations de sécurité](#environment-hardening).|
| ![Registre de conteneurs](./media/security-center-virtual-machine-recommendations/icon-container-registry-rec.png)<br>**Registres ACR (Azure Container Registry)** | - Outils de gestion et d’évaluation des vulnérabilités pour les images de vos registres ACR basés sur Azure Resource Manager avec [Azure Defender pour les registres de conteneurs](defender-for-container-registries-introduction.md), qui est facultatif.<br>[En savoir plus sur la recherche de vulnérabilités dans vos images de conteneur](#vulnerability-management---scanning-container-images). |
|||

Cet article explique comment utiliser Security Center ainsi que les forfaits facultatifs Azure Defender pour les registres de conteneurs, les serveurs et Kubernetes pour améliorer, surveiller et maintenir la sécurité de vos conteneurs et de leurs applications.

Il explique de quelle façon Security Center améliore les principaux aspects de la sécurité des conteneurs :

- [Gestion des vulnérabilités - analyse des images conteneur](#vulnerability-management---scanning-container-images)
- [Sécurisation renforcée de l’environnement](#environment-hardening)
- [Protection au moment de l’exécution pour les nœuds et clusters AKS](#run-time-protection-for-aks-nodes-and-clusters)

La capture d’écran suivante montre la page d’inventaire des ressources et les différents types de ressources de conteneur protégés par Security Center.

:::image type="content" source="./media/container-security/container-security-tab.png" alt-text="Ressources liées au conteneur dans la page d’inventaire des ressources de Security Center" lightbox="./media/container-security/container-security-tab.png":::

## <a name="vulnerability-management---scanning-container-images"></a>Gestion des vulnérabilités : analyse des images conteneur

Pour surveiller les images dans vos registres de conteneurs Azure basés sur Azure Resource Manager, activez [Azure Defender pour les registres de conteneurs](defender-for-container-registries-introduction.md). Security Center analyse les images extraites au cours des 30 derniers jours, envoyées à votre registre ou importées. Le scanner intégré est fourni par le fournisseur d’analyse des vulnérabilités leader du secteur, Qualys.

Si des problèmes sont détectés par Qualys ou Security Center, vous recevez une notification dans le [tableau de bord Azure Defender](azure-defender-dashboard.md). Pour chaque vulnérabilité, Security Center fournit des recommandations actionnables, accompagnées d’une classification de gravité et de conseils sur la façon de corriger le problème. Pour plus d’informations concernant les recommandations de Security Center pour les conteneurs, consultez la [liste de référence des recommandations](recommendations-reference.md#recs-containers).

Security Center filtre et classifie les résultats à partir de l’analyseur. Quand une image est saine, Security Center la marque comme telle. Security Center génère des recommandations de sécurité uniquement pour les images qui ont des problèmes à résoudre. En n’avertissant qu’en cas de problème, Security Center réduit le risque d’alertes d’information indésirables.

## <a name="environment-hardening"></a>Sécurisation renforcée de l’environnement

### <a name="continuous-monitoring-of-your-docker-configuration"></a>Supervision continue de votre configuration Docker

Azure Security Center identifie les conteneurs non managés qui sont hébergés sur des machines virtuelles IaaS Linux, ou d’autres machines Linux exécutant des conteneurs Docker. Security Center évalue en continu les configurations de ces conteneurs. Il les compare ensuite au document de référence [Center for Internet Security (CIS) Docker Benchmark](https://www.cisecurity.org/benchmark/docker/).

Security Center inclut la totalité des règles définies dans le CIS Docker Benchmark et vous envoie une alerte si vos conteneurs ne satisfont pas à tous les contrôles. Quand il détecte des configurations incorrectes, Security Center génère des recommandations de sécurité. Accédez à la **page des recommandations** de Security Center pour lire les suggestions et corriger les problèmes. Ces vérifications des règles du CIS Benchmark ne sont pas effectuées sur les instances managées par AKS ni sur les machines virtuelles managées par Databricks.

Pour plus d’informations sur les recommandations pertinentes de Security Center qui peuvent apparaître pour cette fonctionnalité, consultez la [section relative aux conteneurs](recommendations-reference.md#recs-containers) de la table de référence des recommandations.

Quand vous examinez les problèmes de sécurité d’une machine virtuelle, Security Center fournit des informations supplémentaires à propos des conteneurs sur la machine. Il indique notamment la version de Docker et le nombre d’images exécutées sur la machine hôte. 

Pour surveiller les conteneurs non gérés hébergés sur des machines virtuelles IaaS Linux, activez l’option facultative [Azure Defender pour les serveurs](defender-for-servers-introduction.md).


### <a name="continuous-monitoring-of-your-kubernetes-clusters"></a>Supervision continue de vos clusters Kubernetes
Security Center s’utilise conjointement avec Azure Kubernetes Service (AKS), le service d’orchestration de conteneurs managés de Microsoft conçu pour le développement, le déploiement et la gestion d’applications conteneurisées.

AKS fournit des contrôles de sécurité et apporte une visibilité sur la posture de sécurité de vos clusters. Security Center utilise ces fonctionnalités pour :
* Superviser en continu la configuration de vos clusters AKS
* Générer des recommandations de sécurité en adéquation avec les normes du secteur

Pour plus d’informations sur les recommandations pertinentes de Security Center qui peuvent apparaître pour cette fonctionnalité, consultez la [section relative aux conteneurs](recommendations-reference.md#recs-containers) de la table de référence des recommandations.

###  <a name="workload-protection-best-practices-using-kubernetes-admission-control"></a>Meilleures pratiques pour la protection de charge de travail à l’aide du contrôle d’admission Kubernetes

Pour obtenir un ensemble de recommandations pour la protection des charges de travail de vos conteneurs Kubernetes, installez le **module complémentaire Azure Policy pour Kubernetes**. Vous pouvez également déployer automatiquement ce module complémentaire comme cela est expliqué dans [Activer le provisionnement automatique des extensions](security-center-enable-data-collection.md#enable-auto-provisioning-of-extensions). Lorsque le provisionnement automatique du module complémentaire est défini sur « activé », l’extension est activée par défaut dans tous les clusters existants et futurs (qui répondent à la configuration requise de l’installation du module complémentaire).

Comme expliqué dans [cette page Azure Policy pour page Kubernetes](../governance/policy/concepts/policy-for-kubernetes.md), le module complémentaire étend le webhook de contrôleur d’admission open source [Gatekeeper v3](https://github.com/open-policy-agent/gatekeeper)  pour  [Open Policy Agent](https://www.openpolicyagent.org/). Les contrôleurs d’admission Kubernetes sont des modules complémentaires qui appliquent la manière dont vos clusters sont utilisés. Le module complémentaire s’inscrit en tant que webhook pour le contrôle d’admission Kubernetes et permet d’appliquer une mise en œuvre et une protection à grande échelle sur vos clusters, de manière centralisée et cohérente. 

Avec le module complémentaire sur votre cluster AKS, chaque demande adressée au serveur d’API Kubernetes est analysée par rapport à l’ensemble prédéfini de bonnes pratiques, avant d’être conservée sur le cluster. Vous pouvez ensuite configurer l’**application** des meilleures pratiques et les imposer aux charges de travail futures. 

Par exemple, vous pouvez faire en sorte que des conteneurs privilégiés ne soient pas créés, et que toutes les demandes ultérieures soient bloquées.

Pour plus d’informations, consultez [Protéger vos charges de travail Kubernetes](kubernetes-workload-protections.md).


## <a name="run-time-protection-for-aks-nodes-and-clusters"></a>Protection au moment de l’exécution pour les nœuds et clusters AKS

[!INCLUDE [AKS in ASC threat protection](../../includes/security-center-azure-kubernetes-threat-protection.md)]



## <a name="next-steps"></a>Étapes suivantes

Dans cette vue d’ensemble, vous avez découvert des informations sur les principaux éléments de la sécurité des conteneurs dans Azure Security Center. Pour des documents connexes, consultez :

- [Présentation d’Azure Defender pour Kubernetes](defender-for-kubernetes-introduction.md)
- [Introduction à Azure Defender pour des registres de conteneurs](defender-for-container-registries-introduction.md)