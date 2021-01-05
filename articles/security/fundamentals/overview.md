---
title: Introduction de la sécurité Azure | Microsoft Docs
description: Initiez-vous à Azure Security, à ses services et à son fonctionnement en lisant cette vue d’ensemble.
services: security
documentationcenter: na
author: UnifyCloud
manager: rkarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/18/2019
ms.author: TomSh
ms.openlocfilehash: 749c32091ed10d5bb39d7b67c8b737e002fc9909
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693730"
---
# <a name="introduction-to-azure-security"></a>Présentation de la sécurité Azure
## <a name="overview"></a>Vue d’ensemble
Nous savons que la sécurité est la priorité dans le cloud et combien il est important que vous trouviez des informations précises et à jour sur la sécurité Azure. L’une des meilleures raisons d’utiliser Azure pour vos applications et services est de tirer parti de sa large gamme de fonctionnalités et outils de sécurité. Ces outils et fonctionnalités permettent de créer des solutions sécurisées sur la plateforme Azure sécurisée. Microsoft Azure assure la confidentialité, l’intégrité et la disponibilité des données client, tout en permettant la gestion transparente des responsabilités.

Cet article fournit une vue d’ensemble complète au niveau de la sécurité disponible avec Azure.

### <a name="azure-platform"></a>Plateforme Azure
Azure est une plateforme de services de cloud public qui prend en charge un large éventail de systèmes d’exploitation, de langages de programmation, d’infrastructures, d’outils, de bases de données et d’appareils. Elle permet d’exécuter des conteneurs Linux avec l’intégration de Docker, de créer des applications avec JavaScript, Python, .NET, PHP, Java et Node.js, de créer des serveurs principaux pour des appareils iOS, Android et Windows.

Les services de cloud public Azure prennent en charge les technologies auxquelles des millions de développeurs et de professionnels de l’informatique font déjà confiance. Lorsque vous générez ou faites migrer des ressources informatiques vers un fournisseur de services de cloud public, vous comptez sur les capacités de cette organisation à protéger vos applications et données avec les services et les contrôles qu’elle vous fournit pour gérer la sécurité de vos ressources cloud.

L’infrastructure d’Azure est conçue de l’installation aux applications pour héberger des millions de clients simultanément, et constitue une base fiable permettant de répondre aux besoins des entreprises en matière de sécurité.

En outre, Azure vous offre un large éventail d’options de sécurité configurables, ainsi que la possibilité de contrôler ces options pour vous permettre de personnaliser la sécurité et de répondre ainsi aux exigences uniques des déploiements de votre organisation. Ce document vous permet de comprendre comment les fonctionnalités de sécurité d’Azure peuvent vous aider à répondre à ces besoins.

> [!Note]
> Ce document est principalement consacré aux contrôles destinés aux clients que vous pouvez utiliser pour personnaliser et renforcer la sécurité de vos applications et services.
>
> Pour plus d’informations sur la façon dont Microsoft sécurise la plateforme Azure elle-même, consultez [Sécurité de l’infrastructure Azure](infrastructure.md).

## <a name="summary-of-azure-security-capabilities"></a>Résumé des fonctionnalités de sécurité Azure

### <a name="features-to-secure-the-azure-platform"></a>Fonctionnalités pour sécuriser la plateforme Azure
Vous pouvez consulter les fonctionnalités suivantes pour vous assurer que la plateforme Azure est managée de manière sécurisée. Des liens sont indiqués pour permettre d’explorer davantage comment Microsoft traite les questions de confiance des clients dans quatre zones : plateforme sécurisée, confidentialité et contrôles, conformité et transparence.

