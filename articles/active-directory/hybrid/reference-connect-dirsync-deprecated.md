---
title: Mettre à niveau depuis DirSync et Azure AD Sync | Microsoft Docs
description: Décrit comment effectuer la mise à niveau depuis DirSync et Azure AD Sync vers Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: bd68fb88-110b-4d76-978a-233e15590803
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: 915b56e9a9340920e99f4d3d4de6da4c39233eab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90014801"
---
# <a name="upgrade-windows-azure-active-directory-sync-and-azure-active-directory-sync"></a>Mettre à niveau Microsoft Azure Active Directory Sync et Azure Active Directory Sync
Azure AD Connect est la meilleure façon de connecter votre répertoire local avec Azure AD et Microsoft 365. C’est l’occasion idéale d’effectuer la mise à niveau vers Azure AD Connect à partir de Microsoft Azure Active Directory Sync (DirSync) ou Azure AD Sync, car ces outils sont désormais dépréciés et ne sont plus pris en charge depuis le 13 avril 2017.

Les deux outils de synchronisation des identités qui sont déconseillés étaient proposés pour les clients de forêt unique (DirSync) et pour les clients à forêts multiples et expérimentés (Azure AD Sync). Ces outils plus anciens ont été remplacés par une solution unique qui est disponible pour tous les scénarios : Azure AD Connect. Cette solution offre de nouvelles fonctionnalités, des améliorations de fonctionnalités et la prise en charge de nouveaux scénarios. Pour pouvoir continuer à synchroniser vos données d’identités locales vers Azure AD et Microsoft 365, nous vous recommandons vivement d’effectuer la mise à niveau vers Azure AD Connect. Microsoft ne garantit pas que ces versions plus anciennes fonctionneront après le 31 décembre 2017.

La dernière version de DirSync a été publiée en juillet 2014 et la dernière version d’Azure AD Sync a été publiée en mai 2015.

## <a name="what-is-azure-ad-connect"></a>Qu’est-ce qu’Azure AD Connect ?
Azure AD Connect est le successeur de DirSync et d’Azure AD Sync. Il combine tous les scénarios pris en charge par les deux autres. Pour en savoir plus, consultez [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

## <a name="deprecation-schedule"></a>Planification de la désapprobation
| Date | Commentaire |
| --- | --- |
| 13 avril 2016 |Microsoft Azure Active Directory Sync (« DirSync ») et Azure Active Directory Sync (« Azure AD Sync ») sont annoncés comme déconseillés. |
| 13 avril 2017 |Fin de la prise en charge. Les clients ne seront plus en mesure d’ouvrir un cas de support sans tout d’abord effectuer la mise à niveau vers Azure AD Connect. |
|31 décembre 2017|Il est possible qu’Azure AD n’accepte plus les communications provenant de Windows Azure Active Directory Sync (« DirSync ») et de Microsoft Azure Active Directory Sync (« Azure AD Sync »).

## <a name="how-to-transition-to-azure-ad-connect"></a>Comment passer à Azure AD Connect
Si vous exécutez DirSync, vous disposez de deux méthodes de mise à niveau : Mise à niveau sur place et déploiement parallèle. Une mise à niveau sur place est recommandée pour la plupart des clients et si vous avez un système d’exploitation récent avec moins de 50 000 objets. Dans d’autres cas, il est recommandé d’effectuer un déploiement parallèle dans lequel votre configuration DirSync est déplacée vers un nouveau serveur exécutant Azure AD Connect.

| Solution | Scénario |
| --- | --- |
| [Mise à niveau à partir de DirSync](how-to-dirsync-upgrade-get-started.md) |<li>Si vous disposez d’un serveur DirSync existant déjà en cours d’exécution.</li> |
| [Mise à niveau à partir d’Azure AD Sync](how-to-upgrade-previous-version.md) |<li>Si vous effectuez la mise à niveau à partir d’Azure AD Sync.</li> |

Si vous souhaitez savoir comment effectuer une mise à niveau sur place depuis DirSync vers Azure AD Connect, regardez cette vidéo de Channel 9 :

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-in-place-upgrade-from-legacy-tools/player]
>
>

## <a name="faq"></a>Questions fréquentes (FAQ)
**Q : J’ai reçu une notification par courrier électronique de l’équipe Azure et/ou un message du centre de messages Microsoft 365, mais j’utilise Connect.**  
La notification a été également envoyée aux clients à l'aide d'Azure AD Connect avec un numéro de build 1.0. \*. 0 (à l'aide d'une version pre-1.1). Microsoft recommande aux clients de maintenir à jour leurs versions d’Azure AD Connect. Grâce à la fonctionnalité de [mise à niveau automatique](how-to-connect-install-automatic-upgrade.md) introduite dans la version 1.1, il est facile de toujours avoir une version récente d’Azure AD Connect installée.

**Q : DirSync/Azure AD Sync vont-ils cesser de fonctionner le 13 avril 2017 ?**  
DirSync/Azure AD Sync continueront à fonctionner après le 13 avril 2017.  Cependant, après le 31 décembre 2017, Azure AD risque de ne plus accepter les communications provenant de DirSync/Azure AD Sync.

**Q : Quelles versions de DirSync puis-je mettre à niveau ?**  
Une mise à niveau est prise en charge à partir de n’importe quelle version de DirSync utilisée. 

**Q : Qu’en est-il du connecteur Azure AD pour FIM/MIM ?**  
Le connecteur Azure AD pour FIM/MIM n’a **pas** été annoncé comme déconseillé. Il est dans l’état **feature freeze**; aucune nouvelle fonctionnalité n’est ajoutée et il ne reçoit aucune résolution de bogue. Microsoft recommande aux clients qui l’utilisent d’envisager de passer à Azure AD Connect. Il est fortement recommandé ne pas démarrer de nouveaux déploiements en l’utilisant. Ce connecteur sera annoncé comme déconseillé par la suite.

## <a name="additional-resources"></a>Ressources supplémentaires
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
