---
title: Fonctionnalités et configuration du service Azure AD Connect | Microsoft Docs
description: Décrit les fonctionnalités côté service du service de synchronisation Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 213aab20-0a61-434a-9545-c4637628da81
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/18/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 261ab5d0f039705a2566b7c28ff4c06778bb661a
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94410536"
---
# <a name="azure-ad-connect-sync-service-features"></a>Fonctionnalités de service de synchronisation d’Azure AD Connect

La fonctionnalité de synchronisation d’Azure AD Connect comprend deux composants :

* Le composant local nommé **Azure AD Connect sync** , également appelé **moteur de synchronisation**.
* Le service résidant dans Azure AD, également appelé **service de synchronisation Azure AD Connect**

Cette rubrique explique comment les fonctionnalités suivantes du **service de synchronisation Azure AD Connect** opèrent et comment les configurer à l’aide de Windows PowerShell.

Ces paramètres sont configurés à partir du [module Azure Active Directory pour Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)). Téléchargez et installez-le séparément à partir d’Azure AD Connect. Les applets de commande documentées dans cette rubrique ont été introduites dans la [version de mars 2016 (build 9031.1)](https://social.technet.microsoft.com/wiki/contents/articles/28552.microsoft-azure-active-directory-powershell-module-version-release-history.aspx#Version_9031_1). Si vous n’avez pas les applets de commande documentées dans cette rubrique, ou que vous n’obtenez pas le même résultat, vérifiez que vous exécutez la version la plus récente.

Pour afficher la configuration de votre répertoire Azure AD, exécutez `Get-MsolDirSyncFeatures`.  
![Résultat de Get-MsolDirSyncFeatures](./media/how-to-connect-syncservice-features/getmsoldirsyncfeatures.png)

Plusieurs de ces paramètres peuvent uniquement être modifiés par Azure AD Connect.

Les paramètres suivants peuvent être configurés par `Set-MsolDirSyncFeature`:

| DirSyncFeature | Commentaire |
| --- | --- |
| [EnableSoftMatchOnUpn](#userprincipalname-soft-match) |Permet aux objets d’être joints sur userPrincipalName en plus de l’adresse SMTP principale. |
| [SynchronizeUpnForManagedUsers](#synchronize-userprincipalname-updates) |Permet au moteur de synchronisation de mettre à jour l’attribut userPrincipalName pour les utilisateurs gérés/licenciés(non fédérés). |

Lorsqu’une fonctionnalité a été activée, vous ne pouvez plus la désactiver.

> [!NOTE]
> Depuis le 24 août 2016, la fonctionnalité *Résilience d’attribut en double* est activée par défaut pour les nouveaux répertoires Azure AD. Cette fonctionnalité sera également déployée et activée sur les répertoires créés avant cette date. Vous recevrez une notification par courrier électronique lorsque l’activation de cette fonctionnalité pour votre répertoire sera imminente .
> 
> 

Les paramètres suivants sont configurés par Azure AD Connect et ne peuvent pas être modifiés par `Set-MsolDirSyncFeature`:

| DirSyncFeature | Commentaire |
| --- | --- |
| DeviceWriteback |[Azure AD Connect : Activation de la réécriture d’appareil](how-to-connect-device-writeback.md) |
| DirectoryExtensions |[Synchronisation Azure AD Connect : Extensions d’annuaire](how-to-connect-sync-feature-directory-extensions.md) |
| [DuplicateProxyAddressResiliency<br/>DuplicateUPNResiliency](#duplicate-attribute-resiliency) |Permet à un attribut d’être mis en quarantaine lorsqu’il est un doublon d’un autre objet, plutôt que mettre en échec l’objet entier lors de l’exportation. |
| Synchronisation de hachage de mot de passe |[Implémenter la synchronisation du hachage de mot de passe avec Azure AD Connect Sync](how-to-connect-password-hash-synchronization.md) |
|Authentification directe|[Connexion de l’utilisateur avec l’authentification directe Azure Active Directory](how-to-connect-pta.md)|
| UnifiedGroupWriteback |Écriture différée de groupe|
| UserWriteback |Non pris en charge actuellement. |

## <a name="duplicate-attribute-resiliency"></a>Résilience d’attribut en double

Au lieu de rencontrer des problèmes lors de l’approvisionnement des UPN/proxyAddresses en double, l’attribut en double est « mis en quarantaine » et une valeur temporaire lui est attribuée. Lorsque le conflit est résolu, l’UPN temporaire est automatiquement converti dans la valeur correcte. Pour plus d’informations, voir [Synchronisation des identités et résilience d’attribut en double](how-to-connect-syncservice-duplicate-attribute-resiliency.md).

## <a name="userprincipalname-soft-match"></a>Correspondance souple UserPrincipalName

Lorsque cette fonctionnalité est activée, la correspondance souple est activée pour l’UPN ainsi que pour l’ [adresse SMTP principale](https://support.microsoft.com/kb/2641663), qui est toujours activée. La correspondance souple est utilisée pour faire correspondre les utilisateurs existants du cloud dans Azure AD avec les utilisateurs locaux.

Cette fonctionnalité est utile lorsque vous mettez en correspondance des comptes AD locaux avec des comptes existants créés dans le cloud, et que vous n’utilisez pas Exchange Online. Dans ce scénario, vous n’avez généralement pas de raison pour définir l’attribut SMTP dans le cloud.

Cette fonctionnalité est activée par défaut pour les répertoires Azure AD nouvellement créés. Vous pouvez voir si cette fonctionnalité est activée en exécutant :  

```powershell
Get-MsolDirSyncFeatures -Feature EnableSoftMatchOnUpn
```

Si cette fonctionnalité n’est pas activée pour votre répertoire Azure AD, vous pouvez l’activer en exécutant :  

```powershell
Set-MsolDirSyncFeature -Feature EnableSoftMatchOnUpn -Enable $true
```

## <a name="synchronize-userprincipalname-updates"></a>Synchroniser les mises à jour userPrincipalName

Historiquement, les mises à jour de l’attribut UserPrincipalName à l’aide du service de synchronisation du site ont été bloquées, sauf si les deux conditions suivantes étaient remplies :

* L’utilisateur est géré (non fédéré).
* Aucune licence n’a été attribuée à l’utilisateur.

> [!NOTE]
> À compter de mars 2019, la synchronisation des modifications d’UPN pour les comptes d’utilisateur fédérés est autorisée.
> 

L’activation de cette fonctionnalité permet au moteur de synchronisation de mettre à jour l’attribut userPrincipalName quand il est modifié en local et que vous utilisez la synchronisation de hachage de mot de passe pu l’authentification directe.

Cette fonctionnalité est activée par défaut pour les répertoires Azure AD nouvellement créés. Vous pouvez voir si cette fonctionnalité est activée en exécutant :  

```powershell
Get-MsolDirSyncFeatures -Feature SynchronizeUpnForManagedUsers
```

Si cette fonctionnalité n’est pas activée pour votre répertoire Azure AD, vous pouvez l’activer en exécutant :  

```powershell
Set-MsolDirSyncFeature -Feature SynchronizeUpnForManagedUsers -Enable $true
```

Après avoir activé cette fonctionnalité, les valeurs existantes userPrincipalName demeurent telles quelles. Lors de la prochaine modification de l’attribut userPrincipalName en local, la synchronisation delta normale sur les utilisateurs met à jour l’UPN.  

## <a name="see-also"></a>Voir aussi

* [Synchronisation d’Azure AD Connect](how-to-connect-sync-whatis.md)
* [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).