---
title: Définir l’expiration des groupes Microsoft 365 - Azure Active Directory | Microsoft Docs
description: Comment configurer l’expiration des groupes Microsoft 365 dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 60d71c44ebaba5b0cbcb03afa41ad15237dceaef
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96547472"
---
# <a name="configure-the-expiration-policy-for-microsoft-365-groups"></a>Configurer la stratégie d’expiration pour les groupes Microsoft 365

Cet article vous explique comment gérer le cycle de vie des groupes Microsoft 365 en définissant une stratégie d’expiration. Vous pouvez définir une stratégie d’expiration uniquement pour les groupes Microsoft 365 d’Azure Active Directory (Azure AD).

Une fois que vous avez défini l’expiration d’un groupe :

- Les groupes présentant des activités utilisateur sont automatiquement renouvelés à l’approche de l’expiration.
- En l’absence de renouvellement automatique, les propriétaires du groupe sont invités à le renouveler.
- Les groupes non renouvelés sont supprimés.
- Les groupes Microsoft 365 qui sont supprimés peuvent être restaurés dans les 30 jours qui suivent par les propriétaires des groupes ou par l’administrateur.

À l’heure actuelle, il n’est possible de configurer qu’une seule stratégie d’expiration pour tous les groupes Microsoft 365 d’une organisation Azure AD.

> [!NOTE]
> La configuration et l’utilisation de la stratégie d’expiration pour les groupes Microsoft 365 vous imposent de posséder (mais pas forcément d’attribuer) des licences Azure AD Premium pour les membres de tous les groupes auxquels la stratégie d’expiration est appliquée.

Pour plus d’informations sur le téléchargement et l’installation des applets de commande Azure AD PowerShell, consultez [Azure Active Directory PowerShell for Graph 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.137).

## <a name="activity-based-automatic-renewal"></a>Renouvellement automatique en fonction de l’activité

Grâce à l’intelligence Azure AD, les groupes sont maintenant automatiquement renouvelés selon qu’ils ont été utilisés récemment ou non. Cette fonctionnalité évite aux propriétaires de groupes une action manuelle, car elle s’appuie sur l’activité des utilisateurs dans les groupes sur plusieurs services Microsoft 365, comme Outlook, SharePoint ou Teams. Par exemple, si un propriétaire ou un membre de groupe effectue une opération comme charger un document dans SharePoint, visiter un canal Teams ou envoyer un e-mail au groupe dans Outlook, le groupe est automatiquement renouvelé et le propriétaire ne reçoit aucune notification de renouvellement.

### <a name="activities-that-automatically-renew-group-expiration"></a>Activités qui renouvellent automatiquement l’expiration du groupe

Les actions utilisateur suivantes entraînent le renouvellement automatique du groupe :

- SharePoint : afficher, modifier, télécharger, déplacer, partager ou charger des fichiers
- Outlook : rejoindre un groupe, lire/écrire un message de groupe sur l’espace de groupe, liker un message (dans Outlook Web Access)
- Teams : accéder à un canal Teams

### <a name="auditing-and-reporting"></a>Audit et création de rapports

Les administrateurs trouveront la liste des groupes renouvelés automatiquement dans les journaux d’audit d’activité dans Azure AD.

![Renouvellement automatique des groupes en fonction de l’activité](./media/groups-lifecycle/audit-logs-autorenew-group.png)

## <a name="roles-and-permissions"></a>Rôles et autorisations

Voici les rôles pour lesquels vous pouvez configurer et utiliser l’expiration de groupes Microsoft 365 dans Azure AD.

Role | Autorisations
-------- | --------
Administrateur général, Administrateur de groupes ou Administrateur d’utilisateurs. | Peut créer, lire, mettre à jour ou supprimer les paramètres de stratégie d’expiration de groupes Microsoft 365<br>Peut renouveler n’importe quel groupe Microsoft 365
Utilisateur | Peut renouveler un groupe Microsoft 365 dont il est propriétaire<br>Peut restaurer un groupe Microsoft 365 dont il est propriétaire<br>Peut lire les paramètres de stratégie d’expiration

Pour plus d’informations sur les autorisations nécessaires pour restaurer un groupe supprimé, consultez [Restaurer un groupe Microsoft 365 supprimé dans Azure Active Directory](groups-restore-deleted.md).

## <a name="set-group-expiration"></a>Définir l’expiration d’un groupe

