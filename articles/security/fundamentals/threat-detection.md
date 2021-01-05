---
title: Détection avancée des menaces Azure | Microsoft Docs
description: Découvrez les fonctionnalités de détection avancée des menaces intégrées pour Azure, telles que le service Azure Active Directory Identity Protection.
services: security
documentationcenter: na
author: UnifyCloud
manager: barbkess
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2017
ms.author: TomSh
ms.openlocfilehash: b9770a43309f5471760dc2482833e4bab45c6f5b
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409924"
---
# <a name="azure-advanced-threat-detection"></a>Détection avancée des menaces Azure

Azure intègre une fonctionnalité de détection des menaces avancée via certains services, comme Azure Active Directory (Azure AD), les journaux Azure Monitor et Azure Security Center. Cet ensemble de fonctionnalités et de services de sécurité fournit un moyen simple et rapide de comprendre ce qui se passe dans vos déploiements Azure.

Azure propose un large choix d’options pour configurer et personnaliser la sécurité afin de satisfaire les besoins de vos déploiements d’applications. Cet article mentionne comment répondre à ces exigences.

## <a name="azure-active-directory-identity-protection"></a>Azure Active Directory Identity Protection

[Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md) est une fonctionnalité de l'édition [Azure AD Premium P2](../../active-directory/fundamentals/active-directory-whatis.md) qui offre une vue d'ensemble des détections à risque et des vulnérabilités potentielles auxquelles sont exposées les identités de votre organisation. Identity Protection tire parti des fonctionnalités existantes de détection des anomalies d'Azure AD (disponibles via les [rapports d'activités anormales d'Azure AD](../../active-directory/reports-monitoring/overview-reports.md)) et introduit de nouveaux types de détections à risque capables de repérer les anomalies en temps réel.

![Diagramme Azure AD Identity Protection](./media/threat-detection/azure-threat-detection-fig1.png)

Identity Protection s'appuie sur des algorithmes d'apprentissage automatique adaptatif et des règles heuristiques pour détecter les anomalies et les détections à risque susceptibles d'indiquer qu'une identité a été compromise. À l'aide de ces données, Identity Protection génère des rapports et des alertes qui vous permettent d'analyser ces détections à risque et de prendre les mesures de correction ou d'atténuation qui s'imposent.

Mais Azure Active Directory Identity Protection est bien plus qu’un outil de surveillance et de création de rapports. Sur la base des détections à risque, Identity Protection calcule le niveau de risque pour chaque utilisateur, ce qui vous permet de configurer des stratégies basées sur les risques afin de protéger automatiquement les identités de votre organisation.

Ces stratégies basées sur les risques, en plus des autres [contrôles d’accès conditionnel](../../active-directory/conditional-access/overview.md) fournis par Azure Active Directory et [EMS](../../active-directory/conditional-access/overview.md), peuvent automatiquement bloquer l’accès ou appliquer des mesures de correction adaptatives qui incluent des réinitialisations de mot de passe et la mise en œuvre de l’authentification multifacteur.

### <a name="identity-protection-capabilities"></a>Fonctionnalités d’Identity Protection

Mais Azure Active Directory Identity Protection est bien plus qu’un outil de surveillance et de création de rapports. Pour protéger les identités de votre organisation, vous pouvez configurer des stratégies qui répondent automatiquement aux problèmes détectés lorsqu’un niveau de risque spécifié est atteint. Outre les autres contrôles d’accès conditionnel fournis par Azure Active Directory et EMS, ces stratégies peuvent automatiquement bloquer ou déclencher des mesures de correction adaptatives qui incluent des réinitialisations de mot de passe et la mise en œuvre de l’authentification multifacteur.

Exemples de méthodes qu’utilise Azure Identity Protection pour vous aider à sécuriser vos comptes et identités :

[Repérage des détections et des comptes à risque](../../active-directory/identity-protection/overview-identity-protection.md)
-   Repérez six types de détections à risque à l'aide de l'apprentissage automatique et des règles heuristiques.
-   Calculez le niveau de risque des utilisateurs.
-   Fournissez des recommandations personnalisées visant à améliorer la posture de sécurité globale en mettant en évidence les vulnérabilités.

[Examen des détections à risque](../../active-directory/identity-protection/overview-identity-protection.md)
-   Envoyez des notifications pour les détections à risque.
-   Examinez des détections à risque à l'aide d'informations contextuelles et pertinentes.
-   Fournissez des workflows de base pour le suivi des investigations.
-   Fournissez un accès rapide à des mesures de correction telles que la réinitialisation de mot de passe.

[Stratégies d’accès conditionnel en fonction des risques](../../active-directory/identity-protection/overview-identity-protection.md)
-   Atténuez les connexions à risque en bloquant les connexions ou en imposant des demandes d’authentification multifacteur.
-   Bloquez ou sécurisez les comptes d’utilisateurs à risque.
-   Exigez que les utilisateurs s’inscrivent à l’authentification multifacteur.

### <a name="azure-ad-privileged-identity-management"></a>Azure AD Privileged Identity Management

