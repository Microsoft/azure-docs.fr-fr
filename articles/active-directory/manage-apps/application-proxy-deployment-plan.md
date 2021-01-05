---
title: Planifier un déploiement de proxy d’application Azure Active Directory
description: Guide complet sur la planification du déploiement d’un proxy d’application au sein de votre organisation
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 12/31/2020
ms.author: kenwith
ms.openlocfilehash: 08289987dac81905584ee65b808cfcbf35525728
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97835140"
---
# <a name="plan-an-azure-ad-application-proxy-deployment"></a>Planifier le déploiement d’un proxy d’application Azure AD

Le proxy d’application Azure Active Directory (Azure AD) est une solution d’accès à distance simple, sécurisée et économique, destinée aux applications locales. Il fournit aux organisations « cloud first » un chemin de transition direct leur permettant de gérer l’accès aux applications locales héritées qui ne sont pas encore capables d’utiliser des protocoles modernes. Pour obtenir des informations supplémentaires, consultez [Qu’est-ce qu’un proxy d’application ?](./application-proxy.md).

Le proxy d’application est l’outil recommandé pour permettre aux utilisateurs distants d’accéder aux ressources internes. Le proxy d’application évite d’avoir recours à un VPN ou à un proxy inversé dans le cadre d’un accès à distance. Il n’est pas destiné aux utilisateurs qui font partie du réseau de l’entreprise. Les utilisateurs qui utilisent le proxy d’application pour accéder à l’intranet peuvent rencontrer des problèmes de performances.

Cet article fournit les ressources dont vous avez besoin pour planifier, utiliser et gérer le proxy d’application Azure AD.

## <a name="plan-your-implementation"></a>Planifier l’implémentation

La section suivante présente les éléments clés de la planification qui vous permettront d’effectuer un déploiement efficace.

### <a name="prerequisites"></a>Prérequis

Avant de commencer votre implémentation, vous devez respecter les prérequis suivants. Pour plus d’informations sur la configuration de votre environnement, y compris sur ces prérequis, consultez [ce tutoriel](application-proxy-add-on-premises-application.md).

* **Connecteurs** : les connecteurs sont des agents légers que vous pouvez déployer sur :
   * Du matériel physique local
   * Une machine virtuelle hébergée dans n’importe quelle solution d’hyperviseur
   * Une machine virtuelle hébergée dans Azure permettant une connexion sortante vers le service Proxy d’application

