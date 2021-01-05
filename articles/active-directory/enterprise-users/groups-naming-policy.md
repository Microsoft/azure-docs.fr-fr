---
title: Appliquer une stratégie d’attribution de noms de groupe dans Azure Active Directory | Microsoft Docs
description: Comment configurer une stratégie d’attribution de noms pour les groupes Microsoft 365 dans Azure Active Directory
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
ms.custom: it-pro;seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: a4c1c30e4651136ada7264cad8ff4d3921c3aeeb
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96547914"
---
# <a name="enforce-a-naming-policy-on-microsoft-365-groups-in-azure-active-directory"></a>Appliquer une stratégie d’attribution de noms à des groupes Microsoft 365 dans Azure Active Directory

Pour appliquer des conventions d’affectation de noms cohérentes pour des groupes Microsoft 365 créés ou modifiés par vos utilisateurs, configurez une stratégie d’affectation de noms de groupe pour vos organisations dans Azure Active Directory (Azure AD). Par exemple, vous pouvez utiliser la stratégie de nommage pour communiquer la fonction d’un groupe, l’appartenance, la région géographique ou le nom de la personne qui a créé le groupe. Vous pouvez aussi utiliser la stratégie de nommage pour faciliter le classement des groupes dans le carnet d’adresses. Vous pouvez utiliser la stratégie pour empêcher l’utilisation de certains mots dans les noms et les alias de groupes.

> [!IMPORTANT]
> Pour pouvoir utiliser la stratégie d’attribution de noms Azure AD pour des groupes Microsoft 365, vous devez posséder, mais pas nécessairement attribuer, une licence Azure Active Directory Premium P1 ou Azure AD Basic EDU pour chaque utilisateur membre d’un ou plusieurs groupes Microsoft 365.

La stratégie de nommage s’applique à la création ou à la modification des groupes créés dans toutes les charges de travail (par exemple, Outlook, Microsoft Teams, SharePoint, Exchange ou Planner). Elle s’applique à la fois au nom de groupe et à l’alias de groupe. Si vous avez configuré votre stratégie d’attribution de noms dans Azure AD et qu’il existe une stratégie d’attribution de noms de groupes Exchange, la stratégie d’attribution de noms Azure AD est appliquée dans votre organisation.

Quand une stratégie d’attribution de noms est configurée, elle est appliquée aux nouveaux groupes Microsoft 365 créés par des utilisateurs finaux. La stratégie d’attribution de nom ne s’applique pas à certains rôles d’annuaire, tels que l’administrateur général ou l’administrateur d’utilisateurs (voir ci-dessous pour obtenir la liste complète des rôles exemptés de la stratégie d’attribution de nom à un groupe). Pour les groupes Microsoft 365 existants, la stratégie ne s’applique pas immédiatement au moment de la configuration. Une fois que le propriétaire du groupe modifie le nom de ces groupes, la stratégie d’attribution de nom est appliquée.

## <a name="naming-policy-features"></a>Caractéristiques de la stratégie de nommage

Vous pouvez appliquer une stratégie d’attribution de noms pour des groupes de deux manières différentes :

- **Stratégie de nommage avec préfixe-suffixe** : vous pouvez définir des préfixes ou des suffixes qui sont par la suite ajoutés automatiquement pour appliquer une convention de nommage à vos groupes (par exemple, dans le nom de groupe « GRP\_JAPON\_Mon groupe\_Ingénierie », GRP\_JAPON\_ est le préfixe et \_Ingénierie est le suffixe). 

- **Mots bloqués personnalisés** : vous pouvez charger un ensemble de mots bloqués propres à votre organisation pour interdire leur utilisation dans les groupes créés par les utilisateurs (par exemple, « PDG, Paie, RH »).

### <a name="prefix-suffix-naming-policy"></a>Stratégie de nommage avec préfixe-suffixe