Avec [Azure Active Directory Privileged Identity Management (PIM)](../../active-directory/privileged-identity-management/pim-configure.md), vous pouvez gérer, contrôler et surveiller l’accès au sein de votre organisation. Cette fonctionnalité inclut l'accès aux ressources d'Azure AD et d'autres services en ligne Microsoft comme Microsoft 365 ou Microsoft Intune.

![Diagramme Azure AD Privileged Identity Management](./media/threat-detection/azure-threat-detection-fig2.png)

PIM vous permet de :

-   Recevoir une alerte et un rapport sur les administrateurs Azure AD, et de bénéficier d'un accès administratif « juste à temps » (JIT) aux services Microsoft Online Services comme Microsoft 365 et Intune.

-   Obtenir des rapports sur l'historique des accès administrateur et sur les modifications apportées aux affectations de l'administrateur.

-   Recevoir des alertes sur l'accès à un rôle privilégié.

## <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Les [journaux Azure Monitor](../../azure-monitor/index.yml) sont une solution Microsoft de gestion informatique basée sur le cloud qui vous permet de gérer et protéger votre infrastructure locale et cloud. Les journaux Azure Monitor étant implémentés sous la forme d’un service informatique, ils peuvent être opérationnels rapidement, avec un investissement minimal en services d’infrastructure. Les nouvelles fonctionnalités de sécurité sont fournies automatiquement, ce qui vous permet d’économiser sur les coûts de mise à niveau et de maintenance.

En plus de fournir de précieux services de manière autonome, les journaux Azure Monitor peuvent s’intégrer à des composants System Center tels que [System Center Operations Manager](/archive/blogs/cbernier/monitoring-windows-azure-with-system-center-operations-manager-2012-get-me-started), afin d’étendre dans le cloud vos investissements existants en matière de gestion de la sécurité. System Center et les journaux Azure Monitor peuvent fonctionner ensemble pour fournir une expérience de gestion hybride.

### <a name="holistic-security-and-compliance-posture"></a>Approche globale de la sécurité et de la conformité

Le [tableau de bord Log Analytics Security and Audit](../../security-center/security-center-introduction.md) offre une vue détaillée de la sécurité informatique de votre organisation. Elle fournit des requêtes de recherche intégrées pour détecter les problèmes importants qui requièrent votre attention. Le tableau de bord Security and Audit est l’écran d’accueil pour tout ce qui se rapporte à la sécurité dans les journaux Azure Monitor. Il fournit un aperçu global de l’état de sécurité de vos ordinateurs. Vous pouvez également voir tous les événements des dernières 24 heures, des 7 derniers jours ou de n’importe quel intervalle de temps personnalisé.

Les journaux Azure Monitor vous permettent de comprendre rapidement et facilement les conditions de sécurité globales de n’importe quel environnement, dans le contexte des opérations informatiques, notamment : évaluation des mises à jour logicielles, évaluation des logiciels anti-programmes malveillants et référentiels de configuration. Les données des journaux de sécurité sont facilement accessibles afin de rationaliser les processus d’audit de sécurité et de conformité.

![Tableau de bord Log Analytics Security and Audit](./media/threat-detection/azure-threat-detection-fig3.jpg)

Le tableau de bord Log Analytics Security and Audit est organisé en quatre catégories principales :

-   **Domaines de sécurité**  : vous permet d’explorer plus en détail les enregistrements de sécurité au fil du temps, d’accéder à l’évaluation des programmes malveillants, de mettre à jour les évaluations, d’afficher la sécurité du réseau ainsi que les informations d’identité et d’accès, de consulter les ordinateurs présentant des événements de sécurité et d’accéder rapidement au tableau de bord d’Azure Security Center.

-   **Problèmes importants**  : vous permet d’identifier rapidement le nombre de problèmes actifs et leur gravité.

-   **Détections (préversion)**  : vous permet d’identifier les modèles d’attaques en affichant les alertes de sécurité au fur et à mesure qu’elles affectent vos ressources.

-   **Informations sur les menaces**  : vous permet d’identifier les modèles d’attaques en affichant le nombre total de serveurs présentant un trafic IP sortant malveillant, le type de menace malveillante et une carte indiquant l’emplacement de ces adresses IP.

-   **Requêtes de sécurité courantes**  : liste les requêtes de sécurité les plus courantes que vous pouvez utiliser pour superviser votre environnement. Lorsque vous sélectionnez une requête, le volet Recherche s’ouvre et affiche les résultats de cette requête.

### <a name="insight-and-analytics"></a>Avis et analyses
Au cœur des [journaux Azure Monitor](../../azure-monitor/log-query/log-query-overview.md) se trouve le référentiel qui est hébergé par Azure.

![Diagramme Insight et Analytics](./media/threat-detection/azure-threat-detection-fig4.png)

Vous collectez des données dans le référentiel à partir de sources connectées en configurant des sources de données et en ajoutant des solutions à votre abonnement.

![Tableau de bord des journaux Azure Monitor](./media/threat-detection/azure-threat-detection-fig5.png)

