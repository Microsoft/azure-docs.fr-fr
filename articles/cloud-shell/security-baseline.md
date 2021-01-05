---
title: Ligne de base de sécurité Azure pour Cloud Shell
description: La ligne de base de sécurité Cloud Shell fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 10/19/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 24856e458858ad86d953b50709669823b35794fc
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348260"
---
# <a name="azure-security-baseline-for-cloud-shell"></a>Ligne de base de sécurité Azure pour Cloud Shell

Cette ligne de base de sécurité applique les instructions du [ Benchmark de sécurité Azure version 1,0](../security/benchmarks/overview-v1.md) au Cloud Shell. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure.
Le contenu est regroupé par les **contrôles de sécurité** définis par le Benchmark de sécurité Azure et les conseils associés applicables au Cloud Shell. Les **contrôles** non applicables au Cloud Shell ont été exclus.

 
Pour voir comment Cloud Shell est entièrement mappé au Benchmark de sécurité Azure, consultez le[fichier de mappage complet Cloud Shell de la base de référence de sécurité](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Les clients peuvent déployer Azure Cloud Shell dans un réseau virtuel appartenant à un client.

Lorsque vous déployez Azure Cloud Shell dans un réseau virtuel appartenant à un client, vous devez créer ou utiliser un réseau virtuel existant. Assurez-vous que le réseau virtuel choisi dispose d’un groupe de sécurité réseau appliqué à ses sous-réseaux et des contrôles d’accès réseau configurés spécifiques aux ports et sources de confiance de votre application.

- [Déployer Cloud Shell dans un réseau virtuel Azure](private-vnet.md)

- [Comment créer un groupe de sécurité réseau avec des règles de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

- [Guide pratique pour déployer et configurer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui utilise la même autorisation que celle utilisée pour accéder au Portail Azure. Dans ce cas, une authentification unique dans le Portail Azure vous authentifiera également avec Cloud Shell. 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui utilise la même autorisation que celle utilisée pour accéder au Portail Azure. Dans ce cas, toute authentification MFA requise pour se connecter au Portail Azure est également nécessaire pour Cloud Shell. 

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images de conteneur sont surveillées et mises à jour par l’équipe Cloud Shell.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont chargés d’exécuter les outils d’analyse des vulnérabilités automatisés sur les logiciels qui s’exécutent dans l’environnement.  

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs de logiciels tiers

**Conseils** : Non applicable ; Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images de conteneur sont surveillées et mises à jour par l’équipe Cloud Shell.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de la gestion des correctifs logiciels en cours d’exécution dans leur environnement.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images de conteneur sont surveillées et mises à jour par l’équipe Cloud Shell.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de la correction des vulnérabilités découvertes par le biais de leurs analyses de vulnérabilités logicielles. Exportez les résultats de l’analyse à intervalles réguliers, et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. Lorsque vous suivez les recommandations de gestion des vulnérabilités proposées par Azure Security Center, vous pouvez pivoter vers le portail de la solution sélectionnée pour afficher les données d’analyse historiques.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images de conteneur sont surveillées et mises à jour par l’équipe Cloud Shell.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de la correction des vulnérabilités découvertes par le biais de leurs analyses de vulnérabilités logicielles.  Utilisez un programme de score de risque commun (par exemple, le système de notation des vulnérabilités courantes) ou les niveaux de risque par défaut fournis par votre outil d’analyse tiers. 

- [Publication du NIST--Système de notation des vulnérabilités courantes](https://www.nist.gov/publications/common-vulnerability-scoring-system)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell.  Le client peut installer ses propres outils dans sa propre image en fonction de ses besoins en matière d’organisation et les outils ne nécessitent pas `sudo` d’autorisations lors de l’installation.

Il est recommandé aux clients de créer un inventaire des logiciels approuvés installés par le biais de Azure Cloud Shell conformément aux besoins de leur organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Azure Cloud Shell est un service gratuit sans ressources détenues par le client.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell. 

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont chargés de surveiller les applications logicielles en cours d’exécution dans l’environnement pour s’assurer qu’elles sont approuvées par la politique organisationnelle.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Azure Cloud Shell est un service gratuit sans ressources détenues par le client.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell. 

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de surveiller les applications logicielles en cours d’exécution dans l’environnement pour s’assurer que les logiciels non approuvés sont gérés par la politique organisationnelle.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Azure Cloud Shell est un service gratuit sans ressources détenues par le client.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell.  Les outils spécifiques ne peuvent pas être supprimés par le client.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou applications dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de surveiller les applications en cours d’exécution dans l’environnement pour s’assurer qu’elles sont approuvées par la politique organisationnelle.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Non applicable ; Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images de conteneur sont surveillées et mises à jour par l’équipe Cloud Shell.

Azure Cloud Shell permet aux clients d’installer leurs propres outils ou logiciels dans leur propre image en fonction de leurs besoins organisationnels.

Les clients sont responsables de la tenue d’un inventaire des logiciels approuvés exécutés dans l’environnement pour s’assurer qu’ils soient approuvés par la politique organisationnelle.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Les actions effectuées dans Cloud Shell fonctionnent de la même façon que les actions effectuées à partir des mêmes outils ou langages exécutés dans un environnement local.  Les actions des outils et des langues individuels doivent être restreintes, les clients ne peuvent pas restreindre l’accès à Cloud Shell ou limiter ce qui est disponible pour un utilisateur.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Azure Cloud Shell peut être isolé dans un réseau virtuel client.

- [Déployer Cloud Shell dans un réseau virtuel Azure](private-vnet.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Cloud Shell permet d’exécuter des scripts dans, de les créer et de les télécharger dans l’environnement Cloud Shell.  Nous vous recommandons de déplacer les informations d’identification dans Azure Key Vault.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell.  Le client peut installer ses propres outils dans sa propre image en fonction de ses besoins en matière d’organisation et les outils ne nécessitent pas `sudo` d’autorisations lors de l’installation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="83-ensure-antimalware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont à jour

**Conseils** : Azure Cloud Shell est une expérience de ligne de commande basée sur un navigateur qui est utilisée pour la gestion interactive des ressources de Cloud.  Chaque conteneur de client est éphémère. Un nouveau conteneur est utilisé pour chaque session.  Les images et les outils conteneur sont surveillés et mis à jour par l’équipe Cloud Shell.  Le client peut installer ses propres outils dans sa propre image en fonction de ses besoins en matière d’organisation et les outils ne nécessitent pas `sudo` d’autorisations lors de l’installation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.
- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md) 
- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 
- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 
- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, ainsi qu’à l’intention malveillante estimée de l’activité à l’origine du déclenchement de l’alerte.
En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.
- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md) 
- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.
- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.
- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.
- [Comment configurer l’exportation continue](../security-center/continuous-export.md) 
- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.
- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.
- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 
- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)