1. Ouvrez le [centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte qui est un administrateur général dans votre organisation Azure AD.

2. Sélectionnez **Groupes**, puis **Expiration**, pour ouvrir les paramètres d’expiration.
  
   ![Paramètres d’expiration pour les groupes](./media/groups-lifecycle/expiration-settings.png)

3. Sur la page **Expiration**, vous pouvez :

    - Définir la durée de vie du groupe en jours. Vous pouvez sélectionner l’une des valeurs prédéfinies ou une valeur personnalisée (30 jours ou plus).
    - Spécifier une adresse de messagerie à laquelle les notifications de renouvellement et d’expiration doivent être envoyées lorsqu’un groupe n’a pas de propriétaire.
    - Sélectionnez les groupes Microsoft 365 qui expirent. Vous pouvez définir l’expiration pour :
      - **Tous** les groupes Microsoft 365
      - Liste des groupes Microsoft 365 **sélectionnés**
      - **Aucun** pour limiter l’expiration de tous les groupes
    - Enregistrer vos paramètres lorsque vous avez terminé en sélectionnant **Enregistrer**.

> [!NOTE]
> - Lors de la première configuration de l’expiration, les groupes plus anciens que l’intervalle d’expiration bénéficient d’un délai de 35 jours avant expiration, à moins qu’ils soient renouvelés automatiquement ou renouvelés par le propriétaire.
> - Lorsqu’un groupe dynamique est supprimé et restauré, il est considéré comme un nouveau groupe et complété conformément à la règle. Ce processus peut prendre jusqu’à 24 heures.
> - Les avis d’expiration pour les groupes utilisés dans Teams s’affichent dans le flux Propriétaires Teams.

## <a name="email-notifications"></a>Notifications par e-mail

Si les groupes ne sont pas renouvelés automatiquement, des notifications par e-mail comme celle-ci sont envoyées aux propriétaires de groupes Microsoft 365 30 jours, 15 jours et 1 jour avant l’expiration du groupe. La langue de l’e-mail est déterminée par la langue par défaut du propriétaire des groupes ou par les paramètres de langue d’Azure AD. Si le propriétaire du groupe a défini une langue préférée ou si plusieurs propriétaires ont la même langue préférée, alors cette langue est utilisée. Dans tous les autres cas, les paramètres de langue d’Azure AD sont utilisés.

![Notifications par e-mail de l’expiration](./media/groups-lifecycle/expiration-notification.png)

Dans l’e-mail de notification **Renouveler le groupe**, les propriétaires de groupes peuvent accéder directement à la page d’informations du groupe dans le [volet d’accès](https://account.activedirectory.windowsazure.com/r#/applications). Dans cette page, les utilisateurs peuvent obtenir plus d’informations sur le groupe, comme sa description, la date de son dernier renouvellement, sa date d’expiration, ainsi que la possibilité de son renouvellement. La page d’informations du groupe comprend également des liens vers les ressources de groupe Microsoft 365, pour que le propriétaire du groupe puisse voir facilement le contenu et les activités dans son groupe.

Quand un groupe expire, il est supprimé un jour après la date d’expiration. Une notification par e-mail comme celle-ci est envoyée aux propriétaires de groupes Microsoft 365 pour les informer de l’expiration et de la suppression qui en découle de leur groupe Microsoft 365.

![Notifications par e-mail de la suppression du groupe](./media/groups-lifecycle/deletion-notification.png)

Le groupe peut être restauré dans les 30 jours suivant sa suppression. Pour cela, vous pouvez sélectionner **Restaurer le groupe** ou utiliser les cmdlets PowerShell, comme décrit dans [Restaurer un groupe Microsoft 365 supprimé dans Azure Active Directory](groups-restore-deleted.md). Veuillez noter que la période de restauration des groupes de 30 jours n’est pas personnalisable.

Si le groupe que vous restaurez contient des documents, des sites SharePoint ou d’autres objets persistants, la restauration du groupe et de son contenu peut nécessiter jusqu’à 24 heures.

## <a name="how-to-retrieve-microsoft-365-group-expiration-date"></a>Comment récupérer la date d’expiration du groupe Microsoft 365

En plus du Volet d’accès dans lequel les utilisateurs peuvent afficher les détails du groupe, tels que la date d’expiration et la dernière date de renouvellement, la date d’expiration d’un groupe Microsoft 365 peut être récupérée à partir de la version bêta de l’API REST Microsoft Graph. expirationDateTime a été activé en tant que propriété de groupe dans la version bêta de Microsoft Graph. Il peut être récupéré à l’aide d’une requête GET. Pour plus d’informations, consultez [cet exemple](/graph/api/group-get?view=graph-rest-beta#example).

> [!NOTE]
> Pour gérer les appartenances aux groupes dans le Volet d’accès, « Restreindre l’accès aux groupes dans le volet d’accès » doit être défini sur « Non » dans les paramètres généraux des groupes Azure Active Directory.

## <a name="how-microsoft-365-group-expiration-works-with-a-mailbox-on-legal-hold"></a>Fonctionnement de l’expiration des groupes Microsoft 365 avec une boîte aux lettres en conservation légale

Quand un groupe expire et est supprimé, les données du groupe issues des applications comme Planner, Sites ou Teams sont supprimées définitivement au bout de 30 jours, ce qui n’est pas le cas de la boîte aux lettres de groupe en conservation légale. L’administrateur peut utiliser les applets de commande Exchange pour restaurer la boîte aux lettres afin d’en extraire les données.

## <a name="how-microsoft-365-group-expiration-works-with-retention-policy"></a>Fonctionnement de l’expiration des groupes Microsoft 365 avec la stratégie de conservation

La stratégie de conservation est configurée dans le Centre de conformité et de sécurité. Si vous avez configuré une stratégie de conservation pour des groupes Microsoft 365, lorsqu’un groupe expire et est supprimé, les conversations de groupe situées dans la boîte aux lettres de groupe et les fichiers situés dans le site de groupe sont conservés dans le conteneur de conservation pendant le nombre de jours défini dans la stratégie de conservation. Les utilisateurs ne verront pas le groupe et son contenu après son expiration. Toutefois, ils peuvent récupérer les données de site et de boîte aux lettres via découverte électronique.

## <a name="powershell-examples"></a>Exemples PowerShell

Voici des exemples de la façon dont vous pouvez utiliser les applets de commande PowerShell pour configurer les paramètres d’expiration des groupes Microsoft 365 dans votre organisation Azure AD :

1. Installez le module PowerShell v2.0 et connectez-vous à l’invite de PowerShell :

   ``` PowerShell
   Install-Module -Name AzureAD
   Connect-AzureAD
   ```

1. Configurer les paramètres d’expiration Utilisez l’applet de commande New-AzureADMSGroupLifecyclePolicy pour définir la durée de vie de tous les groupes Microsoft 365 de l’organisation Azure AD sur 365 jours. Les notifications de renouvellement pour les groupes Microsoft 365 sans propriétaires sont envoyées à « emailaddress@contoso.com »
  
   ``` PowerShell
   New-AzureADMSGroupLifecyclePolicy -GroupLifetimeInDays 365 -ManagedGroupTypes All -AlternateNotificationEmails emailaddress@contoso.com
   ```

1. Récupérez la stratégie existante. Get-AzureADMSGroupLifecyclePolicy : cette applet de commande récupère les paramètres d’expiration actuels du groupe Microsoft 365 qui ont été configurés. Dans cet exemple, vous pouvez voir ce qui suit :

   - L’ID de la stratégie
   - La durée de vie de tous les groupes Microsoft 365 de l’organisation Azure AD est définie sur 365 jours
   - Les notifications de renouvellement pour les groupes Microsoft 365 sans propriétaires sont envoyées à « emailaddress@contoso.com ».
  
   ```powershell
   Get-AzureADMSGroupLifecyclePolicy
  
   ID                                    GroupLifetimeInDays ManagedGroupTypes AlternateNotificationEmails
   --                                    ------------------- ----------------- ---------------------------
   26fcc232-d1c3-4375-b68d-15c296f1f077  365                 All               emailaddress@contoso.com
   ```

1. Mettez à jour la stratégie existante. Set-AzureADMSGroupLifecyclePolicy : cette applet de commande sert à mettre à jour une stratégie existante. Dans l’exemple ci-dessous, la durée de vie du groupe dans la stratégie existante est passée de 365 jours à 180 jours.
  
   ```powershell
   Set-AzureADMSGroupLifecyclePolicy -Id "26fcc232-d1c3-4375-b68d-15c296f1f077" -GroupLifetimeInDays 180 -AlternateNotificationEmails "emailaddress@contoso.com"
   ```
  
1. Ajoutez des groupes spécifiques à la stratégie. Add-AzureADMSLifecyclePolicyGroup : cette applet de commande ajoute un groupe à la stratégie du cycle de vie. Par exemple :
  
   ```powershell
   Add-AzureADMSLifecyclePolicyGroup -Id "26fcc232-d1c3-4375-b68d-15c296f1f077" -groupId "cffd97bd-6b91-4c4e-b553-6918a320211c"
   ```
  
1. Supprimez la stratégie existante. Remove-AzureADMSGroupLifecyclePolicy : cette applet de commande supprime les paramètres d’expiration du groupe Microsoft 365, mais nécessite l’ID de stratégie. Cette cmdlet désactive l’expiration des groupes Microsoft 365.
  
   ```powershell
   Remove-AzureADMSGroupLifecyclePolicy -Id "26fcc232-d1c3-4375-b68d-15c296f1f077"
   ```
  
Les applets de commande suivantes peuvent être utilisées pour configurer la stratégie plus en détail. Pour plus d’informations, consultez la [documentation PowerShell](/powershell/module/azuread/?branch=master&view=azureadps-2.0-preview#groups).

- Get-AzureADMSGroupLifecyclePolicy
- New-AzureADMSGroupLifecyclePolicy
- Set-AzureADMSGroupLifecyclePolicy
- Remove-AzureADMSGroupLifecyclePolicy
- Add-AzureADMSLifecyclePolicyGroup
- Remove-AzureADMSLifecyclePolicyGroup
- Reset-AzureADMSLifeCycleGroup
- Get-AzureADMSLifecyclePolicyGroup

## <a name="next-steps"></a>Étapes suivantes

Ces articles fournissent des informations supplémentaires sur les groupes Azure AD.

- [Consulter les groupes existants](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Gérer les paramètres d’un groupe](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Gérer les membres d’un groupe](../fundamentals/active-directory-groups-members-azure-portal.md)
- [Gérer l’appartenance à un groupe](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Gérer les règles dynamiques pour les utilisateurs dans un groupe](groups-dynamic-membership.md)