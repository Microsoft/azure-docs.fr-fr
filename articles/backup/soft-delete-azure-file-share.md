---
title: Protection contre la suppression accidentelle pour les partages de fichiers Azure
description: Découvrez comment la suppression réversible peut protéger vos partages de fichiers Azure contre la suppression accidentelle.
ms.topic: conceptual
ms.date: 02/02/2020
ms.custom: references_regions
ms.openlocfilehash: 52a116320e07c25d4ee7f00b8063ca15faeb8560
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89179910"
---
# <a name="accidental-delete-protection-for-azure-file-shares-using-azure-backup"></a>Protection contre la suppression accidentelle pour les partages de fichiers Azure à l’aide de Sauvegarde Azure

Pour assurer la protection contre les cyberattaques ou les suppressions accidentelles, la [suppression réversible](../storage/files/storage-files-prevent-file-share-deletion.md) est activée pour tous les partages de fichiers dans un compte de stockage quand vous configurez la sauvegarde pour tout partage de fichiers dans le compte de stockage correspondant. Avec la suppression réversible, même si un intervenant malveillant supprime le partage de fichiers, le contenu et les points de récupération (captures instantanées) du partage de fichiers sont conservés pendant un minimum de 14 jours supplémentaires, ce qui permet la récupération des partages de fichiers sans perte de données.  La suppression réversible est prise en charge pour les comptes de stockage standard et Premium, et le paramètre est activé par le service Sauvegarde Azure pour tous les comptes de stockage hébergeant des partages de fichiers sauvegardés.

L’organigramme suivant montre les différentes étapes et états d’un élément de sauvegarde quand la suppression réversible est activée pour les partages de fichiers dans un compte de stockage :

 ![Organigramme de suppression réversible](./media/soft-delete-afs/soft-delete-flow-chart.png)

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="when-will-soft-delete-be-enabled-for-file-shares-in-my-storage-account"></a>Quand la suppression réversible sera activée pour les partages de fichiers dans mon compte de stockage ?

Quand vous configurez la sauvegarde pour la première fois pour un partage de fichiers dans un compte de stockage, le service Sauvegarde Azure active la suppression réversible pour tous les partages de fichiers dans le compte de stockage correspondant.

### <a name="can-i-configure-the-number-of-days-for-which-my-snapshots-and-restore-points-will-be-retained-in-soft-deleted-state-after-i-delete-the-file-share"></a>Puis-je configurer le nombre de jours pendant lesquels mes captures instantanées et mes points de restauration sont conservés dans un état de suppression réversible après la suppression du partage de fichiers ?

Oui, vous pouvez définir la période de conservation en fonction de vos besoins. [Ce document](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal) explique les étapes de configuration de la période de conservation. Pour les comptes de stockage avec des partages de fichiers sauvegardés, le paramètre de conservation minimale doit être de 14 jours.

### <a name="does-azure-backup-reset-my-retention-setting-because-i-configured-it-to-less-than-14-days"></a>Est-ce que la Sauvegarde Azure réinitialise mon paramètre de conservation car je l’ai configuré sur moins de 14 jours ?

Du point de vue de la sécurité, nous vous recommandons d’avoir une conservation minimale de 14 jours pour les comptes de stockage avec des partages de fichiers sauvegardés. Ainsi, à chaque exécution d’un travail de sauvegarde, si Sauvegarde Azure détecte que le paramètre a une valeur inférieure à 14 jours, il le réinitialise à 14 jours.

### <a name="what-is-the-cost-incurred-during-the-retention-period"></a>Quel est le coût encouru pendant la période de conservation ?

Pendant la période de suppression réversible, le coût de stockage des captures instantanées et le coût de l’instance protégée reste le même.  De plus, vous serez facturé pour la capacité utilisée au tarif normal pour les partages de fichiers Standard et au taux de stockage des captures instantanées pour les partages de fichiers Premium.

