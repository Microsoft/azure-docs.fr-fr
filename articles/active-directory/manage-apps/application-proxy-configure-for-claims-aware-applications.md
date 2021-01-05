---
title: Applications prenant en charge les revendications - Proxy d’application Azure AD | Microsoft Docs
description: Comment publier des applications ASP.NET locales qui acceptent les revendications ADFS pour un accès à distance sécurisé par vos utilisateurs.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/08/2018
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: f5c840722ae6b03a0b8a7fa44e5999e14730d4f3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997594"
---
# <a name="working-with-claims-aware-apps-in-application-proxy"></a>Utilisation d’applications prenant en charge les revendications dans le proxy d’application
[Les applications prenant en charge les revendications](/previous-versions/windows/desktop/legacy/bb736227(v=vs.85)) effectuent une redirection vers le service d’émission de jeton de sécurité (STS). Le service d’émission de jeton de sécurité demande des informations d’identification à l’utilisateur en échange d’un jeton, puis redirige l’utilisateur vers l’application. Il existe plusieurs façons d’activer le proxy d’application pour utiliser ces redirections. Utilisez cet article pour configurer votre déploiement pour les applications prenant en charge les revendications. 

## <a name="prerequisites"></a>Prérequis
Vérifiez que le service d’émission de jeton de sécurité vers lequel l’application prenant en charge les revendications effectue la redirection est disponible en dehors de votre réseau local. Vous pouvez rendre le service d’émission de jeton de sécurité disponible en l’exposant par le biais d’un proxy ou en autorisant les connexions externes. 

## <a name="publish-your-application"></a>Publication de votre application

1. Publiez votre application en suivant les instructions décrites dans [Publier des applications avec le proxy d’application](application-proxy-add-on-premises-application.md).
2. Accédez à la page de l’application dans le portail et sélectionnez **Authentification unique**.
3. Si vous choisissez **Azure Active Directory** comme **méthode de préauthentification**, sélectionnez **Authentification unique Azure AD désactivée** comme **méthode d’authentification interne**. Si vous avez choisi **Directe** comme **méthode de préauthentification**, vous n’avez rien à modifier.

## <a name="configure-adfs"></a>Configurer AD FS

Vous pouvez configurer AD FS pour les applications prenant en charge les revendications de deux façons. La première consiste à utiliser des domaines personnalisés. La seconde consiste à utiliser WS-Federation. 

### <a name="option-1-custom-domains"></a>Option 1 : Domaines personnalisés

Si toutes les URL internes de vos applications sont des noms de domaines complets (FQDN), alors vous pouvez configurer des [domaines personnalisés](application-proxy-configure-custom-domain.md) pour vos applications. Utilisez ces domaines personnalisés pour créer des URL externes identiques aux URL internes. Lorsque vos URL externes correspondent à vos URL internes, les redirections STS fonctionnent que vos utilisateurs soient locaux ou à distance. 

### <a name="option-2-ws-federation"></a>Option n°2 : Un certificat de fournisseur d'identité WS-Federation

1. Ouvrez Gestion AD FS.
2. Accédez à **Approbations de la partie de confiance**, cliquez avec le bouton droit sur l’application que vous publiez avec le proxy d’application, puis choisissez **Propriétés**.  

   ![Approbations de partie de confiance, clic droit sur le nom de l’application - capture d’écran](./media/application-proxy-configure-for-claims-aware-applications/appproxyrelyingpartytrust.png)  

3. Sous l’onglet **Points de terminaison**, sous **Type de point de terminaison**, sélectionnez **WS-Federation**.
4. Sous **URL approuvée**, entrez l’URL que vous avez entrée dans le proxy d’application sous **URL externe**, puis cliquez sur **OK**.  

   ![Ajouter un point de terminaison, définition de la valeur de l’URL approuvée – capture d’écran](./media/application-proxy-configure-for-claims-aware-applications/appproxyendpointtrustedurl.png)  

## <a name="next-steps"></a>Étapes suivantes
* [Activation d’applications clientes natives de manière à ce qu’elles interagissent avec des applications proxy](application-proxy-configure-native-client-application.md)