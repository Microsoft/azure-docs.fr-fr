---
title: Démarrage rapide Azure - Exécuter votre premier travail Batch sur le portail Azure
description: Ce guide de démarrage rapide montre comment utiliser le portail Azure pour créer un compte Batch, un pool de nœuds de calcul et un travail qui exécute des tâches de base sur le pool.
ms.topic: quickstart
ms.date: 08/17/2020
ms.custom: mvc
ms.openlocfilehash: d8584ecea8ddd5934771dbe5945a6172a28c801c
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106473"
---
# <a name="quickstart-run-your-first-batch-job-in-the-azure-portal"></a>Démarrage rapide : exécution de votre premier travail Batch dans le portail Azure

Familiarisez-vous avec Azure Batch en utilisant le portail Azure pour créer un compte Batch, un pool de nœuds de calcul (machines virtuelles) et un travail qui exécute des tâches sur le pool. À l'issue de ce démarrage rapide, vous maîtriserez les concepts clés du service Batch et serez prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-batch-account"></a>Création d’un compte Batch

Suivez ces étapes pour créer un compte Batch d’exemple à des fins de test. Vous avez besoin d’un compte Batch pour créer des pools et des travaux. Comme indiqué ici, vous pouvez lier un compte de stockage Azure avec le compte Batch. Bien que ce ne soit pas obligatoire pour ce démarrage rapide, le compte de stockage est utile pour déployer des applications et stocker des données d’entrée et de sortie pour la plupart des charges de travail réelles.

1. Sur le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Compute** > **Service Batch**. 

   :::image type="content" source="media/quick-create-portal/marketplace-batch.png" alt-text="Capture d'écran du service Batch sur la Place de marché Azure":::

1. Dans le champ **Groupe de ressources**, sélectionnez **Créer** et entrez un nom pour votre groupe de ressources.

1. Entrez une valeur dans le champ **Nom du compte**. Ce nom doit être unique au sein de l'**Emplacement** Azure sélectionné. Il ne peut contenir que des minuscules et des chiffres, et doit comporter 3 à 24 caractères.

1. Sous **Compte de stockage**, sélectionnez un compte de stockage existant ou créez-en un.

1. Ne modifiez aucun autre paramètre. Sélectionnez **Vérifier + créer**, puis **Créer** pour créer le compte Batch.

Lorsque le message **Déploiement réussi** s'affiche, accédez au compte Batch que vous avez créé.

## <a name="create-a-pool-of-compute-nodes"></a>Créer un pool de nœuds de calcul

Maintenant que vous avez un compte Batch, créez un pool d’exemple de nœuds de calcul Windows à des fins de test. Le pool créé pour cet exemple rapide se compose de deux nœuds exécutant une image Windows Server 2019 à partir de la Place de marché Microsoft Azure.

1. Dans le compte Batch, cliquez sur **Pools** > **Ajouter**.

1. Entrez un **ID de Pool** appelé *mypool*.

1. Dans **Système d’exploitation**, sélectionnez les paramètres suivants (vous pouvez explorer d’autres options).
  
   |Paramètre  |Value  |
   |---------|---------|
   |**Type d’image**|Place de marché|
   |**Publisher**     |microsoftwindowsserver|
   |**Offre**     |windowsserver|
   |**Sku**     |2019-datacenter-core-smalldisk|

1. Faites défiler pour accéder aux paramètres **Taille du nœud** et **Mise à l’échelle**. La taille de nœud suggérée offre un bon compromis entre performances et coûts pour cet exemple rapide.
  
   |Paramètre  |Value  |
   |---------|---------|
   |**Niveau de tarification du nœud**     |Standard A1|
   |**Nœuds dédiés cibles**     |2|

1. Conservez les valeurs par défaut pour les paramètres restants, puis cliquez sur **OK** pour créer le pool.

Azure Batch crée le pool immédiatement, mais prend quelques minutes pour allouer et démarrer les nœuds de calcul. Pendant ce temps, l’**état de l’allocation** du pool est **Redimensionnement**. Vous pouvez commencer et créer un travail ainsi que des tâches alors que le pool est en redimensionnement.

Au bout de quelques minutes, l'état d'allocation passe à **Stable** et les nœuds démarrent. Pour vérifier l'état des nœuds, sélectionnez le pool, puis choisissez **Nœuds**. Lorsque l’état d’un nœud est **Inactif**, il est prêt à exécuter des tâches.

