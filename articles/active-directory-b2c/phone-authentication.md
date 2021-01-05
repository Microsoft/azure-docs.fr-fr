---
title: Inscription et connexion par téléphone avec des stratégies personnalisées
titleSuffix: Azure AD B2C
description: Envoyez des mots de passe à usage unique (OTP) dans des messages texte vers les téléphones des utilisateurs de votre application à l’aide de stratégies personnalisées dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/01/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 9d4fa80f669493e4dc47a9ad0f9bfe9390d4ab24
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953574"
---
# <a name="set-up-phone-sign-up-and-sign-in-with-custom-policies-in-azure-ad-b2c"></a>Configurer l’inscription et la connexion par téléphone avec des stratégies personnalisées dans Azure AD B2C

L’inscription et la connexion par téléphone dans Azure Active Directory B2C (Azure AD B2C) permet à vos utilisateurs de s’inscrire et de se connecter à vos applications en utilisant un mot de passe à usage unique (OTP) envoyé par SMS sur leur téléphone. Les mots de passe à usage unique permettent de réduire le risque que vos utilisateurs oublient leur mot de passe ou que ce mot de passe soit piraté.

Suivez les étapes de cet article pour utiliser des stratégies personnalisées permettant à vos clients de s’inscrire et de se connecter à vos applications en utilisant un mot de passe à usage unique envoyé sur leur téléphone.

## <a name="pricing"></a>Tarifs

Les mots de passe uniques sont envoyés à vos utilisateurs par SMS, et vous pouvez être facturé pour chaque message envoyé. Pour obtenir des informations sur la tarification, voir la section **Frais distincts** dans [Tarification Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/).

## <a name="user-experience-for-phone-sign-up-and-sign-in"></a>Expérience utilisateur pour l’inscription et la connexion par téléphone

Avec l’inscription et la connexion par téléphone, l’utilisateur peut s’inscrire à l’application en utilisant un numéro de téléphone comme identificateur principal. L’expérience de l’utilisateur final lors de l’inscription et de la connexion est décrite ci-dessous.

