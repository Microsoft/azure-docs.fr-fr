---
title: Utiliser des attributs d’extension de schéma Azure AD dans les revendications
titleSuffix: Microsoft identity platform
description: Décrit l’utilisation des attributs d’extension de schéma d’annuaire pour envoyer des données utilisateur à des applications dans des revendications de jeton.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: how-to
ms.date: 07/29/2020
ms.author: ryanwi
ms.reviewer: paulgarn, hirsin, jeedes, luleon
ms.openlocfilehash: 8861e641f5ee6a10576425a7702ba02da297a0bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91631271"
---
# <a name="using-directory-schema-extension-attributes-in-claims"></a>Utiliser des attributs d’extension de schéma d’annuaire dans les revendications

Les attributs d’extension de schéma d’annuaire proposent une manière de stocker des données supplémentaires dans Azure Active Directory sur des objets utilisateur et d’autres objets d’annuaire, tels que les groupes, les détails des locataires ou les principaux du service.  Seuls les attributs d’extension sur les objets utilisateur peuvent être utilisés pour émettre des revendications vers des applications. Cet article décrit l’utilisation des attributs d’extension de schéma d’annuaire pour envoyer des données utilisateur à des applications dans des revendications de jeton.

> [!NOTE]
> Microsoft Graph fournit deux autres mécanismes d’extension pour personnaliser les objets Graph. Ceux-ci sont appelés extensions d’ouverture de Microsoft Graph et extensions de schéma Microsoft Graph. Pour plus de détails, consultez la [documentation Microsoft Graph](/graph/extensibility-overview). Les données stockées sur des objets Microsoft Graph qui utilisent ces fonctionnalités ne sont pas disponibles comme sources pour les revendications contenues dans les jetons.

Les attributs d’extension de schéma d’annuaire sont toujours associés à une application dans le locataire et référencés avec leur nom par l’*applicationId* de l’application.

L’identificateur d’un attribut d’extension de schéma d’annuaire se présente sous la forme *Extension_xxxxxxxxx_AttributeName*.  Où *xxxxxxxxd* est l’*applicationId* de l’application pour laquelle l’extension a été définie.

## <a name="registering-and-using-directory-schema-extensions"></a>Inscription et utilisation des extensions de schéma d’annuaire
Les attributs d’extension de schéma d’annuaire peuvent être inscrits et renseignés d’une de ces deux façons :

- En configurant AD Connect pour les créer et y synchroniser des données à partir de l’AD local. Consultez [Extensions d’annuaire de synchronisation Azure AD Connect](../hybrid/how-to-connect-sync-feature-directory-extensions.md).
- En utilisant Microsoft Graph pour inscrire, définir les valeurs et lire à partir d’[extensions de schéma](/graph/extensibility-overview). Des [applets de commande PowerShell](/powershell/azure/active-directory/using-extension-attributes-sample) sont également disponibles.

### <a name="emitting-claims-with-data-from-directory-schema-extension-attributes-created-with-ad-connect"></a>Émission de revendications avec les données des attributs d’extension de schéma d’annuaire créés à l’aide d’AD Connect
Les attributs de schéma d’extension d’annuaire créés et synchronisés à l’aide d’AD Connect sont toujours associés à l’ID d’application utilisé par AD Connect. Ils peuvent être utilisés comme source pour les revendications s’ils sont configurés comme des revendications dans la configuration **Applications d'entreprise** dans l’interface utilisateur du portail pour les applications SAML inscrites à l’aide de l’expérience de configuration de l’application galerie ou non-galerie sous **Applications d'entreprise** et via une stratégie de mappage des revendications pour les applications inscrites via l’expérience d’inscription d’application.  Une fois qu’un attribut d’extension d’annuaire créé via AD Connect figure dans l’annuaire, il affiche l’IU de la configuration des revendications SAML SSO.

