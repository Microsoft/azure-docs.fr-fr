---
title: Comment déplacer des coffres Azure Backup Recovery Services
description: Instructions pour déplacer un coffre Recovery Services entre des abonnements Azure et des groupes de ressources.
ms.topic: conceptual
ms.date: 04/08/2019
ms.custom: references_regions
ms.openlocfilehash: 12c276b861e7db8e93e60eea7e9cd7f3aba04860
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325772"
---
# <a name="move-a-recovery-services-vault-across-azure-subscriptions-and-resource-groups"></a>Déplacer un coffre Recovery Services entre des abonnements Azure et des groupes de ressources

Cet article explique comment déplacer un coffre Recovery Services configuré pour Sauvegarde Azure entre des abonnements Azure ou vers un autre groupe de ressources dans le même abonnement. Vous pouvez utiliser le portail Azure ou PowerShell pour déplacer un coffre Recovery Services.

## <a name="supported-regions"></a>Régions prises en charge

Toutes les régions publiques et souveraines sont prises en charge, à l’exception des suivantes : France Centre, France Sud, Allemagne Nord-Est, Allemagne Centre, Chine Nord, Chine Nord 2, Chine Est et Chine Est 2.

## <a name="prerequisites-for-moving-recovery-services-vault"></a>Conditions préalables au déplacement d'un coffre Recovery Services