La structure générale de la convention de nommage est « Préfixe[NomGroupe]Suffixe ». Même s’il ne vous est pas interdit de définir plusieurs préfixes et suffixes, seule une instance de [NomGroupe] doit figurer dans le paramètre. Les préfixes ou les suffixes peuvent être des chaînes fixes ou des attributs utilisateur tels que \[Service\] qui sont substitués en fonction de l’utilisateur qui crée le groupe. Le nombre total de caractères autorisés pour les chaînes de préfixe et de suffixe, nom du groupe compris, est de 53 caractères. 

Les préfixes et les suffixes peuvent contenir les caractères spéciaux pris en charge dans un nom de groupe et un alias de groupe. Les caractères présents dans le préfixe ou le suffixe qui ne sont pas pris en charge dans l’alias de groupe sont quand même appliqués dans le nom de groupe, mais ils sont supprimés de l’alias de groupe. Compte tenu de cette restriction, les préfixes et les suffixes appliqués au nom de groupe peuvent être différents de ceux appliqués à l’alias de groupe. 

#### <a name="fixed-strings"></a>Chaînes fixes

Vous pouvez utiliser des chaînes pour faciliter l’analyse et la différenciation des groupes dans la liste d’adresses globale et dans les liens de navigation de gauche des charges de travail de groupe. Certains préfixes courants sont des mots clés tels que « Nom\_Grp », « \#Nom », «\_Nom »

#### <a name="user-attributes"></a>Attributs utilisateur

Vous pouvez utiliser des attributs pour vous aider, à vous et vos utilisateurs, à identifier le service, le bureau ou la région géographique pour lequel/laquelle le groupe a été créé. Par exemple, si vous définissez la stratégie de nommage `PrefixSuffixNamingRequirement = "GRP [GroupName] [Department]"` et `User’s department = Engineering`, vous pouvez obtenir le nom de groupe appliqué « GRP Mon groupe Ingénierie ». Les attributs Azure AD pris en charge sont \[Department\], \[Company\], \[Office\], \[StateOrProvince\], \[CountryOrRegion \], \[Title\]. Les attributs utilisateur non pris en charge sont traités comme des chaînes fixes ; par exemple, « \[postalCode\] ». Les attributs d’extension et les attributs personnalisés ne sont pas pris en charge.

Nous vous recommandons d’utiliser des attributs dont les valeurs sont remplies pour tous les utilisateurs de votre organisation et évitez d’utiliser des attributs ayant des valeurs de type Long.

### <a name="custom-blocked-words"></a>Mots bloqués personnalisés

Une liste de mots bloqués est une liste d’expressions séparées par des virgules dont l’utilisation est interdite dans les noms et alias de groupe. Aucune recherche de sous-chaîne n’est effectuée. Une correspondance exacte entre le nom de groupe et un ou plusieurs mots bloqués personnalisés est nécessaire pour déclencher un échec. Sachant qu’aucune recherche de sous-chaîne n’est effectuée, les utilisateurs peuvent utiliser des mots courants tels que « Classe » même si « lasse » est un mot bloqué.

Règles de liste de mots bloqués :

- Les mots bloqués ne sont pas sensibles à la casse.
- Quand un utilisateur entre un mot bloqué dans un nom de groupe, un message d’erreur s’affiche avec le mot bloqué.
- Les mots bloqués ne sont soumis à aucune restriction de caractères.
- Le nombre d’expressions pouvant être configurées dans la liste de mots bloqués est limité à 5 000. 

### <a name="roles-and-permissions"></a>Rôles et autorisations

Pour configurer la stratégie de nommage, l’un des rôles suivants est nécessaire :
- Administrateur général
- Administrateur de groupe


Certains administrateurs peuvent être exemptés de ces stratégies dans toutes les charges de travail et tous les points de terminaison de groupe. Ils peuvent ainsi créer des groupes en utilisant des mots bloqués et avec leurs propres conventions de nommage. Les rôles d’administrateur exemptés de la stratégie de nommage de groupes sont indiqués ci-dessous.

- Administrateur général
- Support de niveau 1 partenaire
- Support de niveau 2 partenaire
- Administrateur d’utilisateurs

## <a name="configure-naming-policy-in-azure-portal"></a>Configurer une stratégie d’attribution de noms dans le portail Azure

