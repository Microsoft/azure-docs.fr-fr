---
title: Résolution des problèmes de déverrouillage des disques dans Azure Data Box Disk | Microsoft Docs
description: En savoir plus sur les flux de travail permettant de résoudre les problèmes liés à l’outil de déverrouillage avec Azure Data Box Disk. Consultez les erreurs de l’outil de déverrouillage de Data Box Disk.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 08/05/2020
ms.author: alkohli
ms.openlocfilehash: 866cf01243983863292ada0b086f8f5b2f94e412
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87925559"
---
# <a name="troubleshoot-disk-unlocking-issues-in-azure-data-box-disk"></a>Résoudre les problèmes de déverrouillage des disques dans Azure Data Box Disk

Cet article s’applique à Microsoft Azure Data Box Disk et décrit les flux de travail utilisés pour résoudre les problèmes que vous rencontrez en cas d’utilisation de l’outil de déverrouillage. 


<!--## Query activity logs

Use the activity logs to find who unlocked and accessed the disks. Your Data Box Disk arrive on your premises in a locked state. You can use the device credentials available in the Azure portal for your order to unlock them.  

To figure out who accessed the **Device credentials** blade, you can query the Activity logs.  Any action that involves accessing **Device details > Credentials** blade is logged into the activity logs as `ListCredentials` action.

![Query Activity logs](media/data-box-logs/query-activity-log-1.png)-->


## <a name="data-box-disk-unlock-tool-errors"></a>Erreurs de l’outil de déverrouillage de Data Box Disk


| Message d’erreur/Comportement de l’outil      | Recommandations                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Le composant .NET Framework actuel n’est pas pris en charge. Les versions prises en charge sont les versions 4.5 et ultérieures.<br><br>L’outil se ferme avec un message.  | .NET 4.5 n’est pas installé. Installez .NET 4.5 ou ultérieur sur l’ordinateur hôte qui exécute l’outil de déverrouillage de Data Box Disk.                                                                            |
| Impossible de déverrouiller ou de vérifier des volumes. Contactez le Support Microsoft.  <br><br>L’outil ne parvient pas à déverrouiller ni à vérifier les lecteurs verrouillés. | L’outil n’a pas pu déverrouiller l’un des lecteurs verrouillés avec la clé d’accès fournie. Contactez le Support Microsoft pour les étapes suivantes.                                                |
| Les volumes suivants sont déverrouillés et vérifiés. <br>Lettre de lecteur du volume : E :<br>Impossible de déverrouiller les volumes avec les clés d’accès suivantes : werwerqomnf, qwerwerqwdfda <br><br>L’outil déverrouille certains lecteurs et répertorie les lettres de lecteur ayant réussi ou échoué.| Réussite partielle. Impossible de déverrouiller certains lecteurs avec la clé d’accès fournie. Contactez le Support Microsoft pour les étapes suivantes. |
| Les volumes verrouillés sont introuvables. Vérifiez que le disque envoyé par Microsoft est connecté correctement et qu’il se trouve à l’état verrouillé.          | L’outil ne parvient pas à trouver de lecteurs verrouillés. Soit les lecteurs sont déjà déverrouillés, soit ils ne sont pas détectés. Assurez-vous que les lecteurs sont connectés et verrouillés. <br> <br>Vous pouvez également voir cette erreur si vous avez formaté vos disques. Si vous avez formaté vos disques, ceux-ci sont désormais inutilisables. Contactez le Support Microsoft pour les étapes suivantes.                                                          |
| Erreur irrécupérable : Paramètre non valide<br>Nom de paramètre : invalid_arg<br>UTILISATION :<br>DataBoxDiskUnlock /PassKeys:<liste_clés_accès_séparées_par_points_virgules><br><br>Exemple : DataBoxDiskUnlock /PassKeys:passkey1;passkey2;passkey3<br>Exemple : DataBoxDiskUnlock /SystemCheck<br>Exemple : DataBoxDiskUnlock /Help<br><br>/PassKeys :       Permet d’obtenir cette clé d’accès à partir de la commande Azure Data Box Disk. La clé d’accès déverrouille vos disques.<br>/Help :           Cette option fournit une aide sur l’utilisation et les exemples de cmdlet.<br>/SystemCheck :    Cette option vérifie si votre système répond à la configuration requise pour l’exécution de l’outil.<br><br>Appuyez sur une touche pour quitter. | Paramètre non valide entré. Les seuls paramètres autorisés sont les suivants : /SystemCheck, /PassKey et /Help.|


## <a name="unlock-issues-for-disks-when-using-a-windows-client"></a>Problèmes de déverrouillage des disques en cas d’utilisation d’un client Windows

Cette section détaille certains des principaux problèmes rencontrés lors du déploiement de Data Box Disk en cas d’utilisation d’un client Windows pour la copie de données.

### <a name="issue-could-not-unlock-drive-from-bitlocker"></a>Problème : Impossible de déverrouiller le lecteur depuis BitLocker
 
**Cause** 

Vous avez utilisé le mot de passe dans la boîte de dialogue BitLocker et essayé de déverrouiller le disque via la boîte de dialogue BitLocker. Cette méthode ne fonctionne pas.

**Résolution :**

Pour déverrouiller les Data Box Disks, vous devez utiliser l’outil de déverrouillage Data Box Disk et fournir le mot de passe du portail Azure. Pour plus d'informations, consultez [Tutoriel : Déballer, connecter et déverrouiller un disque Azure Data Box](data-box-disk-deploy-set-up.md#connect-to-disks-and-get-the-passkey).
 
### <a name="issue-could-not-unlock-or-verify-some-volumes-contact-microsoft-support"></a>Problème : Impossible de déverrouiller ou de vérifier certains volumes. Contactez le Support Microsoft.
 
**Cause**

Il se peut que vous voyiez l’erreur suivante dans le journal des erreurs et que vous ne puissiez pas déverrouiller ou vérifier certains volumes.

`Exception System.IO.FileNotFoundException: Could not load file or assembly 'Microsoft.Management.Infrastructure, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies. The system cannot find the file specified.`
 
Cela indique qu’il vous manque probablement la version appropriée de Windows PowerShell sur votre client Windows.

**Résolution :**

Vous pouvez installer [Windows PowerShell v 5.0](https://www.microsoft.com/download/details.aspx?id=54616) et réessayer l'opération.
 
Si vous ne parvenez toujours pas à déverrouiller les volumes, copiez les journaux d’activité du dossier contenant l’outil de déverrouillage Data Box Disk et [contactez le support Microsoft](data-box-disk-contact-microsoft-support.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [résoudre les problèmes de validation](data-box-disk-troubleshoot.md).
