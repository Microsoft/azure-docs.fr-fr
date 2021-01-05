---
title: Recevoir des notifications concernant un coffre de clés et y répondre avec Azure Event Grid
description: Découvrez comment intégrer Key Vault avec Azure Event Grid.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/25/2019
ms.author: mbaldwin
ms.openlocfilehash: d7bb697879f40b45c886cd90bbb1e34906d35f66
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187372"
---
# <a name="receive-and-respond-to-key-vault-notifications-with-azure-event-grid"></a>Recevoir des notifications concernant un coffre de clés et y répondre avec Azure Event Grid

L’intégration d’Azure Key Vault à Azure Event Grid permet de notifier l’utilisateur en cas de changement de l’état d’un secret stocké dans un coffre de clés. Pour obtenir une vue d’ensemble de cette fonctionnalité, consultez [Supervision de Key Vault avec Event Grid](event-grid-overview.md).

Ce guide explique comment recevoir des notifications de Key Vault via Event Grid, et comment répondre aux changements d’état via Azure Automation.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Un coffre de clés dans votre abonnement Azure. Vous pouvez créer rapidement un coffre de clés en suivant les étapes décrites dans [Définir et récupérer un secret depuis Azure Key Vault à l’aide d’Azure CLI](../secrets/quick-create-cli.md).

## <a name="concepts"></a>Concepts

Event Grid est un service de gestion d’événements dans le cloud. En suivant les étapes de ce guide, vous allez vous abonner à des événements pour Key Vault et les router vers Automation. Lorsque l’un des secrets du coffre de clés est sur le point d’expirer, Event Grid est averti de la modification de l’état et lance une requête HTTP POST sur le point de terminaison. Un Webhook déclenche ensuite une exécution Automation d’un script PowerShell.

![Organigramme HTTP POST](../media/event-grid-tutorial-1.png)

## <a name="create-an-automation-account"></a>Créer un compte Automation

