---
title: Authentification avec Microsoft Azure Maps
titleSuffix: Azure Maps
description: 'Découvrez les deux méthodes d’authentification des requêtes dans Azure Maps : authentification par clé partagée et authentification Azure Active Directory (Azure AD).'
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: d47e98273e696a4b6e827d8ebbc71a297f2861cb
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905313"
---
# <a name="authentication-with-azure-maps"></a>Authentification avec Azure Maps

Azure Maps prend en charge deux méthodes pour authentifier les requêtes : Authentification par clé partagée et authentification [Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md). Cet article explique les deux méthodes d’authentification pour vous guider dans votre implémentation des services Azure Maps.

> [!NOTE]
> Pour sécuriser davantage la communication avec Azure Maps, nous prenons désormais en charge le protocole TLS 1.2 et ne prenons plus en charge les versions 1.0 et 1.1 de ce protocole. Si vous utilisez actuellement TLS 1.x, découvrez si vous êtes prêt à passer à TLS 1.2 et créez un plan de migration avec les tests décrits dans [Résoudre les problèmes de dépendances TLS 1.0](/security/solving-tls1-problem).

## <a name="shared-key-authentication"></a>Authentification par clé partagée

 Les clés primaires et secondaires sont générées après la création du compte Azure Maps. Nous vous encourageons à utiliser la clé primaire comme clé d’abonnement lors de l’appel d’Azure Maps avec l’authentification par clé partagée. L’authentification par clé partagée transmet une clé générée par un compte Azure Maps à un service Azure Maps. Pour chaque requête envoyée aux services Azure Maps, ajoutez la *clé d’abonnement* à l’URL en tant que paramètre. La clé secondaire peut être utilisée dans des scénarios tels que le roulement des changements de clés.  

Pour plus d’informations sur l’affichage de vos clés dans le portail Azure, consultez [Gérer l’authentification](./how-to-manage-authentication.md#view-authentication-details).

> [!TIP]
> Pour des raisons de sécurité, il est recommandé de permuter vos clés principale et secondaire. Pour permuter les clés, mettez à jour votre application pour utiliser la clé secondaire, déployez, puis appuyez sur le bouton cycle/actualiser en regard de la clé principale pour générer une nouvelle clé principale. L’ancienne clé principale est désactivée. Pour plus d’informations sur la permutation des clés, consultez [Configurer Azure Key Vault avec la permutation des clés et l’audit](../key-vault/secrets/tutorial-rotation-dual.md)

## <a name="azure-ad-authentication"></a>Authentification Azure AD

Les abonnements Azure sont fournis avec un locataire Azure AD afin de permettre un contrôle d’accès affiné. Azure Maps offre une authentification pour les services Azure Maps à l’aide d’Azure AD. Azure AD fournit une authentification basée sur les identités pour les utilisateurs et les applications inscrits dans le locataire Azure AD.

Azure Maps accepte les jetons d’accès **OAuth 2.0** pour les locataires Azure AD associés à un abonnement Azure comprenant un compte Azure Maps. Azure Maps accepte également les jetons pour :

* Utilisateurs Azure AD
* Applications partenaires qui utilisent des autorisations déléguées par des utilisateurs
* Identités gérées pour les ressources Azure

Azure Maps génère un *identificateur unique (ID client)* pour chaque compte Azure Maps. Vous pouvez demander des jetons à Azure AD lorsque vous combinez cet ID client avec des paramètres supplémentaires.

Si vous souhaitez obtenir plus d’informations sur la configuration d’Azure AD et sur les requêtes de jetons pour Azure Maps, consultez [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md).

Pour obtenir des informations générales sur l’authentification auprès d’Azure AD, consultez [Qu’est-ce que l’authentification ?](../active-directory/develop/authentication-vs-authorization.md).

### <a name="managed-identities-for-azure-resources-and-azure-maps"></a>Identités managées pour les ressources Azure et Azure Maps

Les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) fournissent aux services Azure un principal de sécurité basé sur l’application et managé automatiquement qui peut s’authentifier auprès d’Azure AD. Avec le contrôle Azure d’accès en fonction du rôle (Azure RBAC), le principal de sécurité d’identités managées peut être autorisé à accéder aux services Azure Maps. Voici quelques exemples d’identités managées : Azure App Service, Azure Functions et Machines virtuelles Azure. Pour obtenir une liste d’identités managées, consultez [Identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md).