Les sources de données et les solutions créent chacune différents types d'enregistrements avec leur propre jeu de propriétés, mais vous pouvez toujours les analyser ensemble dans des requêtes vers le référentiel. Vous pouvez utiliser les mêmes outils et méthodes pour utiliser diverses données collectées par différentes sources.


La plupart de vos interactions avec les journaux Azure Monitor s’effectuent via le portail Azure qui s’exécute dans un navigateur et vous fournit un accès aux paramètres de configuration et à plusieurs outils pour analyser les données collectées et agir sur celles-ci. À partir du portail, vous pouvez utiliser :
* [Recherche dans les journaux](../../azure-monitor/log-query/log-query-overview.md) où vous avez élaboré des requêtes pour analyser des données collectées.
* [Tableaux de bord](../../azure-monitor/learn/tutorial-logs-dashboards.md), que vous pouvez personnaliser avec des vues graphiques de vos recherches les plus précieuses.
* [Solutions](../../azure-monitor/insights/solutions.md), qui fournissent des outils d’analyse et des fonctionnalités supplémentaires.

![Outils d’analyse](./media/threat-detection/azure-threat-detection-fig6.png)

Les solutions permettent d’ajouter des fonctionnalités aux journaux Azure Monitor. Elles s’exécutent principalement dans le cloud et fournissent une analyse des données collectées dans le référentiel Log Analytics. Ces solutions peuvent également définir de nouveaux types d’enregistrements à collecter qui peuvent être analysés avec des recherches de journaux ou via une interface utilisateur supplémentaire fournie par la solution dans le tableau de bord Log Analytics.

Le tableau de bord Sécurité et Audit est un exemple de ces types de solutions.

### <a name="automation-and-control-alert-on-security-configuration-drifts"></a>Automation and Control : alerte sur les anomalies de configuration de la sécurité

Azure Automation automatise les processus administratifs grâce à des runbooks basés sur PowerShell et exécutés dans le cloud. Les Runbooks peuvent également être exécutés sur un serveur de votre centre de données local pour gérer des ressources locales. Azure Automation assure la gestion de la configuration avec PowerShell DSC (Desired State Configuration).

![Diagramme Azure Automation](./media/threat-detection/azure-threat-detection-fig7.png)

Vous pouvez créer et gérer les ressources DSC hébergées dans Azure et les appliquer à des systèmes de cloud et locaux. Ce faisant, vous pouvez définir et appliquez automatiquement leur configuration ou obtenir des rapports pour aider à s’assurer que les configurations de sécurité restent au sein de la stratégie.

## <a name="azure-security-center"></a>Azure Security Center

Azure Security Center vous permet de protéger vos ressources Azure. Il fournit des fonctions intégrées de surveillance de la sécurité et de gestion des stratégies sur vos abonnements Azure. Dans le service, vous pouvez définir des stratégies contre vos abonnements Azure et vos [groupes de ressources](../../azure-resource-manager/management/manage-resources-portal.md) pour une granularité supérieure.

![Diagramme Azure Security Center](./media/threat-detection/azure-threat-detection-fig8.png)

Les chercheurs en sécurité de Microsoft sont constamment à l’affût des nouvelles menaces. Ils ont accès à un vaste jeu de télémétrie acquis grâce à la présence globale de Microsoft sur le cloud et localement. Cette collection diverse et étendue de jeux de données permet à Microsoft de détecter de nouveaux modèles et de nouvelles tendances d’attaques dans ses produits locaux destinés au consommateur et aux entreprises, ainsi que dans ses services en ligne.

Azure Security Center peut donc rapidement mettre à jour ses algorithmes de détection, puisque les pirates sont à l’origine d’attaques innovantes de plus en plus sophistiquées. Cette approche permet de faire face à des menaces en pleine mutation.

![Centre de sécurité - Détection de menaces](./media/threat-detection/azure-threat-detection-fig9.jpg)

La détection des menaces d’Azure Security Center fonctionne en collectant automatiquement les informations de sécurité à partir de vos ressources Azure, du réseau et des solutions de partenaires connectées. Elle analyse ces informations, en corrélant les données issues de plusieurs sources, pour identifier les menaces.

Les alertes de sécurité, ainsi que les recommandations sur la façon de répondre à la menace, sont hiérarchisées dans Azure Security Center.

