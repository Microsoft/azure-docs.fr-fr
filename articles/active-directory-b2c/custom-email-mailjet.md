---
title: Vérification des e-mails personnalisée avec Mailjet
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer avec Mailjet pour personnaliser les e-mails de vérification envoyés à vos clients lorsqu'ils s'inscrivent pour utiliser vos applications Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: b74de2bdf1f6239f1006c820579a336946939421
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94949579"
---
# <a name="custom-email-verification-with-mailjet"></a>Vérification des e-mails personnalisée avec Mailjet

Utilisez un e-mail personnalisé dans Azure Active Directory B2C (Azure AD B2C) afin d’envoyer un e-mail personnalisé aux utilisateurs qui s'inscrivent pour utiliser vos applications. En utilisant [DisplayControls](display-controls.md) (actuellement en préversion) et le Mailjet fournisseur d’e-mails tiers, vous pouvez utiliser votre propre modèle d’e-mail ainsi qu’une adresse *De :* et un objet, en bénéficiant également du support de la localisation et des paramètres personnalisés du mot de passe à usage unique (OTP).

La vérification d’e-mails personnalisée nécessite l’utilisation d'un fournisseur d’e-mails tiers comme [Mailjet](https://Mailjet.com), [SendGrid](./custom-email-sendgrid.md) ou [SparkPost](https://sparkpost.com), une API REST personnalisée ou tout fournisseur d'e-mails basé sur HTTP (y compris le vôtre). Cet article décrit la configuration d'une solution qui utilise Mailjet.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="create-a-mailjet-account"></a>Créer un compte Mailjet

Si vous ne disposez pas encore d’un compte, commencez par configurer un compte Mailjet (les clients Azure peuvent débloquer 6 000 e-mails avec une limite de 200 e-mails/jour). 

