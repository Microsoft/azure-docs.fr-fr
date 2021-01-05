---
title: ID de chaînes de localisation - Azure Active Directory B2C | Microsoft Docs
description: Spécifiez les ID d’une définition de contenu avec un ID api.signuporsignin dans une stratégie personnalisée au sein d'Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 11/09/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 4fa0f4d87567bd77faacfd2373dba0d2b8996446
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94873367"
---
# <a name="localization-string-ids"></a>ID de chaînes de localisation

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **Localization** vous permet de prendre en charge plusieurs paramètres régionaux ou langues dans la stratégie pour les parcours utilisateur. Cet article fournit la liste des ID de localisation que vous pouvez utiliser dans votre stratégie. Pour vous familiariser avec la localisation de l’interface utilisateur, consultez [Localisation](localization.md).

## <a name="sign-up-or-sign-in-page-elements"></a>Éléments de page de connexion ou d’inscription

Les ID suivants sont utilisés pour une définition de contenu avec un ID de `api.signuporsignin`, et un [profil technique autodéclaré](self-asserted-technical-profile.md).

| id | Valeur par défaut | Version de mise en page |
| -- | ------------- | ------ |
| **forgotpassword_link** | Vous avez oublié votre mot de passe ? | `All` |
| **createaccount_intro** | Vous n’avez pas de compte ? | `All` |
| **button_signin** | Se connecter | `All` |
| **social_intro** | Connectez-vous avec votre compte social | `All` |
| **remember_me** |Maintenir la connexion. | `All` |
| **unknown_error** | Nous rencontrons des problèmes pour vous connecter. Veuillez réessayer plus tard. | `All` |
| **divider_title** | OR | `All` |
| **local_intro_email** | Connectez-vous avec votre compte existant | `< 2.0.0` |
| **logonIdentifier_email** | Adresse de messagerie | `< 2.0.0` |
| **requiredField_email** | Entrez votre adresse e-mail | `< 2.0.0` |
| **invalid_email** | Entrez une adresse e-mail valide | `< 2.0.0` |
| **email_pattern** | ^[a-zA-Z0-9.!#$%&"\*+/=?^\_\`{\|}~-]+@[a-zA-Z0-9-]+(?:\\.[a-zA-Z0-9-]+)\*$ | `< 2.0.0` |
| **local_intro_username** | Connectez-vous avec votre nom d'utilisateur | `< 2.0.0` |
| **logonIdentifier_username** | Nom d’utilisateur | `< 2.0.0` |
| **requiredField_username** | Entrez votre nom d'utilisateur | `< 2.0.0` |
| **mot de passe** | Mot de passe | `< 2.0.0` |
| **requiredField_password** | Entrez votre mot de passe | `< 2.0.0` |
| **createaccount_link** | ici | `< 2.0.0` |
| **cancel_message** | L'utilisateur a oublié son mot de passe | `< 2.0.0` |
| **invalid_password** | Le mot de passe que vous avez entré n'est pas au format attendu. | `< 2.0.0` |
| **createaccount_one_link** | ici | `>= 2.0.0` |
| **createaccount_two_links** | Inscription avec {0} ou {1} | `>= 2.0.0` |
| **createaccount_three_links** | Inscription avec {0}, {1} ou {2} | `>= 2.0.0` |
| **local_intro_generic** | Connectez-vous avec votre {0} | `>= 2.1.0` |
| **requiredField_generic** | Veuillez entrer votre {0} | `>= 2.1.0` |
| **invalid_generic** | Entrez une valeur {0} valide | `>= 2.1.1` |
| **heading** | Se connecter | `>= 2.1.1` |


> [!NOTE]
> * Les espaces réservés comme {0} seront remplis automatiquement avec la valeur `DisplayName` de `ClaimType`. 
> * Pour savoir comment localiser `ClaimType`, consultez [Exemple d’inscription ou de connexion](#signupsigninexample).

L’exemple suivant illustre l’utilisation de certains des éléments d’interface utilisateur dans la page d’inscription ou de connexion :

:::image type="content" source="./media/localization-string-ids/localization-susi-2.png" alt-text="Capture d’écran montrant les éléments Expérience utilisateur de la page d’inscription ou de connexion.":::

### <a name="sign-up-or-sign-in-identity-providers"></a>Fournisseurs d’identité d’inscription ou de connexion

L’ID des fournisseurs d’identité est configuré dans l’élément **ClaimsExchange** du parcours utilisateur. Pour localiser le titre du fournisseur d’identité, `ClaimsProvider` est affecté comme valeur d’**ElementType**, tandis que l’ID de `ClaimsExchange` est affecté comme valeur de **StringId**.

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="MicrosoftExchange" TechnicalProfileReferenceId="MSA-OIDC" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccount" />
  </ClaimsExchanges>
</OrchestrationStep>
```

L’exemple suivant localise le fournisseur d’identité Facebook pour l’arabe :

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">فيس بوك</LocalizedString>
```

### <a name="sign-up-or-sign-in-error-messages"></a>Messages d’erreur d’inscription ou de connexion

| id | Valeur par défaut |
| -- | ------------- |
| **UserMessageIfInvalidPassword** | Votre mot de passe est incorrect. |
| **UserMessageIfPasswordExpired**| Votre mot de passe a expiré.|
| **UserMessageIfClaimsPrincipalDoesNotExist** | Nous ne trouvons pas votre compte. |
| **UserMessageIfOldPasswordUsed** | Apparemment, vous utilisez un ancien mot de passe. |
| **DefaultMessage** | Nom d’utilisateur ou mot de passe non valide. |
| **UserMessageIfUserAccountDisabled** | Votre compte a été verrouillé. Contactez votre support technique pour le déverrouiller, puis réessayez. |
| **UserMessageIfUserAccountLocked** | Votre compte est temporairement verrouillé pour éviter toute utilisation non autorisée. Réessayez plus tard. |
| **AADRequestsThrottled** | Il y a trop de demandes pour l’instant. Veuillez patienter quelques instants, puis réessayez. |

<a name="signupsigninexample"></a>
### <a name="sign-up-or-sign-in-example"></a>Exemple d’inscription ou de connexion

```xml
<LocalizedResources Id="api.signuporsignin.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="heading">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="social_intro">Sign in with your social account</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_intro_generic">Sign in with your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_password">Please enter your password</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_generic">Please enter your {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_generic">Please enter a valid {0}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_one_link">Sign up now</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_two_links">Sign up with {0} or {1}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="createaccount_three_links">Sign up with {0}, {1}, or {2}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="forgotpassword_link">Forgot your password?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_signin">Sign in</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="divider_title">OR</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="unknown_error">We are having trouble signing you in. Please try again later.</LocalizedString>
    <!-- Uncomment the remember_me only if the keep me signed in is activated. 
    <LocalizedString ElementType="UxElement" StringId="remember_me">Keep me signed in</LocalizedString> -->
    <LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidPassword">Your password is incorrect.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfPasswordExpired">Your password has expired.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalDoesNotExist">We can't seem to find your account.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfOldPasswordUsed">Looks like you used an old password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultMessage">Invalid username or password.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountDisabled">Your account has been locked. Contact your support person to unlock it, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfUserAccountLocked">Your account is temporarily locked to prevent unauthorized use. Try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="AADRequestsThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="sign-up-and-self-asserted-pages-user-interface-elements"></a>Éléments d’interface utilisateur des pages d’inscription et autodéclarées

Voici les ID pour une définition de contenu avec un ID `api.localaccountsignup` ou pour toute définition de contenu qui commence par `api.selfasserted`, telle que `api.selfasserted.profileupdate` et `api.localaccountpasswordreset`, et un [profil technique autodéclaré](self-asserted-technical-profile.md).

| id | Valeur par défaut |
| -- | ------------- |
| **ver_sent** | Le code de vérification a été envoyé à : |
| **ver_but_default** | Default |
| **cancel_message** | L'utilisateur a annulé la saisie d'informations auto-confirmées |
| **preloader_alt** | Veuillez patienter |
| **ver_but_send** | Envoyer le code de vérification |
| **alert_yes** | Oui |
| **error_fieldIncorrect** | Un ou plusieurs champs sont incorrectement renseignés. Vérifiez vos entrées et réessayez. |
| **year** | Year |
| **verifying_blurb** | Veuillez patienter pendant que nous traitons vos informations. |
| **button_cancel** | Annuler |
| **ver_fail_no_retry** | Vous avez effectué trop de tentatives incorrectes. Veuillez réessayer plus tard. |
| **month** | Month |
| **ver_success_msg** | Adresse e-mail vérifiée. Vous pouvez maintenant continuer. |
| **months** | Janvier, février, mars, avril, mai, juin, juillet, août, septembre, octobre, novembre, décembre |
| **ver_fail_server** | Nous rencontrons des problèmes de la vérification de votre adresse e-mail. Entrez une adresse e-mail valide et réessayez. |
| **error_requiredFieldMissing** | Un champ obligatoire est manquant. Veuillez remplir tous les champs obligatoires, puis réessayez. |
| **initial_intro** | Fournissez les détails suivants. |
| **ver_but_resend** | Envoyer le nouveau code |
| **button_continue** | Créer |
| **error_passwordEntryMismatch** | Les champs d’entrée de mot de passe ne correspondent pas. Entrez le même mot de passe dans les deux champs et réessayez. |
| **ver_incorrect_format** | Format incorrect. |
| **ver_but_edit** | Modifier l'adresse e-mail |
| **ver_but_verify** | Vérifier le code |
| **alert_no** | Non |
| **ver_info_msg** | Le code de vérification a été envoyé à votre boîte de réception. Veuillez le copier dans la zone d’entrée ci-dessous. |
| **day** | jour |
| **ver_fail_throttled** | Il y a eu trop de demandes pour vérifier cette adresse e-mail. Veuillez patienter quelques instants, puis réessayez. |
| **helplink_text** | Qu'est-ce que c'est ? |
| **ver_fail_retry** | Ce code est incorrect. Réessayez. |
| **alert_title** | Annuler la saisie de vos informations |
| **required_field** | Ces informations sont obligatoires. |
| **alert_message** | Voulez-vous vraiment annuler la saisie de vos informations ? |
| **ver_intro_msg** | La vérification est nécessaire. Cliquez sur le bouton Envoyer. |
| **ver_input** | Code de vérification |

### <a name="sign-up-and-self-asserted-pages-error-messages"></a>Messages d’erreur des pages d’inscription et autodéclarées

| id | Valeur par défaut |
| -- | ------------- |
| **UserMessageIfClaimsPrincipalAlreadyExists** | Un utilisateur avec l’ID spécifié existe déjà. Choisissez-en un autre. |
| **UserMessageIfClaimNotVerified** | Revendication non vérifiée : {0} |
| **UserMessageIfIncorrectPattern** | Modèle incorrect pour : {0} |
| **UserMessageIfMissingRequiredElement** | Élément requis manquant : {0} |
| **UserMessageIfValidationError** | Erreur lors de la validation par : {0} |
| **UserMessageIfInvalidInput** | {0} a une entrée non valide. |
| **ServiceThrottled** | Il y a trop de demandes pour l’instant. Veuillez patienter quelques instants, puis réessayez. |

L’exemple suivant illustre l’utilisation de certains des éléments d’interface utilisateur dans la page d’inscription :

![Page d’inscription avec ses noms d’éléments d’interface utilisateur étiquetés](./media/localization-string-ids/localization-sign-up.png)

L’exemple suivant illustre l’utilisation de certains des éléments d’interface utilisateur dans la page d’inscription, une fois que l’utilisateur a cliqué sur le bouton Envoyer le code de vérification :

![Éléments de l’expérience utilisateur de vérification d’e-mail dans la page d’inscription](./media/localization-string-ids/localization-email-verification.png)

## <a name="sign-up-and-self-asserted-pages-example"></a>Exemple de pages d’inscription et autodéclarées

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email Address</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Email address that can be used to contact you.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="DisplayName">New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="UserHelpText">Enter new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="newPassword" StringId="PatternHelpText">8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="DisplayName">Confirm New Password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="UserHelpText">Confirm new password</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="reenterPassword" StringId="PatternHelpText">#8-16 characters, containing 3 out of 4 of the following: Lowercase characters, uppercase characters, digits (0-9), and one or more of the following symbols: @ # $ % ^ &amp; * - _ + = [ ] { } | \ : ' , ? / ` ~ " ( ) ; .</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="DisplayName">Display Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="displayName" StringId="UserHelpText">Your display name.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="DisplayName">Surname</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="surname" StringId="UserHelpText">Your surname (also known as family name or last name).</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="DisplayName">Given Name</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="givenName" StringId="UserHelpText">Your given name (also known as first name).</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_continue">Create</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_fieldIncorrect">One or more fields are filled out incorrectly. Please check your entries and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_passwordEntryMismatch">The password entry fields do not match. Please enter the same password in both fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_requiredFieldMissing">A required field is missing. Please fill out all required fields and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="helplink_text">What is this?</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="initial_intro">Please provide the following details.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="preloader_alt">Please wait</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="required_field">This information is required.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_edit">Change e-mail</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_resend">Send new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_send">Send verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_but_verify">Verify code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_code_expired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_no_retry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_retry">That code is incorrect. Please try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_server">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_fail_throttled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_info_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_input">Verification code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="ver_success_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="ServiceThrottled">There are too many requests at this moment. Please wait for some time and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimNotVerified">Claim not verified: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">A user with the specified ID already exists. Please choose a different one.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfIncorrectPattern">Incorrect pattern for: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidInput">{0} has invalid input.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMissingRequiredElement">Missing required element: {0}</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfValidationError">Error in validation by: {0}</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="phone-factor-authentication-page-user-interface-elements"></a>Éléments d’interface utilisateur de la page d’authentification (facteur de forme de téléphone)

Voici les ID pour une définition de contenu avec un ID `api.phonefactor` et un [profil technique de facteur de forme de téléphone](phone-factor-technical-profile.md).

| id | Valeur par défaut |
| -- | ------------- |
| **button_verify** | M’appeler |
| **country_code_label** | Indicatif de pays |
| **cancel_message** | L'utilisateur a annulé l'authentification multifacteur |
| **text_button_send_second_code** | envoyer un nouveau code |
| **code_pattern** | \\d{6} |
| **intro_mixed** | Nous avons le numéro suivant en mémoire pour vous. Nous pouvons envoyer un code par SMS ou nous pouvons appeler pour vous authentifier. |
| **intro_mixed_p** | Nous avons les numéros suivants en mémoire pour vous. Choisissez un numéro que nous pouvons appeler ou auquel envoyer un code par SMS afin de vous authentifier. |
| **button_verify_code** | Vérifier le code |
| **requiredField_code** | Entrez le code de vérification que vous avez reçu |
| **invalid_code** | Entrez le code à 6 chiffres que vous avez reçu |
| **button_cancel** | Annuler |
| **local_number_input_placeholder_text** | Numéro de téléphone |
| **button_retry** | Recommencer |
| **alternative_text** | Je n'ai pas mon téléphone |
| **intro_phone_p** | Nous avons les numéros suivants en mémoire pour vous. Choisissez un numéro que nous pouvons appeler pour vous authentifier. |
| **intro_phone** | Nous avons le numéro suivant en mémoire pour vous. Nous allons appeler afin de vous authentifier. |
| **enter_code_text_intro** | Entrez votre code de vérification ci-dessous, ou  |
| **intro_entry_phone** | Entrez un numéro ci-dessous que nous pouvons appeler pour vous authentifier. |
| **intro_entry_sms** | Entrez un numéro ci-dessous auquel nous pouvons envoyer un code par SMS pour vous authentifier. |
| **button_send_code** | Envoyer le code |
| **invalid_number** | Entrez un numéro de téléphone valide |
| **intro_sms** | Nous avons le numéro suivant en mémoire pour vous. Nous enverrons un code par SMS afin de vous authentifier. |
| **intro_entry_mixed** | Entrez un numéro ci-dessous auquel nous pouvons envoyer un code par SMS ou que nous pouvons appeler pour vous authentifier. |
| **number_pattern** | ^\\+(?:[0-9][\\x20-]?){6,14}[0-9]$ |
| **intro_sms_p** |Nous avons les numéros suivants en mémoire pour vous. Choisissez un numéro auquel nous pouvons envoyer un code par SMS afin de vous authentifier. |
| **requiredField_countryCode** | Sélectionnez le code de votre pays |
| **requiredField_number** | Entrez votre numéro de téléphone |
| **country_code_input_placeholder_text** |Pays ou région |
| **number_label** | Numéro de téléphone |
| **error_tryagain** | Le numéro de téléphone que vous avez fourni est occupé ou non disponible. Veuillez vérifier le numéro et réessayer. |
| **error_incorrect_code** | Le code de vérification que vous avez entré ne correspond pas à nos enregistrements. Réessayez ou demandez un nouveau code. |
| **countryList** | Consultez [la liste des pays](#phone-factor-authentication-page-example). |
| **error_448** | Le numéro de téléphone que vous avez fourni n'est pas accessible. |
| **error_449** | L'utilisateur a dépassé le nombre de nouvelles tentatives. |
| **verification_code_input_placeholder_text** | Code de vérification |

L’exemple suivant illustre l’utilisation de certains des éléments d’interface utilisateur dans la page d’inscription MFA :

![Éléments d’expérience utilisateur d’inscription de l’authentification pour facteur de téléphone](./media/localization-string-ids/localization-mfa1.png)

L’exemple suivant illustre l’utilisation de certains des éléments d’interface utilisateur dans la page de validation MFA :

![Éléments d’expérience utilisateur de validation de l’authentification pour facteur de téléphone](./media/localization-string-ids/localization-mfa2.png)

## <a name="phone-factor-authentication-page-example"></a>Exemple de page d’authentification pour facteur de téléphone

```xml
<LocalizedResources Id="api.phonefactor.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="button_verify">Call Me</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_label">Country Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="cancel_message">The user has canceled multi-factor authentication</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="text_button_send_second_code">Send a new code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="code_pattern">\d{6}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed">We have the following number on record for you. We can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_mixed_p">We have the following numbers on record for you. Choose a number that we can phone or send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_code">Please enter the verification code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_code">Please enter the 6-digit code you received</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="local_number_input_placeholder_text">Phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_retry">Retry</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="alternative_text">I don't have my phone</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone_p">We have the following numbers on record for you. Choose a number that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_phone">We have the following number on record for you. We will phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="enter_code_text_intro">Enter your verification code below, or</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_phone">Enter a number below that we can phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_sms">Enter a number below that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="button_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="invalid_number">Please enter a valid phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms">We have the following number on record for you. We will send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_entry_mixed">Enter a number below that we can send a code via SMS or phone to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_pattern">^\+(?:[0-9][\x20-]?){6,14}[0-9]$</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="intro_sms_p">We have the following numbers on record for you. Choose a number that we can send a code via SMS to authenticate you.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_countryCode">Please select your country code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="requiredField_number">Please enter your phone number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="country_code_input_placeholder_text">Country or region</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="number_label">Phone Number</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_tryagain">The phone number you provided is busy or unavailable. Please check the number and try again.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_incorrect_code">The verification code you have entered does not match our records. Please try again, or request a new code.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="countryList">{"DEFAULT":"Country/Region","AF":"Afghanistan","AX":"Åland Islands","AL":"Albania","DZ":"Algeria","AS":"American Samoa","AD":"Andorra","AO":"Angola","AI":"Anguilla","AQ":"Antarctica","AG":"Antigua and Barbuda","AR":"Argentina","AM":"Armenia","AW":"Aruba","AU":"Australia","AT":"Austria","AZ":"Azerbaijan","BS":"Bahamas","BH":"Bahrain","BD":"Bangladesh","BB":"Barbados","BY":"Belarus","BE":"Belgium","BZ":"Belize","BJ":"Benin","BM":"Bermuda","BT":"Bhutan","BO":"Bolivia","BQ":"Bonaire","BA":"Bosnia and Herzegovina","BW":"Botswana","BV":"Bouvet Island","BR":"Brazil","IO":"British Indian Ocean Territory","VG":"British Virgin Islands","BN":"Brunei","BG":"Bulgaria","BF":"Burkina Faso","BI":"Burundi","CV":"Cabo Verde","KH":"Cambodia","CM":"Cameroon","CA":"Canada","KY":"Cayman Islands","CF":"Central African Republic","TD":"Chad","CL":"Chile","CN":"China","CX":"Christmas Island","CC":"Cocos (Keeling) Islands","CO":"Colombia","KM":"Comoros","CG":"Congo","CD":"Congo (DRC)","CK":"Cook Islands","CR":"Costa Rica","CI":"Côte d'Ivoire","HR":"Croatia","CU":"Cuba","CW":"Curaçao","CY":"Cyprus","CZ":"Czech Republic","DK":"Denmark","DJ":"Djibouti","DM":"Dominica","DO":"Dominican Republic","EC":"Ecuador","EG":"Egypt","SV":"El Salvador","GQ":"Equatorial Guinea","ER":"Eritrea","EE":"Estonia","ET":"Ethiopia","FK":"Falkland Islands","FO":"Faroe Islands","FJ":"Fiji","FI":"Finland","FR":"France","GF":"French Guiana","PF":"French Polynesia","TF":"French Southern Territories","GA":"Gabon","GM":"Gambia","GE":"Georgia","DE":"Germany","GH":"Ghana","GI":"Gibraltar","GR":"Greece","GL":"Greenland","GD":"Grenada","GP":"Guadeloupe","GU":"Guam","GT":"Guatemala","GG":"Guernsey","GN":"Guinea","GW":"Guinea-Bissau","GY":"Guyana","HT":"Haiti","HM":"Heard Island and McDonald Islands","HN":"Honduras","HK":"Hong Kong SAR","HU":"Hungary","IS":"Iceland","IN":"India","ID":"Indonesia","IR":"Iran","IQ":"Iraq","IE":"Ireland","IM":"Isle of Man","IL":"Israel","IT":"Italy","JM":"Jamaica","JP":"Japan","JE":"Jersey","JO":"Jordan","KZ":"Kazakhstan","KE":"Kenya","KI":"Kiribati","KR":"Korea","KW":"Kuwait","KG":"Kyrgyzstan","LA":"Laos","LV":"Latvia","LB":"Lebanon","LS":"Lesotho","LR":"Liberia","LY":"Libya","LI":"Liechtenstein","LT":"Lithuania","LU":"Luxembourg","MO":"Macao SAR","MK":"North Macedonia","MG":"Madagascar","MW":"Malawi","MY":"Malaysia","MV":"Maldives","ML":"Mali","MT":"Malta","MH":"Marshall Islands","MQ":"Martinique","MR":"Mauritania","MU":"Mauritius","YT":"Mayotte","MX":"Mexico","FM":"Micronesia","MD":"Moldova","MC":"Monaco","MN":"Mongolia","ME":"Montenegro","MS":"Montserrat","MA":"Morocco","MZ":"Mozambique","MM":"Myanmar","NA":"Namibia","NR":"Nauru","NP":"Nepal","NL":"Netherlands","NC":"New Caledonia","NZ":"New Zealand","NI":"Nicaragua","NE":"Niger","NG":"Nigeria","NU":"Niue","NF":"Norfolk Island","KP":"North Korea","MP":"Northern Mariana Islands","NO":"Norway","OM":"Oman","PK":"Pakistan","PW":"Palau","PS":"Palestinian Authority","PA":"Panama","PG":"Papua New Guinea","PY":"Paraguay","PE":"Peru","PH":"Philippines","PN":"Pitcairn Islands","PL":"Poland","PT":"Portugal","PR":"Puerto Rico","QA":"Qatar","RE":"Réunion","RO":"Romania","RU":"Russia","RW":"Rwanda","BL":"Saint Barthélemy","KN":"Saint Kitts and Nevis","LC":"Saint Lucia","MF":"Saint Martin","PM":"Saint Pierre and Miquelon","VC":"Saint Vincent and the Grenadines","WS":"Samoa","SM":"San Marino","ST":"São Tomé and Príncipe","SA":"Saudi Arabia","SN":"Senegal","RS":"Serbia","SC":"Seychelles","SL":"Sierra Leone","SG":"Singapore","SX":"Sint Maarten","SK":"Slovakia","SI":"Slovenia","SB":"Solomon Islands","SO":"Somalia","ZA":"South Africa","GS":"South Georgia and South Sandwich Islands","SS":"South Sudan","ES":"Spain","LK":"Sri Lanka","SH":"St Helena, Ascension, Tristan da Cunha","SD":"Sudan","SR":"Suriname","SJ":"Svalbard","SZ":"Swaziland","SE":"Sweden","CH":"Switzerland","SY":"Syria","TW":"Taiwan","TJ":"Tajikistan","TZ":"Tanzania","TH":"Thailand","TL":"Timor-Leste","TG":"Togo","TK":"Tokelau","TO":"Tonga","TT":"Trinidad and Tobago","TN":"Tunisia","TR":"Turkey","TM":"Turkmenistan","TC":"Turks and Caicos Islands","TV":"Tuvalu","UM":"U.S. Outlying Islands","VI":"U.S. Virgin Islands","UG":"Uganda","UA":"Ukraine","AE":"United Arab Emirates","GB":"United Kingdom","US":"United States","UY":"Uruguay","UZ":"Uzbekistan","VU":"Vanuatu","VA":"Vatican City","VE":"Venezuela","VN":"Vietnam","WF":"Wallis and Futuna","YE":"Yemen","ZM":"Zambia","ZW":"Zimbabwe"}</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_448">The phone number you provided is unreachable.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="error_449">User has exceeded the number of retry attempts.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_code_input_placeholder_text">Verification code</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="strongAuthenticationPhoneNumber" StringId="DisplayName">Phone Number</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>

```

## <a name="verification-display-control-user-interface-elements"></a>Éléments de l’interface utilisateur du contrôle de l’affichage de vérification

Voici les ID d’un [contrôle d’affichage de vérification](display-control-verification.md) avec la [version de mise en page](page-layout.md) 2.1.0 ou ultérieure.

| id | Valeur par défaut |
| -- | ------------- |
|intro_msg| La vérification est nécessaire. Cliquez sur le bouton Envoyer.|
|success_send_code_msg | Le code de vérification a été envoyé à votre boîte de réception. Veuillez le copier dans la zone d’entrée ci-dessous.|
|failure_send_code_msg | Nous rencontrons des problèmes de la vérification de votre adresse e-mail. Entrez une adresse e-mail valide et réessayez.|
|success_verify_code_msg | Adresse e-mail vérifiée. Vous pouvez maintenant continuer.|
|failure_verify_code_msg | Nous rencontrons des problèmes de la vérification de votre adresse e-mail. Recommencez.|
|but_send_code | Envoyer le code de vérification|
|but_verify_code | Vérifier le code|
|but_send_new_code | Envoyer le nouveau code|
|but_change_claims | Modifier l'adresse e-mail|

### <a name="verification-display-control-example"></a>Exemple de contrôle d’affichage de vérification

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
   <!-- Display control UI elements-->
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="intro_msg">Verification is necessary. Please click Send button.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_send_code_msg">Verification code has been sent to your inbox. Please copy it to the input box below.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_send_code_msg">We are having trouble verifying your email address. Please enter a valid email address and try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="success_verify_code_msg">E-mail address verified. You can now continue.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="failure_verify_code_msg">We are having trouble verifying your email address. Please try again.</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_code">Send verification code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_verify_code">Verify code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_new_code">Send new code</LocalizedString>
    <LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_change_claims">Change e-mail</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="verification-display-control-user-interface-elements-deprecated"></a>Éléments de l’interface utilisateur du contrôle de l’affichage de vérification (déconseillé)

Voici les ID d’un [contrôle d’affichage de vérification](display-control-verification.md) avec la [version de mise en page](page-layout.md) 2.0.0.

| id | Valeur par défaut |
| -- | ------------- |
|verification_control_but_change_claims |Modifier |
|verification_control_fail_send_code |Impossible d’envoyer le code, réessayez plus tard. |
|verification_control_fail_verify_code |Impossible de vérifier le code, réessayez plus tard. |
|verification_control_but_send_code |Envoyer le code |
|verification_control_but_send_new_code |Envoyer le nouveau code |
|verification_control_but_verify_code |Vérifier le code |
|verification_control_code_sent| Le code de vérification a été envoyé. Veuillez le copier dans la zone d’entrée ci-dessous. |

### <a name="verification-display-control-example-deprecated"></a>Exemple de contrôle d’affichage de vérification (déconseillé)

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_change_claims">Change</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_send_code">Failed to send the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_fail_verify_code">Failed to verify the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_code">Send Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_send_new_code">Send New Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_but_verify_code">Verify Code</LocalizedString>
    <LocalizedString ElementType="UxElement" StringId="verification_control_code_sent">Verification code has been sent. Please copy it to the input box below.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="restful-service-error-messages"></a>Messages d’erreur du service Restful

Voici les ID des messages d’erreur de [profil technique du service Restful](restful-technical-profile.md) :

| id | Valeur par défaut |
| -- | ------------- |
|DefaultUserMessageIfRequestFailed | Échec de l’établissement de la connexion au point de terminaison du service Restful. URL du service Restful : {0} |
|UserMessageIfCircuitOpen | {0}URL du service Restful : {1} |
|UserMessageIfDnsResolutionFailed | Échec de la résolution du nom d’hôte du point de terminaison du service Restful. URL du service Restful : {0} |
|UserMessageIfRequestTimeout | Échec de l’établissement de la connexion au point de terminaison du service Restful dans la limite de délai d'expiration de {0} secondes. URL du service Restful : {1} |


### <a name="restful-service-example"></a>Exemple de service Restful

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="DefaultUserMessageIfRequestFailed">Failed to establish connection to restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCircuitOpen">Unable to connect to the restful service end point.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfDnsResolutionFailed">Failed to resolve the hostname of the restful service endpoint.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfRequestTimeout">Failed to establish connection to restful service end point within timeout limit.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-mfa-error-messages"></a>Messages d’erreur Azure AD MFA

Voici les ID d’un message d’erreur de [profil technique Azure AD MFA](multi-factor-auth-technical-profile.md) :

| id | Valeur par défaut |
| -- | ------------- |
|UserMessageIfCouldntSendSms | Impossible d’envoyer un SMS au téléphone. Essayez un autre numéro de téléphone. |
|UserMessageIfInvalidFormat | Le format de votre numéro de téléphone n’est pas valide. Corrigez-le, puis réessayez.|
|UserMessageIfMaxAllowedCodeRetryReached | Code erroné entré trop de fois. Réessayez plus tard.|
|UserMessageIfServerError | Impossible d'utiliser le service MFA. Réessayez plus tard.|
|UserMessageIfThrottled | Votre requête a été limitée, Réessayez plus tard.|
|UserMessageIfWrongCodeEntered|Code entré erroné. Réessayez.|

### <a name="azure-ad-mfa-example"></a>Exemple Azure AD MFA

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfCouldntSendSms">Cannot Send SMS to the phone, please try another phone number.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidFormat">Your phone number is not in a valid format, please correct it and try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxAllowedCodeRetryReached">Wrong code entered too many times, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfServerError">Cannot use MFA service, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">Your request has been throttled, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfWrongCodeEntered">Wrong code entered, please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="azure-ad-sspr"></a>Azure AD SSPR

Voici les ID des messages d’erreur de [profil technique Azure AD SSPR](aad-sspr-technical-profile.md) :

| id | Valeur par défaut |
| -- | ------------- |
|UserMessageIfChallengeExpired | Le code a expiré.|
|UserMessageIfInternalError | Le service de courrier a rencontré une erreur interne. Réessayez plus tard.|
|UserMessageIfThrottled | Vous avez envoyé trop de requêtes. Réessayez plus tard.|
|UserMessageIfVerificationFailedNoRetry | Vous avez dépassé le nombre maximal de tentatives de vérification.|
|UserMessageIfVerificationFailedRetryAllowed | La vérification a échoué. Réessayez.|


### <a name="azure-ad-sspr-example"></a>Exemple SSPR Azure AD

```XML
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInternalError">We are having trouble verifying your email address. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfThrottled">There have been too many requests to verify this email address. Please wait a while, then try again.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfChallengeExpired">That code is expired. Please request a new code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedNoRetry">You've made too many incorrect attempts. Please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="one-time-password-error-messages"></a>Messages d’erreur du mot de passe à usage unique

Voici les ID d’un message d’erreur de [profil technique à mot de passe à usage unique](one-time-password-technical-profile.md)

| id | Valeur par défaut |
| -- | ------------- |
|UserMessageIfMaxRetryAttempted |Le mot de passe unique fourni pour la vérification a dépassé le nombre maximum de tentatives |
|UserMessageIfSessionDoesNotExist |Une session de vérification du mot de passe à usage unique a expiré |
|UserMessageIfSessionConflict |Une session de vérification du mot de passe à usage unique a un conflit |
|UserMessageIfInvalidCode |Le mot de passe à usage unique fourni pour la vérification est incorrect |
|UserMessageIfVerificationFailedRetryAllowed |Ce code est incorrect. Réessayez. | 

### <a name="one-time-password-example"></a>Exemple de mot de passe à usage unique

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidCode">You have entered the wrong code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</LocalizedString>
   <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">That code is incorrect. Please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="claims-transformations-error-messages"></a>Messages d'erreur relatifs aux transformations de revendications

Voici les ID des messages d’erreur relatifs aux transformations de revendications :

| id | Transformation de revendications | Valeur par défaut |
| -- | ------------- |------------- |
|UserMessageIfClaimsTransformationBooleanValueIsNotEqual |[AssertBooleanClaimIsEqualToValue](boolean-transformations.md#assertbooleanclaimisequaltovalue) | Échec de comparaison des valeurs de la revendication booléenne pour le type de revendication « inputClaim ».| 
|DateTimeGreaterThan |[AssertDateTimeIsGreaterThan](date-transformations.md#assertdatetimeisgreaterthan) | Échec de comparaison des valeurs de la revendication : L’opérande de gauche fourni est supérieur à l’opérande de droite.|
|UserMessageIfClaimsTransformationStringsAreNotEqual |[AssertStringClaimsAreEqual](string-transformations.md#assertstringclaimsareequal) | Échec de comparaison des valeurs de la revendication à l’aide de StringComparison « OrdinalIgnoreCase ».|

### <a name="claims-transformations-example"></a>Exemple de transformations de revendications

```xml
<LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedStrings>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationBooleanValueIsNotEqual">Your email address hasn't been verified.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="DateTimeGreaterThan">Expiration date must be greater that the current date.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsTransformationStringsAreNotEqual">The email entry fields do not match. Please enter the same email address in both fields and try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

## <a name="next-steps"></a>Étapes suivantes

Pour voir des exemples de localisation, consultez les articles suivants :

- [Personnalisation de la langue avec la stratégie personnalisée dans Azure Active Directory B2C](custom-policy-localization.md)
- [Personnalisation de la langue avec les flux d’utilisateurs dans Azure Active Directory B2C](user-flow-language-customization.md)
