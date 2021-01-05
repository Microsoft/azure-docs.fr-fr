---
title: Ajouter les informations de confidentialité de votre organisation - Azure Active Directory | Microsoft Docs
description: Instructions concernant l’ajout d’informations de confidentialité de votre organisation à la zone Propriétés Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
ms.date: 04/17/2018
ms.author: ajburnle
ms.reviewer: bpham
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: f47ae9b087615a77f6bd4d3f14f1b0052037cba6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996750"
---
# <a name="add-your-organizations-privacy-info-using-azure-active-directory"></a>Ajouter les informations de confidentialité de votre organisation à l’aide d’Azure Active Directory
Cet article explique comment un administrateur de locataires peut ajouter des informations de confidentialité au locataire Azure Active Directory (Azure AD) d’une organisation, par le biais du portail Azure.

Nous vous recommandons vivement d’ajouter votre contact international chargé de la confidentialité et la déclaration de confidentialité de votre organisation, pour que vos employés internes et invités externes puissent consulter vos stratégies. Étant donné que les déclarations de confidentialité sont particulièrement créées et adaptées à chaque entreprise, nous vous recommandons vivement de contacter un conseil juridique à des fins d’assistance.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="add-your-privacy-info-on-azure-ad"></a>Ajouter vos informations de confidentialité sur Azure AD
Vous ajoutez les informations de confidentialité de votre organisation dans la zone **Propriétés** d’Azure AD.

### <a name="to-access-the-properties-area-and-add-your-privacy-information"></a>Accéder à la zone Propriétés et ajouter vos informations de confidentialité

1. Connectez-vous au portail Azure en tant qu’administrateur de locataires.

2. Dans la barre de navigation gauche, sélectionnez **Azure Active Directory**, puis sélectionnez **Propriétés**.

    La zone **Propriétés** s’affiche.

    ![Zone Propriétés Azure AD avec la zone d’informations de confidentialité mise en exergue](media/active-directory-properties-area/properties-area.png)

3. Ajoutez vos informations de confidentialité pour vos employés :

    - **Contact technique.** Tapez l’adresse e-mail de la personne à contacter pour obtenir un support technique au sein de votre organisation.
    
    - **Contact international chargé de la confidentialité.** Tapez l’adresse e-mail de la personne à contacter pour toute question concernant la confidentialité des données personnelles. Cette personne est également la personne que Microsoft contacte en cas de fuite de données. Si aucune personne n’est répertoriée ici, Microsoft contacte vos administrateurs généraux.

    - **URL de la déclaration de confidentialité.** Tapez le lien vers le document de votre organisation qui décrit la façon dont elle gère la confidentialité des données des utilisateurs internes et invités externes.

        >[!Important]
        >Si vous n’incluez pas votre propre déclaration de confidentialité ou contact chargé de la confidentialité, vos invités externes voient le texte inclus dans la zone **Révision des autorisations**, à savoir **<_nom de votre organisation_> n’a fourni aucun lien vers ses conditions pour vous permettre de les examiner**. Par exemple, un utilisateur invité voit ce message quand il reçoit une invitation à accéder à une organisation par le biais d’une B2B.

        ![Zone Révision des autorisations B2B Collaboration avec message](media/active-directory-properties-area/active-directory-no-privacy-statement-or-contact.png)

4. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes
- [Utilisation d’invitations Azure Active Directory B2B Collaboration](../external-identities/redemption-experience.md)
- [Ajouter ou modifier les informations de profil pour un utilisateur dans Azure Active Directory](active-directory-users-profile-azure-portal.md)