| [Plateforme sécurisée](https://www.microsoft.com/trustcenter/Security/default.aspx)  | [Confidentialité et contrôles](https://www.microsoft.com/trustcenter/Privacy/default.aspx)  |[Conformité](https://www.microsoft.com/trustcenter/Compliance/default.aspx)   | [Transparence](https://www.microsoft.com/trustcenter/Transparency/default.aspx) |
| :-- | :-- | :-- | :-- |
| [Cycle de développement de sécurité](https://www.microsoft.com/sdl/), audits internes | [Gérer vos données tout le temps](https://www.microsoft.com/trustcenter/Privacy/You-own-your-data) | [Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter/default.aspx) |[Comment Microsoft sécurise les données client dans les services Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx) |
| [Formation obligatoire sur les mesures de sécurité, vérifications de la formation](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx) |  [Contrôler l’emplacement des données](https://www.microsoft.com/trustcenter/Privacy/Where-your-data-is-located) |  [Hub des contrôles communs](https://www.microsoft.com/trustcenter/Common-Controls-Hub) |[Comment Microsoft gère l’emplacement des données dans les services Azure](https://azuredatacentermap.azurewebsites.net/)|
| [Test de pénétration](https://downloads.cloudsecurityalliance.org/star/self-assessment/StandardResponsetoRequestforInformationWindowsAzureSecurityPrivacy.docx), [détection d’intrusion, DDoS](https://www.microsoft.com/trustcenter/Security/ThreatManagement), [audits et journalisation](https://www.microsoft.com/trustcenter/Security/AuditingAndLogging) | [Fournir l’accès aux données selon vos conditions](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms) |  [Liste de contrôle d’obligation de diligence des services cloud](https://www.microsoft.com/trustcenter/Compliance/Due-Diligence-Checklist) |[Qui dans Microsoft peut accéder à vos données dans quelles conditions](https://www.microsoft.com/trustcenter/Privacy/Who-can-access-your-data-and-on-what-terms)|
| [Centre de données ultramoderne](https://www.microsoft.com/cloud-platform/global-datacenters), sécurité physique, [réseau sécurisé](network-overview.md) | [Réponse à l’application de la législation](https://www.microsoft.com/trustcenter/Privacy/Responding-to-govt-agency-requests-for-customer-data) |  [Conformité par service, site et secteur](https://www.microsoft.com/trustcenter/Compliance/default.aspx) |[Comment Microsoft sécurise les données client dans les services Azure](https://www.microsoft.com/trustcenter/Transparency/default.aspx)|
|  [Réponse aux incidents de sécurité](https://aka.ms/SecurityResponsepaper), [responsabilité partagée](./shared-responsibility.md) |[Normes de confidentialité strictes](https://www.microsoft.com/TrustCenter/Privacy/We-set-and-adhere-to-stringent-standards) |  | [Passer en revue la certification des services Azure, concentrateur de transparence](https://www.microsoft.com/trustcenter/Compliance/default.aspx)|

### <a name="features-to-secure-data-and-application"></a>Fonctionnalités pour sécuriser les données et les applications
En fonction du modèle de service cloud, la responsabilité de la personne chargée de la gestion de la sécurité de l’application ou du service varie. La plateforme Azure propose des fonctionnalités pour vous aider à remplir ces tâches via des fonctionnalités intégrées et des solutions pour partenaires qui peuvent être déployées dans un abonnement Azure.

Les fonctionnalités intégrées sont organisées en six zones fonctionnelles : Opérations, Applications, Stockage, Réseau, Calcul et Identité. Des informations supplémentaires sur les fonctionnalités et fonctions disponibles dans ces six zones de la plateforme Azure sont fournies par le biais des informations de résumé.

## <a name="operations"></a>Opérations
Cette section contient des informations supplémentaires sur les fonctionnalités clés des opérations de sécurité et des informations de synthèse sur ces fonctionnalités.

### <a name="security-and-audit-dashboard"></a>Tableau de bord Security and Audit
La [solution Security and Audit](../../security-center/security-center-introduction.md) offre une vue détaillée de la sécurité informatique de votre organisation. Elle fournit des [requêtes de recherche](/archive/blogs/msoms/easy-microsoft-operations-management-suite-search-queries) intégrées pour détecter les problèmes importants qui requièrent votre attention. Le tableau de bord [Security and Audit](../../azure-monitor/overview.md) est l’écran d’accueil pour tout ce qui se rapporte à la sécurité dans Journaux Azure Monitor. Il fournit un aperçu global de l’état de sécurité de vos ordinateurs. Il vous permet également de voir tous les événements des dernières 24 heures, des 7 derniers jours ou de n’importe quel intervalle de temps personnalisé.

En outre, vous pouvez configurer Security & Compliance pour [effectuer automatiquement des actions spécifiques](/archive/blogs/robdavies/simple-look-at-oms-alert-remediation-with-runbooks-part-1) lorsqu’un événement particulier est détecté.

### <a name="azure-resource-manager"></a>Azure Resource Manager
[Azure Resource Manager](../../azure-resource-manager/management/deployment-models.md) vous permet de travailler avec les ressources de votre solution en tant que groupe. Vous pouvez déployer, mettre à jour ou supprimer toutes les ressources de votre solution dans le cadre d’une opération unique et coordonnée. Vous utilisez un [modèle Azure Resource Manager](/archive/blogs/canitpro/devops-basics-infrastructure-as-code-arm-templates) de déploiement pouvant fonctionner dans différents environnements (environnements de test, intermédiaire et de production). Le gestionnaire de ressources assure la sécurité, les fonctions d’audit et de balisage pour vous aider à gérer vos ressources après le déploiement.

Les déploiements basés sur un modèle Azure Resource Manager contribuent à améliorer la sécurité des solutions déployées dans Azure, car des paramètres de contrôle de sécurité standard peuvent être intégrés aux déploiements basés sur un modèle standardisé. De cette façon, le risque d’erreurs de configuration de sécurité, qui peuvent se produire au cours des déploiements manuels, est réduit.

### <a name="application-insights"></a>Application Insights
[Application Insights](/azure/application-insights/) est un service de gestion des performances des applications (APM) extensible destiné aux développeurs web. Application Insights vous permet de surveiller vos applications web dynamiques et de détecter automatiquement les anomalies des performances. Il intègre de puissants outils d’analyse pour vous aider à diagnostiquer les problèmes et à comprendre ce que font les utilisateurs avec votre application. Il analyse votre application tout au long de son exécution, pendant les tests et une fois que vous avez l’avez publiée ou déployée.

Application Insights crée des graphiques et tableaux présentant, par exemple, les heures de la journée à laquelle vous avez le plus d’utilisateurs, la réactivité de l’application et comment elle est prise en charge par les services externes dont elle dépend.

En cas d’incidents, d’échecs ou de problèmes de performances, vous pouvez effectuer une recherche dans les données de télémétrie pour diagnostiquer la cause en détail. Et le service vous envoie des messages électroniques si des modifications sont apportées à la disponibilité et à la performance de votre application. Application Insights devient donc un outil de sécurité précieux, car il apporte la « disponibilité » dans la triade relative à la sécurité : confidentialité, intégrité et disponibilité.

### <a name="azure-monitor"></a>Azure Monitor
[Azure Monitor](/azure/monitoring-and-diagnostics/) offre des services de visualisation, requête, routage, alertes, mise à l’échelle automatique et automatisation pour les données de l’infrastructure Azure ([journal d’activité](../../azure-monitor/platform/platform-logs-overview.md)) et pour chaque ressource Azure ([journaux de diagnostic](../../azure-monitor/platform/platform-logs-overview.md)). Vous pouvez utiliser Azure Monitor pour vous alerter sur les événements liés à la sécurité qui sont générés dans les journaux d’activité Azure.

### <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor
[Journaux Azure Monitor](https://azure.microsoft.com/documentation/services/log-analytics/) fournit une solution de gestion informatique aussi bien pour les infrastructures en local que pour les solutions tierces basées sur le cloud (comme AWS) en plus des ressources Azure. Les données issues d’Azure Monitor peuvent être acheminées directement dans Journaux Azure Monitor, afin que vous puissiez voir les métriques et les journaux pour l’ensemble de votre environnement en un seul endroit.

La solution Journaux Azure Monitor peut se révéler un outil utile dans l’analyse d’investigation et dans les autres analyses de la sécurité, car il vous permet d’effectuer rapidement des recherches dans de grandes quantités d’entrées liées à la sécurité, avec une approche de type requête flexible. Par ailleurs, des [journaux de pare-feu et proxy locaux peuvent être exportés dans Azure, et mis à disposition pour analyse avec Journaux Azure Monitor](../../azure-monitor/platform/agent-windows.md).

### <a name="azure-advisor"></a>Azure Advisor
[Azure Advisor](../../advisor/index.yml) est un conseiller personnalisé basé dans le cloud qui vous aide à optimiser vos déploiements Azure. Il analyse les données de télémétrie d’utilisation et la configuration de vos ressources. Il recommande ensuite des solutions pour améliorer les [performances](../../advisor/advisor-performance-recommendations.md), la [sécurité](../../advisor/advisor-security-recommendations.md) et la [haute disponibilité](../../advisor/advisor-high-availability-recommendations.md) de vos ressources tout en [réduisant vos dépenses Azure globales](../../advisor/advisor-cost-recommendations.md). Azure Advisor fournit des recommandations en termes de sécurité, qui peuvent améliorer sensiblement votre sécurité globale pour les solutions que vous déployez dans Azure. Ces recommandations sont tirées de l’analyse de la sécurité effectuée par [Azure Security Center](../../security-center/security-center-introduction.md).

### <a name="azure-security-center"></a>Azure Security Center
[Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure. Il fournit une surveillance de la sécurité et une gestion des stratégies intégrées pour l’ensemble de vos abonnements Azure, vous aidant ainsi à détecter les menaces qui pourraient passer inaperçues. De plus, il est compatible avec un vaste écosystème de solutions de sécurité.

En outre, Security Center aide aux opérations de sécurité en vous fournissant un tableau de bord unique qui présente les alertes et les suggestions qui peuvent être traitées immédiatement. Dans la plupart des cas, vous pourrez corriger les problèmes d’un simple clic dans la console Azure Security Center.
## <a name="applications"></a>Applications
Cette section contient des informations supplémentaires sur les fonctionnalités clés de la sécurité des applications et des informations de synthèse sur ces fonctionnalités.

### <a name="web-application-vulnerability-scanning"></a>Analyse de la vulnérabilité des applications web
L’une des méthodes les plus simples pour démarrer les tests de vulnérabilité de votre [application App Service](../../app-service/overview.md) consiste à utiliser [l’intégration à Tinfoil Security](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/) pour effectuer l’analyse des vulnérabilités dans votre application en un clic. Vous pouvez afficher les résultats des tests dans un rapport facile à comprendre et découvrir comment résoudre chaque problème de vulnérabilité grâce à des instructions pas à pas.

### <a name="penetration-testing"></a>Test de pénétration
Si vous préférez effectuer vos propres tests d’intrusion ou utiliser une autre suite logicielle d’analyse ou un autre fournisseur, vous devez suivre le [processus d’approbation de test d’intrusion Azure](./pen-testing.md) et obtenir une approbation préalable pour effectuer les tests d’intrusion de votre choix.

### <a name="web-application-firewall"></a>Pare-feu d’application web
Le pare-feu d’applications web (WAF) [d’Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) aide à protéger les applications web des attaques basées sur le web courantes comme l’injection de code SQL, les attaques de script de site à site et les piratages de session. Il est préconfiguré avec une protection contre les menaces identifiées par [OWASP (Open Web Application Security Project) comme les 10 vulnérabilités les plus courantes](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project).

### <a name="authentication-and-authorization-in-azure-app-service"></a>Authentification et autorisation dans Azure App Service
[L’authentification/autorisation App Service](../../app-service/overview-authentication-authorization.md) est une fonctionnalité qui permet à votre application de connecter les utilisateurs. Vous n’êtes ainsi donc pas obligé de modifier le code sur le serveur principal. Elle propose un moyen simple de protéger votre application et fonctionne avec des données par utilisateur.

### <a name="layered-security-architecture"></a>Architecture de sécurité multiniveau
Dans la mesure où les [environnements App Service](../../app-service/environment/app-service-app-service-environment-intro.md) fournissent un environnement d’exécution isolé déployé dans un [réseau virtuel Azure](../../virtual-network/virtual-networks-overview.md), les développeurs peuvent créer une architecture de sécurité multiniveau offrant différents niveaux d’accès réseau pour chaque couche Application. Un souhait commun est de masquer les API principales de l’accès Internet général, et d’autoriser uniquement les API à être appelées par les applications web en amont. Les [groupes de sécurité réseau (NSG)](../../virtual-network/virtual-network-vnet-plan-design-arm.md) peuvent être utilisés sur des sous-réseaux d’un réseau virtuel Azure contenant des environnements App Service pour restreindre l’accès public aux applications API.

### <a name="web-server-diagnostics-and-application-diagnostics"></a>Diagnostics de serveur web et diagnostics d’application
Les applications web App Service fournissent des fonctionnalités de diagnostic pour les informations de journalisation provenant du serveur Web et de l'application web. Ces informations sont réparties, en toute logique, en [diagnostics de serveur web](../../app-service/troubleshoot-diagnostic-logs.md) et en [diagnostics d’application](/previous-versions/system-center/system-center-2012-R2/hh530058(v=sc.12)). Le serveur web inclut deux avancées majeures dans le diagnostic des problèmes et leur résolution sur les sites et dans les applications.

Pour la première nouvelle fonctionnalité, il s’agit des informations d’état en temps réel sur les pools d’applications, les processus Worker, les sites, les domaines d’application et les demandes en cours d’exécution. Quant à la seconde nouvelle fonctionnalité, il s’agit des événements de suivi détaillés qui effectuent le suivi d’une demande dans le processus complet des demandes/réponses.

Pour activer la collecte de ces événements de suivi, IIS 7 peut être configuré pour capturer automatiquement les journaux d’activité de suivi complets, au format XML, pour toute demande spécifique basée sur le temps écoulé ou les codes de réponse d’erreur.

#### <a name="web-server-diagnostics"></a>Diagnostics de serveur web
Vous pouvez activer ou désactiver les types de journaux d’activité suivants :

-   Messages d’erreur détaillés : informations d’erreur détaillées pour les codes d’état HTTP qui indiquent un échec (code d’état 400 ou supérieur). Il peut s'agir d'informations qui vous aident à déterminer la raison pour laquelle le serveur a renvoyé le code d'erreur.

-   Suivi des demandes ayant échoué : informations détaillées sur les demandes qui ont échoué, y compris une trace des composants IIS utilisés pour traiter la demande et la durée dans chaque composant. Cela peut se révéler utile si vous essayez d'améliorer les performances du site ou d'isoler la cause d'une erreur HTTP spécifique.

-   Journalisation du serveur web : informations sur les transactions HTTP à l’aide du format de fichier journal étendu W3C. Ce rapport se révèle utile pour déterminer les métriques globales d’un site, comme le nombre de demandes traitées ou le nombre de demandes émanant d’une adresse IP spécifique.

#### <a name="application-diagnostics"></a>Diagnostic d'application
La fonctionnalité [Diagnostic d’application](../../app-service/troubleshoot-diagnostic-logs.md) vous permet de capturer des informations générées par une application web. Les applications ASP.NET peuvent utiliser la classe [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) pour enregistrer des informations dans le journal de diagnostic d'application. Dans Diagnostic d’application, il existe deux types d’événement principaux, ceux qui sont liés aux performances de l’application et ceux qui sont liés à ses erreurs et échecs. Les échecs et les erreurs peuvent être répartis en problèmes de connectivité, sécurité et défaillance. Les problèmes de défaillance sont généralement associés à un problème lié au code d’application.

Dans Diagnostic d’application, vous pouvez afficher les événements regroupés comme suit :

-   Tout (affiche tous les événements)
-   Erreurs d’application (affiche les événements d’exception)
-   Performances (affiche les événements de performance)

## <a name="storage"></a>Stockage
Cette section contient des informations supplémentaires sur les fonctionnalités clés de la sécurité du stockage Azure et des informations de synthèse sur ces fonctionnalités.

### <a name="azure-role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC)
Vous pouvez sécuriser votre compte de stockage avec le contrôle d’accès en fonction du rôle Azure (RBAC Azure). Restreindre l’accès en fonction des principes du [besoin de connaître](https://en.wikipedia.org/wiki/Need_to_know) et du [privilège minimum](https://en.wikipedia.org/wiki/Principle_of_least_privilege) est impératif pour les organisations désireuses d’appliquer des stratégies de sécurité pour l’accès aux données. Ces droits d’accès sont octroyés en affectant le rôle Azure approprié aux groupes et aux applications, dans une étendue donnée. Vous pouvez utiliser les [rôles intégrés Azure](../../role-based-access-control/built-in-roles.md), comme Contributeur de compte de stockage, pour affecter des privilèges aux utilisateurs. L’accès aux clés de stockage pour un compte de stockage avec le modèle [Azure Resource Manager](../../storage/blobs/security-recommendations.md) peut être contrôlé via RBAC Azure.

### <a name="shared-access-signature"></a>Signature d’accès partagé
Une [signature d’accès partagé (SAP)](../../storage/common/storage-sas-overview.md) fournit un accès délégué aux ressources de votre compte de stockage. La SAP vous permet d’octroyer à un client des autorisations d’accès limité à des objets de votre compte de stockage pendant une période donnée et avec un ensemble défini d’autorisations. Vous pouvez accorder ces autorisations limitées sans partager les clés d’accès de votre compte.

### <a name="encryption-in-transit"></a>Chiffrement en transit
Le chiffrement en transit est un mécanisme de protection des données transmises sur des réseaux. Le stockage Azure vous permet de sécuriser les données à l’aide des éléments suivants :
-   Le [chiffrement au niveau du transport](../../storage/blobs/security-recommendations.md) (HTTPS, par exemple) lorsque vous transférez des données vers ou à partir de Stockage Azure.

-   Le [chiffrement câblé](../../storage/blobs/security-recommendations.md), par exemple le [chiffrement SMB 3.0](../../storage/blobs/security-recommendations.md) pour les [partages de fichiers Azure](../../storage/files/storage-dotnet-how-to-use-files.md).

-   Le chiffrement côté client, pour chiffrer les données avant leur transfert dans le stockage et les déchiffrer après leur transfert à partir du stockage.

### <a name="encryption-at-rest"></a>Chiffrement au repos
Pour de nombreuses organisations, le chiffrement des données au repos est une étape obligatoire du processus de gestion de la confidentialité, de la conformité et de la souveraineté des données. Trois fonctionnalités de la sécurité du stockage Azure fournissent un chiffrement des données « au repos ».

-   [Storage Service Encryption](../../storage/common/storage-service-encryption.md) vous permet de demander que le service de stockage chiffre automatiquement les données lors de leur écriture dans Azure Storage.

-   [Client-side Encryption](../../storage/common/storage-client-side-encryption.md) fournit également la fonctionnalité de chiffrement au repos.

-   [Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md) vous permet de chiffrer les disques de données et de système d’exploitation utilisés par une machine virtuelle IaaS.

### <a name="storage-analytics"></a>Storage Analytics
[L’analyse du stockage Azure](/rest/api/storageservices/fileservices/storage-analytics) effectue la journalisation et fournit les données d’indicateurs de performance d’un compte de stockage. Vous pouvez utiliser ces données pour suivre les demandes, analyser les tendances d'utilisation et diagnostiquer les problèmes liés à votre compte de stockage Storage Analytics enregistre des informations détaillées sur les demandes ayant réussi ou échoué pour un service de stockage. Ces informations peuvent servir à analyser des demandes individuelles et à diagnostiquer les problèmes au niveau d'un service de stockage. Les demandes sont enregistrées sur la base du meilleur effort. Les types de demandes authentifiées suivants sont enregistrés :
-   Demandes ayant réussi.

-   Demandes ayant échoué, y compris les erreurs de délai d'expiration, limitation, réseau, autorisation et autres erreurs.

-   Demandes utilisant une signature d'accès partagé (SAS), y compris les demandes ayant réussi et ayant échoué.

-   Demandes de données d'analyse.

### <a name="enabling-browser-based-clients-using-cors"></a>Activation de clients basés sur le navigateur à l’aide de CORS
[CORS (Cross-Origin Resource Sharing) ](/rest/api/storageservices/fileservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services) est un mécanisme qui autorise les domaines à donner des autorisations mutuelles pour accéder aux ressources de l’un ou l’autre domaine. L’agent utilisateur envoie des en-têtes supplémentaires pour s’assurer que le code JavaScript chargé à partir d’un domaine est autorisé à accéder aux ressources situées dans un autre domaine. Ce dernier domaine répond alors avec des en-têtes supplémentaires qui autorisent ou refusent l’accès du domaine d’origine à ses ressources.

Les services de stockage Azure prennent désormais en charge CORS si bien que lorsque vous avez défini les règles CORS du service, une demande authentifiée exécutée auprès du service à partir d’un autre domaine est évaluée pour déterminer si elle est autorisée conformément aux règles que vous avez spécifiées.

## <a name="networking"></a>Mise en réseau
Cette section contient des informations supplémentaires sur les fonctionnalités clés de la sécurité du réseau Azure et des informations de synthèse sur ces fonctionnalités.

### <a name="network-layer-controls"></a>Contrôles de la couche réseau
Le contrôle d’accès réseau consiste à limiter la connectivité vers et depuis certains appareils ou sous-réseaux, et représente le cœur de la sécurité réseau. Le contrôle d’accès réseau permet de vous assurer que seuls les utilisateurs et appareils autorisés peuvent accéder à vos machines virtuelles et à vos services.

#### <a name="network-security-groups"></a>Network Security Group
Un [groupe de sécurité réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md) est un pare-feu de base de filtrage des paquets avec état qui vous permet de contrôler l’accès sur la base d’un algorithme à [5 tuples](https://www.techopedia.com/definition/28190/5-tuple). Les groupes de sécurité réseau n’effectuent pas d’inspection de la couche d’application ni de contrôles d’accès authentifiés. Ils permettent de contrôler le trafic entre les sous-réseaux d’un réseau virtuel Azure et le trafic entre un réseau virtuel Azure et Internet.

#### <a name="route-control-and-forced-tunneling"></a>Contrôle du routage et tunneling forcé
La possibilité de contrôler le comportement de routage sur vos réseaux virtuels Azure est une fonctionnalité essentielle en matière de contrôle d’accès et de sécurité réseau. Par exemple, si vous voulez vous assurer que tout le trafic vers et depuis votre réseau virtuel Azure passe par cette appliance de sécurité virtuelle, vous devez être en mesure de contrôler et de personnaliser le comportement du routage. Pour ce faire, vous pouvez configurer des itinéraires définis par l’utilisateur dans Azure.

Les [itinéraires définis par l’utilisateur](../../virtual-network/virtual-networks-udr-overview.md) vous permettent de personnaliser les chemins d’accès entrants et sortants pour le trafic existant entre des machines virtuelles individuelles ou des sous-réseaux afin de vous assurer que l’itinéraire soit le plus sécurisé possible. [tunneling forcé](https://www.petri.com/azure-forced-tunneling) est un mécanisme que vous pouvez utiliser pour empêcher vos services de se connecter aux appareils sur Internet.

Cela n’empêche pas les services d’accepter des connexions entrantes ni d’y répondre. Les serveurs web frontaux doivent pouvoir répondre aux demandes provenant d’hôtes Internet, ce qui explique pourquoi le trafic Internet est autorisé à entrer sur ces serveurs web et pourquoi les serveurs web peuvent y répondre.

Le tunneling forcé est fréquemment utilisé pour forcer le trafic sortant vers Internet à passer par des pare-feu et proxys de sécurité locaux.

#### <a name="virtual-network-security-appliances"></a>Appliances de sécurité de réseau virtuel
Les groupes de sécurité réseau, les itinéraires définis par l’utilisateur et le tunneling forcé vous offrent une sécurité au niveau des couches réseau et transport du [modèle OSI](https://en.wikipedia.org/wiki/OSI_model), mais il est parfois judicieux d’étendre la sécurité à des niveaux supérieurs de la pile. Vous pouvez accéder à ces fonctionnalités avancées de sécurité réseau via une solution d’appliance de sécurité réseau de partenaire Azure. Pour connaître les dernières solutions de sécurité réseau des partenaires Azure, rendez-vous sur la [Place de marché Azure](https://azure.microsoft.com/marketplace/) et recherchez les mots clés « sécurité » et « sécurité réseau ».

### <a name="azure-virtual-network"></a>Réseau virtuel Azure
Un réseau virtuel Azure (VNet) est une représentation de votre propre réseau dans le cloud. Il s’agit d’un isolement logique de la structure de réseau Azure dédiée à votre abonnement. Vous pouvez contrôler complètement les blocs d’adresses IP, les paramètres DNS, les stratégies de sécurité et les tables de routage de ce réseau. Vous pouvez segmenter votre réseau virtuel en sous-réseaux et placer des machines virtuelles IaaS Azure et/ou des [services cloud (instances de rôle PaaS)](../../cloud-services/cloud-services-choose-me.md) sur des réseaux virtuels Azure.

En outre, vous pouvez connecter le réseau virtuel à votre réseau local à l’aide des [options de connectivité](../../vpn-gateway/index.yml) disponibles dans Azure. En bref, vous pouvez développer votre réseau sur Azure et maîtriser totalement vos blocs d’adresses IP avec les que procurent la mise à l’échelle d’entreprise d’Azure.

La mise en réseau Azure prend en charge différents scénarios d’accès à distance sécurisé, entres autres :

-   [Connecter des stations de travail à un réseau virtuel Azure](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

-   [Connecter un réseau local à un réseau virtuel Azure à l’aide d’un VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md)

-   [Connecter un réseau local à un réseau virtuel Azure à l’aide d’une liaison réseau étendu dédiée](../../expressroute/expressroute-introduction.md)

-   [Connecter des réseaux virtuels Azure entre eux](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

### <a name="vpn-gateway"></a>Passerelle VPN
Pour envoyer du trafic réseau entre votre réseau virtuel Azure et votre site local, vous devez créer une passerelle VPN pour votre réseau virtuel Azure. Une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) est un type de passerelle de réseau virtuel qui envoie le trafic chiffré à travers une connexion publique. Vous pouvez également utiliser des passerelles VPN pour envoyer du trafic entre les réseaux virtuels Azure via la structure de réseau Azure.

### <a name="express-route"></a>ExpressRoute
Microsoft Azure [ExpressRoute](../../expressroute/expressroute-introduction.md) est une liaison réseau étendu dédiée qui vous permet d’étendre vos réseaux locaux au cloud de Microsoft via une connexion privée dédiée assurée par un fournisseur de connectivité.

![ExpressRoute](./media/overview/azure-security-figure-1.png)

Grâce à ExpressRoute, vous pouvez établir des connexions aux services de cloud computing Microsoft, comme Microsoft Azure, Microsoft 365 et CRM Online. La connectivité peut provenir d'un réseau universel (IP VPN), d’un réseau Ethernet point à point ou d’une interconnexion virtuelle via un fournisseur de connectivité dans un centre de colocalisation.

Les connexions ExpressRoute ne transitent pas par l’Internet public et peuvent donc être considérées comme plus sécurisées que les solutions VPN. Elles offrent ainsi de meilleurs niveaux de fiabilité, de rapidité, de latence et de sécurité que les connexions classiques sur Internet.


### <a name="application-gateway"></a>Application Gateway
Microsoft [Azure Application Gateway](../../application-gateway/overview.md) intègre [Application Delivery Controller (ADC)](https://en.wikipedia.org/wiki/Application_delivery_controller) en tant que service, offrant diverses fonctionnalités d’équilibrage de charge de couche 7 pour votre application.

![Application Gateway](./media/overview/azure-security-figure-2.png)

Ce service vous permet d’optimiser la productivité de la batterie de serveurs web en déchargeant une terminaison TLS gourmande en ressources processeur vers la passerelle Application Gateway (processus également appelé « déchargement TLS » ou « pontage TLS »). Elle fournit également d’autres fonctionnalités de routage de couche 7, notamment la distribution en tourniquet (round robin) du trafic entrant, l’affinité de session basée sur les cookies, le routage basé sur le chemin d’accès de l’URL et la possibilité d’héberger plusieurs sites web derrière une seule passerelle Application Gateway. La passerelle Azure Application Gateway est un équilibreur de charge de couche 7.

Elle assure l’exécution des requêtes HTTP de basculement et de routage des performances entre serveurs locaux ou dans le cloud.

L’application offre de nombreuses fonctionnalités de contrôleur de livraison d’applications (ADC, Application Delivery Controller), notamment l’équilibrage de charge HTTP, l’affinité de session basée sur les cookies, le [déchargement TLS](../../web-application-firewall/ag/tutorial-restrict-web-traffic-powershell.md), les sondes d’intégrité personnalisées, la prise en charge de plusieurs sites, et bien d’autres.

### <a name="web-application-firewall"></a>Pare-feu d’applications web
Le pare-feu d’applications web est une fonctionnalité de la passerelle [Azure Application Gateway](../../application-gateway/overview.md) qui offre une protection pour les applications web qui utilisent la passerelle d’application pour les fonctions Application Delivery Control (ADC) standard. Le pare-feu d’applications web le fait en les protégeant contre la plupart des 10 plus courantes vulnérabilités web de l’OWASP.

![Pare-feu d’applications web](./media/overview/azure-security-figure-3.png)

-   Protection contre les injections de code SQL

-   Protection contre les attaques web courante comme l’injection de commande, les dissimulations de requêtes HTTP, la séparation de réponse HTTP et les attaques RFI (Remote File Inclusion)

-   Protection contre les violations de protocole HTTP

-   Protection contre les anomalies de protocole HTTP comme un agent-utilisateur hôte manquant et les en-têtes Accept

-   Protection contre les robots, les crawlers et les scanneurs

-   Détection des erreurs de configuration d’application courantes (par exemple, Apache, IIS, etc.)

Un pare-feu d’applications web centralisé pour protéger contre les attaques web facilite grandement la gestion de la sécurité et offre une meilleure garantie de protection de l’application contre les menaces d’intrusion. Une solution WAF peut également réagir plus rapidement à une menace de sécurité en exécutant la mise à jour corrective d’une vulnérabilité connue dans un emplacement central plutôt que de sécuriser individuellement chacune des applications web. Les passerelles d’application existantes peuvent être facilement converties en une passerelle d’application avec un pare-feu d’applications web.

### <a name="traffic-manager"></a>Traffic Manager
Microsoft [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) vous permet de contrôler la répartition du trafic utilisateur pour les points de terminaison de service dans différents centres de données. Les points de terminaison de service pris en charge par Traffic Manager incluent des machines virtuelles Azure, des applications web et des services cloud. Vous pouvez également utiliser Traffic Manager avec des points de terminaison externes non-Azure. Traffic Manager utilise le DNS (Domain Name System) pour diriger les demandes des clients vers le point de terminaison approprié en fonction de la [méthode de routage du trafic](../../traffic-manager/traffic-manager-routing-methods.md) et de l’intégrité des points de terminaison.

Traffic Manager fournit un large éventail de méthodes de routage du trafic pour répondre à différents besoins d’application, la [surveillance](../../traffic-manager/traffic-manager-monitoring.md) de l’intégrité des points de terminaison et le basculement automatique. Traffic Manager est résilient aux défaillances, notamment à l’échec d’une région Azure entière.

### <a name="azure-load-balancer"></a>Azure Load Balancer
[Azure Load Balancer](../../load-balancer/load-balancer-overview.md) offre une haute disponibilité et des performances réseau élevées pour vos applications. Il s’agit d’un équilibreur de charge Layer-4 (TCP, UDP) qui distribue le trafic entrant parmi des instances saines de services définis dans un jeu à charge équilibrée. Azure Load Balancer peut être configuré pour :

-   équilibrer la charge du trafic Internet entrant sur les machines virtuelles. Cette configuration est appelée [Équilibrage de charge public](../../load-balancer/components.md#frontend-ip-configurations).

-   équilibrer le trafic entre des machines virtuelles dans un réseau virtuel, entre des machines virtuelles dans les services cloud ou entre des ordinateurs locaux et des machines virtuelles dans un réseau virtuel entre différents locaux. Cette configuration est appelée [équilibrage de charge interne](../../load-balancer/components.md#frontend-ip-configurations).

- Transférer du trafic externe vers une instance spécifique de machine virtuelle

### <a name="internal-dns"></a>DNS interne
Vous pouvez gérer la liste des serveurs DNS utilisés dans un réseau virtuel à partir du portail de gestion ou du fichier de configuration réseau. Le client peut ajouter jusqu’à 12 serveurs DNS par réseau virtuel. Quand vous spécifiez des serveurs DNS, assurez-vous de les indiquer dans l’ordre approprié pour l’environnement du client. Les listes de serveurs DNS ne fonctionnent pas sur le modèle du tourniquet (round-robin). Elles sont utilisées dans l’ordre où elles sont spécifiées. Si le premier serveur DNS de la liste est accessible, le client utilise ce serveur DNS, qu’il fonctionne correctement ou non. Pour modifier l’ordre des serveurs DNS pour le réseau virtuel du client, supprimez-les de la liste et rajoutez-les dans l’ordre souhaité par le client. DNS prend en charge la notion de disponibilité dans la triade relative à la sécurité : confidentialité, intégrité et disponibilité.

### <a name="azure-dns"></a>Azure DNS
Le [système DNS (Domain Name System)](/previous-versions/windows/it-pro/windows-server-2003/cc772774(v=ws.10)) se charge de traduire (ou résoudre) un nom de site web ou de service en une adresse IP. [DNS Azure](../../dns/dns-overview.md) est un service d’hébergement pour les domaines DNS qui offre une résolution de noms à l’aide de l’infrastructure Microsoft Azure. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS avec les mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure. DNS prend en charge la notion de disponibilité dans la triade relative à la sécurité : confidentialité, intégrité et disponibilité.

### <a name="azure-monitor-logs-nsgs"></a>Groupe de sécurité réseau de Journaux Azure Monitor
Vous pouvez activer les catégories de journaux de diagnostic suivantes pour les groupes de sécurité réseau :

-   Événement : Contient les entrées pour lesquelles des règles de groupe de sécurité réseau sont appliquées aux machines virtuelles et aux rôles d’instance en fonction de l’adresse MAC. L’état de ces règles est collecté toutes les 60 secondes.

-   Compteur de règles : contient les entrées correspondant au nombre de fois où chaque règle NSG a été appliquée pour refuser ou autoriser le trafic.

### <a name="security-center"></a>Security Center
[Azure Security Center](../../security-center/security-center-introduction.md) analyse continuellement l’état de sécurité de vos ressources Azure pour les meilleures pratiques de sécurité réseau. Lorsque Security Center identifie des failles de sécurité potentielles, il crée des [suggestions](../../security-center/security-center-recommendations.md) qui vous guident tout au long du processus de configuration des contrôles nécessaires afin de renforcer et protéger vos ressources.

## <a name="compute"></a>Calcul
Cette section contient des informations supplémentaires sur les fonctionnalités clés de cette zone et des informations de synthèse sur ces fonctionnalités.

### <a name="antimalware--antivirus"></a>Logiciels anti-programmes malveillants et antivirus
Azure IaaS met à votre disposition des logiciels anti-programmes malveillants provenant de fournisseurs de sécurité tels que Microsoft, Symantec, Trend Micro, McAfee et Kaspersky. Ceux-ci permettent de protéger vos machines virtuelles contre les fichiers malveillants, les logiciels de publicité et d’autres menaces. [Microsoft Antimalware](antimalware.md) pour Azure Cloud Services et machines virtuelles offre une fonctionnalité de protection qui permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Microsoft Antimalware fournit des alertes configurables lorsqu’un logiciel malveillant ou indésirable connu tente de s’installer ou de s’exécuter sur vos systèmes Azure. Microsoft Antimalware peut également être déployé à l’aide d’Azure Security Center.

### <a name="hardware-security-module"></a>Module de sécurité matériel
Le chiffrement et l’authentification n’améliorent pas la sécurité, sauf si les clés elles-mêmes sont protégées. Vous pouvez simplifier la gestion et la sécurité de vos clés et clés secrètes critiques en les stockant dans [Azure Key Vault](../../key-vault/general/overview.md). Key Vault permet de stocker les clés dans des modules de sécurité matériels (HSM) certifiés conformes aux normes FIPS 140-2 de niveau 2. Vos clés de chiffrement SQL Server pour la sauvegarde ou le [chiffrement transparent des données](/sql/relational-databases/security/encryption/transparent-data-encryption) peuvent toutes être stockées dans Key Vault avec les clés ou secrets de vos applications. Les autorisations et l’accès à ces éléments protégés sont gérés via [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/).

### <a name="virtual-machine-backup"></a>Sauvegarde de machine virtuelle
[Sauvegarde Azure](../../backup/backup-overview.md) est une solution qui protège les données de vos applications, sans investissement en capital et avec des frais de fonctionnement minimaux. Les erreurs rencontrées par les applications peuvent endommager vos données, et les erreurs humaines peuvent introduire des bogues dans vos applications qui peuvent engendrer des problèmes de sécurité. Avec Sauvegarde Azure, vos machines virtuelles exécutant Windows et Linux sont protégées.

### <a name="azure-site-recovery"></a>Azure Site Recovery
Une partie importante de la stratégie de [continuité des activités et de récupération d’urgence (BCDR)](../../best-practices-availability-paired-regions.md) de votre organisation consiste à savoir comment maintenir les charges de travail et les applications d’entreprise opérationnelles lorsque des interruptions planifiées et non planifiées se produisent. [Azure Site Recovery](../../site-recovery/site-recovery-overview.md) aide à coordonner la réplication, le basculement et la récupération des charges de travail et des applications afin qu’elles soient disponibles à partir d’un site secondaire si votre site principal tombe en panne.

### <a name="sql-vm-tde"></a>SQL VM TDE
Le chiffrement [TDE (Transparent data encryption)](/previous-versions/azure/virtual-machines/windows/sqlclassic/virtual-machines-windows-classic-ps-sql-keyvault) et le chiffrement au niveau des colonnes (CLE) sont des fonctionnalités de chiffrement de serveur SQL. Cette forme de chiffrement nécessite que les clients gèrent et stockent les clés de chiffrement que vous utilisez pour le chiffrement.

Le service Azure Key Vault (coffre de clés Azure, AKV) est conçu pour optimiser la sécurité et la gestion de ces clés dans un emplacement sécurisé et hautement disponible. Le connecteur SQL Server permet à SQL Server d’utiliser ces clés depuis Azure Key Vault.

Si vous exécutez SQL Server sur des ordinateurs locaux, vous devez suivre la procédure d’accès à Azure Key Vault à partir de votre instance SQL Server locale. Mais, pour SQL Server dans des machines virtuelles Azure, vous pouvez gagner du temps à l’aide de la fonctionnalité Azure Key Vault Integration. Avec quelques applets de commande Azure PowerShell pour activer cette fonctionnalité, vous pouvez automatiser la configuration nécessaire pour qu'une machine virtuelle SQL puisse accéder à votre coffre de clés.

### <a name="vm-disk-encryption"></a>Chiffrement de disque de machine virtuelle
[Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md) est une nouvelle fonctionnalité qui vous permet de chiffrer vos disques de machine virtuelle IaaS Windows et Linux. Elle utilise la fonctionnalité standard BitLocker de Windows et la fonctionnalité DM-Crypt de Linux pour fournir le chiffrement de volume du système d’exploitation et des disques de données. La solution est intégrée à Azure Key Vault, ce qui vous permet de contrôler et de gérer les clés et clés secrètes de chiffrement de disque dans votre abonnement Key Vault. Elle garantit également que toutes les données sur les disques de vos machines virtuelles sont chiffrées au repos dans votre stockage Azure.

### <a name="virtual-networking"></a>Réseau virtuel
Les machines virtuelles nécessitent une connectivité réseau. Pour cela, les machines virtuelles doivent être connectées à un réseau virtuel Azure. Un réseau virtuel Azure est une construction logique basée sur le réseau physique Azure. Chaque [réseau virtuel logique Azure](../../virtual-network/virtual-networks-overview.md) est isolé des autres réseaux virtuels Azure. Cet isolement permet de s’assurer que le trafic réseau dans votre déploiement n’est pas accessible aux autres clients Microsoft Azure.

### <a name="patch-updates"></a>Mises à jour correctives
Les mises à jour correctives fournissent les références permettant de détecter et de corriger les problèmes potentiels et de simplifier le processus de gestion des mises à jour logicielles. Vous pouvez ainsi réduire le nombre de mises à jour logicielles à déployer dans votre entreprise et augmenter votre capacité à surveiller la conformité.

### <a name="security-policy-management-and-reporting"></a>Gestion des stratégies de sécurité et création de rapports
[Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure. Il fournit une surveillance de la sécurité et une gestion des stratégies intégrées pour l’ensemble de vos abonnements Azure, vous aidant ainsi à détecter les menaces qui pourraient passer inaperçues. De plus, il est compatible avec un vaste écosystème de solutions de sécurité.

## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès
La sécurisation des systèmes, applications et données commencent par les contrôles d’accès basés sur l’identité. Les fonctionnalités de gestion des identités et accès qui sont intégrées aux services et produits Microsoft aident à protéger vos informations personnelles et d’organisation de tout accès non autorisé, tout en les mettant à disposition des utilisateurs légitimes où et lorsqu’ils en ont besoin.

### <a name="secure-identity"></a>Identité sécurisée
Microsoft a recours à plusieurs technologies et pratiques de sécurité dans ses produits et services pour gérer les identités et les accès.

-   [Multi-Factor Authentication](https://azure.microsoft.com/services/multi-factor-authentication/) exige que les utilisateurs utilisent plusieurs méthodes pour l’accès, en local et dans le cloud. Elle fournit une authentification forte avec un éventail d’options de vérification conviviales tout en proposant aux utilisateurs un processus de connexion simple.

-   [Microsoft Authenticator](https://aka.ms/authenticator) fournit une expérience Multi-Factor Authentication conviviale, qui fonctionne avec les comptes Microsoft Azure Active Directory et Microsoft et qui prend en charge les appareils wearable et les approbations par empreinte digitale.

-   [L’application de la stratégie de mot de passe](../../active-directory/authentication/concept-sspr-policy.md) renforce la sécurité des mots de passe traditionnels en imposant des exigences de longueur et de complexité, une rotation périodique forcée et un verrouillage de compte après des tentatives d’authentification en échec.

-   [L’authentification basée sur le jeton](../../active-directory/develop/authentication-vs-authorization.md) permet l’authentification via Azure Active Directory.

-   Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/built-in-roles.md) vous permet d’accorder un accès en fonction du rôle de l’utilisateur. Vous pouvez ainsi donner facilement aux utilisateurs uniquement le niveau d’accès dont ils ont besoin pour effectuer leurs tâches. Vous pouvez personnaliser RBAC Azure en fonction du modèle d’entreprise et de la tolérance au risque de votre organisation.

-   La [gestion d’identité intégrée (identité hybride)](../../active-directory/hybrid/plan-hybrid-identity-design-considerations-overview.md) vous permet de gérer le contrôle d’accès des utilisateurs dans plusieurs plateformes cloud et centres de données internes, en créant une identité de l’utilisateur unique pour l’authentification et l’autorisation sur toutes les ressources.

### <a name="secure-apps-and-data"></a>Sécuriser les applications et données
[Azure Active Directory](https://azure.microsoft.com/services/active-directory/), solution de cloud complète de gestion des identités et des accès, permet de sécuriser l’accès aux données des applications sur site et dans le cloud et simplifie la gestion des utilisateurs et des groupes. Elle combine des services d’annuaire essentiels, la gouvernance avancée des identités, la sécurité et la gestion des accès aux applications, et permet aux développeurs d’intégrer facilement la gestion d’identité basée sur des stratégies à leurs applications. Pour enrichir votre expérience Azure Active Directory, vous pouvez ajouter des fonctionnalités payées à l'aide des éditions Azure Active Directory de base, Premium P1 et Premium P2.

| Fonctionnalités gratuites/courantes     | Fonctionnalités de base    |Fonctionnalités de Premium P1 |Fonctionnalités de Premium P2 | Azure Active Directory Join - Fonctionnalités propres à Windows 10|
| :------------- | :------------- |:------------- |:------------- |:------------- |
|   [Objets annuaire](../../active-directory/fundamentals/active-directory-whatis.md), [Gestion des utilisateurs/groupes (ajout/mise à jour/suppression) / Attribution basée sur l'utilisateur, Inscription des appareils](../../active-directory/fundamentals/active-directory-whatis.md),   [Authentification unique (SSO)](../../active-directory/fundamentals/active-directory-whatis.md),  [Modification libre-service du mot de passe pour les utilisateurs du cloud](../../active-directory/fundamentals/active-directory-whatis.md),  [Connect (moteur de synchronisation qui étend les annuaires locaux à Azure Active Directory)](../../active-directory/fundamentals/active-directory-whatis.md),  [Rapports de sécurité/d’utilisation](../../active-directory/fundamentals/active-directory-whatis.md)       |  [Provisionnement/gestion des accès par groupe](../../active-directory/fundamentals/active-directory-whatis.md), [Réinitialisation libre-service du mot de passe pour les utilisateurs du cloud](../../active-directory/fundamentals/active-directory-whatis.md),  [Image de l'entreprise (personnalisation des pages de connexion/volet d'accès)](../../active-directory/fundamentals/active-directory-whatis.md),    [Proxy d'application](../../active-directory/fundamentals/active-directory-whatis.md),    [SLA de 99,9 %](../../active-directory/fundamentals/active-directory-whatis.md) |  [Gestion d’applications et de groupes libre-service/ajout d’applications libre-service/groupes dynamiques](../../active-directory/fundamentals/active-directory-whatis.md),   [Réinitialisation, modification et déverrouillage de mot de passe libre-service avec écriture différée locale](../../active-directory/fundamentals/active-directory-whatis.md),    [Multi-Factor Authentication (dans le cloud et localement - MFA Server)](../../active-directory/fundamentals/active-directory-whatis.md), [Licence d’accès client MIM + serveur MIM](../../active-directory/fundamentals/active-directory-whatis.md),     [Cloud App Discovery](../../active-directory/fundamentals/active-directory-whatis.md),  [Connect Health](../../active-directory/fundamentals/active-directory-whatis.md),   [Substitution automatique des mots de passe pour les comptes de groupe](../../active-directory/fundamentals/active-directory-whatis.md)|   [Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md),  [Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md)|   [Joindre un appareil à Azure AD, Desktop SSO, Microsoft Passport pour Azure AD, récupération BitLocker de l’administrateur](../../active-directory/fundamentals/active-directory-whatis.md),    [Inscription automatique GPM, récupération Bitlocker libre-service, administrateurs locaux supplémentaires pour appareils Windows 10 par le biais d’Azure AD Join](../../active-directory/fundamentals/active-directory-whatis.md)|


- [Cloud App Discovery](/cloud-app-security/set-up-cloud-discovery) est une fonctionnalité Premium d’Azure Active Directory qui vous permet d’identifier les applications cloud utilisées par les employés de votre organisation.

- [Azure Active Directory Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md) est un service de sécurité qui utilise les fonctionnalités de détection d’anomalie d’Azure Active Directory pour fournir une vue consolidée des détections des risques et des vulnérabilités potentielles qui pourraient affecter les identités de votre organisation.

- La fonctionnalité [Azure Active Directory Domain Services](https://azure.microsoft.com/services/active-directory-ds/) vous permet de joindre des machines virtuelles Azure à un domaine sans devoir déployer des contrôleurs de domaine. Les utilisateurs se connectent à ces machines virtuelles à l’aide de leurs informations d’identification Active Directory d’entreprise et accèdent aux ressources en toute transparence.

- Pour les applications utilisées par les consommateurs, [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) offre un service de gestion de l’identité global et hautement disponible, qui s’adapte à des centaines de millions d’identités et s’intègre aux plateformes mobiles et web. Vos clients peuvent se connecter à toutes vos applications via des expériences personnalisables qui utilisent les comptes de réseaux sociaux existants. Vous pouvez également créer des informations d’identification autonomes.

- [Azure Active Directory B2B Collaboration](../../active-directory/external-identities/what-is-b2b.md) est une solution d’intégration de partenaire sécurisée qui prend en charge les relations interentreprises en permettant aux partenaires d’accéder de façon sélective à vos applications et données d’entreprise à l’aide de leurs identités autogérées.

- [Azure Active Directory Join](../../active-directory/devices/overview.md) vous permet d’étendre les fonctionnalités du cloud aux appareils Windows 10 en vue d’une gestion centralisée. Cette fonctionnalité permet aux utilisateurs de se connecter au cloud d’entreprise ou d’organisation via Azure Active Directory, et simplifie l’accès aux applications et aux ressources.

- Le [proxy d’application Azure Active Directory](../../active-directory/manage-apps/application-proxy.md) offre une authentification unique (SSO) et un accès à distance sécurisé pour les applications web hébergées en local.

## <a name="next-steps"></a>Étapes suivantes

- Comprendre votre [responsabilité partagée dans le cloud](shared-responsibility.md).

- Découvrez comment [Azure Security Center](https://azure.microsoft.com/services/security-center/) vous aide à anticiper, à détecter et à traiter des menaces, grâce à une visibilité et à un contrôle accrus de la sécurité de vos ressources Azure.