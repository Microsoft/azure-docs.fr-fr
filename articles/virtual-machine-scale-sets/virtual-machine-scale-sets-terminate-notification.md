---
title: Notification d’arrêt des instances de groupe de machines virtuelles identiques Azure
description: Découvrez comment activer la notification d’arrêt pour les instances de groupe de machines virtuelles identiques Azure.
author: avirishuv
ms.author: avverma
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 02/26/2020
ms.reviewer: jushiman
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: c4d6de1b3406e6d82bdac5ff9b5c72a2286da988
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747748"
---
# <a name="terminate-notification-for-azure-virtual-machine-scale-set-instances"></a>Notification d’arrêt des instances de groupe de machines virtuelles identiques Azure
Les instances de groupe identique peuvent choisir de recevoir des notifications d’arrêt de l’instance et de configurer un délai d’attente prédéfini pour l’opération d’arrêt. La notification d’arrêt est envoyée par le biais d’Azure Metadata Service – [Scheduled Events](../virtual-machines/windows/scheduled-events.md), qui fournit des notifications et un report des opérations ayant un impact, telles que le redémarrage et le redéploiement. La solution ajoute un autre événement (Terminate) à la liste de Scheduled Events, et le délai associé à l’événement d’arrêt dépend de la limite de délai spécifiée par les utilisateurs dans leurs configurations de modèle de groupe identique.

Une fois inscrites pour la fonctionnalité, les instances de groupe identique n’ont pas besoin d’attendre l’expiration du délai spécifié pour être supprimées. Après la réception d’une notification d’arrêt, l’instance peut choisir d’être supprimée à tout moment avant l’expiration du délai d’arrêt.

## <a name="enable-terminate-notifications"></a>Activer les notifications d’arrêt
Il existe plusieurs façons d’activer les notifications d’arrêt sur vos instances de groupe identique, comme indiqué dans les exemples ci-dessous.

### <a name="azure-portal"></a>Portail Azure

Les étapes suivantes permettent de mettre fin à une notification lors de la création d’un groupe identique. 

1. Accédez à **Groupe de machines virtuelles identiques** .
1. Sélectionnez **+ Ajouter** pour créer un groupe identique.
1. Accédez à l’onglet **Gestion** . 
1. Localisez la section **Arrêt d’instance** .
1. Pour **Notification d’arrêt d’instance** , sélectionnez **Activée** .
1. Pour **Délai d’arrêt (minutes)** , définissez le délai d’expiration par défaut souhaité.
1. Lorsque vous avez fini de créer le groupe identique, sélectionnez le bouton **Vérifier + créer** . 

> [!NOTE]
> Vous ne pouvez pas définir de notifications d’arrêt sur des groupes identiques existants dans le portail Azure

### <a name="rest-api"></a>API REST

L’exemple suivant active la notification d’arrêt sur le modèle de groupe identique.

```
PUT on `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachineScaleSets/{vmScaleSetName}?api-version=2019-03-01`
```

```json
{
  "properties": {
    "virtualMachineProfile": {
            "scheduledEventsProfile": {
                "terminateNotificationProfile": {
                    "notBeforeTimeout":"PT5M",
                    "enable":true
                }
            }
        }
    }        
}

```

Le bloc ci-dessus spécifie un délai d’attente de cinq minutes (comme indiqué par *PT5M* ) pour toute opération d’arrêt sur toutes les instances de votre groupe identique. Le champ *notBeforeTimeout* peut prendre n’importe quelle valeur comprise entre 5 et 15 minutes au format ISO 8601. Vous pouvez changer le délai d’attente par défaut pour l’opération d’arrêt en modifiant la propriété *notBeforeTimeout* sous *terminateNotificationProfile* décrite ci-dessus.

Après avoir activé *scheduledEventsProfile* sur le modèle de groupe identique et défini *notBeforeTimeout* , mettez à jour chaque instance vers le [modèle le plus récent](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) afin de refléter les modifications.

> [!NOTE]
>Les notifications d’arrêt sur les instances de groupe identique peuvent uniquement être activées avec l’API version 2019-03-01 et ultérieures.

### <a name="azure-powershell"></a>Azure PowerShell
Quand vous créez un groupe identique, vous pouvez activer les notifications d’arrêt sur celui-ci à l’aide de la cmdlet [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).

