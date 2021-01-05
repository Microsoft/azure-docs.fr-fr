---
title: Vue d’ensemble d’Azure Operational Security | Microsoft Docs
description: En savoir plus sur Azure Operational Security dans cette vue d’ensemble. Operational Security fait référence aux services, contrôles et fonctionnalités de protection des ressources.
services: security
documentationcenter: na
author: unifycloud
manager: rkarlin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/31/2019
ms.author: tomsh
ms.openlocfilehash: 9dcb1113bb37cb0f5f65c54ae9d6ca76a88330c1
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698663"
---
# <a name="azure-operational-security-overview"></a>Vue d’ensemble d’Azure Operational Security

[Azure Operational Security](./operational-security.md) comprend les services, contrôles et fonctionnalités auxquels les utilisateurs ont accès pour protéger leurs données, leurs applications et d’autres ressources dans Microsoft Azure. C’est un cadre qui intègre les connaissances acquises via différents outils propres à Microsoft. Ces fonctionnalités incluent Microsoft Security Development Lifecycle (SDL), le programme Microsoft Security Response Center et une connaissance approfondie du paysage des menaces de cybersécurité.

## <a name="azure-management-services"></a>Services de gestion Azure

Une équipe des opérations informatiques est chargée de gérer l’infrastructure du centre de données, les applications et les données, ainsi que la stabilité et la sécurité de ces systèmes. Toutefois, l’obtention d’informations de sécurité dans des environnements informatiques de plus en plus complexes requiert souvent de la part des organisations qu’elles bricolent et rassemblent les données de plusieurs systèmes de sécurité et de gestion.

[Journaux Microsoft Azure Monitor](../../azure-monitor/overview.md) est une solution de gestion informatique basée sur le cloud qui vous permet de gérer et protéger votre infrastructure locale et dans le cloud. Ses fonctionnalités principales sont fournies par les services suivants qui s’exécutent dans Azure. Azure comporte plusieurs services qui vous aident à gérer et protéger votre infrastructure locale et cloud. Chaque service propose une fonction de gestion spécifique. Vous pouvez combiner des services pour obtenir différents scénarios d’administration. 

### <a name="azure-monitor"></a>Azure Monitor

[Azure Monitor](../../azure-monitor/overview.md) collecte des données à partir de sources managées dans des banques de données centrales. Ces données peuvent inclure des événements, des données de performance ou des données personnalisées fournies par l’API. Une fois collectées, les données sont disponibles pour les fonctions d’alerte, d’analyse et d’exportation.

Vous pouvez consolide rles données provenant de différentes sources, et combiner celles de vos services Azure à celles de votre environnement local. La solution Journaux Azure Monitor dissocie clairement la collecte des données, de l’action exécutée sur celles-ci, afin que toutes les actions soient disponibles pour tous les types de données.

### <a name="automation"></a>Automatisation

[Azure Automation](../../automation/automation-intro.md) vous permet d’automatiser les tâches répétitives, manuelles, longues et susceptibles de générer des erreurs, qui sont fréquemment exécutées dans un environnement cloud et d’entreprise. Il fait gagner du temps et augmente la fiabilité des tâches d’administration. Il planifie même l’exécution automatique de ces tâches à intervalles réguliers. Vous pouvez automatiser les processus à l’aide de runbooks ou automatiser la gestion de la configuration avec la Configuration de l’état souhaité (DSC, Desired State Configuration).

### <a name="backup"></a>Backup

[Sauvegarde Azure](../../backup/backup-overview.md) est le service Azure qui vous permet de sauvegarder (ou protéger) et de restaurer vos données dans Microsoft Cloud. Il remplace votre solution de sauvegarde locale ou hors site par une solution cloud à la fois fiable, sécurisée et économique.

Sauvegarde Azure propose plusieurs composants que vous pouvez télécharger et déployer sur l’ordinateur ou le serveur approprié, ou encore dans le cloud. Vous déployez un composant (ou un agent) en fonction de ce que vous souhaitez protéger. Vous pouvez utiliser tous les composants de Sauvegarde Azure (que vous protégiez des données en local ou dans le cloud) pour sauvegarder des données dans un coffre Azure Recovery Services.

