---
title: Démarrage rapide - Créer un hub de notification Azure avec Azure CLI | Microsoft Docs
description: Dans ce tutoriel, apprenez à créer un hub de notification Azure avec Azure CLI.
services: notification-hubs
author: dbradish-microsoft
manager: barbkess
editor: sethmanheim
ms.service: notification-hubs
ms.devlang: azurecli
ms.workload: mobile
ms.topic: quickstart
ms.date: 05/27/2020
ms.author: dbradish
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6c565be3aea43fc4dc86fd793e0dc2dea0b08d4c
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563747"
---
# <a name="quickstart-create-an-azure-notification-hub-using-the-azure-cli"></a>Créer un hub de notification Azure avec Azure CLI

Azure Notification Hubs fournit un moteur d’envoi de notifications Push facile à utiliser et à grande échelle qui vous permet d’envoyer des notifications à n’importe quelle plateforme (iOS, Android, Windows, Kindle, Baidu, etc.) à partir de n’importe quel serveur principal (cloud ou local). Pour plus d’informations sur le service, consultez [Présentation d’Azure Notification Hubs](notification-hubs-push-notification-overview.md).

Dans ce guide de démarrage rapide, vous créez un hub de notification avec Azure CLI. La première section présente les étapes à suivre pour créer un espace de noms Notification Hubs. La deuxième section vous présente les étapes à suivre pour créer un hub de notification dans un espace de noms existant. Vous allez également apprendre à créer une stratégie d’accès personnalisée.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

