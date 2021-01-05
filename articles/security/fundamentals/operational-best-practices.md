---
title: Meilleures pratiques de sécurité pour vos ressources Azure
titleSuffix: Azure security
description: Cet article fournit un ensemble de meilleures pratiques opérationnelles pour protéger vos données, applications et autres ressources dans Azure.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: tomsh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/06/2019
ms.author: terrylan
ms.openlocfilehash: 86874a60d48ddcbdaca5ae779ad554ee58cc233b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498844"
---
# <a name="azure-operational-security-best-practices"></a>Bonnes pratiques pour Azure Operational Security
Cet article fournit un ensemble de meilleures pratiques opérationnelles pour protéger vos données, applications et autres ressources dans Azure.

Ces meilleures pratiques font l’objet d’un consensus et sont compatibles avec les capacités et fonctionnalités actuelles de la plateforme Azure. Les opinions et technologies évoluent au fil du temps ; cet article est régulièrement mis à jour de manière à tenir compte de ces changements.

## <a name="define-and-deploy-strong-operational-security-practices"></a>Définir et déployer des pratiques de sécurité opérationnelle renforcées
La sécurité opérationnelle Azure fait référence aux services, contrôles et fonctionnalités auxquels les utilisateurs ont accès pour protéger leurs données, leurs applications et d’autres ressources dans Azure. La sécurité opérationnelle Azure repose sur un framework qui intègre les connaissances acquises via des fonctionnalités propres à Microsoft, notamment [Security Development Lifecycle (SDL)](https://www.microsoft.com/sdl), le programme [Centre de réponse aux problèmes de sécurité Microsoft](https://www.microsoft.com/msrc?rtc=1) et une connaissance approfondie du paysage des cybermenaces.

## <a name="manage-and-monitor-user-passwords"></a>Gérer et surveiller des mots de passe utilisateur
Le tableau suivant répertorie certaines meilleures pratiques relatives à la gestion des mots de passe utilisateur :

**Bonne pratique** : assurez-vous d’avoir le niveau de protection par mot de passe approprié dans le cloud.   
**Détail** : suivez les instructions du document [Password Guidance](https://www.microsoft.com/research/publication/password-guidance/) (Instructions relatives aux mots de passe), qui est limité aux utilisateurs des plateformes d’identité Microsoft (comptes Azure Active Directory, Active Directory et Microsoft).

**Bonne pratique** : surveillez les actions suspectes associées à vos comptes d’utilisateur.   
**Détail** : surveillez les [utilisateurs à risque](../../active-directory/identity-protection/overview-identity-protection.md) et les [connexions risquées](../../active-directory/identity-protection/overview-identity-protection.md) à l’aide des rapports de sécurité Azure AD.

**Bonne pratique** : détectez et corrigez automatiquement les mots de passe présentant des risques élevés.   
**Détail** : [Azure Active Directory Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md) est une fonctionnalité de l’édition Azure AD Premium P2 qui vous permet d’effectuer les actions suivantes :

- Détecter des vulnérabilités potentielles qui affectent les identités de votre organisation
- Configurer des réponses automatiques aux actions suspectes détectées qui sont liées aux identités de votre organisation
- Examiner les incidents suspects et prendre les mesures appropriées pour les résoudre

## <a name="receive-incident-notifications-from-microsoft"></a>Recevoir des notifications d’incident de Microsoft
Garantir que l’équipe responsable des opérations de sécurité reçoit des notifications d’incidents Azure de la part de Microsoft. Une notification d’incident permet d’indiquer à votre équipe de sécurité que vous avez des ressources Azure compromises. Cette dernière peut ainsi réagir rapidement et corriger les risques de sécurité éventuels.

Dans le portail d’inscription Azure, vous pouvez vérifier que les informations de contact administrateur incluent des détails qui informent les opérations de sécurité. Les informations de contact correspondent à une adresse électronique et à un numéro de téléphone.

## <a name="organize-azure-subscriptions-into-management-groups"></a>Organiser les abonnements dans des groupes d’administration
Si votre organisation dispose de plusieurs abonnements, vous pouvez avoir besoin d’un moyen de gérer efficacement l’accès, les stratégies et la conformité de ces abonnements. [Les groupes d’administration Azure](../../governance/management-groups/create-management-group-portal.md) fournissent un niveau d’étendue au-delà des abonnements. Vous organisez les abonnements en conteneurs appelés groupes d’administration et vous appliquez vos conditions de gouvernance aux groupes d’administration. Tous les abonnements d’un groupe d’administration héritent automatiquement des conditions appliquées à ce groupe d’administration.

Vous pouvez créer une structure flexible de groupes d’administration et d’abonnements dans un répertoire. Chaque annuaire reçoit un groupe d’administration de niveau supérieur unique appelé groupe d’administration racine. Ce groupe d’administration racine est intégré à la hiérarchie et contient tous les groupes d’administration et abonnements. Il permet d’appliquer des stratégies globales et des attributions de rôles Azure au niveau du répertoire.

Voici quelques meilleures pratiques pour l’utilisation de groupes d’administration :

**Bonne pratique** : assurez-vous que les nouveaux abonnements appliquent des éléments de gouvernance tels que les stratégies et les autorisations à mesure qu’ils sont ajoutés.   
**Détail** : utilisez le groupe d’administration racine pour affecter des éléments de sécurité d’entreprise qui s’appliquent à toutes les ressources Azure. Les stratégies et les autorisations sont des exemples d’éléments.

**Bonne pratique** : alignez les niveaux supérieurs des groupes d’administration avec la stratégie de segmentation pour fournir un point destiné au contrôle et à la cohérence de la stratégie pour chaque segment.   
**Détail** : créez un groupe d’administration unique pour chaque segment sous le groupe d’administration racine. Ne créez pas d’autres groupes d’administration au niveau racine.

**Bonne pratique** : limitez la profondeur du groupe d’administration pour éviter toute confusion qui ralentit les opérations et la sécurité.   
**Détail** : limitez votre hiérarchie à trois niveaux, niveau racine compris.

**Bonne pratique** : sélectionnez soigneusement les éléments à appliquer à toute l’entreprise avec le groupe d’administration racine.   
**Détail** : vérifiez que les éléments de groupe d’administration doivent vraiment s’appliquer à chaque ressource et qu’ils ont un impact faible.

Les candidats parfaits sont :

- Les exigences réglementaires qui ont un impact métier clair (par exemple, les restrictions liées à la souveraineté des données)
- Les exigences avec un impact potentiel négatif proche de zéro pour les opérations, comme la stratégie avec effet d’audit ou les affectations de mission Azure RBAC qui ont été consciencieusement étudiées

**Bonne pratique** : planifiez et testez scrupuleusement toutes les modifications à l’échelle de l’entreprise sur le groupe d’administration racine avant de les appliquer (stratégie, modèle Azure RBAC, etc.).   
**Détail** : les modifications dans le groupe d’administration racine peuvent affecter toutes les ressources sur Azure. Même si elles permettent d’assurer la cohérence au sein de l’entreprise, les erreurs ou une utilisation incorrecte peuvent avoir un impact négatif sur les opérations de production. Testez toutes les modifications apportées au groupe d’administration racine dans un pilote de production ou un laboratoire de test.

## <a name="streamline-environment-creation-with-blueprints"></a>Simplifier la création d’environnement avec des blueprints
Le service [Azure Blueprints](../../governance/blueprints/overview.md) permet aux architectes cloud et aux membres de l’informatique centrale de définir un ensemble reproductible de ressources Azure qui implémentent et respectent les normes, modèles et exigences d’une organisation. Azure Blueprint permet aux équipes de développement de créer et de mettre en place rapidement de nouveaux environnements avec un ensemble de composants intégrés et en ayant la certitude de créer des environnements conformes à l’organisation.

## <a name="monitor-storage-services-for-unexpected-changes-in-behavior"></a>Superviser les services de stockage pour détecter des changements inattendus du comportement
Il peut s’avérer plus complexe de diagnostiquer et de résoudre des problèmes dans une application distribuée hébergée dans un environnement cloud que dans des environnements traditionnels. Les applications peuvent être déployées dans une infrastructure PaaS ou IaaS, localement, sur un appareil mobile ou dans une combinaison de ces environnements. Le trafic réseau de votre application peut parcourir les réseaux publics et privés, et votre application peut utiliser différentes technologies de stockage.

Vous devez superviser en permanence les services de stockage utilisés par votre application afin de détecter tout changement de comportement inattendu (par exemple, des temps de réponse plus longs). Utilisez la journalisation pour collecter des données plus détaillées et analyser un problème en profondeur. Les informations de diagnostic obtenues par le biais de la supervision et de la journalisation vous aident à déterminer la cause première du problème rencontré par votre application. Vous pouvez alors résoudre le problème et déterminer la procédure appropriée pour y remédier.

[Azure Storage Analytics](../../storage/common/storage-analytics.md) effectue la journalisation et fournit les données de métriques d’un compte de stockage Azure. Nous vous recommandons d’utiliser ces données pour suivre les requêtes, analyser les tendances d’utilisation et diagnostiquer les problèmes liés à votre compte de stockage.

## <a name="prevent-detect-and-respond-to-threats"></a>Prévenir, détecter et traiter les menaces
[Azure Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure. Il fournit une supervision de la sécurité et une gestion des stratégies intégrées pour l’ensemble de vos abonnements Azure, vous aidant ainsi à détecter les menaces qui pourraient passer inaperçues. De plus, il est compatible avec un vaste ensemble de solutions de sécurité.

Le niveau Gratuit de Security Center offre une sécurité limitée pour vos ressources Azure uniquement. Le niveau Standard étend ces fonctions aux ressources locales et à d’autres clouds. Security Center Standard aide à rechercher et à corriger les failles de sécurité, appliquer des contrôles d’accès et d’application pour bloquer les activités malveillantes, détecter les menaces à l’aide de l’analytique et de l’analyse décisionnelle et à répondre rapidement en cas d’attaque. Vous pouvez essayer Security Center Standard gratuitement pendant 60 jours. Nous vous recommandons de [mettre à niveau votre abonnement Azure sur Security Center Standard](../../security-center/security-center-get-started.md).

Utilisez Security Center pour obtenir une vue centralisée de l’état de sécurité de toutes vos ressources Azure. Vérifiez d’un coup d’œil que les contrôles de sécurité appropriés sont en place et configurés correctement, et identifiez rapidement les ressources nécessitant votre attention.

Security Center s’intègre également à [Microsoft Defender ATP (Advanced Threat Protection)](../../security-center/security-center-wdatp.md), qui fournit des fonctionnalités complètes de protection évolutive des points de terminaison (PEPT). L’intégration de Microsoft Defender ATP vous permet de repérer les anomalies. Vous pouvez également détecter les attaques avancées sur les points de terminaison de serveur surveillés par Security Center et y répondre.

Presque toutes les organisations d’entreprises ont un système Security Information and Event Management (SIEM) pour aider à identifier les menaces émergentes en consolidant les informations de journaux à partir de divers appareils de collecte de signaux. Les journaux sont ensuite analysés par un système d’analytique de données pour déterminer ce qui est « intéressant » à partir du bruit inévitable dans toutes les solutions d’analytique et de collecte de données.

[Azure Sentinel](../../sentinel/overview.md) est une solution native cloud et évolutive de type SIEM (Security Information and Event Management) et SOAR (Security Orchestrated Automated Response). Azure Sentinel assure une analyse de sécurité intelligente et fournit des informations sur les menaces via la détection des alertes, la visibilité des menaces, la recherche proactive et la réponse automatisée face aux menaces.

Voici quelques meilleures pratiques pour prévenir, détecter et répondre aux menaces :

**Bonne pratique** : augmentez la vitesse et l’évolutivité de votre solution SIEM en optant pour une solution cloud.   
**Détail** : examinez les fonctionnalités et capacités [d’Azure Sentinel](../../sentinel/overview.md) et comparez-les avec les capacités de ce que vous utilisez actuellement en local. Envisagez d’adopter Azure Sentinel si cette solution répond aux exigences SIEM de votre organisation.

**Bonne pratique** : recherchez les vulnérabilités de sécurité les plus graves afin de définir les priorités d’investigation.   
**Détail** : examinez votre [degré de sécurisation Azure](../../security-center/secure-score-security-controls.md) pour afficher les recommandations résultant des stratégies Azure et des initiatives intégrées à Azure Security Center. Ces recommandations peuvent aider à résoudre les principaux risques tels que les mises à jour de sécurité, la protection de point de terminaison, le chiffrement, les configurations de sécurité, le WAF manquant, les machines virtuelles connectées à Internet et bien plus encore.

Le degré de sécurisation, qui est basé sur les contrôles du Center for Internet Security (CIS), vous permet d’évaluer la sécurité Azure de votre organisation par rapport à des sources externes. Une validation externe permet de confirmer et d’enrichir la stratégie de sécurité de votre équipe.

**Bonne pratique** : surveillez la posture de sécurité des machines, réseaux, services de stockage et de données, et des applications pour découvrir d’éventuels problèmes de sécurité et les classer par ordre de priorité.  
**Détail** : suivez les [recommandations de sécurité](../../security-center/security-center-recommendations.md) de Security Center en commençant par les éléments avec la priorité la plus élevée.

**Bonne pratique** : intégrez des alertes de Security Center dans votre solution SIEM.   
**Détail** : la plupart des organisations avec une solution SIEM l’utilisent comme un centre d’échanges central pour les alertes de sécurité qui nécessitent la réponse d’un analyste. Les événements traités produits par Security Center sont publiés dans le journal d’activité Azure, l’un des types de journaux disponibles avec Azure Monitor. Azure Monitor offre un pipeline centralisé pour router les données de monitoring dans un outil SIEM. Consultez [Diffuser des alertes vers un système SIEM, SOAR ou une solution de gestion des services informatiques](../../security-center/export-to-siem.md) pour obtenir des instructions. Si vous utilisez Azure Sentinel, consultez [Connect data from Azure Security Center](../../sentinel/connect-azure-security-center.md) (Connecter des données à partir d’Azure Security Center).

**Bonne pratique** : intégrez les journaux Azure à votre solution SIEM.   
**Détail** : utilisez [Azure Monitor pour collecter et exporter des données](../../azure-monitor/overview.md#integrate-and-export-data). Cette pratique est critique pour permettre d’investiguer sur des incidents de sécurité, et la rétention des journaux en ligne est limitée. Si vous utilisez Azure Sentinel, consultez [Connecter des sources de données](../../sentinel/connect-data-sources.md).

**Bonne pratique** : accélérez vos processus d’investigation et de recherche et réduisez les faux positifs en intégrant des fonctionnalités de détection de point de terminaison et de réponse (EDR) dans votre examen de l’attaque.   
**Détail** : [Activer l'intégration de Microsoft Defender for Endpoint](../../security-center/security-center-wdatp.md#enabling-the-microsoft-defender-for-endpoint-integration) par le biais de votre stratégie de sécurité Security Center. Envisagez d’utiliser Azure Sentinel pour rechercher les menaces et répondre aux incidents.

## <a name="monitor-end-to-end-scenario-based-network-monitoring"></a>Superviser le réseau selon un scénario de bout en bout
Les clients créent un réseau de bout en bout dans Azure en combinant des ressources réseau comme un réseau virtuel, ExpressRoute, Application Gateway et des équilibreurs de charge. La surveillance est disponible sur chacune des ressources réseau.

[Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) est un service régional. Utilisez ses outils de diagnostic et de visualisation pour superviser et diagnostiquer les conditions au niveau d’un réseau, dans, vers et à partir d’Azure.

Voici des bonnes pratiques pour la supervision du réseau et les outils disponibles.

**Bonne pratique** : Automatisez la surveillance réseau à distance avec la capture de paquets.  
**Détail** : Surveillez et diagnostiquez les problèmes réseau sans vous connecter à vos machines virtuelles à l’aide de Network Watcher. Déclenchez la [capture de paquets](../../network-watcher/network-watcher-alert-triggered-packet-capture.md) en définissant des alertes et bénéficiez d’un accès à des informations en temps réel sur le niveau de performance au niveau du paquet. Quand vous identifiez un problème, vous pouvez l’examiner en détail pour effectuer de meilleurs diagnostics.

**Bonne pratique** : Obtenez des insights sur votre trafic réseau en utilisant des journaux de flux.  
**Détail** : Développez une meilleure compréhension de vos modèles de trafic réseau à l’aide des [journaux de flux des groupes de sécurité réseau](../../network-watcher/network-watcher-nsg-flow-logging-overview.md). Les informations contenues dans les journaux de flux vous aident à recueillir des données sur la conformité, l’audit et la supervision de votre profil de sécurité réseau.

**Bonne pratique** : Diagnostiquez les problèmes de connectivité d’un VPN.  
**Détail** : Utilisez Network Watcher pour [diagnostiquer les problèmes les plus courants liés aux connexions et à la passerelle VPN](../../network-watcher/network-watcher-diagnose-on-premises-connectivity.md). Vous pouvez non seulement identifier le problème, mais également utiliser des journaux d’activité détaillés pour approfondir vos recherches.

## <a name="secure-deployment-by-using-proven-devops-tools"></a>Sécuriser le déploiement à l’aide d’outils DevOps éprouvés
Utilisez les bonnes pratiques DevOps suivantes pour garantir la productivité et l’efficacité de votre entreprise et de vos équipes.

**Bonne pratique** : Automatisez la génération et le déploiement des services.  
**Détail** : [L’infrastructure en tant que code](/azure/devops/learn/what-is-infrastructure-as-code) est un ensemble de techniques et de pratiques qui aident les professionnels de l’informatique à supprimer la charge de travail que représentent la génération et la gestion quotidiennes d’une infrastructure modulaire. Elle permet aux professionnels de l’informatique de générer et de gérer leur environnement serveur moderne d’une façon similaire à celle dont les développeurs de logiciels génèrent et gèrent le code de l’application.

Vous pouvez utiliser [Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) pour provisionner vos applications à l’aide d’un modèle déclaratif. Dans un modèle unique, vous pouvez déployer plusieurs services ainsi que leurs dépendances. Le même modèle vous permet de déployer plusieurs fois votre application à chaque phase du cycle de vie de l’application.

**Bonne pratique** : Générez et déployez automatiquement des applications web Azure ou des services cloud.  
**Détail** : Vous pouvez configurer vos projets Azure DevOps Projects afin de les [générer et de les déployer automatiquement](/azure/devops/pipelines/index) sur des applications web Azure ou des services cloud. Azure DevOps déploie automatiquement les fichiers binaires après avoir effectué une génération sur Azure au terme de chaque archivage de code. Le processus de génération de package est équivalent à la commande Package de Visual Studio, et les étapes de la publication sont identiques à la commande Publier dans Visual Studio.

**Bonne pratique** : automatisez la gestion des mises en production.  
**Détail** : [Azure Pipelines](/azure/devops/pipelines/index) est une solution pour automatiser le déploiement en plusieurs étapes et pour gérer le processus de mise en production. Créez des pipelines de déploiement gérés et continus pour des publications rapides, faciles et fréquentes. Avec Azure Pipelines, vous pouvez automatiser votre processus de mise en production et avoir des workflows d’approbation prédéfinis. Déployez localement et dans le cloud, étendez et personnalisez en fonction de vos besoins.

**Bonne pratique** : Vérifiez les performances de votre application avant de la lancer ou de déployer des mises à jour en production.  
**Détail** : exécutez des [tests de charge](/azure/devops/test/load-test/overview#alternatives) basés sur le cloud pour :

- Rechercher des problèmes de performances dans votre application.
- Améliorer la qualité du déploiement.
- Garantir que votre application est toujours disponible.
- Garantir que votre application peut gérer le trafic de votre prochaine campagne de lancement ou marketing.

[Apache JMeter](https://jmeter.apache.org/) est un outil gratuit, populaire et open source soutenu par une forte communauté.

**Bonne pratique** : Surveillez les performances des applications.  
**Détail** : [Azure Application Insights](../../azure-monitor/app/app-insights-overview.md) est un service extensible de gestion des performances des applications destiné aux développeurs web sur de multiples plateformes. Utilisez Application Insights pour superviser votre application web en direct. Il détecte automatiquement les problèmes de performances. Il intègre des outils d’analyse pour vous aider à diagnostiquer les problèmes et à comprendre ce que font les utilisateurs avec votre application. Il a été conçu pour vous permettre d’améliorer continuellement les performances et la convivialité.

## <a name="mitigate-and-protect-against-ddos"></a>Prévenir les attaques DDoS et s’en protéger
DDoS (Distributed Denial of Service, déni de service distribué) est un type d’attaque qui tente d’épuiser les ressources d’une application. Son objectif est d’affecter la disponibilité de l’application et sa capacité à gérer des demandes légitimes. Ces attaques de plus en plus sophistiquées gagnent en importance et en impact. Elles peuvent être ciblées sur n’importe quel point de terminaison qui est publiquement accessible via Internet.

La conception et la génération d’une résilience DDoS nécessitent la planification et la conception de divers modes d’échec. Voici les bonnes pratiques relatives à la création de services résistants aux attaques DDoS sur Azure.

**Bonne pratique** : Assurez-vous que la sécurité est une priorité tout au long du cycle de vie d’une application, de la conception et de l’implémentation au déploiement et aux opérations. Les applications peuvent contenir des bogues qui laissent un volume relativement faible de requêtes conçues utiliser beaucoup de ressources, ce qui entraîne une interruption de service.  
**Détail** : Pour permettre de protéger un service fonctionnant sur Microsoft Azure, vous devez bien comprendre l’architecture de votre application et respecter les [cinq piliers de la qualité logicielle](/azure/architecture/guide/pillars). Vous devez avoir connaissance des volumes de trafic habituels, du modèle de connectivité entre l’application et d’autres applications, et des points de terminaison de service exposés à l’Internet public.

Il est extrêmement important que vous conceviez une application suffisamment résiliente pour surmonter une attaque ciblée par déni de service. La plateforme Azure intègre des fonctionnalités de sécurité et de confidentialité, à commencer par [SDL (Security Development Lifecycle)](https://www.microsoft.com/sdl). SDL tient compte de la sécurité à chaque phase de développement et vérifie qu’Azure est continuellement mis à jour pour le rendre encore plus sécurisé.

**Bonne pratique** : Concevez vos applications de sorte qu’elles puissent être [mises à l’échelle horizontalement](/azure/architecture/guide/design-principles/scale-out) pour répondre à la demande d’une charge amplifiée, en particulier dans le cadre d’une attaque DDoS. Si votre application repose sur une seule instance unique d’un service, cela crée un point de défaillance unique. L’approvisionnement de plusieurs instances rend votre système plus résilient et plus évolutif.  
**Détail** : Pour [Azure App Service](../../app-service/overview.md), sélectionnez un [plan App Service](../../app-service/overview-hosting-plans.md) qui offre plusieurs instances.

Pour les Azure Cloud Services, configurez chacun de vos rôles de manière à utiliser [plusieurs instances](../../cloud-services/cloud-services-choose-me.md).

Pour [Machines virtuelles Azure](../../virtual-machines/windows/overview.md), vérifiez que votre architecture de machine virtuelle inclut plusieurs machines virtuelles et que chaque machine virtuelle est dans un [groupe à haute disponibilité](../../virtual-machines/windows/tutorial-availability-sets.md). Nous vous recommandons d’utiliser des groupes de machines virtuelles identiques pour les fonctionnalités de mise à l’échelle automatique.

**Bonne pratique** : Le fait de superposer des défenses dans une application réduit les chances de réussite d’une attaque. Implémentez des conceptions sécurisées pour vos applications à l’aide des fonctionnalités intégrées à la plateforme Azure.  
**Détail** : Plus la taille (surface d’exposition) de l’application est importante, plus le risque d’attaque est élevé. Vous pouvez réduire la surface d’exposition en créant une liste d’approbation permettant de limiter l’espace d’adressage IP exposé et les ports d’écoute qui ne sont pas nécessaires sur les équilibreurs de charge ([Azure Load Balancer](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) et [Azure Application Gateway](../../application-gateway/application-gateway-create-probe-portal.md)).

Les [groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md) permettent également de réduire la surface d’attaque. Vous pouvez utiliser des [balises de service](../../virtual-network/network-security-groups-overview.md#service-tags) et des [groupes de sécurité d’application](../../virtual-network/network-security-groups-overview.md#application-security-groups) pour simplifier la création de règles de sécurité et configurer la sécurité réseau comme prolongement naturel de la structure d’une application.

Vous devez déployer les services Azure dans un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) dans la mesure du possible. Les ressources de service peuvent ainsi communiquer par le biais d’adresses IP privées. Le trafic du service Azure à partir d’un réseau virtuel utilise des adresses IP publiques comme adresses IP source par défaut.

Le fait d’utiliser des [points de terminaison de service](../../virtual-network/virtual-network-service-endpoints-overview.md) force le trafic de service à utiliser des adresses privées de réseau virtuel comme adresses IP sources lors de l’accès au service Azure à partir d’un réseau virtuel.

Il arrive fréquemment que les ressources locales d’un client soient attaquées en même temps que ses ressources dans Azure. Si vous connectez un environnement local à Azure, réduisez l’exposition des ressources locales à l’Internet public.

Azure propose deux [offres de service](../../ddos-protection/ddos-protection-overview.md) DDoS qui fournissent une protection contre les attaques réseau :

- La protection de base est intégrée à Azure par défaut sans coût supplémentaire. De par son échelle et sa capacité, le réseau Azure déployé à l’échelle mondiale assure une défense contre les attaques courantes de la couche réseau. Cette défense est par ailleurs renforcée par le monitoring continu du trafic et l’atténuation en temps réel. La protection de base ne nécessite aucun changement de la part de l’utilisateur au niveau de configuration ou de l’application et permet de protéger tous les services Azure, notamment les services PaaS comme Azure DNS.
- La protection standard fournit des fonctions d’atténuation DDoS avancées contre les attaques réseau. Cette solution s’adapte automatiquement pour protéger vos ressources Azure spécifiques. La protection est facile à activer pendant la création de réseaux virtuels. Vous pouvez également l’activer après la création ; aucun changement au niveau de l’application ou de la ressource ne s’impose.

## <a name="enable-azure-policy"></a>Activer Azure Policy
[Azure Policy](../../governance/policy/overview.md) est un service d’Azure que vous utilisez pour créer, affecter et gérer des stratégies. Ces stratégies appliquent des règles et effets sur vos ressources, qui restent donc conformes aux normes et aux contrats de niveau de service de l’entreprise. en évaluant vos ressources pour vérifier leur conformité par rapport aux stratégies affectées.

Activez Azure Policy pour surveiller et appliquer la stratégie écrite de votre organisation. Cela permettra d’assurer la conformité aux exigences de sécurité obligatoires ou de votre société en gérant de façon centralisée les stratégies de sécurité dans toutes vos charges de travail cloud hybrides. Découvrez comment [créer et gérer des stratégies pour assurer la conformité](../../governance/policy/tutorials/create-and-manage.md). Consultez [Azure Policy definition structure](../../governance/policy/concepts/definition-structure.md) (Structure de définition Azure Policy) pour une vue d’ensemble des éléments d’une stratégie.

Voici quelques meilleures pratiques de sécurité à suivre après avoir adopté Azure Policy :

**Bonne pratique** : la stratégie prend en charge plusieurs types d’effet. Vous pouvez en savoir plus en lisant l’article [Azure Policy definition structure](../../governance/policy/concepts/definition-structure.md#policy-rule) (Structure de définition Azure Policy). Les opérations métier peuvent se voir affecter par l’effet **refuser** et l’effet **corriger**. Nous vous conseillons donc de commencer par l’effet **auditer** pour réduire le risque d’impact négatif de la stratégie.   
**Détail** : [démarrez les déploiements de stratégie en mode audit](../../governance/policy/concepts/definition-structure.md#policy-rule), puis passez aux effets **refuser** ou **corriger**. Testez et passez en revue les résultats de l’effet auditer avant de passer aux effets **refuser** ou **corriger**.

Pour plus d’informations, consultez [Didacticiel : Créer et gérer des stratégies pour appliquer la conformité](../../governance/policy/tutorials/create-and-manage.md).

**Bonne pratique** : identifiez les rôles responsables pour surveiller les violations de stratégie et garantir que l’action corrective appropriée est mise en œuvre rapidement.   
**Détail** : attribuez le rôle de conformité de la surveillance via le [portail Azure](../../governance/policy/how-to/get-compliance-data.md#portal) ou la [ligne de commande](../../governance/policy/how-to/get-compliance-data.md#command-line).

**Bonne pratique** : Azure Policy est une représentation technique des stratégies écrites d’une organisation. Mappez toutes les définitions Azure Policy vers des stratégies organisationnelles pour éviter la confusion et améliorer la cohérence.   
**Détail** : Mappage de document dans la documentation de votre organisation ou dans la définition Azure Policy elle-même en ajoutant une référence à la directive organisationnelle dans la [définition de stratégie](../../governance/policy/concepts/definition-structure.md#display-name-and-description) ou la description de la [définition d’initiative](../../governance/policy/concepts/initiative-definition-structure.md#metadata).

## <a name="monitor-azure-ad-risk-reports"></a>Surveiller les rapports de risque Azure AD
La grande majorité des violations de sécurité ont lieu lorsque des cybercriminels parviennent à accéder à un environnement en volant l’identité d’un utilisateur. Détecter les identités compromises n’est pas chose aisée. Azure AD utilise les algorithmes Machine Learning et des modèles heuristiques adaptatifs pour détecter les actions suspectes liées aux comptes de votre utilisateur. Chaque action suspecte détectée est stockée dans un enregistrement appelé [détection d’événement à risque](../../active-directory/identity-protection/overview-identity-protection.md). Les détections de risques sont enregistrés dans les rapports de sécurité Azure AD. Pour plus d’informations, renseignez-vous sur le [rapport sur la sécurité des utilisateurs courant un risque](../../active-directory/identity-protection/overview-identity-protection.md) et le [rapport de connexions risquées](../../active-directory/identity-protection/overview-identity-protection.md).

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.

Les ressources suivantes fournissent des informations générales sur la sécurité Azure et les services Microsoft associés :
* [Blog de l’équipe de sécurité Azure](/archive/blogs/azuresecurity/) : pour obtenir des informations à jour sur les dernières actualités sur la sécurité Azure
* [Centre de réponse aux problèmes de sécurité Microsoft](https://technet.microsoft.com/library/dn440717.aspx) : emplacement où les vulnérabilités de sécurité Microsoft, dont les problèmes rencontrés avec Azure, peuvent être rapportées ou signalées par e-mail à l’adresse secure@microsoft.com