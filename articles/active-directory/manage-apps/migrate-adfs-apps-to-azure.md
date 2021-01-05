---
title: Déplacement de l’authentification d’application d’AD FS vers Azure Active Directory
description: Cet article a pour but d’aider les organisations à comprendre comment déplacer des applications vers Azure AD, et se concentre sur les applications SaaS fédérées.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 04/01/2020
ms.author: kenwith
ms.reviewer: baselden
ms.collection: M365-identity-device-management
ms.openlocfilehash: e15009dd935d91971ce4212ff44b67a1ca6fa363
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745375"
---
# <a name="moving-application-authentication-from-active-directory-federation-services-to-azure-active-directory"></a>Déplacement de l’authentification d’application des services de fédération Active Directory (AD FS) vers Azure Active Directory

[Azure Active Directory (Azure AD)](../fundamentals/active-directory-whatis.md) propose une plateforme d’identité universelle qui offre à vos employés, partenaires et clients une identité unique pour accéder aux applications et collaborer à partir de n’importe quelle plateforme et de n’importe quel appareil. Azure AD comprend une [suite complète de fonctionnalités de gestion des identités](../fundamentals/active-directory-whatis.md). La standardisation de l’authentification et de l’autorisation de votre application avec Azure AD permet d’offrir les avantages de ces fonctionnalités.