Cet exemple de script présente la création d’un groupe identique et des ressources associées à l’aide du fichier config : [Créer un groupe de machines virtuelles identiques complet](./scripts/powershell-sample-create-complete-scale-set.md). Vous pouvez fournir une notification d’arrêt de configuration en ajoutant les paramètres *TerminateScheduledEvents* et *TerminateScheduledEventNotBeforeTimeoutInMinutes* à l’objet configuration pour la création du groupe identique. L’exemple suivant active la fonctionnalité avec un délai d’expiration de 10 minutes.

```azurepowershell-interactive
New-AzVmssConfig `
  -Location "VMSSLocation" `
  -SkuCapacity 2 `
  -SkuName "Standard_DS2" `
  -UpgradePolicyMode "Automatic" `
  -TerminateScheduledEvents $true `
  -TerminateScheduledEventNotBeforeTimeoutInMinutes 10
```

Utilisez l’applet de commande [Update-AzVmss](/powershell/module/az.compute/update-azvmss) pour activer les notifications d’arrêt sur un groupe identique existant.

```azurepowershell-interactive
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -TerminateScheduledEvents $true `
  -TerminateScheduledEventNotBeforeTimeoutInMinutes 15
```
L’exemple ci-dessus active les notifications d’arrêt sur un groupe identique existant et définit un délai d’attente de 15 minutes pour l’événement d’arrêt.

Après avoir activé les événements planifiés sur le modèle de groupe identique et défini le délai d’attente, mettez à jour chaque instance vers le [modèle le plus récent](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) afin de refléter les modifications.

### <a name="azure-cli-20"></a>Azure CLI 2.0

L’exemple suivant permet d’activer une notification d’arrêt lors de la création d’un groupe identique.

```azurecli-interactive
az group create --name <myResourceGroup> --location <VMSSLocation>
az vmss create \
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --image UbuntuLTS \
  --admin-username <azureuser> \
  --generate-ssh-keys \
  --terminate-notification-time 10
```

L’exemple ci-dessus crée un groupe de ressources, puis un groupe identique avec des notifications d’arrêt activées pour un délai d’expiration par défaut de 10 minutes.

L’exemple suivant permet d’activer la notification d’arrêt dans un groupe identique existant.

```azurecli-interactive
az vmss update \  
  --resource-group <myResourceGroup> \
  --name <myVMScaleSet> \
  --enable-terminate-notification true \
  --terminate-notification-time 10
```

## <a name="get-terminate-notifications"></a>Recevoir des notifications d’arrêt

Les notifications d’arrêt sont remises par le biais de [Scheduled Events](../virtual-machines/windows/scheduled-events.md), qui fait partie d’Azure Metadata Service. Le service de métadonnées Azure expose des informations sur les machines virtuelles en cours d’exécution en utilisant un point de terminaison REST accessible depuis la machine virtuelle. Les informations sont disponibles par le biais d’une adresse IP non routable, de sorte qu’elles ne soient pas exposées en dehors de la machine virtuelle.

Le service Scheduled Events est activé pour votre groupe identique la première fois que vous faites une requête d’événements. Attendez-vous à ce que la réponse à votre première demande ait un retard pouvant atteindre deux minutes. Interrogez régulièrement le point de terminaison pour détecter les événements de maintenance à venir et l’état des activités de maintenance en cours.

Le service Scheduled Events est désactivé pour votre groupe identique si les instances du groupe identique n’effectuent pas de requête pendant 24 heures.

### <a name="endpoint-discovery"></a>Découverte de point de terminaison
Pour les machines virtuelles compatibles avec le réseau virtuel, Metadata Service est disponible à partir d’une adresse IP non routable statique, 169.254.169.254.

Le point de terminaison complet de la dernière version des événements planifiés est :
> 'http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01 '

### <a name="query-response"></a>Réponse de la requête
Une réponse contient un tableau d’événements planifiés. Un tableau vide signifie qu’il n’y a actuellement aucun événement planifié.

S’il existe des événements planifiés, la réponse contient un tableau d’événements. Pour un événement « Terminate », la réponse se présente comme suit :
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Terminate",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled",
            "NotBefore": {timeInUTC},
        }
    ]
}
```
*DocumentIncarnation* est un ETag qui permet d’inspecter facilement la charge utile d’événement pour savoir si elle a changé depuis la dernière requête.

