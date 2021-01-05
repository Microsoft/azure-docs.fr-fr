---
title: Fournisseurs d’identité pour les identités externes – Azure AD
description: Découvrez comment utiliser Azure AD comme fournisseur d’identité par défaut pour le partage avec des utilisateurs externes.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: a61eca3340985886ef782b28a59a1283c5f473ce
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91265709"
---
# <a name="identity-providers-for-external-identities"></a>Fournisseurs d’identité pour les identités externes

Un *fournisseur d’identité* crée, entretient et gère les informations d’identité tout en fournissant des services d’authentification pour les applications. Lors du partage de vos applications et ressources avec des utilisateurs externes, Azure AD est le fournisseur d’identité par défaut. Cela signifie que, quand vous invitez des utilisateurs externes qui disposent déjà d’un compte Azure AD ou Microsoft, ceux-ci peuvent se connecter automatiquement sans configuration supplémentaire de votre part.

Cependant, vous pouvez leur permettre de se connecter avec différents fournisseurs d’identité.

- **Google** : la fédération des services Google permet à des utilisateurs externes de donner suite à des invitations que vous leur avez envoyées en se connectant à vos applications avec leurs propres comptes Gmail. Vous pouvez également utiliser la fédération des services Google dans vos flux d’utilisateurs d’inscription en libre-service.
   > [!NOTE]
   > Dans la préversion actuelle de l’inscription en libre-service, si un flux d’utilisateurs est associé à une application et que vous envoyez à une utilisateur une invitation à cette application, l’utilisateur ne peut pas utiliser un compte Gmail pour donner suite à l’invitation. Une solution de contournement pour l’utilisateur consiste à suivre le processus d’inscription en libre-service. Il peut également donner suite à l’invitation en accédant à une autre application ou en utilisant son portail Mes applications à l’adresse https://myapps.microsoft.com.

- **Facebook** : lors de la création d’une application, vous pouvez configurer l’inscription en libre-service et activer un fédération Facebook afin que les utilisateurs puissent s’inscrire à votre application en utilisant leurs propres comptes Facebook. Facebook ne peut être utilisé que pour des flux d’utilisateurs d’inscription en libre-service, et n’est pas disponible en tant qu’option de connexion lorsque des utilisateurs donnent suite à des invitations de votre part.

- **Fédération directe** : vous pouvez également configurer une fédération avec n’importe quel fournisseur d’identité externe prenant en charge les protocoles SAML ou WS-Fed. Une fédération directe permet à des utilisateurs externes de donner suite à des invitations que vous leur avez envoyées en se connectant à vos applications avec leurs comptes professionnels ou de réseaux sociaux existants. 
   > [!NOTE]
   > Vous ne pouvez pas utiliser des fournisseurs d’identité de fédération directe dans vos flux d’utilisateurs d’inscription en libre-service.


## <a name="how-it-works"></a>Fonctionnement

La fonctionnalité d’inscription en libre-service avec des identités externes Azure AD permet aux utilisateurs de s’inscrire avec leur compte Azure AD, Google ou Facebook. Pour configurer des fournisseurs d’identité sociale dans votre locataire Azure AD, vous devez créer une application chez chaque fournisseur d’identité et configurer des informations d’identification. Vous recevez un ID de client ou d’application et une clé secrète de client ou d’application que vous pouvez ensuite ajouter à votre locataire Azure AD.

Une fois que vous avez ajouté un fournisseur d’identité à votre locataire Azure AD :

- Lorsque vous invitez un utilisateur externe à des applications ou ressources se trouvant dans votre organisation, l’utilisateur externe peut se connecter avec son propre compte auprès de ce fournisseur d’identité.
- Lorsque vous activez l’[inscription en libre-service](self-service-sign-up-overview.md) pour vos applications, des utilisateurs externes peuvent s’inscrire pour accéder à vos applications à l’aide de leurs propres comptes auprès des fournisseurs d’identité que vous avez ajoutés.

> [!NOTE]
> Azure AD étant activé par défaut pour l’inscription en libre-service, les utilisateurs ont toujours la possibilité de s’inscrire à l’aide d’un compte Azure AD.

Quand vous échangez votre invitation ou vous inscrivez pour votre application, l’utilisateur externe a la possibilité de se connecter et de s’authentifier auprès du fournisseur d’identité sociale :

![Capture d’écran montrant l’écran de connexion avec les options Google et Facebook](media/identity-providers/sign-in-with-social-identity.png)

Pour une expérience de connexion optimale, fédérez des fournisseurs d’identité autant que possible afin de pouvoir offrir à vos invités une expérience de connexion transparente quand ils accèdent à vos applications.  

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment ajouter des fournisseurs d’identité pour la connexion à vos applications, reportez-vous aux articles suivants :

- [Ajouter Google](google-federation.md) à votre liste de fournisseurs d’identité sociale
- [Ajouter Facebook](facebook-federation.md) à votre liste de fournisseurs d’identité sociale
- [Configurer une fédération directe](direct-federation.md) avec n’importe quelle organisation dont le fournisseur d’identité prend en charge le protocole SAML 2.0 ou WS-Fed. Notez qu’une fédération directe n’est pas possible pour des flux utilisateur d’inscription en libre-service.
