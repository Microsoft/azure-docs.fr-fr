---
title: Vue d’ensemble des clés de stratégie – Azure Active Directory B2C
description: Découvrez les types de clés de stratégie de chiffrement qui peuvent être utilisés dans Azure Active Directory B2C pour la signature et la validation des jetons, des clés secrètes clients, des certificats et des mots de passe.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 30348d7ca12ded2d1f4b0522a7cabeadf0553a07
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953353"
---
# <a name="overview-of-policy-keys-in-azure-active-directory-b2c"></a>Vue d’ensemble des clés de stratégie dans Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) stocke les secrets et les certificats sous forme de clés de stratégie pour établir une relation de confiance avec les services auxquels il s’intègre. Ces approbations sont constituées des éléments suivants :

- Fournisseurs d’identité externes
- Connexion avec les [services d’API REST](restful-technical-profile.md)
- Signature et chiffrement des jetons

 Cet article traite de ce que vous devez savoir sur les clés de stratégie utilisées par Azure AD B2C.

> [!NOTE]
> Actuellement, la configuration des clés de stratégie est limitée aux [stratégies personnalisées](./custom-policy-get-started.md) uniquement.

Vous pouvez configurer des secrets et des certificats pour établir la confiance entre les services dans le portail Azure, sous le menu **Clés de stratégie**. Les clés peuvent être symétriques ou asymétriques. Le chiffrement *symétrique*, ou chiffrement à clé privée, consiste à utiliser un secret partagé est utilisé pour chiffrer et déchiffrer les données. Le chiffrement *asymétrique*, ou chiffrement à clé publique, est un système de chiffrement qui utilise des paires de clés, composées de clés publiques partagées avec l’application par partie de confiance et des clés privées connues uniquement d’Azure AD B2C.

## <a name="policy-keyset-and-keys"></a>Clés et jeu de clés de stratégie

La ressource de niveau supérieur pour les clés de stratégie dans Azure AD B2C est le conteneur **Keyset**. Chaque jeu de clés contient au moins une **clé**. Une clé a les attributs suivants :

| Attribut |  Obligatoire | Notes |
| --- | --- |--- |
| `use` | Oui | Usage : Identifie l’utilisation prévue de la clé publique. Chiffrement des données `enc` ou vérification de la signature sur des données `sig`.|
| `nbf`| Non | Date et heure d’activation. |
| `exp`| Non | Date et heure d’expiration. |

Nous vous recommandons de définir les valeurs d’activation et d’expiration des clés conformément à vos normes PKI. Vous devrez peut-être alterner régulièrement ces certificats pour des raisons de sécurité ou de stratégie. Par exemple, une stratégie peut demander d’alterner l’ensemble de vos certificats tous les ans.

Pour créer une clé, vous pouvez choisir l’une des méthodes suivantes :

- **Manuelle** : crée un secret avec une chaîne que vous définissez. Le secret est une clé symétrique. Vous pouvez définir les dates d’activation et d’expiration.
- **Générée** : génère automatiquement une clé. Vous pouvez définir les dates d’activation et d’expiration. Nous avons deux options :
  - **Secret** : génère une clé symétrique.
  - **RSA** : génère une paire de clés (clés asymétriques).
- **Charger** : charge un certificat ou une clé PKCS12. Le certificat doit contenir les clés privées et publiques (clés asymétriques).

## <a name="key-rollover"></a>Substitution de clé

Pour des raisons de sécurité, Azure AD B2C peut substituer des clés régulièrement ou immédiatement en cas d’urgence. Toute application, tout fournisseur d’identité ou toute API REST qui sont intégrés à Azure AD B2C doivent être prêt à gérer un événement de substitution de clé, quelle que soit sa fréquence. Dans le cas contraire, si votre application ou Azure AD B2C tente d’utiliser une clé expirée pour effectuer une opération de chiffrement, la demande de connexion échouera.

Si un jeu de clés Azure AD B2C possède plusieurs clés, une seule d’entre elles est active à la fois, en fonction des critères suivants :

- L’activation de la clé est basée sur la **date d’activation**.
  - Les clés sont triées par date d’activation dans l’ordre croissant. Les clés dont la date d’activation se situe plus loin dans le futur apparaissent plus bas dans la liste. Les clés sans date d’activation sont situées tout en bas de la liste.
  - Lorsque la date et l’heure actuelles sont supérieures à la date d’activation d’une clé, Azure AD B2C active la clé et cesse d’utiliser la clé active précédente.
- Lorsque l’heure d’expiration de la clé actuelle est passée et que le conteneur de clés contient une nouvelle clé avec des heures *pas avant* et *expiration* valides, la nouvelle clé devient active automatiquement.
- Lorsque l’heure d’expiration de la clé actuelle est passée et que le conteneur de clés *ne contient pas* de nouvelle clé avec des heures *pas avant* et *expiration* valides, Azure AD B2C ne peut pas utiliser la clé expirée. Azure AD B2C générera un message d’erreur dans un composant dépendant de votre stratégie personnalisée. Pour éviter ce problème, vous pouvez créer une clé par défaut sans dates d’activation et d’expiration par mesure de sécurité.
- Le point de terminaison de la clé (URI JWKS) du point de terminaison de configuration bien connu d’OpenId Connect reflète les clés configurées dans le conteneur de clés, lorsque la clé est référencée dans le [profil technique JwtIssuer](./jwt-issuer-technical-profile.md). Une application qui utilise une bibliothèque OIDC récupèrera automatiquement ces métadonnées pour s’assurer qu’elle utilise les bonnes clés pour valider les jetons. Pour plus d’informations, découvrez comment utiliser la [bibliothèque d’authentification de Microsoft](../active-directory/develop/msal-b2c-overview.md), qui récupère toujours automatiquement les clés de signature de jetons les plus récentes.

## <a name="policy-key-management"></a>Gestion des clés de stratégie

Pour obtenir la clé active actuelle dans un conteneur de clés, utilisez le point de terminaison [getActiveKey](/graph/api/trustframeworkkeyset-getactivekey) de l’API Microsoft Graph.

Pour ajouter ou supprimer des clés de signature et de chiffrement :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page de vue d’ensemble, sous **Stratégies**, sélectionnez **Identity Experience Framework**.
1. Sélectionner **Clés de stratégie** 
    1. Pour ajouter une nouvelle clé, sélectionnez **Ajouter**.
    1. Pour supprimer une nouvelle clé, sélectionnez la clé, puis sélectionnez **Supprimer**. Pour supprimer la clé, saisissez le nom du conteneur de clés à supprimer. Azure AD B2C supprimera la clé et créera une copie de la clé avec le suffixe .bak.

### <a name="replace-a-key"></a>Remplacer une clé

Les clés d’un jeu de clés ne sont pas remplaçables ni amovibles. Si vous avez besoin de modifier une clé existante :

- Nous vous recommandons d’ajouter une nouvelle clé avec la **date d’activation** définie sur la date et l’heure actuelles. Azure AD B2C activera la nouvelle clé et cessera d’utiliser la clé active précédente.
- Vous pouvez également créer un nouveau jeu de clés avec les clés appropriées. Mettez à jour votre stratégie pour utiliser le nouveau jeu de clés, puis supprimez l’ancien jeu de clés. 

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser Microsoft Graph pour automatiser le déploiement d’un [jeu de clés](microsoft-graph-operations.md#trust-framework-policy-keyset) et de [clés de stratégie](microsoft-graph-operations.md#trust-framework-policy-key).