1. Suivez les instructions de configuration dans [Créer un compte de Mailjet](https://www.mailjet.com/guides/azure-mailjet-developer-resource-user-guide/enabling-mailjet/).
1. Pour pouvoir envoyer des e-mails, [inscrire et valider](https://www.mailjet.com/guides/azure-mailjet-developer-resource-user-guide/enabling-mailjet/#how-to-configure-mailjet-for-use) l’adresse e-mail ou le domaine de l’expéditeur.
2. Accédez à la [page Gestion des clés de l’API](https://app.mailjet.com/account/api_keys). Enregistrez la **Clé API** et la **Clé secrète** pour l'utiliser à une étape ultérieure. Les deux clés sont générées automatiquement lorsque votre compte est créé.  

## <a name="create-azure-ad-b2c-policy-key"></a>Créer une clé de stratégie Azure AD B2C

Stockez ensuite la clé API Mailjet dans une clé de stratégie Azure AD B2C pour que vos stratégies puissent y faire référence.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire + abonnement** dans le menu du haut, puis choisissez votre annuaire Azure AD B2C.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
1. Dans la page **de présentation**, sélectionnez **Identity Experience Framework**.
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Dans **Options**, choisissez **Manuel**.
1. Entrez un **nom** pour la clé de stratégie. Par exemple : `MailjetApiKey`. Le préfixe `B2C_1A_` est ajouté automatiquement au nom de votre clé.
1. Dans **Secret**, entrez votre **Clé API** Mailjet que vous avez précédemment enregistrée.
1. Pour **Utilisation de la clé**, sélectionnez **Signature**.
1. Sélectionnez **Create** (Créer).
1. Sélectionnez **Clés de stratégie**, puis **Ajouter**.
1. Dans **Options**, choisissez **Manuel**.
1. Entrez un **nom** pour la clé de stratégie. Par exemple : `MailjetSecretKey`. Le préfixe `B2C_1A_` est ajouté automatiquement au nom de votre clé.
1. Dans **Secret**, entrez la **Clé API** Mailjet que vous avez précédemment enregistrée.
1. Pour **Utilisation de la clé**, sélectionnez **Signature**.
1. Sélectionnez **Create** (Créer).

## <a name="create-a-mailjet-template"></a>Créer un modèle Mailjet

Avec un compte Mailjet créé et une Clé API Mailjet stockée dans une clé de stratégie Azure AD B2C, créez un [modèle transactionnel dynamique](https://sendgrid.com/docs/ui/sending-email/how-to-send-an-email-with-dynamic-transactional-templates/) Mailjet.

1. Sur le site Mailjet, ouvrez la page des [modèles transactionnels](https://app.mailjet.com/templates/transactional) et sélectionnez **Créer un nouveau modèle**.
1. Sélectionnez **En le codant dans le HTML**, puis sélectionnez **Code à partir de zéro**.
1. Entrez un nom de modèle unique comme `Verification email`, puis sélectionnez **Créer**.
1. Dans l'éditeur HTML, collez le modèle HTML suivant ou utilisez le vôtre. Les paramètres `{{var:otp:""}}` et `{{var:email:""}}` seront remplacés dynamiquement par la valeur du mot de passe à usage unique et l'adresse e-mail de l'utilisateur.

    ```HTML
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

    <html xmlns="http://www.w3.org/1999/xhtml" dir="ltr" lang="en"><head id="Head1">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"><title>Contoso demo account email verification code</title><meta name="ROBOTS" content="NOINDEX, NOFOLLOW">
       <!-- Template B O365 -->
       <style>
           table td {border-collapse:collapse;margin:0;padding:0;}
       </style>
    </head>
    <body dir="ltr" lang="en">
       <table width="100%" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
            <tr>
               <td valign="top" width="50%"></td>
               <td valign="top">
                  <!-- Email Header -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en" style="border-left:1px solid #e3e3e3;border-right: 1px solid #e3e3e3;">
                   <tr style="background-color: #0072C6;">
                       <td width="1" style="background:#0072C6; border-top:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-top:1px solid #e3e3e3;border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="310" valign="middle" style="border-top:1px solid #e3e3e3; border-bottom:1px solid #e3e3e3;padding:12px 0;">
                           <h1 style="line-height:20pt;font-family:Segoe UI Light; font-size:18pt; color:#ffffff; font-weight:normal;">
                            <span id="HeaderPlaceholder_UserVerificationEmailHeader"><font color="#FFFFFF">Verify your email address</font></span>
                           </h1>
                       </td>
                       <td width="24" style="border-top: 1px solid #e3e3e3;border-bottom: 1px solid #e3e3e3;">&nbsp;</td>
                   </tr>
                  </table>
                  <!-- Email Content -->
                  <table width="640" cellpadding="0" cellspacing="0" border="0" dir="ltr" lang="en">
                   <tr>
                       <td width="1" style="background:#e3e3e3;"></td>
                       <td width="24">&nbsp;</td>
                       <td id="PageBody" width="640" valign="top" colspan="2" style="border-bottom:1px solid #e3e3e3;padding:10px 0 20px;border-bottom-style:hidden;">
                           <table cellpadding="0" cellspacing="0" border="0">
                               <tr>
                                   <td width="630" style="font-size:10pt; line-height:13pt; color:#000;">
                                       <table cellpadding="0" cellspacing="0" border="0" width="100%" style="" dir="ltr" lang="en">
                                           <tr>
                                               <td>

       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence1">Thanks for verifying your {{var:email:""}} account!</span>
       </div>
       <br>
       <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333; font-weight: bold">
           <span id="BodyPlaceholder_UserVerificationEmailBodySentence2">Your code is: {{var:otp:""}}</span>
       </div>
       <br>
       <br>

                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; color:#333;">
                                                   Sincerely,
                                                   </div>
                                                   <div style="font-family:'Segoe UI', Tahoma, sans-serif; font-size:14px; font-style:italic; color:#333;">
                                                       Contoso
                                                   </div>
                                               </td>
                                           </tr>
                                       </table>
                                   </td>
                               </tr>
                           </table>

                       </td>

                       <td width="1">&nbsp;</td>
                       <td width="1"></td>
                       <td width="1">&nbsp;</td>
                       <td width="1" valign="top"></td>
                       <td width="29">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3;"></td>
                   </tr>
                   <tr>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                       <td width="24" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td id="PageFooterContainer" width="585" valign="top" colspan="6" style="border-bottom:1px solid #e3e3e3;padding:0px;">


                       </td>

                       <td width="29" style="border-bottom:1px solid #e3e3e3;">&nbsp;</td>
                       <td width="1" style="background:#e3e3e3; border-bottom:1px solid #e3e3e3;"></td>
                   </tr>
                  </table>

               </td>
               <td valign="top" width="50%"></td>
           </tr>
       </table>
    <img src="https://mucp.api.account.microsoft.com/m/v2/v?d=AIAACWEPFYXYIUTJIJVV4ST7XLBHVI5MLLYBKJAVXHBDTBHUM5VBSVVPTTVRWDFIXJ5JQTHYOH5TUYIPO4ZAFRFK52UAMIS3UNIPPI7ZJNDZPRXD5VEJBN4H6RO3SPTBS6AJEEAJOUYL4APQX5RJUJOWGPKUABY&amp;i=AIAACL23GD2PFRFEY5YVM2XQLM5YYWMHFDZOCDXUI2B4LM7ETZQO473CVF22PT6WPGR5IIE6TCS6VGEKO5OZIONJWCDMRKWQQVNP5VBYAINF3S7STKYOVDJ4JF2XEW4QQVNHMAPQNHFV3KMR3V3BA4I36B6BO7L4VQUHQOI64EOWPLMG5RB3SIMEDEHPILXTF73ZYD3JT6MYOLAZJG7PJJCAXCZCQOEFVH5VCW2KBQOKRYISWQLRWAT7IINZ3EFGQI2CY2EMK3FQOXM7UI3R7CZ6D73IKDI" width="1" height="1"></body>
    </html>
    ```

1. Développez **Modifier l’objet** en haut à gauche
    1. Dans **Objet**, entrez une valeur par défaut pour l’objet. Mailjet utilise cette valeur lorsque l’API ne contient pas de paramètre d’objet.
    1. Dans la case **Nom**, indiquez le nom de votre société.
    1. Dans la case **Adresse**, sélectionnez votre adresse e-mail
    1. Sélectionnez **Enregistrer**.
1. Dans le coin supérieur droit, sélectionnez **Enregistrer et publier**, puis **Oui, publier les modifications**
1. Enregistrez l’**ID du modèle** que vous avez créé pour l'utiliser à une étape ultérieure. Vous spécifiez cet ID lorsque vous [ajoutez la transformation de revendications](#add-the-claims-transformation).


## <a name="add-azure-ad-b2c-claim-types"></a>Ajouter des types de revendications Azure AD B2C

Dans votre stratégie, ajoutez les types de revendications suivants à l'élément `<ClaimsSchema>` dans `<BuildingBlocks>`.

Ces types de revendications sont nécessaires pour générer et vérifier l’adresse e-mail à l'aide d'un code de mot de passe à usage unique (OTP).

```XML
<ClaimType Id="Otp">
  <DisplayName>Secondary One-time password</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="emailRequestBody">
  <DisplayName>Mailjet request body</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="VerificationCode">
  <DisplayName>Secondary Verification Code</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Enter your email verification code</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

## <a name="add-the-claims-transformation"></a>Ajouter la transformation de revendications

Ensuite, vous avez besoin d'une transformation de revendications pour générer une revendication de chaîne JSON qui constituera le corps de la requête envoyée à Mailjet.

La structure de l'objet JSON est définie par les ID en notation par points des éléments InputParameters et des éléments TransformationClaimTypes de InputClaims. Les nombres dans la notation par points impliquent des tableaux. Les valeurs proviennent des valeurs InputClaims et des propriétés « Value » des éléments InputParameters. Pour plus d'informations sur les transformations de revendications JSON, voir [Transformations de revendications JSON](json-transformations.md).

Ajoutez la transformation de revendications suivante à l'élément `<ClaimsTransformations>` dans `<BuildingBlocks>`. Effectuez les mises à jour suivantes dans le code XML de la transformation de revendications :

* Mettez à jour la valeur InputParameter `Messages.0.TemplateID` avec l'ID du modèle transactionnel Mailjet que vous avez créé précédemment dans [Créer un modèle Mailjet](#create-a-mailjet-template).
* Mettez à jour la valeur de l'adresse `Messages.0.From.Email`. Utilisez une adresse e-mail valide pour éviter que l’e-mail de vérification ne soit considéré comme un courrier indésirable.
* Mettez à jour la valeur du paramètre de saisie de la ligne d’objet `Messages.0.Subject` avec une ligne d’objet appropriée pour votre organisation.

```XML
<ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="Messages.0.To.0.Email" />
    <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="Messages.0.Variables.otp" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="Messages.0.Variables.email" />
  </InputClaims>
  <InputParameters>
    <!-- Update the template_id value with the ID of your Mailjet template. -->
    <InputParameter Id="Messages.0.TemplateID" DataType="int" Value="1234567"/>
    <InputParameter Id="Messages.0.TemplateLanguage" DataType="boolean" Value="true"/>

    <!-- Update with an email appropriate for your organization. -->
    <InputParameter Id="Messages.0.From.Email" DataType="string" Value="my_email@mydomain.com"/>

    <!-- Update with a subject line appropriate for your organization. -->
    <InputParameter Id="Messages.0.Subject" DataType="string" Value="Contoso account email verification code"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

## <a name="add-datauri-content-definition"></a>Ajouter la définition du contenu DataUri

Sous les transformations de revendications dans `<BuildingBlocks>`, ajoutez l’élément [ContentDefinition](contentdefinitions.md) suivant pour faire référence à l’URI de données de la version 2.1.0 :

```XML
<ContentDefinitions>
 <ContentDefinition Id="api.localaccountsignup">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
  </ContentDefinition>
  <ContentDefinition Id="api.localaccountpasswordreset">
    <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
  </ContentDefinition>
</ContentDefinitions>
```

## <a name="create-a-displaycontrol"></a>Créer un élément DisplayControl

Un contrôle d’affichage de vérification est utilisé pour vérifier l’adresse e-mail à l’aide d’un code de vérification envoyé à l’utilisateur.

Cet exemple de contrôle d'affichage est configuré pour :

1. Recueillir le type de revendication d'adresse `email` auprès de l'utilisateur.
1. Attendre que l'utilisateur fournisse le type de réclamation `verificationCode` à l’aide du code qui lui a été envoyé.
1. Retourner `email` au profil technique autodéclaré qui contient une référence à ce contrôle d'affichage.
1. En utilisant l’action `SendCode`, générer un code OTP et envoyer un e-mail avec le code OTP à l'utilisateur.

   ![Action d’envoi du code de vérification par e-mail](media/custom-email-mailjet/display-control-verification-email-action-01.png)

Sous les définitions de contenu, toujours dans `<BuildingBlocks>`, ajoutez à votre stratégie l’élément [DisplayControl](display-controls.md) suivant de type [VerificationControl](display-control-verification.md).

```XML
<DisplayControls>
  <DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
    <DisplayClaims>
      <DisplayClaim ClaimTypeReferenceId="email" Required="true" />
      <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
    </DisplayClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="email" />
    </OutputClaims>
    <Actions>
      <Action Id="SendCode">
        <ValidationClaimsExchange>
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="GenerateOtp" />
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="SendOtp" />
        </ValidationClaimsExchange>
      </Action>
      <Action Id="VerifyCode">
        <ValidationClaimsExchange>
          <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="VerifyOtp" />
        </ValidationClaimsExchange>
      </Action>
    </Actions>
  </DisplayControl>
</DisplayControls>
```

## <a name="add-otp-technical-profiles"></a>Ajouter des profils techniques OTP

Le profil technique `GenerateOtp` génère un code pour l’adresse e-mail. Le profil technique `VerifyOtp` vérifie le code associé à l’adresse e-mail. Vous pouvez modifier la configuration du format et l’expiration du mot de passe à usage unique. Pour plus d'informations sur les profils techniques OTP, voir [Définir un profil technique de mot de passe à usage unique](one-time-password-technical-profile.md).

Ajoutez les profils techniques suivants à l'élément `<ClaimsProviders>`.

```XML
<ClaimsProvider>
  <DisplayName>One time password technical profiles</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="GenerateOtp">
      <DisplayName>Generate one time password</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="Operation">GenerateCode</Item>
        <Item Key="CodeExpirationInSeconds">1200</Item>
        <Item Key="CodeLength">6</Item>
        <Item Key="CharacterSet">0-9</Item>
        <Item Key="ReuseSameCode">true</Item>
        <Item Key="MaxNumAttempts">5</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="otp" PartnerClaimType="otpGenerated" />
      </OutputClaims>
    </TechnicalProfile>

    <TechnicalProfile Id="VerifyOtp">
      <DisplayName>Verify one time password</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.OneTimePasswordProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="Operation">VerifyCode</Item>
      </Metadata>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="identifier" />
        <InputClaim ClaimTypeReferenceId="verificationCode" PartnerClaimType="otpToVerify" />
      </InputClaims>
    </TechnicalProfile>
   </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="add-a-rest-api-technical-profile"></a>Ajouter un profil technique API REST

Ce profil technique API REST génère le contenu de l’e-mail (en utilisant le format Mailjet). Pour plus d'informations sur les profils techniques RESTful, voir [Définir un profil technique RESTful](restful-technical-profile.md).

Comme pour les profils techniques OTP, ajoutez les profils techniques suivants à l'élément `<ClaimsProviders>`.

```XML
<ClaimsProvider>
  <DisplayName>RestfulProvider</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="sendOtp">
      <DisplayName>Use email API to send the code the the user</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://api.mailjet.com/v3.1/send</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <Item Key="ClaimUsedForRequestPayload">emailRequestBody</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_MailjetApiKey" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_MailjetSecretKey" />
      </CryptographicKeys>
      <InputClaimsTransformations>
        <InputClaimsTransformation ReferenceId="GenerateEmailRequestBody" />
      </InputClaimsTransformations>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="emailRequestBody" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="make-a-reference-to-the-displaycontrol"></a>Faire référence à DisplayControl

Dans l'étape finale, ajoutez une référence à l’élément DisplayControl que vous avez créé. Remplacez vos profils techniques autodéclarés `LocalAccountSignUpWithLogonEmail` et `LocalAccountDiscoveryUsingEmailAddress` existants par les suivants. Si vous avez utilisé une version antérieure de la stratégie Azure AD B2C, ces profils techniques utilisent `DisplayClaims` avec une référence à l’élément DisplayControl.

Pour plus d'informations, voir [Profil technique autodéclaré](restful-technical-profile.md) et [DisplayControl](display-controls.md).

```XML
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
        <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
        <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
      </DisplayClaims>
    </TechnicalProfile>
    <TechnicalProfile Id="LocalAccountDiscoveryUsingEmailAddress">
      <Metadata>
        <!--OTP validation error messages-->
        <Item Key="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</Item>
        <Item Key="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</Item>
        <Item Key="UserMessageIfInvalidCode">You have entered the wrong code.</Item>
        <Item Key="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</Item>
      </Metadata>
      <DisplayClaims>
        <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
      </DisplayClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="optional-localize-your-email"></a>(Facultatif) Localiser votre e-mail

Pour localiser l’e-mail, vous devez envoyer des chaînes localisées à Mailjet ou à votre fournisseur d’e-mails. Par exemple, vous pouvez localiser l’objet, le corps, le message du code ou la signature de l’e-mail. Pour ce faire, vous pouvez utiliser la transformation de revendications [GetLocalizedStringsTransformation](string-transformations.md) pour copier les chaînes localisées dans les types de revendications. La transformation de revendications `GenerateEmailRequestBody`, qui génère la charge utile JSON, utilise des revendications d’entrée qui contiennent les chaînes localisées.

1. Dans votre stratégie, définissez les revendications de chaîne suivantes : objet, message, codeIntro et signature.
1. Définissez une transformation de revendications [GetLocalizedStringsTransformation](string-transformations.md) pour substituer les valeurs de chaîne localisées dans les revendications de l’étape 1.
1. Modifiez la transformation de revendications `GenerateEmailRequestBody` pour utiliser des revendications d’entrée avec l’extrait de code XML suivant.
1. Mettez à jour votre modèle Mailjet pour utiliser des paramètres dynamiques à la place de toutes les chaînes qui seront localisées par Azure AD B2C.

    ```XML
    <ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
        <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
        <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
        <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="GenerateEmailRequestBody" TransformationMethod="GenerateJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="Messages.0.To.0.Email" />
        <InputClaim ClaimTypeReferenceId="subject" TransformationClaimType="Messages.0.Subject" />
        <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="Messages.0.Variables.otp" />
        <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="Messages.0.Variables.email" />
        <InputClaim ClaimTypeReferenceId="message" TransformationClaimType="Messages.0.Variables.otpmessage" />
        <InputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="Messages.0.Variables.otpcodeIntro" />
        <InputClaim ClaimTypeReferenceId="signature" TransformationClaimType="Messages.0.Variables.otpsignature" />
      </InputClaims>
      <InputParameters>
        <!-- Update the template_id value with the ID of your Mailjet template. -->
        <InputParameter Id="Messages.0.TemplateID" DataType="int" Value="1234567"/>
        <InputParameter Id="Messages.0.TemplateLanguage" DataType="boolean" Value="true"/>

        <!-- Update with an email appropriate for your organization. -->
        <InputParameter Id="Messages.0.From.Email" DataType="string" Value="my_email@mydomain.com"/>
      </InputParameters>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="emailRequestBody" TransformationClaimType="outputClaim"/>
      </OutputClaims>
    </ClaimsTransformation>
    ```

1. Ajoutez l’élément [Localisation](localization.md) suivant.

    ```xml
    <Localization Enabled="true">
      <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
        <SupportedLanguage>en</SupportedLanguage>
        <SupportedLanguage>es</SupportedLanguage>
      </SupportedLanguages>
      <LocalizedResources Id="api.custom-email.en">
        <LocalizedStrings>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for validating the account</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
        </LocalizedStrings>
        </LocalizedStrings>
      </LocalizedResources>
      <LocalizedResources Id="api.custom-email.es">
        <LocalizedStrings>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
          <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sinceramente</LocalizedString>
        </LocalizedStrings>
      </LocalizedResources>
    </Localization>
    ```

1. Ajoutez des références aux éléments LocalizedResources en mettant à jour l’élément [ContentDefinitions](contentdefinitions.md).

    ```xml
    <ContentDefinitions>
      <ContentDefinition Id="api.localaccountsignup">
        <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
        <LocalizedResourcesReferences MergeBehavior="Prepend">
          <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
          <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
        </LocalizedResourcesReferences>
      </ContentDefinition>
      <ContentDefinition Id="api.localaccountpasswordreset">
        <DataUri>urn:com:microsoft:aad:b2c:elements:contract:selfasserted:2.1.0</DataUri>
        <LocalizedResourcesReferences MergeBehavior="Prepend">
          <LocalizedResourcesReference Language="en" LocalizedResourcesReferenceId="api.custom-email.en" />
          <LocalizedResourcesReference Language="es" LocalizedResourcesReferenceId="api.custom-email.es" />
        </LocalizedResourcesReferences>
      </ContentDefinition>
    </ContentDefinitions>
    ```

1. Enfin, ajoutez la transformation de revendications d’entrée suivante aux profils techniques `LocalAccountSignUpWithLogonEmail` et `LocalAccountDiscoveryUsingEmailAddress`.

    ```xml
    <InputClaimsTransformations>
      <InputClaimsTransformation ReferenceId="GetLocalizedStringsForEmail" />
    </InputClaimsTransformations>
    ```
    
## <a name="optional-localize-the-ui"></a>[Facultatif] Localiser l’interface utilisateur

L’élément Localization vous permet de prendre en charge plusieurs paramètres régionaux ou langues dans la stratégie pour les parcours utilisateur. La prise en charge de la localisation dans les stratégies vous permet de fournir des chaînes spécifiques à la langue à la fois pour les [Éléments de l’interface utilisateur du contrôle de l’affichage de vérification](localization-string-ids.md#verification-display-control-user-interface-elements) et les [Messages d’erreur du mot de passe à usage unique](localization-string-ids.md#one-time-password-error-messages). Ajoutez la LocalizedString suivante à vos LocalizedResources. 

```XML
<LocalizedResources Id="api.custom-email.en">
  <LocalizedStrings>
    ...
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
    <!-- Claims-->
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="DisplayName">Verification Code</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="UserHelpText">Verification code received in the email.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="emailVerificationCode" StringId="AdminHelpText">Verification code received in the email.</LocalizedString>
    <LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Eamil</LocalizedString>
    <!-- Email validation error messages-->
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionDoesNotExist">You have exceeded the maximum time allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfMaxRetryAttempted">You have exceeded the number of retries allowed.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfInvalidCode">You have entered the wrong code.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfSessionConflict">Cannot verify the code, please try again later.</LocalizedString>
    <LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfVerificationFailedRetryAllowed">The verification has failed, please try again.</LocalizedString>
  </LocalizedStrings>
</LocalizedResources>
```

Après avoir ajouté les chaînes localisées, supprimez les métadonnées des messages d’erreur de validation de mot de passe à usage unique des profils techniques LocalAccountSignUpWithLogonEmail et LocalAccountDiscoveryUsingEmailAddress.

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez un exemple de stratégie de vérification d’e-mail personnalisée sur GitHub :

- [Vérification d’e-mail personnalisée - DisplayControls](https://github.com/azure-ad-b2c/samples/tree/master/policies/custom-email-verifcation-displaycontrol)
- Pour plus d'informations sur l'utilisation d'une API REST personnalisée ou d’un fournisseur de messagerie SMTP basé sur HTTP, voir [Définir un profil technique RESTful dans une stratégie personnalisée Azure AD B2C](restful-technical-profile.md).
