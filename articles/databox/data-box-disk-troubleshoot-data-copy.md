---
title: Résolution des problèmes de copie de données d’Azure Data Box Disk | Microsoft Docs
description: Décrit la résolution des problèmes rencontrés pendant la copie de données dans Azure Data Box Disk à l’aide de journaux.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/13/2019
ms.author: alkohli
ms.openlocfilehash: 5d977fe0b7459af35f678e77681d3b27c31431cc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85849176"
---
# <a name="troubleshoot-data-copy-issues-in-azure-data-box-disk"></a>Résoudre les problèmes de copie de données rencontrés dans Azure Data Box Disk

Cet article s’applique à Microsoft Azure Data Box Disk et décrit comment résoudre les problèmes que vous rencontrez lors de la copie de données sur des disques. L’article couvre également les problèmes rencontrés lors de l’utilisation de l’outil de copie de fractionnement.


## <a name="data-copy-issues-when-using-a-linux-system"></a>Problèmes de copie de données lors de l’utilisation d’un système Linux

Cette section décrit en détails certains des principaux problèmes rencontrés lors de l’utilisation d’un client Linux pour copier des données sur des disques.

### <a name="issue-drive-getting-mounted-as-read-only"></a>Problème : Lecteur monté en lecture seule
 
**Cause** 

Cela peut être dû à un système de fichiers qui n’est pas propre.

Le remontage d'un lecteur en lecture-écriture ne fonctionne pas avec la solution Data Box Disk. Ce scénario n'est pas pris en charge avec les lecteurs déchiffrés par dislocker. Vous avez peut-être réussi à remonter le périphérique à l’aide de la commande suivante :

```
# mount -o remount, rw /mnt/DataBoxDisk/mountVol1
```

Bien que le remontage ait réussi, les données ne seront pas conservées.

**Résolution :**

Sur votre système Linux, procédez comme suit :

1. Installez le package `ntfsprogs` pour l’utilitaire ntfsfix.
2. Démontez les points de montage fournis pour le lecteur avec l’outil de déverrouillage. Le nombre de points de montage varie en fonction des lecteurs.

    ```
    unmount /mnt/DataBoxDisk/mountVol1
    ```

3. Exécutez `ntfsfix` sur le chemin d’accès correspondant. Le nombre en surbrillance doit être identique à celui de l’étape 2.

    ```
    ntfsfix /mnt/DataBoxDisk/bitlockerVol1/dislocker-file
    ```

4. Exécutez la commande suivante pour supprimer les métadonnées de mise en veille prolongée qui peuvent entraîner un problème de montage.

    ```
    ntfs-3g -o remove_hiberfile /mnt/DataBoxDisk/bitlockerVol1/dislocker-file /mnt/DataBoxDisk/mountVol1
    ```

5. Procédez à un démontage consciencieux.

    ```
    ./DataBoxDiskUnlock_x86_64 /unmount
    ```

6. Procédez à un montage et déverrouillage consciencieux.
7. Testez le point de montage en écrivant un fichier.
8. Démontez et remontez pour valider la persistance du fichier.
9. Continuez avec la copie des données.
 
### <a name="issue-error-with-data-not-persisting-after-copy"></a>Problème : Erreur avec des données non persistantes après copie
 
**Cause** 

Si vous constatez que votre lecteur est vide après démontage (bien que des données aient été copiées dessus), il est possible que vous ayez remonté un lecteur en lecture-écriture alors qu’il a été monté précédemment en lecture seule.

**Résolution :**
 
Si tel est le cas, reportez-vous à la résolution des [Lecteurs montés en lecture seule](#issue-drive-getting-mounted-as-read-only).

Si ce n’est pas le cas, copiez les journaux d’activité du dossier contenant l’outil de déverrouillage Data Box Disk et [contactez le support Microsoft](data-box-disk-contact-microsoft-support.md).


## <a name="data-box-disk-split-copy-tool-errors"></a>Erreurs de l’outil de copie Data Box Disk Split

Les problèmes rencontrés lors de l’utilisation d’un outil Split Copy pour fractionner les données de plusieurs disques sont résumés dans le tableau suivant.

|Messages d’erreur/avertissements |Recommandations |
|---------|---------|
|[Info] Récupération du mot de passe BitLocker du volume : m <br>[Erreur] Exception détectée lors de la récupération de la clé BitLocker du volume m :<br> La séquence ne contient aucun élément.|Cette erreur se produit lorsque la destination Data Box Disk est hors ligne. <br> Utilisez l’outil `diskmgmt.msc` sur les disques en ligne.|
|[Erreur] Exception levée : Échec de l'opération WMI :<br> Method=UnlockWithNumericalPassword, ReturnValue=2150694965, <br>Win32Message=Le format du mot de passe de récupération fourni est invalide. <br>Les mots de passe de récupération BitLocker sont composés de 48 chiffres. <br>Vérifiez le format du mot de passe de récupération, puis essayez à nouveau.|Utilisez l’outil de déverrouillage de Data Box Disk Unlock pour déverrouiller les disques en premier lieu, puis essayez à nouveau la commande. Pour plus d’informations, consultez <li> [Déverrouillez Data Box Disk pour les clients Windows](data-box-disk-deploy-set-up.md#unlock-disks-on-windows-client). </li><li> [Déverrouillez Data Box Disk pour les clients Linux](data-box-disk-deploy-set-up.md#unlock-disks-on-linux-client). </li>|
|[Erreur] Exception levée : Un fichier DriveManifest.xml se trouve sur le lecteur cible. <br> Cela signifie que le lecteur cible peut avoir été préparé avec un fichier journal différent. <br>Pour ajouter de nouvelles données sur le même lecteur, utilisez le fichier journal précédent. Pour supprimer les données existantes et réutiliser le lecteur cible pour un nouveau travail d’importation, supprimez le fichier *DriveManifest.xml* sur le lecteur. Exécutez à nouveau cette commande avec un nouveau fichier journal.| Cette erreur se produit lorsque vous essayez d’utiliser le même jeu de lecteurs pour plusieurs sessions d’importation. <br> Utilisez un jeu de lecteurs uniquement pour une seule session de fractionnement et de copie.|
|[Erreur] Exception levée : CopySessionId importdata-sept-test-1 fait référence à une session de copie précédente et ne peut pas être réutilisé pour une nouvelle session de copie.|Cette erreur est signalée lorsque vous essayez de nommer un nouveau travail avec le même nom que celui d’un travail précédent terminé avec succès.<br> Attribuez un nom unique à votre nouveau travail.|
|[Info] Le nom du fichier ou du répertoire de destination dépasse la limite de longueur NTFS. |Ce message est signalé lorsque le fichier de destination a été renommé en raison de la longueur du chemin d’accès au fichier.<br> Modifiez l’option de disposition dans le fichier `config.json` pour contrôler ce comportement.|
|[Erreur] Exception levée : Séquence d’échappement JSON incorrecte. |Ce message est signalé lorsque le format du fichier config.json n’est pas valide. <br> Validez votre `config.json` avec [JSONlint](https://jsonlint.com/) avant de sauvegarder le fichier.|


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [résoudre les problèmes d’outil de validation](data-box-disk-troubleshoot.md).