1. Connectez-vous au [centre d’administration Azure AD](https://aad.portal.azure.com) avec un compte d’administrateur de groupe.
1. Sélectionnez **Groupes**, puis **Stratégie d'attribution de noms** pour ouvrir la page Stratégie d'attribution de noms.

    ![Ouvrez la page Stratégie d'attribution de noms dans le Centre d'administration](./media/groups-naming-policy/policy.png)

### <a name="view-or-edit-the-prefix-suffix-naming-policy"></a>Afficher ou modifier la stratégie d’attribution de suffixes/préfixes

1. Sur la page **Stratégie d'attribution de noms**, sélectionnez **Stratégie de noms de groupes**.
1. Vous pouvez afficher ou modifier individuellement les stratégies actuelles d'attribution de suffixes/préfixes en sélectionnant les attributs ou les chaînes que vous souhaitez appliquer dans le cadre de la stratégie d'attribution de noms.
1. Pour supprimer un préfixe ou un suffixe dans la liste, sélectionnez-le et choisissez **Supprimer**. Plusieurs éléments peuvent être supprimés en même temps.
1. Enregistrez les modifications de la nouvelle stratégie pour qu’elle prenne effet en sélectionnant **Enregistrer**.

### <a name="edit-custom-blocked-words"></a>Modifier les mots bloqués personnalisés

1. Sur la page **Stratégie d'attribution de noms**, sélectionnez **Mots bloqués**.

    ![Modifier et charger la liste de mots bloqués pour la stratégie d'attribution de noms](./media/groups-naming-policy/blockedwords.png)

1. Affichez ou modifiez la liste actuelle des mots bloqués personnalisés en sélectionnant **Télécharger**.
1. Chargez la nouvelle liste de mots bloqués personnalisés en sélectionnant l'icône de fichier.
1. Enregistrez les modifications de la nouvelle stratégie pour qu’elle prenne effet en sélectionnant **Enregistrer**.

## <a name="install-powershell-cmdlets"></a>Installer les applets de commande PowerShell

Veillez à désinstaller toute version ancienne du module Azure Active Directory PowerShell for Graph pour Windows PowerShell et à installer [Azure Active Directory PowerShell for Graph - Public Preview Release 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.137) avant d’exécuter les commandes PowerShell.

1. Ouvrez l’application Windows PowerShell en tant qu’administrateur.
2. Désinstallez toute version précédente d’AzureADPreview.
  
   ``` PowerShell
   Uninstall-Module AzureADPreview
   ```

3. Installez la dernière version d’AzureADPreview.
  
   ``` PowerShell
   Install-Module AzureADPreview
   ```

   Si vous êtes invité à accéder à un référentiel non approuvé, entrez **Y**. L’installation du nouveau module peut prendre quelques minutes.

## <a name="configure-naming-policy-in-powershell"></a>Configurer une stratégie d’attribution de noms dans PowerShell

1. Ouvrez une fenêtre Windows PowerShell sur votre ordinateur. Vous pouvez l’ouvrir sans privilèges élevés.

1. Exécutez les commandes suivantes pour préparer l’exécution des applets de commande.
  
   ``` PowerShell
   Import-Module AzureADPreview
   Connect-AzureAD
   ```

   Dans l’écran **Connectez-vous à votre compte** qui s’ouvre, entrez votre compte d’administrateur et votre mot de passe pour vous connecter à votre service, puis sélectionnez **Se connecter**.

1. Si vous souhaitez créer des paramètres de groupe pour cette organisation, suivez les étapes fournies dans [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md).

### <a name="view-the-current-settings"></a>Afficher les paramètres actuels

1. Récupérez la stratégie de nommage active pour afficher les paramètres actuels.
  
   ``` PowerShell
   $Setting = Get-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id
   ```
  
1. Affichez les paramètres de groupe actuels.
  
   ``` PowerShell
   $Setting.Values
   ```
  
### <a name="set-the-naming-policy-and-custom-blocked-words"></a>Définir la stratégie de nommage et les mots bloqués personnalisés

1. Définissez les préfixes et les suffixes de nom de groupe dans Azure AD PowerShell. Pour que la fonctionnalité fonctionne correctement, [GroupName] doit être inclus dans le paramètre.
  
   ``` PowerShell
   $Setting["PrefixSuffixNamingRequirement"] =“GRP_[GroupName]_[Department]"
   ```
  
1. Définissez les mots bloqués personnalisés sur lesquels vous voulez imposer des restrictions. L’exemple suivant illustre la façon dont vous pouvez ajouter vos propres mots personnalisés.
  
   ``` PowerShell
   $Setting["CustomBlockedWordsList"]=“Payroll,CEO,HR"
   ```
  
1. Enregistrez les paramètres de la nouvelle stratégie pour qu’elle prenne effet, comme dans l’exemple suivant.
  
   ``` PowerShell
   Set-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id -DirectorySetting $Setting
   ```
  
Vous avez terminé. Vous avez défini votre stratégie de nommage et ajouté vos mots bloqués.

## <a name="export-or-import-custom-blocked-words"></a>Exporter ou importer des mots bloqués personnalisés

Pour plus d’informations, consultez l’article [Configuration des paramètres de groupe avec les applets de commande Azure Active Directory](../enterprise-users/groups-settings-cmdlets.md).

Voici un exemple de script PowerShell permettant d’exporter plusieurs mots bloqués :

``` PowerShell
$Words = (Get-AzureADDirectorySetting).Values | Where-Object -Property Name -Value CustomBlockedWordsList -EQ 
Add-Content "c:\work\currentblockedwordslist.txt" -Value $words.value.Split(",").Replace("`"","")  
```

Voici un exemple de script PowerShell permettant d’importer plusieurs mots bloqués :

``` PowerShell
$BadWords = Get-Content "C:\work\currentblockedwordslist.txt"
$BadWords = [string]::join(",", $BadWords)
$Settings = Get-AzureADDirectorySetting | Where-Object {$_.DisplayName -eq "Group.Unified"}
if ($Settings.Count -eq 0)
    {$Template = Get-AzureADDirectorySettingTemplate | Where-Object {$_.DisplayName -eq "Group.Unified"}
    $Settings = $Template.CreateDirectorySetting()
    New-AzureADDirectorySetting -DirectorySetting $Settings
    $Settings = Get-AzureADDirectorySetting | Where-Object {$_.DisplayName -eq "Group.Unified"}}
$Settings["CustomBlockedWordsList"] = $BadWords
Set-AzureADDirectorySetting -Id $Settings.Id -DirectorySetting $Settings 
```

## <a name="remove-the-naming-policy"></a>Supprimer la stratégie de noms de groupes

### <a name="remove-the-naming-policy-using-azure-portal"></a>Supprimer la stratégie d’attribution de noms à l’aide du portail Azure

1. Sur la page **Stratégie d'attribution de noms**, sélectionnez **Supprimer une stratégie**.
1. Une fois la suppression confirmée, la stratégie d'attribution de noms est supprimée, de même que toutes les stratégies d'attribution de suffixes/préfixes et tous les mots bloqués personnalisés.

### <a name="remove-the-naming-policy-using-azure-ad-powershell"></a>Supprimer la stratégie d’attribution de noms à l’aide d’Azure AD PowerShell

1. Supprimez les préfixes et les suffixes de nom de groupe dans Azure AD PowerShell.
  
   ``` PowerShell
   $Setting["PrefixSuffixNamingRequirement"] =""
   ```
  
1. Supprimez les mots bloqués personnalisés.
  
   ``` PowerShell
   $Setting["CustomBlockedWordsList"]=""
   ```
  
1. Enregistrez les paramètres.
  
   ``` PowerShell
   Set-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id -DirectorySetting $Setting
   ```

## <a name="experience-across-microsoft-365-apps"></a>Expérience sur les applications Microsoft 365

Une fois qu’une stratégie de nommage de groupes a été définie dans Azure AD, voici ce que voit un utilisateur quand il crée un groupe dans une application Microsoft 365 :

- Un aperçu du nom conformément à votre stratégie de nommage (avec préfixes et suffixes) à mesure que l’utilisateur tape le nom du groupe.
- Si l’utilisateur entre des mots bloqués, un message d’erreur s’affiche pour qu’il supprime ces mots.

Charge de travail | Conformité
----------- | -------------------------------
Portails Azure Active Directory | Le portail Azure AD et le portail Panneau d’accès affichent le nom appliqué de la stratégie de nommage quand l’utilisateur tape un nom de groupe pour créer ou modifier un groupe. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche avec le mot bloqué pour qu’il puisse le supprimer.
Outlook Web Access (OWA) | Outlook Web Access affiche le nom appliqué de la stratégie de nommage quand l’utilisateur tape un nom ou alias de groupe. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche dans l’interface utilisateur avec le mot bloqué pour qu’il puisse le supprimer.
Bureau Outlook | Les groupes créés dans le bureau Outlook sont conformes aux paramètres de stratégie de nommage. L’application de bureau Outlook n’affiche pas encore l’aperçu du nom de groupe appliqué et ne retourne pas d’erreurs liées aux mots bloqués personnalisés au moment où l’utilisateur entre le nom du groupe. Cependant, la stratégie de nommage est appliquée automatiquement pendant la création ou la modification d’un groupe, et les utilisateurs obtiennent des messages d’erreur si le nom ou l’alias de groupe contient des mots bloqués personnalisés.
Microsoft Teams | Microsoft Teams présente le nom appliqué de la stratégie de nommage de groupes quand l’utilisateur entre un nom d’équipe. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche avec le mot bloqué pour qu’il puisse le supprimer.
SharePoint  |  SharePoint affiche le nom appliqué de la stratégie de nommage quand l’utilisateur tape un nom de site ou une adresse e-mail de groupe. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche avec le mot bloqué pour qu’il puisse le supprimer.
Microsoft Stream | Microsoft Stream affiche le nom appliqué de la stratégie de nommage de groupes quand l’utilisateur tape un nom de groupe ou un alias d’e-mail de groupe. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche avec le mot bloqué pour qu’il puisse le supprimer.
Application Outlook pour iOS et Android | Les groupes créés dans les applications Outlook sont conformes à la stratégie de nommage configurée. L’application mobile Outlook n’affiche pas encore l’aperçu du nom appliqué de la stratégie de nommage et ne retourne pas d’erreurs liées aux mots bloqués personnalisés au moment où l’utilisateur entre le nom du groupe. Cependant, la stratégie de nommage est appliquée automatiquement quand l’utilisateur clique sur Créer ou Modifier, et des messages d’erreur s’affichent en présence de mots bloqués personnalisés dans le nom ou l’alias de groupe.
Application mobile Groups | Les groupes créés dans l’application mobile Groups sont conformes à la stratégie de nommage. L’application mobile Groups n’affiche pas l’aperçu de la stratégie de nommage et ne retourne pas d’erreurs liées aux mots bloqués personnalisés au moment où l’utilisateur entre le nom du groupe. En revanche, la stratégie de nommage est appliquée automatiquement pendant la création ou la modification d’un groupe, et les erreurs appropriées sont présentées aux utilisateurs si le nom ou l’alias de groupe contient des mots bloqués personnalisés.
Planner | Planner est conforme à la stratégie de nommage. Planner affiche un aperçu de la stratégie de nommage au moment où le nom du plan est entré. Quand un utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche pendant la création du plan.
Dynamics 365 for Customer Engagement | Dynamics 365 for Customer Engagement est conforme à la stratégie de nommage. Dynamics 365 affiche le nom appliqué de la stratégie d’appellation quand l’utilisateur tape un nom de groupe ou un alias d’e-mail de groupe. Quand l’utilisateur entre un mot bloqué personnalisé, un message d’erreur s’affiche avec le mot bloqué pour qu’il puisse le supprimer.
School Data Sync (SDS) | Les groupes créés via SDS sont conformes à la stratégie de nommage, mais celle-ci ne s’applique pas automatiquement. Les administrateurs SDS doivent ajouter les préfixes et les suffixes aux noms de classes pour lesquels des groupes doivent être créés puis chargés dans SDS. À défaut, la création ou la modification de groupe échoue.
Application Classroom | Les groupes créés dans l’application Classroom sont conformes à la stratégie de nommage, mais celle-ci ne s’applique pas automatiquement et l’aperçu de la stratégie de nommage n’est pas présenté aux utilisateurs quand ils entrent un nom de groupe de classe. Les utilisateurs doivent entrer le nom de groupe de classe appliqué avec les préfixes et les suffixes. Dans le cas contraire, l’opération de création ou de modification du groupe de classe échoue avec des erreurs.
Power BI | Les espaces de travail Power BI sont conformes à la stratégie de nommage.    
Yammer | Lorsqu’un utilisateur connecté à Yammer avec son compte Azure Active Directory crée un groupe ou modifie un nom de groupe, le nom de groupe doit respecter la stratégie d’affectation de noms. Cela s’applique à la fois aux groupes connectés à Microsoft 365 et à tous les autres groupes Yammer.<br>Si un groupe connecté à Microsoft 365 a été créé avant que la stratégie de nommage ne soit en place, le nom de groupe ne suit pas automatiquement les stratégies de nommage. Lorsqu’un utilisateur modifie le nom de groupe, il est invité à ajouter le préfixe et le suffixe.
StaffHub  | Les équipes StaffHub ne respectent pas la stratégie d’affectation de noms, contrairement au groupe Microsoft 365 sous-jacent. Le nom de l’équipe StaffHub n’applique pas les préfixes et les suffixes et ne recherche pas les mots bloqués personnalisés. En revanche, StaffHub applique les préfixes et suffixes, et supprime les mots bloqués du groupe Microsoft 365 sous-jacent.
Exchange PowerShell | Les applets de commande Exchange PowerShell sont conformes à la stratégie de nommage. Les utilisateurs reçoivent des messages d’erreur appropriés avec les préfixes et les suffixes suggérés et pour les mots bloqués personnalisés s’ils ne suivent pas la stratégie de nommage dans le nom de groupe et l’alias de groupe (mailNickname).
Applets de commande Azure Active Directory PowerShell | Les applets de commande Azure Active Directory PowerShell sont conformes à la stratégie de nommage. Les utilisateurs reçoivent des messages d’erreur appropriés avec les préfixes et les suffixes suggérés et pour les mots bloqués personnalisés s’ils ne suivent pas la convention de nommage dans les noms de groupe et les alias de groupe.
Centre d’administration Exchange | Le centre d’administration Exchange est conforme à la stratégie de nommage. Les utilisateurs reçoivent des messages d’erreur appropriés avec les préfixes et les suffixes suggérés et pour les mots bloqués personnalisés s’ils ne suivent pas la convention de nommage dans le noms de groupe et l’alias de groupe.
Centre d’administration Microsoft 365 | Le centre d’administration Microsoft 365 est conforme à la stratégie d’attribution de noms. Quand un utilisateur crée ou modifie des noms de groupes, la stratégie de nommage s’applique automatiquement, et les utilisateurs reçoivent les erreurs appropriées quand ils entrent des mots bloqués personnalisés. Le centre d’administration Microsoft 365 n’affiche pas encore un aperçu de la stratégie d’attribution de noms et ne renvoie pas d’erreurs liées aux mots bloqués personnalisés au moment où l’utilisateur entre le nom du groupe.

## <a name="next-steps"></a>Étapes suivantes

Ces articles fournissent des informations supplémentaires sur les groupes Azure AD.

- [Consulter les groupes existants](../fundamentals/active-directory-groups-view-azure-portal.md)
- [Stratégie d’expiration pour les groupes Microsoft 365](groups-lifecycle.md)
- [Gérer les paramètres d’un groupe](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Gérer les membres d’un groupe](../fundamentals/active-directory-groups-members-azure-portal.md)
- [Gérer l’appartenance à un groupe](../fundamentals/active-directory-groups-membership-azure-portal.md)
- [Gérer les règles dynamiques pour les utilisateurs dans un groupe](groups-dynamic-membership.md)
