---
title: Configurer la connexion directe avec Azure Active Directory B2C | Microsoft Docs
description: Découvrez comment préremplir le nom de connexion ou rediriger directement vers un fournisseur d’identité sociale.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: b2342ed978204284bee4d2be0f1c983aa10ade36
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585035"
---
# <a name="set-up-direct-sign-in-using-azure-active-directory-b2c"></a>Configurer la connexion directe avec Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Quand vous configurez la connexion pour votre application avec Azure Active Directory (AD) B2C, vous pouvez préremplir le nom de connexion ou diriger la connexion vers un fournisseur d’identité sociale spécifique, comme Facebook, LinkedIn ou un compte Microsoft.

## <a name="prepopulate-the-sign-in-name"></a>Préremplir le nom de connexion

Pendant la procédure de connexion d’un utilisateur, une application par partie de confiance peut cibler un nom d’utilisateur ou de domaine spécifique. Quand vous ciblez un utilisateur, une application peut spécifier, dans la demande d’autorisation, le paramètre de requête `login_hint` avec le nom de connexion de l’utilisateur. Azure AD B2C remplit automatiquement le nom de connexion, l’utilisateur n’a que le mot de passe à fournir.

![Page d’inscription avec le paramètre de requête login_hint en surbrillance dans l’URL](./media/direct-signin/login-hint.png)

L’utilisateur peut changer la valeur présente dans la zone de texte de connexion.

::: zone pivot="b2c-custom-policy"

Pour prendre en charge le paramètre d’indice de connexion, remplacez le profil technique `SelfAsserted-LocalAccountSignin-Email`. Dans la section `<InputClaims>`, définissez la valeur DefaultValue de la revendication signInName sur `{OIDC:LoginHint}`. La variable `{OIDC:LoginHint}` contient la valeur du paramètre `login_hint`. Azure AD B2C lit la valeur de la revendication signInName et préremplit la zone de texte signInName.

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
      <InputClaims>
        <!-- Add the login hint value to the sign-in names claim type -->
        <InputClaim ClaimTypeReferenceId="signInName" DefaultValue="{OIDC:LoginHint}" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

::: zone-end

## <a name="redirect-sign-in-to-a-social-provider"></a>Rediriger la connexion vers un fournisseur social

Si vous avez configuré la procédure de connexion pour votre application afin d’inclure des comptes de réseaux sociaux comme Facebook, LinkedIn ou Google, vous pouvez spécifier le paramètre `domain_hint`. Ce paramètre de requête fournit un indicateur à Azure AD B2C concernant le fournisseur d’identité sociale qui doit être utilisé pour la connexion. Par exemple, si l’application spécifie `domain_hint=facebook.com`, la connexion accède directement à la page de connexion Facebook.

![Page d’inscription avec le paramètre de requête domain_hint en surbrillance dans l’URL](./media/direct-signin/domain-hint.png)

::: zone pivot="b2c-user-flow"

Le paramètre de chaîne de requête d’indice de domaine peut être défini sur l’un des domaines suivants :

- amazon.com
- facebook.com
- github.com
- google.com
- linkedin.com
- microsoft.com
- qq.com
- twitter.com
- wechat.com
- weibo.com 
- Pour [Generic OpenID Connect](identity-provider-generic-openid-connect.md), consultez [Indication du domaine](identity-provider-generic-openid-connect.md#response-mode).

::: zone-end

::: zone pivot="b2c-custom-policy"

Pour prendre en charge un paramètre d’indice du domaine, vous pouvez configurer le nom de domaine à l’aide de l’élément XML `<Domain>domain name</Domain>` de toute section `<ClaimsProvider>`.

```xml
<ClaimsProvider>
    <!-- Add the domain hint value to the claims provider -->
    <Domain>facebook.com</Domain>
    <DisplayName>Facebook</DisplayName>
    <TechnicalProfiles>
    ...
```

::: zone-end

