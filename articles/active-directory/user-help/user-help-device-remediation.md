---
title: Corrections pour l’erreur « Vous ne pouvez pas y accéder à partir d’ici » - Azure AD
description: Trouvez les corrections possibles à la cause de l’affichage du message d’erreur « Vous ne pouvez pas y accéder à partir d’ici ».
services: active-directory
author: curtand
manager: daveba
ms.assetid: 8ad0156c-0812-4855-8563-6fbff6194174
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 10/10/2018
ms.author: curtand
ms.reviewer: jairoc
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: df3941c895ce67862eb53b8e96bc7a6d53c1ed02
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88799415"
---
# <a name="possible-fixes-for-the-you-cant-get-there-from-here-error-message"></a>Corrections possibles par rapport au message d’erreur « Vous ne pouvez pas y accéder à partir d’ici ».

Lorsque vous accédez aux applications ou services web internes de votre organisation, le message d’erreur **Vous ne pouvez pas y accéder à partir d’ici** peut s’afficher. Ce message signifie que votre organisation a mis en place une stratégie qui empêche votre appareil d’accéder à ses ressources. Avant de contacter votre service d’assistance pour résoudre le problème, voici un certain nombre de choses que vous pouvez essayer préalablement.

## <a name="make-sure-youre-using-a-supported-browser"></a>Assurez-vous d’utiliser un navigateur pris en charge
Si vous obtenez le message **Vous ne pouvez pas y accéder à partir d’ici** indiquant que vous essayez d’accéder aux sites de votre organisation à partir d’un navigateur non pris en charge, vérifiez quel navigateur vous utilisez.

![Message d’erreur lié à la prise en charge de votre navigateur](media/user-help-device-remediation/browser-version.png)

Pour résoudre ce problème, vous devez installer et exécuter un navigateur compatible avec votre système d’exploitation. Si vous utilisez Windows 10, les navigateurs pris en charge incluent Microsoft Edge, Internet Explorer et Google Chrome. Si vous utilisez un autre système d’exploitation, vous pouvez vérifier la liste complète des [navigateurs pris en charge](../conditional-access/concept-conditional-access-conditions.md#supported-browsers).

## <a name="make-sure-youre-using-a-supported-operating-system"></a>Vérifiez que vous utilisez un système d’exploitation pris en charge
Vérifiez que vous exécutez un système d’exploitation pris en charge, notamment :

- **Client Windows.** Windows 7 ou version ultérieure.

- **Windows Server.** Windows Server 2008 R2 ou version ultérieure.

- **macOS.** macOS X ou version ultérieure

- **Android et iOS.** Version la plus récente des systèmes d’exploitation mobiles Android et iOS.

Pour résoudre ce problème, vous devez installer et exécuter un système d’exploitation pris en charge.

## <a name="make-sure-your-device-is-joined-to-your-network"></a>Assurez-vous que votre appareil est joint à votre réseau
Si vous obtenez le message **Vous ne pouvez pas y accéder à partir d’ici** indiquant que votre appareil n’est pas conforme à la stratégie d’accès de votre organisation, assurez-vous d’avoir joint votre appareil au réseau de votre organisation.

![Message d’erreur lié à votre absence sur le réseau](media/user-help-device-remediation/network-version.png)

### <a name="to-check-whether-your-device-is-joined-to-your-network"></a>Pour vérifier si votre appareil est joint à votre réseau
1. Connectez-vous à Windows à l’aide de votre compte professionnel ou scolaire. Par exemple : alain@contoso.com.

2. Connectez-vous au réseau de votre organisation via un réseau privé virtuel (VPN) ou DirectAccess.

3. Une fois connecté, appuyez sur la **touche Windows + la touche L** pour verrouiller votre appareil.

4. Déverrouillez votre appareil en utilisant votre compte professionnel ou scolaire, puis essayez à nouveau d’accéder à l’application ou au service posant problème.

    Si le message d’erreur **Vous ne pouvez pas y accéder à partir d’ici** s’affiche, sélectionnez le lien **Plus d’informations**, puis contactez votre support technique en fournissant ces informations.

### <a name="to-join-your-device-to-your-network"></a>Pour joindre votre appareil professionnel à votre réseau
Si votre appareil n’est pas joint au réseau de votre organisation, vous pouvez faire l’une des deux opérations suivantes :

- **Joindre votre appareil de travail** Joignez votre appareil Windows 10 professionnel au réseau de votre organisation afin de pouvoir accéder à des ressources potentiellement restreintes. Pour obtenir plus d’informations et des instructions pas à pas, reportez-vous à [Joindre votre appareil professionnel au réseau de votre organisation](user-help-join-device-on-network.md).

- **Inscrire votre appareil personnel pour le travail** Inscrivez votre appareil personnel, généralement un téléphone ou une tablette, sur le réseau de votre organisation. Une fois votre appareil inscrit, il peut accéder aux ressources restreintes de votre organisation. Pour obtenir plus d’informations et des instructions pas à pas, reportez-vous à [Inscrire votre appareil personnel au réseau de votre organisation](user-help-register-device-on-network.md).

## <a name="next-steps"></a>Étapes suivantes
- [Qu’est-ce que le portail MyApps ?](./my-apps-portal-end-user-access.md)

- [Connectez-vous avec votre téléphone, et non votre mot de passe](user-help-auth-app-sign-in.md)