---
title: Mettre à jour le mot de passe du compte de stockage AD DS
description: Découvrez comment mettre à jour le mot de passe du compte Active Directory Domain Services qui représente votre compte de stockage. Cela permet d’éviter le nettoyage du compte de stockage quand le mot de passe arrive à expiration, empêchant ainsi les échecs d’authentification.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: d16f11a85c6b370b0187975cce965bf3e1b5ba3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85510252"
---
# <a name="update-the-password-of-your-storage-account-identity-in-ad-ds"></a>Mettre à jour le mot de passe de l’identité de votre compte de stockage dans AD DS

Si vous avez inscrit l’identité/le compte AD DS (Active Directory Domain Services) qui représente votre compte de stockage dans une unité d’organisation ou un domaine qui applique l’heure d’expiration du mot de passe, vous devez modifier le mot de passe avant sa durée de vie maximale. Votre organisation peut exécuter des scripts de nettoyage automatisés qui suppriment les comptes une fois que leur mot de passe arrive à expiration. Pour cette raison, si vous ne modifiez pas votre mot de passe avant son expiration, votre compte peut être supprimé, rendant l’accès à vos partages de fichiers Azure impossible.

Pour déclencher la rotation des mots de passe, vous pouvez exécuter la commande `Update-AzStorageAccountADObjectPassword` à partir du [module AzFilesHybrid](https://github.com/Azure-Samples/azure-files-samples/releases). Cette commande doit être exécutée dans un environnement joint à un domaine AD DS en local à l’aide d’un utilisateur hybride disposant de l’autorisation de propriétaire sur le compte de stockage et d’autorisations AD DS pour modifier le mot de passe de l’identité représentant le compte de stockage. La commande effectue des actions similaires à la rotation des clés du compte de stockage. Plus précisément, elle récupère la deuxième clé Kerberos du compte de stockage et l’utilise pour mettre à jour le mot de passe du compte inscrit dans AD DS. Ensuite, elle regénère la clé Kerberos cible du compte de stockage et met à jour le mot de passe du compte inscrit dans AD DS. Vous devez exécuter cette commande dans un environnement joint à un domaine AD DS en local.

```PowerShell
# Update the password of the AD DS account registered for the storage account
# You may use either kerb1 or kerb2
Update-AzStorageAccountADObjectPassword `
        -RotateToKerbKey kerb2 `
        -ResourceGroupName "<your-resource-group-name-here>" `
        -StorageAccountName "<your-storage-account-name-here>"
```
