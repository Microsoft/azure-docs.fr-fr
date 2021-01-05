---
title: Champs d’inscription du portail Azure pour les applications personnalisées
description: Conseils pour l’inscription d’une application personnalisée auprès d’Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 06/28/2019
ms.author: ryanwi
ms.openlocfilehash: 1469fcfa68b10353b78d31ccc758c61bf7746692
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88120692"
---
# <a name="azure-portal-registration-fields-for-custom-developed-apps"></a>Champs d’inscription du portail Azure pour les applications personnalisées

Cet article décrit brièvement tous les champs disponibles dans le formulaire d’inscription d’application du [portail Azure](https://portal.azure.com).

## <a name="register-a-new-application"></a>Inscrire une nouvelle application

-   Pour inscrire une nouvelle application, accédez au [portail Azure](https://portal.azure.com).

-   Dans le volet de navigation gauche, cliquez sur **Azure Active Directory.**

-   Choisissez **Inscriptions des applications**, puis cliquez sur **Ajouter**.

-   Le formulaire d’inscription d’application s’affiche ensuite.

## <a name="fields-in-the-application-registration-form"></a>Champs du formulaire d’inscription d’application

| Champ            | Description                                                                              |
|------------------|------------------------------------------------------------------------------------------|
| Nom             | Le nom de l’application. Il doit être composé de quatre caractères minimum.                |
| Types de comptes pris en charge| Sélectionnez les comptes que vous souhaitez que votre application prenne en charge : les comptes dans cet annuaire organisationnel uniquement, les comptes dans n’importe quel annuaire organisationnel ou les comptes dans n’importe quel annuaire organisationnel et compte Microsoft personnel.  |
| URI de redirection (facultatif) | sélectionnez le type d’application que vous créez, **Web** ou **Client public (mobile et bureau)** , puis entrez l’URI de redirection (ou URL de réponse) de votre application. Pour les applications web, indiquez l’URL de base de votre application. Par exemple, http://localhost:31544 peut être l’URL pour une application web en cours d’exécution sur votre ordinateur local. Les utilisateurs peuvent utiliser cette URL pour se connecter à une application web cliente. Pour les applications de client public, indiquez l’URI utilisé par Azure AD pour retourner les réponses de jeton. Entrez une valeur spécifique de votre application, par exemple, myapp://auth. Pour voir des exemples spécifiques pour les applications web ou natives, consultez les [Guides de démarrage rapides](./index.yml).|

Après avoir rempli les champs ci-dessus, l’application est inscrite dans le portail Azure et vous êtes redirigé vers la page de vue d’ensemble de l’application. Les pages des paramètres dans le volet gauche, sous **Gérer**, ont plus de champs pour personnaliser votre application. Les tableaux ci-dessous décrivent tous les champs. Vous voyez uniquement un sous-ensemble de ces champs. Cet affichage peut changer si vous avez créé une application web ou une application cliente publique.

### <a name="overview"></a>Vue d’ensemble

| Champ           | Description        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID de l'application  | Lorsque vous inscrivez une application, Azure AD attribue un ID d’application à votre application. Cet ID d’application peut être utilisé pour identifier de façon unique votre application dans les demandes d’authentification à Azure AD, ainsi que pour accéder à des ressources telles que l’API Graph.                                                          |
| URI ID d’application      | Cette valeur doit être un URI unique, généralement au format **https://&lt;nom\_locataire&gt;/&lt;nom\_application&gt;.** Elle est utilisée dans le flux d’octroi d’autorisation en tant qu’identificateur unique pour spécifier la ressource pour laquelle le jeton doit être émis. Il devient également la revendication « aud » dans le jeton d’accès émis. |

### <a name="branding"></a>Personnalisation

| Champ           | Description        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Charger un nouveau logo | Ce champ vous permet de charger un logo pour votre application. Le logo doit être au format .bmp, .jpg ou .png, et la taille du fichier doit être inférieure à 100 Ko. Les dimensions de l’image doivent être de 215 x 215 pixels et les dimensions de l’image centrale de 94 x 94 pixels.|
| URL de la page d’accueil   | Il s’agit de l’URL de connexion spécifiée lors de l’inscription de l’application.|

### <a name="authentication"></a>Authentification

| Champ           | Description        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| URL de déconnexion      | Il s’agit de l’URL de déconnexion à authentification unique. Azure AD envoie une demande de déconnexion à cette URL lorsque l’utilisateur efface sa session avec Azure AD à l’aide d’une autre application inscrite.|
| Types de comptes pris en charge  | Ce paramètre spécifie si l’application peut être utilisée par plusieurs locataires. En règle générale, cela signifie que des organisations externes peuvent utiliser votre application en l’inscrivant dans leur locataire et en accordant l’accès aux données de leur organisation.|
| URL de redirection      | Les URL de redirection (ou de réponse) sont les points de terminaison auxquels Azure AD renvoie les jetons demandés par votre application. Pour les applications natives, c’est l’emplacement vers lequel l’utilisateur est dirigé après avoir été authentifié. Azure AD vérifie que l’URI de redirection que votre application fournit dans la demande OAuth 2.0 correspond à l’une des valeurs inscrites dans le portail.|

### <a name="certificates-and-secrets"></a>Certificats et secrets

| Champ           | Description        |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Clés secrètes client            | Vous pouvez créer des clés secrètes client (ou clés) pour accéder par programme aux API web sécurisées par Azure AD sans aucune interaction de l’utilisateur. Depuis la page **Nouvelle clé secrète client**, saisissez une description de clé et la date d’expiration, puis enregistrez pour générer la clé. Veillez à l’enregistrer dans un endroit sûr, car vous ne pourrez pas y accéder ultérieurement.             |

## <a name="next-steps"></a>Étapes suivantes

[Gestion des applications avec Azure Active Directory](../manage-apps/what-is-application-management.md)