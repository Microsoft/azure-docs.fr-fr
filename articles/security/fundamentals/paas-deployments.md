---
title: Meilleures pratiques pour les déploiements PaaS sécurisés - Microsoft Azure
description: Découvrez les meilleures pratiques pour concevoir, générer et gérer les applications cloud sécurisées sur Azure. Découvrez également les avantages de sécurité de PaaS par rapport aux autres modèles de service cloud.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: techlake
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/28/2019
ms.author: terrylan
ms.openlocfilehash: 77a2a32b9a6358c39a14cfe37eeb44f7cb90af0a
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841986"
---
# <a name="securing-paas-deployments"></a>Sécurisation des déploiements PaaS

Cet article fournit des informations qui vous permettent :

- de comprendre les avantages de l'hébergement d'application dans le cloud pour la sécurité ;
- d'évaluer les avantages de la plateforme en tant que service (PaaS) et autres modèles de service cloud pour la sécurité ;
- de passer pour la sécurité d'une approche orientée réseau à une approche de sécurité de périmètre orientée identité ;
- d'implémenter les bonnes pratiques recommandées de la sécurité de la PaaS.

[Développer des applications sécurisées sur Azure](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/) est un guide général des questions de sécurité et des contrôles que vous devez prendre en compte à chaque phase du cycle de vie du développement logiciel lors du développement d’applications pour le cloud.

## <a name="cloud-security-advantages"></a>Avantages du cloud en matière de sécurité
Il est important de comprendre la [répartition des responsabilités](shared-responsibility.md) entre vous et Microsoft. Localement, vous avez toute la responsabilité, mais, lorsque vous vous déplacez vers le cloud, certaines responsabilités sont transférées à Microsoft.

