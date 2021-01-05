---
title: Guide pratique pour gérer les comptes d’utilisateur inactifs dans Azure AD | Microsoft Docs
description: Découvrir comment détecter et gérer des comptes d’utilisateur devenus obsolètes dans Azure AD
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/07/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5b48a84bb69a356815cccd1e33c555eeb667699f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89244719"
---
# <a name="how-to-manage-inactive-user-accounts-in-azure-ad"></a>Procédure : Gérer les comptes d’utilisateur inactifs dans Azure AD

Dans les grands environnements, les comptes d’utilisateur ne sont pas toujours supprimés quand les employés quittent une organisation. En tant qu’administrateur informatique, vous souhaitez détecter et gérer ces comptes d’utilisateur obsolètes parce qu’ils constituent un risque pour la sécurité.

Cet article décrit une méthode permettant de gérer les comptes d’utilisateur obsolètes dans Azure AD. 

## <a name="what-are-inactive-user-accounts"></a>Que sont les comptes d’utilisateur inactifs ?

Les comptes inactifs sont des comptes d’utilisateur qui ne sont plus requis par les membres de votre organisation pour accéder à vos ressources. Un indicateur clé pour les comptes inactifs est qu’ils n’ont pas été utilisés *pendant un certain temps* pour se connecter à votre environnement. Étant donné que les comptes inactifs sont liés à l’activité de connexion, vous pouvez utiliser l’horodatage de la dernière connexion réussie pour les détecter. 

Le défi de cette méthode consiste à définir à quoi correspond *un certain temps* dans le cas de votre environnement. Par exemple, les utilisateurs peuvent ne pas se connecter à un environnement *pendant un certain temps* parce qu’ils sont en congés. Lorsque vous définissez la valeur de votre delta pour les comptes d’utilisateur inactifs, vous devez prendre en compte toutes les raisons légitimes pour ne pas se connecter à votre environnement. Dans de nombreuses organisations, le delta pour les comptes d’utilisateur inactifs est compris entre 90 et 180 jours. 

La dernière connexion réussie fournit des informations potentielles sur le besoin constant d’un utilisateur d’accéder aux ressources.  Elle contribue à déterminer si l’appartenance au groupe ou l’accès à l’application est toujours nécessaire ou peut être supprimé. Pour la gestion des utilisateurs externes, vous pouvez déterminer si un utilisateur externe est toujours actif dans le locataire ou s’il doit être nettoyé. 

    
## <a name="how-to-detect-inactive-user-accounts"></a>Procédure de détection des comptes d’utilisateur inactifs

Vous détectez les comptes inactifs en évaluant la propriété **lastSignInDateTime** exposée par le type de ressource **signInActivity** de l’API **Microsoft Graph**. À l’aide de cette propriété, vous pouvez implémenter une solution pour les scénarios suivants :

- **Utilisateurs par nom** : Dans ce scénario, vous recherchez un utilisateur spécifique par son nom, ce qui vous permet d’évaluer lastSignInDateTime : `https://graph.microsoft.com/beta/users?$filter=startswith(displayName,'markvi')&$select=displayName,signInActivity`

- **Utilisateurs par date** : Dans ce scénario, vous demandez une liste d’utilisateurs avec une propriété lastSignInDateTime avant une date spécifiée : `https://graph.microsoft.com/beta/users?filter=signInActivity/lastSignInDateTime le 2019-06-01T00:00:00Z`






## <a name="what-you-need-to-know"></a>Bon à savoir

Cette section liste ce que vous devez savoir sur la propriété lastSignInDateTime.

### <a name="how-can-i-access-this-property"></a>Comment je peux accéder à cette propriété ?

La propriété **lastSignInDateTime** est exposée par le [type de ressource signInActivity](/graph/api/resources/signinactivity?view=graph-rest-beta) de l’[API REST Microsoft Graph](/graph/overview?view=graph-rest-beta#whats-in-microsoft-graph).   

### <a name="is-the-lastsignindatetime-property-available-through-the-get-azureaduser-cmdlet"></a>La propriété lastSignInDateTime est-elle disponible via l’applet de commande Get-AzureAdUser ?

Non.

### <a name="what-edition-of-azure-ad-do-i-need-to-access-the-property"></a>De quelle édition d’Azure AD ai-je besoin pour accéder à cette propriété ?

Vous pouvez accéder à cette propriété dans toutes les éditions d’Azure AD.

### <a name="what-permission-do-i-need-to-read-the-property"></a>De quelle autorisation ai-je besoin pour lire cette propriété ?

Pour lire cette propriété, vous devez accorder les droits suivants : 

- AuditLogs.Read.All
- Organisation.Read.All  


### <a name="when-does-azure-ad-update-the-property"></a>Quand Azure AD met-il à jour cette propriété ?

Chaque connexion interactive réussie aboutit à une mise à jour du magasin de données sous-jacent. En général, les connexions réussies s’affichent dans le rapport de connexion associé dans un délai de 10 minutes.
 

### <a name="what-does-a-blank-property-value-mean"></a>Que signifie une valeur de propriété vide ?

Pour générer un horodatage lastSignInDateTime, vous avez besoin d’une connexion réussie. Étant donné que la propriété lastSignInDateTime est une nouvelle fonctionnalité, la valeur de la propriété lastSignInDateTime peut être vide dans les cas suivants :

- La dernière connexion réussie d’un utilisateur a été effectuée avant la publication de cette fonctionnalité (1er décembre 2019).
- Le compte d’utilisateur affecté n’a jamais été utilisé pour une connexion réussie.

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir des données à l’aide de l’API de création de rapports Azure Active Directory avec des certificats](tutorial-access-api-with-certificates.md)
* [Informations de référence sur l’API d’audit](/graph/api/resources/directoryaudit?view=graph-rest-beta) 
* [Informations de référence sur l’API de création de rapports relatifs à l’activité de connexion](/graph/api/resources/signin?view=graph-rest-beta)