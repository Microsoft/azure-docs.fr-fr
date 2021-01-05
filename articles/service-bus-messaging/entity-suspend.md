---
title: Azure Service Bus – Interruption d’entités de messagerie
description: Cet article explique comment suspendre temporairement et réactiver des entités de message Azure Service Bus (files d’attente, rubriques et abonnements).
ms.topic: article
ms.date: 09/29/2020
ms.openlocfilehash: ea1acab3d0a86b0064f8b3eef7bfd1496bd17041
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94543049"
---
# <a name="suspend-and-reactivate-messaging-entities-disable"></a>Interrompre et réactiver des entités de messagerie (désactiver)

Les files d’attente, rubriques et abonnements peuvent être temporairement interrompus. L’interruption place l’entité dans un état désactivé dans lequel tous les messages sont conservés dans le stockage. Toutefois, aucun message ne peut être supprimé ou ajouté, et les opérations de protocole respectif produisent des erreurs.

Vous pouvez suspendre une entité pour des raisons urgentes liées à l’administration. Par exemple, un récepteur défectueux prend des messages dans la file d’attente, échoue à les traiter, et les termine néanmoins incorrectement et les supprime. Dans ce cas, vous pouvez désactiver la réception pour la file d’attente jusqu’à la correction et le déploiement du code. 

Aussi bien l’utilisateur que le système peuvent effectuer une interruption ou une réactivation. Le système met en suspens des entités seulement pour des raisons graves liées à l’administration, comme quand la limite de dépense de l’abonnement est atteinte. Les entités désactivées par le système ne peuvent pas être réactivées par l’utilisateur, mais sont restaurées une fois la cause de l’interruption traitée.

## <a name="queue-status"></a>État de la file d’attente 
Les états qui peuvent être définis pour une **file d’attente** sont :

-   **Actif** : la file d’attente est active. Vous pouvez envoyer des messages à la file d’attente et recevoir des messages de celle-ci. 
-   **Disabled** : la file d’attente est suspendue. Cela revient à définir **SendDisabled** et **ReceiveDisabled**. 
-   **SendDisabled** : Vous ne pouvez pas envoyer de messages à la file d’attente, mais vous pouvez en recevoir de celle-ci. Vous recevez une exception si vous essayez d’envoyer des messages à la file d’attente. 
-   **ReceiveDisabled** : Vous pouvez envoyer des messages à la file d’attente, mais vous ne pouvez pas en recevoir de celle-ci. Vous recevez une exception si vous essayez de recevoir des messages de la file d’attente.


### <a name="change-the-queue-status-in-the-azure-portal"></a>Modifiez l’état de la file d’attente dans le portail Azure : 

1. Dans le portail Azure, accédez à votre espace de noms Service Bus. 
1. Sélectionnez la file d’attente dont vous souhaitez modifier l’état. Les files d’attente sont affichées au milieu du volet inférieur. 
1. Sur la page **File d’attente Service Bus**, examinez l’état actuel de la file d’attente sous forme de lien hypertexte. Si l’option **Vue d’ensemble** n’est pas sélectionnée dans le menu de gauche, sélectionnez-la pour afficher l’état de la file d’attente. Sélectionnez l’état actuel de la file d’attente pour le modifier. 

    :::image type="content" source="./media/entity-suspend/select-state.png" alt-text="Sélectionner l’état de la file d’attente":::
4. Sélectionnez le nouvel état de la file d’attente, puis **OK**. 

    :::image type="content" source="./media/entity-suspend/entity-state-change.png" alt-text="Définir l’état de la file d’attente":::
    
Vous pouvez également désactiver les opérations d’envoi et de réception en utilisant les API [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) de Service Bus dans le SDK .NET, ou en utilisant un modèle Azure Resource Manager via Azure CLI ou Azure PowerShell.

### <a name="change-the-queue-status-using-azure-powershell"></a>Modifier l’état de la file d’attente à l’aide d’Azure PowerShell
La commande PowerShell pour désactiver une file d’attente est présentée dans l’exemple suivant. La commande de réactivation est équivalente et définit le paramètre `Status` sur **Active**.

```powershell
$q = Get-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue

$q.Status = "Disabled"

Set-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue -QueueObj $q
```

## <a name="topic-status"></a>État de la rubrique
Vous pouvez changer l’état de la rubrique dans le portail Azure. Sélectionnez l’état actuel de la rubrique pour voir la page suivante, qui vous permet de changer l’état. 

:::image type="content" source="./media/entity-suspend/topic-state-change.png" alt-text="Modifier l’état de la rubrique":::

Les états pouvant être définis pour une **rubrique** sont :
- **Actif** : La rubrique est active. Vous pouvez envoyer des messages à la rubrique. 
- **Disabled** : La rubrique est interrompue. Vous ne pouvez pas envoyer de messages à la rubrique. 
- **SendDisabled** : Même effet que **Désactivé**. Vous ne pouvez pas envoyer de messages à la rubrique. Vous recevez une exception si vous essayez d’envoyer des messages à la rubrique. 

## <a name="subscription-status"></a>État de l’abonnement
Vous pouvez changer l’état de l’abonnement dans le portail Azure. Sélectionnez l’état actuel de l’abonnement pour voir la page suivante, qui vous permet de changer l’état. 

:::image type="content" source="./media/entity-suspend/subscription-state-change.png" alt-text="Modifier l’état de l’abonnement":::

Les états pouvant être définis pour un **abonnement** sont :
- **Actif** : L’abonnement est actif. Vous pouvez recevoir des messages de l’abonnement.
- **Disabled** : L’abonnement est suspendu. Vous ne pouvez pas recevoir de messages de l’abonnement. 
- **ReceiveDisabled** : Même effet que **Désactivé**. Vous ne pouvez pas recevoir de messages de l’abonnement. Vous recevez une exception si vous essayez de recevoir des messages de l’abonnement.

| État de la rubrique | État de l’abonnement | Comportement | 
| ------------ | ------------------- | -------- | 
| Actif | Actif | Vous pouvez envoyer des messages à la rubrique et recevoir des messages de l’abonnement. | 
| Actif | Désactivé ou Réception désactivée | Vous pouvez envoyer des messages à la rubrique, mais vous ne pouvez pas recevoir de messages de l’abonnement. | 
| Désactivé ou Envoi désactivé | Actif | Vous ne pouvez pas envoyer de messages à la rubrique, mais vous pouvez recevoir des messages qui sont déjà dans l’abonnement. | 
| Désactivé ou Envoi désactivé | Désactivé ou Réception désactivée | Vous ne pouvez pas envoyer de messages à la rubrique et vous ne pouvez pas en recevoir de l’abonnement. | 

## <a name="other-statuses"></a>Autres états
L’énumération [EntityStatus](/dotnet/api/microsoft.servicebus.messaging.entitystatus) définit également un ensemble d’état de transition qui peuvent uniquement être configurés par le système. 


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/entity-suspend/entity-state-change.png

