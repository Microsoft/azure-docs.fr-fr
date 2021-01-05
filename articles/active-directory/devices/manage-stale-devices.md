---
title: Guide pratique pour gérer les appareils obsolètes dans Azure AD | Microsoft Docs
description: Découvrez comment supprimer des appareils obsolètes de votre base de données d’appareils inscrits dans Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: d12679e64d690614aaf788837a02af007448f83d
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93393674"
---
# <a name="how-to-manage-stale-devices-in-azure-ad"></a>Procédure : Gérer les appareils obsolètes dans Azure AD

Dans l’idéal, pour compléter le cycle de vie, vous devez annuler l’inscription des appareils dont vous n’avez plus besoin. Toutefois, lorsque des appareils sont perdus, volés ou défectueux par exemple, ou lorsque des réinstallations de système d’exploitation sont nécessaires, votre environnement inclut généralement des appareils obsolètes. En tant qu’administrateur informatique, vous avez probablement besoin d’une méthode pour supprimer les appareils obsolètes et ainsi pouvoir consacrer vos ressources à la gestion des appareils qui en ont réellement besoin.

Cet article vous apprend à gérer efficacement les appareils obsolètes présents dans votre environnement.
  

## <a name="what-is-a-stale-device"></a>Qu’est qu’un appareil obsolète ?

Un appareil obsolète est un appareil qui a été inscrit auprès d’Azure AD, mais n’a pas été utilisé pour accéder à des applications cloud depuis un certain temps. Les appareils obsolètes ont un impact sur votre capacité à gérer et prendre en charge vos utilisateurs et appareils associés au locataire, et ce, pour les raisons suivantes : 

- Les appareils dupliqués peuvent rendre compliquée l’identification de l’appareil actuellement actif par le personnel du support technique.
- Le nombre accru d’appareils entraîne des réécritures d’appareil inutiles qui allongent la durée des synchronisations Azure AD Connect.
- À des fins de maintenance générale et de conformité, vous souhaitez peut-être disposer d’appareils présentant un état propre. 

Les appareils obsolètes dans Azure AD peuvent interférer avec les stratégies générales de cycle de vie des appareils de votre organisation.

## <a name="detect-stale-devices"></a>Détecter les appareils obsolètes

Un appareil obsolète se définit comme un appareil inscrit qui n’a pas été utilisé pour accéder à des applications cloud depuis un certain temps. Par conséquent, la détection des appareils obsolètes nécessite une propriété de timestamp. Dans Azure AD, cette propriété est appelée **ApproximateLastLogonTimestamp** ou **timestamp d’activité**. Si la différence entre le moment présent et la valeur du **timestamp d’activité** dépasse la plage de temps que vous avez définie pour les appareils actifs, l’appareil est considéré comme obsolète. Ce **timestamp d’activité** est actuellement disponible en préversion publique.

## <a name="how-is-the-value-of-the-activity-timestamp-managed"></a>Comment la valeur du timestamp d’activité est-elle gérée ?  

L’évaluation du timestamp d’activité est déclenchée par la tentative d’authentification d’un appareil. Azure AD évalue le timestamp d’activité dans les cas suivants :

- Une stratégie d’accès conditionnel exigeant des [appareils gérés](../conditional-access/require-managed-devices.md) ou des [applications clientes approuvées](../conditional-access/app-based-conditional-access.md) a été déclenchée.
- Des appareils Windows 10 joints à Azure AD ou à une version hybride d’Azure AD sont actifs sur le réseau. 
- Des appareils gérés par Intune ont fait l’objet d’un archivage dans le service.

Si la différence entre la valeur existante du timestamp d’activité et la valeur actuelle est supérieure à 14 jours (écart de +/-5 jours), la valeur existante est remplacée par la nouvelle valeur.

## <a name="how-do-i-get-the-activity-timestamp"></a>Comment faire pour obtenir le timestamp d’activité ?

Vous pouvez récupérer la valeur du timestamp d’activité de deux façons :