### <a name="emitting-claims-with-data-from-directory-schema-extension-attributes-created-for-an-application-using-graph-or-powershell"></a>Émission de revendications avec les données des attributs d’extension de schéma d’annuaire créés pour une application qui utilise Graph ou PowerShell
Si un attribut d’extension de schéma d’annuaire est inscrit pour une application qui utilise Microsoft Graph ou PowerShell (par exemple, via une configuration initiale pour applications ou une étape d’approvisionnement), la même application peut être configurée dans Azure Active Directory pour recevoir des données dans cet attribut d’un objet utilisateur dans une revendication lorsque l’utilisateur se connecte.  Cette application peut être configurée pour recevoir des données dans des extensions de schéma d’annuaire qui ont été inscrites sur la même application à l’aide de [revendications facultatives](active-directory-optional-claims.md#configuring-directory-extension-optional-claims).  Celles-ci peuvent être configurées dans le manifeste de l'application.  Cela permet à une application multi-locataire d’inscrire les attributs d’extension de schéma d’annuaire pour son propre usage. Lorsque l’application est configurée dans un locataire, les extensions de schéma d’annuaire associées sont disponibles pour configuration sur les utilisateurs de ce locataire et pour consommation.  Une fois la configuration dans le locataire terminée et l’autorisation accordée, l’application peut être utilisée pour le stockage et la récupération de données via Graph, mais également pour le mappage des revendications contenues dans les jetons émis par la plateforme d’identité Microsoft vers les applications.

Les attributs d’extension de schéma d’annuaire peuvent être inscrits et renseignés pour n’importe quelle application.

Si une application doit envoyer des revendications avec des données d’attribut d’extension inscrites sur une application différente, une [stratégie de mappage des revendications](active-directory-claims-mapping.md) doit être utilisée pour mapper l’attribut d’extension vers la revendication.  Un modèle courant de gestion des attributs d’extension de schéma d’annuaire consiste à créer une application spécialement pour être le point d’inscription de toutes les extensions de schéma dont vous avez besoin.  Cela ne doit pas nécessairement être une application réelle et cette technique implique que toutes les extensions contiennent le même ID d’application dans leur nom.

Par exemple, voici la stratégie de mappage de revendications pour l’émission d’une revendication depuis un attribut d’extension de schéma d’annuaire dans un jeton OAuth/OIDC :

```json
{
    "ClaimsMappingPolicy": {
        "Version": 1,
        "IncludeBasicClaimSet": "false",
        "ClaimsSchema": [{
                "Source": "User",
                "ExtensionID": "extension_xxxxxxx_test",
                "JWTClaimType": "http://schemas.contoso.com/identity/claims/exampleclaim"
            },
        ]
    }
}
```

Lorsque *xxxxxxx* est l’ID d’application avec lequel l’extension a été inscrite.

> [!TIP]
> La cohérence de la casse est importante lors de la configuration des attributs d’extension d’annuaire sur les objets.  Les noms d’attribut d’extension ne sont pas sensibles à la casse lors de leur configuration, mais ils le sont lorsqu’ils sont lus par le service de jetons depuis l’annuaire.  Si un attribut d’extension est configuré sur un objet utilisateur avec le nom « LegacyId » et sur un autre objet utilisateur avec le nom « legacyid », lorsque l’attribut est mappé à une revendication avec le nom « LegacyId », les données sont récupérées et la revendication est incluse dans le jeton pour le premier utilisateur, mais pas pour le deuxième.
>
> Le paramètre « Id » dans le schéma des revendications utilisé pour les attributs de l’annuaire inclus est « ExtensionID » pour les attributs d’extension de l’annuaire.

## <a name="next-steps"></a>Étapes suivantes
- Découvrez la procédure d’[ajout de revendications supplémentaires ou personnalisées aux jetons SAML 2.0 et JSON Web Token (JWT)](active-directory-optional-claims.md).
- Découvrez la procédure de [personnalisation des revendications émises dans les jetons pour une application spécifique](active-directory-claims-mapping.md).