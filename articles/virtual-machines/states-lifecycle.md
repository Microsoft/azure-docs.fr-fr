---
title: Cycle de vie et états d’une machine virtuelle dans Azure
description: Vue d’ensemble du cycle de vie d’une machine virtuelle dans Azure, y compris des descriptions des différents états qu’une machine virtuelle peut avoir à n’importe quel moment.
services: virtual-machines
author: shandilvarun
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 08/09/2018
ms.author: vashan
ms.openlocfilehash: 8c9641e722cf3892450bbfbea54f169ac6dc764b
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963293"
---
# <a name="virtual-machines-lifecycle-and-states"></a>Cycle de vie et états des machines virtuelles

Les machines virtuelles Microsoft Azure passent par différents états qui peuvent être classés en états *d’approvisionnement* et *d’alimentation*. Cet article vise à décrire ces états et en particulier à mettre en exergue la date de facturation des clients pour l’utilisation des instances. 

## <a name="power-states"></a>États d’alimentation

L’état d’alimentation représente le dernier état connu de la machine virtuelle.

![Diagramme d’état de l’alimentation des machines virtuelles](./media/virtual-machines-common-states-lifecycle/vm-power-states.png)

<br>
Le tableau suivant contient une description de l’état de chaque instance et indique si elle est facturée pour utilisation ou non.

