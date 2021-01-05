---
title: Fichier Include
description: Fichier Include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 09/16/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: e2a1eb8524c9d9a4b0ae603da3c05fbb20900111
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995470"
---
SMB Multichannel pour les partages de fichiers Azure présente actuellement les restrictions suivantes :
- Peut uniquement être utilisé avec les comptes FileStorage localement redondants.
- Pris en charge uniquement pour les clients Windows. 
- Le nombre maximal de canaux est de quatre.
- SMB Direct n’est pas pris en charge.
- Pour les comptes de stockage dotés de [l’authentification basée sur l’identité](../articles/storage/files/storage-files-active-directory-overview.md) Active Directory Domain Services (AD DS) ou Azure AD DS locale pour Azure Files, les clients SMB ne peuvent pas utiliser l’Explorateur de fichiers Windows pour configurer des autorisations NTFS sur les répertoires et les fichiers.
    - Utilisez plutôt l’outil Windows [icacls](/windows-server/administration/windows-commands/icacls) ou la commande [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl?view=powershell-7) pour configurer les autorisations.

