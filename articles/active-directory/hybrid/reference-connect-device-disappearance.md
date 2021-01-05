---
title: Présentation d’Azure AD Connect 1.4.xx.x et de la disparition des appareils | Microsoft Docs
description: Ce document décrit un problème qui se pose avec la version 1.4.xx.x d’Azure AD Connect
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/25/2019
ms.subservice: hybrid
ms.author: billmath
ms.openlocfilehash: bc159452c81a673ca4a7ed46aa7eff19fd9209eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73176026"
---
# <a name="understanding-azure-ad-connect-14xxx-and-device-disappearance"></a>Présentation d’Azure AD Connect 1.4.xx.x et de la disparition des appareils
Avec la version 1.4.xx.x d’Azure AD Connect, certains clients peuvent voir une partie ou l’ensemble de leurs appareils Windows disparaître d’Azure AD. Cela n’est pas préoccupant car Azure AD n’utilise pas ces identités d’appareil durant l’autorisation d’accès conditionnel. Ce changement ne supprime pas les appareils Windows correctement inscrits auprès d’Azure AD pour une jonction Azure AD Hybride.

Si vous constatez que les suppressions d’objets d’appareil dans Azure AD dépassent le seuil de suppression des exportations, il est conseillé que le client autorise la poursuite de ces suppressions. [Guide pratique pour autoriser le déroulement des suppressions quand elles dépassent le seuil de suppression](how-to-connect-sync-feature-prevent-accidental-deletes.md)

## <a name="background"></a>Arrière-plan
Les appareils Windows inscrits comme étant joints à une version hybride d’Azure AD sont représentés dans Azure AD en tant qu’objets d’appareil. Ces objets appareils peuvent être utilisés pour un accès conditionnel. Les appareils Windows 10 sont synchronisés dans le cloud via Azure AD Connect. Les appareils Windows de bas niveau sont inscrits directement à l’aide d’AD FS ou d’une authentification unique transparente.

## <a name="windows-10-devices"></a>Appareils Windows 10
Seuls les appareils Windows 10 avec une valeur d’attribut userCertificate spécifique configurée par une jonction Azure AD Hybride sont censés être synchronisés dans le cloud par Azure AD Connect. Dans les versions précédentes d’Azure AD Connect, cette exigence n’était pas rigoureusement appliquée, ce qui entraînait la présence d’objets appareils superflus dans Azure AD. Dans Azure AD, de tels appareils conservaient toujours l’état « en attente », car ils n’avaient pas pour vocation d’être inscrits auprès d’Azure AD. 

Cette version d’Azure AD Connect synchronise uniquement les appareils Windows 10 correctement configurés pour être joints à une version hybride d’Azure AD. Les objets d’appareil Windows 10 sans valeur d’attribut userCertificate spécifique à la jonction Azure AD sont supprimés d’Azure AD.

## <a name="down-level-windows-devices"></a>Appareils Windows de bas niveau
Azure AD Connect ne doit jamais se synchroniser [des appareils Windows de niveau supérieur](../devices/hybrid-azuread-join-plan.md#windows-down-level-devices). Tous les appareils dans Azure AD précédemment synchronisés de manière incorrecte sont désormais supprimés d’Azure AD. Si Azure AD Connect tente de supprimer un [appareil Windows de bas niveau](../devices/hybrid-azuread-join-plan.md#windows-down-level-devices), l’appareil n’est pas celui qui a été créé par le [MSI Microsoft Workplace Join pour les ordinateurs autres que Windows 10](https://www.microsoft.com/download/details.aspx?id=53554) et n’est utilisable par aucune autre fonctionnalité Azure AD.

Il se peut que certains clients doivent réexaminer [Guide pratique pour planifier l’implémentation de la jonction Azure Active Directory Hybride](../devices/hybrid-azuread-join-plan.md) pour inscrire leurs appareils Windows correctement et vérifier que ces appareils peuvent bénéficier pleinement d’un accès conditionnel basé sur les appareils. 

## <a name="how-can-i-verify-which-devices-are-deleted-with-this-update"></a>Comment vérifier quels appareils sont supprimés avec cette mise à jour ?

Pour vérifier quels appareils sont supprimés, vous pouvez utiliser ce script PowerShell : https://gallery.technet.microsoft.com/scriptcenter/Export-Hybrid-Azure-AD-f8e51436

Ce script génère un rapport sur les certificats stockés dans les objets d’ordinateur Active Directory, en particulier les certificats émis par la fonctionnalité de jonction Azure AD Hybride.
Il vérifie les certificats présents dans la propriété UserCertificate d’un objet d’ordinateur dans AD et, pour chaque certificat non expiré présent, valide si le certificat a été émis pour la fonctionnalité de jonction Azure AD Hybride (autrement dit, le nom d’objet correspond à CN={ObjectGUID}).
Avant, Azure AD Connect synchronisait dans Azure AD tout ordinateur qui contenait au moins un certificat valide mais, à partir d’Azure AD Connect version 1.4, le moteur de synchronisation peut identifier les certificats de jonction Azure AD Hybride et « filtrer dans le cloud » l’objet d’ordinateur pour ne pas le synchroniser avec Azure AD, sauf s’il existe un certificat de jonction Azure AD Hybride valide.
Les appareils Azure AD déjà synchronisés dans AD, mais sans certificat de jonction Azure AD Hybride valide, sont supprimés (CloudFiltered=TRUE) par le moteur de synchronisation.

## <a name="next-steps"></a>Étapes suivantes
- [Historique des versions d’Azure AD Connect](reference-connect-version-history.md)
