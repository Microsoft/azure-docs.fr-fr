---
title: Inscription en libre-service pour les identités externes – Azure AD
description: Découvrez comment permettre à des utilisateurs externes de s’inscrire eux-même pour accéder à vos applications en activant l’inscription en libre-service. Créez une expérience d’inscription sur mesure en personnalisant le flux utilisateur d’inscription en libre-service.
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
ms.openlocfilehash: 34b08e2e530843dd98c87e424812706247388228
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87907130"
---
# <a name="self-service-sign-up-preview"></a>Inscription en libre-service (préversion)

> [!NOTE]
> L’inscription en libre-service est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Lorsque vous partagez une application avec des utilisateurs externes, vous ne savez pas toujours à l’avance qui aura besoin d’y accéder. Au lieu d’envoyer des invitations directement à des personnes, vous pouvez autoriser les utilisateurs externes à s’inscrire pour accéder à des applications spécifiques en activant l’inscription en libre-service. Vous pouvez créez une expérience d’inscription sur mesure en personnalisant le flux utilisateur d’inscription en libre-service. Par exemple, vous pouvez fournir des options d’inscription auprès d’Azure AD ou de fournisseurs d’identité sociale, et collecter des informations sur l’utilisateur durant l’inscription.

> [!NOTE]
> Vous pouvez associer des flux d’utilisateurs à des applications générées par votre organisation. Les flux d’utilisateurs ne peuvent pas être utilisés pour les applications Microsoft telles que SharePoint ou Teams.

## <a name="user-flow-for-self-service-sign-up"></a>Flux utilisateur pour l’inscription en libre-service

Un flux utilisateur d’inscription en libre-service crée une expérience d’inscription pour vos utilisateurs externes par le biais de l’application que vous souhaitez partager. Le flux utilisateur peut être associé à une ou plusieurs de vos applications. Tout d’abord, vous allez activer l’inscription en libre-service pour votre locataire, et fédérer avec les fournisseurs d’identité que les utilisateurs externes seront autorisés à utiliser pour la connexion. Ensuite, vous allez créer et personnaliser le flux utilisateur d’inscription et lui affecter vos applications.
Vous pouvez configurer des paramètres de flux utilisateur pour contrôler la façon dont l’utilisateur s’inscrit pour accéder à l’application :

- Types de compte utilisés pour la connexion, comme des comptes de réseaux sociaux de type Facebook ou des comptes Azure AD
- Attributs à collecter lors de l’inscription de l’utilisateur, tels que le prénom, le code postal ou le pays/la région de résidence

Quand un utilisateur veut se connecter à votre application, qu’il s’agisse d’une application web, mobile, de bureau ou monopage (SPA), cette application envoie une demande d’autorisation au point de terminaison fourni par un flux utilisateur. Le flux utilisateur définit et contrôle l’expérience de l’utilisateur. Quand l’utilisateur termine le flux d’inscription, Azure AD génère un jeton, puis redirige l’utilisateur vers votre application. Une fois l’inscription terminée, un compte invité est approvisionné pour l’utilisateur dans le répertoire. Plusieurs applications peuvent utiliser les mêmes flux utilisateur.

## <a name="example-of-self-service-sign-up"></a>Exemple d’inscription en libre-service

L’exemple suivant montre comment nous dirigeons les fournisseurs d’identité sociale vers Azure AD avec des fonctionnalités d’inscription en libre-service pour les utilisateurs invités.  
Un partenaire de Woodgrove ouvre l’application Woodgrove. Il décide qu’il souhaite s’inscrire pour accéder à un compte de fournisseur. Il choisit donc de demander votre compte fournisseur, ce qui initie le flux d’inscription en libre-service.

![Exemple de page de démarrage d’inscription en libre-service](media/self-service-sign-up-overview/example-start-sign-up-flow.png)

Il utilise l’e-mail de son choix pour s’inscrire.

![Exemple illustrant la sélection de Facebook pour la connexion](media/self-service-sign-up-overview/example-sign-in-with-facebook.png)

Azure AD crée une relation avec Woodgrove en utilisant le compte Facebook du partenaire, et crée un compte invité pour l’utilisateur une fois celui-ci inscrit.

Woodgrove souhaite en savoir plus sur l’utilisateur, par exemple, son nom, son nom d’entreprise, son numéro d’immatriculation d’entreprise et son numéro de téléphone.

![Exemple montrant des attributs d’inscription d’un utilisateur](media/self-service-sign-up-overview/example-enter-user-attributes.png)

L’utilisateur entre les informations, poursuit le flux d’inscription et obtient l’accès aux ressources dont il a besoin.

![Exemple montrant l’utilisateur connecté](media/self-service-sign-up-overview/example-signed-in.png)

## <a name="next-steps"></a>Étapes suivantes

 Pour plus d’informations, découvrez comment [ajouter l’inscription en libre-service à une application](self-service-sign-up-user-flow.md).