Le cloud offre [certains avantages pour la sécurité](shared-responsibility.md#cloud-security-advantages). Dans un environnement local, les organisations ont probablement des obligations non respectées et des ressources limitées pour investir dans la sécurité, avec pour résultat un environnement où les pirates informatiques sont en mesure d’exploiter des vulnérabilités à tous les niveaux.

Les organisations peuvent améliorer la détection des menaces et leur temps de réponse à l’aide de fonctionnalités de sécurité basées sur le cloud d’un fournisseur et l'intelligence du cloud.  En transférant les responsabilités au fournisseur de cloud, les organisations peuvent optimiser leur couverture de sécurité, ce qui leur permet de réaffecter des ressources de sécurité et leur budget à d'autres priorités de l’entreprise.

## <a name="security-advantages-of-a-paas-cloud-service-model"></a>Avantages d'un modèle de service cloud PaaS en matière de sécurité
Passons en revue les avantages pour la sécurité d’un déploiement PaaS Azure par rapport à un déploiement local.

![Les avantages d'une PaaS en matière de sécurité](./media/paas-deployments/advantages-of-paas.png)

En commençant par le premier élément, l’infrastructure physique, Microsoft réduit les risques courants et les responsabilités. Étant donné que le cloud Microsoft est surveillé en permanence par Microsoft, il est difficile à attaquer. Il n’y a aucun sens pour une personne malveillante de choisir le cloud Microsoft comme cible. À moins que le pirate informatique n'ait beaucoup d’argent et de ressources, il préfèrera passer à une autre cible.  

Au milieu de l'infrastructure, il n’existe aucune différence entre un déploiement PaaS et un déploiement local. Les risques sont similaires dans les couches d'application et de gestion des accès et des comptes. Dans la section suivante de cet article, nous vous présentons les bonnes pratiques pour éliminer ou réduire ces risques.

En haut de l'infrastructure (gouvernance des données et gestion des droits), vous prenez un risque qui peut être limité par la gestion de clés (la gestion des clés est traitée dans les bonnes pratiques). Même si la gestion des clés est une responsabilité supplémentaire, vous n'avez plus à gérer certaines zones d'un déploiement PaaS, et vous pouvez donc transférer des ressources vers la gestion des clés.

La plateforme Azure offre également une protection contre le DDoS renforcée à travers diverses technologies de réseau. Toutefois, tous les types de méthodes de protection contre le déni de service distribué (DDoS) basée sur le réseau ont leurs limites par lien et par centre de données. Afin d’éviter l’impact des attaques DDoS de grande envergure, vous pouvez tirer parti des capacités de cloud de base d’Azure qui permettent d'effectuer un scale-out rapidement et de vous défendre contre les attaques DDoS. Nous aborderons la procédure plus en détail dans les articles sur les pratiques recommandées.

## <a name="modernizing-the-defenders-mindset"></a>Une nouvelle mentalité pour la défense
Avec les déploiements PaaS, votre approche globale de la sécurité évolue. Vous n'avez pas besoin de tout contrôler vous-même et partagez la responsabilité avec Microsoft.

Une autre différence importante entre les déploiements PaaS et les déploiements locaux traditionnels est une nouvelle définition du périmètre de sécurité principal. Historiquement, le périmètre de sécurité local principal était votre réseau et la plupart des conceptions de sécurité locales utilisent le réseau en tant que pivot de sécurité principal. Pour les déploiements PaaS, il est préférable de tenir compte de l’identité comme périmètre de sécurité principal.

## <a name="adopt-a-policy-of-identity-as-the-primary-security-perimeter"></a>Adopter une stratégie d‘identité en tant que périmètre de sécurité principal
L'une des cinq caractéristiques essentielles du cloud computing est un large accès au réseau, ce qui rend l'approche centrée sur le réseau moins pertinente. Le cloud computing a en grande partie pour but de permettre aux utilisateurs d’accéder à des ressources, quel que soit leur emplacement. Pour la plupart des utilisateurs, l'emplacement est quelque part sur Internet.

La figure suivante montre comment le périmètre de sécurité a évolué d’un périmètre de réseau vers un périmètre d’identité. La sécurité se résume moins à la protection de votre réseau et plus à la défense de vos données, ainsi qu'à la gestion de la sécurité de vos applications et de vos utilisateurs. La principale différence est que vous pouvez axer la sécurité sur ce qui est important pour votre entreprise.

![L'identité en tant que nouveau périmètre de sécurité](./media/paas-deployments/identity-perimeter.png)

Initialement, les services PaaS Azure (par exemple, les rôles web et Azure SQL) fournissaient peu ou pas de défenses du périmètre de réseau traditionnelles. Il était entendu que l'élément devait être exposé sur Internet (rôle web) et que l’authentification fournit le nouveau périmètre (par exemple, BLOB ou Azure SQL).

Les pratiques de sécurité modernes partent du principe que l’adversaire a violé le périmètre du réseau. Par conséquent, les pratiques de défense modernes sont axées sur l'identité. Les organisations doivent établir un périmètre de sécurité basé sur les identités avec forte hygiène d’authentification et d’autorisation (bonnes pratiques).

Les principes et modèles pour le périmètre du réseau existaient depuis des décennies. En revanche, l’approche basée sur l'identité comme périmètre de sécurité principal n'est pas répandue dans l'industrie. Cela étant dit, nous avons accumulé suffisamment d’expérience pour fournir des recommandations générales qui s'appliquent dans la pratique à presque tous les services PaaS.

Voici les bonnes pratiques en matière de gestion du périmètre d’identité.

**Bonne pratique** : sécurisez vos clés et informations d’identification pour sécuriser votre déploiement PaaS.   
**Détail** : La perte de clés ou d'informations d’identification est un problème courant. Vous pouvez utiliser une solution centralisée où les clés et les secrets peuvent être stockés dans des modules de sécurité matériels (HSM). [Azure Key Vault](../../key-vault/general/overview.md) sauvegarde vos clés et vos secrets en chiffrant les clés d’authentification, les clés de compte de stockage, les clés de chiffrement de données, les fichiers .pfx et les mots de passe à l’aide de clés protégées par des HSM.

**Bonne pratique** : ne placez pas vos informations d’identification et autres secrets dans le code source ni GitHub.   
**Détail** : la seule chose qui est pire que la perte de vos clés et informations d’identification serait qu’un tiers non autorisé y accède. Des pirates peuvent tirer parti de technologies de robot pour rechercher les clés et les secrets stockés dans des référentiels de code, tels que GitHub. Ne placez pas de clé ni de secrets dans ces référentiels de code publics.

**Bonne pratique** : protégez vos interfaces de gestion de machine virtuelle sur les services hybrides PaaS et IaaS à l’aide d’une interface de gestion qui vous permet de gérer directement à distance ces machines virtuelles.   
**Détail** : des protocoles de gestion à distance tels que [SSH](https://en.wikipedia.org/wiki/Secure_Shell), [RDP](https://support.microsoft.com/kb/186607) et [communication à distance PowerShell](/powershell/module/microsoft.powershell.core/enable-psremoting) peuvent être utilisés. En règle générale, nous vous recommandons de ne pas activer l’accès à distance direct aux machines virtuelles à partir d’Internet.

Si possible, utilisez d’autres approches, comme les réseaux privés virtuels dans un réseau virtuel Azure. Si aucune autre approche n'est disponible, utilisez des phrases secrètes complexes et une authentification à deux facteurs (comme [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)).

**Bonne pratique** : utilisez une authentification forte et des plateformes d’autorisation.   
**Détail** : Utilisez des identités fédérées dans Azure AD au lieu des magasins d’utilisateurs personnalisés. Lorsque vous utilisez des identités fédérées, vous tirez parti d'une approche basée sur une plateforme et vous déléguez la gestion des identités autorisées à vos partenaires. Une approche d’identité fédérée est particulièrement importante quand le contrat de certains employés prend fin et que des informations doivent être répercutées sur plusieurs systèmes d’identité et d’autorisation.

Utilisez les mécanismes d‘authentification et d‘autorisation fournis par les plateformes au lieu d‘un code personnalisé. La raison est que le développement d'un code d’authentification personnalisé peut être sujet aux erreurs. La plupart de vos développeurs ne sont pas des experts en sécurité et ont peu de chance d’être conscients des subtilités et des derniers développements dans les domaines de l’authentification et l’autorisation. Le code commercial (par exemple, de Microsoft) est souvent très contrôlé en ce qui concerne la sécurité.

Utilisez une authentification à deux facteurs. L’authentification à deux facteurs est la norme actuelle pour l’authentification et l’autorisation, car elle évite les failles de sécurité inhérentes aux types d’authentification par nom d’utilisateur et mot de passe. L'accès aux interfaces de gestion Azure (portail / PowerShell à distance) et aux services destinés aux clients doit être conçu et configuré pour utiliser [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md).

Utilisez des protocoles d’authentification standard, tels qu'OAuth2 et Kerberos. Ces protocoles ont été largement contrôlés et sont probablement implémentés dans le cadre de vos bibliothèques de plateforme pour l’authentification et l’autorisation.

## <a name="use-threat-modeling-during-application-design"></a>Utiliser la modélisation des menaces lors de la conception d’une application
Microsoft [Security Development Lifecycle](https://www.microsoft.com/en-us/sdl) spécifie que les équipes doivent s’engager dans un processus appelé modélisation des menaces pendant la phase de conception. Pour faciliter ce processus, Microsoft a créé l’[outil SDL de modélisation des menaces](../develop/threat-modeling-tool.md). La modélisation de la conception de l’application et l’énumération des menaces [STRIDE](https://docs.google.com/viewer?a=v&pid=sites&srcid=ZGVmYXVsdGRvbWFpbnxzZWN1cmVwcm9ncmFtbWluZ3xneDo0MTY1MmM0ZDI0ZjQ4ZDMy) au-delà des limites d’approbation peuvent intercepter des erreurs de conception dès le début.

Le tableau suivant répertorie les menaces STRIDE et donne des exemples d’atténuation des risques que les fonctionnalités Azure utilisent. Ces atténuations ne fonctionnent pas dans tous les cas.

| Menace | Propriété de sécurité | Atténuation des risques pour la plateforme Azure |
| --- | --- | --- |
| Usurpation d’identité | Authentification | Exigez des connexions HTTPS. |
| Falsification | Intégrité | Validez des certificats TLS/SSL. |
| Répudiation | Non-répudiation | Activez [la surveillance et les diagnostics Azure](/azure/architecture/best-practices/monitoring). |
| Divulgation d’informations | Confidentialité | Chiffrez les données sensibles au repos à l’aide de [certificats de service](/rest/api/appservice/certificates). |
| Denial of service (déni de service) | Disponibilité | Surveillez les mesures de performances pour des conditions potentielles de déni de service. Implémentez des filtres de connexion. |
| Élévation de privilège | Autorisation | Utilisez [Privileged Identity Management](../../active-directory/privileged-identity-management/subscription-requirements.md). |

## <a name="develop-on-azure-app-service"></a>Développer sur Azure App Service
[Azure App Service](../../app-service/overview.md) est une offre PaaS qui vous permet de créer des applications mobiles et web pour tout type d’appareil ou de plateforme, et de vous connecter à des données en tout lieu, dans le cloud ou localement. App Service inclut les fonctionnalités web et mobiles qui étaient précédemment fournies séparément dans Sites Web Azure et Azure Mobile Services. Il inclut également de nouvelles fonctionnalités d’automatisation des processus d’entreprise et d’hébergement d’API cloud. App Service est un service intégré unique qui apporte un ensemble complet de fonctionnalités pour les scénarios web, mobiles et d’intégration.

Voici les bonnes pratiques relatives à l’utilisation d’App Service.

**Bonne pratique** : [S’authentifier par le biais d’Azure Active Directory (AD)](../../app-service/overview-authentication-authorization.md).   
**Détail** : App Service fournit un service OAuth 2.0 pour votre fournisseur d’identité. OAuth 2.0 privilégie la simplicité du développement client tout en fournissant des flux d’autorisation spécifiques pour les applications web, les applications de bureau et les téléphones mobiles. Azure AD utilise OAuth 2.0 pour vous permettre d’autoriser l’accès aux applications mobiles et web.

**Bonne pratique** : Restreindre l’accès en fonction des principes du besoin de connaître et du privilège minimum.   
**Détail** : La restriction de l’accès est indispensable pour les organisations qui veulent appliquer des stratégies de sécurité portant sur l’accès aux données. Vous pouvez utiliser la fonction de contrôle d’accès en fonction du rôle Azure (RBAC Azure) pour affecter des autorisations à des utilisateurs, groupes et applications dans une certaine étendue. Pour en savoir plus sur l’octroi aux utilisateurs du droit d’accès aux applications, consultez [Prise en main de la gestion des accès](../../role-based-access-control/overview.md).

**Bonne pratique** : Protégez vos clés.   
**Détail** : Azure Key Vault permet de protéger les clés de chiffrement et les secrets utilisés par les services et les applications cloud. Avec Key Vault, vous pouvez chiffrer les clés et les secrets (tels que les clés d’authentification, les clés de compte de stockage, les clés de chiffrement de données, les fichiers .PFX et les mots de passe) à l’aide de clés protégées par des modules de sécurité matériels (HSM). Pour une meilleure garantie, vous pouvez importer ou générer des clés HSM. Pour en savoir plus, consultez [Azure Key Vault](../../key-vault/general/overview.md). Vous pouvez également utiliser Key Vault pour gérer vos certificats TLS avec renouvellement automatique.

**Bonne pratique** : Limitez les adresses IP source entrantes.   
**Détail** : [App Service Environment](../../app-service/environment/intro.md) propose une fonctionnalité d’intégration de réseau virtuel qui vous permet de limiter les adresses IP sources entrantes par le biais de groupes de sécurité réseau. Les réseaux virtuels vous permettent de placer des ressources Azure dans un réseau routable non-Internet auquel vous contrôlez l’accès. Pour en savoir plus, consultez [Intégrer une application à un réseau virtuel Azure](../../app-service/web-sites-integrate-with-vnet.md).

**Bonne pratique** : supervisez l’état de sécurité de vos environnements App Service.   
**Détail** : utilisez Azure Security Center pour superviser vos environnements App Service. Lorsque Security Center identifie des failles de sécurité potentielles, il crée des [recommandations](../../security-center/asset-inventory.md) qui vous guident tout au long du processus de configuration des contrôles nécessaires.

> [!NOTE]
> La surveillance d’App Service est uniquement disponible en préversion, au [niveau Standard](../../security-center/security-center-pricing.md) dans Security Center.
>
>

## <a name="install-a-web-application-firewall"></a>Installer un pare-feu d’application web
Les applications Web sont de plus en plus la cible d’attaques malveillantes qui exploitent des vulnérabilités connues. Les types d’attaques les plus courantes sont l’injection de code SQL, les attaques de script site à site, entre autres. Empêcher ces attaques dans le code d’application peut se révéler difficile et nécessiter une maintenance rigoureuse, des mises à jour correctives ainsi que la surveillance au niveau d’un grand nombre de couches de la topologie de l’application. Un pare-feu d’applications web centralisé facilite grandement la gestion de la sécurité et offre une meilleure garantie de protection aux administrateurs de l’application contre les menaces ou les intrusions. Une solution WAF peut également réagir plus rapidement à une menace de sécurité en exécutant la mise à jour corrective d’une vulnérabilité connue dans un emplacement central plutôt que de sécuriser individuellement chacune des applications web. Les passerelles d’application existantes peuvent être facilement converties en une passerelle d’application avec un pare-feu d’applications web.

Le [pare-feu d’application Web (WAF, Web Application Firewall)](../../web-application-firewall/afds/afds-overview.md) est une fonctionnalité d’Application Gateway qui protège vos applications web de manière centralisée contre les vulnérabilités et exploitations courantes. WAF suit les règles des [ensembles de règles de base OWASP (Open Web Application Security Project)](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 ou 2.2.9.

## <a name="monitor-the-performance-of-your-applications"></a>Surveiller les performances de vos applications
La surveillance consiste à collecter et analyser des données afin de déterminer les performances, l’intégrité et la disponibilité de votre application. Une stratégie de surveillance efficace permet d’identifier le fonctionnement détaillé des composants de votre application. Elle vous permet d’augmenter votre temps d’activité en vous avertissant des problèmes critiques afin que vous puissiez les résoudre avant qu’ils ne deviennent effectifs. Elle vous aide également à détecter les anomalies susceptibles d’être liées à la sécurité.

Utilisez [Azure Application Insights](https://azure.microsoft.com/documentation/services/application-insights) pour surveiller la disponibilité, les performances et l’utilisation de votre application, qu’elle soit hébergée dans le cloud ou localement. En utilisant Application Insights, vous pouvez rapidement identifier et diagnostiquer les erreurs dans votre application sans attendre qu’un utilisateur ne les signale. Grâce aux informations recueillies, vous pouvez prendre des décisions avisées quant à la maintenance et à l’amélioration de votre application.

Application Insights dispose d’outils complets pour interagir avec les données qu’il collecte. Application Insights stocke ses données dans un référentiel commun. Il peut tirer parti des fonctionnalités partagées telles que les alertes, les tableaux de bord et une analyse approfondie grâce au langage de requête du service Kusto.

## <a name="perform-security-penetration-testing"></a>Effectuer des tests d’intrusion sécurisés
Le fait de valider les défenses de sécurité est aussi important que de tester toute autre fonctionnalité. Intégrez les [tests d’intrusion](pen-testing.md) à vos processus de génération et de déploiement. Planifiez régulièrement des tests de sécurité et des analyses de vulnérabilité sur les applications déployées, et surveillez les ports ouverts, les points de terminaison et les attaques.

Les tests à données aléatoires (fuzzing) sont une méthode de recherche des défaillances de programmes (erreurs de code) permettant de fournir les données d’entrée incorrectes aux interfaces de programme (points d’entrée) qui analysent et utilisent ces données. La [détection des risques de sécurité Microsoft](https://www.microsoft.com/en-us/security-risk-detection/) est un outil informatique que vous pouvez utiliser pour rechercher des bogues et d’autres vulnérabilités de sécurité dans votre logiciel avant de le déployer vers Azure. L’outil est conçu pour intercepter les vulnérabilités avant de déployer le logiciel sans avoir besoin de corriger un bogue, de gérer les incidents ou de répondre à une attaque une fois que le logiciel est mis en production.


## <a name="next-steps"></a>Étapes suivantes
Dans cet article, nous avons vu les avantages d’un déploiement PaaS Azure et les bonnes pratiques en matière de sécurité relatives aux applications cloud. Découvrez ensuite les pratiques recommandées pour sécuriser vos solutions PaaS web et mobiles PaaS à l’aide de services Azure spécifiques. Nous allons commencer par Azure App Service, Azure SQL Database, Azure Synapse Analytics et Stockage Azure. Des liens seront fournis dans la liste suivante quand des articles sur les pratiques recommandées pour d’autres services Azure seront disponibles :

- [Azure App Service](paas-applications-using-app-services.md)
- [Azure SQL Database et Azure Synapse Analytics](paas-applications-using-sql.md)
- [Stockage Azure](paas-applications-using-storage.md)
- Cache Azure pour Redis
- Azure Service Bus
- Pare-feu d’applications web

Consultez [Développer des applications sécurisées sur Azure](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/) pour les questions de sécurité et les contrôles que vous devez prendre en compte à chaque phase du cycle de vie du développement logiciel lors du développement d’applications pour le cloud.

Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.

Les ressources suivantes fournissent des informations générales sur la sécurité Azure et les services Microsoft associés :
* [Blog de l’équipe de sécurité Azure](/archive/blogs/azuresecurity/) : pour obtenir des informations à jour sur les dernières actualités sur la sécurité Azure
* [Centre de réponse aux problèmes de sécurité Microsoft](https://technet.microsoft.com/library/dn440717.aspx) : emplacement où les vulnérabilités de sécurité Microsoft, dont les problèmes rencontrés avec Azure, peuvent être rapportées ou signalées par e-mail à l’adresse secure@microsoft.com