---
title: Notifications de maintenance pour les groupes de machines virtuelles identiques dans Azure
description: Affichez les notifications de maintenance et démarrez la maintenance en libre-service pour les groupes de machines virtuelles identiques dans Azure.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 11/12/2020
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 2aa589d237a8cfeb8e0dc947896dba82e755631c
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94564767"
---
# <a name="planned-maintenance-notifications-for-virtual-machine-scale-sets"></a>Notifications de maintenance planifiées pour les groupes de machines virtuelles identiques


Azure exécute régulièrement des mises à jour afin d’améliorer la fiabilité, les performances et la sécurité de l’infrastructure hôte des machines virtuelles. Les mises à jour peuvent inclure la mise à jour corrective de l’environnement d’hébergement ou la mise à niveau et la désactivation de matériel. La plupart des mises à jour n’affectent pas les machines virtuelles hébergées. Toutefois, les mises à jour affectent les machines virtuelles dans les scénarios suivants :

- Si la maintenance ne nécessite pas de redémarrage, Azure met en pause la machine virtuelle pendant quelques secondes lors de la mise à jour de l’hôte. Ces types d’opérations de maintenance sont appliquées domaine d’erreur par domaine d’erreur. Elles sont arrêtées si des signaux d’avertissement sont reçus.

- Si la maintenance nécessite un redémarrage, une notification vous dira pour quand est prévue la maintenance. Dans ces cas, vous disposez d’une période qui s’étend généralement sur 35 jours pour commencer la maintenance vous-même, au moment qui vous convient.


La maintenance planifiée nécessitant un redémarrage s’effectue par vagues. Chaque vague a une portée différente (régions) :

- Une vague commence par une notification aux clients. Par défaut, la notification est envoyée aux propriétaire et copropriétaires de l’abonnement. Vous pouvez ajouter des destinataires et des options de messagerie (par exemple, e-mails, SMS et Webhooks) aux notifications à l’aide des [alertes de journal d’activité](../azure-monitor/platform/platform-logs-overview.md) Azure.  
- La notification vous propose une *fenêtre de libre-service*. Pendant cet intervalle, qui est généralement de 35 jours, vous pouvez rechercher quelles machines virtuelles sont incluses dans cette vague. Vous pouvez alors démarrer la maintenance de manière proactive en fonction de vos besoins de planification.
- Après la fenêtre de libre-service, une *fenêtre de maintenance planifiée* apparaît. Au cours de cette fenêtre, Azure planifie et applique la maintenance requise à votre machine virtuelle. 

L’objectif de ces deux fenêtres est de vous donner suffisamment de temps pour commencer la maintenance et redémarrer votre machine virtuelle tout en sachant à quel moment Azure démarrera automatiquement la maintenance.

Vous pouvez utiliser le portail Azure, PowerShell, les API REST et l’interface de ligne de commande Azure pour demander les fenêtres de maintenance pour vos machines virtuelles de groupes de machines virtuelles identiques et démarrer la maintenance en libre-service.

## <a name="should-you-start-maintenance-during-the-self-service-window"></a>Devez-vous démarrer la maintenance pendant la fenêtre de libre-service ?  

Les instructions suivantes vous aideront à déterminer si vous devez démarrer vous-même la maintenance au moment qui vous convient le mieux.

> [!NOTE] 
> La maintenance de libre-service n’est peut-être pas disponible pour toutes vos machines virtuelles. Pour déterminer si le redéploiement proactif est disponible pour votre machine virtuelle, recherchez l’option **Démarrer maintenant** dans l’état de maintenance. La maintenance en libre-service n’est pas disponible pour Azure Cloud Services (rôle de travail/web) et Azure Service Fabric.


La maintenance en libre-service n’est pas recommandée pour les déploiements qui utilisent *les groupes à haute disponibilité*. Les groupes à haute disponibilité sont des configurations hautement disponibles dans lesquelles un seul domaine de mise à jour est affecté à la fois. Pour les groupes à haute disponibilité :