* Pour une présentation plus détaillée, consultez [Présentation des connecteurs du proxy d’application Azure AD](application-proxy-connectors.md).

     * Avant d’installer les connecteurs, vous devez [activer TLS 1.2](application-proxy-add-on-premises-application.md) sur les machines connecteurs.

     * Si possible, déployez les connecteurs sur le [même réseau](application-proxy-network-topology.md) et le même segment que les serveurs d’applications web back-end. Il est préférable de déployer les connecteurs après avoir lancé la découverte des applications.
     * Nous recommandons la présence d’au moins deux connecteurs par groupe de connecteurs à des fins de haute disponibilité et de mise à l’échelle. La présence de trois connecteurs est idéale si vous devez effectuer une maintenance sur l’une de vos machines. Pour savoir sur quel type de machine installer les connecteurs, consultez le [tableau des fonctionnalités de connecteurs](./application-proxy-connectors.md#capacity-planning). Plus la machine est volumineuse, plus le connecteur sera performant.

* **Paramètres d’accès réseau** : Les connecteurs du proxy d’application Azure AD [se connectent à Azure via HTTPS (port TCP 443) et HTTP (port TCP 80)](application-proxy-add-on-premises-application.md).

   * Il n’est pas possible d’arrêter le trafic TLS du connecteur, car cela empêche les connecteurs d’établir un canal sécurisé vers leurs points de terminaison de proxy d’application Azure respectifs.

   * Évitez toute forme d’inspection inline sur les communications TLS sortantes établies entre les connecteurs et Azure. Vous pouvez réaliser une inspection interne entre un connecteur et une application back-end. Toutefois, cette pratique n’est pas recommandée, car elle peut dégrader l’expérience utilisateur.

   * L’équilibrage de charge des connecteurs n’est pas pris en charge, ni même nécessaire.

### <a name="important-considerations-before-configuring-azure-ad-application-proxy"></a>Points importants à prendre en compte avant de configurer le proxy d’application Azure AD

Vous devez répondre aux exigences de base suivantes pour configurer et implémenter le proxy d’application Azure AD.

*  **Intégration Azure** : Avant le déploiement du proxy d’application, les identités utilisateur doivent être synchronisées à partir d’un annuaire local ou créées directement dans vos locataires Azure AD. La synchronisation des identités permet à Azure AD de pré-authentifier les utilisateurs avant de leur accorder l’accès aux applications publiées via le proxy d’application, mais aussi d’obtenir les identificateurs d’utilisateur nécessaires pour effectuer l’authentification unique (SSO).

* **Exigences relatives à l’accès conditionnel** : Nous vous déconseillons d’utiliser le proxy d’application pour accéder à l’intranet, car cela engendre une latence qui impacte les utilisateurs. Dans le cadre d’un accès à distance à partir d’Internet, nous vous recommandons d’utiliser le proxy d’application avec des stratégies de pré-authentification et d’accès conditionnel.  Pour permettre un accès conditionnel à l’intranet, vous pouvez moderniser les applications de sorte qu’elles puissent s’authentifier directement auprès d’AAD. Pour plus d’informations, consultez [Ressources pour la migration d’applications vers AAD](./migration-resources.md).

* **Limites du service** : pour éviter qu’un même locataire ne surconsomme les ressources, des limites sont définies pour chaque application et chaque locataire. Pour voir ces limites, consultez [Restrictions et limites du service Azure Active Directory](../enterprise-users/directory-service-limits-restrictions.md). Ces limites sont basées sur un point de référence bien supérieur à l’utilisation classique, et fournissent une mémoire tampon largement suffisante pour la plupart des déploiements.

* **Certificat public** : Si vous utilisez des noms de domaine personnalisés, vous devez fournir un certificat TLS/SSL. En fonction des exigences de votre organisation, l’obtention d’un certificat peut prendre un certain temps. Par conséquent, nous vous recommandons de démarrer le processus aussi tôt que possible. Le proxy d’application Azure prend en charge les certificats standard, [génériques](application-proxy-wildcard.md) et SAN. Pour plus de détails, consultez [Configurer des domaines personnalisés avec le Proxy d’application Azure AD](application-proxy-configure-custom-domain.md).

* **Exigences relatives aux domaines** : L’authentification unique à vos applications publiées à l’aide de la délégation Kerberos contrainte (KCD) nécessite que les serveurs exécutant le connecteur et l’application soient joints au domaine et qu’ils fassent partie du même domaine ou des domaines approuvés.
Pour des informations détaillées, consultez [KCD pour l’authentification unique auprès du proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md). Le service du connecteur s’exécute sur un système local et ne doit pas être configuré pour utiliser une identité personnalisée.

* **Enregistrements DNS pour les URL**

   * Avant d’utiliser des domaines personnalisés dans le proxy d’application, vous devez créer un enregistrement CNAME dans le DNS public. Ainsi, les clients pourront résoudre l’URL externe personnalisée vers l’adresse du proxy d’application prédéfinie. Si vous ne créez pas d’enregistrement CNAME pour une application qui utilise un domaine personnalisé, les utilisateurs distants ne pourront pas se connecter à l’application. Les étapes nécessaires à l’ajout d’enregistrements CNAME peuvent varier en fonction du fournisseur DNS. Vous devez donc savoir comment [gérer les enregistrements et jeux d’enregistrements DNS à l’aide du portail Azure](../../dns/dns-operations-recordsets-portal.md).

   * De même, les hôtes connecteurs doivent pouvoir résoudre l’URL interne des applications en cours de publication.

* **Rôles et droits d’administration**

   * L’**installation de connecteurs** nécessite des droits d’administrateur local pour le serveur Windows d’installation. En outre, elle nécessite au minimum un rôle d’*administrateur d’application* pour authentifier et inscrire l’instance de connecteur auprès de votre locataire Azure AD.

   * La **publication et l’administration d’applications** nécessitent le rôle d’*administrateur d’application*. Les administrateurs d’application peuvent gérer toutes les applications dans l’annuaire, y compris les inscriptions, les paramètres d’authentification unique, les licences et les affectations d’utilisateurs et de groupes, les paramètres du proxy d’application et le consentement. Il n’accorde pas la possibilité de gérer l’accès conditionnel. L’*administrateur d’application cloud* dispose des mêmes droits que l’administrateur d’application, sauf qu’il ne peut pas gérer les paramètres du proxy d’application.

* **Licences**. Le proxy d’application est disponible via l’abonnement Azure AD Premium. Pour obtenir la liste complète des options et des fonctionnalités de chaque licence, reportez-vous à la [page des tarifs Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="application-discovery"></a>Découverte des applications

Compilez un inventaire de toutes les applications de l’étendue qui sont publiées via le proxy d’application, en collectant les informations suivantes :

| Type d’informations| Informations à collecter |
|---|---|
| Type de service| Par exemple : SharePoint, SAP, CRM, application web personnalisée, API |
| Plateforme d’application | Par exemple : Windows IIS, Apache sur Linux, Tomcat, NGINX |
| Appartenance au domaine| Nom de domaine complet (FQDN) du serveur web |
| Emplacement de l’application | Emplacement du serveur web ou de la batterie de serveurs au sein de votre infrastructure |
| Accès interne | URL exacte utilisée lors de l’accès à l’application en interne. <br> S’il s’agit d’une batterie de serveurs, quel type d’équilibrage de charge est utilisé ? <br> Indique si l’application extrait du contenu à partir de sources autres qu’elle-même.<br> Déterminer si l’application fonctionne sur des WebSockets. |
| Accès externe | Solution du fournisseur à laquelle l’application est déjà exposée depuis l’extérieur. <br> URL que vous souhaitez utiliser pour l’accès externe. Pour SharePoint, vérifiez que les mappages des accès de substitution sont configurés selon [ces instructions](/SharePoint/administration/configure-alternate-access-mappings). Si ce n’est pas le cas, vous devrez définir des URL externes. |
| Certificat public | Si vous utilisez un domaine personnalisé, obtenez un certificat avec un nom d’objet correspondant. S’il existe un certificat, notez le numéro de série et l’emplacement à partir duquel il peut être obtenu. |
| Type d'authentification| Les types d’authentification pris en charge par l’application sont l’authentification de base, l’authentification Windows intégrée, l’authentification basée sur les formulaires, l’authentification basée sur les en-têtes, et les revendications. <br>Si l’application est configurée pour s’exécuter sous un compte de domaine spécifique, notez le nom de domaine complet (FQDN) du compte de service.<br> Pour une authentification basée sur SAML, ce sont l’URL de l’identificateur et l’URL de réponse. <br> Pour une authentification basée sur l’en-tête, il s’agit de la solution du fournisseur et d’une exigence spécifique concernant la gestion du type d’authentification. |
| Nom du groupe de connecteurs | Nom logique du groupe de connecteurs qui sera désigné pour fournir le conduit et l’authentification unique à l’application back-end. |
| Accès des utilisateurs et des groupes | Utilisateurs ou groupes d’utilisateurs qui recevront un accès externe à l’application. |
| Autres conditions requises | Notez toutes les exigences supplémentaires relatives à l’accès à distance et à la sécurité qui doivent être factorisées dans la publication de l’application. |

Pour inventorier vos applications, vous pouvez télécharger cette [feuille de calcul pour l’inventaire des applications](https://aka.ms/appdiscovery).

### <a name="define-organizational-requirements"></a>Définir les exigences de l’organisation

Les domaines ci-dessous nécessitent que vous définissiez des besoins métier pour votre organisation. Des exemples de besoins métier sont cités pour chacun de ces domaines.

 **y accéder**

* Les utilisateurs distants qui disposent d’appareils joints à un domaine ou joints à Azure AD peuvent accéder aux applications publiées de façon sécurisée, grâce à l’authentification unique (SSO).

* Les utilisateurs distants dotés d’appareils personnels approuvés peuvent accéder de façon sécurisée aux applications publiées, à condition qu’ils soient inscrits auprès de l’authentification multifacteur et qu’ils aient inscrit l’application Microsoft Authenticator sur leur téléphone mobile dans le cadre de l’authentification.

**Gouvernance**

* Les administrateurs peuvent définir et superviser le cycle de vie des affectations d’utilisateurs aux applications publiées via le Proxy d’Application.

**Sécurité**

* Seuls les utilisateurs associés aux applications (via leur appartenance à un groupe ou à titre individuel) peuvent accéder à ces applications.

**Performances**

* Dans ce cas, les performances des applications ne sont pas dégradées, comme c’est le cas lorsque vous accéder à une application à partir du réseau interne.

**Expérience utilisateur**

* Les utilisateurs savent comment accéder à leurs applications grâce aux URL de l’entreprise qui leur sont familières, et ce, sur n’importe quelle plateforme d’appareil.

**Audit**
* Les administrateurs sont capables d’auditer les accès utilisateur.


### <a name="best-practices-for-a-pilot"></a>Bonnes pratiques pour un pilote

Déterminez le temps et les efforts nécessaires afin de commissionner entièrement une application pour l’accès à distance à l’aide de l’authentification unique (SSO). Pour cela, exécutez un pilote qui effectue sa découverte initiale, sa publication et ses tests. L’utilisation d’une application web simple basée sur IIS et préconfigurée pour l’authentification Windows intégrée permet d’établir une base de référence, et de piloter l’accès à distance et l’authentification unique avec très peu d’efforts.

L’implémentation de votre pilote directement dans un locataire de production doit être facilitée par les éléments de conception suivants.

**Gestion des connecteurs** :

* les connecteurs jouent un rôle essentiel dans la fourniture du conduit local vers vos applications. Le groupe de connecteurs **par défaut** est adapté pour les premiers tests pilotes des applications publiées, avant le passage de celles-ci en production. Les applications ayant réussi les tests peuvent ensuite être déplacées vers les groupes de connecteurs de production.

**Gestion des applications** :

* vos employés sont davantage susceptibles de retenir une URL externe si elle leur est familière et si elle est explicite. Évitez de publier votre application à l’aide des suffixes prédéfinis msappproxy.net et onmicrosoft.com. Fournissez un domaine vérifié familier de niveau supérieur, préfixé par un nom d’hôte logique comme *intranet.<domaine_client>.com*.

* Autorisez uniquement un groupe pilote à voir l’icône de l’application et rendez-la invisible dans le portail Azure MyApps pour les autres utilisateurs. Lorsque vous êtes prêt pour la production, vous pouvez limiter la portée de l’application au public cible, soit dans le même locataire de préproduction, soit dans votre locataire de production.

**Paramètres d’authentification unique** : certains paramètres d’authentification unique ont des dépendances qui peuvent être longues à configurer. Par conséquent, pour éviter les retards liés au contrôle des modifications, il est conseillé de configurer les dépendances bien à l’avance. Cela inclut le fait de joindre les hôtes connecteurs au domaine afin d’effectuer l’authentification unique à l’aide de la délégation contrainte Kerberos (KCD), et de vous occuper des autres activités à exécution longue, comme la configuration d’une instance PINGAccess, par exemple, si vous avez besoin de l’authentification unique basée sur les en-têtes.

**Protocole TLS entre l’hôte connecteur et l’application cible** : la sécurité est primordiale. Par conséquent, vous devez systématiquement utiliser TLS entre les hôtes connecteurs et les applications cibles, en particulier si l’application web est configurée pour l’authentification basée sur les formulaires (FBA), car les informations d’identification utilisateur sont transmises en texte clair.

**Implémenter de manière incrémentielle et tester chaque étape** :
effectuez des tests fonctionnels de base après la publication d’une application afin de vérifier que toutes les exigences relatives aux utilisateurs et aux entreprises sont respectées. Pour cela, effectuez les étapes suivantes :

1. Testez et validez l’accès général à l’application web en désactivant la pré-authentification.
2. Si le test réussit, activez la pré-authentification, puis affectez des utilisateurs et des groupes. Testez puis validez l’accès.
3. Ensuite, ajoutez la méthode d’authentification unique pour votre application, puis réexécutez le test pour valider l’accès.
4. Si besoin, appliquez des stratégies d’accès conditionnel et d’authentification multifacteur. Testez puis validez l’accès.

**Outils de résolution des problèmes** : lors de la résolution de problèmes, commencez toujours par valider l’accès à l’application publiée à partir du navigateur sur l’hôte connecteur, puis vérifiez que l’application fonctionne comme prévu. Plus votre configuration est simple, plus il sera facile de déterminer la cause racine. Vous devez donc tenter de reproduire les problèmes avec une configuration minimale, par exemple, en n’utilisant qu’un seul connecteur et sans utiliser l’authentification unique. Dans certains cas, des outils de débogage web, tels que Fiddler de Telerik, peuvent s’avérer indispensables pour résoudre les problèmes liés à l’accès ou au contenu des applications qui sont accessibles via un proxy. Fiddler peut également servir de proxy pour tracer et déboguer le trafic sur les plateformes mobiles telles qu’iOS et Android, ainsi que tout ce qui peut être configuré pour être routé via un proxy. Pour plus d’informations, consultez le [guide de dépannage](application-proxy-troubleshoot.md).

## <a name="implement-your-solution"></a>Implémenter votre solution

### <a name="deploy-application-proxy"></a>Déployer le proxy d’application

Les étapes nécessaires au déploiement de votre proxy d’application sont fournies dans ce [tutoriel sur l’ajout d’une application locale pour l’accès à distance](application-proxy-add-on-premises-application.md). Si l’installation a échoué, sélectionnez **Dépanner le proxy d’application** dans le portail ou utilisez le guide de dépannage [Problèmes lors de l’installation du connecteur d’agent de proxy d’application](application-proxy-connector-installation-problem.md).

### <a name="publish-applications-via-application-proxy"></a>Publier des applications via le proxy d’application

La publication d’applications suppose que vous respectez tous les prérequis et que vous disposez de plusieurs connecteurs signalés comme étant inscrits et actifs dans la page Proxy d’application.

Vous pouvez également publier des applications à l’aide de [PowerShell](/powershell/module/azuread/?view=azureadps-2.0-preview).

Voici quelques bonnes pratiques à suivre lorsque vous publiez des applications :

* **Utilisez des groupes de connecteurs** : Affectez un groupe de connecteurs ayant été désigné pour publier chaque application. Nous recommandons la présence d’au moins deux connecteurs par groupe de connecteurs à des fins de haute disponibilité et de mise à l’échelle. La présence de trois connecteurs est idéale si vous devez effectuer une maintenance sur l’une de vos machines. Pour voir comment vous pouvez utiliser des groupes de connecteurs afin de segmenter vos connecteurs en fonction de leur réseau ou de leur emplacement, consultez [Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs](application-proxy-connector-groups.md).

* **Définissez un délai d’expiration pour l’application back-end** : Ce paramètre est utile dans les scénarios où l’application peut nécessiter plus de 75 secondes pour traiter une transaction client. C’est le cas, par exemple, lorsqu’un client envoie une requête à une application web qui sert de front-end à une base de données. Le front-end envoie cette requête à son serveur de base de données back-end et attend une réponse. Cependant, lorsqu’il reçoit la réponse, le côté client de la conversation a expiré. Lorsque vous configurez le délai sur la valeur Long, les transactions à exécution longue disposent de 180 secondes pour s’exécuter.

* **Utiliser des types de cookies adaptés**

   * **Cookie HTTPOnly** : fournit une sécurité supplémentaire en demandant au proxy d’application d’inclure l’indicateur HTTPOnly dans les en-têtes de réponse HTTP Set-Cookie. Ce paramètre permet d’atténuer les attaques de type script de site à site (XSS). Conservez la valeur Non pour les agents clients ou utilisateurs qui nécessitent un accès au cookie de session (par exemple, pour un client RDP/MTSC se connectant à la passerelle des services Bureau à distance publiée via le proxy d’application).

   * **Cookie sécurisé** : quand un cookie est défini avec l’attribut Secure, l’agent utilisateur (application côté client) inclut le cookie uniquement dans les requêtes HTTP si celles-ci sont transmises via un canal sécurisé TLS. Cela permet de réduire le risque qu’un cookie soit compromis sur des canaux en texte clair. Vous devez donc activer cette option.

   * **Cookie persistant** : permet au cookie de session du proxy d’application d’être conservé après la fermeture du navigateur, en restant valide jusqu’à son expiration ou sa suppression. Utilisé pour les scénarios où une application riche comme Office accède à un document dans une application web publiée, sans que l’utilisateur soit réinvité à s’authentifier. Toutefois, vous devez être prudent lorsque vous activez cette option, car les cookies persistants peuvent exposer votre service à des risques, (comme les accès non autorisés) si vous ne les utilisez pas conjointement à des systèmes permettant d’atténuer ces risques. Ce paramètre doit être utilisé uniquement pour les applications plus anciennes qui ne peuvent pas partager de cookies d’un processus à l’autre. Pour gérer le partage des cookies entre les processus, il est préférable de mettre votre application à jour plutôt que d’utiliser ce paramètre.

* **Traduire les URL dans les en-têtes** : activez cette option pour les scénarios où le DNS interne ne peut pas être configuré pour correspondre à l’espace de noms public de l’entreprise (c’est ce qu’on appelle un DNS divisé). Conservez la valeur Oui, sauf si votre application a besoin de l’en-tête d’hôte d’origine dans la requête cliente. Une autre solution consiste à configurer le connecteur pour qu’il utilise le nom de domaine complet dans l’URL interne dans le but de router le trafic, et qu’il utilise le nom de domaine complet dans l’URL externe comme en-tête de l’hôte. Dans la plupart des cas, cette alternative doit permettre à l’application de fonctionner normalement lors d’un accès à distance. Toutefois, vos utilisateurs perdent les avantages que présente le fait d’avoir une URL interne et une URL externe identiques.

* **Traduisez les URL dans le corps de l’application** : Activez la traduction de liens dans le corps de l’application lorsque vous souhaitez que les liens d’une application soient traduits en réponses au client. Lorsqu’elle est activée, cette fonctionnalité constitue la meilleure solution pour traduire tous les liens internes trouvés par le proxy d’application dans les réponses HTML et CSS qui sont retournées aux clients. Elle s’avère utile lors de la publication d’applications qui contiennent des liens absolus codés en dur ou des liens de nom court NetBIOS, ou d’applications dont le contenu est lié à d’autres applications locales.

Dans les scénarios où une application publiée est liée à d’autres applications publiées, activez la traduction de liens pour chaque application de manière à pouvoir contrôler l’expérience utilisateur de chacune d’elles séparément.

Par exemple, supposons que vous avez trois applications publiées via le proxy d’application et toutes liées entre elles : Bénéfices, Dépenses et Déplacements, plus une quatrième application, Commentaires, qui n’est pas publiée via le proxy d’application.

![Image 1](media/App-proxy-deployment-plan/link-translation.png)

Lorsque vous activez la traduction de liens pour l’application Bénéfices, les liens vers Dépenses et Déplacements sont redirigés vers les URL externes de ces applications. Ainsi, les utilisateurs peuvent accéder aux applications même s’ils ne font pas partie du réseau de l’entreprise. Les liens allant des applications Dépenses et Déplacements vers l’application Bénéfices ne fonctionnent pas, car la traduction de liens n’a pas été activée pour ces deux applications. Le lien vers l’application Commentaires n’est pas redirigé car il n’existe aucune URL externe. De fait, les utilisateurs de l’application Bénéfices ne pourront pas accéder à l’application Commentaires s’ils ne se trouvent pas dans le réseau de l’entreprise. Pour des informations détaillées, consultez [Traduction de liens et autres options de redirection](application-proxy-configure-hard-coded-link-translation.md).

### <a name="access-your-application"></a>Accéder à votre application

Il existe plusieurs options pour gérer l’accès aux ressources publiées du proxy d’application. Vous devez donc choisir celle qui est la mieux adaptée à votre scénario et à vos besoins de scalabilité. Les approches les plus courantes sont notamment : l’utilisation de groupes locaux qui sont synchronisés via Azure AD Connect, la création de groupes dynamiques dans Azure AD en fonction des attributs utilisateurs, l’utilisation de groupes en libre-service gérés par un propriétaire de ressources, ou une combinaison de toutes ces approches. Pour connaître les avantages de chacune de ces approches, reportez-vous aux ressources liées.

Le moyen le plus simple d’accorder à des utilisateurs l’accès à une application est d’accéder aux options de la section **Utilisateurs et groupes** située dans le volet gauche de votre application publiée, et d’accorder directement l’accès aux groupes ou aux utilisateurs individuels.

![Image 24](media/App-proxy-deployment-plan/add-user.png)

Vous pouvez également accorder aux utilisateurs un accès en libre-service à votre application, en affectant un groupe dont ils ne sont pas membres et en configurant les options d’accès en libre-service.

![Image 25](media/App-proxy-deployment-plan/allow-access.png)

Si cette option est activée, les utilisateurs pourront se connecter au portail MyApps et demander l’accès. Ils seront alors approuvés automatiquement puis ajoutés au groupe d’accès en libre-service déjà autorisé, ou ils devront attendre que l’approbateur désigné les approuve.

Les utilisateurs invités peuvent également être [invités à accéder aux applications internes publiées via le proxy d’application via Azure AD B2B](../external-identities/add-users-information-worker.md).

Pour les applications locales qui sont normalement accessibles de manière anonyme et qui ne nécessitent pas d’authentification, il peut être préférable de désactiver l’option située dans les **propriétés** de l’application.

![Image 26](media/App-proxy-deployment-plan/assignment-required.png)


Si vous conservez la valeur Non pour cette option, les utilisateurs pourront accéder à l’application locale via le proxy d’application Azure AD sans autorisations. Vous devez donc utiliser cette option avec précaution.

Une fois que l’application est publiée, vous devez pouvoir y accéder en tapant son URL dans un navigateur externe ou en cliquant sur son icône à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com/).

### <a name="enable-pre-authentication"></a>Activer la pré-authentification

Vérifiez que votre application est accessible via le proxy d’application en y accédant via l’URL externe.

1. Accédez à **Azure Active Directory** > **Applications d’entreprise** > **Toutes les applications** et sélectionnez l’application que vous souhaitez gérer.

2. Sélectionnez **Proxy d’application**.

3. Dans le champ **Pré-authentification**, cliquez sur la liste déroulante pour sélectionner **Azure Active Directory**, puis sélectionnez **Enregistrer**.

Lorsque la pré-authentification est activée, Azure AD demande d’abord aux utilisateurs de s’authentifier. Si l’authentification unique est configurée, l’application principale vérifie également l’utilisateur avant de lui accorder l’accès à l’application. Le fait de passer du mode de pré-authentification Relais au mode Azure AD configure également l’URL externe avec le protocole HTTPS. Ainsi, toutes les applications initialement configurées pour le protocole HTTP sont maintenant sécurisées avec le protocole HTTPS.

### <a name="enable-single-sign-on"></a>activation de l'authentification unique

L’authentification unique fournit la meilleure expérience possible et le meilleur niveau de sécurité, puisque les utilisateurs ne doivent se connecter qu’une seule fois lors de l’accès à Azure AD. Une fois qu’un utilisateur a été pré-authentifié, l’authentification unique est réalisée par le connecteur du proxy d’application qui s’authentifie auprès de l’application locale, pour le compte de l’utilisateur. L’application back-end traite la connexion comme s’il s’agissait de l’utilisateur.

En choisissant l’option **Relais**, vous permettez aux utilisateurs d’accéder à l’application publiée sans jamais avoir à s’authentifier auprès d’Azure AD.

L’authentification unique n’est possible que si Azure AD peut identifier l’utilisateur qui demande l’accès à une ressource. Pour que l’authentification unique fonctionne, votre application doit donc être configurée pour la pré-authentification des utilisateurs auprès d’Azure AD lors de l’accès. Sinon, les options de l’authentification unique seront désactivées.

Pour savoir quelle méthode d’authentification unique est la mieux adaptée lorsque vous configurez vos applications, lisez [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](what-is-single-sign-on.md).

###  <a name="working-with-other-types-of-applications"></a>Utilisation des autres types d’applications

Le proxy d’application Azure AD peut également prendre en charge les applications qui ont été développées pour utiliser la bibliothèque d’authentification Microsoft ([MSAL](../develop/v2-overview.md)). Pour prendre en charge les applications clientes natives, il utilise les jetons émis par Azure AD qui sont compris dans les informations d’en-tête de la requête cliente. Il effectue ainsi la pré-authentification à la place des utilisateurs.

Pour plus d’informations sur les configurations possibles du proxy d’application, lisez [Publication d’applications clientes natives et mobiles](./application-proxy-configure-native-client-application.md) et [Applications basées sur les revendications](./application-proxy-configure-for-claims-aware-applications.md).

### <a name="use-conditional-access-to-strengthen-security"></a>Utiliser l’accès conditionnel pour renforcer la sécurité

La sécurité des applications nécessite un ensemble de fonctionnalités de sécurité avancées capables de protéger les applications locales et les applications cloud contre les menaces complexes. Les attaquants parviennent souvent à accéder au réseau de l’entreprise par le biais d’informations d’identification volées, peu sécurisées ou non personnalisées.  La sécurité basée sur les identités Microsoft réduit le risque de vol des informations d’identification, car elle gère et protège aussi bien les identités privilégiées que les identités non privilégiées.

Les fonctionnalités suivantes peuvent être utilisées pour prendre en charge le proxy d’application Azure AD :

* Accès conditionnel basé sur les utilisateurs et la localisation : Assurez la protection de vos données sensibles en limitant l’accès utilisateur en fonction de leur géolocalisation ou de leur adresse IP, à l’aide de [stratégies d’accès conditionnel basées sur la localisation](../conditional-access/location-condition.md).

* Accès conditionnel basé sur les appareils : faites en sorte que seuls les appareils inscrits, approuvés et conformes puissent accéder aux données d’entreprise grâce à l’[accès conditionnel basé sur les appareils](../conditional-access/require-managed-devices.md).

* Accès conditionnel basé sur les applications : l’utilisateur peut continuer de travailler, même après avoir quitté le réseau de l’entreprise. [Sécurisez l’accès aux applications d’entreprise cloud et locales](../conditional-access/app-based-conditional-access.md) et gardez le contrôle grâce à l’accès conditionnel.

* Accès conditionnel basé sur les risques : Protégez vos données contre les pirates grâce à une [stratégie d’accès conditionnel basée sur les risques](https://www.microsoft.com/cloud-platform/conditional-access), qui peut être appliquée à l’ensemble de vos applications et utilisateurs, aussi bien localement que dans le cloud.

* Mes applications Azure AD : une fois votre service Proxy d’application déployé et vos applications publiées de façon sécurisée, proposez à vos utilisateurs un hub simple à partir duquel ils pourront découvrir et accéder à toutes leurs applications. Améliorez votre productivité grâce à des fonctionnalités en libre-service, telles que la possibilité de demander l’accès à de nouveaux groupes et applications, ou de gérer l’accès à ces ressources pour le compte d’autres utilisateurs via le volet [Mes applications](https://aka.ms/AccessPanelDPDownload).

## <a name="manage-your-implementation"></a>Gérer l’implémentation

### <a name="required-roles"></a>Rôles nécessaires

Microsoft applique le principe des privilèges minimum pour l’exécution de tâches à l’aide d’Azure AD. [Passez en revue les différents rôles Azure](../roles/permissions-reference.md) et choisissez celui qui correspond le mieux aux besoins de chaque personne. Certains rôles devront peut-être être appliqués temporairement, puis supprimés une fois le déploiement terminé.

| Rôle| Tâches| Rôles Azure AD |
|---|---|---|
| Administrateur du support technique | Généralement, son rôle se limite à classer les problèmes signalés par l’utilisateur final et à effectuer des tâches limitées telles que la modification des mots de passe utilisateurs, l’invalidation des jetons d’actualisation et la supervision de l’intégrité du service. | Administrateur du support technique |
| Administrateur d’identité| Pour déboguer les problèmes liés au proxy d’application, lisez les rapports de connexion et les journaux d’audit Azure AD.| Lecteur de sécurité |
| Propriétaire de l’application| Pour créer et gérer tous les aspects des applications d’entreprise, des inscriptions d’applications et des paramètres du proxy d’application.| Administrateur d’application |
| Administrateur d’infrastructure | Propriétaire de la substitution de certificat | Administrateur d’application |

Le fait de limiter le nombre de personnes qui ont accès aux informations et aux ressources sensibles aide à réduire le risque qu’un acteur malveillant obtienne un accès non autorisé, ou qu’une ressource sensible soit accidentellement impactée par un utilisateur autorisé.

Toutefois, les utilisateurs doivent toujours effectuer des opérations nécessitant des privilèges au quotidien. Par conséquent, nous vous recommandons d’appliquer des stratégies [Privileged Identity Management](../privileged-identity-management/pim-configure.md) juste-à-temps (JIT) afin de fournir un accès privilégié à la demande aux ressources Azure et Azure AD, dans le but de gérer et d’auditer efficacement les accès administratifs.

### <a name="reporting-and-monitoring"></a>Création de rapports et surveillance

Azure AD fournit des insights supplémentaires sur l’utilisation des applications et l’intégrité opérationnelle de votre organisation par le biais de [journaux d’audit et de rapports](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). Le proxy d’application facilite également la supervision des connecteurs à partir du portail Azure AD et des journaux des événements Windows.

#### <a name="application-audit-logs"></a>Journaux d’audit des applications

Ces journaux fournissent des informations détaillées sur les connexions aux applications configurées avec le proxy d’application, ainsi que sur l’appareil et l’utilisateur qui accèdent à l’application. Les [journaux d’audit](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) sont accessibles via le portail Azure et l’[API Audit](/graph/api/resources/directoryaudit?view=graph-rest-beta) desquels ils peuvent être exportés. Des [rapports d’utilisation et d’insights](../reports-monitoring/concept-usage-insights-report.md?context=azure/active-directory/manage-apps/context/manage-apps-context) sont également disponibles pour votre application.

#### <a name="application-proxy-connector-monitoring"></a>Supervision du connecteur Proxy d’application

Les connecteurs et le service se chargent de toutes les tâches de haut niveau de disponibilité. Vous pouvez superviser l’état de vos connecteurs à partir de la page Proxy d’application dans le portail Azure AD. Pour plus d’informations sur la maintenance des connecteurs, consultez [Présentation des connecteurs de proxy d’application Azure AD](./application-proxy-connectors.md#maintenance).

![Exemple : Connecteurs de proxy d’application Azure AD](./media/application-proxy-connectors/app-proxy-connectors.png)

#### <a name="windows-event-logs-and-performance-counters"></a>Journaux des événements Windows et compteurs de performances

Les connecteurs ont des journaux d’activité de session et d’administration. Les journaux d’activité admin incluent les événements principaux et leurs erreurs. Les journaux d’activité de session incluent toutes les transactions et les détails de traitement. Les journaux et les compteurs se trouvent dans les journaux des événements Windows. Pour plus d’informations, consultez [Présentation des connecteurs de proxy d’application Azure AD](./application-proxy-connectors.md#under-the-hood). Suivez ce [tutoriel pour configurer les sources de données de journaux des événements dans Azure Monitor](../../azure-monitor/platform/data-sources-windows-events.md).

### <a name="troubleshooting-guide-and-steps"></a>Guide de résolution des problèmes

Pour plus d’informations sur les problèmes courants et sur leur résolution, consultez notre guide pour [résoudre les messages d’erreur](application-proxy-troubleshoot.md).

Les articles suivants abordent des scénarios courants qui peuvent également être utilisés dans le but de créer des guides de dépannage pour votre organisation.

* [Problème lors de l’affichage de la page de l’application](application-proxy-page-appearance-broken-problem.md)
* [Le chargement de l’application est trop long](application-proxy-page-load-speed-problem.md)
* [Les liens sur la page de l’application ne fonctionnent pas](application-proxy-page-links-broken-problem.md)
* [Quels ports dois-je ouvrir pour mon application ?](application-proxy-add-on-premises-application.md)
* [Aucun connecteur ne fonctionne dans un groupe de connecteurs pour mon application](application-proxy-connectivity-no-working-connector.md)
* [Configurer dans le portail d’administration](application-proxy-config-how-to.md)
* [Configurer l’authentification unique vers mon application](application-proxy-config-sso-how-to.md)
* [Problème de création d’une application dans le portail d’administration](application-proxy-config-problem.md)
* [Configurer a délégation Kerberos contrainte](application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
* [Configurer avec PingAccess](./application-proxy-ping-access-publishing-guide.md)
* [Erreur Impossible d’accéder à cette application d’entreprise](application-proxy-sign-in-bad-gateway-timeout-error.md)
* [Problèmes lors de l’installation du connecteur d’agent de proxy d’application](application-proxy-connector-installation-problem.md)
* [Problème de connexion](application-sign-in-problem-on-premises-application-proxy.md)