Créez un compte Automation via le [portail Azure](https://portal.azure.com) :

1.  Accédez à portal.azure.com et connectez-vous à votre abonnement.

1.  Dans la zone de recherche, entrez **Comptes Automation**.

1.  Dans la section **Services** de la liste déroulante de la barre de recherche, sélectionnez **Comptes Automation**.

1.  Sélectionnez **Ajouter**.

    ![Volet Comptes Automation](../media/event-grid-tutorial-2.png)

1.  Entrez les informations nécessaires dans le volet **Ajouter un compte Automation**, puis sélectionnez **Créer**.

## <a name="create-a-runbook"></a>Créer un runbook

Une fois votre compte Automation prêt, créez un runbook.

![Créer une IU de runbook](../media/event-grid-tutorial-3.png)

1.  Sélectionnez le compte Automation que vous venez de créer.

1.  Sélectionnez **Runbooks** sous **Automatisation des processus**.

1.  Sélectionnez **Créer un runbook**.

1.  Donnez un nom à votre runbook et sélectionnez **PowerShell** comme type de runbook.

1.  Sélectionnez le runbook que vous avez créé, puis sélectionnez le bouton **Modifier**.

1.  Entrez le code suivant (à des fins de test), puis sélectionnez le bouton **Publier**. Cette action retourne le résultat de la requête POST reçue.

```azurepowershell
param
(
[Parameter (Mandatory = $false)]
[object] $WebhookData
)

#If runbook was called from Webhook, WebhookData will not be null.
if ($WebhookData) {

#rotate secret:
#generate new secret version in key vault
#update db/service with generated secret

#Write-Output "WebhookData <$WebhookData>"
Write-Output $WebhookData.RequestBody
}
else
{
# Error
write-Error "No input data found." 
}
```

![Publier l’IU du runbook](../media/event-grid-tutorial-4.png)

## <a name="create-a-webhook"></a>Créer un webhook

Créez un Webhook pour déclencher le runbook que vous venez de créer.

1.  Sélectionnez **Webhooks** dans la section **Ressources** du runbook que vous venez de publier.

1.  Sélectionnez **Ajouter un Webhook**.

    ![Bouton Ajouter un Webhook](../media/event-grid-tutorial-5.png)

1.  Sélectionnez **Créer un Webhook**.

1. Nommez le Webhook, définissez une date d’expiration, puis copiez l’URL.

    > [!IMPORTANT] 
    > Vous ne pouvez pas voir l’URL une fois que vous l’avez créée. Veillez à en enregistrer une copie à un emplacement sûr et accessible pour le reste de ce guide.

1. Sélectionnez **Paramètres et valeurs pour l’exécution**, puis **OK**. N’entrez aucun paramètre. Cela permet d’activer le bouton **Créer**.

1. Sélectionnez **OK**, puis **Créer**.

    ![Créer une IU de Webhook](../media/event-grid-tutorial-6.png)

## <a name="create-an-event-grid-subscription"></a>Créer un abonnement Event Grid

Créez un abonnement Event Grid sur le [Portail Azure](https://portal.azure.com).

1.  Accédez à votre coffre de clés et sélectionnez l’onglet **Événements**.

    ![Onglet Événements du portail Azure](../media/event-grid-tutorial-7.png)

1.  Sélectionnez le bouton **Abonnement aux événements**.

1.  Donnez un nom descriptif à l’abonnement.

1.  Choisissez **Schéma Event Grid**.

1.  La **ressource de rubrique** doit être le coffre de clés dont vous souhaitez superviser les changements d’état.

1.  Pour **Filtrer sur les types d’événement**, laissez toutes les options sélectionnées (**9 sélectionnées**).

1.  Pour **Type de point de terminaison**, sélectionnez **Webhook**.

1.  Choisissez **Sélectionner un point de terminaison**. Dans le nouveau volet contextuel, collez l’URL de Webhook de l’étape [Créer un Webhook](#create-a-webhook) dans le champ **Point de terminaison de l’abonné**.

1.  Sélectionnez **Confirmer la sélection** dans le volet contextuel.

1.  Sélectionnez **Create** (Créer).

    ![Créer un abonnement aux événements](../media/event-grid-tutorial-8.png)

## <a name="test-and-verify"></a>Tester et vérifier

Vérifiez que votre abonnement Event Grid est correctement configuré. Ce test suppose que vous êtes abonné à la notification « Version du secret créée » dans [Créer un abonnement Event Grid](#create-an-event-grid-subscription), et que vous disposez des autorisations nécessaires pour créer une version d’un secret dans un coffre de clés.

![Tester la configuration de l’abonnement Event Grid](../media/event-grid-tutorial-9.png)

![Volet Créer un secret](../media/event-grid-tutorial-10.png)

1.  Accédez à votre coffre de clés dans le portail Azure.

1.  Créer un secret. À des fins de test, définissez la date d’expiration au jour suivant.

1.  Sous l’onglet **Événements** de votre coffre de clés, sélectionnez l’abonnement Event Grid que vous avez créé.

1.  Sous **Métriques**, vérifiez si un événement a été capturé. Deux événements sont attendus : SecretNewVersion et SecretNearExpiry. Ces événements confirment que Event Grid a correctement capturé le changement d’état du secret dans votre coffre de clés.

    ![Volet Métriques : recherche des événements capturés](../media/event-grid-tutorial-11.png)

1.  Accédez à votre compte Automation.

1.  Sélectionnez l’onglet **Runbooks**, puis sélectionnez le runbook que vous avez créé.

1.  Sélectionnez l’onglet **Webhooks**, puis vérifiez que l’horodatage du « dernier déclenchement » se situe dans un délai de 60 secondes après la création du secret. Ce résultat confirme qu’Event Grid a envoyé une requête POST au Webhook en indiquant les détails de l’événement de changement d’état relatif à votre coffre de clés, et que le Webhook a été déclenché.

    ![Onglet Webhooks, horodatage du dernier déclenchement](../media/event-grid-tutorial-12.png)

1. Retournez à votre runbook, puis sélectionnez l’onglet **Vue d’ensemble**.

1. Examinez la liste **Tâches récentes**. Vous devriez voir qu’une tâche a été créée et que l’état est terminé. Cela confirme que le webhook a déclenché le runbook pour commencer l’exécution de son script.

    ![Liste Tâches récente du Webhook](../media/event-grid-tutorial-13.png)

1. Sélectionnez la tâche récente, puis examinez la requête POST envoyée au Webhook par Event Grid. Examinez le JSON et vérifiez que les paramètres de votre coffre de clés et du type d’événement sont corrects. Si le paramètre « type d’événement » de l’objet JSON correspond à l’événement qui s’est produit dans le coffre de clés (dans cet exemple, Microsoft.KeyVault.SecretNearExpiry), le test a réussi.

## <a name="troubleshooting"></a>Dépannage

### <a name="you-cant-create-an-event-subscription"></a>Vous ne pouvez pas créer un abonnement à un événement

Réinscrivez Event Grid et le fournisseur de coffres de clés auprès de vos fournisseurs de ressources d’abonnement Azure. Voir [Fournisseurs et types de ressources Azure](../../azure-resource-manager/management/resource-providers-and-types.md).

## <a name="next-steps"></a>Étapes suivantes

Félicitations ! Si vous avez correctement suivi toutes ces étapes, vous êtes maintenant prêt à répondre par programmation aux changements d’état des secrets stockés dans votre coffre de clés.

Si vous avez utilisé un système d’interrogation pour rechercher les changements d’état des secrets dans vos coffres de clés, vous pouvez désormais vous servir de cette fonctionnalité de notification. Vous pouvez également remplacer le script de test de votre runbook par du code pour renouveler vos secrets par programmation au moment où ils sont sur le point d’expirer.

En savoir plus :


- Présentation : [Supervision de Key Vault avec Azure Event Grid](event-grid-overview.md)
- Procédure : [Recevoir un e-mail en cas de changement d’un secret de coffre de clés](event-grid-logicapps.md)
- [Schéma des événements Azure Event Grid pour Azure Key Vault](../../event-grid/event-schema-key-vault.md)
- [Vue d’ensemble d’Azure Key Vault](overview.md)
- [Vue d’ensemble d’Azure Event Grid](../../event-grid/overview.md)
- [Vue d’ensemble d’Azure Automation](../../automation/index.yml)