- Laissez Azure déclencher la maintenance. Pour une maintenance nécessitant un redémarrage, les opérations sont effectuées domaine de mise à jour par domaine de mise à jour. Les domaines de mise à jour ne reçoivent pas nécessairement la maintenance de manière séquentielle. Un délai de 30 minutes s’écoule entre chaque domaine de mise à jour.
- Si une perte temporaire d’une partie de votre capacité (1/nombre de domaines de mise à jour) pose problème, elle peut facilement être compensée par l’allocation d’instances additionnelles au cours de la période de maintenance.
- Dans le cas d’une maintenance qui ne nécessite pas un redémarrage, les mises à jour sont appliquées au niveau du domaine d’erreur. 
    
**Ne pas** utiliser la maintenance de libre-service dans les scénarios suivants : 

- Si vous arrêtez vos machines virtuelles fréquemment, soit manuellement, à l’aide de DevTest Labs, en utilisant l’arrêt automatique ou en suivant un planning. Cela peut rétablir l’état de maintenance et donc entraîner un temps d’arrêt supplémentaire.
- Sur les machines virtuelles à durée de vie limitée dont vous savez qu’elles seront supprimées avant la fin de la vague de maintenance. 
- Pour les charges de travail avec un état volumineux stockées dans le disque local (éphémère) qui doivent être maintenues après la mise à jour. 
- Si vous redimensionnez souvent votre machine virtuelle. Cela peut rétablir l’état de maintenance. 
- Si vous avez adopté des événements planifiés qui permettent un basculement proactif ou l’arrêt approprié de votre charge de travail 15 minutes avant le début de l’arrêt de la maintenance.

**Utilisez** la maintenance en libre-service si vous envisagez d’exécuter votre machine virtuelle sans interruption pendant la phase de maintenance planifiée et qu’aucune des contre-indications mentionnées ci-dessus ne s’applique. 

Il est conseillé d’utiliser la maintenance en libre-service dans les cas suivants :

- Vous devez communiquer une fenêtre de maintenance exacte à votre administration ou à votre client. 
- Vous devez réaliser la maintenance à une date spécifique. 
- Vous devez contrôler la séquence de maintenance, par exemple dans le cas d’une application multiniveaux, pour garantir la récupération sans échec.
- Vous avez besoin de plus de 30 minutes de temps de récupération de machine virtuelle entre deux domaines de mise à jour. Pour contrôler le délai entre les domaines de mise à jour, vous devez déclencher la maintenance sur vos machines virtuelles, un domaine de mise à jour à la fois.

 
## <a name="view-virtual-machine-scale-sets-that-are-affected-by-maintenance-in-the-portal"></a>Afficher les groupes de machines virtuelles identiques affectés par la maintenance dans le portail

Lorsqu’une vague d’opérations de maintenance est planifiée, vous pouvez afficher la liste des groupes de machines virtuelles identiques affectés par la maintenance à venir à l’aide du portail Azure. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, sélectionnez **Tous les services**, puis **Groupes de machines virtuelles identiques**.
3. Sous **Groupes de machines virtuelles identiques**, sélectionnez **Modifier les colonnes** pour ouvrir la liste des colonnes disponibles.
4. Dans la section **Colonnes disponibles**, sélectionnez l’élément **Maintenance en libre-service**, puis déplacez-le dans la liste **Colonnes sélectionnées**. Sélectionnez **Appliquer**.  

    Pour trouver plus facilement l’élément **Maintenance en libre-service**, vous pouvez modifier l’option de liste déroulante de la section **Colonnes disponibles** de **Toutes** à **Propriétés**.

La colonne **Maintenance en libre-service** s’affiche désormais dans la liste des groupes de machines virtuelles identiques. Chaque groupe de machines virtuelles identiques peut avoir une des valeurs suivantes pour la colonne de maintenance en libre-service :

| Valeur | Description |
|-------|-------------|
| Oui | Au moins une machine virtuelle dans votre groupe de machines virtuelles identiques est dans une fenêtre de libre-service. Vous pouvez démarrer la maintenance à tout moment pendant cette fenêtre de libre-service. | 
| Non | Il n’y a aucune machine virtuelle définie dans une fenêtre de libre-service dans le groupe de machines virtuelles identiques affecté. | 
| - | Vos groupes de machines virtuelles identiques ne font pas partie d’un cycle de maintenance planifiée.| 

