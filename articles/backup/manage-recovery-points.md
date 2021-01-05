---
title: Gérer les points de récupération
description: Découvrez comment le service Sauvegarde Azure gère les points de récupération pour les machines virtuelles
ms.topic: conceptual
ms.date: 11/08/2020
ms.openlocfilehash: 256df693aba0f799c24bcba6defe846e5c37ccaa
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94428474"
---
# <a name="manage-recovery-points"></a>Gérer les points de récupération

Cet article décrit le fonctionnement de la rétention pour les machines virtuelles. Chaque fois que des sauvegardes se produisent, des points de récupération sont créés à partir desquels les opérations de restauration peuvent être effectuées.

Pour les machines virtuelles, la sauvegarde initiale est une sauvegarde complète et les sauvegardes ultérieures sont des sauvegardes incrémentielles.

## <a name="recovery-points-and-retention"></a>Points de récupération et rétention

### <a name="scheduled-initial-and-incremental-backup"></a>Sauvegarde initiale et incrémentielle planifiée

Prenons un exemple simplifié de la machine virtuelle *V1* avec un disque de données composé de quatre blocs : Bloc 1, bloc 2, bloc 3 et bloc 4. Chaque bloc a une taille de 16 Ko.

![Machine virtuelle avec quatre blocs](./media/manage-recovery-points/four-blocks.png)

**Étape 1 - Sauvegarde initiale :** La sauvegarde initiale est une sauvegarde complète. Elle fonctionne comme une ligne de base sur laquelle les sauvegardes incrémentielles suivantes sont appliquées. Supposons que des données soient écrites dans les blocs 1 et 2 sur la machine virtuelle source. Les mêmes données seront répliquées en tant que D1 et D2 sur le stockage du coffre Recovery Services.

![La sauvegarde initiale est répliquée](./media/manage-recovery-points/initial-backup.png)

**Étape 2 - Sauvegarde incrémentielle 1 :** Tenez compte du fait que de nouvelles données ont été ajoutées au bloc 3 de la machine virtuelle. Les mêmes données seront répliquées lors de la sauvegarde incrémentielle suivante, et seul le bloc modifié est stocké en tant que D3.  Au cours de chaque étape, même si 1 Ko du bloc change, l’intégralité du bloc de 16 Ko est chargée dans le point de récupération.

![Première sauvegarde incrémentielle](./media/manage-recovery-points/first-incremental-backup.png)

**Étape 3 - Sauvegarde incrémentielle 2 :**  Considérons à présent que les données sont modifiées sur les blocs 3 et 2 sur la machine virtuelle source. Ces modifications seront répliquées lors de la sauvegarde incrémentielle suivante en tant que D3’ et D2’.

![Deuxième sauvegarde incrémentielle](./media/manage-recovery-points/second-incremental-backup.png)

### <a name="on-demand-backup"></a>Sauvegarde à la demande

Vous pouvez choisir d’exécuter une sauvegarde à la demande d’une machine virtuelle à tout moment une fois que vous avez configuré sa protection.

- La sauvegarde à la demande sera une sauvegarde complète si elle est déclenchée avant la première sauvegarde planifiée initiale.
- Si la sauvegarde initiale est terminée et qu’une sauvegarde à la demande est déclenchée, il s’agit d’une sauvegarde incrémentielle.
- La durée de rétention des points de récupération créés pour une sauvegarde à la demande correspond à la valeur de rétention que vous spécifiez quand vous déclenchez la sauvegarde.

### <a name="storage-cost"></a>Coût de stockage

Le **point de récupération** créé pour la sauvegarde initiale contient tous les blocs qui reprennent les données. Les points de récupération incrémentiels suivants sont constitués uniquement des blocs qui contiennent des données modifiées. Les coûts de stockage correspondent à la somme de tous les blocs répartis sur tous les points de récupération.

Nous allons utiliser l’exemple ci-dessus pour comprendre le coût de stockage après chaque étape :

|Étape  |Type de sauvegarde  |Blocs modifiés  |Type de stockage |
|------|---------|---------|---------|
|1     |     Sauvegarde initiale    | Bloc 1, bloc 2        |    Correspondant au point de récupération 1 (D1+D2)     |
|2     |  Sauvegarde incrémentielle 1       |  Bloc 3       |   Correspondant au point de récupération 1 (D1+D2) + point de récupération 2 (D3)      |
|3     |    Sauvegarde incrémentielle 2     |    Bloc 2, bloc 3     |   Correspondant au point de récupération 1 (D1+D2) + point de récupération 2 (D3) + point de récupération 3 (D2’+D3’)      |