- Colonne **Activité** de la [page des appareils](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices) dans le Portail Azure

    :::image type="content" source="./media/manage-stale-devices/01.png" alt-text="Capture d’écran d’une page du portail Azure répertoriant le nom, le propriétaire et d’autres informations sur les appareils. Une colonne répertorie le timestamp de l’activité." border="false":::

- Cmdlet [Get-AzureADDevice](/powershell/module/azuread/Get-AzureADDevice)

    :::image type="content" source="./media/manage-stale-devices/02.png" alt-text="Capture d’écran montrant la sortie de la ligne de commande. Une ligne est mise en évidence et répertorie un timestamp pour la valeur ApproximateLastLogonTimeStamp." border="false":::

## <a name="plan-the-cleanup-of-your-stale-devices"></a>Planifier le nettoyage de vos appareils obsolètes

Pour nettoyer efficacement les appareils obsolètes présents dans votre environnement, vous devez définir une stratégie associée. Cette stratégie vous aide à vous assurer que vous capturez toutes les considérations relatives aux appareils obsolètes. Les sections suivantes offrent des exemples de considérations généralement prises en compte dans une stratégie. 

### <a name="cleanup-account"></a>Compte de nettoyage

Pour mettre à jour un appareil dans Azure AD, vous avez besoin d’un compte auquel est assigné l’un des rôles suivants :

- Administrateur général
- Administrateur d’appareil cloud
- Administrateur de services Intune

Dans votre stratégie de nettoyage, sélectionnez des comptes auxquels les rôles requis sont assignés. 

### <a name="timeframe"></a>Délai d’exécution

Définissez une plage de temps qui servira d’indicateur pour un appareil obsolète. Quand vous définissez votre plage de temps, considérez la fenêtre indiquée pour mettre à jour le timestamp d’activité dans votre valeur. Par exemple, n’utilisez pas un timestamp inférieur à 21 jours (écart compris) comme indicateur pour un appareil obsolète. Dans certains scénarios, un appareil peut apparaître comme obsolète alors qu’il ne l’est pas. Par exemple, le propriétaire de l’appareil concerné peut être en vacances ou en arrêt maladie.  La plage de temps que vous avez définie pour les appareils obsolètes peut alors être dépassée.

### <a name="disable-devices"></a>Désactivation d’appareils

Il n’est pas recommandé de supprimer immédiatement un appareil apparaissant comme obsolète. En effet, en cas de faux positif, vous ne pouvez pas annuler la suppression. La meilleure pratique consiste à désactiver l’appareil pour une période de grâce avant de le supprimer. Dans votre stratégie, définissez une plage de temps pour désactiver un appareil avant de le supprimer.

### <a name="mdm-controlled-devices"></a>Appareils contrôlés par une solution GPM

Si votre appareil est contrôlé par Intune ou toute autre solution GPM, retirez-le du système de gestion avant de le désactiver ou de le supprimer.

### <a name="system-managed-devices"></a>Appareils gérés par le système

Ne supprimez pas des appareils gérés par le système. Il s’agit souvent d’appareils comme Autopilot. Une fois supprimés, ces appareils ne peuvent pas être reprovisionnés. Par défaut, la nouvelle cmdlet `Get-AzureADDevice` exclut les appareils gérés par le système. 

### <a name="hybrid-azure-ad-joined-devices"></a>Appareils joints Azure AD hybrides

Vos appareils joints à une version hybride d’Azure AD doivent respecter vos stratégies en matière de gestion des appareils obsolètes locaux. 

Pour nettoyer l’environnement Azure AD :

- **Appareils Windows 10** : désactivez ou supprimez les appareils Windows 10 dans votre environnement AD local, et laissez Azure AD Connect synchroniser l’état modifié des appareils sur Azure AD.
- **Windows 7/8**  : commencez par désactiver ou supprimer les appareils Windows 7/8 dans votre service AD local. Vous ne pouvez pas utiliser Azure AD Connect pour désactiver ou supprimer des appareils Windows 7/8 dans Azure AD. Au lieu de cela, quand vous apportez la modification à votre service local, vous devez désactiver/supprimer les appareils dans Azure AD.

