---
title: Authentification unique basée sur l’en-tête pour les applications locales avec le Proxy d’application Azure AD
description: Découvrez comment fournir l’authentification unique pour les applications locales sécurisées avec une authentification basée sur l’en-tête.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/05/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: d39d72a79d4b273918986d0d350df4706592c77d
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95503171"
---
# <a name="header-based-single-sign-on-for-on-premises-apps-with-azure-ad-app-proxy-preview"></a>Authentification unique basée sur l’en-tête pour les applications locales avec le Proxy d’application Azure AD (préversion)

Le Proxy d’application Azure Active Directory (Azure AD) prend en charge en mode natif l’accès en authentification unique aux applications qui utilisent des en-têtes pour l’authentification. Vous pouvez configurer les valeurs d’en-tête que votre application requiert dans Azure AD. Les valeurs d’en-tête seront envoyées à l’application via un Proxy d’application. Voici quelques avantages de l’utilisation de la prise en charge native de l’authentification basée sur l’en-tête avec le Proxy d’application :  

* **Simplification de la fourniture de l’accès à distance à vos applications locales** : le Proxy d’application vous permet de simplifier votre architecture d’accès à distance existante. Vous pouvez remplacer l’accès VPN à ces applications. Vous pouvez également supprimer les dépendances sur les solutions d’identité locales pour l’authentification. Vos utilisateurs ne voient pas de différence quand ils se connectent pour utiliser vos applications d’entreprise. Ils pourront toujours travailler depuis n’importe où et sur n’importe quel appareil.  

* **Aucun logiciel supplémentaire ou aucune modification de vos applications** : vous pouvez utiliser vos connecteurs de Proxy d’application existants sans que cela nécessite l’installation de logiciels supplémentaires.  

