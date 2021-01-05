---
title: Vérifier les revendications avec des contrôles d'affichage
titleSuffix: Azure AD B2C
description: Découvrez comment utiliser les contrôles d'affichage Azure AD B2C pour vérifier les revendications dans les parcours utilisateur prévus par vos stratégies personnalisées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/10/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: bd90be6d93dc5ca399ac87daba0ca44fa7e88ff8
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94532492"
---
# <a name="verification-display-control"></a>Contrôle d’affichage de vérification

Utilisez un [contrôle d’affichage](display-controls.md) de vérification pour vérifier une revendication, par exemple une adresse e-mail ou un numéro de téléphone, à l’aide d’un code de vérification envoyé à l'utilisateur.

## <a name="verificationcontrol-actions"></a>Actions VerificationControl

Le contrôle d’affichage de vérification comporte deux étapes (actions) :

1. Demander à l'utilisateur une destination, par exemple une adresse e-mail ou un numéro de téléphone, où le code de vérification doit être envoyé. Lorsque l’utilisateur sélectionne le bouton **Envoyer le code**, l’**action SendCode** du contrôle d'affichage de vérification est exécutée. L’**action SendCode** génère un code, construit le contenu à envoyer, puis l’envoie à l’utilisateur. La valeur de l’adresse peut être pré-remplie et servir de second facteur d’authentification.

    ![Exemple de page pour une action d’envoi de code](media/display-control-verification/display-control-verification-email-action-01.png)

1. Après l’envoi du code, l’utilisateur lit le message, saisit le code de vérification dans le contrôle fourni par le contrôle d’affichage, puis sélectionne **Vérifier le code**. En sélectionnant **Vérifier le code**, l’action **VerifyCode** est exécutée pour vérifier le code associé à l’adresse. Si l’utilisateur sélectionne **Envoyer un nouveau code**, la première action est exécutée à nouveau.

    ![Exemple de page pour l’action de vérification du code](media/display-control-verification/display-control-verification-email-action-02.png)

## <a name="verificationcontrol-required-elements"></a>Éléments VerificationControl requis

**VerificationControl** doit contenir les éléments suivants :

- Le type du `DisplayControl` est `VerificationControl`.
- `DisplayClaims`
  - **Envoyer à** : une ou plusieurs revendications précisant où envoyer le code de vérification. Par exemple, *e-mail* ou *indicatif du pays* et *numéro de téléphone*.
  - **Code de vérification** : revendication du code de vérification que l’utilisateur fournit après l’envoi du code. Cette revendication doit être définie si nécessaire, et `ControlClaimType` doit afficher la valeur `VerificationCode`.
- La revendication de sortie (facultative) doit être renvoyée à la page autodéclarée une fois que l’utilisateur a terminé le processus de vérification. Par exemple, *e-mail* ou *indicatif du pays* et *numéro de téléphone*. Le profil technique autodéclaré utilise les revendications pour rendre persistantes les données ou pour faire remonter les revendications de sortie à l'étape d’orchestration suivante.
- Deux éléments `Action` avec les noms suivants :
  - **SendCode** : envoie un code à l’utilisateur. Cette action contient généralement deux profils techniques de validation : un pour générer un code, et un autre pour l’envoyer.
  - **VerifyCode** : vérifie le code. Cette action contient généralement un seul profil technique de validation.

Dans l’exemple ci-dessous, une zone de texte **e-mail** est affichée sur la page. Lorsque l'utilisateur entre son adresse e-mail et sélectionne **SendCode**, l’action **SendCode** est déclenchée dans le back end Azure AD B2C.

L’utilisateur entre ensuite la valeur **verificationCode** puis sélectionne **VerifyCode** pour déclencher l'action **VerifyCode** dans le back end. Si toutes les validations sont confirmées, le processus **VerificationControl** est considéré comme terminé et l’utilisateur peut passer à l’étape suivante.

```xml
<DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
  <DisplayClaims>
    <DisplayClaim ClaimTypeReferenceId="email"  Required="true" />
    <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
  </DisplayClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" />
  </OutputClaims>
  <Actions>
    <Action Id="SendCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="GenerateOtp" />
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="SendGrid" />
      </ValidationClaimsExchange>
    </Action>
    <Action Id="VerifyCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="VerifyOtp" />
      </ValidationClaimsExchange>
    </Action>
  </Actions>
</DisplayControl>
```
