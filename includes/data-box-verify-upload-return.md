---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 09/30/2019
ms.author: alkohli
ms.openlocfilehash: ca7b83d24f2416b224963559361faf5a7775cd0d
ms.sourcegitcommit: d479ad7ae4b6c2c416049cb0e0221ce15470acf6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2020
ms.locfileid: "91631532"
---
Lorsque Microsoft reçoit et analyse l’appareil, l’état de la commande est mis à jour sur **Reçue** . L’appareil subit une vérification physique afin de détecter des dommages ou des signes d’altération potentiels.

Une fois la vérification terminée, la Data Box est connectée au réseau du centre de données Azure. La copie des données démarre automatiquement. Selon la taille des données, l’opération de copie peut prendre de quelques heures à quelques jours. Vous pouvez suivre la progression du travail de copie dans le portail.

Une fois la copie terminée, la commande passe à l’état **Completed (Terminée)** .

Vérifiez que vos données ont été chargées dans Azure avant de les supprimer de la source. Vos données peuvent se trouver aux emplacements suivants :

- Votre ou vos comptes de stockage Azure. Lorsque vous copiez les données sur Data Box, les données sont chargées vers l’un des chemins ci-après dans votre compte Stockage Azure :

  - Pour les objets blob de blocs et de pages : `https://<storage_account_name>.blob.core.windows.net/<containername>/files/a.txt`
  - Pour les fichiers Azure : `https://<storage_account_name>.file.core.windows.net/<sharename>/files/a.txt`

    Vous pouvez également accéder à votre compte de stockage Azure dans le Portail Azure et naviguer à partir de cet emplacement.

- Votre ou vos groupes de ressources spécifiés pour les disques managés. Quand vous créez des disques managés, les disques durs virtuels sont chargés en tant qu’objets blob de pages, puis convertis en disques managés. Les disques managés sont attachés aux groupes de ressources spécifiés au moment de la création de la commande. 

    - Si la copie vers des disques managés dans Azure a réussi, vous pouvez accéder aux **détails de la commande** dans le portail Azure et noter les noms des groupes de ressources spécifiés pour les disques managés.

        ![Identifier les groupes de ressources spécifiés pour les disques managés](media/data-box-verify-upload-return/order-details-managed-disk-resource-groups.png)

        Accédez au groupe de ressources noté précédemment pour trouver vos disques managés.

        ![Disque managé attaché à des groupes de ressources](media/data-box-verify-upload-return/managed-disks-resource-group.png)

    - Si vous avez copié un VHDX, ou un VHD dynamique ou de différenciation, le VHDX ou VHD est chargé dans le compte de stockage de préproduction comme un objet blob de pages, mais la conversion du VHD en disque managé échoue. Accédez à votre compte de stockage de préproduction ( **Compte de stockage > Objets blob** ), et sélectionnez le conteneur approprié (SSD Standard, HDD Standard ou SSD Premium). Les disques durs virtuels sont chargés en tant qu’objets blob de pages dans votre compte de stockage de préproduction, et engendrent des coûts supplémentaires.


## <a name="erasure-of-data-from-data-box"></a>Effacer les données de la Data Box
 
Une fois que le chargement des données vers Azure est terminé, la Data Box efface les données de ses disques, conformément aux [instructions du standard NIST SP 800-88 Revision 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).


## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
> * Conditions préalables requises
> * Préparer l’expédition
> * Expédier la Data Box à Microsoft
> * Vérifier le chargement des données dans Azure
> * Effacer les données de la Data Box

Passez à l’article suivant pour apprendre à gérer la Data Box via l’interface utilisateur web locale.

> [!div class="nextstepaction"]
> [Administrer l’appareil Azure Data Box à l’aide de l’interface utilisateur web locale](../articles/databox/data-box-local-web-ui-admin.md)