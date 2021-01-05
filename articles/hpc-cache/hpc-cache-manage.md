---
title: Gérer et mettre à jour Azure HPC Cache
description: Comment gérer et mettre à jour Azure HPC Cache à l’aide du portail Azure ou d’Azure CLI
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 08/31/2020
ms.author: v-erkel
ms.openlocfilehash: 067b12d4dcfd5ba2b730204ef680b900d79f1b72
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94648070"
---
# <a name="manage-your-cache"></a>Gérer votre cache

La page aperçu du cache du Portail Azure affiche les détails du projet, l’état du cache et les statistiques de base de votre cache. Elle dispose également de contrôles pour arrêter ou démarrer le cache, supprimer le cache, vider les données dans un stockage à long terme et mettre à jour les logiciels.

Cet article explique également comment effectuer ces tâches de base à l’aide d’Azure CLI.

Pour ouvrir la page d’aperçu, sélectionnez votre ressource de cache dans le Portail Azure. Par exemple, chargez la page **Toutes les ressources** et cliquez sur le nom du cache.

![capture d’écran d’une page d’aperçu de l’instance de cache Azure HPC](media/hpc-cache-overview.png)

Les boutons situés en haut de la page peuvent vous aider à gérer le cache :

* **Démarrer** et [**Arrêter**](#stop-the-cache) : reprend ou interrompt l’opération de mise en cache
* [**Vider**](#flush-cached-data) : écrit les données modifiées dans les cibles de stockage
* [**Mettre à niveau**](#upgrade-cache-software) : met à jour le logiciel de cache
* [**Collecter des diagnostics**](#collect-diagnostics) : charge les informations de débogage
* **Actualiser** : recharge la page d’aperçu
* [**Supprimer**](#delete-the-cache) : supprime définitivement le cache

Apprenez-en davantage sur ces options ci-dessous.

Cliquez sur l’image ci-dessous pour regarder une [vidéo](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) illustrant les tâches de gestion du cache.

[![Miniature de vidéo : Azure HPC Cache : Gestion (cliquez pour visiter la page vidéo)](media/video-5-manage.png)](https://azure.microsoft.com/resources/videos/managing-hpc-cache/)

## <a name="stop-the-cache"></a>Arrêter le cache

Vous pouvez arrêter le cache pour réduire les coûts pendant une période inactive. Vous n’êtes pas facturé pour la durée de fonctionnement lorsque le cache est arrêté, mais vous l’êtes pour le stockage sur disque alloué au cache. (Pour plus d’informations, voir la page de [tarification](https://aka.ms/hpc-cache-pricing).)

Un cache arrêté ne répond pas aux requêtes des clients. Vous devez démonter les clients avant d’arrêter le cache.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Le bouton **Arrêter** interrompt un cache actif. Le bouton **Arrêter** est disponible lorsque l’état d’un cache est **Sain** ou **Détérioré**.

![capture d’écran des boutons du haut avec Arrêter mis en surbrillance et un message contextuel décrivant l’action d’arrêt et demandant « voulez-vous continuer ? » avec les boutons Oui (par défaut) et Non](media/stop-cache.png)

Une fois que vous avez cliqué sur Oui pour confirmer l’arrêt du cache, le cache vide automatiquement son contenu dans les cibles de stockage. Ce processus peut prendre un certain temps, mais il garantit la cohérence des données. Enfin, l’état du cache passe à **Arrêté**.

Pour réactiver un cache arrêté, cliquez sur le bouton **Démarrer**. Aucune confirmation n’est nécessaire.

![capture d’écran des boutons du haut avec Démarrer mis en surbrillance](media/start-cache.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Suspendez temporairement un cache avec la commande [az hpc-cache stop](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-stop). Cette action est valide uniquement lorsque l’état d’un cache est **Sain** ou **Détérioré**.

Le cache vide automatiquement son contenu dans les cibles de stockage avant de s’arrêter. Ce processus peut prendre un certain temps, mais il garantit la cohérence des données.

Lorsque l’action est terminée, l’état du cache prend la valeur **Arrêté**.

Réactivez un cache arrêté avec la commande [az hpc-cache start](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-start).

Lorsque vous lancez la commande de démarrage ou d’arrêt, la ligne de commande affiche un message d’état « En cours d’exécution » jusqu’à ce que l’opération se termine.

```azurecli
$ az hpc-cache start --name doc-cache0629
 - Running ..
```

Une fois l’opération terminée, le message est mis à jour avec la valeur « Terminé » et affiche les codes de retour et d’autres informations.

```azurecli
$ az hpc-cache start --name doc-cache0629
{- Finished ..
  "endTime": "2020-07-01T18:46:43.6862478+00:00",
  "name": "c48d320f-f5f5-40ab-8b25-0ac065984f62",
  "properties": {
    "output": "success"
  },
  "startTime": "2020-07-01T18:40:28.5468983+00:00",
  "status": "Succeeded"
}
```

---

## <a name="flush-cached-data"></a>Vider les données en cache

Le bouton **Vider** de la page d’aperçu indique au cache d’écrire immédiatement toutes les données modifiées stockées dans le cache sur les cibles de stockage back-end. Le cache enregistre régulièrement les données sur les cibles de stockage. il n’est donc pas nécessaire de le faire manuellement, sauf si vous souhaitez vous assurer que le système de stockage back-end est à jour. Par exemple, vous pouvez utiliser **Vider** avant de créer un instantané de stockage ou de vérifier la taille du jeu de données.

> [!NOTE]
> Pendant le processus de vidage, le cache ne peut pas traiter les demandes des clients. L’accès au cache est suspendu et reprend une fois l’opération terminée.

Lorsque vous démarrez l’opération de vidage du cache, le cache cesse d’accepter les requêtes du client et l’état du cache sur la page d’aperçu passe à **Vidage**.

Les données du cache sont enregistrées dans les cibles de stockage appropriées. Selon la quantité de données qui doit être vidée, le processus peut prendre quelques minutes ou plus d’une heure.

Une fois que toutes les données ont été enregistrées sur les cibles de stockage, le cache recommence automatiquement à réexécuter les requêtes des clients. L’état du cache revient à **Sain**.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Pour vider le cache, cliquez sur le bouton **Vider**, puis cliquez sur **Oui** pour confirmer l’action.

![capture d’écran des boutons du haut avec Vider mis en surbrillance et un message contextuel décrivant l’action de vidage et demandant « voulez-vous continuer ? » avec les boutons Oui (par défaut) et Non](media/hpc-cache-flush.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Utilisez [az hpc-cache flush](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-flush) pour forcer le cache à écrire toutes les données modifiées dans les cibles de stockage.

Exemple :

```azurecli
$ az hpc-cache flush --name doc-cache0629 --resource-group doc-rg
 - Running ..
```

Lorsque le vidage est terminé, un message de réussite est retourné.

```azurecli
{- Finished ..
  "endTime": "2020-07-09T17:26:13.9371983+00:00",
  "name": "c22f8e12-fcf0-49e5-b897-6a6e579b6489",
  "properties": {
    "output": "success"
  },
  "startTime": "2020-07-09T17:25:21.4278297+00:00",
  "status": "Succeeded"
}
$
```

---

## <a name="upgrade-cache-software"></a>Mettre à niveau le logiciel du cache

Si une nouvelle version du logiciel est disponible, le bouton **Mettre à niveau** devient actif. Un message peut également s’afficher en haut de la page sur la mise à jour de logiciel.

![capture d’écran de la première ligne de boutons avec le bouton Mettre à niveau activé](media/hpc-cache-upgrade-button.png)

L’accès client n’est pas interrompu pendant une mise à niveau de logiciel, mais les performances du cache ralentissent. Planifiez la mise à niveau des logiciels pendant les heures d’utilisation creuses ou pendant une période de maintenance planifiée.

La mise à jour de logiciel peut prendre plusieurs heures. Les caches configurés avec un débit plus élevé prennent plus de temps à mettre à niveau les caches avec des valeurs de débit de pic plus faibles.

Lorsqu’une mise à niveau de logiciel est disponible, il faudra une semaine environ pour l’appliquer manuellement. La date de fin est indiquée dans le message de mise à niveau. Si vous n’effectuez pas de mise à niveau pendant cette période, Azure applique automatiquement la mise à jour à votre cache. Le minutage de la mise à niveau automatique n’est pas configurable. Si vous vous inquiétez de l’impact sur les performances du cache, vous devez vous-même mettre à niveau le logiciel avant l’expiration du délai.

Si votre cache est arrêté au moment où la date de fin est atteinte, il met à niveau le logiciel automatiquement au prochain démarrage. (Il se peut que la mise à jour ne démarre pas immédiatement. Elle démarrera cependant durant la première heure.)

### <a name="portal"></a>[Portail](#tab/azure-portal)

Cliquez sur le bouton **Mettre à niveau** pour lancer la mise à jour de logiciel. L’état du cache prend la valeur **Mise à niveau** jusqu’à ce que l’opération se termine.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Sur l’interface de ligne de commande Azure, les nouvelles informations relatives au logiciel sont incluses à la fin du rapport d’état du cache. (Utilisez [az hpc-cache show](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-show) pour vérifier.) Recherchez la chaîne « upgradeStatus » dans le message.

Utilisez [az hpc-cache upgrade-firmware](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-upgrade-firmware) pour appliquer la mise à jour, le cas échéant.

Si aucune mise à jour n’est disponible, cette opération n’a aucun effet.

Cet exemple montre l’état du cache (aucune mise à jour n’est disponible) et les résultats de la commande upgrade-firmware.

```azurecli
$ az hpc-cache show --name doc-cache0629
{
  "cacheSizeGb": 3072,
  "health": {
    "state": "Healthy",
    "statusDescription": "The cache is in Running state"
  },

<...>

  "tags": null,
  "type": "Microsoft.StorageCache/caches",
  "upgradeStatus": {
    "currentFirmwareVersion": "5.3.61",
    "firmwareUpdateDeadline": "0001-01-01T00:00:00+00:00",
    "firmwareUpdateStatus": "unavailable",
    "lastFirmwareUpdate": "2020-06-29T22:18:32.004822+00:00",
    "pendingFirmwareVersion": null
  }
}
$ az hpc-cache upgrade-firmware --name doc-cache0629
$
```

---

## <a name="collect-diagnostics"></a>Collecter des diagnostics

Le bouton **Collecter des diagnostics** permet de démarrer manuellement la collecte des informations système et leur chargement dans le service de support Microsoft en vue du dépannage. Le cache collecte et charge automatiquement les mêmes informations de diagnostic en cas de grave problème du cache.

Utilisez ce contrôle si le service de support Microsoft le demande.

Après avoir cliqué sur le bouton, cliquez sur **Oui** pour confirmer le chargement.

![Capture d’écran du message de confirmation contextuel « Démarrer la collecte de diagnostics ». Le bouton par défaut « Oui » est mis en surbrillance.](media/diagnostics-confirm.png)

## <a name="delete-the-cache"></a>Supprimer le cache

Le bouton **Supprimer** détruit le cache. Lorsque vous supprimez un cache, toutes ses ressources sont détruites et n’occasionnent plus de frais de compte.

Les volumes de stockage back-end utilisés comme cibles de stockage ne sont pas affectés quand vous supprimez le cache. Vous pouvez les ajouter ultérieurement à un cache futur ou les désactiver séparément.

> [!NOTE]
> Azure HPC Cache n’écrit pas automatiquement les données modifiées du cache sur les systèmes de stockage back-end avant de supprimer le cache.
>
> Pour vous assurer que toutes les données du cache ont été écrites dans un stockage à long terme, [arrêtez le cache](#stop-the-cache) avant de le supprimer. Assurez-vous qu’il affiche l’état **Arrêté** avant de le supprimer.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Après avoir arrêté le cache, cliquez sur le bouton **Supprimer** pour supprimer définitivement le cache.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

Utilisez la commande [az hpc-cache delete](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-delete) dans l’interface de ligne de commande Azure pour supprimer définitivement le cache.

Exemple :
```azurecli
$ az hpc-cache delete --name doc-cache0629
 - Running ..

<...>

{- Finished ..
  "endTime": "2020-07-09T22:24:35.1605019+00:00",
  "name": "7d3cd0ba-11b3-4180-8298-d9cafc9f22c1",
  "startTime": "2020-07-09T22:13:32.0732892+00:00",
  "status": "Succeeded"
}
$
```

---

## <a name="cache-metrics-and-monitoring"></a>Mesures et supervision du cache

La page d’aperçu affiche des graphiques pour des statistiques de cache de base, le débit du cache, les opérations par seconde et la latence.

![capture d’écran de trois graphiques linéaires montrant les statistiques mentionnées ci-dessus pour un exemple de cache](media/hpc-cache-overview-stats.png)

Ces graphiques font partie des outils de surveillance et d’analyse intégrés d’Azure. Des outils et des alertes supplémentaires sont disponibles à partir des pages sous l’en-tête **Surveillance** dans la barre latérale du portail. Pour plus d’informations, consultez la section portail de la [documentation Surveillance Azure](../azure-monitor/insights/monitor-azure-resource.md#monitoring-in-the-azure-portal).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [outils de mesures et statistiques Azure](../azure-monitor/index.yml)
* Obtenir [de l’aide avec Azure HPC Cache](hpc-cache-support-ticket.md)