> [!NOTE]
> Nous vous suggérons vivement d’inclure des informations relatives au consentement dans votre expérience d’inscription et de connexion, comme dans l’exemple de texte ci-dessous. Cet exemple de texte est fourni à titre d’information uniquement. Reportez-vous au manuel relatif à la surveillance des codes courts sur le [site web de CTIA](https://www.ctia.org/programs) et consultez vos propres experts en droit ou en conformité pour obtenir des conseils sur le texte final et la configuration des fonctionnalités afin de répondre à vos propres besoins en matière de conformité :
>
> *En fournissant votre numéro de téléphone, vous acceptez de recevoir un code secret à usage unique envoyé par SMS pour vous aider à vous connecter à *&lt;insérer : le nom de votre application&gt;* . Des frais standard de messagerie et de données peuvent s’appliquer.*
>
> *&lt;insérer : lien vers votre déclaration de confidentialité&gt;*<br/>*&lt;insérer : lien vers vos conditions d’utilisation du service&gt;*

Pour ajouter vos propres informations relatives au consentement, personnalisez l’exemple suivant et incluez-le dans la propriété LocalizedResources du paramètre ContentDefinition utilisé par la page d’auto-déclaration avec le contrôle d’affichage (le fichier Phone-Email-Base.xml dans le pack de démarrage de d’inscription et de connexion par téléphone) :

```xml
<LocalizedResources Id="phoneSignUp.en">        
    <LocalizedStrings>
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_msg_intro">By providing your phone number, you consent to receiving a one-time passcode sent by text message to help you sign into {insert your application name}. Standard messsage and data rates may apply.</LocalizedString>          
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_1_text">Privacy Statement</LocalizedString>                
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_1_url">{insert your privacy statement URL}</LocalizedString>          
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_2_text">Terms and Conditions</LocalizedString>             
    <LocalizedString ElementType="DisplayControl" ElementId="phoneControl" StringId="disclaimer_link_2_url">{insert your terms and conditions URL}</LocalizedString>          
    <LocalizedString ElementType="UxElement" StringId="initial_intro">Please verify your country code and phone number</LocalizedString>        
    </LocalizedStrings>      
</LocalizedResources>
   ```

### <a name="phone-sign-up-experience"></a>Expérience d’inscription par téléphone

Si l’utilisateur n’a pas encore de compte pour votre application, il peut en créer un en choisissant le lien **S’inscrire**. Une page d’inscription s’affiche, dans laquelle l’utilisateur sélectionne son **pays**, entre son numéro de téléphone et sélectionne **Envoyer un code**.

![L’utilisateur commence l’inscription par téléphone](media/phone-authentication/phone-signup-start.png)

Un code de vérification unique est envoyé au numéro de téléphone de l’utilisateur. L’utilisateur entre le **code de vérification** sur la page d’inscription, puis il sélectionne **Vérifier le code**. (Si l’utilisateur n’a pas pu récupérer le code, il peut sélectionner **Envoyer un nouveau code**.)

![L’utilisateur vérifie le code lors de l’inscription par téléphone](media/phone-authentication/phone-signup-verify-code.png)

 L’utilisateur entre toutes les autres informations requises sur la page d’inscription, par exemple, le **nom d’affichage**, le **prénom** et le **nom de famille** (le pays et le numéro de téléphone sont déjà renseignés). Si l’utilisateur souhaite utiliser un autre numéro de téléphone, il peut choisir **Changer de numéro** pour recommencer l’inscription. Lorsque il a terminé, l’utilisateur sélectionne **Continuer**.

![L’utilisateur fournit des informations supplémentaires](media/phone-authentication/phone-signup-additional-info.png)

Ensuite, l’utilisateur est invité à fournir une adresse e-mail de récupération. L’utilisateur entre son adresse e-mail, puis sélectionne **Envoyer le code de vérification**. Un code est envoyé à l’adresse e-mail de l’utilisateur, qu’il peut récupérer et entrer dans la zone **Code de vérification**. L’utilisateur sélectionne ensuite **Vérifier le code**. 

Une fois le code vérifié, l’utilisateur sélectionne **Créer** pour créer son compte. Ou, si l’utilisateur souhaite utiliser une autre adresse e-mail, il peut choisir **Changer d’adresse e-mail**.

![L’utilisateur crée un compte](media/phone-authentication/email-verification.png)

### <a name="phone-sign-in-experience"></a>Expérience de connexion par téléphone

Si l’utilisateur possède déjà un compte avec un numéro de téléphone comme identificateur, il entre son numéro de téléphone et sélectionne **Continuer**. Il confirme le pays et le numéro de téléphone en sélectionnant **Continuer**, et un code de vérification unique est envoyé sur son téléphone. L’utilisateur entre le code de vérification et sélectionne **Continuer** pour se connecter.

![Expérience utilisateur de connexion par téléphone](media/phone-authentication/phone-signin-screens.png)

## <a name="deleting-a-user-account"></a>Suppression d’un compte d’utilisateur

Dans certains cas, vous devrez peut-être supprimer un utilisateur et les données associées de votre répertoire Azure AD B2C. Pour plus d’informations sur la suppression d’un compte d’utilisateur par le biais du portail Azure, consultez [ces instructions](/microsoft-365/compliance/gdpr-dsr-azure#step-5-delete). 

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]



## <a name="prerequisites"></a>Prérequis

Vous devez disposer des ressources suivantes avant de configurer un mot de passe unique.

* [Locataire Azure AD B2C](tutorial-create-tenant.md)
* [Application web inscrite](tutorial-register-applications.md) dans votre locataire
* [Stratégies personnalisées](custom-policy-get-started.md) téléchargées sur votre locataire

## <a name="get-the-phone-sign-up--sign-in-starter-pack"></a>Obtenir le pack de démarrage pour l’inscription et la connexion par téléphone

Commencez par mettre à jour les fichiers de stratégies personnalisées d’inscription et de connexion par téléphone afin de les utiliser avec votre locataire Azure AD B2C.

Les étapes suivantes supposent que vous avez défini les [préférences](#prerequisites) et que vous avez déjà cloné le référentiel du [pack de démarrage de stratégies personnalisées][starter-pack] sur votre machine locale.

1. Vous trouverez les [fichiers de stratégies personnalisés d’inscription et de connexion par téléphone][starter-pack-phone] dans le clone local du référentiel de votre pack de démarrage, ou vous pouvez directement les télécharger. Les fichiers de stratégie XML se trouvent dans le répertoire suivant :

    `active-directory-b2c-custom-policy-starterpack/scenarios/`**`phone-number-passwordless`**

1. Dans chaque fichier, remplacez la chaîne `yourtenant` par le nom de votre locataire Azure AD B2C. Par exemple, si le nom de votre locataire B2C est *contosob2c*, toutes les instances de `yourtenant.onmicrosoft.com` deviennent `contosob2c.onmicrosoft.com`.

1. Effectuez les étapes décrites dans la section [Ajouter des ID d’applications à la stratégie personnalisée](custom-policy-get-started.md#add-application-ids-to-the-custom-policy) de la rubrique [Bien démarrer avec les stratégies personnalisées dans Azure Active Directory B2C](custom-policy-get-started.md). Dans ce cas, mettez à jour `/phone-number-passwordless/`**`Phone_Email_Base.xml`** avec les **ID d’application (client)** des deux applications que vous avez enregistrées en remplissant les conditions préalables, *IdentityExperienceFramework* et *ProxyIdentityExperienceFramework*.

## <a name="upload-the-policy-files"></a>Télécharger les fichiers de stratégie

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et accédez à votre locataire Azure AD B2C.
1. Sous **Stratégies**, sélectionnez **Identity Experience Framework**.
1. Sélectionnez **Charger une stratégie personnalisée**.
1. Téléchargez les fichiers de stratégie dans l'ordre suivant :
    1. *Phone_Email_Base.xml*
    1. *SignUpOrSignInWithPhone.xml*
    1. *SignUpOrSignInWithPhoneOrEmail.xml*
    1. *ProfileEditPhoneOnly.xml*
    1. *ProfileEditPhoneEmail.xml*
    1. *ChangePhoneNumber.xml*
    1. *PasswordResetEmail.xml*

À mesure que vous téléchargez chaque fichier, Azure ajoute le préfixe `B2C_1A_`.

## <a name="test-the-custom-policy"></a>Tester la stratégie personnalisée

1. Sous **Stratégies personnalisées**, sélectionnez **B2C_1A_SignUpOrSignInWithPhone**.
1. Sous **Sélectionner l’application**, sélectionnez l’application *webapp1* que vous avez enregistrée lorsque vous avez rempli les conditions préalables.
1. Pour **Sélectionner l’URL de réponse**, choisissez `https://jwt.ms`.
1. Sélectionnez **Exécuter maintenant** et inscrivez-vous en utilisant une adresse e-mail ou un numéro de téléphone.
1. Sélectionnez de nouveau **Exécuter maintenant** et connectez-vous avec le même compte pour vérifier que votre configuration est correcte.

## <a name="get-user-account-by-phone-number"></a>Obtenir un compte d’utilisateur par numéro de téléphone

Un utilisateur qui s'inscrit avec un numéro de téléphone, mais qui ne fournit pas d'adresse électronique de récupération est enregistré dans votre annuaire B2C Azure AD avec son numéro de téléphone comme nom de connexion. Si l'utilisateur souhaite ensuite modifier son numéro de téléphone, votre service d'assistance ou votre équipe de support doit d'abord identifier son compte, puis mettre à jour son numéro de téléphone.

Vous pouvez identifier un utilisateur par son numéro de téléphone (nom de connexion) en utilisant [Microsoft Graph](manage-user-accounts-graph-api.md) :

```http
GET https://graph.microsoft.com/v1.0/users?$filter=identities/any(c:c/issuerAssignedId eq '+{phone number}' and c/issuer eq '{tenant name}.onmicrosoft.com')
```

Par exemple :

```http
GET https://graph.microsoft.com/v1.0/users?$filter=identities/any(c:c/issuerAssignedId eq '+450334567890' and c/issuer eq 'contosob2c.onmicrosoft.com')
```

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez des informations sur l’inscription par téléphone et le pack de démarrage de stratégie personnalisée (et d'autres packs de démarrage) sur GitHub : [Azure-Samples/active-directory-b2c-custom-policy-starterpack/scenarios/phone-number-passwordless][starter-pack-phone]. Les fichiers de stratégie du pack de démarrage utilisent des profils techniques d’authentification multifacteur et des transformations de revendications de numéro de téléphone :
* [Définir un profil technique Azure AD Multi-Factor Authentication](multi-factor-auth-technical-profile.md)
* [Définir des transformations de revendications de numéros de téléphone](phone-number-claims-transformations.md)

<!-- LINKS - External -->
[starter-pack]: https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
[starter-pack-phone]: https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/phone-number-passwordless