## <a name="create-a-job"></a>Créer un travail

Maintenant que vous disposez d’un pool, créez un travail à exécuter sur celui-ci. Un travail Batch est un groupe logique d’une ou de plusieurs tâches. Un travail inclut les paramètres communs aux tâches, tels que la priorité et le pool pour exécuter des tâches. Dans un premier temps, le travail n’a aucune tâche.

1. Dans la vue de compte Batch, sélectionnez **Travaux** > **Ajouter**.

1. Entrez un **ID de travail** appelé *myjob*. Dans **Pool**, sélectionnez *mypool*. Conservez les valeurs par défaut pour les paramètres restants, puis cliquez sur **OK**.

## <a name="create-tasks"></a>Créer des tâches

À présent, sélectionnez le travail pour ouvrir la page **Tâches**. Vous allez créer des exemples de tâches à exécuter dans le travail. En général, vous créez plusieurs tâches qu'Azure Batch met en file d'attente et distribue pour les exécuter sur les nœuds de calcul. Dans cet exemple, vous créez deux tâches identiques. Chaque tâche exécute une ligne de commande pour afficher les variables d’environnement Azure Batch sur un nœud de calcul, puis attend 90 secondes.

Lorsque vous utilisez Azure Batch, la ligne de commande se trouve là où vous spécifiez votre application ou votre script. Azure Batch fournit plusieurs façons de déployer des applications et des scripts sur des nœuds de calcul.

Pour créer la première tâche :

1. Sélectionnez **Ajouter**.

1. Entrez un **ID de tâche** appelé *mytask*.

1. Dans **Ligne de commande**, entrez `cmd /c "set AZ_BATCH & timeout /t 90 > NUL"`. Conservez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Envoyer**.

Après avoir créé une tâche, Azure Batch la met en file d’attente pour l’exécuter sur le pool. Lorsqu’un nœud est disponible pour l’exécuter, la tâche s’exécute.

Pour créer une deuxième tâche, répétez les étapes ci-dessus. Entrez un autre **ID de tâche**, mais spécifiez une ligne de commande identique. Si la première tâche est en cours d’exécution, Azure Batch démarre la deuxième tâche sur l’autre nœud dans le pool.

## <a name="view-task-output"></a>Afficher la sortie des tâches

Les exemples de tâches que vous avez créés seront terminés en quelques minutes. Pour afficher la sortie d'une tâche terminée, sélectionnez la tâche, puis choisissez **Fichiers sur le nœud**. Sélectionnez le fichier `stdout.txt` pour visualiser la sortie standard de la tâche. Le contenu ressemble à ce qui suit :

:::image type="content" source="media/quick-create-portal/task-output.png" alt-text="Capture d'écran de la sortie d'une tâche terminée.":::

Le contenu affiche les variables d’environnement Azure Batch qui sont définies sur le nœud. Lorsque vous créez vos propres travaux Batch et tâches, vous pouvez référencer ces variables d’environnement dans des lignes de commande de tâche, ainsi que dans les applications et les scripts exécutés par les lignes de commande.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez poursuivre les exemples et didacticiels Azure Batch, utilisez le compte Batch et le compte de stockage lié créés dans ce démarrage rapide. Il n’existe aucun frais pour le compte Batch proprement dit.

Vous êtes facturé pour le pool pendant que les nœuds sont en cours d’exécution, même si aucun travail n’est planifié. Lorsque vous n’avez plus besoin du pool, supprimez-le. Dans la vue de compte, sélectionnez **Pools** et le nom du pool. Puis sélectionnez **Supprimer**.  Lorsque vous supprimez le pool, toutes les sorties de tâche sur les nœuds sont supprimées.

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, le compte Batch et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources pour le compte Batch, puis cliquez sur **Supprimer le groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un compte Batch, un pool Batch et un travail Batch. Le travail a exécuté des tâches d’exemple et vous avez affiché des sorties créées sur un des nœuds. Maintenant que vous maîtriserez les concepts clés du service Batch, vous êtres prêt à essayer Azure Batch avec des charges de travail plus réalistes à plus grande échelle. Pour plus d’informations sur Microsoft Azure Batch, consultez les didacticiels Azure Batch.

> [!div class="nextstepaction"]
> [Didacticiels Azure Batch](./tutorial-parallel-dotnet.md)