### <a name="configuring-application-azure-ad-authentication"></a>Configuration de l’authentification Azure AD des applications

Les applications s’authentifient auprès du locataire Azure AD par le biais d’un ou plusieurs scénarios pris en charge fournis par Azure AD. Chaque scénario d’application Azure AD représente des exigences différentes en fonction des besoins de l’entreprise. Certaines applications peuvent nécessiter des expériences de connexion utilisateur, tandis que d’autres peuvent nécessiter une expérience de connexion d’application. Pour plus d’informations, consultez [Flux d’authentification et scénarios d’applications](../active-directory/develop/authentication-flows-app-scenarios.md).

Une fois que l’application a reçu un jeton d’accès, le SDK et/ou l’application envoie une requête HTTPS avec l’ensemble suivant d’en-têtes HTTP requis en plus des autres en-têtes HTTP de l’API REST :

| Nom de l’en-tête    | Valeur               |
| :------------- | :------------------ |
| x-ms-client-id | 30d7cc….9f55        |
| Autorisation  | Bearer eyJ0e….HNIVN |

> [!NOTE]
> `x-ms-client-id` est le compte Azure Maps basé sur le GUID, qui apparaît dans la page d’authentification Azure Maps.

Voici un exemple de requête de routage Azure Maps qui utilise un jeton du porteur OAuth Azure AD :

```http
GET /route/directions/json?api-version=1.0&query=52.50931,13.42936:52.50274,13.43872
Host: atlas.microsoft.com
x-ms-client-id: 30d7cc….9f55
Authorization: Bearer eyJ0e….HNIVN
```

Pour plus d’informations sur l’affichage de votre ID client, consultez [Voir les détails de l’authentification](./how-to-manage-authentication.md#view-authentication-details).

## <a name="authorization-with-role-based-access-control"></a>Autorisation avec contrôle d’accès en fonction du rôle

Azure Maps prend en charge l’accès à tous les types de principaux pour le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md), notamment les utilisateurs Azure AD, les groupes, les applications, les ressources Azure et les identités managées Azure. Un jeu d’autorisations, également appelé définition de rôle, est accordé aux types de principaux. Une définition de rôle fournit des autorisations pour les actions de l’API REST. L’application de l’accès à un ou plusieurs comptes Azure Maps est appelée « étendue ». Lors de l’application d’un principal, d’une définition de rôle et d’une étendue, une attribution de rôle est créée. 

Les sections ci-après traitent des concepts et des composants de l’intégration d’Azure Maps à Azure RBAC. Dans le cadre du processus de configuration de votre compte Azure Maps, un annuaire Azure AD est associé à l’abonnement Azure dans lequel réside le compte Azure Maps. 

Quand vous configurez Azure RBAC, vous choisissez un principal de sécurité et l’appliquez à une attribution de rôle. Pour découvrir comment ajouter des attributions de rôles dans le portail Azure, consultez [Ajouter ou supprimer des attributions de rôles Azure](../role-based-access-control/role-assignments-portal.md).

### <a name="picking-a-role-definition"></a>Sélection d’une définition de rôle

Les types de définitions de rôles suivants sont disponibles pour prendre en charge les scénarios d’application.

| Définition de rôle Azure       | Description                                                                                              |
| :-------------------------- | :------------------------------------------------------------------------------------------------------- |
| Lecteur de données Azure Maps      | Fournit l’accès aux API REST Azure Maps immuables.                                                       |
| Contributeur aux données Azure Maps | Fournit l’accès aux API REST Azure Maps mutables. La mutabilité est définie par les actions Écrire et Supprimer. |
| Définition de rôle personnalisée      | Créez un rôle conçu pour permettre un accès limité flexible aux API REST Azure Maps.                      |

Certains services Azure Maps peuvent nécessiter des privilèges élevés pour effectuer des actions d’écriture ou de suppression sur les API REST Azure Maps. Le rôle Contributeur aux données Azure Maps est obligatoire pour les services qui fournissent des actions d’écriture ou de suppression. Le tableau suivant décrit les services auxquels s’applique le rôle Contributeur aux données Azure Maps lors de l’utilisation d’actions d’écriture ou de suppression sur le service donné. Si seules des actions de lecture sont utilisées sur le service, vous pouvez utiliser le rôle Lecteur de données Azure Maps plutôt que Contributeur aux données Azure Maps.

