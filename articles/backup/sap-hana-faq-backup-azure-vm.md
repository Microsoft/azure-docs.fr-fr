---
title: FAQ - Sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure
description: Dans cet article, découvrez des réponses à des questions courantes sur la sauvegarde de bases de données SAP HANA avec le service Sauvegarde Microsoft Azure.
ms.topic: conceptual
ms.date: 11/7/2019
ms.openlocfilehash: bf662600bafcd18b00c8f8d3b673fc3f9c110aca
ms.sourcegitcommit: 1d366d72357db47feaea20c54004dc4467391364
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95400205"
---
# <a name="frequently-asked-questions--back-up-sap-hana-databases-on-azure-vms"></a>Forum aux questions – Sauvegarde de bases de données SAP HANA sur des machines virtuelles Azure

Cet article fournit des réponses à des questions courantes sur la sauvegarde de bases de données SAP HANA avec le service Sauvegarde Microsoft Azure.

## <a name="backup"></a>Sauvegarde

### <a name="how-many-full-backups-are-supported-per-day"></a>Combien de sauvegardes complètes sont-elles prises en charge par jour ?

Nous prenons en charge une seule sauvegarde complète par jour. Les sauvegardes différentielles et complètes ne peuvent pas être déclenchées le même jour.

### <a name="do-successful-backup-jobs-create-alerts"></a>La réussite des travaux de sauvegarde génère-t-elle des alertes ?

Non. Les travaux de sauvegarde réussis ne génèrent pas d’alertes. Les alertes ne sont envoyées qu’en cas d’échec de la sauvegarde. Le comportement détaillé des alertes de portail est détaillé [ici](./backup-azure-monitoring-built-in-monitor.md). Toutefois, si vous souhaitez obtenir des alertes même pour des travaux réussis, vous pouvez utiliser [Azure Monitor](./backup-azure-monitoring-use-azuremonitor.md).

### <a name="can-i-see-scheduled-backup-jobs-in-the-backup-jobs-menu"></a>Les travaux de sauvegarde planifiés sont-ils affichés dans le menu Travaux de sauvegarde ?

Le menu Travail de sauvegarde affiche uniquement les travaux de sauvegarde à la demande. Pour les tâches planifiées, utilisez [Azure Monitor](./backup-azure-monitoring-use-azuremonitor.md).

### <a name="are-future-databases-automatically-added-for-backup"></a>Les bases de données futures sont-elles automatiquement ajoutées pour la sauvegarde ?

Non, cette opération n’est pas prise en charge actuellement.

### <a name="if-i-delete-a-database-from-an-instance-what-will-happen-to-the-backups"></a>Si je supprime une base de données d’une instance, qu’advient-il des sauvegardes ?

Si une base de données est supprimée d’une instance SAP HANA, des tentatives de sauvegarde de cette base de données sont toujours effectuées. Cela implique que la base de données supprimée devient défectueuse sous **Éléments de sauvegarde** mais reste protégée.
Pour ne plus protéger cette base de données, utilisez **Arrêter la sauvegarde avec Supprimer des données** sur cette base de données.

### <a name="if-i-change-the-name-of-the-database-after-it-has-been-protected-what-will-the-behavior-be"></a>Si je modifie le nom de la base de données après qu’elle a été protégée, comment se comportera-t-elle ?

Une base de données renommée est traitée en tant que nouvelle base de données. Par conséquent, le service gère cette situation comme si la base de données était introuvable et les sauvegardes échouent. La base de données renommée s’affiche sous la forme d’une nouvelle base de données et doit être configurée pour la protection.

### <a name="what-are-the-prerequisites-to-back-up-sap-hana-databases-on-an-azure-vm"></a>Quelles sont les conditions préalables à la sauvegarde des bases de données SAP HANA sur une machine virtuelle Azure ?

