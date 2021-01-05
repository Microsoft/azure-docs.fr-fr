---
title: Attribuer des étiquettes de sensibilité à des groupes-Azure AD | Microsoft Docs
description: Découvrez comment attribuer des étiquettes de sensibilité à des groupes. Consultez les informations de dépannage et les ressources supplémentaires disponibles.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3b40171d87ccffa7c5c4fb72d2e79af015b1d16f
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96859961"
---
# <a name="assign-sensitivity-labels-to-microsoft-365-groups-in-azure-active-directory"></a>Attribuer des étiquettes de sensibilité aux groupes Microsoft 365 dans Azure Active Directory

Azure Active Directory (Azure AD) prend en charge l’application d’étiquettes de sensibilité publiées par le [Centre de conformité Microsoft 365](https://sip.protection.office.com/homepage) dans les groupes Microsoft 365. Les étiquettes de sensibilité s’appliquent au groupe parmi des services tels qu’Outlook, Microsoft Teams et SharePoint. Pour plus d’informations sur la prise en charge des applications Microsoft 365, consultez [Prise en charge de Microsoft 365 pour les étiquettes de sensibilité](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites#support-for-the-sensitivity-labels).

> [!IMPORTANT]
> Pour configurer cette fonctionnalité, au moins une licence Azure Active Directory Premium P1 doit être active dans votre organisation Azure AD.

## <a name="enable-sensitivity-label-support-in-powershell"></a>Activer la prise en charge des étiquettes de sensibilité dans PowerShell

Pour appliquer des étiquettes publiées à des groupes, vous devez d’abord activer la fonctionnalité. Ces étapes activent la fonctionnalité dans Azure AD.

1. Ouvrez une fenêtre Windows PowerShell sur votre ordinateur. Vous pouvez l’ouvrir sans privilèges élevés.
1. Exécutez les commandes suivantes pour préparer l’exécution des applets de commande.

    ```PowerShell
    Import-Module AzureADPreview
    Connect-AzureAD
    ```

    Dans la page **Connectez-vous à votre compte**, entrez votre compte d’administrateur et votre mot de passe pour vous connecter à votre service, puis sélectionnez **Se connecter**.
1. Récupérez les paramètres de groupe actuels pour l’organisation Azure AD.

    ```PowerShell
    $Setting = Get-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id
    ```

    > [!NOTE]
    > Si aucun paramètre de groupe n’a été créé pour cette organisation Azure AD, vous devez d’abord créer les paramètres. Suivez les étapes de [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md) pour créer des paramètres de groupe pour cette organisation Azure AD.

1. Ensuite, affichez les paramètres de groupe actuels.

    ```PowerShell
    $Setting.Values
    ```

1. Activez ensuite la fonctionnalité :

    ```PowerShell
    $Setting["EnableMIPLabels"] = "True"
    ```

1. Ensuite, enregistrez les modifications et appliquez les paramètres :

    ```PowerShell
    Set-AzureADDirectorySetting -Id $Setting.Id -DirectorySetting $Setting
    ```

Vous devrez également synchroniser vos étiquettes de sensibilité dans Azure AD. Pour obtenir des instructions, consultez [Guide pratique pour activer des étiquettes de sensibilité pour des conteneurs et synchroniser des étiquettes](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites#how-to-enable-sensitivity-labels-for-containers-and-synchronize-labels).

## <a name="assign-a-label-to-a-new-group-in-azure-portal"></a>Attribuer une étiquette à un nouveau groupe dans le Portail Azure

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com).
1. Sélectionnez **Groupes**, puis sélectionnez **Nouveau groupe**.
1. Dans la page **Nouveau groupe**, sélectionnez **Office 365**, puis remplissez les informations requises pour le nouveau groupe et sélectionnez une étiquette de sensibilité dans la liste.

   ![Attribuer une étiquette de sensibilité à la page Nouveaux groupes](./media/groups-assign-sensitivity-labels/new-group-page.png)

1. Enregistrez vos modifications, puis sélectionnez **Créer**.

Votre groupe est créé et les paramètres de site et de groupe associés à l'étiquette sélectionnée sont automatiquement appliqués.

## <a name="assign-a-label-to-an-existing-group-in-azure-portal"></a>Attribuer une étiquette à un groupe existant dans le Portail Azure

1. Connectez-vous au [Centre d'administration Azure AD](https://aad.portal.azure.com) avec un compte administrateur de groupes, ou en tant que propriétaire du groupe.
1. Sélectionnez **Groupes**.
1. Dans la page **Tous les groupes**, sélectionnez le groupe que vous souhaitez étiqueter.
1. Sur la page du groupe sélectionné, sélectionnez **Propriétés** et sélectionnez une étiquette de sensibilité dans la liste.

   ![Attribuer une étiquette de sensibilité à la page Vue d’ensemble d’un groupe](./media/groups-assign-sensitivity-labels/assign-to-existing.png)

1. Cliquez sur **Enregistrer** pour enregistrer vos modifications.

## <a name="remove-a-label-from-an-existing-group-in-azure-portal"></a>Retirer une étiquette d’un groupe existant dans le Portail Azure

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte administrateur général ou administrateur de groupes, ou en tant que propriétaire du groupe.
1. Sélectionnez **Groupes**.
1. Dans la page **Tous les groupes**, sélectionnez le groupe duquel vous souhaitez supprimer l’étiquette.
1. Dans la page **Groupe**, sélectionnez **Propriétés**.
1. Sélectionnez **Supprimer**.
1. Sélectionnez **Enregistrer** pour enregistrer vos modifications.

## <a name="using-classic-azure-ad-classifications"></a>Utilisation des classifications Azure AD classiques

Une fois cette fonctionnalité activée, les classifications « classiques » des groupes n'apparaîtront que pour les groupes et sites existants, et vous ne devez les utiliser pour de nouveaux groupes que si vous créez des groupes dans des applications qui ne prennent pas en charge les étiquettes de sensibilité. Si nécessaire, votre administrateur peut ultérieurement les convertir en étiquettes de sensibilité. Les classifications classiques sont les anciennes classifications que vous avez configurées en définissant des valeurs pour le paramètre `ClassificationList` dans Azure AD PowerShell. Lorsque cette fonctionnalité est activée, ces classifications ne sont pas appliquées aux groupes.

## <a name="troubleshooting-issues"></a>Résolution des problèmes

### <a name="sensitivity-labels-are-not-available-for-assignment-on-a-group"></a>Les étiquettes de sensibilité ne peuvent pas être attribuées à un groupe

L’option Étiquette de sensibilité s’affiche uniquement pour les groupes lorsque toutes les conditions suivantes sont remplies :

1. Les étiquettes sont publiées dans le centre de conformité Microsoft 365 pour cette organisation Azure AD.
1. La fonctionnalité est activée, EnableMIPLabels est défini sur True dans le module PowerShell Azure AD.
1. Les étiquettes sont synchronisées avec Azure AD avec l’applet de commande Execute-AzureAdLabelSync dans le module PowerShell Security & Compliance.
1. Le groupe est un groupe Microsoft 365.
1. L’organisation dispose d’une licence active Azure Active Directory Premium P1.
1. L’utilisateur actuellement connecté dispose de privilèges suffisants pour attribuer des étiquettes. L’utilisateur doit être un administrateur général, un administrateur de groupe ou le propriétaire du groupe.

Assurez-vous que toutes les conditions sont remplies pour attribuer des étiquettes à un groupe.

### <a name="the-label-i-want-to-assign-is-not-in-the-list"></a>L’étiquette que je souhaite attribuer ne figure pas dans la liste

Si l’étiquette que vous recherchez ne figure pas dans la liste, cela peut être le cas pour l’une des raisons suivantes :

- L’étiquette n’est peut-être pas publiée dans le centre de conformité Microsoft 365. Cela peut également s’appliquer aux étiquettes qui ne sont plus publiées. Pour plus d’informations, contactez votre administrateur.
- L’étiquette peut être publiée, mais elle n’est pas disponible pour l’utilisateur qui est connecté. Contactez votre administrateur pour plus d’informations sur la façon d’obtenir l’accès à l’étiquette.

### <a name="how-to-change-the-label-on-a-group"></a>Modifier l'étiquette d'un groupe

Les étiquettes peuvent être échangées à tout moment à l’aide des mêmes étapes que l’attribution d’une étiquette à un groupe existant, comme suit :

1. Connectez-vous au [Centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte administrateur général ou administrateur de groupes, ou en tant que propriétaire du groupe.
1. Sélectionnez **Groupes**.
1. Dans la page **Tous les groupes**, sélectionnez le groupe que vous souhaitez étiqueter.
1. Dans la page du groupe sélectionné, sélectionnez **Propriétés** et sélectionnez une nouvelle étiquette de sensibilité dans la liste.
1. Sélectionnez **Enregistrer**.

### <a name="group-setting-changes-to-published-labels-are-not-updated-on-the-groups"></a>Les modifications apportées aux paramètres de groupe sur les étiquettes publiées ne sont pas mises à jour sur les groupes

Il est recommandé de ne pas modifier les paramètres de groupe pour une étiquette une fois l’étiquette appliquée aux groupes. Lorsque vous apportez des modifications aux paramètres de groupe associés aux étiquettes publiées dans le [Centre de conformité Microsoft 365](https://sip.protection.office.com/homepage), ces modifications de stratégie ne sont pas appliquées automatiquement sur les groupes concernés.

Si vous devez apporter une modification, utilisez un [script PowerShell Azure AD](https://github.com/microsoftgraph/powershell-aad-samples/blob/master/ReassignSensitivityLabelToO365Groups.ps1) pour appliquer manuellement les mises à jour aux groupes concernés. Cette méthode garantit que tous les groupes existants appliquent le nouveau paramètre.

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser des étiquettes de sensibilité avec Microsoft Teams, les groupes Microsoft 365 et les sites SharePoint](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites)
- [Mettre à jour les groupes après la modification de la stratégie d’étiquette manuellement avec un script PowerShell Azure AD](https://github.com/microsoftgraph/powershell-aad-samples/blob/master/ReassignSensitivityLabelToO365Groups.ps1)
- [Modifier vos paramètres de groupe](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Gérer des groupes au moyen de commandes PowerShell](../enterprise-users/groups-settings-v2-cmdlets.md)