> [!TIP]
> Cet article est destiné aux développeurs. Les chefs de projet et les administrateurs qui planifient le passage d’une application à Azure AD doivent envisager de lire notre livre blanc [Déplacement de l’authentification d’application vers Azure AD](https://aka.ms/migrateapps/whitepaper) (PDF).

## <a name="introduction"></a>Introduction

Si vous disposez d’un annuaire local qui contient des comptes d’utilisateur, vous avez probablement de nombreuses applications auxquelles les utilisateurs s’authentifient. Chacune de ces applications est configurée pour que les utilisateurs puissent y accéder à l’aide de leurs identités.


Les utilisateurs peuvent également s’authentifier directement auprès de votre instance Active Directory locale. AD FS est un service d’identité local basé sur des normes. Il étend la capacité à utiliser la fonctionnalité d’authentification unique (SSO) entre partenaires commerciaux approuvés sans obliger les utilisateurs à se connecter séparément à chaque application. C’est ce que l’on appelle la fédération.

De nombreuses organisations disposent d’applications SaaS (Software as a Service) ou métiers personnalisées (LoB), fédérées directement dans AD FS, et des applications Microsoft 365 et Azure AD.

![Applications directement connectées localement](media/migrate-adfs-apps-to-azure/app-integration-before-migration1.png)

**Pour améliorer la sécurité de l’application, votre objectif est de disposer d’un seul ensemble de contrôles d’accès et de stratégies dans vos environnements locaux et cloud**.

![Applications connectées via Azure AD](media/migrate-adfs-apps-to-azure/app-integration-after-migration1.png)



## <a name="types-of-apps-to-migrate"></a>Types d’applications à migrer

La migration de l’ensemble des authentifications de vos applications vers Azure AD est optimale, car elle vous donne un plan de contrôle unique pour la gestion des identités et des accès.

Vos applications peuvent utiliser des protocoles modernes ou hérités pour l’authentification. Envisagez d’abord de migrer les applications qui utilisent des protocoles d’authentification modernes (tels que SAML et Open ID Connect). Ces applications peuvent être reconfigurées pour s’authentifier auprès d’Azure AD via un connecteur intégré dans notre galerie d’applications, ou en inscrivant l’application dans Azure AD. Les applications utilisant des protocoles plus anciens peuvent être intégrées à l’aide du [proxy d’application](./what-is-application-proxy.md).

Pour plus d’informations, consultez [Quels types d’applications puis-je intégrer à Azure AD](./what-is-application-management.md) ?

Vous pouvez [Utiliser le rapport d’activité des applications AD FS pour migrer des applications vers Azure AD](./migrate-adfs-application-activity.md) si vous avez [activé Azure AD Connect Health](../hybrid/how-to-connect-health-adfs.md).

### <a name="the-migration-process"></a>Le processus de migration

Pendant le processus de migration de l’authentification de vos applications vers Azure AD, testez correctement vos applications et votre configuration. Nous vous recommandons de continuer à utiliser les environnements de test existants pour les tests de migration lors du passage à l’environnement de production. Si aucun environnement de test n’est actuellement disponible, vous pouvez en configurer un à l’aide [d’Azure App Service](https://azure.microsoft.com/services/app-service/) ou de [machines virtuelles Azure](https://azure.microsoft.com/free/virtual-machines/search/?OCID=AID2000128_SEM_lHAVAxZC&MarinID=lHAVAxZC_79233574796345_azure%20virtual%20machines_be_c__1267736956991399_kwd-79233582895903%3Aloc-190&lnkd=Bing_Azure_Brand&msclkid=df6ac75ba7b612854c4299397f6ab5b0&ef_id=XmAptQAAAJXRb3S4%3A20200306231230%3As&dclid=CjkKEQiAhojzBRDg5ZfomsvdiaABEiQABCU7XjfdCUtsl-Abe1RAtAT35kOyI5YKzpxRD6eJS2NM97zw_wcB), en fonction de l’architecture de l’application.

Vous pouvez choisir de configurer un client de test Azure AD distinct à utiliser lorsque vous développez vos configurations d’application.

Votre processus de migration peut se présenter comme suit :

**Étape 1 : état actuel : Application de production s’authentifiant avec AD FS**

![Étape de migration 1 ](media/migrate-adfs-apps-to-azure/stage1.jpg)


**Étape 2 : FACULTATIF : Instance de test de l’application pointant vers un locataire de test Azure**

Mettez à jour la configuration de façon à faire pointer votre instance de test de l’application vers un locataire de test Azure AD et apportez les modifications nécessaires. L’application peut être testée avec les utilisateurs dans le locataire de test Azure AD. Pendant le processus de développement, vous pouvez utiliser des outils tels que [Fiddler](https://www.telerik.com/fiddler) pour comparer et vérifier les requêtes et les réponses.

Si la configuration d’un locataire de test distinct n’est pas possible, ignorez cette étape et mettez en place une instance de test d’application et pointez-la vers votre locataire Azure AD de production comme décrit à l’étape 3 ci-dessous.

![Étape de migration 2 ](media/migrate-adfs-apps-to-azure/stage2.jpg)

**Étape 3 : application de test pointant vers le locataire Azure de production**

Mettez à jour la configuration de façon à faire pointer votre instance de test d’application vers votre instance Azure de production. Vous pouvez maintenant la tester avec les utilisateurs de votre instance de production. Si nécessaire, consultez la section de cet article sur la transition des utilisateurs.

![Étape de migration 3 ](media/migrate-adfs-apps-to-azure/stage3.jpg)

**Étape 4 : application de production pointant vers le locataire AD de production**

Mettez à jour la configuration de votre application de production pour qu’elle pointe vers votre locataire Azure de production.

![Étape de migration 4 ](media/migrate-adfs-apps-to-azure/stage4.jpg)

 Les applications qui s’authentifient avec AD FS peuvent utiliser des groupes Active Directory pour les autorisations. Utilisez la [synchronisation Azure AD Connect](../hybrid/how-to-connect-sync-whatis.md) pour synchroniser les données d’identité entre votre environnement local et Azure AD avant de commencer la migration. Vérifiez ces groupes et les appartenances avant la migration afin de pouvoir accorder l’accès aux mêmes utilisateurs lors de la migration de l’application.

### <a name="line-of-business-lob-apps"></a>Applications métier

Les applications métier sont développées en interne par votre organisation ou disponibles en tant que produit groupé standard installé dans votre centre de données. Les exemples incluent des applications basées sur Windows Identity Foundation et des applications SharePoint (et non SharePoint Online).

Les applications métier qui utilisent OAuth 2.0, OpenID Connect ou WS-Federation peuvent être intégrées à Azure AD en tant [qu’inscriptions d’application](../develop/quickstart-register-app.md). Intégrez des applications personnalisées qui utilisent SAML 2.0 ou WS-Federation, comme les [applications hors galerie](./add-application-portal.md) sur la page Applications d’entreprise dans le [portail Azure](https://portal.azure.com/).

## <a name="saml-based-single-sign-on"></a>Authentification unique SAML

Les applications qui utilisent SAML 2.0 pour l’authentification peuvent être configurées pour [l’authentification unique basée sur SAML](./what-is-single-sign-on.md) (SSO basée sur SAML). Avec [l’authentification unique SAML](./what-is-single-sign-on.md), vous pouvez mapper les utilisateurs à des rôles d’application spécifiques en fonction de règles que vous définissez dans vos revendications SAML.

Pour configurer une application SaaS pour l’authentification unique via SAML, consultez [Configurer l’authentification unique via SAML](./view-applications-portal.md).

![Captures d’écran d’utilisateur de l’authentification unique SAML ](media/migrate-adfs-apps-to-azure/sso-saml-user-attributes-claims.png)


De nombreuses applications SaaS offrent un [tutoriel spécifique à l’application](../saas-apps/tutorial-list.md) qui vous guide dans la configuration de l’authentification unique basée sur SAML.

![Tutoriel basé sur une application](media/migrate-adfs-apps-to-azure/app-tutorial.png)

Certaines applications peuvent facilement être migrées. D’autres plus complexes telles que les revendications personnalisées nécessitent une configuration supplémentaire dans Azure AD et/ou Azure AD Connect. Pour plus d’informations sur les mappages de revendications pris en charge, consultez [Mappage des revendications dans Azure Active Directory](../develop/active-directory-claims-mapping.md).

Gardez à l’esprit les limitations suivantes lors du mappage des attributs :

* Tous les attributs qui ne peuvent pas être émis dans AD FS apparaissent dans Azure AD en tant qu’attributs à émettre aux jetons SAML, même si ces attributs sont synchronisés. Lorsque vous modifiez l’attribut, la liste déroulante Valeur affiche les différents attributs disponibles dans Azure AD. Vérifiez la configuration de synchronisation d’Azure AD Connect afin de garantir qu’un attribut requis (par exemple [samAccountName](../hybrid/how-to-connect-sync-whatis.md)) est bien synchronisé avec Azure AD. Vous pouvez utiliser les attributs d’extension pour émettre toute revendication qui ne fait pas partie du schéma utilisateur standard dans Azure AD.

* Dans les scénarios les plus courants, seules la revendication NameID et d’autres revendications d’identificateur utilisateur courantes sont requises pour une application. Pour déterminer si une revendication supplémentaire est nécessaire, examinez les revendications émises à partir d’AD FS.

* Toutes les revendications ne peuvent pas être émises, car certaines sont protégées dans Azure AD.

* La possibilité d’utiliser des jetons SAML chiffrés est actuellement en préversion. Voir [Procédure : personnaliser des revendications émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-saml-claims-customization.md).



### <a name="software-as-a-service-saas-apps"></a>Applications SaaS (Software as a Service)

Si vos utilisateurs se connectent à des applications SaaS telles que Salesforce, ServiceNow ou Workday, et qu’elles sont intégrés à AD FS, vous utilisez l’authentification fédérée pour les applications SaaS.

La plupart des applications SaaS peuvent déjà être configurées dans Azure AD. Microsoft propose de nombreuses connexions préconfigurées pour les applications SaaS dans la [galerie d’applications Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps), ce qui facilitera la transition. Les applications SAML 2.0 peuvent être intégrées à Azure AD soit via la galerie d’applications Azure AD, soit comme des [applications ne figurant pas dans la galerie](./add-application-portal.md).

Les applications qui utilisent OAuth 2.0 ou OpenID Connect peuvent être intégrées à Azure AD de façon similaire en tant qu’[inscriptions d’application](../develop/quickstart-register-app.md). Les applications qui utilisent des protocoles hérités peuvent utiliser le [proxy d'application AD](./application-proxy.md) pour s’authentifier auprès d’Azure AD.

Pour les problèmes liés à l’intégration de vos applications SaaS, vous pouvez contacter [l’alias du support de l’intégration d’applications SaaS](mailto:SaaSApplicationIntegrations@service.microsoft.com).

**Certificats de signature SAML pour l’authentification unique** : Les certificats de signature sont une partie importante de tout déploiement d’authentification unique. Azure AD crée les certificats de signature pour établir une authentification unique fédérée basée sur SAML pour vos applications SaaS. Une fois que vous avez ajouté les applications de la galerie ou hors galerie, vous configurez l’application ajoutée à l’aide de l’option d’authentification unique fédérée. Voir [Gérer des certificats pour l’authentification unique fédérée sur Azure Active Directory](./manage-certificates-for-federated-single-sign-on.md).

### <a name="apps-and-configurations-that-can-be-moved-today"></a>Applications et configurations qui peuvent être déplacées aujourd’hui

Actuellement, les applications que vous pouvez déplacer facilement sont les applications SAML 2.0 qui utilisent l’ensemble standard d’éléments de configuration et de revendications :

* Nom d’utilisateur principal

* Adresse de messagerie

* Prénom

* Surname

* D’un autre attribut comme **NameID** de SAML, y compris l’attribut de messagerie Azure AD, le préfixe de messagerie, l’attribut employeeid, les attributs d’extension 1-15, ou l’attribut local **SamAccountName**. Pour plus d’informations, consultez la section [Modification de la revendication NameIdentifier](../develop/active-directory-saml-claims-customization.md).

* Revendications personnalisées.

Les éléments suivants requièrent des étapes de configuration supplémentaires pour migrer vers Azure AD :

* Règles d’autorisation personnalisées ou règles d’authentification multifacteur (MFA) dans AD FS. Vous pouvez les configurer à l’aide de la fonction [Accès conditionnel Azure AD](../conditional-access/overview.md).

* Applications avec plusieurs points de terminaison d’URL de réponse. Vous les configurez dans Azure AD à l’aide de PowerShell ou de l’interface du portail Azure.

* Applications WS-Federation telles que les applications SharePoint qui nécessitent des jetons SAML version 1.1. Vous pouvez les configurer manuellement à l’aide de PowerShell. Vous pouvez également ajouter un modèle générique préintégré pour les applications SharePoint et SAML 1.1 à partir de la Galerie. Nous prenons en charge le protocole SAML 2.0.

* Règles de transformation des émissions de revendications complexes. Pour plus d’informations sur les mappages de revendications pris en charge, consultez :
   *  [Mappage des revendications dans Azure Active Directory](../develop/active-directory-claims-mapping.md)
   * [Personnalisation des revendications émises dans le jeton SAML pour les applications d’entreprise dans Azure Active Directory](../develop/active-directory-saml-claims-customization.md)



### <a name="apps-and-configurations-not-supported-in-azure-ad-today"></a>Applications et configurations non prises en charge dans Azure AD actuellement

Les applications qui nécessitent les fonctionnalités suivantes ne peuvent actuellement pas être migrées.

**Fonctionnalités de protocole**

* Prise en charge du modèle WS-Trust ActAs

* Résolution d’artefacts SAML

* Vérification de signature des requêtes SAML signées (remarque : les requêtes signées sont acceptées mais la signature n’est pas vérifiée).
Étant donné qu’Azure AD renverra uniquement le jeton aux points de terminaison préconfigurés dans l’application, la vérification de la signature n’est probablement pas nécessaire dans la plupart des cas.

**Revendications dans les fonctionnalités des jetons**

* Les revendications des magasins d’attributs autres que l’annuaire Azure AD, sauf si ces données sont synchronisées avec Azure AD. Pour plus d’informations, consultez la page [Vue d’ensemble de l’API de synchronisation Azure AD](/graph/api/resources/synchronization-overview?view=graph-rest-beta).

* Émission d’attributs à plusieurs valeurs d’annuaire. Par exemple, nous ne pouvons pas émettre une revendication à plusieurs valeurs pour les adresses proxy pour l’instant.

## <a name="map-app-settings-from-ad-fs-to-azure-ad"></a>Mapper les paramètres de l’application d’AD FS à Azure AD

La migration commence par l’évaluation de la configuration locale de l’application, et par le mappage de cette configuration dans Azure AD. AD FS et Azure AD fonctionnent de la même manière. La configuration de la confiance, des URL et des identificateurs d’authentification et de déconnexion s’appliquent pour les deux. Documentez les paramètres de configuration d’AD FS de vos applications afin de pouvoir les configurer facilement dans Azure AD.

### <a name="map-app-configuration-settings"></a>Paramètres de configuration d'application de mappage

Le tableau suivant décrit certains des mappages les plus courants de paramètres entre une partie de confiance AD FS et une application d'entreprise Azure AD :

* AD FS : recherchez le paramètre dans la partie de confiance AD FS pour l’application. Cliquez avec le bouton droit sur la partie de confiance et sélectionnez Propriétés.

* Azure AD : le paramètre est configuré dans le [portail Azure](https://portal.azure.com/) dans les propriétés d’authentification unique de chaque application.

| Paramètre de configuration| AD FS| Configuration dans Azure AD| Jeton SAML |
| - | - | - | - |
| **URL d’authentification à une application** <p>URL permettant à l’utilisateur de se connecter à l’application dans un flux SAML initié par le fournisseur de services.| N/A| Ouvrir la configuration SAML de base à partir de l’authentification basée sur SAML| N/A |
| **URL de réponse de l’application** <p>L’URL de l’application du point de vue du fournisseur d’identité. L’IdP envoie l’utilisateur et le jeton ici une fois que l’utilisateur s’est connecté au fournisseur d’identité (IdP).  Cela est également appelé **point de terminaison consommateur d’assertion SAML**.| Sélectionner l’onglet **Points de terminaison**| Ouvrir la configuration SAML de base à partir de l’authentification basée sur SAML| Élément Destination dans le jeton SAML. Exemple de valeur : `https://contoso.my.salesforce.com` |
| **URL de déconnexion de l’application** <p>Il s’agit de l’URL vers laquelle les requêtes de « nettoyage de déconnexion » sont envoyées lorsqu’un utilisateur se déconnecte d’une application. Le fournisseur d’identité (IdP) envoie la requête pour déconnecter l’utilisateur de toutes les autres applications.| Sélectionner l’onglet **Points de terminaison**| Ouvrir la configuration SAML de base à partir de l’authentification basée sur SAML| N/A |
| **Identificateur de l’application** <p>Il s’agit de l’identificateur d’application du point de vue de l’IdP. La valeur de l’URL d’authentification est souvent utilisée pour l’identificateur (mais pas toujours).  L’application l’appelle parfois l’« ID d’entité ».| Sélectionner l'onglet **Identificateurs**|Ouvrir la configuration SAML de base à partir de l’authentification basée sur SAML| Correspond à l’élément **Audience** du jeton SAML. |
| **Métadonnées de fédération de l’application** <p>C’est l’emplacement des métadonnées de fédération de l’application. Les fournisseurs d’identité l’utilisent pour mettre à jour automatiquement des paramètres de configuration spécifiques tels que les points de terminaison ou les certificats de chiffrement.| Sélectionner l'onglet **Surveillance**| N/A. Azure AD ne prend pas en charge la consommation directe de métadonnées de fédération de l’application. Vous pouvez importer manuellement les métadonnées de fédération.| N/A |
| **Identificateur utilisateur/ID nom** <p>Attribut utilisé pour indiquer uniquement l’utilisateur d’Azure AD ou AD FS vers votre application.  Il s’agit en général de l’UPN ou de l’adresse de messagerie de l’utilisateur.| Règles de revendication. Dans la plupart des cas, la règle de revendication génère une revendication dont le type se termine par NameIdentifier.| Vous pouvez trouver l’identificateur sous l’en-tête **Attributs utilisateur et revendications**. Par défaut, c’est le nom d’utilisateur principal qui est utilisé| Correspond à l’élément **NameID** dans le jeton SAML. |
| **Autres revendications** <p>Voici des exemples d’autres informations de revendication généralement envoyées de l’IdP à l’application, notamment le prénom, le nom, l’adresse de messagerie et l’appartenance au groupe.| Dans AD FS, elles se trouvent sur la partie de confiance en tant qu’autres règles de revendication.| Vous pouvez trouver l’identificateur sous l’en-tête **Attributs utilisateur et revendications**. Sélectionnez **Afficher** et modifier tous les autres attributs utilisateur.| N/A |


### <a name="map-identity-provider-idp-settings"></a>Paramètres de l’IdP (fournisseur d'identité) de mappage

Configurez vos applications pour qu’elles pointent vers Azure AD plutôt que AD FS pour l’authentification unique. Ici, nous nous concentrons sur les applications SaaS qui utilisent le protocole SAML. Toutefois, ce concept s’étend également aux applications métier personnalisées.

> [!NOTE]
> Les valeurs de configuration pour Azure AD suivent le modèle où votre ID de locataire Azure remplace {tenant-id} et l’ID d’application remplace {application-id}. Vous trouverez ces informations dans le [portail Azure](https://portal.azure.com/), sous la page Azure Active Directory > Propriétés :

* Sélectionnez l’ID d’annuaire pour afficher l’ID de votre locataire.

* Sélectionnez l’ID d’application pour afficher l’ID de votre application.

 À un niveau élevé, mappez les éléments clés de configuration d’application SaaS suivants à Azure AD.



| Élément| Valeur de configuration |
| - | - |
| Type d’émetteur d’identité| https:\//sts.windows.net/{tenant-id}/ |
| URL de connexion au fournisseur d'identité| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| URL de déconnexion de fournisseur d'identité| [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) |
| Emplacement des métadonnées de fédération| [https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}](https://login.windows.net/{tenant-id}/federationmetadata/2007-06/federationmetadata.xml?appid={application-id}) |


### <a name="map-sso-settings-for-saas-apps"></a>Mapper les paramètres d’authentification unique pour les applications SaaS

Les applications SaaS doivent savoir où envoyer les requêtes d’authentification et comment valider les jetons ainsi reçus. Le tableau suivant décrit les éléments de configuration des paramètres d’authentification unique dans l’application, ainsi que leurs valeurs et emplacements au sein d’AD FS et Azure AD

| Paramètre de configuration| AD FS| Configuration dans Azure AD |
| - | - | - |
| **URL de connexion IdP** <p>URL de connexion du fournisseur d’identité depuis l’application (où l’utilisateur est redirigé pour s’identifier).| L’URL de connexion AD FS correspond au nom de service de fédération AD FS, suivi par « /adfs/ls/ ». <p>Par exemple : `https://fs.contoso.com/adfs/ls/`| Remplacez {ID de locataire} par l’ID de votre locataire. <p> Pour les applications qui utilisent le protocole SAML-P : [https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p>Pour les applications qui utilisent le protocole WS-Federation : [https://login.microsoftonline.com/{tenant-id}/wsfed](https://login.microsoftonline.com/{tenant-id}/wsfed) |
| **URL de déconnexion IdP**<p>URL de déconnexion du fournisseur d’identité depuis l’application (où l’utilisateur est redirigé lorsqu’il choisit de se déconnecter de l’application).| L’URL de déconnexion correspond soit à l’URL de connexion, soit à l’URL de connexion suivie de « wa=wsignout1.0 ». Par exemple : `https://fs.contoso.com/adfs/ls/?wa=wsignout1.0`| Remplacez {ID de locataire} par l’ID de votre locataire.<p>Pour les applications qui utilisent le protocole SAML-P :<p>[https://login.microsoftonline.com/{tenant-id}/saml2](https://login.microsoftonline.com/{tenant-id}/saml2) <p> Pour les applications qui utilisent le protocole WS-Federation : [https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0](https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0) |
| **Certificat de signature de jetons**<p>L’IdP utilise la clé privée du certificat pour signer les jetons émis. Il vérifie que le jeton provient du fournisseur d’identité auquel l’application fait confiance.| Le certificat de signature de jetons AD FS se situe dans Gestion AD FS sous **Certificats**.| Recherchez-le dans le portail Azure dans les **Propriétés d’authentification unique** de l’application sous l’en-tête **Certificat de signature SAML**. Vous pouvez y télécharger le certificat pour le charger dans l’application.  <p>Si l’application possède plus d’un certificat, vous pouvez tous les trouver dans le fichier XML des métadonnées de fédération. |
| **Identificateur/« émetteur »**<p>Identificateur du fournisseur d’identité depuis l’application (parfois appelé « ID émetteur »).<p>Dans le jeton SAML, la valeur apparaît comme l’élément Émetteur.| L’identificateur pour AD FS correspond généralement à l’identificateur du service FS dans Gestion AD FS sous **Service -> Modifier propriétés du service FS**. Par exemple : `http://fs.contoso.com/adfs/services/trust`| Remplacez {ID de locataire} par l’ID de votre locataire.<p>https:\//sts.windows.net/{tenant-id}/ |
| **Métadonnées de fédération IdP**<p>Emplacement des métadonnées de fédération du fournisseur d’identité disponibles au public. (Certaines applications utilisent les métadonnées de fédération comme une alternative à l’administrateur pour configurer individuellement les URL, l’identificateur et le certificat de signature de jetons.)| L’URL des métadonnées de fédération AD FS se trouve dans Gestion AD FS sous **Service > Points de terminaison > Métadonnées > Type : métadonnées de fédération**. Par exemple : `https://fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`| La valeur correspondante pour Azure AD suit le modèle [https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/{TenantDomainName}/FederationMetadata/2007-06/FederationMetadata.xml). Remplacez {TenantDomainName} par votre nom de locataire au format « contoso.onmicrosoft.com ».   <p>Pour plus d’informations, consultez la section [Métadonnées de fédération](../azuread-dev/azure-ad-federation-metadata.md). |


## <a name="represent-ad-fs-security-policies-in-azure-ad"></a>Représenter les stratégies de sécurité d’AD FS dans Azure AD

Lors du déplacement de l’authentification de votre application vers Azure AD, créez des mappages à partir des stratégies de sécurité existantes vers leur équivalent ou leurs variantes alternatives disponibles dans Azure AD. S’assurer que ces mappages peuvent être effectués tout en répondant aux normes de sécurité requises par les propriétaires de votre application facilitera considérablement le reste de la migration de l’application.

Pour chaque type de règle et ses exemples, nous suggérons ici comment la règle se présente dans AD FS, le code équivalent en langage de règle AD FS et la façon dont le mappage se fait dans Azure AD.

### <a name="map-authorization-rules"></a>Règles d’autorisation de mappage

Voici quelques exemples de types de règles d’autorisation dans AD FS, ainsi que la façon dont vous pouvez les mapper à Azure AD :

#### <a name="example-1-permit-access-to-all-users"></a>Exemple 1 : Autoriser l’accès à tous les utilisateurs

L’option Autoriser l’accès à tous les utilisateurs ressemble à ceci dans AD FS :

![Capture d’écran montrant la boîte de dialogue Configurer l’authentification unique avec SAML.](media/migrate-adfs-apps-to-azure/sso-saml-user-attributes-claims.png)


Cela correspond à Azure AD d’une des manières suivantes :

Dans le [portail Azure](https://portal.azure.com/):
* Option 1 : Régler Affectation utilisateur requise ? sur Non ![Modifier la stratégie de contrôle d’accès pour les applications SaaS ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-2.png)

    Notez que la définition du commutateur Affectation de l'utilisateur obligatoire sur Oui nécessite que des utilisateurs soient affectés à l’application pour obtenir l’accès. Si la valeur est Non, tous les utilisateurs ont l’accès. Ce commutateur ne contrôle pas ce qui est affiché pour les utilisateurs dans l’expérience Mes applications.


* Option n°2 : Sous l’onglet Utilisateurs et groupes, affectez votre application au groupe automatique « Tous les utilisateurs ». <p>
Vous devez [activer les groupes dynamiques](../enterprise-users/groups-create-rule.md) dans votre locataire Azure AD pour que le groupe « Tous les utilisateurs » par défaut soit disponible.

   ![Mes applications SaaS dans Azure AD ](media/migrate-adfs-apps-to-azure/permit-access-to-all-users-3.png)


#### <a name="example-2-allow-a-group-explicitly"></a>Exemple 2 : Autoriser un groupe explicitement

Autorisation de groupe explicite dans AD FS :


![Capture d’écran montrant la boîte de dialogue Modifier la règle pour la règle de revendication pour autoriser les administrateurs du domaine.](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-1.png)


Voici comment la règle est mappée sur Azure AD :

Dans le [portail Azure](https://portal.azure.com/), vous devez d’abord [créer un groupe d’utilisateurs](../fundamentals/active-directory-groups-create-azure-portal.md) qui correspond au groupe d’utilisateurs d’AD FS, puis affecter des autorisations d’application à ce groupe :

![Ajouter une attribution ](media/migrate-adfs-apps-to-azure/allow-a-group-explicitly-2.png)


#### <a name="example-3-authorize-a-specific-user"></a>Exemple 3 : Autoriser un utilisateur spécifique

Autorisation explicite d’un utilisateur dans AD FS :

![Capture d’écran montrant la boîte de dialogue Modifier la règle pour la règle de revendication Autoriser un utilisateur spécifique avec le type de revendication entrante « SID principal ».](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-1.png)

Voici comment la règle est mappée sur Azure AD :

Dans le [portail Azure](https://portal.azure.com/), ajoutez un utilisateur à l’application via l’onglet Ajouter une affectation de l’application, comme indiqué ci-dessous :

![Mes applications SaaS dans Azure ](media/migrate-adfs-apps-to-azure/authorize-a-specific-user-2.png)


### <a name="map-multi-factor-authentication-rules"></a>Mappage de règles d’authentification multifacteur

Un déploiement local de [l’authentification multifacteur](../authentication/concept-mfa-howitworks.md) et d’AD FS continuera à fonctionner après la migration, car vous êtes fédéré avec AD FS. Toutefois, envisagez de migrer vers les fonctionnalités d’authentification multifacteur intégrées d’Azure, qui sont liées aux flux de travail d’accès conditionnel d’Azure AD.

Voici quelques exemples de types de règles d’authentification multifacteur dans AD FS, ainsi que la façon dont vous pouvez les mapper à Azure AD en fonction de différentes conditions :

Paramètres de la règle d’authentification multifacteur dans AD FS :

![Capture d’écran montrant les Conditions pour Azure AD dans le portail Azure.](media/migrate-adfs-apps-to-azure/mfa-location-1.png)


#### <a name="example-1-enforce-mfa-based-on-usersgroups"></a>Exemple 1 : Appliquer l’authentification multifacteur en fonction des utilisateurs/groupes

Le sélecteur Utilisateurs/groupes est une règle qui vous permet d’appliquer l’authentification multifacteur à des groupes (SID de groupe) ou des utilisateurs (SID principal). Outre les affectations d’utilisateurs/groupes, toutes les cases supplémentaires de l’interface utilisateur de configuration de l’authentification multifacteur d’AD FS fonctionnent comme des règles supplémentaires qui sont évaluées après l’application de la règle Utilisateur/Groupes.


Spécifiez les règles d’authentification multifacteur pour un utilisateur ou un groupe dans Azure AD :

1. Créez une [nouvelle stratégie d’accès conditionnel](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).

2. Sélectionnez **Affectations**. Ajoutez le ou les utilisateurs ou groupes pour lesquels vous souhaitez appliquer l’authentification multifacteur.

3. Configurez les options de **contrôle d’accès** comme indiqué ci-dessous :

![Capture d’écran montrant le volet Accorder dans lequel vous pouvez accorder l’accès.](media/migrate-adfs-apps-to-azure/mfa-usersorgroups.png)


 #### <a name="example-2-enforce-mfa-for-unregistered-devices"></a>Exemple 2 : Appliquer l’authentification multifacteur pour les appareils non inscrits

Spécifiez les règles d’authentification multifacteur pour les appareils non inscrits dans Azure AD :

1. Créez une [nouvelle stratégie d’accès conditionnel](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).

2. Définissez les **Affectations** sur **Tous les utilisateurs**.

3. Configurez les options de **contrôle d’accès** comme indiqué ci-dessous :

![Capture d’écran montrant le volet Accorder dans lequel vous pouvez accorder l’accès et spécifier d’autres restrictions.](media/migrate-adfs-apps-to-azure/mfa-unregistered-devices.png)


Lorsque vous définissez l’option Pour plusieurs contrôles sur Demander un des contrôles sélectionnés, cela signifie que si l’une des conditions spécifiées par la case à cocher est remplie par l’utilisateur, l’accès à votre application lui est accordé.

#### <a name="example-3-enforce-mfa-based-on-location"></a>Exemple 3 : Appliquer l’authentification multifacteur en fonction de l’emplacement

Spécifiez les règles d’authentification multifacteur en fonction de l’emplacement de l’utilisateur dans Azure AD :

1. Créez une [nouvelle stratégie d’accès conditionnel](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json).

1. Définissez les **Affectations** sur **Tous les utilisateurs**.

1. [Configurez des emplacements nommés dans Azure AD](../reports-monitoring/quickstart-configure-named-locations.md), sans quoi la fédération à partir de l’intérieur de votre réseau d’entreprise est approuvée.

1. Configurez des **Règles de conditions**  pour spécifier les emplacements pour lesquels vous souhaitez appliquer l’authentification multifacteur.

![Capture d’écran montrant le volet Emplacements pour les règles de Conditions.](media/migrate-adfs-apps-to-azure/mfa-location-1.png)

5. Configurez les options de **contrôle d’accès** comme indiqué ci-dessous :


![Mapper les stratégies de contrôle d’accès](media/migrate-adfs-apps-to-azure/mfa-location-2.png)


### <a name="map-emit-attributes-as-claims-rule"></a>Mapper les attributs d’émission en tant que règle de revendications

Voici un exemple de mappage des attributs dans AD FS :

![Capture d’écran montrant la boîte de dialogue Modifier la règle pour utiliser les attributs d’émission comme revendications.](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claimsrule-1.png)


Voici comment la règle est mappée sur Azure AD :

Dans le [portail Azure](https://portal.azure.com/), sélectionnez **Applications d’entreprise**, **Authentification unique**, puis ajoutez les **Attributs du jeton SAML** comme indiqué ci-dessous :

![Capture d’écran montrant la page Authentification unique de votre application d’entreprise.](media/migrate-adfs-apps-to-azure/map-emit-attributes-as-claimsrule-2.png)



### <a name="map-built-in-access-control-policies"></a>Mapper des stratégies de contrôle d’accès intégrées

AD FS 2016 possède plusieurs stratégies de contrôle d’accès intégrées que vous pouvez choisir :

![Contrôle d’accès intégré Azure AD](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-1.png)


Pour implémenter des stratégies intégrées dans Azure AD, vous pouvez utiliser une [nouvelle stratégie d’accès conditionnel](../authentication/tutorial-enable-azure-mfa.md?bc=%2fazure%2factive-directory%2fconditional-access%2fbreadcrumb%2ftoc.json&toc=%2fazure%2factive-directory%2fconditional-access%2ftoc.json) et configurer les contrôles d’accès, ou vous pouvez utiliser le concepteur de stratégie personnalisée dans AD FS 2016 pour configurer des stratégies de contrôle d’accès. L’éditeur de règles comprend une liste exhaustive d’options d’autorisation et d’exclusion qui peuvent vous aider à effectuer tous types de permutations.

![Stratégies de contrôle d’accès Azure AD](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-2.png)



Dans ce tableau, nous avons répertorié des options d’autorisation et d’exclusion utiles, ainsi que la façon dont elles sont mappées sur Azure AD.


| Option | Comment configurer l’option Autoriser dans Azure AD ?| Comment configurer l’option Exclure dans Azure AD ? |
| - | - | - |
| À partir d’un réseau spécifique| Correspond à un [Emplacement nommé](../reports-monitoring/quickstart-configure-named-locations.md) dans Azure AD| Utilisez l’option **Exclure** pour les [emplacements approuvés](../conditional-access/location-condition.md) |
| À partir de groupes spécifiques| [Définir une affectation Utilisateur/Groupes](./assign-user-or-group-access-portal.md)| Utiliser l’option **Exclure** dans les utilisateurs et les groupes |
| À partir d’appareils avec un niveau de confiance spécifique| Définissez cette valeur à partir du contrôle « État de l’appareil » sous Affectations -> Conditions| Utilisez l’option **Exclure** sous la condition État de l’appareil et Inclure **Tous les appareils** |
| Avec des revendications spécifiques dans la requête| Ce paramètre ne peut pas être migré| Ce paramètre ne peut pas être migré |


Voici un exemple de configuration de l’option Exclure pour les emplacements approuvés dans le portail Azure :

![Capture d’écran des stratégies de contrôle d’accès de mappage](media/migrate-adfs-apps-to-azure/map-builtin-access-control-policies-3.png)



## <a name="transition-users-from-ad-fs-to-azure-ad"></a>Transition d’utilisateurs d’AD FS à Azure AD

### <a name="sync-ad-fs-groups-in-azure-ad"></a>Synchronisation de groupes AD FS dans Azure AD

Lorsque vous mappez les règles d'autorisation, les applications qui s’authentifient avec AD FS peuvent utiliser des groupes Active Directory pour les autorisations. Dans ce cas, utilisez [Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) pour synchroniser ces groupes avec Azure AD avant de migrer les applications. Vérifiez ces groupes et les appartenances avant la migration afin de pouvoir accorder l’accès aux mêmes utilisateurs lors de la migration de l’application.

Pour plus d’informations, consultez [Conditions préalables à l’utilisation d’attributs de groupe synchronisés à partir d’Active Directory](../hybrid/how-to-connect-fed-group-claims.md).

### <a name="setup-user-self-provisioning"></a>Configurer la configuration automatique des utilisateurs

Certaines applications SaaS prennent en charge la possibilité de configurer automatiquement les utilisateurs lorsqu’ils se connectent pour la première fois à l’application. Dans Azure Active Directory (Azure AD), le terme approvisionnement de l’application désigne la création automatique des identités et rôles des utilisateurs dans les applications cloud ([SaaS](https://azure.microsoft.com/overview/what-is-saas/)) auxquelles les utilisateurs ont besoin d’accéder. Les utilisateurs migrés auront déjà un compte dans l’application SaaS. Les nouveaux utilisateurs ajoutés après la migration devront être configurés. Testez [le provisionnement d'applications SaaS](../app-provisioning/user-provisioning.md) fois l’application migrée.

### <a name="sync-external-users-in-azure-ad"></a>Synchroniser les utilisateurs externes dans Azure AD

Vos utilisateurs externes existants peuvent être configurés de deux manières principales dans AD FS :

#### <a name="external-users-with-a-local-account-within-your-organization"></a>Utilisateurs externes avec un compte local au sein de votre organisation

Vous pourrez continuer à utiliser ces comptes de la même façon que vos comptes d’utilisateur internes. Ces comptes d’utilisateur externes ont un nom principal au sein de votre organisation, même si l’e-mail du compte peut pointer en externe. À mesure que vous progressez dans votre migration, vous pouvez tirer parti des avantages [d’Azure AD B2B](../external-identities/what-is-b2b.md) en migrant ces utilisateurs pour qu’ils utilisent leur propre identité d’entreprise lorsqu’une telle identité est disponible. Cela simplifie le processus de connexion pour ces utilisateurs, car ils sont souvent connectés avec leur propre session d’entreprise. L’administration de votre organisation sera également facilitée, sans qu’il soit nécessaire de gérer les comptes pour les utilisateurs externes.

#### <a name="federated-external-identities"></a>Identités externes fédérées

Si vous vous fédérez avec une organisation externe, vous disposez de plusieurs approches pour effectuer les opérations suivantes :

* [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](../external-identities/add-users-administrator.md). Vous pouvez envoyer de manière proactive des invitations de collaboration B2B à partir du portail d’administration Azure AD à l’organisation partenaire pour que les membres individuels continuent à utiliser les applications et les ressources auxquelles ils sont habitués.

* [Créez un flux de travail d’inscription B2B en libre-service](../external-identities/self-service-portal.md) qui génère une requête pour des utilisateurs individuels de votre organisation partenaire à l’aide de l’API d’invitation B2B.

Quelle que soit la façon dont vos utilisateurs externes existants sont configurés, ils ont probablement des autorisations associées à leur compte, que ce soit dans l’appartenance à un groupe ou dans des autorisations spécifiques. Déterminez si ces autorisations doivent être migrées ou nettoyées. Les comptes au sein de votre organisation qui représentent un utilisateur externe doivent être désactivés une fois que l’utilisateur a été migré vers une identité externe. Le processus de migration doit être discuté avec vos partenaires commerciaux, car il peut y avoir une interruption de leur capacité à se connecter à vos ressources.

## <a name="migrate-and-test-your-apps"></a>Migrer et tester vos applications

Suivez le processus de migration détaillé dans cet article.

Accédez ensuite au [portail Azure](https://aad.portal.azure.com/) pour tester si la migration a réussi. Procédez comme suit :
1. Sélectionnez **Applications d’entreprise** > **Toutes les applications** et recherchez votre application dans la liste.

1. Sélectionnez **Gérer**  > **Utilisateurs et groupes** pour affecter au moins un utilisateur ou un groupe à l’application.

1. Sélectionnez **Gérer** > **Accès conditionnel**. Passez en revue votre liste de stratégies et assurez-vous que vous ne bloquez pas l’accès à l’application avec une [stratégie d’accès conditionnel](../conditional-access/overview.md).

Selon la façon dont vous configurez votre application, vérifiez que l’authentification unique fonctionne correctement.

| Type d'authentification| Test |
| - | - |
| OAuth/OpenID Connect| Sélectionnez **Applications d’entreprise -> Autorisations** et assurez-vous que vous avez accepté que l’application soit utilisée dans votre organisation dans les paramètres utilisateur de votre application.
‎ |
| Authentification unique SAML| Utilisez le bouton [Tester les paramètres SAML](./debug-saml-sso-issues.md) qui se trouve sous **Authentification unique**.
‎ |
| Authentification unique par mot de passe| Télécharger et installer [l’extension de connexion sécurisée ](../user-help/my-apps-portal-end-user-access.md)[-](../user-help/my-apps-portal-end-user-access.md)[ Mes applications](../user-help/my-apps-portal-end-user-access.md). Cette extension vous permet de lancer les applications cloud de votre organisation qui vous obligent à utiliser un processus d’authentification unique.
‎ |
| Application Proxy (Proxy d’application)| Vérifiez que votre connecteur est en cours d’exécution et affecté à votre application. Pour obtenir de l’aide, consultez le [Guide de résolution des problèmes de proxy d’application](./application-proxy-troubleshoot.md).
‎ |

> [!NOTE]
> Les cookies de l’ancien environnement AD FS resteront persistants sur les machines de l’utilisateur. Ces cookies peuvent entraîner des problèmes lors de la migration, car les utilisateurs peuvent être dirigés vers l’ancien environnement de connexion AD FS au lieu de la nouvelle connexion Azure AD. Vous devrez peut-être effacer manuellement les cookies du navigateur de l’utilisateur ou utiliser un script. Vous pouvez également utiliser le System Center Configuration Manager ou une plateforme similaire.

### <a name="troubleshoot"></a>Dépanner

En cas d’erreur dans le test des applications migrées, la résolution des problèmes pourrait être la première étape avant de revenir aux parties de confiance AD FS existantes. Consultez [Débogage d’une authentification unique basée sur SAML aux applications dans Azure Active Directory](./debug-saml-sso-issues.md).

### <a name="rollback-migration"></a>Restaurer la migration

Si la migration échoue, nous vous recommandons de laisser les parties de confiance existantes sur les serveurs AD FS et de supprimer l’accès aux parties de confiance. Cela permettra une solution de secours rapide si nécessaire pendant le déploiement.

### <a name="end-user-communication"></a>Communication auprès des utilisateurs finaux

Bien que la fenêtre d’indisponibilité planifiée elle-même puisse être minime, vous devez toujours prévoir de communiquer ces délais de manière proactive aux employés lorsque vous passez d’AD FS à Azure AD. Assurez-vous que votre application a un bouton Commentaires, ou des instructions de support technique en cas de problèmes.

Une fois le déploiement terminé, vous pouvez envoyer des communications informant les utilisateurs que le déploiement a réussi et leur rappelant les nouvelles étapes à suivre.

* Demandez aux utilisateurs d’utiliser le volet [Mes applications](https://myapps.microsoft.com) pour accéder à toutes les applications migrées.

* Rappelez aux utilisateurs qu’ils devront peut-être mettre à jour leurs paramètres d’authentification multifacteur.

* Si la réinitialisation du mot de passe en libre-service est déployée, les utilisateurs devront peut-être mettre à jour ou vérifier leurs méthodes d’authentification. Consultez les modèles de communication pour [l’authentification multifacteur](https://aka.ms/mfatemplates) et [SSPR](https://aka.ms/ssprtemplates).

Communication avec les utilisateurs externes : Ce groupe d’utilisateurs est généralement le plus lourdement affecté en cas de problème. Cela est particulièrement vrai si votre position de sécurité dicte un ensemble différent de règles d’accès conditionnel ou de profils de risque pour les partenaires externes. Assurez-vous que les partenaires externes sont au courant de la planification de la migration cloud et du fait qu’il y aura une plage de temps pendant laquelle ils seront encouragés à participer à un déploiement pilote qui teste tous les flux propres à la collaboration externe. Enfin, assurez-vous qu’ils disposent d’un moyen d’accéder à votre support technique en cas de problème cassant.

## <a name="next-steps"></a>Étapes suivantes

Lire [Migrer l’authentification d’application vers Azure AD](https://aka.ms/migrateapps/whitepaper)<p>
Configurer [l’accès conditionnel](../conditional-access/overview.md) et [l’authentification multifacteur](../authentication/concept-mfa-howitworks.md)

Essayez un exemple de code par étapes : [Playbook de migration d’application AD FS vers Azure AD pour les développeurs](https://aka.ms/adfsplaybook)