### <a name="recovery-point-expiration"></a>Expiration du point de récupération

À chaque point de récupération correspond une durée de rétention, comme spécifié dans la stratégie de sauvegarde. Le nettoyage se produit à intervalles réguliers, et tous les points de récupération expirés sont nettoyés.

Lorsque le point de récupération expire, il est supprimé ou fusionné.

### <a name="case-1-initial-recovery-point-expires"></a>Cas n° 1 : Expiration du point de récupération initial

Lorsque le point de récupération initial expire, il fusionne avec le point de récupération incrémentiel suivant. Tous les blocs de données qui sont remplacés dans le point de récupération incrémentiel sont supprimés, et le reste est fusionné. La sauvegarde incrémentielle devient alors la sauvegarde complète initiale. Prenons un exemple :

- Le *Point de récupération 1* qui est créé lors de la sauvegarde initiale contient la sauvegarde complète de la machine virtuelle.
- Lorsque le *Point de récupération 1* expire, le *Point de récupération 2* est la sauvegarde complète suivante.
- Le bloc D1 est fusionné avec le *Point de récupération 2* et D2 est supprimé, car les données du bloc 2 sont remplacées dans *Point de récupération 2*. Cette modification est capturée en tant que bloc D2’.
- Le bloc D1 est conservé comme tel dans les points de récupération consécutifs, jusqu’à ce que des modifications y aient été apportées avant la sauvegarde suivante.

![Premier cas](./media/manage-recovery-points/first-case.png)

### <a name="case-2-in-between-incremental-recovery-point-expires"></a>Cas n° 2 : Expiration du point de récupération incrémentiel intermédiaire

- Si le *Point de récupération 2* expire avant le *Point de récupération 1*, les données du *Point de récupération 2* sont fusionnées avec le point de récupération disponible suivant : *Point de récupération 3*. Le bloc D3 est donc fusionné avec le *Point de récupération 3*.
- Le *Point de récupération 1* est toujours la sauvegarde complète avec les blocs D1 et D2.

![Deuxième cas](./media/manage-recovery-points/second-case.png)

### <a name="case-3-on-demand-recovery-point-expires"></a>Cas n° 3 : Expiration du point de récupération à la demande

Dans cet exemple, une stratégie de planification (sauvegarde quotidienne) est planifiée pour s’exécuter avec une période de rétention de *n* jours.  Si une sauvegarde à la demande est déclenchée le quatrième jour précédant la prochaine sauvegarde planifiée et que sa période de rétention est définie sur 10 jours, il s’agira toujours d’une sauvegarde incrémentielle. Un point de récupération (*RP1 à la demande*) sera créé après le *Point de récupération 3* et avant le *Point de récupération 4*.  À la fin de la journée 14, le point de récupération à la demande (*RP1 à la demande*) est arrivé à expiration et sera fusionné avec le point de récupération disponible suivant. Les blocs de données toujours présents sur le serveur sont fusionnés, tandis que ceux qui ont été modifiés (remplacés ou supprimés) sont supprimés du point de récupération expiré.

![Troisième cas de figure :](./media/manage-recovery-points/third-case.png)

### <a name="impact-of-policy-change-on-recovery-points"></a>Impact du changement de stratégie sur les points de récupération