| Service Azure Maps | Définition de rôle Azure Maps  |
| :----------------- | :-------------------------- |
| Données               | Contributeur aux données Azure Maps |
| Creator            | Contributeur aux données Azure Maps |
| Spatial            | Contributeur aux données Azure Maps |

Pour plus d’informations sur la consultation de vos paramètres Azure RBAC, consultez [Comment configurer le contrôle Azure RBAC pour Azure Maps](./how-to-manage-authentication.md).

#### <a name="custom-role-definitions"></a>Définitions de rôles personnalisées

L’un des aspects de la sécurité des applications concerne l’application du principe du privilèges minimum. Ce principe implique qu’il ne faut accorder au principal de sécurité que l’accès nécessaire, et aucun accès supplémentaire. La création de définitions de rôles personnalisées peut prendre en charge des cas d’usage qui nécessitent une granularité supplémentaire du contrôle d’accès. Pour créer une définition de rôle personnalisée, vous pouvez sélectionner des actions de données spécifiques à inclure ou exclure pour la définition.

La définition de rôle personnalisée peut ensuite être utilisée dans une attribution de rôle pour n’importe quel principal de sécurité. Pour en savoir plus sur les définitions de rôles personnalisées Azure, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).

Voici quelques exemples de scénarios dans lesquels des rôles personnalisés peuvent améliorer la sécurité de l’application.

| Scénario                                                                                                                                                                                                                 | Action(s) de données de rôles personnalisés                                                                                                                  |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| Une page web de connexion interactive ou publique avec des mosaïques de base et aucune autre API REST.                                                                                                                              | `Microsoft.Maps/accounts/services/render/read`                                                                                              |
| Une application qui nécessite uniquement le géocodage inversé et aucune autre API REST.                                                                                                                                             | `Microsoft.Maps/accounts/services/search/read`                                                                                              |
| Un rôle pour un principal de sécurité qui demande la lecture de données cartographiques basées sur Azure Maps Creator et des API REST de mosaïques de base.                                                                                                 | `Microsoft.Maps/accounts/services/data/read`, `Microsoft.Maps/accounts/services/render/read`                                                |
| Un rôle pour un principal de sécurité qui nécessite la lecture, l’écriture et la suppression de données cartographiques basées sur Creator. Cela peut être défini en tant que rôle d’éditeur de données de carte, mais n’autorise pas l’accès à d’autres API REST comme les mosaïques de base. | `Microsoft.Maps/accounts/services/data/read`, `Microsoft.Maps/accounts/services/data/write`, `Microsoft.Maps/accounts/services/data/delete` |

### <a name="understanding-scope"></a>Présentation de l’étendue

Quand vous créez une attribution de rôle, elle est définie dans la hiérarchie de ressources Azure. En haut de la hiérarchie figure un [groupe d’administration](../governance/management-groups/overview.md), et tout en bas se trouve une ressource Azure, par exemple un compte Azure Maps.
L’attribution d’une attribution de rôle à un groupe de ressources peut permettre l’accès à plusieurs comptes Azure Maps ou à plusieurs ressources du groupe.

> [!TIP]
> La recommandation générale de Microsoft consiste à attribuer l’accès à l’étendue de compte Azure Maps, car cela empêche tout **accès involontaire à d’autres comptes Azure Maps** existants dans le même abonnement Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure RBAC, consultez :
> [!div class="nextstepaction"]
> [Contrôle d’accès en fonction du rôle Azure](../role-based-access-control/overview.md)

Pour plus d’informations sur l’authentification d’une application avec Azure AD et Azure Maps, consultez :
> [!div class="nextstepaction"]
> [Gérer l’authentification dans Azure Maps](./how-to-manage-authentication.md)

Pour plus d’informations sur l’authentification de Map Control d’Azure Maps avec Azure AD, consultez :
> [!div class="nextstepaction"]
> [Utiliser la bibliothèque Map Control d’Azure Maps](./how-to-use-map-control.md)