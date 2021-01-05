---
title: Partage externe dans Microsoft 365 et collaboration B2B - Azure AD
description: Décrit le partage de ressources avec des partenaires externes à l’aide de Microsoft 365 et Azure Active Directory B2B Collaboration.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 35ffae55e749c7493b9cd8af13dfd5216abf5822
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94515504"
---
# <a name="microsoft-365-external-sharing-and-azure-active-directory-azure-ad-b2b-collaboration"></a>Partage externe dans Microsoft 365 et collaboration dans Azure Active Directory B2B

Qu’il s’agisse d’une collaboration dans Azure AD B2B ou d’un partage externe dans Microsoft 365 (OneDrive, SharePoint Online, groupes unifiés, etc.), les utilisateurs externes sont authentifiés à l’aide d’Azure AD B2B.

## <a name="how-does-azure-ad-b2b-differ-from-external-sharing-in-sharepoint-online"></a>En quoi Azure AD B2B diffère-t-il du partage externe dans SharePoint Online ?

OneDrive/SharePoint Online possède un gestionnaire d’invitation distinct. La prise en charge par OneDrive/SharePoint Online du partage externe a démarré avant qu’Azure AD ne propose sa prise en charge. Au fil du temps, plusieurs fonctionnalités ont été ajoutées au partage externe OneDrive/SharePoint Online, ce qui a attiré de des millions d’utilisateurs exploitant le modèle de partage intégré. Toutefois, il existe quelques différences subtiles entre le fonctionnement du partage externe OneDrive/SharePoint Online et le fonctionnement d’Azure AD B2B Collaboration. Pour plus d’informations sur le partage externe OneDrive/SharePoint Online, consultez l’article [Vue d’ensemble du partage externe](/sharepoint/external-sharing-overview). Les différences entre ce processus et celui d’Azure AD B2B sont les suivantes :

- OneDrive/SharePoint Online ajoute les utilisateurs au répertoire une fois qu’ils ont utilisé leurs invitations. Par conséquent, avant qu’il utilise son invitation, l’utilisateur n’apparaît pas dans le portail Azure AD. Si un autre site invite un utilisateur dans l’intervalle, une nouvelle invitation est générée. En revanche, lorsque vous utilisez Azure AD B2B Collaboration, les utilisateurs sont ajoutés immédiatement sur invitation afin qu’ils apparaissent partout.

- L’expérience d’utilisation de l’invitation dans OneDrive/SharePoint Online est différente de celle d’Azure AD B2B Collaboration. Une fois l’invitation utilisée, les expériences sont similaires.

- Les utilisateurs invités par Azure AD B2B Collaboration peuvent être sélectionnés à partir des boîtes de dialogue de partage de OneDrive/SharePoint Online. Les utilisateurs invités par OneDrive/SharePoint Online apparaissent également dans Azure AD une fois qu’ils ont utilisé leur invitation.

- Les exigences en termes de licence diffèrent. Pour plus d’informations sur les licences, consultez le [guide d’attribution de licences Azure AD B2B](./external-identities-pricing.md), ainsi que la section [« Qu’est-ce qu’un utilisateur externe ? » dans la vue d’ensemble du partage externe SharePoint Online](/sharepoint/external-sharing-overview#what-happens-when-users-share).

Pour gérer le partage externe dans OneDrive/SharePoint Online avec Azure AD B2B Collaboration, définissez le paramètre de partage externe OneDrive/SharePoint Online sur **Autoriser le partage uniquement avec les utilisateurs externes qui existent déjà dans l’annuaire de votre organisation**. Les utilisateurs peuvent accéder à des sites partagés de manière externe et choisir parmi les collaborateurs externes ajoutés par l’administrateur. L’administrateur peut ajouter les collaborateurs externes par le biais des API d’invitation B2B Collaboration.


![Paramètre OneDrive/SharePoint de partage externe](media/o365-external-user/odsp-sharing-setting.png)

Une fois le partage externe activé, la fonctionnalité de recherche d’utilisateurs invités existants dans le sélecteur de personnes SharePoint Online (SPO) est désactivée par défaut pour correspondre au comportement hérité.

Vous pouvez activer cette fonctionnalité à l’aide du paramètre ShowPeoplePickerSuggestionsForGuestUsers au niveau du client et de la collection du site. Vous pouvez définir cette fonctionnalité à l’aide des applets de commande Set-SPOTenant et Set-SPOSite qui permettent aux membres de rechercher tous les utilisateurs invités existants dans le répertoire. Les modifications apportées à la portée du client n’affectent pas les sites SPO déjà configurés.

## <a name="next-steps"></a>Étapes suivantes

* [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
* [Ajout d’un utilisateur B2B Collaboration à un rôle](add-guest-to-role.md)
* [Déléguer des invitations B2B Collaboration](delegate-invitations.md)
* [Groupes dynamiques et B2B Collaboration](use-dynamic-groups.md)
* [Résolution des problèmes d’Azure Active Directory B2B Collaboration](troubleshoot.md)