### <a name="can-i-perform-a-restore-operation-when-my-data-is-in-soft-deleted-state"></a>Puis-je effectuer une opération de restauration quand mes données sont à l’état de suppression réversible ?

Vous devez d’abord annuler la suppression du partage de fichiers supprimé de manière réversible pour effectuer des opérations de restauration. L’opération d’annulation de la suppression placera le partage de fichiers à l’état sauvegardé, et vous pourrez alors effectuer la restauration à n’importe quel point dans le temps. Pour découvrir comment annuler la suppression de votre partage de fichiers, cliquez sur [ce lien](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) ou consultez le [script d’annulation de la suppression de partage de fichiers](./scripts/backup-powershell-script-undelete-file-share.md).

### <a name="how-can-i-purge-the-data-of-a-file-share-in-a-storage-account-that-has-at-least-one-protected-file-share"></a>Comment vider les données d’un partage de fichiers dans un compte de stockage ayant au moins un partage de fichiers protégé ?

Si vous disposez d’au moins un partage de fichiers protégé dans un compte de stockage, cela signifie que la suppression réversible est activée pour tous les partages de fichiers de ce compte et que vos données sont conservées pendant 14 jours après l’opération de suppression. Si toutefois vous souhaitez vider les données immédiatement et ne pas les conserver, effectuez les étapes suivantes :

1. Si vous avez déjà supprimé le partage de fichiers alors que la suppression réversible était activée, commencez par annuler la suppression du partage de fichiers à partir du [Portail des fichiers](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) ou à l’aide du [script d’annulation de la suppression de partage de fichiers](./scripts/backup-powershell-script-undelete-file-share.md).
2. Désactivez la suppression réversible pour les partages de fichiers dans votre compte de stockage en suivant les étapes mentionnées dans [ce document](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#disable-soft-delete).
3. Ensuite, supprimez le partage de fichiers dont vous souhaitez vider le contenu immédiatement.

>[!NOTE]
>Vous devez effectuer l’étape 2 avant l’exécution de la tâche de sauvegarde planifiée suivante sur le partage de fichiers protégé dans votre compte de stockage. En effet, chaque fois que le travail de sauvegarde s’exécute, il réactive la suppression réversible pour tous les partages de fichiers dans le compte de stockage.

>[!WARNING]
>Après la désactivation de la suppression réversible à l’étape 2, toute opération de suppression effectuée sur les partages de fichiers est une opération de suppression permanente. Donc, si vous supprimez accidentellement le partage de fichiers sauvegardé après la désactivation de la suppression réversible, vous perdrez toutes vos captures instantanées et ne pourrez pas récupérer vos données.

### <a name="in-the-context-of-a-file-shares-soft-delete-setting-what-changes-does-azure-backup-do-when-i-unregister-a-storage-account"></a>Dans le contexte du paramètre de suppression réversible d’un partage de fichiers, quelles sont les modifications apportées par Sauvegarde Azure quand j’annule l’inscription d’un compte de stockage ?

Au moment de l’annulation de l’inscription, Sauvegarde Azure vérifie le paramètre de période de conservation pour les partages de fichiers et, s’il est supérieur ou inférieur à 14 jours, il laisse la conservation telle quelle. En revanche, si la conservation est de 14 jours, nous la considérons comme étant activée par Sauvegarde Azure et nous désactivons donc la suppression réversible pendant le processus de désinscription. Si vous souhaitez annuler l’inscription du compte de stockage tout en gardant le paramètre de conservation, réactivez-le à partir du volet du compte de stockage après avoir terminé l’annulation de l’inscription. Vous pouvez consulter [ce lien](../storage/files/storage-files-enable-soft-delete.md?tabs=azure-portal#restore-soft-deleted-file-share) pour connaître les étapes de configuration.

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [sauvegarder des partages de fichiers Azure à partir du portail Azure](backup-afs.md)