- Lors du déplacement du coffre entre des groupes de ressources, les groupes de ressources source et cible sont verrouillés pour empêcher les opérations d'écriture et de suppression. Pour plus d’informations, consultez cet [article](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Seul l'abonnement Administrateur dispose des autorisations pour déplacer un coffre.
- Pour déplacer des coffres entre des abonnements, il faut que l’abonnement cible se trouve dans le même locataire que l’abonnement source et son état doit être défini sur Activé.
- Vous devez être autorisé à effectuer des opérations d’écriture sur le groupe de ressources cible.
- Le déplacement du coffre change seulement le groupe de ressources. Le coffre Recovery Services se trouve au même endroit et ne peut pas être changé.
- Vous ne pouvez déplacer qu'un seul coffre Recovery Services à la fois par région.
- Si une machine virtuelle n’est pas déplacée avec le coffre Recovery Services entre des abonnements ou vers un nouveau groupe de ressources, les points de récupération actuels de la machine virtuelle restent intacts dans le coffre jusqu’à leur expiration.
- Que la machine virtuelle soit déplacée ou non avec le coffre, vous pouvez toujours la restaurer à partir de l’historique des sauvegardes conservé dans le coffre.
- Azure Disk Encryption exige que le coffre de clés et les machines virtuelles se trouvent dans la même région et le même abonnement Azure.
- Pour déplacer une machine virtuelle avec des disques managés, consultez cet [article](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Les options de déplacement des ressources déployées avec le modèle classique diffèrent selon que vous déplacez les ressources au sein d’un abonnement ou vers un nouvel abonnement. Pour plus d’informations, consultez cet [article](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Les stratégies de sauvegarde définies pour le coffre sont conservées après le déplacement du coffre entre des abonnements ou vers un groupe de ressources.
- Vous pouvez uniquement déplacer un coffre contenant l’un des types d’éléments de sauvegarde suivants. Vous devez arrêter les éléments de sauvegarde de types non répertoriés ci-dessous et supprimer les données définitivement avant le déplacement du coffre.
  - Machines virtuelles Azure
  - Agent Microsoft Azure Recovery Services (MARS)
  - Serveur Sauvegarde Microsoft Azure (MABS)
  - Data Protection Manager (DPM)
- Si vous déplacez un coffre contenant des données de sauvegarde de machine virtuelle entre des abonnements, vous devez déplacer vos machines virtuelles dans le même abonnement et utiliser le même nom de groupe de ressources de machine virtuelle cible (celui de l’ancien abonnement) pour continuer les sauvegardes.

> [!NOTE]
> Le déplacement de coffres Recovery Services pour Sauvegarde Azure d’une région Azure à une autre n’est pas pris en charge.<br><br>
> Si vous avez configuré des machines virtuelles (IaaS Azure, Hyper-V, VMware) ou des machines physiques pour la récupération d’urgence avec **Azure Site Recovery**, l’opération de déplacement est bloquée. Si vous souhaitez déplacer des coffres pour Azure Site Recovery, consultez [cet article](../site-recovery/move-vaults-across-regions.md) pour en savoir plus sur le déplacement manuel des coffres.

## <a name="use-azure-portal-to-move-recovery-services-vault-to-different-resource-group"></a>Utiliser le portail Azure pour déplacer un coffre Recovery Services vers un autre groupe de ressources

Pour déplacer un coffre Recovery Services et ses ressources associées vers un autre groupe de ressources :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Ouvrez la liste des **coffres Recovery Services** et sélectionnez le coffre à déplacer. Quand le tableau de bord du coffre s’ouvre, il s’affiche comme illustré dans l’image suivante.

   ![Ouvrir le coffre Recovery Services](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

   Si vous ne voyez pas les informations **Essentials** pour votre coffre, sélectionnez l’icône de liste déroulante. Vous devez maintenant voir les informations Essentials pour votre coffre.

   ![Onglet Informations Essentials](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. Dans le menu Vue d’ensemble du coffre, sélectionnez **Modifier** à côté du **groupe de ressources** pour ouvrir le volet **Déplacer des ressources**.

   ![Changer le groupe de ressources](./media/backup-azure-move-recovery-services/change-resource-group.png)

4. Dans le volet **Déplacer des ressources**, pour le coffre sélectionné, il est recommandé de déplacer les ressources associées facultatives en cochant la case comme illustré dans l’image suivante.

   ![Déplacer un abonnement](./media/backup-azure-move-recovery-services/move-resource.png)

5. Pour ajouter le groupe de ressources cible, dans la liste déroulante **Groupe de ressources**, sélectionnez un groupe de ressources existant ou l’option **Créer un groupe**.

   ![Créer une ressource](./media/backup-azure-move-recovery-services/create-a-new-resource.png)

6. Après avoir ajouté le groupe de ressources, confirmez l’option **Je comprends que les outils et les scripts associés aux ressources déplacées ne fonctionnent pas tant que je ne les mets pas à jour pour utiliser de nouveaux ID de ressource**, puis sélectionnez **OK** pour effectuer le déplacement du coffre.

   ![Message de confirmation](./media/backup-azure-move-recovery-services/confirmation-message.png)

## <a name="use-azure-portal-to-move-recovery-services-vault-to-a-different-subscription"></a>Utiliser le portail Azure pour déplacer un coffre Recovery Services vers un autre abonnement

Vous pouvez déplacer un coffre Recovery Services et ses ressources associées vers un autre abonnement.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Ouvrez la liste des coffres Recovery Services et sélectionnez le coffre à déplacer. Quand le tableau de bord du coffre s’ouvre, il apparaît comme illustré dans l’image suivante.

    ![Ouvrir le coffre Recovery Services](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

    Si vous ne voyez pas les informations **Essentials** pour votre coffre, sélectionnez l’icône de liste déroulante. Vous devez maintenant voir les informations Essentials pour votre coffre.

    ![Onglet Informations Essentials](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. Dans le menu Vue d’ensemble du coffre, sélectionnez **Modifier** à côté d’**Abonnement**  pour ouvrir le volet **Déplacer des ressources**.

   ![Changer l’abonnement](./media/backup-azure-move-recovery-services/change-resource-subscription.png)

4. Sélectionnez les ressources à déplacer : nous vous recommandons d’utiliser ici l’option **Sélectionner tout** pour sélectionner toutes les ressources facultatives listées.

   ![Déplacer une ressource](./media/backup-azure-move-recovery-services/move-resource-source-subscription.png)

5. Dans la liste déroulante **Abonnement**, sélectionnez l’abonnement cible où vous voulez déplacer le coffre.
6. Pour ajouter le groupe de ressources cible, dans la liste déroulante **Groupe de ressources**, sélectionnez un groupe de ressources existant ou l’option **Créer un groupe**.

   ![Ajouter un abonnement](./media/backup-azure-move-recovery-services/add-subscription.png)

7. Sélectionnez l’option **Je comprends que les outils et les scripts associés aux ressources déplacées ne fonctionnent pas tant que je ne les mets pas à jour pour utiliser de nouveaux ID de ressource;** , puis sélectionnez **OK**.

> [!NOTE]
> La sauvegarde entre abonnements (le coffre RS et les machines virtuelles protégées se trouvent dans des abonnements différents) n’est pas un scénario pris en charge. En outre, pendant l’opération de déplacement du coffre, vous ne pouvez pas modifier l’option de redondance du stockage de stockage localement redondant (LRS) en stockage globalement redondant (GRS) et inversement.
>
>

## <a name="use-powershell-to-move-recovery-services-vault"></a>Utiliser PowerShell pour déplacer un coffre Recovery Services

Pour déplacer un coffre Recovery Services vers un autre groupe de ressources, utilisez l’applet de commande `Move-AzureRMResource`. `Move-AzureRMResource` nécessite le nom de la ressource et le type de ressource. Vous pouvez les obtenir avec l’applet de commande `Get-AzureRmRecoveryServicesVault`.

```powershell
$destinationRG = "<destinationResourceGroupName>"
$vault = Get-AzureRmRecoveryServicesVault -Name <vaultname> -ResourceGroupName <vaultRGname>
Move-AzureRmResource -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Pour déplacer les ressources vers un autre abonnement, incluez le paramètre `-DestinationSubscriptionId`.

```powershell
Move-AzureRmResource -DestinationSubscriptionId "<destinationSubscriptionID>" -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Après l’exécution des applets de commande ci-dessus, vous devez confirmer que vous voulez déplacer les ressources spécifiées. Tapez **O** pour le confirmer. Après une validation réussie, la ressource est déplacée.

## <a name="use-cli-to-move-recovery-services-vault"></a>Utiliser l'interface CLI pour déplacer un coffre Recovery Services

Pour déplacer un coffre Recovery Services vers un autre groupe de ressources, utilisez l’applet de commande suivante :

```azurecli
az resource move --destination-group <destinationResourceGroupName> --ids <VaultResourceID>
```

Pour déplacer des ressources vers un nouvel abonnement, spécifiez le paramètre `--destination-subscription-id`.

## <a name="post-migration"></a>Après la migration

1. Définissez/vérifiez les contrôles d’accès pour les groupes de ressources.  
2. La fonctionnalité de création de rapports et d’analyse de Sauvegarde doit être reconfigurée pour le coffre une fois le déplacement terminé. La configuration antérieure est perdue lors de l’opération de déplacement.

## <a name="move-an-azure-virtual-machine-to-a-different-recovery-service-vault"></a>Déplacement d’une machine virtuelle Azure vers un autre coffre Recovery Services 

Si vous souhaitez déplacer une machine virtuelle Azure sur laquelle la Sauvegarde Azure est activée, vous avez deux possibilités en fonction des besoins de votre entreprise :

- [Vous n’avez pas besoin de conserver les données de sauvegarde précédentes](#dont-need-to-preserve-previous-backed-up-data)
- [Vous devez conserver les données de sauvegarde précédentes](#must-preserve-previous-backed-up-data)

### <a name="dont-need-to-preserve-previous-backed-up-data"></a>Vous n’avez pas besoin de conserver les données de sauvegarde précédentes

Pour protéger les charges de travail dans un nouveau coffre, vous devez supprimer la protection et les données actuelles dans l’ancien coffre. La sauvegarde est reconfigurée.

>[!WARNING]
>L’opération suivante est destructrice et ne peut pas être annulée. Toutes les données de sauvegarde et tous les éléments de sauvegarde associés au serveur protégé seront définitivement supprimés. Procédez avec prudence.

**Arrêtez et supprimez la protection actuelle sur l’ancien coffre :**

1. Désactivez la suppression réversible dans les propriétés du coffre. Pour désactiver la suppression réversible, [procédez comme suit](backup-azure-security-feature-cloud.md#disabling-soft-delete-using-azure-portal).

2. Arrêtez la protection et supprimez les sauvegardes du coffre actuel. Dans le menu du tableau de bord du coffre, sélectionnez **Éléments de sauvegarde**. Les éléments présentés ici qui doivent être déplacés vers le nouveau coffre doivent être supprimés avec leurs données de sauvegarde. Voir comment [supprimer des éléments protégés dans le cloud](backup-azure-delete-vault.md#delete-protected-items-in-the-cloud) et [supprimer des éléments protégés sur site](backup-azure-delete-vault.md#delete-protected-items-on-premises).

3. Si vous envisagez de déplacer des partages de fichiers Azure, des serveurs SQL ou des serveurs SAP HANA, vous devrez également les désinscrire. Dans le menu du tableau de bord du coffre, sélectionnez **Infrastructure de sauvegarde**. Voir comment [désinscrire le serveur SQL](manage-monitor-sql-database-backup.md#unregister-a-sql-server-instance), [désinscrire un compte de stockage associé à des partages de fichiers Azure](manage-afs-backup.md#unregister-a-storage-account) et [désinscrire une instance SAP HANA](sap-hana-db-manage.md#unregister-an-sap-hana-instance).

4. Une fois qu’elles sont supprimées de l’ancien coffre, procédez à la configuration des sauvegardes de votre charge de travail dans le nouveau coffre.

### <a name="must-preserve-previous-backed-up-data"></a>Vous devez conserver les données de sauvegarde précédentes

Si vous devez conserver les données protégées actuelles dans l’ancien coffre et poursuivre la protection dans un nouveau coffre, les possibilités sont limitées pour certaines des charges de travail :

- Pour MARS, vous pouvez [arrêter la protection avec conservation des données](backup-azure-manage-mars.md#stop-protecting-files-and-folder-backup) et inscrire l’agent dans le nouveau coffre.

  - Le service Sauvegarde Azure conservera tous les points de récupération existants de l’ancien coffre.
  - Vous devrez payer pour conserver les points de récupération dans l’ancien coffre.
  - Vous ne pourrez restaurer les données sauvegardées que pour les points de récupération non expirés dans l’ancien coffre.
  - Un nouveau réplica initial des données devra être créé sur le nouveau coffre.

- Pour une machine virtuelle Azure, vous pouvez [arrêter la protection avec conservation des données](backup-azure-manage-vms.md#stop-protecting-a-vm) pour la machine virtuelle dans l’ancien coffre, déplacer la machine virtuelle vers un autre groupe de ressources, puis la protéger dans le nouveau coffre. Voir [les conseils et les limites](../azure-resource-manager/management/move-limitations/virtual-machines-move-limitations.md) pour déplacer une machine virtuelle vers un autre groupe de ressources.

  Une machine virtuelle ne peut être protégée que dans un coffre à la fois. Toutefois, la machine virtuelle du nouveau groupe de ressources peut être protégée sur le nouveau coffre, car elle est considérée comme une autre machine virtuelle.

  - Le service Sauvegarde Azure conservera les points de récupération sauvegardés dans l’ancien coffre.
  - Vous devrez payer pour conserver les points de récupération dans l’ancien coffre (pour plus d’informations, consultez [Tarifs de la Sauvegarde Azure](azure-backup-pricing.md)).
  - Vous pourrez si nécessaire restaurer la machine virtuelle à partir de l’ancien coffre.
  - La première sauvegarde effectuée sur le nouveau coffre de la machine virtuelle placée dans la nouvelle ressource sera un réplica initial.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez déplacer de nombreux types de ressources différents entre les groupes de ressources et les abonnements.

Pour plus d’informations, consultez la page [Déplacement de ressources vers un nouveau groupe de ressources ou un abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).