Reportez-vous aux sections [Prérequis](tutorial-backup-sap-hana-db.md#prerequisites) et [Ce que fait le script de préinscription](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does).

### <a name="what-permissions-should-be-set-so-azure-can-back-up-sap-hana-databases"></a>Quelles autorisations doivent être définies pour qu’Azure puisse sauvegarder les bases de données SAP HANA ?

L’exécution du script de pré-inscription définit les autorisations requises pour permettre à Azure de sauvegarder des bases de données SAP HANA. Pour plus d’informations sur ce qu’un script de préinscription permet de faire, [cliquez ici](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does).

### <a name="will-backups-work-after-migrating-sap-hana-from-sdc-to-mdc"></a>Les sauvegardes seront-elles opérationnelles après la migration SAP HANA de SDC vers MDC ?

Reportez-vous à [cette section](./backup-azure-sap-hana-database-troubleshoot.md#sdc-to-mdc-upgrade-with-a-change-in-sid) du Guide de résolution des problèmes.

### <a name="what-should-be-done-while-upgrading-within-the-same-version"></a>Que dois-je faire lors d’une mise à niveau dans la même version ?

Reportez-vous à [cette section](backup-azure-sap-hana-database-troubleshoot.md#sdc-version-upgrade-or-mdc-version-upgrade-on-the-same-vm) du Guide de résolution des problèmes.

### <a name="can-azure-hana-backup-be-set-up-against-a-virtual-ip-load-balancer-and-not-a-virtual-machine"></a>La sauvegarde Azure HANA peut-elle être configurée sur une adresse IP virtuelle (équilibreur de charge) et non une machine virtuelle ?

Nous ne sommes pas en mesure de configurer la solution sur une adresse IP virtuelle seule pour le moment. Une machine virtuelle est nécessaire pour exécuter la solution.

### <a name="how-can-i-move-an-on-demand-backup-to-the-local-file-system-instead-of-the-azure-vault"></a>Comment puis-je déplacer une sauvegarde à la demande vers le système de fichiers local au lieu du coffre Azure ?

1. Attendez la fin de la sauvegarde en cours d’exécution sur la base de données souhaitée (vérifiez son accomplissement dans Studio).
1. Désactivez les sauvegardes de journaux et définissez la sauvegarde du catalogue sur **Filesystem** pour la base de données souhaitée en procédant comme suit :
1. Double-cliquez sur **SYSTEMDB** -> **configuration** -> **Sélectionner une base de données** -> **Filtre (journal)**
    1. Définissez enable_auto_log_backup sur **no** (non).
    1. Définissez catalog_backup_using_backint sur **false**.
1. Effectuez une sauvegarde à la demande (complète / différentielle/ incrémentielle) sur la base de données souhaitée et attendez la fin de la sauvegarde et de la sauvegarde du catalogue.
1. Si vous souhaitez également déplacer les sauvegardes du journal vers le système de fichiers, définissez enable_auto_log_backup sur **yes** (oui).
1. Rétablissez les paramètres précédents pour autoriser la circulation des sauvegardes vers le coffre Azure :
    1. Définissez enable_auto_log_backup sur **yes** (oui).
    1. Définissez catalog_backup_using_backint sur **true**.

>[!NOTE]
>Le déplacement des sauvegardes vers le système de fichiers local et le retour vers le coffre Azure peuvent entraîner une rupture de la chaîne de journalisation des sauvegardes de journaux dans le coffre. Cela déclenche une sauvegarde complète qui, une fois terminée, démarre la sauvegarde des journaux.

### <a name="how-can-i-use-sap-hana-backup-with-my-hana-replication-set-up"></a>Comment utiliser la sauvegarde SAP HANA avec la configuration de la réplication HANA ?

Actuellement, la sauvegarde Azure n’a pas la possibilité de comprendre une configuration HSR. Cela signifie que les nœuds principal et secondaire de la réplication HSR seront traités comme deux machines virtuelles indépendantes. Vous devez d’abord configurer la sauvegarde sur le nœud principal. Lorsqu’un basculement se produit, la sauvegarde doit être configurée sur le nœud secondaire (qui devient alors le nœud principal). Il n’y a pas de basculement automatique de la sauvegarde vers l’autre nœud.

Pour sauvegarder des données à partir du nœud actif (principal) à un moment donné, vous pouvez **basculer la protection** vers le nœud secondaire, qui devient alors le nœud principal.

Pour effectuer ce **basculement de protection**, procédez comme suit :

- [Arrêtez la protection](sap-hana-db-manage.md#stop-protection-for-an-sap-hana-database) (avec conservation des données) sur le nœud principal
- Exécutez le [script de pré-inscription](https://aka.ms/scriptforpermsonhana) sur le nœud secondaire
- [Détectez les bases de données](tutorial-backup-sap-hana-db.md#discover-the-databases) sur le nœud secondaire et [configurez les sauvegardes](tutorial-backup-sap-hana-db.md#configure-backup) sur celles-ci

Ces étapes doivent être effectuées manuellement après chaque basculement. Vous pouvez effectuer ces étapes via la ligne de commande / HTTP REST en plus du portail Azure. Pour automatiser ces étapes, vous pouvez utiliser un runbook Azure.

Voici un exemple détaillé de la façon dont le **basculement de protection** doit être effectué :

Dans cet exemple, vous avez deux nœuds – le nœud 1 (principal) et le nœud 2 (secondaire) dans la configuration HSR.  Les sauvegardes sont configurées sur le nœud 1. Comme indiqué ci-dessus, ne tentez pas encore de configurer des sauvegardes sur le nœud 2.

Lorsque le premier basculement se produit, le nœud 2 devient le nœud principal. Ainsi,

1. Arrêtez la protection du nœud 1 (principal précédent) avec l’option de conservation des données.
1. Exécutez le script de pré-inscription sur le nœud 2 (qui est désormais le nœud principal).
1. Découvrez les bases de données sur le nœud 2, affectez la stratégie de sauvegarde et configurez les sauvegardes.

Ensuite, une première sauvegarde complète est déclenchée sur le nœud 2 et, une fois celle-ci terminée, les sauvegardes de fichiers journaux démarrent.

Lorsque le basculement suivant se produit, le nœud 1 redevient principal et le nœud 2 devient secondaire. À présent, répétez le processus :

1. Arrêtez la protection du nœud 2 avec l’option de conservation des données.
1. Exécutez le script de pré-inscription sur le nœud 1 (qui est redevenu principal).
1. Ensuite, [reprenez la sauvegarde](sap-hana-db-manage.md#resume-protection-for-an-sap-hana-database) sur le nœud 1 avec la stratégie requise (car les sauvegardes ont été arrêtées plus tôt sur le nœud 1).

Ensuite, une sauvegarde complète est déclenchée de nouveau sur le nœud 1 et, une fois celle-ci terminée, les sauvegardes de fichiers journaux démarrent.

## <a name="restore"></a>Restaurer

### <a name="why-cant-i-see-the-hana-system-i-want-my-database-to-be-restored-to"></a>Pourquoi ne puis-je pas voir le système HANA sur lequel ma base de données doit être restaurée ?

Vérifiez que toutes les conditions préalables pour la restauration vers la cible de l’instance SAP HANA sont remplies. Pour plus d’informations, consultez [Conditions préalables - Restaurer des bases de données SAP HANA dans une machine virtuelle Azure](./sap-hana-db-restore.md#prerequisites).

### <a name="why-is-the-overwrite-db-restore-failing-for-my-database"></a>Pourquoi la restauration du remplacement de la base de données a-t-elle échoué pour ma base de données ?

Assurez-vous que l’option **Forcer le remplacement** est sélectionnée lors de la restauration.

### <a name="why-do-i-see-the-source-and-target-systems-for-restore-are-incompatible-error"></a>Pourquoi l’erreur : « Les systèmes source et cible pour la restauration ne sont pas compatibles » apparaît-elle ?

Reportez-vous à la note SAP HANA [1642148](https://launchpad.support.sap.com/#/notes/1642148) pour voir quels types de restauration sont actuellement pris en charge.

### <a name="can-i-use-a-backup-of-a-database-running-on-sles-to-restore-to-an-rhel-hana-system-or-vice-versa"></a>Est-ce que je peux utiliser une sauvegarde d’une base de données en cours d’exécution sur SLES pour effectuer une restauration vers un système RHEL HANA ou vice versa ?

Oui, vous pouvez utiliser des sauvegardes en continu déclenchées sur une base de données HANA en cours d’exécution sur SLES pour la restaurer vers un système RHEL HANA, et vice versa. Autrement dit, la restauration entre systèmes d’exploitation est possible à l’aide de sauvegardes en continu. Toutefois, vous devez vous assurer que le système HANA vers lequel vous souhaitez restaurer et le système HANA utilisé pour la restauration sont tous deux compatibles pour la restauration en fonction de SAP. Pour connaître les types de restauration compatibles, consultez la Note SAP HANA [1642148](https://launchpad.support.sap.com/#/notes/1642148).

## <a name="policy"></a>Stratégie

### <a name="different-options-available-during-creation-of-a-new-policy-for-sap-hana-backup"></a>Différentes options disponibles lors de la création d’une stratégie pour la sauvegarde SAP HANA

Avant de créer une stratégie, vous devez connaître les exigences en matière de RPO et de RTO, ainsi que ses implications en termes de coûts.

Le RPO (objectif de point de récupération) indique la quantité de perte de données acceptable pour l’utilisateur ou le client. Celle-ci est déterminée par la fréquence de sauvegarde du journal. Des sauvegardes de fichier journal plus fréquentes indiquent un RPO inférieur et la valeur minimale que prend en charge le service Sauvegarde Azure est de 15 minutes. La fréquence de sauvegarde de fichier journal peut être de 15 minutes ou plus.

Le RTO (objectif de temps de récupération) indique la vitesse à laquelle les données doivent être restaurées sur le dernier point dans le temps disponible après un scénario de perte de données. Celle-ci dépend de la stratégie de récupération employée par HANA, qui dépend généralement du nombre de fichiers requis pour la restauration. Elle a également des implications financières et le tableau suivant doit vous aider à comprendre tous les scénarios et leurs implications.

|Stratégie de sauvegarde  |RTO  |Coût  |
|---------|---------|---------|
|Sauvegarde quotidienne complète + journaux     |   Option la plus rapide, puisque nous n’avons besoin que d’une seule copie complète et des journaux requis pour une restauration à un instant dans le passé      |    Option la plus coûteuse puisqu’une copie complète est effectuée quotidiennement, de sorte que de plus en plus de données sont accumulées dans le back-end jusqu’à la fin de la période de rétention   |
|Sauvegarde hebdomadaire complète + sauvegarde différentielle quotidienne + journaux     |   Option plus lente que l’option ci-dessus, mais plus rapide que l’option suivante puisque nous n’avons besoin que d’une copie complète, d’une copie différentielle et de fichiers journaux pour une restauration à un instant dans le passé      |    Option moins coûteuse puisque la sauvegarde différentielle quotidienne est généralement de plus petite taille que la sauvegarde complète et qu’une copie complète n’est effectuée qu’une fois par semaine      |
|Sauvegarde hebdomadaire complète + sauvegarde incrémentielle quotidienne + journaux     |  Option la plus lente, car nous avons besoin d’une copie complète, de « n » sauvegardes incrémentielles et de fichiers journaux pour la restauration à un instant dans le passé       |     Option la moins coûteuse puisque la sauvegarde incrémentielle quotidienne est de plus petite taille que la sauvegarde différentielle et qu’une copie complète n’est effectuée qu’une fois par semaine    |

> [!NOTE]
> Les options ci-dessus sont les plus courantes, mais ne sont pas les seules. Par exemple, vous pouvez avoir une sauvegarde hebdomadaire complète, des sauvegardes différentielles deux fois par semaine et des journaux.

Par conséquent, vous pouvez sélectionner la variante de stratégie en fonction d’objectifs de RPO et de RTO et de considérations financières.

### <a name="impact-of-modifying-a-policy"></a>Impact de la modification d’une stratégie

Il convient de prendre en compte quelques principes lors de la détermination de l’impact du changement de stratégie de sauvegarde d’un élément, passant de la stratégie 1 (P1) à la stratégie 2 (P2), ou de la modification de la stratégie 1 (P1).

- Toutes les modifications sont également appliquées de façon rétroactive. La stratégie de sauvegarde la plus récente est appliquée également aux points de récupération pris précédemment. Par exemple, supposons que la rétention de sauvegarde quotidienne complète est de 30 jours et que 10 points de récupération ont été pris conformément à la stratégie actuellement active. Si la rétention de la sauvegarde quotidienne complète est modifiée en 10 jours, l’heure d’expiration du point précédent est également recalculée comme l’heure de début + 10 jours, et supprimée si elle a expiré.
- L’étendue de la modification inclut également le jour de la sauvegarde, le type de sauvegarde, ainsi que la rétention. Exemple : Si une stratégie est modifiée de sauvegarde quotidienne complète en sauvegarde hebdomadaire complète le dimanche, toutes les sauvegardes complètes antérieures qui n’ont pas été effectuées un dimanche sont marquées pour suppression.
- Un parent n’est pas supprimé tant que l’enfant est actif ou n’a pas expiré. Chaque type de sauvegarde a une heure d’expiration conformément à la stratégie actuellement active. Toutefois, un type de sauvegarde complète est considéré comme parent des « sauvegardes différentielles », des « sauvegardes incrémentielles » et des « journaux » suivants. Une « sauvegarde différentielle » et un « journal » ne sont parents de rien. Une « sauvegarde incrémentielle » peut être parente d’une « sauvegarde incrémentielle » ultérieure. Même si un « parent » est marqué pour suppression, il n’est pas réellement supprimé si les « sauvegardes différentielles » ou les « journaux » enfants ne sont pas arrivés à expiration. Par exemple, si une stratégie est modifiée de sauvegarde quotidienne complète en sauvegarde hebdomadaire complète le dimanche, toutes les sauvegardes complètes antérieures qui n’ont pas été effectuées un dimanche sont marquées pour suppression. Mais elles ne sont pas réellement supprimées tant que les journaux quotidiens précédents n’ont pas expiré. En d’autres termes, elles sont conservées en fonction de la durée du dernier journal. Quand les journaux expirent, les journaux et les sauvegardes complètes sont supprimés.

Avec ces principes à l’esprit, vous pouvez lire le tableau suivant pour comprendre les implications d’une modification de stratégie.

|Ancienne stratégie / Nouvelle stratégie  |Sauvegardes quotidiennes complètes + journaux  | Sauvegardes hebdomadaires complètes + sauvegardes quotidiennes différentielles + journaux  |Sauvegardes hebdomadaires complètes + sauvegardes quotidiennes incrémentielles + journaux  |
|---------|---------|---------|---------|
|Sauvegardes quotidiennes complètes + journaux     |   -      |    Les sauvegardes complètes précédentes qui n’ont pas eu lieu le même jour de la semaine sont marquées pour suppression, mais conservées jusqu’à la fin de la période de rétention du journal     |    Les sauvegardes complètes précédentes qui n’ont pas eu lieu le même jour de la semaine sont marquées pour suppression, mais conservées jusqu’à la fin de la période de rétention du journal     |
|Sauvegardes hebdomadaires complètes + sauvegardes quotidiennes différentielles + journaux     |   La rétention des sauvegardes hebdomadaires complètes précédentes est recalculée conformément à la dernière stratégie. Les sauvegardes différentielles précédentes sont immédiatement supprimées      |    -     |    Les sauvegardes différentielles précédentes sont immédiatement supprimées     |
|Sauvegardes hebdomadaires complètes + sauvegardes quotidiennes incrémentielles + journaux     |     La rétention des sauvegardes hebdomadaires complètes précédentes est recalculée conformément à la dernière stratégie. Les sauvegardes incrémentielles précédentes sont immédiatement supprimées    |     Les sauvegardes incrémentielles précédentes sont immédiatement supprimées    |    -     |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [sauvegarder des bases de données SAP HANA](./backup-azure-sap-hana-database.md) qui s’exécutent sur des machines virtuelles Azure.
