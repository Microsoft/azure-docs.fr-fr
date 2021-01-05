---
title: Flux d’utilisateur dans Azure Active Directory B2C | Microsoft Docs
titleSuffix: Azure AD B2C
description: Découvrez plus d’informations sur le framework de stratégie extensible d’Azure Active Directory B2C et la création de différents flux d’utilisateur.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/30/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 06253b571fd71623501c27fd5b0d9d4013727fc2
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94840189"
---
# <a name="user-flows-in-azure-active-directory-b2c"></a>Flux d’utilisateur dans Azure Active Directory B2C

Pour vous aider à configurer les tâches d’identité les plus courantes pour vos applications, le portail Azure AD B2C inclut des stratégies configurables prédéfinies, appelées **flux d’utilisateur**. Un flux d’utilisateur vous permet de déterminer la façon dont les utilisateurs interagissent avec votre application quand ils effectuent différentes opérations : connexion, inscription, modification d’un profil, réinitialisation d’un mot de passe, etc. Avec les flux d’utilisateur, vous pouvez contrôler les éléments suivants :

- Types de compte utilisés pour la connexion, comme les comptes de réseaux sociaux de type Facebook ou les comptes locaux
- Attributs à collecter auprès du consommateur, comme le prénom, le code postal et la pointure
- Authentification multifacteur Azure AD
- Personnalisation de l’interface utilisateur
- Informations que l’application reçoit sous forme de revendications dans un jeton

Vous pouvez créer plusieurs flux d’utilisateur de types différents dans votre locataire et les utiliser dans vos applications selon vos besoins. Les flux d’utilisateurs peuvent être réutilisés entre les différentes applications. Cette flexibilité vous permet de définir et de modifier les expériences relatives à l’identité avec peu ou pas de modification du code. Votre application déclenche un flux d’utilisateur à l’aide d’une demande d’authentification HTTP standard qui inclut un paramètre de flux d’utilisateur. La réponse reçue est un [jeton](tokens-overview.md) personnalisé.

Les exemples suivants montrent le paramètre de chaîne de requête « p » qui spécifie le flux d’utilisateur à utiliser :

```
https://contosob2c.b2clogin.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siup                                       // Your sign-up user flow
```

```
https://contosob2c.b2clogin.com/contosob2c.onmicrosoft.com/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e      // Your registered Application ID
&redirect_uri=https%3A%2F%2Flocalhost%3A44321%2F    // Your registered Reply URL, url encoded
&response_mode=form_post                            // 'query', 'form_post' or 'fragment'
&response_type=id_token
&scope=openid
&nonce=dummy
&state=12345                                        // Any value provided by your application
&p=b2c_1_siin                                       // Your sign-in user flow
```

## <a name="user-flow-versions"></a>Versions de flux utilisateur

Azure AD B2C inclut plusieurs types de flux d’utilisateur.

- **Inscription et connexion** : Gère les expériences d’inscription et de connexion avec une seule configuration. Les utilisateurs sont dirigés vers le chemin approprié en fonction du contexte. Des flux d’utilisateur d’**inscription** et de **connexion** distincts sont également inclus. Toutefois, nous recommandons généralement le flux d’utilisateur d’inscription et de connexion combiné.
- **Modification de profil** : Permet aux utilisateurs de modifier leurs informations de profil.
- **Réinitialisation de mot de passe** : Vous permet de configurer si et comment les utilisateurs peuvent réinitialiser leur mot de passe.

Pour la plupart des types de flux d’utilisateur, une version **recommandée** et une version **standard** sont généralement disponibles. Pour plus d’informations, consultez [Versions de flux utilisateur](user-flow-versions.md).

> [!IMPORTANT]
> Si vous avez déjà travaillé avec des flux d’utilisateur dans Azure AD B2C, vous remarquerez que nous avons changé la manière dont nous référençons les versions de flux d’utilisateur. Avant, nous proposions des versions V1 (prêtes pour la production) et des versions V1.1 et V2 (préversions). Maintenant, nous avons regroupé les flux d’utilisateur en deux versions :
>
>- Les flux d’utilisateur **recommandés** sont les nouvelles préversions des flux d’utilisateur. Ils sont minutieusement testés et combinent toutes les fonctionnalités des versions **V2** et **V1.1** héritées. À l’avenir, les nouveaux flux d’utilisateur recommandés seront maintenus et mis à jour. Quand vous passez à ces nouveaux flux d’utilisateur recommandés, vous avez accès aux nouvelles fonctionnalités à mesure qu’elles sont publiées.
>- Les flux d’utilisateur **standard**, anciennement **V1**, sont en disponibilité générale et sont prêts pour la production. Si vous utilisez des flux d’utilisateur stratégiques, qui dépendent de versions très stables, vous pouvez continuer à utiliser des flux d’utilisateur standard tout en gardant à l’esprit que ces versions ne seront pas mises à jour ni maintenues.
>
>Tous les flux d’utilisateur en préversion hérités (V1.1 et V2) seront dépréciés d’ici le **1er août 2021**. Dans la mesure du possible, nous vous recommandons vivement de [basculer vers les nouveaux flux d’utilisateur **recommandés**](user-flow-versions.md#how-to-switch-to-a-new-recommended-user-flow) dès que possible pour continuer à tirer parti des dernières fonctionnalités et mises à jour.

## <a name="linking-user-flows"></a>Liaison de flux d’utilisateur

Un flux d’utilisateur **Inscription ou connexion** avec des comptes locaux comprend un lien **Mot de passe oublié ?** sur la première page. Lorsque vous cliquez sur ce lien, le flux d’utilisateur de réinitialisation du mot de passe n’est pas automatiquement déclenché.

À la place, le code d’erreur `AADB2C90118` est retourné à votre application. Votre application doit gérer ce code d’erreur en exécutant un flux d’utilisateur spécifique qui réinitialise le mot de passe. Pour voir un exemple, examinons un [simple exemple ASP.NET](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-DotNet-SUSI) qui illustre la liaison des flux d’utilisateur.

## <a name="email-address-storage"></a>Stockage d’adresse e-mail

Vous pouvez avoir besoin d’une adresse e-mail dans le cadre d’un flux d’utilisateur. Si l’utilisateur s’authentifie avec un fournisseur d’identité de réseau social, l’adresse e-mail est stockée dans la propriété **otherMails**. Si un compte local est basé sur un nom d’utilisateur, l’adresse e-mail est stockée dans une propriété détaillée d’authentification forte. Si un compte local est basé sur une adresse e-mail, l’adresse e-mail est stockée dans la propriété **signInNames**.

L’adresse e-mail n’est pas forcément vérifiée dans tous les cas. Un administrateur de locataire peut désactiver la vérification d’e-mail dans les stratégies de base des comptes locaux. Même si la vérification de l’adresse e-mail est activée, les adresses ne sont pas vérifiés si elles proviennent d’un fournisseur d’identité de réseau social et qu’elles n’ont pas été changées.

Seules les propriétés **otherMails** et **signInNames** sont exposées via l’API Microsoft Graph. L’adresse e-mail dans la propriété détaillée de l’authentification forte n’est pas disponible.

## <a name="next-steps"></a>Étapes suivantes

Pour créer les flux d’utilisateur recommandés, suivez les instructions du [Tutoriel : Créer un flux d’utilisateur](tutorial-create-user-flows.md).