Azure Security Center emploie des analyses de sécurité avancées allant bien au-delà des approches simplement basées sur la signature. Les avancées des technologies de big data et d’[apprentissage automatique](https://azure.microsoft.com/blog/machine-learning-in-azure-security-center/) sont utilisées pour évaluer les événements de toute la structure fabric cloud. Les fonctions d’analyse avancées peuvent détecter les menaces qui seraient impossibles à identifier avec des approches manuelles et peuvent prévoir l’évolution des attaques. Ces types d’analyses de sécurité sont mentionnés dans les prochaines sections.

### <a name="threat-intelligence"></a>Informations sur les menaces

Microsoft dispose d’une multitude d’informations en matière de menaces à l’échelle mondiale.

La télémétrie provient de plusieurs sources, telles qu’Azure, Microsoft 365, Microsoft CRM Online, Microsoft Dynamics AX, outlook.com, MSN.com, Microsoft Digital Crimes Unit (DCU) et Microsoft Security Response Center (MSRC).

![Découvertes d’informations sur les menaces](./media/threat-detection/azure-threat-detection-fig10.jpg)

Les chercheurs reçoivent également les informations sur les menaces partagées par les principaux fournisseurs de services cloud et s’abonnent aux flux d’informations sur les menaces provenant de tiers. Azure Security Center peut utiliser ces informations pour vous alerter en cas de menaces provenant d’éléments malveillants connus. Voici quelques exemples :

-   **Exploitation de la puissance du machine learning**  : Azure Security Center a accès à une grande quantité de données sur l’activité réseau cloud, qui peuvent être utilisées pour détecter les menaces ciblant vos déploiements Azure.

-   **Détection des attaques par force brute**  : Le machine learning permet de créer un modèle historique des tentatives d’accès à distance, qui facilite la détection des attaques par force brute sur les ports SQL, RDP (Remote Desktop Protocole) et SSH (Secure Shell).

-   **Détection des attaques DDoS sortantes et des botnets**  : les attaques ciblant les ressources cloud visent généralement à utiliser la puissance de calcul de ces ressources pour exécuter d’autres attaques.

-   **Nouveaux serveurs d’analyse comportementale et machines virtuelles**  : dès lors qu’un serveur ou qu’une machine virtuelle est attaqué(e), les pirates utilisent diverses techniques pour exécuter du code malveillant sur le système ciblé, en évitant la détection, en assurant la persistance des attaques et en contournant les contrôles de sécurité.

-   **Détection des menaces sur Azure SQL Database**  : la détection des menaces pour Azure SQL Database identifie les activités de base de données anormales qui indiquent les tentatives inhabituelles et potentiellement dangereuses d’accès ou d’exploitation des bases de données.

### <a name="behavioral-analytics"></a>Analyse comportementale

L’analyse comportementale est une technique qui analyse et compare les données à une collection de modèles connus. Toutefois, ces modèles ne sont pas de simples signatures. Ils sont déterminés par le biais d’algorithmes d’apprentissage automatique appliqués aux ensembles de données massifs.

![Découvrtes sur l’analyse comportementale](./media/threat-detection/azure-threat-detection-fig11.jpg)

Ils sont également déterminés à travers une analyse minutieuse des comportements malveillants par des experts. Azure Security Center peut utiliser l’analyse comportementale pour identifier les ressources compromises en se basant sur l’analyse des journaux d’activité de la machine virtuelle, des journaux d’activité du périphérique réseau virtuel, des journaux d’activité Service Fabric, des vidages sur incident et d’autres sources.

En outre, il existe une corrélation entre les modèles et les autres signaux pour rechercher les preuves d’une campagne généralisée. Ce rapprochement permet d’identifier les événements qui sont cohérents avec les indicateurs de compromission établis.

Voici quelques exemples :
-   **Exécution de processus suspect** : les attaquants utilisent diverses techniques pour exécuter des logiciels malveillants en l’absence de détection. Par exemple, un pirate peut donner au programme malveillant le même nom que celui utilisé pour des fichiers système légitimes, mais placer ces fichiers à un autre emplacement, utiliser un nom similaire à un fichier anodin ou masquer la véritable extension du fichier. Security Center modélise les comportements et surveille les exécutions du processus pour détecter les valeurs aberrantes telles que celles-ci.

-   **Programmes malveillants masqués et tentatives d’exploitation** : des programmes malveillants sophistiqués peuvent échapper aux produits anti-programme malveillant traditionnels en choisissant de ne jamais écrire sur le disque ou en chiffrant des composants logiciels stockés sur le disque. Toutefois, ces logiciels malveillants peuvent être détectés à l’aide de l’analyse de mémoire, car le programme malveillant laisse des traces en mémoire pour pouvoir fonctionner. Lorsque le logiciel se bloque, un vidage sur incident capture une partie de la mémoire au moment de l’incident. En analysant la mémoire dans le vidage sur incident, Azure Security Center peut détecter les techniques utilisées pour exploiter la vulnérabilité des logiciels, accéder aux données confidentielles et persister subrepticement au sein d’un ordinateur compromis sans affecter les performances de votre machine.

-   **Mouvement latéral et reconnaissance interne** : afin de persister dans un réseau compromis et de localiser/récolter des données précieuses, les attaquants tentent souvent de se déplacer latéralement à partir de l’ordinateur compromis vers d’autres ordinateurs au sein du même réseau. Azure Security Center surveille les activités de processus et de connexion afin de détecter les tentatives de développement du pirate au sein du réseau, telles que l’exécution de commande à distance, la détection de réseau et l’énumération de compte.

-   **Scripts PowerShell malveillants**  : les pirates peuvent utiliser PowerShell pour exécuter du code malveillant sur des machines virtuelles cibles à des fins diverses. Azure Security Center inspecte l’activité PowerShell à la recherche d’activité suspecte.

-   **Attaques sortantes** : les pirates ciblent souvent les ressources cloud en vue d’utiliser ces ressources pour lancer d’autres attaques. Les machines virtuelles compromises peuvent, par exemple, servir à lancer des attaques par force brute contre d’autres machines virtuelles, envoyer du courrier indésirable, ou analyser les ports ouverts et autres appareils sur Internet. En appliquant l’apprentissage automatique au trafic réseau, Azure Security Center peut détecter lorsque les communications réseau sortantes dépassent la norme. Lorsque du courrier indésirable est détecté, Azure Security Center met également en corrélation le trafic de messagerie inhabituel avec l'intelligence issue de Microsoft 365 pour déterminer si le courrier est susceptible d'être mal intentionné ou est le résultat d'une campagne de courrier électronique légitime.

### <a name="anomaly-detection"></a>Détection des anomalies

Azure Security Center utilise également la détection des anomalies pour identifier les menaces. Contrairement à l’analyse comportementale (qui dépend des modèles connus dérivés de grands jeux de données), la détection des anomalies est plus « personnalisée » et se concentre sur les lignes de base propres à vos déploiements. L’apprentissage automatique est appliqué pour déterminer l’activité normale de vos déploiements et les règles sont générées pour définir les conditions de valeurs aberrantes pouvant signaler un événement de sécurité. Voici un exemple :

-   **Attaques par force brute RDP/SSH entrantes** : vos déploiements peuvent comporter des machines virtuelles occupées par un nombre plus ou moins important de connexions quotidiennes. Azure Security Center peut déterminer l’activité de connexion de base de ces machines virtuelles et utiliser l’apprentissage automatique pour définir ce qui correspond à des activités normales. Une alerte peut être générée en cas d’écart par rapport à la référence définie pour les caractéristiques de connexion. Là encore, l’apprentissage automatique détermine ce qui est significatif.

### <a name="continuous-threat-intelligence-monitoring"></a>Analyse continue des informations sur les menaces

Azure Security Center s’appuie sur des équipes de recherche de sécurité et de sciences des données implantées dans le monde entier, qui surveillent en continu les évolutions en matière de menaces. Cela inclut les initiatives suivantes :

-   **Analyse des informations sur les menaces** : les informations sur les menaces incluent des mécanismes, des indicateurs, des implications et des conseils pratiques sur les menaces, nouvelles ou existantes. Ces informations sont partagées avec la communauté dédiée à la sécurité, et Microsoft surveille en permanence le flux des informations sur les menaces provenant de sources internes et externes.

-   **Partage de signal** : les insights fournis par les équipes de sécurité sur le portefeuille complet de services cloud et locaux, de serveurs et d’appareils de point de terminaison client de Microsoft sont partagés et analysés.

-   **Spécialistes de la sécurité Microsoft** : engagement continu avec les équipes Microsoft travaillant dans des domaines de la sécurité spécialisés, tels que la forensique et la détection d’attaques web.

-   **Réglage de la détection** : des algorithmes sont exécutés sur les jeux de données client réels, et les chercheurs en sécurité collaborent avec les clients pour valider les résultats. Les vrais et les faux positifs sont utilisés pour affiner les algorithmes d’apprentissage automatique.

Ces efforts combinés aboutissent à des détections nouvelles et améliorées, dont vous pouvez bénéficier instantanément. Aucune action de votre part n’est requise.

## <a name="advanced-threat-detection-features-other-azure-services"></a>Fonctionnalités de détection de menaces avancées : Autres services Azure

### <a name="virtual-machines-microsoft-antimalware"></a>Machines virtuelles : Microsoft Antimalware

[Microsoft Antimalware](antimalware.md) est une solution d’agent unique pour les applications et les environnements client, conçue pour s’exécuter en arrière-plan sans intervention humaine. Vous pouvez déployer la protection en fonction des besoins de vos charges de travail d’application, avec une configuration de base sécurisée par défaut ou une configuration personnalisée avancée, y compris pour la surveillance anti-programmes malveillants. Azure Antimalware est une option de sécurité dédiée aux machines virtuelles Azure qui est automatiquement installée sur toutes les machines virtuelles PaaS Azure.

#### <a name="microsoft-antimalware-core-features"></a>Principales fonctionnalités de Microsoft Antimalware

Voici les fonctionnalités d’Azure permettant de déployer et activer Microsoft Antimalware pour vos applications :

-   **Protection en temps réel** : supervise l’activité dans les services cloud et sur les machines virtuelles pour détecter et bloquer l’exécution de logiciels malveillants.

-   **Analyse planifiée** : Effectue périodiquement une analyse ciblée pour détecter les logiciels malveillants, notamment les programmes en cours d’exécution.

-   **Correction de logiciels malveillants** : prend automatiquement des mesures sur les programmes malveillants détectés, notamment la suppression ou la mise en quarantaine des fichiers malveillants et le nettoyage des entrées de Registre malveillantes.

-   **Mises à jour de signatures** : installe automatiquement les dernières signatures de protection (définitions de virus) pour garantir que la protection est à jour d’après une fréquence prédéfinie.

-   **Mises à jour du moteur Antimalware**  : met automatiquement à jour le moteur Microsoft Antimalware.

-   **Mises à jour du logiciel anti-programme malveillant pour la plateforme** : met automatiquement à jour la plateforme Microsoft Antimalware.

-   **Protection active** : signale les métadonnées de télémétrie relatives aux menaces détectées et aux ressources suspectes à Microsoft Azure afin d’assurer une réaction rapide au paysage de menaces en constante évolution, en permettant de fournir une signature synchrone en temps réel via le système MAPS (Microsoft Active Protection System).

-   **Exemples de création de rapport** : crée et fournit des exemples de rapports au service Microsoft Antimalware afin de contribuer à l’amélioration du service et permettre la résolution des problèmes.

-   **Exclusions** : permet aux administrateurs d’applications et de services de configurer certains fichiers, processus et lecteurs pour les exclure de la protection et de l’analyse, entre autres pour des raisons de performances.

-   **Collecte d’événements du logiciel anti-programme malveillant** : enregistre l’intégrité du service Antimalware, les activités suspectes et les mesures de correction prises dans le journal des événements du système d’exploitation et les rassemble dans le compte de stockage Azure du client.

### <a name="azure-sql-database-threat-detection"></a>Détection des menaces Azure SQL Database

[Détection des menaces Azure SQL Database](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/) est une nouvelle fonctionnalité de sécurité intelligence intégrée dans le service Azure SQL Database. Conçue pour identifier, profiler et détecter 24 heures sur 24 les activités anormales sur la base de données, la détection des menaces Azure SQL Database identifie les menaces potentielles pour la base de données.

Les responsables de la sécurité ou autres administrateurs désignés peuvent obtenir une notification immédiate concernant les activités suspectes qui interviennent sur la base de données. Chaque notification contient des détails sur l’activité suspecte et fournit des recommandations pour vous aider à étudier et corriger la menace.

La détection des menaces Azure SQL Database détecte actuellement les vulnérabilités potentielles et les attaque par injection SQL, ainsi que les schémas d’accès anormaux à la base de données.

Lorsqu’ils reçoivent une notification de détection des menaces par e-mail, les utilisateurs sont en mesure d’accéder aux enregistrements d’audit concernés et de les consulter via un lien profond dans l’e-mail. Le lien ouvre une visionneuse d’audit ou un modèle Excel d’audit préconfiguré qui affiche les enregistrements d’audit concernés au moment de l’événement suspect, selon :

-   Stockage d’audit du serveur/de la base de données, retraçant les activités anormales sur la base de données

-   Table de stockage d’audit pertinente ayant été utilisée au moment de l’événement pour écrire le journal d’audit.

-   Enregistrements d’audit de l’heure suivant immédiatement l’événement.

-   Enregistrements d’audit associés à un ID d’événement similaire au moment de l’événement (facultatif pour certains détecteurs).

Les fonctions SQL Database Threat Detector utilisent les méthodes de détection suivantes :

-   **Détection déterministe**  : détecte les séquences suspectes (à partir de règles) dans les requêtes clientes SQL qui correspondent à des attaques connues. Cette méthodologie offre une haute capacité de détection et génère peu de faux positifs ; elle présente toutefois une couverture assez limitée puisqu’elle relève de la catégorie des « détections atomiques ».

-   **Détection comportementale**  : détecte les activités anormales, autrement dit les comportements anormaux de la base de données qui n’ont pas été observés au cours des 30 derniers jours. Dans un client SQL, une activité anormale peut, par exemple, se traduire par un pic d’échecs de connexion/requête, par un volume important de données extraites, par des requêtes canoniques inhabituelles ou encore par l’utilisation d’adresses IP inconnues pour accéder à la base de données

### <a name="application-gateway-web-application-firewall"></a>Pare-feu d’applications web sur Application Gateway

Le [pare-feu d’applications web (WAF)](../../app-service/environment/app-service-app-service-environment-web-application-firewall.md) est une fonction de [passerelle Azure Application Gateway](../../web-application-firewall/ag/ag-overview.md) qui offre une protection pour les applications web qui utilisent la passerelle d’application pour les fonctions de [contrôle de remise d’application](https://kemptechnologies.com/in/application-delivery-controllers) standard. Le pare-feu d’applications web le fait en les protégeant contre la plupart des [10 plus courantes vulnérabilités web de l’OWASP (Open Web Application Security Project](https://owasp.org/www-project-top-ten/).

![Diagramme du Pare-feu d’applications web sur Application Gateway](./media/threat-detection/azure-threat-detection-fig13.png)

Les protections sont les suivantes :

-   Protection contre les injections de code SQL.

-   Protection de l’exécution de script de site à site.

-   Protection contre les attaques web courantes comme l’injection de commande, les dissimulations de requêtes HTTP, la séparation de réponse HTTP et les attaques RFI (Remote File Inclusion).

-   Protection contre les violations de protocole HTTP.

-   Protection contre les anomalies de protocole HTTP comme un agent-utilisateur hôte manquant et les en-têtes Accept.

-   Protection contre les robots, les crawlers et les scanneurs.

-   Détection des erreurs de configuration d’application courantes (par exemple, Apache, IIS, etc.)

La configuration WAF au niveau de votre passerelle d’application vous offre plusieurs avantages :

-   Protection de votre application web contre les vulnérabilités et les attaques web sans modification du code principal.

-   Protection simultanée de plusieurs applications web derrière une passerelle d’application. Prise en charge d’une passerelle d’application qui peut héberger jusqu’à 20 sites web.

-   Supervision des applications web contre les attaques à l’aide de rapports en temps réel générés par les journaux d’activité WAF de la passerelle d’application.

-   Aide au respect des exigences en matière de conformité. Certains contrôles de conformité nécessitent que tous les points de terminaison qui accèdent à Internet soient protégés par une solution WAF.

### <a name="anomaly-detection-api-built-with-azure-machine-learning"></a>API Détection d’anomalies : intégrée à Azure Machine Learning

L’API Anomaly Detection est une API très utile pour détecter une variété de modèles d’anomalies dans les données de la série chronologique. L’API attribue un score d’anomalie à chaque point de données de la série chronologique, qui peut être utilisé pour générer des alertes, en établissant une surveillance via les tableaux de bord ou en créant une connexion avec vos systèmes de gestion de tickets.

[L’API Anomaly Detection](../../machine-learning/team-data-science-process/apps-anomaly-detection-api.md) peut détecter les types suivants de schémas anormaux dans les données de séries chronologiques :

-   **Pics et creux**  : quand vous supervisez le nombre d’échecs de connexion pour un service ou le nombre de validations dans un site de commerce électronique, les pics ou creux d’activité inhabituels peuvent indiquer des attaques de sécurité ou des interruptions de service.

-   **Tendances positives et négatives**  : quand vous supervisez l’utilisation de la mémoire en informatique, une taille de mémoire gratuite qui diminue indique une fuite de mémoire potentielle. Pour la surveillance de la longueur d’une file d'attente d’un service, une tendance montante persistante pourrait indiquer un problème logiciel sous-jacent.

-   **Changements de niveau et changements dans la plage dynamique de valeurs**  : il peut être intéressant de superviser les changements de niveau de latence d’un service après une mise à niveau du service, ou encore une réduction des niveaux d’exceptions après une mise à niveau.

L’API de Machine Learning offre les avantages suivants :

-   **Détection flexible et fiable**  : les modèles de détection d’anomalies permettent aux utilisateurs de configurer les paramètres de sensibilité et de détecter des anomalies entre les jeux de données saisonnières et non saisonnières. Les utilisateurs peuvent ajuster le modèle de détection d’anomalies pour rendre l’API de détection plus ou moins sensible en fonction de leurs besoins. Cela permet de détecter les anomalies plus ou moins visibles au niveau des données, avec et sans schémas saisonniers.

-   **Détection scalable et rapide**  : l’approche de supervision traditionnelle, qui repose sur les seuils actuels définis à partir des connaissances d’experts dans le domaine, est coûteuse et ne peut pas être déployée sur des millions de jeux de données qui évoluent de façon dynamique. Les modèles de détection d’anomalies proposés dans cette API s’appuient sur un apprentissage et sont paramétrés automatiquement à partir des données historiques et en temps réel.

-   **Détection proactive et exploitable**  : il est possible d’appliquer une détection des lents changements de tendance et de niveau afin d’anticiper les anomalies. Les équipes peuvent utiliser les signaux anormaux rapidement détectés pour étudier et agir sur les aspects problématiques. En outre, il est possible de développer des modèles d’analyse des causes ainsi que des outils d’alerte sur ce service d’API de détection des anomalies.

L’API de détection des anomalies est une solution efficace pour un large éventail de scénarios, notamment la surveillance de l’intégrité du service et des indicateurs de performance clés, l’IoT, l’analyse des performances et l’analyse du trafic réseau. Voici quelques scénarios courants où cette API peut se révéler utile :

- Les services informatiques ont besoin d’outils pour suivre les événements, les codes d’erreur, les journaux d’utilisation et les performances (processeur, mémoire, etc.) en temps voulu.

-   Les sites de commerce en ligne ont besoin d’effectuer un suivi des activités du client, des pages consultées, du nombre de clics, etc.

-   Les entreprises de services publics, quant à elles, ont besoin d’effectuer un suivi de la consommation d’eau, de gaz, d’électricité et d’autres ressources.

-   Les services de gestion d’installations ou de bâtiments doivent surveiller la température, l’humidité, le trafic, etc.

-   Les entreprises IoT/fabricants veulent utiliser les données de capteur dans des séries chronologiques pour analyser le flux de travail, la qualité, etc.

-   Les fournisseurs de services, tels que les centres d’appel, doivent analyser les tendances de demande de service, le volume d’incidents, la durée de la file d’attente, etc.

-   Les groupes d’analyse marketing doivent surveiller en temps réel les changements anormaux des KPI commerciaux (par exemple, le volume des ventes, les sentiments du client, les prix).

### <a name="cloud-app-security"></a>Cloud App Security

[Cloud App Security](/cloud-app-security/what-is-cloud-app-security) est un composant essentiel de la pile de sécurité de Microsoft Cloud. Cette solution complète peut aider votre organisation lorsque vous cherchez à tirer pleinement parti des promesses des applications cloud. Elle vous laisse le contrôle grâce à une meilleure visibilité dans l’activité. Elle vous permet également de renforcer la protection des données critiques dans les applications cloud.

Grâce à des outils qui vous permettent de détecter le Shadow IT, d’évaluer les risques, d’appliquer des stratégies, d’examiner les activités et d’arrêter les menaces, votre organisation est en mesure de migrer en toute sécurité vers le cloud tout en conservant le contrôle sur ses données critiques.

| Category | Description |
| -------- | ----------- |
| Découvrez | Découvrez le Shadow IT grâce à Cloud App Security. Bénéficiez d’une meilleure visibilité grâce à une détection des applications, des activités, des utilisateurs, des données et des fichiers dans votre environnement cloud. Détectez les applications tierces connectées à votre cloud.|
|Examiner | Examinez vos applications cloud à l’aide d’outils d’investigation cloud conçus pour explorer en profondeur les applications à risque, les utilisateurs spécifiques et les fichiers de votre réseau. Recherchez des modèles parmi les données collectées à partir de votre cloud. Générez des rapports pour surveiller votre cloud. |
| Control | Limitez les risques en définissant des stratégies et des alertes de manière à accentuer le contrôle sur le trafic réseau cloud. Utilisez Cloud App Security pour migrer vos utilisateurs vers d’autres solutions cloud sécurisées et approuvées. |
| Protéger | Utilisez Cloud App Security pour approuver ou interdire des applications, appliquer des mesures de prévention des fuites de données, contrôler les autorisations et le partage, et générer des alertes et des rapports personnalisés. |
| Control | Limitez les risques en définissant des stratégies et des alertes de manière à accentuer le contrôle sur le trafic réseau cloud. Utilisez Cloud App Security pour migrer vos utilisateurs vers d’autres solutions cloud sécurisées et approuvées. |


![Diagramme Cloud App Security](./media/threat-detection/azure-threat-detection-fig14.png)

Cloud App Security intègre la visibilité à votre cloud :

-   Utilisation de Cloud Discovery pour mapper et identifier votre environnement cloud et les applications cloud utilisées par votre organisation.

-   en approuvant et en interdisant les applications dans votre cloud ;

-   en utilisant des connecteurs faciles à déployer qui tirent parti des API de fournisseurs pour améliorer la visibilité et la gouvernance des applications auxquelles vous vous connectez ;

-   en vous aidant à préserver le contrôle grâce à la définition et à l’optimisation de stratégies.

Cloud App Security exécute une analyse complexe sur les données recueillies auprès de ces sources. Cette solution vous alerte immédiatement en cas d’activités anormales et vous procure une visibilité totale sur votre environnement cloud. Vous pouvez configurer une stratégie dans Cloud App Security et l’utiliser pour protéger tous les éléments de votre environnement cloud.

## <a name="third-party-advanced-threat-detection-capabilities-through-the-azure-marketplace"></a>Fonctionnalités tierces avancées de détection des menaces via la Place de marché Azure

### <a name="web-application-firewall"></a>Pare-feu d’applications web

Le pare-feu d’applications web inspecte le trafic web entrant et bloque les injections SQL, les attaques XSS, les téléchargements de programmes malveillants, les attaques DDoS, ainsi que les autres attaques ciblées sur vos applications web. Ce type de pare-feu inspecte également les réponses des serveurs Web principaux pour prévention de perte de données (DLP). Le moteur de contrôle d’accès intégré permet aux administrateurs de créer des stratégies de contrôle d’accès granulaire pour l’authentification, l’autorisation et la traçabilité (AAA), qui offre aux organisations une authentification renforcée et un meilleur contrôle des utilisateurs.

Le pare-feu d’applications Web offre les avantages suivants :

-   Détecte et bloque les injections SQL, les attaques XSS, les téléchargements de logiciels malveillants, les attaques DDoS d’application ou toute autre attaque dirigée contre votre application.

-   Authentification et contrôle d’accès.

-   Analyse le trafic sortant pour détecter les données sensibles, avec la possibilité de masquer ou bloquer la fuite d’informations.

-   Accélère la distribution de contenus d’application web, à l’aide de fonctionnalités telles que la mise en cache, la compression et d’autres optimisations du trafic.

Pour obtenir des exemples de pare-feu d’applications web disponibles sur la Place de marché Azure, consultez [Pare-feu d'applications web Barracuda, pare-feu d'applications web virtuel Brocade (vWAF), Imperva SecureSphere et le pare-feu ThreatSTOP IP](https://azuremarketplace.microsoft.com/marketplace/apps/barracudanetworks.waf).

## <a name="next-steps"></a>Étapes suivantes

- [Répondre aux menaces actuelles](../../security-center/security-center-alerts-overview.md#respond-threats) : elles permettent d’identifier les menaces actives ciblant vos ressources Azure et fournissent les insights nécessaires pour y répondre rapidement.

- [Détection des menaces sur Azure SQL Database](https://azure.microsoft.com/blog/azure-sql-database-threat-detection-your-built-in-security-expert/) : permet d’apaiser vos inquiétudes concernant les menaces potentielles qui pèsent sur vos bases de données.