Lorsqu’une stratégie est modifiée, elle est appliquée aux points de récupération nouveaux et existants. Pour plus d’informations, consultez [Impact du changement de stratégie sur les points de récupération](backup-architecture.md#impact-of-policy-change-on-recovery-points).

### <a name="impact-of-stop-protection-on-recovery-points"></a>Impact de l’arrêt de la protection sur les points de récupération

Il existe deux façons d’arrêter la protection d’une machine virtuelle :

- **Arrêtez la protection et supprimez les données de sauvegarde.** Cette option empêche toutes les futures tâches de sauvegarde de protéger votre machine virtuelle et supprime tous les points de récupération. Si la [suppression réversible](backup-azure-security-feature-cloud.md) est activée, les données supprimées sont conservées pendant 14 jours. Des frais ne sont pas facturés pour les éléments avec la suppression réversible activée. Les données peuvent être supprimées au cours de la période de 14 jours. Si la suppression réversible n’est pas activée, les données sont immédiatement nettoyées et vous ne pourrez pas restaurer la machine virtuelle ni utiliser l’option **Reprendre la sauvegarde**.
- **Arrêtez la protection et conservez les données de sauvegarde.** Avec cette option, tous les futurs travaux de sauvegarde cesseront de protéger votre machine virtuelle. Cependant, le service Sauvegarde Azure conservera pour toujours les points de récupération qui ont été sauvegardés. Vous devrez payer pour conserver ces points de récupération dans le coffre (consultez l’article [Tarification de Sauvegarde Azure](https://azure.microsoft.com/pricing/details/backup/) pour en savoir plus). Vous serez en mesure de restaurer la machine virtuelle si nécessaire. Si vous décidez de reprendre la protection des machines virtuelles, vous pourrez utiliser l’option **Reprendre la sauvegarde**. Une fois la sauvegarde reprise, les règles de rétention sont appliquées aux points d’expiration. Vous pouvez également supprimer les données sauvegardées à l’aide de l’option  **Supprimer les données de sauvegarde**.

## <a name="impact-of-deleting-a-vm-without-stop-protection"></a>Impact de la suppression d’une machine virtuelle sans interruption de la protection

La suppression d’une machine virtuelle sans interruption de la protection a un impact sur les points de récupération et constitue un scénario indésirable. Idéalement, les sauvegardes doivent être arrêtées avant la suppression de la machine virtuelle. Étant donné que la ressource n’existe pas, les sauvegardes planifiées échouent avec l’[erreur VMNotFoundV2](backup-azure-vms-troubleshoot.md#320001-resourcenotfound---could-not-perform-the-operation-as-vm-no-longer-exists--400094-bcmv2vmnotfound---the-virtual-machine-doesnt-exist--an-azure-virtual-machine-wasnt-found). Les points de récupération seront nettoyés régulièrement selon la stratégie de rétention, mais la dernière copie de la machine virtuelle ne disparaîtra jamais, et vous serez facturé en conséquence. Selon votre scénario, vous avez les deux options suivantes :

- **Option 1 :** Restaurez la machine virtuelle à l’aide de l’un des points de récupération. Si vous voulez récupérer la machine virtuelle supprimée, restaurez-la en utilisant le même nom et en restant dans le même groupe de ressources. Si vous protégez la machine virtuelle restaurée dans le même coffre, les points de récupération existants sont automatiquement joints.
- **Option 2 :** Accédez au coffre Recovery Services et arrêtez la protection pour les données supprimées.

### <a name="impact-of-expired-recovery-points-for-items-in-soft-deleted-state"></a>Impact des points de récupération arrivés à expiration pour les éléments avec suppression réversible

Si la [suppression réversible](backup-azure-security-feature-cloud.md) est activée pour le coffre Recovery Services, ce statut demeure pour le point de récupération expiré et ce dernier n’est pas nettoyé. Aucuns frais ne sont facturés lorsque la suppression réversible est activée pour un point de récupération.

### <a name="impact-of-churn-on-backup-performance"></a>Impact de l’activité sur les performances de sauvegarde

Supposons que le stockage total d’une machine virtuelle est de 8 To et que l’activité est de 5 %. Le stockage de sauvegarde incrémentielle correspondant s’élèvera alors à 5 % de 8 To, soit 0,4 To. Une plus grande activité correspond à un stockage principal plus élevé pour les sauvegardes incrémentielles suivantes. L’activité a un impact sur les performances de sauvegarde. Plus l’activité est élevée, plus le processus de sauvegarde est lent et plus la consommation de stockage back-end est importante.

Pour comprendre la façon dont l’activité affecte les performances de sauvegarde, consultez ce scénario :

|Machines virtuelles  |MV1  |MV2  |MV3  |
|---------|---------|---------|---------|
|Nombre de disques de données    | 4 (A1, A2, A3, A4)        | 4 (B1, B2, B3, B4)        |  4 (C1, C2, C3, C4)       |
|Taille de chaque disque   |      4 To   | 4 To        |  4 To       |
|Activité des données de sauvegarde    |   A1- 4 To      | B1-1 To ; B2-1 To <br> B3-1 To ; B4-1 To  |   C1-2 To ; C4-2 To      |

Les performances de sauvegarde seront dans l’ordre suivant : VM2 > VM3 > VM1. Cela est dû au fait que les données hautement évolutives sont réparties entre les différents disques. Étant donné que la sauvegarde des disques s’effectue en parallèle, VM2 affiche les meilleures performances.

## <a name="next-steps"></a>Étapes suivantes

- [Architecture et composants d’Azure Backup](backup-architecture.md)