## <a name="notification-and-alerts-in-the-portal"></a>Notification et alertes dans le portail

Azure communique une planification de maintenance planifiée en envoyant un e-mail au propriétaire et au groupe de copropriétaires de l’abonnement. Vous pouvez ajouter des destinataires et des chaînes à cette communication en créant des alertes de journal d’activité. Pour en savoir plus, consultez [Surveiller l’activité d’abonnement avec le journal d’activité Azure](../azure-monitor/platform/platform-logs-overview.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu de gauche, sélectionnez **Surveiller**. 
3. Dans le volet **Surveiller - Alertes (classique)** , sélectionnez **+ Ajouter une alerte de journal d’activité**.
4. Sur la page **Ajouter une alerte de journal d’activité**, sélectionnez ou saisissez les informations requises. Sous **Critères**, renseignez les valeurs suivantes :
   - **Catégorie d’événement** : sélectionnez **Service Health**.
   - **Services** : sélectionnez **Groupes de machines virtuelles identiques et Machines virtuelles**.
   - **Type** : sélectionnez **Maintenance planifiée**. 
    
Pour découvrir plus en détail comment configurer des alertes de journal d’activité, consultez [Créer des alertes de journal d’activité](../azure-monitor/platform/activity-log-alerts.md).
    
    
## <a name="start-maintenance-on-your-virtual-machine-scale-set-from-the-portal"></a>Démarrer la maintenance sur votre groupe de machines virtuelles identiques à partir du portail

La présentation des groupes de machines virtuelles identiques contient davantage d’informations relatives à la maintenance. Si au moins une machine virtuelle dans le groupe de machines virtuelles identiques est incluse dans le cycle de maintenance planifiée, un nouveau ruban de notification sera ajouté au sommet de la page. Sélectionnez le ruban de notification pour accéder à la page **Maintenance**. 

Sur la page **Maintenance**, vous pouvez voir quelle instance de machine virtuelle est affectée par la maintenance planifiée. Pour démarrer la maintenance, cochez la case correspondant à la machine virtuelle affectée. Puis, sélectionnez **Démarrer la maintenance**.

Lorsque vous démarrez la maintenance, les machines virtuelles affectées dans votre groupe de machines virtuelles identiques sont soumises à une maintenance et temporairement indisponibles. Si vous avez raté la fenêtre de libre-service, vous pourrez toujours afficher la fenêtre quand votre groupe de machines virtuelles identiques sera soumis à une maintenance par Azure.
 
## <a name="check-maintenance-status-by-using-powershell"></a>Vérifier l’état de maintenance à l’aide de PowerShell

Vous pouvez utiliser Azure PowerShell pour voir quand les machines virtuelles dans vos groupes de machines virtuelles identiques sont planifiées pour la maintenance. Les informations de maintenance planifiée sont disponibles via la cmdlet [Get-AzVmss](/powershell/module/az.compute/get-azvmss) lorsque vous utilisez le paramètre `-InstanceView`.
 
Les informations de maintenance sont retournées uniquement en cas de maintenance planifiée. Si aucune maintenance planifiée n’affecte l’instance de machine virtuelle, la cmdlet ne retourne pas d’informations de maintenance. 

```powershell
Get-AzVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -InstanceView
```

Les propriétés suivantes sont retournées sous **MaintenanceRedeployStatus** : 

| Valeur | Description   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indique si vous pouvez démarrer la maintenance sur la machine virtuelle à l’instant T. |
| PreMaintenanceWindowStartTime         | Début de la fenêtre de maintenance en libre-service au cours de laquelle vous pouvez démarrer la maintenance sur votre machine virtuelle. |
| PreMaintenanceWindowEndTime           | Fin de la fenêtre de maintenance en libre-service au cours de laquelle vous pouvez démarrer la maintenance sur votre machine virtuelle. |
| MaintenanceWindowStartTime            | Début de la maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle. |
| MaintenanceWindowEndTime              | Fin de la fenêtre de maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle. |
| LastOperationResultCode               | Résultat de la dernière tentative de lancement de la maintenance sur la machine virtuelle. |



### <a name="start-maintenance-on-your-vm-instance-by-using-powershell"></a>Démarrer la maintenance sur votre instance de machine virtuelle à l’aide de PowerShell

Vous pouvez démarrer la maintenance sur une machine virtuelle si **IsCustomerInitiatedMaintenanceAllowed** est défini sur **true**. Utilisez la cmdlet [Set-AzVmss](/powershell/module/az.compute/set-azvmss) avec le paramètre `-PerformMaintenance`.

```powershell
Set-AzVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -PerformMaintenance 
```

## <a name="check-maintenance-status-by-using-the-cli"></a>Vérifier l’état de maintenance à l’aide d’interface de ligne de commande

Vous pouvez afficher les informations de maintenance planifiée à l’aide de la commande [az vmss list-instances](/cli/azure/vmss?view=azure-cli-latest#az-vmss-list-instances).
 
Les informations de maintenance sont retournées uniquement en cas de maintenance planifiée. Si aucune maintenance planifiée n’affecte l’instance de machine virtuelle, la commande ne retourne pas d’informations de maintenance. 

```azurecli
az vmss list-instances -g rgName -n vmssName --expand instanceView
```

Les propriétés suivantes sont retournées sous **MaintenanceRedeployStatus** pour chaque instance de machine virtuelle : 

| Valeur | Description   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indique si vous pouvez démarrer la maintenance sur la machine virtuelle à l’instant T. |
| PreMaintenanceWindowStartTime         | Début de la fenêtre de maintenance en libre-service au cours de laquelle vous pouvez démarrer la maintenance sur votre machine virtuelle. |
| PreMaintenanceWindowEndTime           | Fin de la fenêtre de maintenance en libre-service au cours de laquelle vous pouvez démarrer la maintenance sur votre machine virtuelle. |
| MaintenanceWindowStartTime            | Début de la maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle. |
| MaintenanceWindowEndTime              | Fin de la fenêtre de maintenance planifiée pendant laquelle Azure lance la maintenance sur votre machine virtuelle. |
| LastOperationResultCode               | Résultat de la dernière tentative de lancement de la maintenance sur la machine virtuelle. |


### <a name="start-maintenance-on-your-vm-instance-by-using-the-cli"></a>Démarrer la maintenance sur votre instance de machine virtuelle à l’aide de l’interface de ligne de commande

L’appel suivant lance la maintenance sur une instance de machine virtuelle si `IsCustomerInitiatedMaintenanceAllowed` est défini sur **true** :

```azurecli
az vmss perform-maintenance -g rgName -n vmssName --instance-ids id
```

## <a name="faq"></a>Questions fréquentes (FAQ)

**Q : Pourquoi devez-vous redémarrer mes machines virtuelles maintenant ?**

**R :** Bien que la plupart des mises à jour et des mises à niveau de la plateforme Azure n’affectent pas la disponibilité des machines virtuelles, nous sommes contraints, dans certains cas, de redémarrer les machines virtuelles hébergées dans Azure. Nous avons accumulé plusieurs modifications qui nous obligent à redémarrer nos serveurs, ce qui entraînera le redémarrage des machines virtuelles.

**Q : Si je suis vos recommandations concernant la haute disponibilité en utilisant un groupe à haute disponibilité, mon environnement est-il sécurisé ?**

**R :** Les machines virtuelles déployées dans un groupe à haute disponibilité ou dans des groupes de machines virtuelles identiques utilisent des domaines de mise à jour. Lors des opérations de maintenance, Azure respecte la contrainte de domaine de mise à jour et ne redémarre pas les machines virtuelles à partir d’un domaine de mise à jour distinct (dans le même groupe à haute disponibilité). Azure attend également au moins 30 minutes avant de passer au groupe de machines virtuelles suivant. 

Pour plus d’informations sur la haute disponibilité, consultez [Régions et disponibilité des machines virtuelles dans Azure](../virtual-machines/availability.md).

**Q : Comment puis-je être averti d’une maintenance planifiée ?**

**R :** Une vague d’opérations de maintenance planifiée commence par une planification sur une ou plusieurs régions Azure. Peu après, une notification par e-mail est envoyée aux administrateurs, aux coadministrateurs, aux propriétaires et aux contributeurs d’abonnement (un e-mail par abonnement). Il est possible de configurer des canaux et des destinataires supplémentaires pour cette notification à l’aide de la fonctionnalité Alertes de journal d’activité. Dans le cas où vous déployez une machine virtuelle dans une région où la planification de la maintenance est déjà effectuée, vous ne recevez pas la notification. Vous devrez dans ce cas consulter l’état de maintenance de la machine virtuelle.

**Q : Je ne vois aucune indication de maintenance planifiée dans le portail, dans PowerShell ou dans l’interface CLI. Est-ce normal ?**

**R :** Les informations relatives à la maintenance planifiée ne sont disponibles pendant une vague d’opérations de maintenance planifiée que pour les machines virtuelles concernées par la maintenance planifiée. Si aucune information ne s’affiche, il est possible que la vague de maintenance soit déjà terminée (ou non démarrée), ou que votre machine virtuelle soit déjà hébergée sur un serveur mis à jour.

**Q : Est-il possible de savoir exactement quand ma machine virtuelle sera affectée ?**

**R :** Lors de la planification, nous définissons une fenêtre de temps de plusieurs jours. Le séquencement exact des serveurs (et des machines virtuelles) au sein de cette fenêtre est inconnu. Si vous souhaitez connaître la date exacte à laquelle vos machines virtuelles seront mises à jour, vous pouvez utiliser les [événements planifiés](../virtual-machines/windows/scheduled-events.md). Avec les événements planifiés, vous pouvez envoyer des requêtes à partir de la machine virtuelle et recevoir une notification 15 minutes avant un redémarrage de la machine virtuelle.

**Q : Combien de temps vous faudra-t-il pour redémarrer ma machine virtuelle ?**

**R :**  En fonction de la taille de votre machine virtuelle, le redémarrage peut prendre plusieurs minutes pendant la fenêtre de maintenance en libre-service. Tout redémarrage lancé par Azure pendant la fenêtre de maintenance planifiée dure habituellement 25 minutes environ. Si vous utilisez des services cloud (rôle de travail/web), des groupes de machines virtuelles identiques ou des groupes à haute disponibilité, vous disposez de 30 minutes entre chaque groupe de machines virtuelles (domaine de mise à jour) pendant la fenêtre de maintenance planifiée. 

**Q : Je ne vois aucune information de maintenance sur mes machines virtuelles. Quelle est la cause du problème ?**

**R :** Plusieurs raisons peuvent expliquer pourquoi vous ne voyez aucune information de maintenance sur vos machines virtuelles :
   - Vous utilisez un abonnement marqué comme *interne à Microsoft*.
   - Vos machines virtuelles ne sont pas planifiées pour la maintenance. Il est possible que la vague d’opérations de maintenance soit terminée, annulée ou modifiée, de sorte que celle-ci n’impacte plus vos machines virtuelles.
   - La colonne **Maintenance** n’a pas été ajoutée à l’affichage de liste de votre machine virtuelle. Bien que nous ayons ajouté cette colonne à la vue par défaut, si vous avez configuré votre vue pour afficher des colonnes non définies par défaut, vous devez ajouter manuellement la colonne **Maintenance** à la vue liste de vos machines virtuelles.

**Q : La maintenance de ma machine virtuelle est planifiée pour la deuxième fois. Pourquoi ?**

**R :** Dans plusieurs cas, votre machine virtuelle apparaît comme planifiée pour la maintenance alors que vous avez déjà effectué votre maintenance-redéploiement :
   - Nous avons annulé la vague d’opérations de maintenance et l’avons redémarrée avec une charge utile différente. Nous avons peut-être détecté une charge utile ayant généré une erreur et voulons simplement déployer une charge utile supplémentaire.
   - Votre machine virtuelle a été *réparée par service* sur un autre nœud en raison d’une panne matérielle.
   - Vous avez choisi d’arrêter (de libérer) et de redémarrer la machine virtuelle.
   - Vous avez activé **Arrêt automatique** pour la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment vous inscrire aux événements de maintenance à partir de la machine virtuelle à l’aide des [événements planifiés](../virtual-machines/windows/scheduled-events.md).
