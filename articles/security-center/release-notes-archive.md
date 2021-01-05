---
title: Archive des nouveautés d’Azure Security Center
description: Description des nouveautés et modifications apportées à Azure Security Center jusqu’à il y a six mois.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/30/2020
ms.author: memildin
ms.openlocfilehash: 854926c64b50cf4b8e7df9fa82da58b924ddbd83
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510418"
---
# <a name="archive-for-whats-new-in-azure-security-center"></a>Archive des nouveautés d’Azure Security Center

La page principale des notes de publication [Nouveautés d’Azure Security Center](release-notes.md) a trait aux mises à jour des six derniers mois, tandis que celle-ci a trait à des mises à jour plus anciennes.

Cette page fournit des informations sur les points suivants :

- Nouvelles fonctionnalités
- Résolution des bogues
- Fonctionnalités dépréciées


## <a name="june-2020"></a>Juin 2020

Les mises à jour du mois de juin incluent :
- [API Degré de sécurisation (préversion)](#secure-score-api-preview)
- [Sécurité avancée des données pour les machines SQL (Azure, autres clouds et en local) (préversion)](#advanced-data-security-for-sql-machines-azure-other-clouds-and-on-premises-preview)
- [Deux nouvelles recommandations pour déployer l’agent Log Analytics sur des machines Azure Arc (préversion)](#two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview)
- [Nouvelles stratégies pour créer des configurations d’exportation continue et d’automatisation de flux de travail à l’échelle](#new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale)
- [Nouvelle recommandation pour l’utilisation de NSG afin de protéger les machines virtuelles non accessibles sur Internet](#new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines)
- [Nouvelles stratégies pour activer la protection contre les menaces et la sécurité avancée des données](#new-policies-for-enabling-threat-protection-and-advanced-data-security)



### <a name="secure-score-api-preview"></a>API Degré de sécurisation (préversion)

Vous pouvez maintenant accéder à votre degré de sécurisation par le biais de [l’API Degré de sécurisation](/rest/api/securitycenter/securescores/) (actuellement en préversion). Les méthodes de l’API offrent la flexibilité nécessaire pour interroger les données et créer votre propre mécanisme de création de rapports sur vos degrés de sécurisation au fil du temps. Par exemple, vous pouvez utiliser l’API **Degré de sécurisation** pour obtenir le degré de sécurisation d’un abonnement spécifique. En outre, vous pouvez utiliser l’API **Contrôles du degré de sécurisation** pour répertorier les contrôles de sécurité et le degré de sécurisation actuel de vos abonnements.

Pour obtenir des exemples d’outils externes accessibles avec l’API Degré de sécurisation, consultez [la zone consacrée au degré de sécurisation de notre communauté GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score).

En savoir plus sur [le degré de sécurisation et les contrôles de sécurité dans Azure Security Center](secure-score-security-controls.md).



### <a name="advanced-data-security-for-sql-machines-azure-other-clouds-and-on-premises-preview"></a>Sécurité avancée des données pour les machines SQL (Azure, autres clouds et en local) (préversion)

La sécurité des données avancée d’Azure Security Center pour les machines SQL protège désormais les serveurs SQL hébergés dans Azure, dans d’autres environnements cloud et même sur des machines locales. Cela étend les protections de vos serveurs SQL Azure natifs pour prendre entièrement en charge les environnements hybrides.

La sécurité avancée des données fournit une évaluation des vulnérabilités et une protection avancée contre les menaces pour vos machines SQL où qu’elles soient.

La configuration se fait en deux étapes :

1. Déploiement de l’agent Log Analytics sur l’ordinateur hôte de votre serveur SQL Server pour fournir la connexion au compte Azure.

1. Activation du bundle facultatif dans la page de tarification et des paramètres de Security Center.

En savoir plus sur la [sécurité avancée des données pour les machines SQL](defender-for-sql-usage.md).



### <a name="two-new-recommendations-to-deploy-the-log-analytics-agent-to-azure-arc-machines-preview"></a>Deux nouvelles recommandations pour déployer l’agent Log Analytics sur des machines Azure Arc (préversion)

Deux nouvelles recommandations ont été ajoutées pour vous aider à déployer [l’agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) sur vos machines Azure Arc et vous assurer qu’elles sont protégées par Azure Security Center :

- **L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Windows (préversion)**
- **L’agent Log Analytics doit être installé sur vos machines Azure Arc basées sur Linux (préversion)**

Ces nouvelles recommandations s’affichent dans les quatre mêmes contrôles de sécurité que la recommandation existante (associée), **L’agent d’analyse doit être installé sur vos machines** : corriger les configurations de sécurité, appliquer le contrôle d’application adaptatif, appliquer les mises à jour système et activer la protection de point de terminaison.

Les recommandations incluent également la fonctionnalité de correction rapide pour accélérer le processus de déploiement. 

Pour plus d’informations sur ces deux nouvelles recommandations, consultez le tableau [Recommandations relatives au calcul et aux applications](recommendations-reference.md#recs-computeapp).

En savoir plus sur la façon dont Azure Security Center utilise l’agent dans [Qu’est-ce que l’agent Log Analytics ?](faq-data-collection-agents.md#what-is-the-log-analytics-agent).

En savoir plus sur les [extensions pour les machines Azure Arc](../azure-arc/servers/manage-vm-extensions.md).


### <a name="new-policies-to-create-continuous-export-and-workflow-automation-configurations-at-scale"></a>Nouvelles stratégies pour créer des configurations d’exportation continue et d’automatisation de flux de travail à l’échelle

L’automatisation des processus d’analyse et de réponse aux incidents de votre organisation peut réduire de manière significative le temps requis pour examiner et atténuer les incidents de sécurité.

Pour déployer vos configurations d’automatisation à l’échelle de votre organisation, utilisez ces stratégies Azure « DeployIfdNotExist » intégrées pour créer et configurer les procédures [d’exportation continue](continuous-export.md) et [d’automatisation de flux de travail](workflow-automation.md) :

Les stratégies se trouvent dans Azure Policy :


|Objectif  |Policy  |ID de stratégie  |
|---------|---------|---------|
|Exportation continue vers Event Hub|[Déployer l’exportation vers Event Hub pour les alertes et les recommandations Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fcdfcce10-4578-4ecd-9703-530938e4abcb)|cdfcce10-4578-4ecd-9703-530938e4abcb|
|Exportation continue vers l’espace de travail Log Analytics|[Déployer l’exportation vers un espace de travail Log Analytics pour les alertes et les recommandations Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fffb6f416-7bd2-4488-8828-56585fef2be9)|ffb6f416-7bd2-4488-8828-56585fef2be9|
|Automatisation du flux de travail pour les alertes de sécurité|[Déployer l’automatisation de workflow pour les alertes Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff1525828-9a90-4fcf-be48-268cdd02361e)|f1525828-9a90-4fcf-be48-268cdd02361e|
|Automatisation du flux de travail pour les recommandations de sécurité|[Déployer l’automatisation de workflow pour les recommandations Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-2475-4850-afd6-43795f3492ef)|73d6ab6c-2475-4850-afd6-43795f3492ef|
||||

Prise en main des [modèles d’automatisation de flux de travail](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation).

Apprenez-en davantage sur l’utilisation des deux stratégies d’exportation dans [Configurer l’automatisation du workflow à grande échelle à l’aide des stratégies fournies](workflow-automation.md#configure-workflow-automation-at-scale-using-the-supplied-policies) et [Configurer une exportation continue](continuous-export.md#set-up-a-continuous-export).


### <a name="new-recommendation-for-using-nsgs-to-protect-non-internet-facing-virtual-machines"></a>Nouvelle recommandation pour l’utilisation de NSG afin de protéger les machines virtuelles non accessibles sur Internet

Le contrôle de sécurité « implémenter les meilleures pratiques de sécurité » comprend désormais la nouvelle recommandation suivante :

- **Les machines virtuelles non connectées à Internet doivent être protégées avec des groupes de sécurité réseau**

Une recommandation existante, **Les machines virtuelles accessibles à partir d’Internet doivent être protégées avec des groupes de sécurité réseau**, ne fait pas la distinction entre les machines virtuelles accessibles sur Internet et celles qui ne le sont pas. Pour les deux types de machines virtuelles, une recommandation à gravité élevée était générée si une machine virtuelle n’était pas affectée à un groupe de sécurité réseau. Cette nouvelle recommandation sépare les machines non accessibles à partir d’Internet pour réduire les faux positifs et éviter les alertes à gravité élevée inutiles.

En savoir plus dans le tableau [Recommandations pour le réseau](recommendations-reference.md#recs-network).




### <a name="new-policies-for-enabling-threat-protection-and-advanced-data-security"></a>Nouvelles stratégies pour activer la protection contre les menaces et la sécurité avancée des données

Les nouvelles stratégies ci-dessous ont été ajoutées à l’initiative ASC par défaut et sont conçues pour aider à activer la protection contre les menaces ou la sécurité avancée des données pour les types de ressources appropriés.

Les stratégies se trouvent dans Azure Policy :


| Policy                                                                                                                                                                                                                                                                | ID de stratégie                            |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| [Advanced Data Security doit être activé sur les serveurs Azure SQL Database](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f7fe3b40f-802b-4cdd-8bd4-fd799c948cc2)     | 7fe3b40f-802b-4cdd-8bd4-fd799c948cc2 |
| [Advanced Data Security doit être activé sur les serveurs SQL sur les machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f6581d072-105e-4418-827f-bd446d56421b) | 6581d072-105e-4418-827f-bd446d56421b |
| [Advanced Threat Protection doit être activé sur les comptes Stockage Azure](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f308fbb08-4ab8-4e67-9b29-592e93fb94fa)           | 308fbb08-4ab8-4e67-9b29-592e93fb94fa |
| [Advanced Threat Protection doit être activé sur les coffres Azure Key Vault](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f0e6763cc-5078-4e64-889d-ff4d9a839047)           | 0e6763cc-5078-4e64-889d-ff4d9a839047 |
| [Advanced Threat Protection doit être activé sur les plans Azure App Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f2913021d-f2fd-4f3d-b958-22354e2bdbcb)                | 2913021d-f2fd-4f3d-b958-22354e2bdbcb |
| [Advanced Threat Protection doit être activé sur les registres Azure Container Registry](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2fc25d9a16-bc35-4e15-a7e5-9db606bf9ed4)   | c25d9a16-bc35-4e15-a7e5-9db606bf9ed4 |
| [Advanced Threat Protection doit être activé sur les clusters Azure Kubernetes Service](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f523b5cd1-3e23-492f-a539-13118b6d1e3a)   | 523b5cd1-3e23-492f-a539-13118b6d1e3a |
| [Advanced Threat Protection doit être activé sur les machines virtuelles](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f4da35fc9-c9e7-4960-aec9-797fe7d9051d)           | 4da35fc9-c9e7-4960-aec9-797fe7d9051d |
|                                                                                                                                                                                                                                                                       |                                      |

En savoir plus sur la [protection contre les menaces dans Azure Security Center](azure-defender.md).


## <a name="may-2020"></a>Mai 2020

Les mises à jour du mois de mai incluent :
- [Règles de suppression d’alerte (préversion)](#alert-suppression-rules-preview)
- [Disponibilité de l’évaluation des vulnérabilités des machines virtuelles](#virtual-machine-vulnerability-assessment-is-now-generally-available)
- [Modifications apportées à l’accès juste-à-temps (JAT) des machines virtuelles](#changes-to-just-in-time-jit-virtual-machine-vm-access)
- [Déplacement des recommandations personnalisées vers un contrôle de sécurité distinct](#custom-recommendations-have-been-moved-to-a-separate-security-control)
- [Ajout d’une option permettant d’afficher les recommandations dans les contrôles ou sous forme de liste plate](#toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list)
- [Extension du contrôle de sécurité « Implémenter les bonnes pratiques de sécurité »](#expanded-security-control-implement-security-best-practices)
- [Disponibilité générale des stratégies personnalisées avec des métadonnées personnalisées](#custom-policies-with-custom-metadata-are-now-generally-available)
- [Migration des fonctionnalités d’analyse du vidage sur incident vers la détection d’attaque sans fichier](#crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection)


### <a name="alert-suppression-rules-preview"></a>Règles de suppression d’alerte (préversion)

Cette nouvelle fonctionnalité (actuellement en préversion) permet de réduire la fréquence des alertes. Utilisez des règles pour masquer automatiquement les alertes connues pour être anodines ou liées à des activités normales au sein votre organisation. Cela vous permet de vous concentrer sur les menaces les plus pertinentes. 

Les alertes correspondant à vos règles de suppression activées sont toujours générées, mais leur état est défini sur ignoré. Vous pouvez voir leur état sur le portail Azure ou en accédant aux alertes de votre Security Center.

Les règles de suppression définissent les critères en vertu desquels les alertes doivent être automatiquement ignorées. En règle générale, vous utilisez une règle de suppression pour effectuer les opérations suivantes :

- supprimer des alertes identifiées comme faux positifs ;

- supprimer des alertes déclenchées trop souvent pour être utiles.

Apprenez-en davantage sur la [suppression des alertes à partir de la protection contre les menaces d’Azure Security Center](alerts-suppression-rules.md).


### <a name="virtual-machine-vulnerability-assessment-is-now-generally-available"></a>Disponibilité de l’évaluation des vulnérabilités des machines virtuelles

Le niveau standard de Security Center intègre désormais l’évaluation des vulnérabilités des machines virtuelles sans frais supplémentaires. Cette extension est fournie par Qualys mais renvoie ses résultats directement à Security Center. Vous n’avez pas besoin d’une licence Qualys ni même de compte Qualys : tout est traité de manière fluide dans Security Center.

La nouvelle solution peut analyser en continu vos machines virtuelles pour trouver des vulnérabilités et présenter les résultats au Security Center. 

Pour déployer la solution, suivez la nouvelle recommandation de sécurité :

« Activer la solution intégrée d’évaluation des vulnérabilités sur les machines virtuelles (optimisées par Qualys) (Préversion)

Apprenez-en davantage sur l’[évaluation des vulnérabilités des machines virtuelles intégrée dans le Security Center](deploy-vulnerability-assessment-vm.md#overview-of-the-integrated-vulnerability-scanner).



### <a name="changes-to-just-in-time-jit-virtual-machine-vm-access"></a>Modifications apportées à l’accès juste-à-temps (JAT) des machines virtuelles

Le Security Center intègre une fonctionnalité facultative destinée à protéger les ports de gestion de vos machines virtuelles. Celle-ci offre une protection contre la forme la plus courante d’attaques par force brute.

Cette mise à jour apporte à cette fonctionnalité les modifications suivantes :

- La recommandation suggérant d’activer l’accès JAT sur une machine virtuelle a été reformulée. L’ancien libellé, « Un contrôle d’accès réseau juste-à-temps doit être appliqué aux machines virtuelles », est remplacé par « Les ports de gestion des machines virtuelles doivent être protégés par un contrôle d’accès réseau juste-à-temps ».

- La recommandation n’est déclenchée que s’il existe des ports de gestion ouverts.

Apprenez-en davantage sur la [fonctionnalité accès JAT](security-center-just-in-time.md).


### <a name="custom-recommendations-have-been-moved-to-a-separate-security-control"></a>Déplacement des recommandations personnalisées vers un contrôle de sécurité distinct

L’un des contrôles de sécurité introduits avec le degré de sécurisation amélioré était « Implémenter les meilleures pratiques de sécurité ». Toutes les recommandations personnalisées créées pour vos abonnements ont été placées automatiquement dans ce contrôle. 

Pour faciliter la recherche de vos recommandations personnalisées, nous les avons déplacées vers un contrôle de sécurité dédié nommé « Recommandations personnalisées ». Ce contrôle n’a aucun impact sur votre degré de sécurisation.

Pour en savoir plus sur les contrôles de sécurité, consultez [Version améliorée du degré de sécurisation (préversion) dans Azure Security Center](secure-score-security-controls.md).


### <a name="toggle-added-to-view-recommendations-in-controls-or-as-a-flat-list"></a>Ajout d’une option permettant d’afficher les recommandations dans les contrôles ou sous forme de liste plate

Les contrôles de sécurité sont des groupes logiques de recommandations de sécurité associées. Ceux-ci reflètent vos surfaces d’attaque vulnérables. Un contrôle est un ensemble de recommandations de sécurité, avec des instructions qui vous permettent de les implémenter.

Pour voir immédiatement dans quelle mesure votre organisation sécurise chacune des surfaces d’attaque, examinez le degré de chaque contrôle de sécurité.

Par défaut, vos recommandations s’affichent dans les contrôles de sécurité. À partir de cette mise à jour, vous pouvez également les afficher sous forme de liste. Pour les afficher sous la forme d’une liste simple triée en fonction de l’état d’intégrité des ressources affectées, utilisez la nouvelle option « Grouper par contrôles ». Cette option se trouve au-dessus de la liste dans le portail.

Les contrôles de sécurité, et cette option, font partie de la nouvelle expérience de degré de sécurisation. N’oubliez pas de nous envoyer vos commentaires à partir du portail.

Pour en savoir plus sur les contrôles de sécurité, consultez [Version améliorée du degré de sécurisation (préversion) dans Azure Security Center](secure-score-security-controls.md).

:::image type="content" source="./media/secure-score-security-controls/recommendations-group-by-toggle.gif" alt-text="Activer/désactiver « Regrouper par contrôles » pour les recommandations":::

### <a name="expanded-security-control-implement-security-best-practices"></a>Extension du contrôle de sécurité « Implémenter les bonnes pratiques de sécurité » 

L’un des contrôles de sécurité introduits avec le degré de sécurisation amélioré est « Implémenter les meilleures pratiques de sécurité ». Quand ce contrôle contient une recommandation, celle-ci n’a aucun impact sur le degré de sécurisation. 

Avec cette mise à jour, trois recommandations ont été déplacées des contrôles dans lesquels elles étaient placées à l’origine vers ce contrôle des bonnes pratiques. Nous avons pris cette mesure parce que nous avons constaté que le risque que ces trois recommandations visaient à prévenir était moindre que le risque initialement prévu.

En outre, deux nouvelles recommandations ont été introduites et ajoutées à ce contrôle.

Les trois recommandations déplacées sont les suivantes :

- **L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement** (à l’origine, dans le contrôle « Activer la MFA »).
- **Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement** (à l’origine, dans le contrôle « Gérer l’accès et les autorisations »).
- **Trois propriétaires au plus doivent être désignés pour votre abonnement** (à l’origine, dans le contrôle « Gérer l’accès et les autorisations »).

Les deux nouvelles recommandations ajoutées au contrôle sont les suivantes :

- **L’extension de configuration d’invité doit être installée sur les machines virtuelles Windows (préversion)**  : la [Configuration d’invité Azure Policy](../governance/policy/concepts/guest-configuration.md) apporte une visibilité des machines virtuelles aux paramètres de serveur et d’application (Windows uniquement).

- **Windows Defender Exploit Guard doit être activé sur vos machines (préversion)**  : Windows Defender Exploit Guard tire parti de l’agent de configuration d’invité (Guest Configuration) Azure Policy. Windows Defender Exploit Guard compte quatre composants conçus pour verrouiller les appareils afin de les protéger contre un vaste éventail de vecteurs d’attaque et comportements de blocage couramment utilisés par les programmes malveillants, tout en permettant aux entreprises de trouver un juste équilibre entre sécurité et productivité (Windows uniquement).

Pour en savoir plus sur Windows Defender exploit Guard, consultez [Créer et déployer une stratégie Windows Defender Exploit Guard](/mem/configmgr/protect/deploy-use/create-deploy-exploit-guard-policy).

Pour en savoir plus sur les contrôles de sécurité, consultez [Version améliorée du degré de sécurisation (préversion)](secure-score-security-controls.md).



### <a name="custom-policies-with-custom-metadata-are-now-generally-available"></a>Disponibilité générale des stratégies personnalisées avec des métadonnées personnalisées

Les stratégies personnalisées font désormais partie de l’expérience des Recommandations relatives à Azure Security Center, du degré de sécurisation et du tableau de bord des normes de conformité réglementaire. Cette fonctionnalité désormais généralement disponible vous permet d’étendre la couverture de l’évaluation de la sécurité de votre organisation dans Azure Security Center. 

Créez une initiative personnalisée dans Azure Policy, ajoutez-y des stratégies, intégrez-la à Azure Security Center et visualisez-la sous la forme de recommandations.

Nous avons également ajouté une option permettant de modifier les métadonnées de recommandation personnalisées. Les options des métadonnées incluent la gravité, des mesures de correction, des informations sur les menaces, etc.  

Pour en savoir plus, consultez [Amélioration de vos recommandations personnalisées avec des informations détaillées](custom-security-policies.md#enhance-your-custom-recommendations-with-detailed-information).



### <a name="crash-dump-analysis-capabilities-migrating-to-fileless-attack-detection"></a>Migration des fonctionnalités d’analyse du vidage sur incident vers la détection d’attaque sans fichier 

Nous intégrons les fonctionnalités de détection de l’analyse du vidage sur incident Windows dans la [détection d’attaque sans fichier](defender-for-servers-introduction.md#what-are-the-benefits-of-azure-defender-for-servers). L’analyse de la détection d’attaque sans fichier offre des versions améliorées des alertes de sécurité suivantes pour les ordinateurs Windows : Injection de code découverte, usurpation d’identité de module Windows détectée, Shellcode détecté et segment de code suspect détecté.

Voici quelques-uns des avantages de cette transition :

- **Détection proactive et en temps opportun des programmes malveillants** : l’approche de l’analyse du vidage sur incident impliquait l’attente de la survenance d’un incident, puis l’exécution d’une analyse pour trouver des artefacts malveillants. La détection d’attaque sans fichier introduit l’identification de manière proactive des menaces en mémoire pendant leur exécution. 

- **Alertes enrichies** : les alertes de sécurité liées à la détection d’attaque sans fichier apportent des enrichissements par rapport à une simple analyse du vidage sur incident, tels que les informations de connexions réseau actives. 

- **Agrégation d’alertes** : quand l’analyse du vidage sur incident détectait plusieurs modèles d’attaque au sein d’un même vidage sur incident, elle déclenchait plusieurs alertes de sécurité. La détection d’attaque sans fichier combine tous les modèles d’attaque identifiés participant d’un même processus dans une alerte unique, ce qui évite d’avoir à mettre en corrélation plusieurs alertes.

- **Exigences réduites concernant votre espace de travail Log Analytics** : les vidages sur incident contenant des données potentiellement sensibles ne sont plus chargés dans votre espace de travail Log Analytics.






## <a name="april-2020"></a>Avril 2020

Les mises à jour du mois d’avril incluent :
- [Disponibilité générale des packages de conformité dynamique](#dynamic-compliance-packages-are-now-generally-available)
- [Inclusion des recommandations relatives aux identités dans le niveau gratuit d’Azure Security Center](#identity-recommendations-now-included-in-azure-security-center-free-tier)


### <a name="dynamic-compliance-packages-are-now-generally-available"></a>Disponibilité générale des packages de conformité dynamique

Le tableau de bord de conformité réglementaire d’Azure Security Center inclut désormais des **packages de conformité dynamique** (en disponibilité générale) pour suivre des normes réglementaires et sectorielles supplémentaires.

Vous pouvez ajouter ces packages de conformité dynamique à votre abonnement ou à votre groupe d’administration à partir de la page Stratégie de sécurité d’Azure Security Center. Après l’intégration d’une norme ou un benchmark, ceux-ci apparaissent dans votre tableau de bord de conformité réglementaire avec toutes les données de conformité associées en tant qu’évaluations. Un rapport de synthèse pour toutes les normes intégrées sera disponible en téléchargement.

Désormais, vous pouvez ajouter des normes telles que les suivantes :

- **NIST SP 800-53 R4**
- **SWIFT CSP CSCF-v2020**
- **UK Official et UK NHS**
- **PBMM fédéral du Canada**
- **Azure CIS 1.1.0 (nouveau)** (représentation plus complète d’Azure CIS 1.1.0)

De plus, nous avons récemment ajouté le **Benchmark de sécurité Azure**, les directives spécifiques d’Azure créées par Microsoft pour les meilleures pratiques de sécurité et de conformité, basées sur des infrastructures de conformité courantes. Des normes supplémentaires seront prises en charge dans le tableau de bord dès qu’elles seront disponibles.  
 
Apprenez-en davantage sur la [personnalisation de l’ensemble de normes de votre tableau de bord de conformité réglementaire](update-regulatory-compliance-packages.md).


### <a name="identity-recommendations-now-included-in-azure-security-center-free-tier"></a>Inclusion des recommandations relatives aux identités dans le niveau gratuit d’Azure Security Center

Les recommandations de sécurité relatives aux identités et aux accès sont désormais généralement disponibles dans le niveau gratuit d’Azure Security Center. Cela fait partie de l’effort visant à atteindre la gratuité des fonctionnalités de gestion de la posture de sécurité cloud. Jusqu’à présent, ces recommandations n’étaient disponibles qu’au niveau tarifaire standard.

Voici des exemples de recommandations relatives aux identités et aux accès :

- « L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement. »
- « Au maximum trois propriétaires doivent être désignés pour votre abonnement. »
- « Les comptes déconseillés doivent être supprimés de votre abonnement. »

Si vous avez des abonnements au niveau tarifaire gratuit, ce changement affectera leurs degrés de sécurisation, car ils n’ont jamais été évalués sur le plan de la sécurité des identités et des accès.

Apprenez-en davantage sur les [recommandations relatives aux identités et aux accès](recommendations-reference.md#recs-identity).

Apprenez-en davantage sur la [surveillance des identités et des accès](security-center-identity-access.md).



## <a name="march-2020"></a>Mars 2020

Les mises à jour du mois de mars incluent :

- [Disponibilité générale de l’automatisation de flux de travail](#workflow-automation-is-now-generally-available)
- [Intégration d’Azure Security Center avec le Centre d’administration Windows](#integration-of-azure-security-center-with-windows-admin-center)
- [Protection pour Azure Kubernetes Service](#protection-for-azure-kubernetes-service)
- [Amélioration de l’expérience juste-à-temps](#improved-just-in-time-experience)
- [Deux recommandations de sécurité pour les applications web déconseillées](#two-security-recommendations-for-web-applications-deprecated)


### <a name="workflow-automation-is-now-generally-available"></a>Disponibilité générale de l’automatisation de flux de travail

La fonctionnalité d’automatisation de flux de travail d’Azure Security Center est désormais généralement disponible. Elle permet de déclencher automatiquement Logic Apps sur des alertes et recommandations de sécurité. En outre, des déclencheurs manuels sont disponibles pour les alertes et toutes les recommandations pour lesquelles l’option de correction rapide est disponible.

Chaque programme de sécurité comprend plusieurs workflows pour la réponse aux incidents. Ces processus peuvent inclure l’envoi de notifications aux parties prenantes concernées, le lancement d’un processus de gestion des changements et l’application d’étapes de correction spécifiques. Les experts en sécurité vous conseillent d’automatiser le plus possible les étapes de ces processus. L’automatisation contribue à réduire la surcharge et à renforcer votre sécurité en garantissant que les étapes du processus se déroulent rapidement, de manière cohérente et selon les exigences que vous avez prédéfinies.

Pour plus d’informations sur les fonctionnalités automatiques et manuelles d’Azure Security Center pour l’exécution de vos flux de travail, consultez [Automatisation des workflows](workflow-automation.md).

Apprenez-en davantage sur la [création de Logic Apps](../logic-apps/logic-apps-overview.md).


### <a name="integration-of-azure-security-center-with-windows-admin-center"></a>Intégration d’Azure Security Center avec le Centre d’administration Windows

Vous pouvez désormais déplacer vos serveurs Windows locaux du Centre d’administration Windows directement vers Azure Security Center. Azure Security Center devient alors votre unique fenêtre pour l’affichage des informations de sécurité de toutes vos ressources du Centre d’administration Windows, à savoir les serveurs locaux, les machines virtuelles et les charges de travail PaaS supplémentaires.

Après avoir déplacé un serveur du Centre d’administration Windows vers Azure Security Center, vous pourrez effectuer les opérations suivantes :

- Afficher les alertes et recommandations de sécurité dans l’extension Security Center du Centre d’administration Windows.
- Afficher la posture de sécurité et des informations détaillées supplémentaires sur vos serveurs gérés par le Centre d’administration Windows dans le Security Center à l’intérieur du portail Azure (ou via une API).

Apprenez-en davantage sur la [façon d’intégrer Azure Security Center avec le Centre d’administration Windows](windows-admin-center-integration.md).


### <a name="protection-for-azure-kubernetes-service"></a>Protection pour Azure Kubernetes Service

Azure Security Center développe ses fonctionnalités de sécurité de conteneur pour protéger Azure Kubernetes Service (AKS).

La plateforme open source populaire Kubernetes a été adoptée si largement qu’elle fait désormais figure de norme sectorielle pour l’orchestration de conteneurs. En dépit de cette implémentation largement répandue, il subsiste un manque de compréhension de la manière de sécuriser un environnement Kubernetes. La défense des surfaces d’attaque d’une application en conteneur requiert de l’expertise pour s’assurer que l’infrastructure est configurée de façon totalement sécurisée et constamment surveillée pour détecter des menaces potentielles.

La défense orchestrée par Azure Security Center comprend les composantes suivantes :

- **Détection et visibilité** : détection continue des instances AKS gérées à l’intérieur des abonnements inscrits auprès d’Azure Security Center.
- **Recommandations de sécurité** : recommandations actionnables pour vous aider à vous conformer aux meilleures pratiques en matière de sécurité pour AKS. Ces recommandations sont incluses dans votre degré de sécurisation pour garantir leur visibilité en lien avec la posture de sécurité de votre organisation. Voici un exemple de recommandation relative à AKS : « Le contrôle d’accès en fonction du rôle doit être utilisé pour limiter l’accès à un cluster Kubernetes Service ».
- **Protection contre les menaces** : grâce à une analyse continue de votre déploiement AKS, Azure Security Center vous avertit des menaces et activités malveillantes détectées au niveau de l’hôte et du cluster AKS.

Pour en savoir plus, consultez [Intégration d’Azure Kubernetes Service avec Security Center](defender-for-kubernetes-introduction.md).

Apprenez-en davantage sur les [fonctionnalités de sécurité de conteneur d’Azure Security Center](container-security.md).


### <a name="improved-just-in-time-experience"></a>Amélioration de l’expérience juste-à-temps

Les fonctionnalités, le fonctionnement et l’interface utilisateur des outils juste-à-temps de l’Azure Security Center qui sécurisent vos ports de gestion ont été améliorés comme suit : 

- **Champ de justification** : lors de la demande d’accès à une machine virtuelle via la page Juste-à-temps du portail Azure, un nouveau champ facultatif est disponible pour entrer une justification de la demande. Le journal d’activité permet de suivre les informations entrées dans ce champ. 
- **Nettoyage automatique des règles JAT redondantes** : chaque fois que vous mettez à jour une stratégie JAT, un outil de nettoyage s’exécute automatiquement pour vérifier la validité de votre ensemble de règles. L’outil recherche les incompatibilités entre les règles de votre stratégie et les règles du groupe de sécurité réseau. Si l’outil de nettoyage détecte une incompatibilité, il en détermine la cause et, lorsque cela ne présente aucun risque, supprime les règles intégrées qui ne sont plus nécessaires. Le nettoyeur ne supprime jamais les règles que vous avez créées. 

Apprenez-en davantage sur la [fonctionnalité d’accès JAT](security-center-just-in-time.md).


### <a name="two-security-recommendations-for-web-applications-deprecated"></a>Deux recommandations de sécurité pour les applications web déconseillées

Deux recommandations de sécurité relatives aux applications web sont déconseillées : 

- Les règles relatives aux applications web sur des groupes de sécurité réseau IaaS doivent être renforcées.
    (Stratégie associée : Les règles de groupe de sécurité réseau pour les applications web IaaS doivent être renforcées)

- L’accès à App Services doit être limité.
    (Stratégie associée : L’accès à App Services doit être restreint [préversion])

Ces recommandations n’apparaissent plus dans la liste de recommandations d’Azure Security Center. Les stratégies associées ne seront plus incluses dans l’initiative nommée « Security Center par défaut ».

Apprenez-en davantage sur les [recommandations de sécurité](recommendations-reference.md).




## <a name="february-2020"></a>Février 2020

### <a name="fileless-attack-detection-for-linux-preview"></a>Détection d’attaque sans fichier pour Linux (préversion)

À mesure que les attaquants intensifient le recours à des méthodes de plus en plus furtives pour éviter d’être détectés, Azure Security Center étend la détection d’attaque sans fichier pour Linux, en plus de Windows. Les attaques sans fichier exploitent des vulnérabilités logicielles, injectent des charges utiles malveillantes dans des processus système inoffensifs et se cachent en mémoire. Ces techniques sont les suivantes :

- réduire ou éliminer les traces de logiciels malveillants sur disque ;
- réduire considérablement les risques de détection par des solutions d’analyse de programmes malveillants sur disque.

Pour contrer cette menace, Azure Security Center a publié une fonctionnalité de détection d’attaque sans fichier pour Windows en octobre 2018, et a maintenant étendu la détection d’attaque sans fichier également sur Linux. 



## <a name="january-2020"></a>Janvier 2020

### <a name="enhanced-secure-score-preview"></a>Amélioration du degré de sécurisation (préversion)

Une version améliorée de la fonctionnalité de degré de sécurisation d’Azure Security Center est désormais disponible en préversion. Dans cette version, plusieurs recommandations sont regroupées en contrôles de sécurité qui reflètent mieux vos surfaces d’attaque vulnérables (et, par exemple, restreignent l’accès aux ports de gestion).

Familiarisez-vous avec les changements apportés au degré de sécurisation au cours de la phase de préversion, et épinglez d’autres corrections qui vous aideront à sécuriser davantage votre environnement.

Pour en savoir plus, consultez [Version améliorée du degré de sécurisation (préversion)](secure-score-security-controls.md).



## <a name="november-2019"></a>Novembre 2019

Les mises à jour en novembre sont les suivantes :
 - [Protection contre les menaces pour Azure Key Vault dans les régions d’Amérique du Nord (préversion)](#threat-protection-for-azure-key-vault-in-north-america-regions-preview)
 - [La protection contre les menaces pour Stockage Azure inclut le filtrage de la réputation des programmes malveillants](#threat-protection-for-azure-storage-includes-malware-reputation-screening)
 - [Automatisation des workflows avec Azure Logic Apps (préversion)](#workflow-automation-with-logic-apps-preview)
 - [Disponibilité générale d’un correctif rapide pour les ressources en bloc](#quick-fix-for-bulk-resources-generally-available)
 - [Analyser des images conteneur pour détecter les vulnérabilités (préversion)](#scan-container-images-for-vulnerabilities-preview)
 - [Autres normes de conformité réglementaire (préversion)](#additional-regulatory-compliance-standards-preview)
 - [Protection contre les menaces pour Azure Kubernetes Service (préversion)](#threat-protection-for-azure-kubernetes-service-preview)
 - [Évaluation des vulnérabilités des machines virtuelles (préversion)](#virtual-machine-vulnerability-assessment-preview)
 - [Advanced Data Security pour les serveurs SQL sur Machines virtuelles Microsoft Azure (préversion)](#advanced-data-security-for-sql-servers-on-azure-virtual-machines-preview)
 - [Prise en charge de stratégies personnalisées (préversion)](#support-for-custom-policies-preview)
 - [Extension de la couverture d’Azure Security Center avec une plateforme pour la communauté et les partenaires](#extending-azure-security-center-coverage-with-platform-for-community-and-partners)
 - [Intégrations avancées avec exportation de recommandations et d’alertes (préversion)](#advanced-integrations-with-export-of-recommendations-and-alerts-preview)
 - [Intégrer des serveurs locaux à Security Center à partir du Centre d’administration Windows (préversion)](#onboard-on-prem-servers-to-security-center-from-windows-admin-center-preview)

### <a name="threat-protection-for-azure-key-vault-in-north-america-regions-preview"></a>Protection contre les menaces pour Azure Key Vault dans les régions d’Amérique du Nord (préversion)

Azure Key Vault est un service essentiel pour la protection des données et l’amélioration des performances des applications cloud, offrant la possibilité de gérer de manière centralisée les clés, les secrets, les clés de chiffrement et les stratégies dans le cloud. Étant donné qu’Azure Key Vault stocke des données sensibles et stratégiques, ses coffres de clés et les données qui y sont stockées exigent une sécurité maximale.

La prise en charge par Azure Security Center de la protection d’Azure Key Vault contre les menaces fournit une couche supplémentaire de veille de sécurité qui détecte les tentatives inhabituelles et potentiellement nuisibles d’accès aux coffres de clés ou d’exploitation de ceux-ci. Cette nouvelle couche de protection permet aux clients de traiter les menaces pesant sur leurs coffres de clés sans être experts en sécurité, ainsi que de gérer des systèmes de surveillance de la sécurité. Cette fonctionnalité est en préversion publique dans les régions d’Amérique du Nord.


### <a name="threat-protection-for-azure-storage-includes-malware-reputation-screening"></a>La protection contre les menaces pour Stockage Azure inclut le filtrage de la réputation des logiciels malveillants

La protection contre les menaces pour le service Stockage Azure offre de nouvelles fonctionnalités de détection optimisées par les renseignements sur les menaces Microsoft, qui permettent de détecter les chargements de programmes malveillants sur le service Stockage Azure à partir d’une de réputation de hachage et des accès suspects en provenance d’un nœud de sortie Tor actif (proxy d’anonymisation). Vous pouvez désormais afficher les programmes malveillants détectés parmi les comptes de stockage à l’aide d’Azure Security Center.


### <a name="workflow-automation-with-logic-apps-preview"></a>Automatisation des flux de travail avec Azure Logic Apps (préversion)

Les organisations dont la sécurité, l’informatique et les opérations sont gérées de manière centralisée implémentent des processus de flux de travail internes pour conduire les actions requises en leur sein en cas de détection d’incohérences dans leur environnement. Dans de nombreux cas, ces flux de travail étant des processus reproductibles, une automatisation peut considérablement simplifier leur exécution au sein de l’organisation.

Aujourd’hui, nous introduisons dans Security Center une nouvelle fonctionnalité qui permet aux clients de créer des configurations d’automatisation tirant parti d’Azure Logic Apps, et d’élaborer des stratégies qui les déclencheront automatiquement en fonction de résultats d’ASC spécifiques, tels que des recommandations ou des alertes. Il est possible de configurer une application logique pour effectuer toute action personnalisée prise en charge par le vaste éventail de connecteurs d’applications logiques, ou d’utiliser l’un des modèles fournis par Security Center, tels que l’envoi d’un e-mail ou l’ouverture d’un ticket ServiceNow™.

Pour plus d’informations sur les fonctionnalités automatiques et manuelles d’Azure Security Center disponibles pour l’exécution de vos flux de travail, consultez [Automatisation des workflows](workflow-automation.md).

Pour en savoir plus sur la création d’applications logiques, consultez [Azure Logic Apps](../logic-apps/logic-apps-overview.md).


### <a name="quick-fix-for-bulk-resources-generally-available"></a>Disponibilité générale d’un correctif rapide pour les ressources en bloc

Compte tenu des nombreuses tâches confiées aux utilisateurs en lien avec le degré de sécurisation, il peut devenir difficile de résoudre efficacement les problèmes survenant au sein d’un part informatique de grande taille.

Pour simplifier la correction des configurations de sécurité incorrectes et appliquer rapidement des recommandations à un lot de ressources afin d’améliorer votre degré de sécurisation, utilisez une remédiation par correctif rapide.

Cette opération vous permet de sélectionner les ressources auxquelles à corriger et de lancer une action de correction qui configure le paramètre à votre place.

Un correctif rapide est dès aujourd’hui en disponibilité générale pour les clients dans la page Recommandations du Security Center.

Pour connaître les recommandations assorties d’un correctif rapide, consultez le [guide de référence relatif aux recommandations de sécurité](recommendations-reference.md).


### <a name="scan-container-images-for-vulnerabilities-preview"></a>Analyser des images de conteneur pour détecter des vulnérabilités (préversion)

Azure Security Center peut désormais analyser des images de conteneur dans Azure Container Registry pour rechercher des vulnérabilités.

Le balayage d’image fonctionne en analysant le fichier image du conteneur, puis en vérifiant s’il existe des vulnérabilités connues (optimisées par Qualys).

L’analyse proprement dite est déclenchée automatiquement lors de l’envoi (push) de nouvelles images de conteneur au Registre de conteneurs Azure (Azure Container Registry). Les vulnérabilités détectées sont présentées en tant que recommandations d’Azure Security Center, et incluses dans le degré de sécurisation Azure avec des informations sur la façon de les corriger afin de réduire la surface d’attaque autorisée.


### <a name="additional-regulatory-compliance-standards-preview"></a>Normes de conformité réglementaire supplémentaires (préversion)

Le tableau de bord Conformité réglementaire fournit des insights sur votre posture de conformité en fonction des évaluations du Security Center. Le tableau de bord indique la conformité de votre environnement avec des contrôles et exigences stipulés par des normes réglementaires et des benchmarks sectoriels spécifiques, et fournit des recommandations prescriptives sur la façon de s’y conformer.

À ce jour, ce tableau de bord prend en charge quatre normes intégrées :  Azure CIS 1.1.0, PCI-DSS, ISO 27001 et SOC-TSP. Nous annonçons à présent la pris en charge en préversion publique de normes supplémentaires, à savoir : NIST SP 800-53 R4, SWIFT CSP CSCF v2020, Canada Federal PBMM et UK Official avec UK NHS. Nous publions également une version mise à jour d’Azure CIS 1.1.0, qui couvre davantage de contrôles à partir de l’extensibilité standard et améliorée.

[Apprenez-en davantage sur la personnalisation de l’ensemble de normes de votre tableau de bord de conformité réglementaire](update-regulatory-compliance-packages.md).


### <a name="threat-protection-for-azure-kubernetes-service-preview"></a>Protection contre les menaces pour Azure Kubernetes Service (préversion)

Kubernetes devient rapidement la nouvelle norme pour le déploiement et la gestion de logiciels dans le cloud. Rares sont les utilisateurs qui ont une expérience approfondie de Kubernetes et nombreux sont ceux qui se concentrent uniquement sur l’ingénierie et l’administration générales en négligeant la sécurité. Il convient de configurer l’environnement Kubernetes avec soin pour le sécuriser en s’assurant qu’aucune porte laissée ouverte, donnant sur une surface d’attaque de conteneurs, n’est exposée pour des attaquants. Azure Security Center étend son support en lien avec les conteneurs à un service Azure qui connaît une croissance des plus rapides, à savoir Azure Kubernetes Service (AKS).

Les nouvelles fonctionnalités de cette préversion publique sont les suivantes :

- **Détection et visibilité** : détection continue des instances AKS gérées à l’intérieur des abonnements inscrits de Security Center.
- **Recommandations concernant le degré de sécurisation** : éléments actionnables pour aider les clients à se conformer aux meilleures pratiques de sécurité pour AKS, et renforcer leur degré de sécurisation. Ces recommandations incluent des éléments tels que « Le contrôle d’accès en fonction du rôle doit être utilisé pour limiter l’accès à un cluster de service Kubernetes ».
- **Détection des menaces** : analyses basées sur un hôte ou un cluster, par exemple, « un conteneur privilégié détecté ».


### <a name="virtual-machine-vulnerability-assessment-preview"></a>Évaluation des vulnérabilités des machines virtuelles (préversion)

Les applications installées sur les machines virtuelles peuvent souvent présenter des vulnérabilités susceptibles d’entraîner une violation de ces machines. Nous annonçons que le niveau de service Standard de Security Center comprend une évaluation des vulnérabilités intégrée pour les machines virtuelles sans frais supplémentaires. L’évaluation des vulnérabilités, optimisée par Qualys dans la préversion publique, vous permet d’analyser en continu toutes les applications installées sur une machine virtuelle afin d’épingler celles qui sont vulnérables, et de présenter les résultats sur le portail Security Center. Security Center prend en charge toutes les opérations de déploiement afin que l’utilisateur n’ait aucun travail supplémentaire à accomplir. À l’avenir, nous envisageons de fournir des options d’évaluation des vulnérabilités pour satisfaire aux besoins uniques de nos clients.

[Apprenez-en davantage sur l’évaluation des vulnérabilités de vos machines virtuelles Azure](deploy-vulnerability-assessment-vm.md).


### <a name="advanced-data-security-for-sql-servers-on-azure-virtual-machines-preview"></a>Advanced Data Security pour SQL Server sur machines virtuelles Azure (préversion)

Le support d’Azure Security Center en lien avec la protection contre les menaces et l’évaluation des vulnérabilités pour les bases de données SQL s’exécutant sur des machines virtuelles IaaS est désormais en préversion.

La fonctionnalité [Évaluation des vulnérabilités](../azure-sql/database/sql-vulnerability-assessment.md) est un service simple à configurer, qui vous permet de découvrir, suivre et de corriger des vulnérabilités de base de données potentielles. Il offre une visibilité sur votre posture de sécurité en lien avec le degré de sécurisation Azure, et inclut des étapes pour résoudre des problèmes de sécurité et renforcer la protection de votre base de données.

La [protection avancée contre les menaces](../azure-sql/database/threat-detection-overview.md) détecte des activités anormales indiquant des tentatives inhabituelles et potentiellement dangereuses d’accès à votre serveur SQL ou d’exploitation de celui-ci. Elle supervise en permanence votre base de données pour détecter des activités suspectes, et envoie des alertes de sécurité orientées action en cas de modèles d’accès anormaux à la base de données. Ces alertes fournissent des détails sur les activités suspectes et des mesures recommandées pour examiner et atténuer la menace.


### <a name="support-for-custom-policies-preview"></a>Prise en charge de stratégies personnalisées (préversion)

Azure Security Center prend désormais en charge les stratégies personnalisées (en préversion).

Nos clients ont exprimé le souhait d’étendre la couverture de leurs évaluations de la sécurité actuelles dans Security Center en y ajoutant leurs propres évaluations de la sécurité, basées sur des stratégies qu’ils créent dans Azure Policy. Grâce à la prise en charge des stratégies personnalisées, c’est désormais possible.

Ces stratégies personnalisées font désormais partie des recommandations d’Azure Security Center, du degré de sécurisation et du tableau de bord des normes de conformité réglementaire. Avec la prise en charge des stratégies personnalisées, vous pouvez désormais créer une initiative personnalisée dans Azure Policy, puis l’ajouter en tant que stratégie dans Azure Security Center et la visualiser en tant que recommandation.


### <a name="extending-azure-security-center-coverage-with-platform-for-community-and-partners"></a>Extension de la couverture d’Azure Security Center avec une plateforme pour la communauté et les partenaires

Utilisez Security Center pour recevoir des recommandations, non seulement de Microsoft, mais aussi de solutions existantes de partenaires tels que Check Point, Tenable et CyberArk, avec de nombreuses intégrations supplémentaires à venir.  Le flux d’intégration simple de Security Center peut connecter vos solutions existantes à Security Center, ce qui vous permet d’afficher les recommandations relatives à votre posture de sécurité dans un emplacement unique, de générer des rapports unifiés et de tirer parti de toutes les fonctionnalités de Security Center en lien avec les recommandations intégrées et de partenaires. Vous pouvez également exporter les recommandations de Security Center vers des produits partenaires.

[Apprenez-en davantage sur l’Association de sécurité intelligente de Microsoft](https://www.microsoft.com/security/partnerships/intelligent-security-association).



### <a name="advanced-integrations-with-export-of-recommendations-and-alerts-preview"></a>Intégrations avancées avec exportation de recommandations et d’alertes (préversion)

Afin d’activer des scénarios de niveau entreprise en plus de Security Center, il est désormais possible d’utiliser des alertes et recommandations du Security Center dans des emplacements supplémentaires, à l’exception du portail Azure ou de l’API. Vous pouvez les exporter directement vers un Event Hub et des espaces de travail Log Analytics. Voici quelques flux de travail que vous pouvez créer autour de ces nouvelles fonctionnalités :

- Avec une exportation vers un espace de travail Log Analytics, vous pouvez créer des tableaux de bord personnalisés avec Power BI.
- Avec une exportation vers Event Hub, vous pouvez exporter les alertes et recommandations d’Azure Security Center vers vos informations de sécurité et gestion d’événements (SIEM) tierces, vers une solution tierce en temps réel ou vers Azure Data Explorer.


### <a name="onboard-on-prem-servers-to-security-center-from-windows-admin-center-preview"></a>Intégrer des serveurs locaux à Security Center à partir du Centre d’administration Windows (préversion)

Le Centre d’administration Windows est un portail de gestion pour les serveurs Windows non déployés dans Azure, qui offre à ceux-ci plusieurs fonctionnalités de gestion Azure, telles que la sauvegarde et les mises à jour système. Nous avons récemment ajouté la possibilité d’intégrer ces serveurs non-Azure de façon à ce qu’ils soient protégés par ASC directement à partir du Centre d’administration Windows.

Avec cette nouvelle expérience, les utilisateurs devront intégrer un serveur WAC à Azure Security Center et activer l’affichage de des alertes et recommandations de sécurité de celui-ci directement dans le Centre d’administration Windows.


## <a name="september-2019"></a>Septembre 2019

Les mises à jour en septembre sont les suivantes :

 - [Gestion des règles avec des améliorations des contrôles d’application adaptatifs](#managing-rules-with-adaptive-application-controls-improvements)
 - [Contrôler la recommandation de sécurité de conteneur à l’aide d’Azure Policy](#control-container-security-recommendation-using-azure-policy)

### <a name="managing-rules-with-adaptive-application-controls-improvements"></a>Gestion des règles avec des améliorations des contrôles d’application adaptatifs

L’expérience de gestion des règles pour les machines virtuelles à l’aide de contrôles d’application adaptatifs a été améliorée. Les contrôles d’application adaptatifs d’Azure Security Center vous aident à contrôler les applications qui peuvent s’exécuter sur vos machines virtuelles. En plus d’une amélioration générale de la gestion des règles, un nouvel avantage vous permet de contrôler les types de fichiers qui seront protégés lorsque vous ajoutez une nouvelle règle.

[Apprenez-en davantage sur les contrôles d’application adaptatifs](security-center-adaptive-application.md).


### <a name="control-container-security-recommendation-using-azure-policy"></a>Contrôler la recommandation de sécurité de conteneur à l’aide d’Azure Policy

La recommandation d’Azure Security Center pour corriger des vulnérabilités dans la sécurité de conteneur peut désormais être activée ou désactivée via Azure Policy.

Pour afficher vos stratégies de sécurité activées, dans le Security Center, ouvrez la page Stratégie de sécurité.


## <a name="august-2019"></a>Août 2019

Les mises à jour en août sont les suivantes :

 - [Accès de machine virtuelle juste-à-temps (JAT) pour Pare-feu Azure](#just-in-time-jit-vm-access-for-azure-firewall)
 - [Correction en un seul clic pour améliorer votre posture de sécurité (préversion)](#single-click-remediation-to-boost-your-security-posture-preview)
 - [Gestion multilocataire](#cross-tenant-management)

### <a name="just-in-time-jit-vm-access-for-azure-firewall"></a>Accès de machine virtuelle juste-à-temps (JAT) pour le Pare-feu Azure 

L’accès de machine virtuelle juste-à-temps (JAT) pour le Pare-feu Azure est désormais généralement disponible. Utilisez-le pour sécuriser vos environnements protégés par un Pare-feu Azure en plus de vos environnements protégés par un groupe de sécurité réseau.

L’accès de machine virtuelle JAT réduit l’exposition aux attaques volumétriques de réseau en fournissant aux machines virtuelles un accès contrôlé uniquement si nécessaire, à l’aide de vos règles de groupe de sécurité réseau et de Pare-feu Azure.

Lorsque vous activez l’accès JAT pour vos machines virtuelles, vous créez une stratégie qui détermine les ports à protéger, la durée pendant laquelle les ports doivent rester ouverts et les adresses IP approuvées à partir desquelles ces ports sont accessibles. Cette stratégie vous permet contrôler ce que les utilisateurs peuvent faire quand ils demandent l’accès.

Les demandes étant consignées dans le journal d’activité Azure, vous pouvez surveiller et vérifier facilement l’accès. La page juste-à-temps vous aide également à identifier rapidement les machines virtuelles existantes pour lesquelles l’accès JAT est activé et celles pour lesquelles il est recommandé.

[Apprenez-en davantage sur le Pare-feu Azure](../firewall/overview.md).


### <a name="single-click-remediation-to-boost-your-security-posture-preview"></a>Correction en un seul clic pour améliorer votre posture de sécurité (préversion)

Le degré de sécurisation est un outil qui vous aide à évaluer la sécurité de la charge de travail. Il examine vos recommandations de sécurité et les hiérarchise de façon à ce que vous sachiez celles que vous devez appliquer en premier. Cela vous aide à épingler les vulnérabilités de sécurité les plus graves pour hiérarchiser l’investigation.

Afin de simplifier la correction des configurations de sécurité incorrectes et de vous aider à améliorer rapidement votre degré de sécurisation, nous avons ajouté une nouvelle fonctionnalité qui vous permet d’appliquer une recommandation à un lot de ressources en un seul clic.

Cette opération vous permettra de sélectionner les ressources auxquelles vous souhaitez appliquer la correction, et de lancer une action de correction qui configurera le paramètre pour vous.

Pour connaître les recommandations assorties d’un correctif rapide, consultez le [guide de référence relatif aux recommandations de sécurité](recommendations-reference.md).


### <a name="cross-tenant-management"></a>Gestion multilocataire

Azure Security Center prend désormais en charge les scénarios de gestion mutualisée dans Azure Lighthouse. Cela vous permet d’acquérir une visibilité et de gérer la posture de sécurité de plusieurs locataires dans Azure Security Center. 

[Apprenez-en davantage sur les expériences de gestion mutualisée](security-center-cross-tenant-management.md).


## <a name="july-2019"></a>Juillet 2019

### <a name="updates-to-network-recommendations"></a>Mises à jour des recommandations pour le réseau

Azure Security Center a publié de nouvelles recommandations pour la mise en réseau, et amélioré des recommandations existantes. Désormais, l’utilisation d’Azure Security Center garantit une meilleure protection réseau pour vos ressources. 

[Apprenez-en davantage sur les recommandations pour le réseau](recommendations-reference.md#recs-network).


## <a name="june-2019"></a>Juin 2019

### <a name="adaptive-network-hardening---generally-available"></a>Disponibilité générale du renforcement du réseau adaptatif

L’une des principales surfaces d’attaque pour les charges de travail s’exécutant dans le cloud public est celle des interconnexions avec l’Internet public. Nos clients trouvent difficile de savoir quelles règles de groupe de sécurité mettre en place pour s’assurer que les charges de travail Azure ne sont disponibles que pour les plages sources requises. Cette fonctionnalité permet à Security Center d’apprendre le trafic réseau et les modèles de connectivité des charges de travail Azure, et fournit des recommandations de règle de groupe de sécurité réseau pour les machines virtuelles accessibles via Internet. Cela permet à nos clients de mieux configurer leurs stratégies d’accès réseau et de limiter leur exposition aux attaques. 

[Apprenez-en davantage sur le renforcement du réseau adaptative](security-center-adaptive-network-hardening.md).