Pour plus d’informations sur chacun des champs ci-dessus, consultez la documentation de Scheduled Events pour [Windows](../virtual-machines/windows/scheduled-events.md#event-properties) et [Linux](../virtual-machines/linux/scheduled-events.md#event-properties).

### <a name="respond-to-events"></a>Répondre aux événements
Une fois que vous avez pris connaissance d’un événement à venir et effectué votre logique d’arrêt appropriée, vous pouvez approuver l’événement en suspens en effectuant un appel POST au service de métadonnées avec l’EventId. L’appel POST indique à Azure qu’il peut poursuivre la suppression de la machine virtuelle.

Voici le code JSON attendu dans le corps de la requête POST. La requête doit contenir une liste de StartRequests. Chaque StartRequest contient l’EventId de l’événement que vous souhaitez accélérer :
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

Vérifiez que chaque machine virtuelle du groupe identique n’approuve que l’EventID correspondant à cette machine virtuelle. Une machine virtuelle peut obtenir son propre nom de machine virtuelle [par le biais de métadonnées d’instance](virtual-machine-scale-sets-instance-ids.md#instance-metadata-vm-name). Ce nom prend la forme « {nom_groupe_identique}_{ID_instance} » et est affiché dans la section « Resources » de la réponse de la requête décrite ci-dessus.

Vous pouvez également vous reporter aux exemples de scripts pour l’interrogation et la réponse aux événements à l’aide de [Python](../virtual-machines/linux/scheduled-events.md#python-sample).

## <a name="tips-and-best-practices"></a>Conseils et meilleures pratiques
-   Notifications d’arrêt uniquement sur les opérations de suppression : toutes les opérations de suppression (suppression manuelle ou scale-in lancé par la mise à l’échelle automatique) génèrent des événements Terminate si *scheduledEventsProfile* est activé sur votre groupe identique. Les autres opérations telles que le redémarrage, la réinitialisation, le redéploiement et l’arrêt/désallocation ne génèrent pas d’événements Terminate. Les notifications d’arrêt ne peuvent pas être activées pour les machines virtuelles de faible priorité.
-   Aucune attente obligatoire pour l’expiration : vous pouvez démarrer l’opération d’arrêt à tout moment après la réception de l’événement et avant l’expiration du délai *NotBefore* .
-   Suppression obligatoire une fois le délai d’attente atteint : il n’existe aucune possibilité d’étendre la valeur du délai d’expiration après la génération d’un événement. Une fois le délai d’attente atteint, l’événement Terminate en attente est traité et la machine virtuelle est supprimée.
-   Valeur de délai d’attente modifiable : vous pouvez modifier la valeur du délai d’attente à tout moment avant la suppression d’une instance, en modifiant la propriété *notBeforeTimeout* sur le modèle de groupe identique et en mettant à jour les instances de machine virtuelle vers le dernier modèle.
-   Approbation de toutes les suppressions en attente : s’il existe sur VM_1 une suppression en attente qui n’est pas approuvée, et que vous avez approuvé un autre événement Terminate sur VM_2, VM_2 n’est pas supprimé tant que l’événement Terminate pour VM_1 n’a pas été approuvé ou que son délai d’attente ne s’est pas écoulé. Une fois que vous avez approuvé l’événement Terminate pour VM_1, VM_1 et VM_2 sont supprimées.
-   Approbation de toutes les suppressions simultanées : si l’on poursuit avec l’exemple ci-dessus, si VM_1 et VM_2 ont le même délai *NotBefore* , les deux événements Terminate doivent être approuvés, sinon aucune des machines virtuelles n’est supprimée avant l’expiration du délai d’attente.

## <a name="troubleshoot"></a>Dépanner
### <a name="failure-to-enable-scheduledeventsprofile"></a>Impossible d’activer scheduledEventsProfile
Si vous obtenez une erreur « BadRequest » avec un message d’erreur indiquant que le membre « scheduledEventsProfile » est introuvable sur l’objet de type « VirtualMachineProfile », vérifiez la version de l’API utilisée pour les opérations du groupe identique. La version de l’API de calcul **2019-03-01** ou une version ultérieure est requise. 

### <a name="failure-to-get-terminate-events"></a>Impossible d’obtenir des événements Terminate
Si vous n’obtenez aucun événement **Terminate** par le biais de Scheduled Events, vérifiez la version de l’API utilisée pour obtenir les événements. L’API Metadata Service version  **2019-01-01** ou ultérieure est requise pour les événements Terminate.
>'http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01 '

### <a name="getting-terminate-event-with-incorrect-notbefore-time"></a>Obtention d’événement Terminate avec délai NotBefore incorrect  
Après avoir activé *scheduledEventsProfile* sur le modèle de groupe identique et défini *notBeforeTimeout* , mettez à jour chaque instance vers le [modèle le plus récent](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) afin de refléter les modifications.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [déployer votre application](virtual-machine-scale-sets-deploy-app.md) sur des groupes de machines virtuelles identiques.