Pour plus d’informations, consultez le [tableau des composants de Sauvegarde Azure](../../backup/backup-overview.md#what-can-i-back-up).

### <a name="site-recovery"></a>Site Recovery

[Azure Site Recovery](https://azure.microsoft.com/documentation/services/site-recovery) assure la continuité de l’activité en orchestrant la réplication des machines virtuelles et physiques locales vers Azure ou vers un site secondaire. Si votre site principal est indisponible, vous basculez vers l’emplacement secondaire afin que les utilisateurs puissent continuer à travailler. Puis, vous rebasculez vers le site principal lorsque les systèmes redeviennent opérationnels. Utilisez Azure Security Center pour effectuer une détection des menaces plus intelligente et efficace.

## <a name="azure-active-directory"></a>Azure Active Directory

[Azure Active Directory (Azure AD)](../../active-directory/manage-apps/what-is-application-management.md) est un service d’identité complet qui :

-   Autorise la gestion des identités et des accès (IAM) dans un service cloud.
-   Assure une gestion centralisée des accès, l’authentification unique et la création de rapports.
-   Prend en charge la gestion intégrée des accès à des [milliers d’applications](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory) dans Place de marché Microsoft Azure, notamment Salesforce, Google Apps, Box et Concur.

Azure AD inclut également une suite complète de [fonctionnalités de gestion des identités](./identity-management-overview.md#security-monitoring-alerts-and-machine-learning-based-reports), dont :

- [Authentification multifacteur](../../active-directory/authentication/concept-mfa-howitworks.md)
- [Gestion des mots de passe en libre-service](https://azure.microsoft.com/resources/videos/self-service-password-reset-azure-ad/)
- [Gestion des groupes en libre service](../../active-directory/user-help/active-directory-passwords-update-your-own-password.md)
- [Gestion des comptes privilégiés](../../active-directory/privileged-identity-management/pim-configure.md)
- [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)
- [Surveillance de l’utilisation des applications](../../active-directory/hybrid/whatis-hybrid-identity.md)
- [Audit détaillé](../../active-directory/reports-monitoring/concept-audit-logs.md)
- [Surveillance et alerte de sécurité](../../security-center/security-center-managing-and-responding-alerts.md)

Avec Azure Active Directory, toutes les applications que vous publiez pour vos partenaires et clients (professionnels et particuliers) ont les mêmes fonctionnalités de gestion des identités et des accès. Vous êtes ainsi en mesure de réduire de manière significative vos coûts opérationnels.

## <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure. Il intègre la gestion des stratégies de sécurité et de la surveillance dans vos abonnements. Il permet de détecter les menaces qui pourraient passer inaperçues et fonctionne avec un vaste écosystème de solutions de sécurité.

[Sauvegardez les données des machines virtuelles](../../security-center/security-center-introduction.md) dans Azure en offrant une visibilité sur les paramètres de sécurité de vos machines virtuelles et en surveillant les menaces. Security Center surveille les éléments suivants sur vos machines virtuelles :

- Paramètres de sécurité du système d’exploitation avec les règles de configuration recommandées.
- Mise à jour critiques et de sécurité du système qui sont manquantes.
- Suggestions pour la protection des points de terminaison.
- Validation du chiffrement des disques.
- Attaques réseau.

Security Center utilise le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md). Azure RBAC fournit des [rôles intégrés](../../role-based-access-control/built-in-roles.md) qui peuvent être attribués à des utilisateurs, des groupes et des services dans Azure.

Security Center évalue la configuration de vos ressources pour identifier les vulnérabilités et les problèmes de sécurité. Dans Security Center, vous ne voyez les informations concernant une ressource que si vous avez reçu le rôle propriétaire, collaborateur ou lecteur pour l’abonnement ou le groupe auquel appartient la ressource.

>[!Note]
>Pour plus d’informations sur les rôles et les actions autorisées dans Security Center, consultez [Permissions in Azure Security Center (Autorisations dans Azure Security Center)](../../security-center/security-center-permissions.md).

Security Center utilise Microsoft Monitoring Agent. Il s’agit du même agent que celui utilisé par le service Azure Monitor. Les données collectées par cet agent sont stockées dans un [espace de travail](../../azure-monitor/platform/manage-access.md) Log Analytics associé à votre abonnement Azure ou dans un nouvel espace de travail, selon l’emplacement géographique de la machine virtuelle.

## <a name="azure-monitor"></a>Azure Monitor

Les problèmes de performances dans votre application cloud peuvent affecter votre entreprise. Avec plusieurs composants interconnectés et de nouvelles versions fréquentes, des dégradations peuvent se produire à tout moment. Et si vous développez une application, vos utilisateurs découvrent généralement des problèmes que vous n’avez pas trouvés dans le test. Vous devez être informé de ces problèmes immédiatement et avoir en main des outils pour les diagnostiquer et les résoudre.

[Azure Monitor](../../azure-monitor/overview.md) est l’outil de base pour la surveillance des services s’exécutant sur Azure. Il vous fournit des données au niveau de l’infrastructure sur le débit d’un service et l’environnement. Si vous gérez toutes vos applications dans Azure et décidez d’augmenter ou de diminuer les ressources, Azure Monitor est l’outil qu’il vous faut.

Vous pouvez également utiliser les données de surveillance pour obtenir des informations détaillées sur votre application. Ces connaissances peuvent vous aider à améliorer les performances de l’application ou sa facilité de gestion, ou à automatiser des actions qui exigeraient normalement une intervention manuelle.

Azure Monitor comprend les composants suivants :

### <a name="azure-activity-log"></a>Journaux d’activité

Le [journal d’activité Azure](../../azure-monitor/platform/platform-logs-overview.md) fournit des informations sur les opérations qui ont été effectuées sur les ressources de votre abonnement. Il était auparavant appelé « journal d’audit » ou « journal des opérations », car il indique les événements de plan de contrôle concernant vos abonnements.

### <a name="azure-diagnostic-logs"></a>Journaux de diagnostic Azure

Les [journaux de diagnostic Azure](../../azure-monitor/platform/platform-logs-overview.md) sont générés par une ressource et fournissent des informations complètes et fréquentes sur le fonctionnement de cette ressource. Le contenu de ces journaux d’activité varie en fonction du type de ressource.

Les journaux d’activité système des événements Windows sont une catégorie de journaux de diagnostic pour les machines virtuelles. Les journaux d’activité d’objet blob, de table et de file d’attente sont des catégories de journaux de diagnostic pour les comptes de stockage.

Les journaux de diagnostic sont différents du [journal d’activité](../../azure-monitor/platform/platform-logs-overview.md). Le journal d’activité fournit des informations sur les opérations qui ont été effectuées sur les ressources de votre abonnement. Les journaux de diagnostic fournissent des informations détaillées sur les opérations effectuées par votre ressource.

### <a name="metrics"></a>Mesures

Azure Monitor vous fournit des données de télémétrie qui vous offrent une visibilité sur les performances et l’intégrité de vos charges de travail sur Azure. Les [métriques](../../azure-monitor/platform/data-platform.md) (aussi appelées compteurs de performances) générées par la plupart des ressources Azure sont les données de télémétrie Azure les plus importantes. Azure Monitor propose plusieurs façons de configurer et d’utiliser ces mesures pour l’analyse et le dépannage.

### <a name="azure-diagnostics"></a>Diagnostics Azure

Diagnostics Azure autorise la collecte de données de diagnostic sur une application déployée. Vous pouvez utiliser l’extension Diagnostics de différentes sources. Les sources actuellement prises en charge sont les [rôles de service cloud Azure](/visualstudio/azure/vs-azure-tools-configure-roles-for-cloud-service), les [machines virtuelles Azure](/visualstudio/azure/vs-azure-tools-configure-roles-for-cloud-service) exécutant Microsoft Windows et [Azure Service Fabric](../../azure-monitor/platform/diagnostics-extension-overview.md).

## <a name="azure-network-watcher"></a>Azure Network Watcher

Les clients créent un réseau de bout en bout dans Azure en orchestrant et en composant des ressources réseau telles que des réseaux virtuels, ExpressRoute, Azure Application Gateway, des équilibreurs de charge, etc. La surveillance est disponible sur chacune des ressources réseau.

Le réseau de bout en bout peut avoir des configurations complexes et les interactions entre les ressources. Au final, ces scénarios complexes requièrent une surveillance spécifique via [Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md).

Network Watcher simplifie la surveillance et le diagnostic de votre réseau Azure. Vous pouvez utiliser les outils de diagnostic et de visualisation de Network Watcher pour :

- Effectuer des captures de paquets à distance sur une machine virtuelle Azure.
- Analyser votre trafic réseau de manière détaillée à l’aide des journaux de flux.
- Diagnostiquer la passerelle VPN Azure et les connexions.

Network Watcher propose actuellement les fonctionnalités suivantes :

- [Topologie](../../network-watcher/view-network-topology.md) : affiche une vue des différentes interconnexions et associations entre les ressources réseau d'un groupe de ressources.
- [Capture de paquets variables](../../network-watcher/network-watcher-packet-capture-overview.md) : capture les données des paquets qui entrent et sortent d’une machine virtuelle. Les options avancées de filtrage et les contrôles précis, comme la possibilité de définir des limites de taille et de temps, sont sources de polyvalence. Les données des paquets peuvent être stockées dans un magasin de blobs ou sur le disque local au format .cap.
- [Vérification des flux IP](../../network-watcher/network-watcher-ip-flow-verify-overview.md) : détermine si un paquet est autorisé ou refusé en fonction des paramètres de flux (adresse IP de destination, adresse IP source, port de destination, port source et protocole) des paquets à 5 tuples. Si un groupe de sécurité refuse le paquet, la règle et le groupe qui ont refusé ce paquet sont renvoyés.
- [Tronçon suivant](../../network-watcher/network-watcher-next-hop-overview.md) : détermine le tronçon suivant des paquets routés dans la structure de réseau Azure pour diagnostiquer les itinéraires définis par l'utilisateur mais mal configurés.
- [Affichage des groupes de sécurité](../../network-watcher/network-watcher-security-group-view-overview.md) : Obtient les règles de sécurité effectives et appliquées à une machine virtuelle.
- [Journaux de flux NSG des groupes de sécurité réseau](../../network-watcher/network-watcher-nsg-flow-logging-overview.md) : vous permettent de capturer les journaux d’activité relatifs au trafic autorisé ou refusé par les règles de sécurité du groupe. Le flux est défini par des informations à 5 tuples : adresse IP source, adresse IP de destination, port source, port de destination et protocole.
- [Résolution des problèmes de passerelle de réseau virtuel et de connexion](../../network-watcher/network-watcher-troubleshoot-manage-rest.md) : permet de résoudre les problèmes des connexions et des passerelles de réseau virtuel.
- [Limites d’abonnement réseau](../../network-watcher/network-watcher-monitoring-overview.md) : vous permet d’afficher l’utilisation des ressources réseau par rapport aux limites.
- [Journaux de diagnostic](../../network-watcher/network-watcher-monitoring-overview.md) : fournit un volet permettant d’activer ou de désactiver les journaux de diagnostic des ressources réseau d’un groupe de ressources.

Pour plus d’informations, consultez [Créer une instance d’Azure Network Watcher](../../network-watcher/network-watcher-create.md).

## <a name="cloud-service-provider-access-transparency"></a>Transparence de l’accès du fournisseur de service cloud

[Customer Lockbox pour Microsoft Azure](customer-lockbox-overview.md) est un service intégré au portail Azure qui vous offre un contrôle explicite dans les rares cas où un ingénieur du support Microsoft peut avoir besoin d’accéder à vos données pour résoudre un problème.
Les cas où un ingénieur du support Microsoft requiert des autorisations élevées pour résoudre un problème sont rares, par exemple : débogage d’un problème d’accès à distance. Dans ces cas, les ingénieurs Microsoft utilisent le service d’accès juste-à-temps qui fournit une autorisation limitée avec un accès limité au service.  
Microsoft a toujours obtenu le consentement des clients concernant l’accès, mais Customer Lockbox vous permet à présent de vérifier et d’approuver ou de refuser ces demandes à partir du portail Azure. Les ingénieurs du support Microsoft n’obtiendront l’accès que si vous approuvez la requête.

## <a name="standardized-and-compliant-deployments"></a>Déploiements standardisés et conformes

[Azure Blueprints](../../governance/blueprints/overview.md) permet aux architectes cloud et aux membres de l’informatique centrale de définir un ensemble reproductible de ressources Azure qui implémentent et respectent les normes, modèles et exigences d’une organisation.  
Cela permet aux équipes DevOps de créer et de mettre en place rapidement de nouveaux environnements et de savoir qu’elles les génèrent à l’aide d’une infrastructure qui respecte la conformité de l’organisation.
Blueprints fournit un moyen déclaratif d’orchestrer le déploiement de divers modèles de ressources et d’autres artefacts, notamment ceux-ci :

- Affectations de rôles
- Affectations de stratégies
- Modèles Microsoft Azure Resource Manager
- Groupes de ressources

## <a name="devops"></a>DevOps

Avant le développement d’applications [DevOps](https://www.visualstudio.com/learn/what-is-devops/), des équipes sont chargées de définir les fonctionnalités métier d’un programme logiciel et d’écrire le code. Ensuite, une autre équipe de contrôle qualité teste le programme dans un environnement de développement isolé. Si le programme passe le contrôle, l’équipe de contrôle qualité transmet le code à l’équipe des opérations pour qu’elle le déploie. Les équipes de déploiement sont ensuite réparties en groupes, par exemple mise en réseau et base de données. Chaque fois qu’un logiciel est transmis à une équipe indépendante, cela crée des goulots d’étranglement.

DevOps permet aux équipes de proposer des solutions plus sécurisées et de meilleure qualité, plus rapidement et à moindre coût. Les clients s’attendent à une expérience dynamique et fiable lors de l’utilisation de logiciels et de services. Les équipes doivent effectuer rapidement une itération sur les mises à jour logicielles et mesurer l’impact de ces mises à jour. Elles doivent réagir rapidement avec des nouvelles itérations de développement pour résoudre les problèmes ou valoriser la solution.  

Les plateformes cloud comme Microsoft Azure ont supprimé les goulots d’étranglement traditionnels et aidé à faire de l’infrastructure une marchandise. Dans tous les secteurs, ce sont les logiciels qui font la différence dans les résultats des entreprises. Aucune organisation, aucun développeur ni aucun professionnel de l’informatique ne peut se permettre d’ignorer le mouvement DevOps.

Les professionnels DevOps expérimentés adoptent plusieurs des pratiques suivantes. Ces pratiques [impliquent de faire appel à des collaborateurs](https://www.visualstudio.com/learn/what-is-devops-culture/) pour concevoir des stratégies basées sur les scénarios spécifiques de l’entreprise. Les outils contribuent à automatiser les différentes pratiques.

- Des techniques de [planification et gestion de projets agiles](https://www.visualstudio.com/learn/what-is-agile/) sont utilisées pour planifier et répartir le travail en sprints, pour gérer la capacité de l’équipe et pour aider les équipes à s’adapter rapidement à l’évolution des besoins de l’entreprise.
- Le [contrôle de version, généralement avec Git](https://www.visualstudio.com/learn/what-is-git/), permet à des équipes situées n’importe où dans le monde de partager la source et d’effectuer des intégrations avec des outils de développement logiciel pour automatiser le pipeline de mise en production.
- L’[intégration continue ](https://www.visualstudio.com/learn/what-is-continuous-integration/) stimule la fusion et le test de code en continu, ce qui permet de trouver les erreurs plus tôt.  Les autres avantages sont notamment un gain de temps sur la résolution des problèmes de fusion et la rapidité des retours pour les équipes de développement.
- La [livraison continue](https://www.visualstudio.com/learn/what-is-continuous-delivery/) de solutions logicielles dans les environnements de production et de test permet aux organisations de corriger rapidement les bogues et de répondre aux besoins de l’entreprise en constante évolution.
- La [surveillance](https://www.visualstudio.com/learn/what-is-monitoring/) des applications en cours d’exécution, y compris des environnements de production pour l’intégrité des applications et l’utilisation par les clients, aide les organisations à formuler une hypothèse et à confirmer/infirmer rapidement des stratégies.  Les données enrichies sont capturées et stockées dans divers formats d’enregistrement.
- L’[infrastructure en tant que code (IaC)](https://www.visualstudio.com/learn/what-is-infrastructure-as-code/) est une pratique qui permet d’automatiser et de valider la création et la suppression de réseaux et de machines virtuelles afin de fournir des plateformes stables et sûres pour l’hébergement d’applications.
- L’architecture en [microservices](https://www.visualstudio.com/learn/what-are-microservices/) permet d’isoler des cas d’usage métier en petits services réutilisables.  Cette architecture est synonyme d’extensibilité et d’efficacité.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la solution Security and Audit, consultez les articles suivants :

- [Sécurité et conformité](https://azure.microsoft.com/overview/trusted-cloud/)
- [Centre de sécurité Azure](../../security-center/security-center-introduction.md)
- [Azure Monitor](../../azure-monitor/overview.md)