> [!NOTE]
>* La suppression d’appareils dans votre service AD local ou Azure AD ne supprime pas l’inscription sur le client. Elle empêche uniquement d’accéder aux ressources en utilisant l’appareil en tant qu’identité (accès conditionnel, par exemple). Pour plus d’informations, reportez-vous à la section sur la [suppression de l’inscription sur le client](faq.md#hybrid-azure-ad-join-faq).
>* La suppression d’un appareil Windows 10 uniquement dans Azure AD resynchronise l’appareil à partir de votre service local à l’aide d’Azure AD Connect, mais en tant que nouvel objet avec l’état « En attente ». Une nouvelle inscription est nécessaire sur l’appareil.
>* La suppression d’appareils Windows 10/Server 2016 de l’étendue de synchronisation supprime l’appareil Azure AD. Si vous l’ajoutez de nouveau à l’étendue de synchronisation, un nouvel objet est placé avec l’état « En attente ». Une réinscription de l’appareil est nécessaire.
>* Si vous n’utilisez pas Azure AD Connect pour la synchronisation des appareils Windows 10 (par exemple, en utilisant uniquement AD FS pour l’inscription), vous devez gérer le cycle de vie de la même façon que pour les appareils Windows 7/8.


### <a name="azure-ad-joined-devices"></a>Appareils joints Azure AD

Désactivez ou supprimez les appareils joints à Azure AD dans Azure AD.

> [!NOTE]
>* La suppression d’un appareil Azure AD ne supprime pas l’inscription sur le client. Elle empêche uniquement d’accéder aux ressources en utilisant l’appareil en tant qu’identité (accès conditionnel, par exemple). 
>* Pour plus d’informations, reportez-vous à la section sur la [disjonction d’un appareil sur Azure AD](faq.md#azure-ad-join-faq). 

### <a name="azure-ad-registered-devices"></a>Appareils inscrits sur Azure AD

Désactivez ou supprimez les appareils inscrits à Azure AD dans Azure AD.

> [!NOTE]
>* La suppression d’un appareil inscrit à Azure AD dans Azure AD ne supprime pas l’inscription sur le client. Elle empêche uniquement d’accéder aux ressources en utilisant l’appareil en tant qu’identité (accès conditionnel, par exemple).
>* Pour plus d’informations, reportez-vous à la section sur la [suppression d’une inscription sur le client](faq.md#azure-ad-register-faq).

## <a name="clean-up-stale-devices-in-the-azure-portal"></a>Nettoyer les appareils obsolètes dans le Portail Azure  

Vous pouvez nettoyer les appareils obsolètes dans le Portail Azure. Cependant, il est plus efficace d’effectuer cette tâche à l’aide d’un script PowerShell. Utilisez le dernier module PowerShell V1 pour utiliser le filtre d’horodatage et filtrer les appareils gérés par le système comme Autopilot. À ce stade, l’utilisation de PowerShell V2 est déconseillée.

La procédure classique se déroule comme suit :

1. Connectez-vous à Azure Active Directory à l’aide de la cmdlet [Connect-AzureAD](/powershell/module/azuread/connect-azuread).
1. Obtenir la liste des appareils
1. Désactivez l’appareil à l’aide de la cmdlet [Set-AzureADDevice](/powershell/module/azuread/Set-AzureADDevice) (désactivez à l’aide de l’option -AccountEnabled). 
1. Attendez que la période de grâce du nombre de jours que vous choisissez soit passée avant de supprimer l’appareil.
1. Supprimez l’appareil à l’aide de la cmdlet [Remove-AzureADDevice](/powershell/module/azuread/Remove-AzureADDevice).

### <a name="get-the-list-of-devices"></a>Obtenir la liste des appareils

Pour obtenir la liste complète des appareils et stocker les données retournées dans un fichier CSV :

```PowerShell
Get-AzureADDevice -All:$true | select-object -Property Enabled, DeviceId, DisplayName, DeviceTrustType, ApproximateLastLogonTimestamp | export-csv devicelist-summary.csv
```

Si votre annuaire comporte un grand nombre d’appareils, utilisez le filtre de timestamp pour limiter le nombre d’appareils retournés. Pour obtenir la liste complète des appareils dont le timestamp est plus ancien qu’une date spécifique et stocker les données retournées dans un fichier CSV : 

```PowerShell
$dt = [datetime]’2017/01/01’
Get-AzureADDevice -All:$true | Where {$_.ApproximateLastLogonTimeStamp -le $dt} | select-object -Property Enabled, DeviceId, DisplayName, DeviceTrustType, ApproximateLastLogonTimestamp | export-csv devicelist-olderthan-Jan-1-2017-summary.csv
```

## <a name="what-you-should-know"></a>Ce que vous devez savoir

### <a name="why-is-the-timestamp-not-updated-more-frequently"></a>Pourquoi le timestamp n’est-il pas mis à jour plus fréquemment ?

La mise à jour du timestamp vise à simplifier les scénarios de cycle de vie des appareils. Il ne s’agit pas d’un audit. Utilisez les journaux d’audit de connexion pour les mises à jour plus fréquentes sur l’appareil.

### <a name="why-should-i-worry-about-my-bitlocker-keys"></a>Pourquoi dois-je me soucier de mes clés BitLocker ?

Lorsque vous configurez des clés BitLocker pour des appareils Windows 10, elles sont stockées sur l’objet appareil dans Azure AD. Si vous supprimez un appareil obsolète, vous supprimez également les clés BitLocker qui sont stockées sur l’appareil. Vous devez déterminer si votre stratégie de nettoyage correspond au cycle de vie réel de votre appareil avant de supprimer un appareil obsolète. 

### <a name="why-should-i-worry-about-windows-autopilot-devices"></a>Pourquoi dois-je surveiller de près l’utilisation des appareils Windows AutoPilot ?

Quand vous supprimez un appareil Azure AD associé à un objet Windows AutoPilot, les trois scénarios suivants peuvent se produire en cas de réaffectation de l’appareil :
- Dans le cadre d’un déploiement de Windows AutoPilot piloté par l’utilisateur sans pré-provisionnement, un appareil Azure AD est créé, sans que ce dernier soit identifié par un ZTDID.
- Dans le cadre d’un déploiement en mode automatique de Windows AutoPilot, la réaffectation échouera, car le système ne pourra pas trouver l’appareil Azure AD associé.  Il s’agit d’un mécanisme de sécurité qui permet de se prémunir contre les tentatives d’infiltrations d’appareils « frauduleux » dans Azure AD, sans informations d’identification. L’échec entraîne l’affichage d’un message d’erreur indiquant une incompatibilité ZTDID.
- Dans le cadre d’un déploiement de pré-approvisionnement Windows Autopilot, l’affectation échouera, car le système ne pourra pas trouver l’appareil Azure AD associé. (En arrière-plan, les déploiements de pré-provisionnement utilisent le même processus en mode de déploiement automatique, de sorte qu’ils appliquent les mêmes mécanismes de sécurité.)

### <a name="how-do-i-know-all-the-type-of-devices-joined"></a>Comment faire pour connaître tous les types d’appareils joints ?

Pour en savoir plus sur les différents types, consultez l’article [Vue d’ensemble de la gestion des appareils](overview.md).

### <a name="what-happens-when-i-disable-a-device"></a>Que se passe-t-il lorsque je désactive un appareil ?

Toute authentification auprès d’Azure AD effectuée avec l’appareil est refusée. Voici des exemples courants :

- **Appareil joint à une version hybride d’Azure AD**  : les utilisateurs peuvent être autorisés à utiliser l’appareil pour se connecter à leur domaine local. Toutefois, ils ne peuvent pas accéder aux ressources Azure AD telles que Microsoft 365.
- **Appareil joint à Azure AD** : les utilisateurs ne peuvent pas utiliser l’appareil pour se connecter. 
- **Appareils mobiles** : les utilisateurs ne peuvent pas accéder aux ressources Azure AD telles que Microsoft 365. 

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une vue d’ensemble de la gestion des appareils dans le portail Azure, consultez [Gestion des appareils via le portail Azure](device-management-azure-portal.md).
