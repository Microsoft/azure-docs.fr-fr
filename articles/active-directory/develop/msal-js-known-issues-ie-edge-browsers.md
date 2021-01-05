---
title: Problèmes liés sur Internet Explorer et Microsoft Edge (MSAL.js) | Azure
titleSuffix: Microsoft identity platform
description: Apprenez-en davantage sur les problèmes connus d’utilisation de la bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) avec les navigateurs Internet Explorer et Microsoft Edge.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 05/18/2020
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 2a471504b88791b5bfb6ce6cc7c81d60bfbe5028
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83772078"
---
# <a name="known-issues-on-internet-explorer-and-microsoft-edge-browsers-msaljs"></a>Problèmes connus sur les navigateurs Internet Explorer et Microsoft Edge (MSAL.js)

## <a name="issues-due-to-security-zones"></a>Problèmes dus à des zones de sécurité
Des problèmes d’authentification liés à IE et à Microsoft Edge (depuis la mise à jour de la *version du navigateur Microsoft Edge vers 40.15063.0.0* ) nous ont été signalés à plusieurs reprises. Nous les suivons et avons informé l’équipe Microsoft Edge. Pendant que celle-ci travaille à les résoudre, voici une description des problèmes fréquents et des solutions de contournement possibles.

### <a name="cause"></a>Cause
La cause de la plupart de ces problèmes est la suivante. Le stockage de session et le stockage local sont partitionnés par des zones de sécurité dans le navigateur Microsoft Edge. Dans cette version de Microsoft Edge, lorsque l’application est redirigée vers plusieurs zones, le stockage de session et le stockage local sont effacés. Plus précisément, le stockage de session est effacé dans la navigation normale, et tant le stockage de session et que le stockage local sont effacés dans le mode InPrivate du navigateur. MSAL.js enregistre certains états dans le stockage de session et s’appuie sur la vérification de cet état dans le cadre des flux d’authentification. Lorsque le stockage de session est effacé, cet état est perdu et entraîne des expériences interrompues.

### <a name="issues"></a>Problèmes

- **Boucles de redirection infinies et rechargements de page lors de l’authentification**. Lorsque des utilisateurs se connectent à l’application sur Microsoft Edge, ils sont redirigés à partir de la page de connexion AAD et se retrouvent enfermés dans une boucle de redirection infinie entraînant des rechargements de page à répétition. Cela s’accompagne généralement d’une erreur `invalid_state` dans le stockage de session.

- **Boucles d’acquisition de jeton infinies et erreur AADSTS50058**. Quand une application s’exécutant sur Microsoft Edge tente d’acquérir un jeton pour une ressource, elle peut se retrouver bloquée dans une boucle infinie d’appels pour l’acquisition de jeton avec l’erreur suivante d’AAD dans votre trace réseau :

    `Error :login_required; Error description:AADSTS50058: A silent sign-in request was sent but no user is signed in. The cookies used to represent the user's session were not sent in the request to Azure AD. This can happen if the user is using Internet Explorer or Edge, and the web app sending the silent sign-in request is in different IE security zone than the Azure AD endpoint (login.microsoftonline.com)`

- **La fenêtre contextuelle ne se ferme pas ou est bloquée lors de l’utilisation de la connexion via la fenêtre contextuelle pour l’authentification**. Lors de l’authentification via une fenêtre contextuelle dans Microsoft Edge ou IE (InPrivate), une fois la saisie des informations d’identification et la connexion effectuées, si plusieurs domaines dans les zones de sécurité sont impliqués dans la navigation, la fenêtre contextuelle ne se ferme pas parce que MSAL.js perd le descripteur de la fenêtre contextuelle.  

### <a name="update-fix-available-in-msaljs-023"></a>Mise à jour : Correctif disponible dans MSAL.js 0.2.3
Des correctifs pour les problèmes de boucle de redirection d’authentification ont été publiés dans [MSAL.js 0.2.3](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases). Activez l’indicateur `storeAuthStateInCookie` dans la configuration MSAL.js pour tirer parti de ce correctif. Par défaut, cet indicateur est défini sur false.

Lorsque l’indicateur `storeAuthStateInCookie` est activé, MSAL.js utilise les cookies du navigateur pour stocker l’état de demande requis pour la validation des flux d’autorisations.

> [!NOTE]
> Ce correctif n’est pas encore disponible pour les wrappers msal-angular et msal-angularjs. Ce correctif ne règle pas le problème lié aux fenêtres contextuelles.

Utilisez les solutions de contournement ci-dessous.

#### <a name="other-workarounds"></a>Autres solutions de contournement
Avant de mettre en œuvre ces solutions de contournement, testez pour vérifier que votre problème ne se produit que sur cette version du navigateur Microsoft Edge et fonctionne sur les autres navigateurs.  
1. En guise de première étape, pour résoudre ces problèmes, assurez-vous que le domaine d’application et tous les autres sites impliqués dans les redirections du flux d’authentification sont ajoutés en tant que sites de confiance dans les paramètres de sécurité du navigateur, de sorte qu’ils appartiennent à la même zone de sécurité.
Pour ce faire, procédez comme suit :
    - Ouvrez **Internet Explorer**, puis cliquez sur **Paramètres** (icône d’engrenage) dans l’angle supérieur droit.
    - Sélectionnez **Options Internet**.
    - Sélectionnez l’onglet **Sécurité**.
    - Sous l’option **Sites de confiance**, cliquez sur le bouton **Sites**, puis ajoutez les URL dans la boîte de dialogue qui s’ouvre.

2. Comme mentionné précédemment, étant donné que seul le stockage de session est effacé durant la navigation normale, vous pouvez configurer MSAL.js pour utiliser le stockage local à la place. Vous pouvez définir cela via le paramètre de configuration `cacheLocation` lors de l’initialisation de MSAL.

Notez que cela ne résout pas le problème de navigation InPrivate, car tant le stockage de session que le stockage local sont effacés.

## <a name="issues-due-to-popup-blockers"></a>Problèmes dus à des bloqueurs de fenêtres contextuelles

Parfois, des fenêtres contextuelles sont bloquées dans IE ou Microsoft Edge, par exemple quand une deuxième fenêtre contextuelle s’affiche lors de l’[authentification multifacteur](../authentication/concept-mfa-howitworks.md). Une alerte s’affiche dans le navigateur, qui vous permet de décider si la fenêtre contextuelle doit s’afficher une seule fois ou toujours. Si vous choisissez d’autoriser l’affichage de la fenêtre, le navigateur l’ouvre automatiquement et retourne un descripteur `null` approprié. Par conséquent, la bibliothèque ne contient pas de descripteur pour la fenêtre contextuelle, et il n’existe aucun moyen de fermer celle-ci. Ce problème ne se produit pas dans Chrome quand il vous invite à autoriser les fenêtres contextuelles, parce qu’il n’ouvre pas automatiquement de fenêtre contextuelle.

Pour **éviter ce problème**, les développeurs doivent autoriser les fenêtres contextuelles dans IE et Microsoft Edge avant de commencer à utiliser leur application.

## <a name="next-steps"></a>Étapes suivantes
Apprenez-en davantage sur l’[utilisation de MSAL.js dans Internet Explorer](msal-js-use-ie-browser.md).
