---
title: Fonctionnalités techniques de la sécurité dans Azure - Microsoft Azure
description: Introduction aux services de sécurité d’Azure qui vous aident à protéger vos données, ressources et applications dans le cloud.
services: security
author: terrylanfear
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.date: 07/13/2020
ms.author: terrylan
ms.openlocfilehash: 575486a89cd078325b76fa684ff9bd3d59a619c9
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844265"
---
# <a name="azure-security-technical-capabilities"></a>Fonctionnalités techniques de la sécurité Azure
Cet article fournit une introduction aux services de sécurité d’Azure qui vous aident à protéger vos données, vos ressources et vos applications dans le cloud et à répondre aux besoins de sécurité de votre entreprise.

## <a name="azure-platform"></a>Plateforme Azure

[Microsoft Azure](https://azure.microsoft.com/overview/what-is-azure/) est une plateforme cloud, composée de services d’infrastructure et d’application, avec des services de données intégrés et une analytique poussée, ainsi que des outils et des services de développement, tous hébergés dans les centres de données du cloud public de Microsoft. Les clients utilisent la plateforme Azure pour un grand nombre de fonctionnalités et de scénarios différents : calcul de base, mise en réseau, stockage, services d’application web et mobiles, sans oublier les scénarios entièrement basés sur le cloud (par exemple l’Internet des objets). La plateforme peut être utilisée avec les technologies open source et déployée en tant que cloud hybride à moins d’être hébergée dans le centre de données d’un client. Azure fournit des technologies cloud qui permettent aux entreprises de réduire leurs coûts, d’innover rapidement et de gérer les systèmes de façon proactive. Lorsque vous générez ou faites migrer des ressources informatiques vers un fournisseur de cloud, vous comptez sur les capacités de cette organisation à protéger vos applications et vos données, avec les services et les contrôles qu’elle vous fournit pour gérer la sécurité de vos ressources cloud.

Microsoft Azure est le seul fournisseur de cloud computing à offrir une plateforme d’applications sécurisée et homogène et une infrastructure IaaS pour que les équipes travaillent selon leurs différentes compétences de cloud computing et niveaux de complexité de projets, avec des services de données intégrés et une analytique qui révèlent l’intelligence des données où qu’elles se trouvent, dans les plateformes Microsoft et non-Microsoft, dans des outils et des frameworks ouverts, en donnant le choix d’intégrer le cloud à l’environnement local et de déployer des services cloud Azure dans des centres de données locaux. Dans le cadre de la solution Microsoft Trusted Cloud, les clients comptent sur Azure et sur sa sécurité, sa fiabilité, sa conformité, sa confidentialité et son vaste réseau de personnes, de partenaires et de processus leaders sur le marché pour prendre en charge les organisations dans le cloud.

Avec Microsoft Azure, vous pouvez :

- Innover plus vite avec le cloud.

- Obtenir des insights pour dynamiser les décisions et les applications métier.

- Créer sans limite et déployer où que vous soyez.

- Protéger vos activités.

## <a name="security-technical-capabilities-to-fulfil-your-responsibility"></a>Fonctionnalités techniques de sécurité permettant d’honorer vos responsabilités

Microsoft Azure fournit des services qui vous permettent de répondre aux besoins de sécurité, de confidentialité et de conformité. L’illustration suivante explique les différents services Azure mis à votre disposition pour créer une infrastructure d’application sécurisée et conforme, s’appuyant sur des normes industrielles.

![Fonctionnalités techniques de sécurité disponibles - Vue d’ensemble](./media/technical-capabilities/azure-security-technical-capabilities-fig1.png)

## <a name="manage-and-control-identity-and-user-access"></a>Gérer et contrôler l’identité et l’accès utilisateur

Azure vous aide à protéger les informations professionnelles et personnelles en vous permettant de gérer les informations d’identification et les identités des utilisateurs, et d’en contrôler l’accès.

### <a name="azure-active-directory"></a>Azure Active Directory

Les solutions de gestion des identités et des accès de Microsoft aident les services informatiques à protéger l’accès aux applications et aux ressources dans le centre de données de l’entreprise, mais aussi dans le cloud. Elles activent des niveaux supplémentaires de validation comme l’authentification multifacteur et les stratégies d’accès conditionnel. En surveillant les activités suspectes via les fonctions avancées de reporting, d’audit et d’alertes de sécurité, vous êtes en mesure de limiter les problèmes de sécurité potentiels. [Azure Active Directory Premium](../../active-directory/fundamentals/active-directory-whatis.md) fournit une authentification unique à des milliers d’applications cloud et assure un accès aux applications web que vous exécutez en local.

Azure Active Directory (Azure AD) présente de nombreux avantages en termes de sécurité :

- Création et gestion d’une identité unique pour chaque utilisateur de l’entreprise hybride, tout en maintenant la synchronisation des utilisateurs, des groupes et des appareils

- Accès par authentification unique à vos applications, notamment à des milliers d’applications SaaS pré-intégrées

- Sécurisation de l’accès aux applications en appliquant une authentification multifacteur basée sur des règles pour les applications aussi bien locales que cloud

- Accès à distance sécurisé aux applications web locales via le proxy d’application Azure AD

Le [portail Azure Active Directory](https://aad.portal.azure.com/) fait partie du portail Azure. À partir de ce tableau de bord, vous pouvez obtenir une vue d’ensemble de l’état de votre organisation et gérer facilement l’accès aux annuaires, aux utilisateurs ou aux applications.

![Azure Active Directory](./media/technical-capabilities/azure-security-technical-capabilities-fig2.png)

Voici les principales fonctionnalités de gestion des identités Azure :

- Authentification unique

- Authentification multifacteur

- Surveillance de la sécurité, alertes et rapports Machine Learning

- Gestion des identités et des accès des consommateurs

- Enregistrement de l’appareil

- Privileged Identity Management

- Identity Protection

#### <a name="single-sign-on"></a>Authentification unique

Avec l’[authentification unique (SSO)](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/), vous pouvez accéder à toutes les applications et à toutes les ressources dont vous avez besoin pour travailler, en vous connectant une seule fois avec un seul compte d’utilisateur. Une fois connecté, vous accédez à toutes les applications qui vous sont indispensables, sans avoir besoin de vous réauthentifier (par exemple, en tapant un mot de passe).

De nombreuses organisations s'appuient sur des applications SaaS telles que Microsoft 365, Box et Salesforce pour accroître la productivité des utilisateurs finaux. Historiquement, le personnel informatique devait créer et mettre à jour chaque compte d’utilisateur dans chaque application SaaS et les utilisateurs devaient mémoriser un mot de passe pour chaque application SaaS.

[Azure AD étend Active Directory local dans le cloud](../../active-directory/manage-apps/what-is-single-sign-on.md), ce qui permet aux utilisateurs d’utiliser leur compte professionnel principal, non seulement pour se connecter à leurs appareils liés au domaine et aux ressources de l’entreprise, mais aussi à toutes les applications SaaS et web nécessaires à leur travail.

Non seulement les utilisateurs n’ont plus besoin de gérer plusieurs noms d’utilisateur et mots de passe, mais l’accès aux applications peut être automatiquement activé ou désactivé en fonction des groupes de l’organisation et de leur statut d’employé. [Azure AD ajoute des contrôles de sécurité et de gouvernance de l’accès](../../active-directory/manage-apps/view-applications-portal.md) qui vous permettent de gérer de manière centralisée l’accès des utilisateurs sur les différentes applications SaaS.

#### <a name="multi-factor-authentication"></a>Authentification multifacteur

[Azure AD Multi-Factor Authentication (MFA)](../../active-directory/authentication/concept-mfa-howitworks.md) est une méthode d’authentification qui nécessite l’utilisation de plusieurs méthodes de vérification et ajoute une deuxième couche essentielle de sécurité aux connexions et transactions des utilisateurs. [MFA contribue à sécuriser](../../active-directory/authentication/concept-mfa-howitworks.md) l’accès aux données et aux applications tout en répondant à la demande des utilisateurs souhaitant un processus d’authentification simple. Cette méthode fournit une authentification forte par le biais de diverses options de vérification : appel téléphonique, SMS, notification par application mobile ou code de vérification et jetons OAuth tiers.

#### <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>Surveillance de la sécurité, alertes et rapports Machine Learning

Vous pouvez protéger votre entreprise grâce à la surveillance de la sécurité, aux alertes et aux rapports Machine Learning qui identifient les comportements d’accès incohérents. Vous pouvez utiliser les rapports d'accès et d'utilisation Azure Active Directory pour obtenir une visibilité complète sur l'intégrité et la sécurité du répertoire de votre société. Grâce à ces informations, un administrateur de répertoire est capable de déterminer plus précisément les risques de sécurité potentiels et donc de les atténuer au maximum.

Dans le portail Azure ou le [portail Azure Active Directory](https://aad.portal.azure.com/), les [rapports](../../active-directory/reports-monitoring/overview-reports.md) sont classés comme suit :

- Rapports d’anomalies : contiennent les événements de connexion qui peuvent nous sembler anormaux. Notre objectif est de vous faire part de ces activités et de vous permettre de décider si un événement est suspect.

- Rapports d’application intégrée : fournissent des indications sur l’utilisation des applications cloud au sein de votre organisation. Azure Active Directory permet d’intégrer des milliers d'applications du cloud.

- Rapports d’erreurs : indiquent les erreurs qui peuvent survenir lors de la configuration de comptes sur des applications externes.

- Rapports propres à l’utilisateur : affichent les données d’activité relatives aux appareils et connexions d’un utilisateur spécifique.

- Journaux d’activité : contiennent un enregistrement de tous les événements audités durant les dernières 24 heures, les derniers 7 jours ou les derniers 30 jours, des modifications d’activité de groupes, et des activités d’enregistrement et de réinitialisation de mot de passe.

#### <a name="consumer-identity-and-access-management"></a>Gestion des identités et des accès des consommateurs

Pour les applications utilisées par les consommateurs, [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) offre un service de gestion de l’identité, global et hautement disponible, qui s’adapte à des centaines de millions d’identités. Le service peut être intégré sur l’ensemble des plateformes web et mobiles. Vos consommateurs peuvent se connecter à toutes vos applications via une expérience personnalisable en utilisant leurs comptes de réseaux sociaux existants ou en créant des comptes avec de nouvelles informations d’identification.

Auparavant, les développeurs d’applications qui souhaitaient [inscrire et connecter les consommateurs](../../active-directory-b2c/overview.md) à leurs applications auraient écrit leur propre code. Et ils auraient utilisé des systèmes ou des bases de données locaux pour stocker les noms d'utilisateur et les mots de passe. Azure Active Directory B2C offre à votre organisation un meilleur moyen d’intégrer la gestion des identités des consommateurs dans vos applications grâce à une plateforme sécurisée reposant sur des normes et à un ensemble complet de stratégies extensibles.

Lorsque vous utilisez Azure Active Directory B2C, vos consommateurs peuvent s’inscrire auprès de vos applications à l’aide de leurs comptes sociaux existants (Facebook, Google, Amazon, LinkedIn) ou en créant des informations d’identification (adresse de messagerie et mot de passe, ou nom d’utilisateur et mot de passe).

#### <a name="device-registration"></a>Enregistrement de l’appareil

[L’inscription d’appareil Azure AD](../../active-directory/devices/overview.md) constitue la base des scénarios [d’accès conditionnel](../../active-directory/devices/overview.md) basé sur les appareils. Lors de l’inscription d’un appareil, Azure AD Device Registration fournit une identité à l’appareil qui sera utilisée pour l’authentifier lors de la connexion de l’utilisateur. L’appareil authentifié et ses attributs peuvent alors être utilisés pour appliquer des stratégies d’accès conditionnel pour les applications qui sont hébergées sur le cloud et localement.

Quand ils sont associés à une solution de [gestion des appareils mobiles (MDM)](https://www.microsoft.com/itshowcase/Article/Content/588/Mobile-device-management-at-Microsoft) comme Intune, les attributs de l’appareil dans Azure Active Directory sont mis à jour avec des informations supplémentaires sur l’appareil. Cela vous permet de créer des règles d’accès conditionnel qui imposent que l’accès à partir des appareils réponde à vos critères de sécurité et de conformité.

#### <a name="privileged-identity-management"></a>Privileged Identity Management

Le service [Azure Active Directory (AD) Privileged Identity Management](../../active-directory/privileged-identity-management/pim-configure.md) vous permet de gérer, de contrôler et de surveiller vos identités privilégiées et l'accès aux ressources dans Azure AD et dans d'autres services en ligne Microsoft, tels que Microsoft 365 ou Microsoft Intune.

Les utilisateurs doivent parfois effectuer des opérations privilégiées dans des ressources Azure ou Microsoft 365, ou dans d'autres applications SaaS. Cela signifie souvent que les entreprises doivent leur donner un accès privilégié permanent à Azure AD. C’est un risque de sécurité croissant pour les ressources hébergées dans le cloud, car les entreprises ne peuvent pas suffisamment surveiller ce que ces utilisateurs font avec leurs privilèges d'administrateur. En outre, si un compte d’utilisateur disposant d’un accès privilégié est compromis, cette seule faille peut affecter sa sécurité globale sur le cloud. Azure AD Privileged Identity Management contribue à minimiser ce risque.

Grâce à Azure AD Privileged Identity Management, vous pouvez :

- Identifier les utilisateurs qui ont un rôle d’administrateur dans Azure AD

- Activer à la demande un accès administrateur « juste à temps » aux services Microsoft Online Services comme Microsoft 365 et Intune

- Obtenir des rapports sur l'historique des accès administrateur et sur les modifications apportées aux affectations de l'administrateur

- Recevoir des alertes sur l'accès à un rôle privilégié

#### <a name="identity-protection"></a>Identity Protection

[Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md) est un service de sécurité offrant une vue centralisée des détections des risques et des vulnérabilités potentielles qui affectent les identités de votre organisation. Identity Protection utilise des fonctionnalités existantes de détection des anomalies d’Azure Active Directory (disponibles via les rapports d’activités anormales d’Azure AD) et introduit de nouveaux types de détections des risque capables de détecter les anomalies en temps réel.

## <a name="secure-resource-access"></a>Sécuriser l’accès aux ressources

Le contrôle des accès dans Azure s’envisage d’abord dans une perspective de facturation. Le propriétaire d’un compte Azure accessible par le biais du [Centre des comptes Azure](https://account.windowsazure.com/subscriptions) est l’administrateur de compte. Les abonnements sont un conteneur de facturation, mais ils constituent également une limite de sécurité : chaque abonnement a un administrateur de service qui peut ajouter, supprimer et modifier des ressources Azure dans cet abonnement à l’aide du portail Azure. L'administrateur de sécurité par défaut d'un nouvel abonnement est l'administrateur de compte, mais ce dernier peut modifier l'administrateur de sécurité dans le Centre des comptes Azure.

![Accès des ressources sécurisées dans Azure](./media/technical-capabilities/azure-security-technical-capabilities-fig3.png)

Les abonnements sont également associés à un répertoire. Le répertoire définit un ensemble d'utilisateurs. Ceux-ci peuvent être des utilisateurs professionnels ou scolaires qui ont créé l’annuaire, ou bien des utilisateurs externes (autrement dit, des comptes Microsoft). Les abonnements sont accessibles par un sous-ensemble des utilisateurs du répertoire qui ont été affectés comme administrateur de service ou comme coadministrateur ; la seule exception est que, pour des raisons d'héritage, les comptes Microsoft (anciennement Windows Live ID) peuvent être affectés comme administrateur de service ou comme co-admistrateur sans être présents dans le répertoire.

Les entreprises orientées sécurité doivent s’efforcer de donner aux employés les autorisations exactes dont ils ont besoin. Un trop grand nombre d’autorisations peut exposer un compte aux attaquants. Si le nombre d’autorisations est trop faible, les employés ne peuvent pas effectuer leur travail efficacement. Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) permet de résoudre ce problème en offrant une gestion précise de l’accès pour Azure.

![Accès des ressources sécurisées](./media/technical-capabilities/azure-security-technical-capabilities-fig4.png)

Avec Azure RBAC, vous pouvez séparer les tâches au sein de votre équipe et accorder aux utilisateurs uniquement les accès nécessaires pour accomplir leur travail. Plutôt que de donner à tous des autorisations illimitées au sein de votre abonnement ou de vos ressources Azure, vous pouvez autoriser uniquement certaines actions. Par exemple, vous pouvez utiliser Azure RBAC pour permettre à un employé donné de gérer les machines virtuelles dans un abonnement, tandis qu’un autre pourra gérer les bases de données SQL au sein du même abonnement.

![Accès aux ressources sécurisées à l’aide d’Azure RBAC](./media/technical-capabilities/azure-security-technical-capabilities-fig5.png)

## <a name="data-security-and-encryption"></a>Chiffrement et sécurité des données

Pour assurer la protection des données dans le cloud, l’un des facteurs clés consiste à tenir compte des états que les données peuvent présenter, mais aussi des contrôles disponibles pour ces états. Pour les bonnes pratiques de chiffrement et de sécurité des données dans Azure, les recommandations que nous formulons s’articulent autour des états suivants des données :

- Au repos : Cela inclut tous les objets de stockage, conteneurs et types d’informations présents de manière statique sur un support physique, qu’il s’agisse d’un disque magnétique ou d’un disque optique.
- En transit : Lorsque des données sont transférées entre des composants, des emplacements ou des programmes (sur le réseau, par exemple) via un bus de service (depuis un emplacement local vers le cloud, ou vice-versa, y compris via des connexions hybrides comme ExpressRoute), ou lors d’un processus d’entrée/sortie, on parle de données « en transit ».

### <a name="encryption-at-rest"></a>Chiffrement au repos

Le chiffrement au repos est détaillé dans [Chiffrement de données Azure au repos](encryption-atrest.md).

### <a name="encryption-in-transit"></a>Chiffrement en transit

La protection des données en transit doit être un aspect essentiel de votre stratégie de protection des données. Comme les données transitent entre différents emplacements, dans les deux sens, nous vous recommandons d’utiliser systématiquement les protocoles SSL/TLS pour ces déplacements de données. Dans certains cas, vous souhaiterez isoler l’intégralité du canal de communication entre votre infrastructure locale et sur le cloud, via un réseau privé virtuel (VPN).

Pour les données qui transitent entre votre infrastructure locale et Azure, vous devez envisager le recours aux dispositifs de protection appropriés, comme HTTPS ou VPN.

Pour les organisations devant sécuriser l’accès à Azure à partir de plusieurs postes de travail locaux, utilisez la fonction [VPN de site à site d’Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-classic-portal.md).

Pour les organisations devant sécuriser l’accès à Azure à partir d’un seul poste de travail local, utilisez la fonction [VPN de point à site d’Azure](../../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md).

Les jeux de données volumineux peuvent être transmis par le biais d’une liaison réseau étendu haut débit dédiée, comme [ExpressRoute](https://azure.microsoft.com/services/expressroute/). Si vous choisissez d’utiliser ExpressRoute, vous pouvez également chiffrer les données au niveau des applications par le biais de [SSL/TLS](https://support.microsoft.com/kb/257591) ou d’autres protocoles, pour optimiser la protection.

Si vous interagissez avec Azure Storage via le portail Azure, toutes les transactions se produisent via HTTPS. L’[API de stockage REST](/rest/api/storageservices/) par le biais de HTTPS peut également être utilisée pour interagir avec [Stockage Azure](https://azure.microsoft.com/services/storage/) et [Azure SQL Database](https://azure.microsoft.com/services/sql-database/).

Les organisations qui ne parviennent pas à protéger les données en transit sont plus sensibles aux [attaques d’intercepteur](/previous-versions/office/skype-server-2010/gg195821(v=ocs.14)), aux [écoutes électroniques](/previous-versions/office/skype-server-2010/gg195641(v=ocs.14)) et au piratage de session. Ces attaques peuvent être la première étape d’un processus visant à accéder à des données confidentielles.

Vous pouvez en savoir plus sur l’option de VPN Azure en lisant l’article [Planification et conception de la passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="enforce-file-level-data-encryption"></a>Application du chiffrement des données au niveau fichier

[Azure RMS](/azure/information-protection/what-is-azure-rms) utilise des stratégies de chiffrement, d’identité et d’autorisation pour vous aider à sécuriser vos fichiers et vos e-mails. Azure RMS peut fonctionner sur plusieurs appareils (téléphones, tablettes et PC), en protégeant les données au sein de votre organisation et en dehors de cette dernière. Cette fonctionnalité est possible, car Azure RMS ajoute un niveau de protection qui reste avec les données, même lorsqu’elles quittent les limites de votre organisation.

Quand vous utilisez Azure RMS pour protéger vos fichiers, vous recourez au chiffrement standard avec prise en charge complète de la norme [FIPS 140-2](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-2.pdf). Lorsque vous tirez parti d’Azure RMS pour la protection des données, vous avez l’assurance que la protection reste avec le fichier, même s’il est copié sur un stockage qui n’est pas sous le contrôle du département informatique (service de stockage cloud, par exemple). Il en va de même pour les fichiers partagés par e-mail ; ils sont protégés en tant que pièces jointes à un message électronique. Des instructions expliquant comment ouvrir la pièce jointe protégée sont fournies.
Si vous planifiez l’adoption d’Azure RMS, nous vous recommandons de suivre les conseils ci-après :

- Installez l’[application de partage RMS](/azure/information-protection/rms-client/sharing-app-windows) Cette application s’intègre avec les applications Office en installant un complément Office, afin que les utilisateurs puissent directement protéger leurs fichiers, en toute simplicité.

- Configurez des applications et des services pour prendre en charge Azure RMS.

- Créez des [modèles personnalisés](/azure/information-protection/configure-policy-templates) qui reflètent les besoins de votre entreprise (exemple : un modèle portant sur des données ultra-secrètes, qui doit être appliqué à tous les e-mails ultra-secrets).

Les organisations bénéficiant d’une [classification](https://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf) et d’une protection insuffisantes des données sont plus vulnérables aux fuites de données. Si la protection des fichiers n’est pas appropriée, les organisations ne pourront pas optimiser leur visibilité, ni surveiller les abus ou empêcher tout accès malveillant aux fichiers.

> [!Note]
> Vous pouvez en savoir plus sur Azure RMS en lisant l’article [Prise en main d’Azure Rights Management](/azure/information-protection/requirements).

## <a name="secure-your-application"></a>Sécuriser votre application
Si Azure est responsable de la sécurisation de l’infrastructure et de la plateforme sur laquelle votre application s’exécute, il vous incombe de sécuriser votre application elle-même. En d’autres termes, vous devez développer, déployer et gérer votre code d’application et le contenu de manière sécurisée. Sinon, votre code d’application ou le contenu peut encore être vulnérable aux menaces.

### <a name="web-application-firewall"></a>Pare-feu d’application web
Le [pare-feu d’applications Web (WAF, Web Application Firewall)](../../web-application-firewall/ag/ag-overview.md) est une fonctionnalité de [Application Gateway](../../application-gateway/overview.md) qui protège vos applications web de manière centralisée contre les vulnérabilités et exploits courants.

Le pare-feu d’applications web suit les règles des [Ensembles de règles de base OWASP](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.0 ou 2.2.9. Les applications Web sont de plus en plus la cible d’attaques malveillantes qui exploitent des vulnérabilités connues. Les types d’attaques les plus courantes sont l’injection de code SQL, les attaques de script site à site, entre autres. Empêcher ces attaques dans le code d’application peut se révéler difficile et nécessiter une maintenance rigoureuse, des mises à jour correctives ainsi que la surveillance au niveau de plusieurs couches de la topologie de l’application. Un pare-feu d’applications web centralisé facilite grandement la gestion de la sécurité et offre une meilleure garantie de protection aux administrateurs de l’application contre les menaces ou les intrusions. Une solution WAF peut également réagir plus rapidement à une menace de sécurité en exécutant la mise à jour corrective d’une vulnérabilité connue dans un emplacement central plutôt que de sécuriser individuellement chacune des applications web. Les passerelles d’application existantes peuvent être facilement converties en une passerelle d’application avec un pare-feu d’applications web.

Le pare-feu d’applications web protège notamment des vulnérabilités web courantes suivantes :

- Protection contre les injections de code SQL

- Protection de l’exécution de script de site à site

- Protection contre les attaques web courante comme l’injection de commande, les dissimulations de requêtes HTTP, la séparation de réponse HTTP et les attaques RFI (Remote File Inclusion)

- Protection contre les violations de protocole HTTP

- Protection contre les anomalies de protocole HTTP comme un agent-utilisateur hôte manquant et les en-têtes Accept

- Protection contre les robots, les crawlers et les scanneurs

- Détection des erreurs de configuration d’application courantes (par exemple, Apache, IIS, etc.)

> [!Note]
> Pour une liste plus détaillée des règles et de leurs protections, consultez ces [Ensembles de règles de base](../../web-application-firewall/ag/ag-overview.md).

Azure dispose de plusieurs fonctionnalités simples d’utilisation pour aider à sécuriser le trafic entrant et sortant de votre application. Azure aide également les clients à sécuriser leur code d’application en fournissant des fonctionnalités en externe pour analyser votre application web et y rechercher des vulnérabilités.

- [Configuration de l'authentification Azure Active Directory pour votre application](https://azure.microsoft.com/blog/azure-websites-authentication-authorization/)

- [Sécurisation du trafic vers votre application avec l'activation du protocole TLS/SSL (Transport Layer Security) - HTTPS](../../app-service/configure-ssl-bindings.md)

  - [Affectation de force de tout le trafic entrant sur la connexion HTTPS](http://microsoftazurewebsitescheatsheet.info/)

  - [Activation du protocole HSTS (Strict Transport Security)](http://microsoftazurewebsitescheatsheet.info/#enable-http-strict-transport-security-hsts)

- [Restriction de l'accès à votre application selon l'adresse IP du client](http://microsoftazurewebsitescheatsheet.info/#filtering-traffic-by-ip)

- [Restriction de l'accès à votre application selon le comportement du client - fréquence des demandes et accès concurrentiel](http://microsoftazurewebsitescheatsheet.info/#dynamic-ip-restrictions)

- [Analyse du code de votre application web pour rechercher les vulnérabilités à l'aide de l'analyse Tinfoil Security](https://azure.microsoft.com/blog/web-vulnerability-scanning-for-azure-app-service-powered-by-tinfoil-security/)

- [Configuration de l'authentification mutuelle TLS pour exiger des certificats client pour la connexion à votre application web](../../app-service/app-service-web-configure-tls-mutual-auth.md)

- [Configuration d'un certificat client à utiliser à partir de votre application pour la connexion sécurisée à des ressources externes](https://azure.microsoft.com/blog/using-certificates-in-azure-websites-applications/)

- [Suppression des en-têtes standard de serveur pour éviter la prise d'empreinte de votre application par des outils](https://azure.microsoft.com/blog/removing-standard-server-headers-on-windows-azure-web-sites/)

- [Connexion sécurisée de votre application aux ressources d'un réseau privé à l'aide d'un VPN de point à site](../../app-service/web-sites-integrate-with-vnet.md)

- [Connexion sécurisée de votre application aux ressources d'un réseau privé à l'aide de connexions hybrides](../../app-service/app-service-hybrid-connections.md)

Azure App Service utilise la même solution anti-programme malveillant qu’Azure Cloud Services et que les ordinateurs virtuels. Pour en savoir plus à ce sujet, consultez notre [documentation sur les logiciels anti-programme malveillant](antimalware.md).

## <a name="secure-your-network"></a>Sécuriser votre réseau
Microsoft Azure inclut une infrastructure réseau solide pour prendre en charge les exigences de connectivité de vos applications et services. La connectivité réseau est possible entre les ressources hébergées dans Azure, entre les ressources hébergées sur site et dans Azure, mais aussi vers et à partir d’Internet et d’Azure.

L’[infrastructure réseau d’Azure](/previous-versions/azure/virtual-machines/windows/infrastructure-example) vous permet de connecter en toute sécurité les ressources Azure entre elles en utilisant des [réseaux virtuels](../../virtual-network/virtual-networks-overview.md). Un réseau virtuel est une représentation de votre propre réseau dans le cloud. Un réseau virtuel est une isolation logique du réseau cloud Azure dédié à votre abonnement. Vous pouvez connecter des réseaux virtuels à vos réseaux locaux.

![Sécuriser votre réseau (protection)](./media/technical-capabilities/azure-security-technical-capabilities-fig6.png)

Si vous avez besoin d’un contrôle d’accès au niveau du réseau de base (reposant sur l’adresse IP et les protocoles TCP ou UDP), vous pouvez utiliser des [groupes de sécurité réseau](../../virtual-network/virtual-network-vnet-plan-design-arm.md). Un groupe de sécurité réseau est un pare-feu de filtrage des paquets avec état qui vous permet de contrôler l’accès sur la base d’un algorithme à [5 tuples](https://www.techopedia.com/definition/28190/5-tuple).

La mise en réseau Azure permet de personnaliser le comportement de routage du trafic réseau sur vos réseaux virtuels Azure. Pour cela, vous pouvez configurer les [itinéraires définis par l’utilisateur](../../virtual-network/virtual-networks-udr-overview.md) dans Azure.

[tunneling forcé](https://www.petri.com/azure-forced-tunneling) est un mécanisme que vous pouvez utiliser pour empêcher vos services de se connecter aux appareils sur Internet.

Azure prend en charge la connectivité d’une liaison de réseau étendu dédiée entre votre réseau local et un réseau virtuel Microsoft Azure via [ExpressRoute](../../expressroute/expressroute-introduction.md). La liaison entre Azure et votre site se fait via une connexion dédiée qui ne passe pas par l’Internet public. Si votre application Azure s’exécute dans plusieurs centres de données, vous pouvez utiliser [Microsoft Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) pour acheminer intelligemment les requêtes des utilisateurs vers différentes instances de l’application. Vous pouvez aussi acheminer le trafic vers des services qui ne s’exécutent pas dans Azure s’ils sont accessibles par Internet.

## <a name="virtual-machine-security"></a>Sécurité de la machine virtuelle

Les [Machines virtuelles Microsoft Azure](../../virtual-machines/index.yml) vous permettent de déployer un large éventail de solutions de calcul de façon agile. Avec la prise en charge de Microsoft Windows, Linux, Microsoft SQL Server, Oracle, IBM, SAP et Azure BizTalk Services, vous pouvez déployer des charges de travail et des langages variés sur la plupart des systèmes d’exploitation.

Azure met à votre disposition des [logiciels anti-programmes malveillants](antimalware.md) provenant de fournisseurs de sécurité tels que Microsoft, Symantec, Trend Micro et Kaspersky. Ceux-ci permettent de protéger vos machines virtuelles contre les fichiers malveillants, les logiciels de publicité et d’autres menaces.

Microsoft Antimalware pour Azure Cloud Services et Machines virtuelles offre une fonctionnalité de protection en temps réel qui permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Microsoft Antimalware fournit des alertes configurables lorsqu’un logiciel malveillant ou indésirable connu tente de s’installer ou de s’exécuter sur vos systèmes Azure.

[Sauvegarde Azure](../../backup/backup-overview.md) est une solution évolutive qui protège les données de vos applications, sans investissement en capital et avec des frais de fonctionnement minimaux. Les erreurs rencontrées par les applications peuvent endommager vos données et les erreurs humaines peuvent introduire des bogues dans vos applications. Avec Sauvegarde Azure, vos machines virtuelles exécutant Windows et Linux sont protégées.

[Azure Site Recovery](../../site-recovery/site-recovery-overview.md) aide à coordonner la réplication, le basculement et la récupération des charges de travail et des applications afin qu’elles soient disponibles à partir d’un site secondaire si votre site principal tombe en panne.

## <a name="ensure-compliance-cloud-services-due-diligence-checklist"></a>Garantissez la conformité : Liste de vérification d’obligation de diligence des services cloud

Microsoft a développé la [liste des points à vérifier avec une prudence mesurée pour les services cloud](https://aka.ms/cloudchecklist.download), afin d’aider les organisations à faire preuve d’une sage circonspection lorsqu’elles envisagent une migration vers le cloud. Cette liste fournit une structure à une organisation, quels que soient sa taille et son type — entreprises privées et organisations du secteur public, notamment tous les niveaux de l’administration et les associations —, en lui permettant d’identifier ses propres performances, services, gestion des données, de même que ses exigences et objectifs de gouvernance. L’organisation peut alors comparer les offres de divers fournisseurs de services cloud tout en jetant en définitive les bases d’un contrat de service cloud.

Cette liste de vérifications préalables fournit un cadre qui s’aligne précisément sur les clauses d’une nouvelle norme internationale des contrats de service cloud, la norme ISO/IEC 19086. Cette norme offre un ensemble unifié de réflexions, afin d’aider les organisations à prendre des décisions concernant leur adoption du cloud, tout en constituant une base commune qui permet la comparaison entre les offres de service cloud.

La liste de vérifications préalables favorise une migration vers le cloud soigneusement étudiée, en apportant une aide structurée et une approche aussi cohérente que reproductible qui permet de choisir un fournisseur de services cloud.

Opter pour le cloud ne relève plus d’un choix purement technologique. Attendu que les conditions requises de la liste de vérifications préalables abordent tous les aspects d’une organisation, celles-ci servent à réunir l’ensemble des principaux décideurs internes : le directeur des systèmes d’information et le chef de la sécurité informatique, ainsi que des juristes, des acheteurs professionnels, des spécialistes en gestion des risques et en conformité. L’efficacité des processus de prise de décision s’en trouve accrue, et l’enracinement des décisions dans un raisonnement sain est renforcé, ce qui réduit la probabilité de blocages imprévus sur le chemin menant à l’adoption.

Par ailleurs, la liste de vérifications préalables :

- Expose les points de discussion fondamentaux qui se posent aux décideurs dès le début du processus d’adoption du cloud

- Alimente les débats approfondis au sein de l’entreprise sur la réglementation et les objectifs propres de l’organisation en matière de confidentialité, d’informations d’identification personnelle et de sécurité des données

- Aide les organisations à identifier les problèmes potentiels qui pourraient avoir un impact sur un projet cloud

- Fournit une série logique de questions, avec les mêmes conditions, définitions, mesures et résultats pour chaque fournisseur, afin de simplifier le processus de comparaison des offres émanant de différents fournisseurs de services cloud

## <a name="azure-infrastructure-and-application-security-validation"></a>Validation de la sécurité des applications et de l’infrastructure Azure

[Azure Operational Security](operational-security.md) comprend les services, contrôles et fonctionnalités auxquels les utilisateurs ont accès pour protéger leurs données, leurs applications et d’autres ressources dans Microsoft Azure.

![Validation de la sécurité (détection)](./media/technical-capabilities/azure-security-technical-capabilities-fig7.png)

Azure Operational Security repose sur un framework qui intègre les connaissances acquises par le biais de diverses possibilités spécifiques à Microsoft, notamment Microsoft Security Development Lifecycle (SDL), le programme Microsoft Security Response Center et une connaissance approfondie des cybermenaces.

### <a name="microsoft-azure-monitor"></a>Microsoft Azure Monitor

[Azure Monitor](../../azure-monitor/index.yml) est la solution de gestion informatique pour le cloud hybride. Utilisés seuls ou pour étendre votre déploiement System Center existant, les journaux Azure Monitor vous donnent le maximum de flexibilité et de contrôle pour gérer votre infrastructure sur le cloud.

![Azure Monitor](./media/technical-capabilities/azure-security-technical-capabilities-fig8.png)

Avec Azure Monitor, vous pouvez gérer n’importe quelle instance dans n’importe quel cloud, notamment local, Azure, AWS, Windows Server, Linux, VMware et OpenStack, à moindre coût par rapport aux solutions concurrentes. Conçu pour les environnements cloud, Azure Monitor offre une nouvelle approche de la gestion de votre entreprise, qui constitue le moyen le plus rapide et le plus économique pour répondre aux nouveaux défis métiers et pour gérer les nouvelles charges de travail, applications et environnements cloud.

### <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Les [journaux Azure Monitor](https://azure.microsoft.com/documentation/services/log-analytics) assurent des services de surveillance en collectant les données des ressources managées et en les regroupant dans un référentiel central. Ces données peuvent comprendre des événements, des données de performances ou des données personnalisées fournies par le biais de l’API. Une fois collectées, les données sont disponibles pour les fonctions de génération d’alertes, d’analyse et d’exportation.

![Journaux d’activité Azure Monitor](./media/technical-capabilities/azure-security-technical-capabilities-fig9.png)

Cette méthode vous permet de regrouper les données issues de différentes sources, et de combiner des données de vos services Azure avec votre environnement local existant. En outre, cette approche dissocie clairement la collecte des données de l’exécution d’actions sur ces dernières, de sorte que toutes les actions sont disponibles sur tous les types de données.

### <a name="azure-security-center"></a>Azure Security Center

[Azure Security Center](../../security-center/security-center-introduction.md) vous aide à prévenir, détecter et résoudre les menaces grâce à une visibilité et un contrôle accrus de la sécurité de vos ressources Azure. Il fournit une surveillance de la sécurité et une gestion des stratégies intégrées pour l’ensemble de vos abonnements Azure, vous aidant ainsi à détecter les menaces qui pourraient passer inaperçues. De plus, il est compatible avec un vaste écosystème de solutions de sécurité.

Le Centre de sécurité analyse l’état de sécurité de vos ressources Azure pour identifier les vulnérabilités potentielles. Une liste de recommandations vous guide tout au long du processus de configuration des contrôles nécessaires.

Voici quelques exemples :

- Approvisionnement d’un logiciel anti-programme malveillant pour identifier et supprimer les programmes malveillants

- Configuration de groupes de sécurité réseau et de règles pour contrôler le trafic vers les machines virtuelles

- Approvisionnement de pare-feu d’applications web pour protéger vos applications web contre les attaques ciblées

- Déploiement de mises à jour système manquantes

- Correction des configurations de système d’exploitation qui ne suivent pas les recommandations

Le Centre de sécurité collecte, analyse et intègre automatiquement les données du journal à partir de vos ressources Azure, du réseau et des solutions partenaires telles que les logiciels anti-programme malveillant et les pare-feu. Quand des menaces sont détectées, une alerte de sécurité est créée. Voici quelques exemples de détections :

- Des machines virtuelles compromises qui communiquent avec des adresses IP connues comme étant malveillantes

- Des programmes malveillants avancés qui sont détectés à l’aide du rapport d’erreurs Windows

- Des attaques par force brute contre des machines virtuelles

- Des alertes de sécurité de logiciels anti-programme malveillant et de pare-feu intégrés

### <a name="azure-monitor"></a>Azure Monitor

[Azure Monitor](../../azure-monitor/overview.md) propose des références à des informations sur des types de ressources spécifiques. Il offre des services de visualisation, requête, routage, alertes, mise à l’échelle et automatisation pour les données de l’infrastructure Azure (journal d’activité) et pour chaque ressource Azure (journaux de diagnostic).

Les applications cloud sont complexes, et se composent de nombreux éléments mobiles. L’analyse fournit des données visant à garantir que votre application reste opérationnelle et soit exécutée en toute intégrité. Elle vous permet également de parer à des problèmes potentiels ou de résoudre des problèmes déjà survenus.

![Diagramme montrant comment vous pouvez utiliser les données de surveillance pour obtenir des informations détaillées sur votre application.](./media/technical-capabilities/azure-security-technical-capabilities-fig10.png)
En outre, vous pouvez utiliser les données d’analyse pour obtenir des informations détaillées sur votre application. Ces connaissances peuvent vous aider à améliorer les performances de l’application ou sa facilité de gestion, ou à automatiser des actions qui exigeraient normalement une intervention manuelle.

L’audit de sécurité de votre réseau est essentiel pour détecter ses vulnérabilités et assurer la conformité avec votre modèle de gouvernance réglementaire et de sécurité informatique. Grâce à l’affichage du groupe de sécurité, vous pouvez récupérer le groupe de sécurité réseau configuré et ses règles de sécurité, ainsi que les règles de sécurité en vigueur. Avec la liste des règles appliquées, vous pouvez identifier les ports ouverts et évaluer la vulnérabilité du réseau.

### <a name="network-watcher"></a>Network Watcher

[Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md) est un service régional qui vous permet de surveiller et de diagnostiquer l’état au niveau d’un réseau dans, vers et depuis Azure. Les outils de visualisation et de diagnostic réseau disponibles avec Network Watcher vous aident à comprendre, diagnostiquer et obtenir des informations sur votre réseau dans Azure. Ce service inclut la capture de paquets, le tronçon saut suivant, la vérification des flux IP, l’affichage de groupe de sécurité, les journaux de flux de groupe de sécurité réseau. La surveillance basée sur des scénarios fournit une vue de bout en bout des ressources réseau, à l’inverse de la surveillance des ressources réseau qui s’intéresse à chaque ressource spécifique.

### <a name="storage-analytics"></a>Storage analytics

[Storage Analytics](/rest/api/storageservices/fileservices/storage-analytics) peut stocker des mesures qui comprennent les statistiques de transactions agrégées et les données de capacité relatives aux demandes adressées à un service de stockage. Les transactions sont indiquées au niveau de l'opération d'API ainsi qu'au niveau du service de stockage, et la capacité est indiquée au niveau du service de stockage. Les données de métriques peuvent être utilisées pour analyser l'utilisation du service de stockage, diagnostiquer les problèmes au niveau des demandes effectuées auprès du service de stockage et améliorer les performances des applications qui utilisent un service.

### <a name="application-insights"></a>Application Insights

[Application Insights](../../azure-monitor/app/app-insights-overview.md) est un service extensible de gestion des performances des applications (APM) destiné aux développeurs web sur de multiples plateformes. Utilisez-le pour analyser votre application web en direct. Ce service détecte automatiquement les problèmes de performances. Il intègre de puissants outils d’analyse pour vous aider à diagnostiquer les problèmes et à comprendre ce que font les utilisateurs avec votre application. Il a été conçu pour vous permettre d’améliorer continuellement les performances et la convivialité. Il fonctionne avec des applications sur un large éventail de plates-formes, notamment .NET, Node.js et Java EE, hébergées sur site ou dans le cloud. Il s’intègre à votre processus DevOps et offre des points de connexion à divers outils de développement.

Il analyse les éléments suivants :

- **Taux de demandes, temps de réponse et taux d’échec** : identifiez les pages les plus consultées, à quel moment de la journée, et déterminez où se trouvent vos utilisateurs. Identifiez les pages qui offrent les meilleures performances. Si vos temps de réponse et votre taux d’échec augmentent lorsqu’il y a plus de requêtes, vous avez peut-être un problème de ressources.

- **Taux de dépendance, temps de réponse et taux d’échec** : déterminez si des services externes vous ralentissent.

- **Exceptions** : analysez les statistiques agrégées, ou choisissez des instances en particulier et explorez l’arborescence des appels de procédure et les requêtes connexes. Les exceptions de serveur et de navigateur sont signalées.

- **Consultations de pages et performances de chargement** : indiquées par le navigateur de vos utilisateurs.

- **Appels AJAX à partir de pages web** : taux, temps de réponse et taux d’échec.

- **Nombre de sessions et d’utilisateurs**.

- **Compteurs de performances** de vos ordinateurs serveurs Windows ou Linux, par exemple le processeur, la mémoire et l’utilisation du réseau.

- **Diagnostics d’hébergement** de Docker ou Azure.

- **Journaux d’activité de suivi des diagnostics** de votre application : pour pouvoir mettre en corrélation les événements de suivi avec les demandes.

- **Mesures et événements personnalisés**, vous les écrivez vous-même dans le code client ou serveur, pour effectuer le suivi des événements commerciaux, tels que les articles vendus ou les matchs gagnés.

L’infrastructure de votre application est généralement constituée de plusieurs composants (peut-être une machine virtuelle, un compte de stockage et un réseau virtuel ou une application web, une base de données, un serveur de base de données et 3 services de tiers). Vous ne voyez pas ces composants comme des entités distinctes, mais plutôt comme des parties associées et interdépendantes d’une seule et même entité. Vous avez alors besoin de regrouper le déploiement, la gestion et la surveillance de ces différentes parties. [Azure Resource Manager](../../azure-resource-manager/management/overview.md) vous permet de travailler avec les ressources de votre solution en tant que groupe.

Vous pouvez déployer, mettre à jour ou supprimer toutes les ressources de votre solution dans le cadre d’une opération unique et coordonnée. Vous utilisez un modèle de déploiement pouvant fonctionner avec différents environnements (environnements de test, intermédiaire et de production). Le gestionnaire de ressources assure la sécurité, les fonctions d’audit et de balisage pour vous aider à gérer vos ressources après le déploiement.

**Avantages liés à l’utilisation de Resource Manager**

Resource Manager offre plusieurs avantages :

- Vous pouvez déployer, gérer et surveiller toutes les ressources de votre solution comme un groupe, plutôt que de les gérer individuellement.

- Vous pouvez déployer votre solution à plusieurs reprises tout au long du cycle de vie de développement et avoir ainsi l’assurance que vos ressources présentent un état cohérent lors de leur déploiement.

- Vous pouvez gérer votre infrastructure à l’aide de modèles déclaratifs plutôt que de scripts.

- Vous pouvez définir les dépendances entre les ressources, de façon à les déployer dans le bon ordre.

- Vous pouvez appliquer le contrôle d’accès à tous les services dans votre groupe de ressources, car le contrôle d’accès en fonction du rôle Azure (Azure RBAC) est intégré en mode natif à la plateforme de gestion.

- Vous pouvez appliquer des balises aux ressources pour organiser logiquement toutes les ressources de votre abonnement.

- Vous pouvez clarifier la facturation de votre organisation en affichant les coûts d’un groupe de ressources partageant la même balise.

> [!Note]
> Resource Manager propose une nouvelle façon de déployer et de gérer vos solutions. Si vous avez utilisé un modèle de déploiement antérieur et souhaitez obtenir des informations sur les modifications, consultez [Présentation du déploiement de Resource Manager et du déploiement classique](../../azure-resource-manager/management/deployment-models.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la sécurité, lisez nos rubriques détaillées sur la sécurité :

- [Audit et journalisation](https://www.microsoft.com/en-us/trustcenter/security/auditingandlogging)

- [Cybercrime](https://www.microsoft.com/en-us/trustcenter/security/cybercrime)

- [Conception et sécurité opérationnelle](https://www.microsoft.com/en-us/trustcenter/security/designopsecurity)

- [Chiffrement](https://www.microsoft.com/en-us/trustcenter/security/encryption)

- [Gestion de l’identité et de l’accès](https://www.microsoft.com/en-us/trustcenter/security/identity)

- [Sécurité du réseau](https://www.microsoft.com/en-us/trustcenter/security/networksecurity)

- [Gestion des menaces](https://www.microsoft.com/en-us/trustcenter/security/threatmanagement)