* **Liste étendue d’attributs et de transformations disponibles** : toutes les valeurs d’en-tête disponibles sont basées sur des revendications standard émises par Azure AD. L’ensemble des attributs et transformations disponibles pour la [configuration des revendications pour les applications SAML ou OIDC](../develop/active-directory-saml-claims-customization.md#attributes) sont également disponibles pour être utilisés en tant que valeurs d’en-tête. 

## <a name="pre-requisites"></a>Conditions préalables
Avant de commencer à utiliser l’authentification unique pour les applications à authentification basée sur l’en-tête, assurez-vous que votre environnement est prêt avec les configurations et paramètres suivants :
- Vous devez activer le Proxy d’application et installer un connecteur disposant d’une ligne de site pour vos applications. Pour découvrir comment préparer votre environnement local, installer et inscrire un connecteur, et tester celui-ci, consultez le tutoriel [Ajouter une application locale pour un accès à distance via le service Proxy d’application](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad). 

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Le tableau suivant répertorie les fonctionnalités courantes requises pour les applications d’authentification basée sur l’en-tête prises en charge avec le service Proxy d’application. 

|Condition requise   |Description|
|----------|-----------|
|Authentification unique fédérée |En mode pré-authentifié, toutes les applications sont protégées avec l’authentification Azure AD, et permettent aux utilisateurs d’utiliser l’authentification unique. |
|Accès à distance |Le Proxy d’application permet l’accès à distance à l’application. Les utilisateurs peuvent accéder à l’application à partir d’Internet sur n’importe quel navigateur en utilisant l’URL externe. Le Proxy d’application n’est pas destiné à une utilisation pour un accès au réseau d’entreprise. |
|Intégration basée sur l’en-tête |Le Proxy d’application effectue l’intégration de l’authentification unique avec Azure AD, puis transmet l’identité ou d’autres données d’application en tant qu’en-têtes HTTP à l’application. |
|Autorisation de l’application |Vous pouvez spécifier des stratégies communes en fonction de l’application faisant l’objet de l’accès, de l’appartenance à un groupe de l’utilisateur et d’autres stratégies. Dans Azure AD, les stratégies sont implémentées à l’aide d’un [accès conditionnel](../conditional-access/overview.md). Les stratégies d’autorisation d’application s’appliquent uniquement à la demande d’authentification initiale. |
|Configurer l’authentification |Vous pouvez définir des stratégies pour forcer une authentification ajoutée, par exemple, pour accéder à des ressources sensibles. |
|Autorisation affinée |Fournit un contrôle d’accès au niveau de l’URL. Des stratégies ajoutées peuvent être appliquées en fonction de l’URL faisant l’objet de l’accès. L’URL interne configurée pour l’application définit l’étendue de l’application à laquelle la stratégie est appliquée. La stratégie configurée pour le chemin le plus granulaire est appliquée.  |

> [!NOTE] 
> Cet article présente la connexion d’applications à authentification basée sur l’en-tête à Azure AD à l’aide d’un Proxy d’application, qui est le modèle recommandé. En guise d’alternative, il existe également un modèle d’intégration utilisant PingAccess avec Azure AD pour activer l’authentification basée sur l’en-tête. Pour plus de détails, consultez [Authentification basée sur l’en-tête pour une authentification unique avec le Proxy d’application et PingAccess](application-proxy-ping-access-publishing-guide.md).

## <a name="how-it-works"></a>Fonctionnement

:::image type="content" source="./media/application-proxy-configure-single-sign-on-with-headers/how-it-works.png" alt-text="Fonctionnement de l’authentification unique basée sur l’en-tête avec le Proxy d’application." lightbox="./media/application-proxy-configure-single-sign-on-with-headers/how-it-works.png":::

1. L’administrateur personnalise les mappages d’attributs que requiert l’application dans le portail Azure AD. 
2. Quand un utilisateur accède à l’application, le Proxy d’application s’assure que l’utilisateur est authentifié par Azure AD. 
3. Le service cloud du Proxy d’application est informé des attributs requis. Le service extrait les revendications correspondantes du jeton d’ID reçu pendant l’authentification. Le service traduit ensuite les valeurs pour obtenir les en-têtes HTTP requis dans le cadre de la demande adressée au connecteur. 
4. La demande est alors transmise au connecteur qui est ensuite transmis à l’application principale. 
5. L’application reçoit les en-têtes et peut utiliser ceux-ci en fonction des besoins. 

## <a name="publish-the-application-with-application-proxy"></a>Publier l’application avec le Proxy d’application

1. Publiez votre application en suivant les instructions décrites dans  [Publier des applications avec le Proxy d’application](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).  
    - La valeur de l’URL interne détermine l’étendue de l’application. Si vous configurez la valeur de l’URL interne au niveau du chemin d’accès racine de l’application, tous les sous-chemins d’accès sous la racine recevront les mêmes configurations d’en-tête et d’application. 
    - Créez une application pour définir une configuration d’en-tête ou une attribution d’utilisateur différente pour un chemin d’accès plus granulaire que celui de l’application que vous avez configurée. Dans la nouvelle application, configurez l’URL interne avec le chemin d’accès spécifique dont vous avez besoin, puis configurez les en-têtes spécifiques nécessaires pour cette URL. Le Proxy d’application fera toujours correspondre vos paramètres de configuration au chemin d’accès le plus granulaire défini pour une application. 

2. Sélectionnez **Azure Active Directory** comme **méthode de pré-authentification**. 
3. Attribuez un utilisateur de test en accédant à **Utilisateurs et groupes**, puis en attribuant les utilisateurs et les groupes appropriés. 
4. Ouvrez un navigateur et accédez à l’ **URL externe** à partir des paramètres du Proxy d’application. 
5. Vérifiez que vous pouvez vous connecter à l’application. Même si vous pouvez vous connecter, vous ne pouvez pas encore accéder à l’application, car les en-têtes ne sont pas configurés. 

## <a name="configure-single-sign-on"></a>Configurer l’authentification unique 
Avant de commencer à utiliser l’authentification unique pour des applications basées sur un en-tête, vous devez avoir déjà installé un connecteur de Proxy d’application pouvant accéder aux applications cibles. Si ce n’est pas le cas, suivez les étapes décrites dans [Tutoriel : Proxy d’application Azure AD](application-proxy-add-on-premises-application.md), puis revenez ici. 

1. Dès que votre application apparaît dans la liste des applications d’entreprise, sélectionnez-la, puis sélectionnez  **Authentification unique**. 
2. Définissez le mode d’authentification unique sur **Basée sur l’en-tête**. 
3. Dans Configuration de base, **Azure Active Directory** est sélectionné par défaut. 
4. Sélectionnez le crayon de modification dans En-têtes pour configurer les en-têtes à envoyer à l’application. 
5. Sélectionnez **Ajouter un nouvel en-tête**. Fournissez un **Nom** pour l’en-tête, sélectionnez **Attribut** ou **Transformation**, puis sélectionnez dans la liste déroulante l’en-tête dont votre application a besoin.  
    - Pour en savoir plus sur la liste des attributs disponibles, consultez [Personnalisations des revendications- Attributs](../develop/active-directory-saml-claims-customization.md#attributes). 
    - Pour en savoir plus sur la liste des transformations disponibles, consultez [Personnalisations des revendications- Transformations de revendication](../develop/active-directory-saml-claims-customization.md#claim-transformations). 
    - Vous pouvez également ajouter un **En-tête de groupe** pour envoyer tous les groupes auxquels un utilisateur appartient ou les groupes attribués à l’application sous forme d’en-tête. Pour en savoir plus sur la configuration des groupes en tant que valeur, consultez : [Configurer des revendications de groupe pour les applications](../hybrid/how-to-connect-fed-group-claims.md#add-group-claims-to-tokens-for-saml-applications-using-sso-configuration). 
6. Sélectionnez Enregistrer. 

## <a name="test-your-app"></a>Test de l'application 

Une fois toutes ces étapes accomplies, votre application doit être opérationnelle. Pour tester l’application : 
1. Ouvrez une nouvelle fenêtre de navigateur ou une fenêtre de navigation privée pour que les en-têtes précédemment mis en cache soient bien effacés. Ensuite, accédez à  **l’URL externe** à partir des paramètres du Proxy d’application.
2. Connectez-vous au compte de test que vous avez attribué à l’application. Si vous pouvez charger l’application et vous y connecter à l’aide de l’authentification unique, tout va bien. 


## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce que l’authentification unique ?](what-is-single-sign-on.md)
- [Qu’est-ce que le Proxy d’application ?](what-is-application-proxy.md)
- [Série de guides de démarrage rapide sur la gestion des applications](view-applications-portal.md)
