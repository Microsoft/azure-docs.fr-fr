---
title: API de soumission de l’Espace partenaires pour intégrer des applications Azure dans la Place de marché commerciale Microsoft
description: Découvrez les conditions préalables à l’utilisation de l’API de soumission de l’Espace partenaires pour les applications Azure dans la Place de marché commerciale sur l’Espace partenaires Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 12/10/2019
ms.author: mingshen
author: mingshen-ms
ms.openlocfilehash: 9863ed24da9e427f885a4794bda7e103b0c1cc8e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455453"
---
# <a name="partner-center-submission-api-to-onboard-azure-apps-in-partner-center"></a>API de soumission de l’Espace partenaires pour l’intégration d’applications Azure dans l’Espace partenaires

Utilisez *l’API de soumission de l’Espace partenaires* pour interroger des offres Azure programmatiquement, créer des soumissions et publier les offres.  Cette API est utile si votre compte gère de nombreuses offres et que vous souhaitez automatiser et optimiser le processus de soumission de ces offres.

## <a name="api-prerequisites"></a>Prérequis de l’API

Différentes ressources de programmation sont nécessaires pour pouvoir utiliser l’API de l’Espace partenaires pour les produits Azure : 

- une application Azure Active Directory ;
- un jeton d’accès Azure Active Directory (Azure AD).

### <a name="step-1-complete-prerequisites-for-using-the-partner-center-submission-api"></a>Étape 1 : Suivre les prérequis de l’API de soumission de l’Espace partenaires

Avant de commencer à écrire du code pour appeler l’API de soumission de l’Espace partenaires, veillez à respecter les prérequis suivants.

- Vous (ou votre organisation) devez disposer d’un annuaire Azure AD et de l’autorisation [Administrateur général](../../active-directory/roles/permissions-reference.md) sur l’annuaire. Si vous utilisez déjà Microsoft 365 ou d’autres services professionnels de Microsoft, vous disposez déjà d’un annuaire Azure AD. Dans le cas contraire, vous pouvez [créer un annuaire Azure AD dans l’Espace partenaires](/windows/uwp/publish/associate-azure-ad-with-partner-center#create-a-brand-new-azure-ad-to-associate-with-your-partner-center-account) sans frais supplémentaires.

- Vous devez [associer une application Azure AD à votre compte Espace partenaires](/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services#associate-an-azure-ad-application-with-your-windows-partner-center-account) et récupérer votre ID tenant, votre ID client et votre clé. Ces valeurs sont nécessaires pour obtenir un jeton d’accès Azure AD, que vous utiliserez dans les appels à l’API de soumission au Microsoft Store.

#### <a name="how-to-associate-an-azure-ad-application-with-your-partner-center-account"></a>Associer une application Azure AD à un compte Espace partenaires

Pour utiliser l’API de soumission au Microsoft Store, vous devez associer une application Azure AD à votre compte Espace partenaires, récupérer l’ID tenant et l’ID client de l’application, et générer une clé. L’application Azure AD représente l’application ou le service à partir duquel vous appellerez l’API de soumission de l’Espace partenaires. Il vous faut l’ID tenant, l’ID client et la clé pour obtenir un jeton d’accès Azure AD à transmettre à l’API.

>[!Note]
>Vous ne devez effectuer cette tâche qu’une seule fois. Une fois que vous les avez, vous pouvez réutiliser l’ID tenant, l’ID client et la clé chaque fois que vous devez créer un jeton d’accès Azure AD.

1. Dans l’Espace partenaires, [associez le compte Espace partenaires de votre organisation à l’annuaire Azure AD de votre organisation](/windows/uwp/publish/associate-azure-ad-with-partner-center).
1. Ensuite, sur la page **Utilisateurs** de la section **Paramètres de compte** de l’Espace partenaires, [ajoutez l’application Azure AD](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#add-azure-ad-applications-to-your-partner-center-account) représentant l’application ou le service que vous utiliserez pour accéder aux soumissions de votre compte Espace partenaires. Veillez à attribuer à cette application le rôle **Gestionnaire**. Si l’application n’existe pas encore dans votre annuaire Azure AD, vous pouvez [créer une application Azure AD dans l’Espace partenaires](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#create-a-new-azure-ad-application-account-in-your-organizations-directory-and-add-it-to-your-partner-center-account).
1. Revenez à la page **Utilisateurs**, cliquez sur le nom de votre application Azure AD pour accéder à ses paramètres, puis copiez les valeurs **ID tenant** et **ID client**.
1. Cliquez sur **Ajouter une nouvelle clé**. Sur l’écran suivant, copiez la valeur **Clé**. Vous ne pourrez plus accéder à cette information une fois que vous aurez quitté cette page. Pour plus d’informations, voir [Gérer les clés pour une application Azure AD](/windows/uwp/publish/add-users-groups-and-azure-ad-applications#manage-keys).

### <a name="step-2-obtain-an-azure-ad-access-token"></a>Étape 2 : Obtenir un jeton d’accès Azure AD

Avant d’appeler les méthodes de l’API de soumission de l’Espace partenaires, vous devez récupérer un jeton d’accès Azure AD à transmettre à l’en-tête **Autorisation** de chacune des méthodes de l’API. Une fois que vous avez récupéré le jeton d’accès, vous avez 60 minutes pour l’utiliser avant qu’il n’expire. Après expiration, vous pouvez l’actualiser pour pouvoir continuer à l’utiliser dans de nouveaux appels à l’API.

Pour obtenir le jeton d’accès, suivez les instructions de la section [Appels de service à service à l’aide des informations d’identification du client](../../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md) pour envoyer une requête `HTTP POST` au point de terminaison `https://login.microsoftonline.com/<tenant_id>/oauth2/token`. Voici un exemple de requête :

JSONCopy
```Json
POST https://login.microsoftonline.com/<tenant_id>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8

grant_type=client_credentials
&client_id=<your_client_id>
&client_secret=<your_client_secret>
&resource= https://api.partner.microsoft.com
```

Pour la valeur *tenant_id* dans le `POST URI` et les paramètres *client_id* et *client_secret*, spécifiez l’ID tenant, l’ID client et la clé de votre application que vous avez récupérée dans l’Espace partenaires à la section précédente. Pour le paramètre *resource*, vous devez spécifier `https://api.partner.microsoft.com`.

### <a name="step-3-use-the-microsoft-store-submission-api"></a>Étape 3 : Utiliser l’API de soumission au Microsoft Store

Une fois que vous disposez d’un jeton d’accès Azure AD, vous pouvez appeler des méthodes de l’API de soumission de l’Espace partenaires. Pour créer ou mettre à jour des soumissions, on appelle généralement plusieurs méthodes de l’API de soumission de l’Espace partenaires dans un ordre spécifique. Pour plus d’informations sur les différents scénarios et la syntaxe de chaque méthode, consultez l’API d’ingestion Swagger.

https://apidocs.microsoft.com/services/partneringestion/

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [Créer une ressource technique de machine virtuelle Azure](../create-azure-container-technical-assets.md)
* Découvrez comment [Créer une offre de conteneur Azure](../create-azure-container-offer.md)