:::row:::
   :::column span="":::

   **State**
   
   :::column-end:::
   :::column span="":::

   **Description**

   :::column-end:::
   :::column span="":::

   **Facturation de l’utilisation de l’instance**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Démarrage en cours**

   :::column-end:::
   :::column span="":::

   La machine virtuelle démarre.

   ```json
   "statuses": [
    {
    "code": "PowerState/starting",
    "level": "Info",
    "displayStatus": "VM starting"
    }
   ]
   ```
   :::column-end:::
   :::column span="":::

   **Pas de facturation**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Exécution**

   :::column-end:::
   :::column span="":::

   État de fonctionnement normal d’une machine virtuelle

   ```json
   "statuses": [
    {
    "code": "PowerState/running",
    "level": "Info",
    "displayStatus": "VM running"
    }
  ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturation**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **En cours d’arrêt**

   :::column-end:::
   :::column span="":::

   Il s’agit d’un état transitoire. À l’issue de l’opération, l’état indiqué est **Arrêté**.

   ```json
   "statuses": [
    {
    "code": "PowerState/stopping",
    "level": "Info",
    "displayStatus": "VM stopping"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturation**

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Arrêté**

   :::column-end:::
   :::column span="":::

   La machine virtuelle a été arrêtée à partir du système d’exploitation invité ou à l’aide des API PowerOff.

   Le matériel est toujours alloué à la machine virtuelle et demeure sur l’hôte.

   ```json
   "statuses": [
    {
    "code": "PowerState/stopped",
    "level": "Info",
    "displayStatus": "VM stopped"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Facturation***

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Libération**

   :::column-end:::
   :::column span="":::

   État transitoire. À l’issue de l’opération, la machine virtuelle indique l’état **Désalloué**.

   ```json
   "statuses": [
    {
    "code": "PowerState/deallocating",
    "level": "Info",
    "displayStatus": "VM deallocating"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Pas de facturation***

   :::column-end:::
:::row-end:::

:::row:::
   :::column span="":::

   **Libéré**

   :::column-end:::
   :::column span="":::

   La machine virtuelle a été arrêtée et supprimée de l’hôte.

   ```json
   "statuses": [
    {
    "code": "PowerState/deallocated",
    "level": "Info",
    "displayStatus": "VM deallocated"
    }
   ]
  ```
   :::column-end:::
   :::column span="":::

   **Pas de facturation**

   :::column-end:::
:::row-end:::


&#42; Certaines ressources Azure, par exemple Azure Data Box Disk et Azure Networking, impliquent des frais. Les licences de logiciels sur l’instance n’engendrent aucun frais.

## <a name="provisioning-states"></a>États d’approvisionnement

Un état d’approvisionnement est l’état d’une opération de type plan de contrôle initiée par l’utilisateur sur la machine virtuelle. Ces états sont distincts de l’état d’alimentation d’une machine virtuelle.

- **Créer** : création d’une machine virtuelle.

- **Mettre à jour** : met à jour le modèle correspondant à une machine virtuelle existante. Certaines modifications apportées aux machines virtuelles et ne concernant pas un modèle, par exemple Démarrer/Redémarrer, relèvent également d’une mise à jour.

- **Supprimer** : suppression d’une machine virtuelle.

- **Désallouer** : lorsqu’une machine virtuelle est arrêtée, puis supprimée de l’hôte. Étant donné que la désallocation d’une machine virtuelle est considérée comme une mise à jour, elle affiche les états d’approvisionnement liés à la mise à jour.



Voici les états transitoires des opérations une fois que la plateforme a accepté une action initiée par l’utilisateur :

:::row:::
   :::column span="":::

   **State**
   
   :::column-end:::
   :::column span="2":::

   **Description**

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Creating**

   :::column-end:::
   :::column span="2":::

  ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating",
    "level": "Info",
    "displayStatus": "Creating"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Mise à jour**

   :::column-end:::
   :::column span="2":::

   ```json
   "statuses": [
    {
    "code": "ProvisioningState/updating",
    "level": "Info",
    "displayStatus": "Updating"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **Suppression en cours**

   :::column-end:::
   :::column span="2":::

   ```json
   "statuses": [
    {
    "code": "ProvisioningState/deleting",
    "level": "Info",
    "displayStatus": "Deleting"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **États d’approvisionnement d’un système d’exploitation**
   
   :::column-end:::
   :::column span="2":::

   **Description**

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::



   :::column-end:::
   :::column span="2":::

   Si une machine virtuelle est créée avec une image de système d’exploitation et pas avec une image spécialisée, les sous-états suivants peuvent être observés :

   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **OSProvisioningInprogress**

   :::column-end:::
   :::column span="2":::

   La machine virtuelle est en cours d’exécution et l’installation du système d’exploitation invité est en cours.
 
   ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating/OSProvisioningInprogress",
    "level": "Info",
    "displayStatus": "OS Provisioning In progress"
    }
   [
   ```
   :::column-end:::

:::row-end:::

:::row:::
   :::column span="":::

   **OSProvisioningComplete**

   :::column-end:::
   :::column span="2":::
   
   État à courte durée de vie. La machine virtuelle passe rapidement à l’état **Réussite**, sauf si des extensions doivent être installées. L’installation des extensions peut prendre du temps.
   
   ```json
   "statuses": [
    {
    "code": "ProvisioningState/creating/OSProvisioningComplete",
    "level": "Info",
    "displayStatus": "OS Provisioning Complete"
    }
   [
   ```
   
   **Remarque** : l’approvisionnement du système d’exploitation peut passer à l’état **Échec** en cas de défaillance du système d’exploitation ou si ce dernier n’est pas installé à temps. Les clients sont facturés pour la machine virtuelle déployée dans l’infrastructure.

   :::column-end:::

:::row-end:::

Une fois l’opération terminée, la machine virtuelle passe à l’un des états suivants :

- **Réussi** : les actions initiées par l’utilisateur se sont terminées.

    ```
  "statuses": [ 
  {
     "code": "ProvisioningState/succeeded",
     "level": "Info",
     "displayStatus": "Provisioning succeeded",
     "time": "time"
  }
  ]
    ```

 

- **Échec** : représente une opération en échec. Reportez-vous aux codes d’erreur pour obtenir plus d’informations ainsi que les solutions possibles.

    ```
  "statuses": [
    {
      "code": "ProvisioningState/failed/InternalOperationError",
      "level": "Error",
      "displayStatus": "Provisioning failed",
      "message": "Operation abandoned due to internal error. Please try again later.",
      "time": "time"
    }
    ]
    ```



## <a name="vm-instance-view"></a>Vue d’instance de machine virtuelle

L’API de vue d’instance fournit des informations sur l’état d’exécution d’une machine virtuelle. Pour plus d’informations, consultez la documentation relative à l’API [Virtual Machines - Instance View](/rest/api/compute/virtualmachines/instanceview) (Machines virtuelles : vue d’instance).

Azure Resource Explorer présente une interface utilisateur simple permettant d’afficher l’état d’exécution des machines virtuelles : [Resource Explorer](https://resources.azure.com/).

Les états d’approvisionnement sont visibles dans la vue d’instance et les propriétés des machines virtuelles. Les états d’alimentation sont disponibles dans la vue d’instance des machines virtuelles.

Pour récupérer l’état d’alimentation de toutes les machines virtuelles de votre abonnement, utilisez l’[API Machines virtuelles - Répertorier tout](/rest/api/compute/virtualmachines/listall) avec le paramètre **statusOnly** défini sur *true*.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la surveillance de votre machine virtuelle, consultez [Surveillance des machines virtuelles dans Azure](../azure-monitor/insights/monitor-vm-azure.md).