> [!IMPORTANT]
> Notification Hubs nécessite la version 2.0.67 ou ultérieure d’Azure CLI. Exécutez [az version](/cli/azure/reference-index#az_version) pour rechercher la version et les bibliothèques dépendantes installées. Pour effectuer une mise à niveau vers la dernière version, exécutez [az upgrade](/cli/azure/reference-index#az_upgrade).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Azure Notification Hubs, comme toutes les ressources Azure, doit être déployé dans un groupe de ressources.  Les groupes de ressources vous permettent d’organiser et de gérer les ressources Azure connexes.  Pour en savoir plus sur les groupes de ressources, consultez [Qu’est-ce qu’Azure Resource Manager ?](../azure-resource-manager/management/overview.md)

Pour ce guide de démarrage rapide, créez un groupe de ressources nommé **spnhubrg** dans la zone **eastus** à l’aide de la commande [az group create](/cli/azure/group#az-group-create) suivante.

```azurecli
az group create --name spnhubrg --location eastus
```

## <a name="create-a-notification-hubs-namespace"></a>Créer un espace de noms Notification Hubs

1. Créez un espace de noms pour vos hubs de notification.

   Un espace de noms contient un ou plusieurs hubs. Le nom doit être unique dans tous les abonnements Azure et doit comporter au moins six caractères. Pour vérifier la disponibilité d’un nom, utilisez la commande [az notification-hub namespace check-availability](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-check-availability).

   ```azurecli
   az notification-hub namespace check-availability --name spnhubns
   ```

   Azure CLI répond à votre demande de disponibilité en affichant la sortie de console suivante :

   ```shell
   {
   "id": "/subscriptions/yourSubscriptionID/providers/Microsoft.NotificationHubs/checkNamespaceAvailability",
   "isAvailable": true,
   "location": null,
   "name": "spnhubns",
   "properties": false,
   "sku": null,
   "tags": null,
   "type": "Microsoft.NotificationHubs/namespaces/checkNamespaceAvailability"
   }
   ```

   Observez la deuxième ligne de la réponse d’Azure CLI `"isAvailable": true`. Cette ligne indique `false` si le nom que vous avez spécifié pour l’espace de noms n’est pas disponible. Après avoir vérifié la disponibilité du nom, exécutez la commande [az notification-hub namespace create](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-create) pour créer l’espace de noms.  

   ```azurecli
   az notification-hub namespace create --resource-group spnhubrg --name spnhubns  --location eastus --sku Free
   ```

   Si le `--name` que vous avez indiqué dans la commande `az notification-hub namespace create` n’est pas disponible ou ne respecte pas les [les règles et les restrictions de nommage pour les ressources Azure](../azure-resource-manager/management/resource-name-rules.md), Azure CLI répond avec la sortie de console suivante :

   ```shell
   #the name is not available
   The specified name is not available. For more information visit https://aka.ms/eventhubsarmexceptions.

   #the name is invalid
   The specified service namespace is invalid.
   ```

   Si le premier nom que vous avez essayé n’est pas correct, sélectionnez un autre nom pour votre nouvel espace de noms et réexécutez la commande `az notification-hub namespace create`.

   > [!NOTE]
   > À partir de cette étape, vous devez remplacer la valeur du paramètre `--namespace` dans chaque commande Azure CLI que vous copiez à partir de ce guide de démarrage rapide.

2. Récupérez la liste des espaces de noms.

   Pour voir les informations relatives à votre nouvel espace de noms, utilisez la commande [az notification-hub namespace list](/cli/azure/ext/notification-hub/notification-hub/namespace#ext-notification-hub-az-notification-hub-namespace-list). Le paramètre facultatif `--resource-group` vous permet de voir tous les espaces de noms d’un abonnement.

   ```azurecli
   az notification-hub namespace list --resource-group spnhubrg
   ```

## <a name="create-notification-hubs"></a>Créer des hubs de notification

1. Créez votre premier hub de notification.

   Vous pouvez désormais créer un ou plusieurs hubs de notification dans le nouvel espace de noms. Exécutez la commande [az notification-hub create](/cli/azure/ext/notification-hub/notification-hub#ext-notification-hub-az-notification-hub-create) pour créer un hub de notification.

   ```azurecli
   az notification-hub create --resource-group spnhubrg --namespace-name spnhubns --name spfcmtutorial1nhub --location eastus --sku Free
   ```

2. Créez un deuxième hub de notification.

   Vous pouvez créer plusieurs hubs de notification dans un seul espace de noms. Pour créer un deuxième hub de notification dans le même espace de noms, réexécutez la commande `az notification-hub create` avec un autre nom de hub.

   ```azurecli
   az notification-hub create --resource-group spnhubrg --namespace-name spnhubns --name mysecondnhub --location eastus --sku Free
   ```

3. Récupérez la liste des hubs de notification.

   Azure CLI retourne un message de réussite ou d’erreur avec chaque commande exécutée. La possibilité de demander une liste des hubs de notification est cependant rassurante. La commande [az notification-hub list](/cli/azure/ext/notification-hub/notification-hub#ext-notification-hub-az-notification-hub-list) a été conçue à cet effet.

   ```azurecli
   az notification-hub list --resource-group spnhubrg --namespace-name spnhubns --output table
   ```

## <a name="work-with-access-policies"></a>Utiliser des stratégies d’accès

1. Azure Notification Hubs utilise la [sécurité de signature d’accès partagé](./notification-hubs-push-notification-security.md) par le biais de stratégies d’accès. Deux stratégies sont créées automatiquement quand vous créez un hub de notification. Les chaînes de connexion de ces stratégies sont nécessaires pour configurer les notifications Push. La commande [az notification-hub authorization-rule list](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list) fournit une liste de noms de stratégie avec les groupes de ressources correspondants.

   ```azurecli
   az notification-hub authorization-rule list --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --output table
   ```

   > [!IMPORTANT]
   > N’utilisez pas la stratégie _DefaultFullSharedAccessSignature_ dans votre application. Elle est destinée à être utilisée seulement dans votre back-end. Utilisez uniquement les stratégies d’accès `Listen` dans votre application cliente.

2. Si vous voulez créer des règles d’autorisation supplémentaires avec des noms explicites, vous pouvez créer et personnaliser votre propre stratégie d’accès en utilisant la commande [az notification-hub authorization-rule create](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-create). Le paramètre `--rights` est une liste des autorisations à attribuer délimitées par des espaces.

   ```azurecli
   az notification-hub authorization-rule create --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name spnhub1key --rights Listen Manage Send
   ```

3. Il existe deux jeux de clés et de chaînes de connexion pour chaque stratégie d’accès. Vous en aurez besoin pour [configurer un hub de notification](./configure-notification-hub-portal-pns-settings.md). Pour lister les clés et les chaînes de connexion d’une stratégie d’accès Notification Hubs, utilisez la commande [az notification-hub authorization-rule list-keys](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list-keys).

   ```azurecli
   # query the keys and connection strings for DefaultListenSharedAccessSignature
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name DefaultListenSharedAccessSignature --output table
   ```

   ```azurecli
   # query the keys and connection strings for a custom policy
   az notification-hub authorization-rule list-keys --resource-group spnhubrg --namespace-name spnhubns --notification-hub-name spfcmtutorial1nhub --name spnhub1key --output table
   ```

   > [!NOTE]
   > Un [espace de noms Notification Hubs](/cli/azure/ext/notification-hub/notification-hub/namespace/authorization-rule#ext-notification-hub-az-notification-hub-namespace-authorization-rule-list-keys) et un [hub de notification](/cli/azure/ext/notification-hub/notification-hub/authorization-rule#ext-notification-hub-az-notification-hub-authorization-rule-list-keys) ont des stratégies d’accès distinctes. Vérifiez que vous utilisez la commande de référence Azure CLI appropriée quand vous recherchez des clés et des chaînes de connexion.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, utilisez la commande [az group delete](/cli/azure/group) pour supprimer ce groupe ainsi que toutes les ressources associées :

```azurecli
az group delete --name spnhubrg
```

## <a name="next-steps"></a>Étapes suivantes

* Dans ce guide de démarrage rapide, vous avez créé un hub de notification. Pour savoir comment configurer le hub avec les paramètres PNS (Platform Notification System), consultez [Configurer les notifications Push dans un hub de notification](configure-notification-hub-portal-pns-settings.md).

* Découvrez les fonctionnalités étendues de gestion des hubs de notification avec Azure CLI :

  [Liste de référence complète des commandes relatives à Notification Hubs](/cli/azure/ext/notification-hub/notification-hub)

  [Liste de référence des commandes relatives aux espaces des noms Notification Hubs](/cli/azure/ext/notification-hub/notification-hub/namespace)

  [Liste de référence des commandes relatives aux règles d’autorisation Notification Hubs](/cli/azure/ext/notification-hub/notification-hub/authorization-rule)

  [Liste de référence des commandes relatives aux informations d’identification Notification Hubs](/cli/azure/ext/notification-hub/notification-hub/credential)