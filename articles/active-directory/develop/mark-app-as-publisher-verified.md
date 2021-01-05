---
title: Marquer une application comme vérifiée par l'éditeur - Plateforme d'identités Microsoft | Azure
description: Décrit comment marquer une application avec la mention « éditeur vérifié ». Lorsqu'une application est accompagnée de la mention « éditeur vérifié », cela signifie que l'éditeur a validé son identité à l'aide d'un compte Microsoft Partner Network auquel le processus de vérification a été appliqué jusqu'à son terme et qu'il a associé ce compte MPN à l'inscription de l'application.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 05/08/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jesakowi
ms.openlocfilehash: 2b658cdc58777274bb14f9e8069cef2facdb0479
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92363451"
---
# <a name="mark-your-app-as-publisher-verified"></a>Marquer votre application avec la mention « éditeur vérifié »

Quand une inscription d’application a un éditeur vérifié, cela signifie que l'éditeur de l’application a [vérifié](/partner-center/verification-responses) son identité à l'aide de son compte Microsoft Partner Network (MPN) et a associé ce compte MPN à l’inscription de l’application. Cet article explique comment accomplir le processus de [vérification de l’éditeur](publisher-verification-overview.md).

## <a name="quickstart"></a>Démarrage rapide
Si vous êtes déjà inscrit à Microsoft Partner Network (MPN) et que vous remplissez les [conditions préalables](publisher-verification-overview.md#requirements), vous pouvez commencer immédiatement : 

1. Se connecter au [portail d’inscription des applications](https://aka.ms/PublisherVerificationPreview) à l’aide de l’[authentification multifacteur](../fundamentals/concept-fundamentals-mfa-get-started.md)

1. Choisissez une application, puis cliquez sur **Personnalisation** . 

1. Cliquez sur **Ajouter l’ID MPN pour valider l'éditeur** et passez en revue les conditions requises.

1. Entrez votre ID MPN, puis cliquez sur **Vérifier et enregistrer** .

Pour plus d’informations sur les avantages spécifiques, les conditions requises et les questions fréquemment posées, consultez la [vue d'ensemble](publisher-verification-overview.md).


## <a name="mark-your-app-as-publisher-verified"></a>Marquer votre application avec la mention « éditeur vérifié »
Assurez-vous d'avoir respecté les [conditions préalables](publisher-verification-overview.md#requirements), puis procédez comme suit pour marquer vos applications avec la mention « éditeur vérifié ».  

1. Vérifiez que vous êtes connecté avec une [authentification multifacteur](../fundamentals/concept-fundamentals-mfa-get-started.md) à un compte d’organisation (Azure AD) autorisé à apporter des modifications aux applications que vous souhaitez marquer avec la mention « éditeur vérifié » et sur le compte MPN dans l’Espace partenaires.

    - Dans Azure AD, cet utilisateur doit être membre d’un des [rôles](../roles/permissions-reference.md) suivants : Administrateur d'application, Administrateur d'application cloud, Administrateur général. 

    - Dans l'Espace partenaires, cet utilisateur doit disposer des [rôles](/partner-center/permissions-overview) suivants : Administrateur MPN, Administrateur de comptes ou Administrateur général (rôle partagé maîtrisé dans Azure AD). 

1. Accédez au portail d’inscription des applications :  

1. Cliquez sur une application que vous souhaitez marquer avec la mention « éditeur vérifié » et ouvrez le panneau Personnalisation. 

1. Vérifiez que le [domaine de l’éditeur](howto-configure-publisher-domain.md) de l’application est défini. 

1. Assurez-vous que le domaine de l’éditeur ou un [domaine personnalisé](../fundamentals/add-custom-domain.md) vérifié par DNS sur le locataire correspond au domaine de l’adresse e-mail utilisée durant le processus de vérification de votre compte MPN.

1. Cliquez sur **Ajouter l'ID MPN pour vérifier l'éditeur** en bas de la page. 

1. Entrez votre **ID MPN** . Cet ID MPN doit correspondre à ce qui suit : 

    - Un compte Microsoft Partner Network valide auquel le processus de vérification a été appliqué jusqu'à son terme.  

    - Le compte global partenaire (PGA) de votre organisation. 

1. Cliquez sur **Vérifier et enregistrer** . 

1. Attendez le traitement de la requête, ce qui peut prendre quelques minutes. 

1. Si la vérification aboutit, la fenêtre de vérification de l'éditeur se ferme et vous revenez au panneau Personnalisation. Vous notez alors la présence d'un badge bleu vérifié en regard de votre **nom d'affichage d'éditeur** vérifié. 

1. Les utilisateurs invités à donner leur consentement à votre application voient votre badge une fois le processus terminé avec succès, même si sa réplication dans le système peut prendre un certain temps. 

1. Testez cette fonctionnalité en vous connectant à votre application et en vérifiant que le badge vérifié s’affiche dans l’écran de consentement. Si vous êtes connecté en tant qu’utilisateur ayant donné son consentement à l’application, vous pouvez utiliser le paramètre de requête *prompt=consent* pour forcer une invite de consentement. Ce paramètre doit être utilisé uniquement à des fins de test et n’est jamais codé en dur dans les demandes de votre application.

1. Répétez ce processus si nécessaire pour toutes les applications supplémentaires pour lesquelles vous souhaitez afficher le badge. Vous pouvez utiliser Microsoft Graph pour effectuer cette opération en bloc. Des cmdlets PowerShell seront prochainement disponibles. Pour plus d’informations, consultez [Création d'appels Microsoft API Graph](troubleshoot-publisher-verification.md#making-microsoft-graph-api-calls). 

Et voilà ! Faites-nous part de vos commentaires sur le processus, les résultats ou la fonctionnalité en général. 

## <a name="next-steps"></a>Étapes suivantes
En cas de problèmes, consultez les [informations relatives à la résolution des problèmes](troubleshoot-publisher-verification.md).
