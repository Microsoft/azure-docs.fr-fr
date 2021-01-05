---
title: Déplacer un espace de noms Azure Event Hubs vers une autre région | Microsoft Docs
description: Cet article explique comment déplacer un espace de noms Azure Event Hubs de la région actuelle vers une autre région.
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: b177c3916919e3d97325f9d8c6b6027c00cb476f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019920"
---
# <a name="move-an-azure-event-hubs-namespace-to-another-region"></a>Déplacer un espace de noms Azure Event Hubs vers une autre région
Cet article vous explique comment exporter un modèle Azure Resource Manager pour un espace de noms Event Hubs existant, puis comment utiliser ce modèle pour créer un espace de noms avec les mêmes paramètres de configuration dans une autre région. Toutefois, ce processus ne déplace pas les événements qui n’ont pas encore été traités. Vous devez traiter les événements à partir de l’espace de noms d’origine avant de le supprimer.
 
Si vous avez d’autres ressources dans le groupe de ressources Azure qui contient l’espace de noms Event Hubs, vous pouvez exporter le modèle au niveau du groupe de ressources afin que toutes les ressources associées puissent être déplacées vers la nouvelle région en une seule étape. Les étapes décrites dans cet article montrent comment exporter un **espace de noms** vers le modèle. Les étapes d’exportation d’un **groupe de ressources** vers le modèle sont similaires. 

## <a name="prerequisites"></a>Prérequis

- Vérifiez que les services et fonctionnalités utilisés par votre compte sont pris en charge dans la région cible.
- Si vous avez activé la **fonctionnalité de capture** pour les hubs d’événements dans l’espace de noms, déplacez les comptes [Azure Storage ou Azure Data Lake Store Gen 2](../storage/common/storage-account-move.md) ou [Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-migration-cross-region.md) avant de déplacer l’espace de noms Event Hubs. Vous pouvez également déplacer le groupe de ressources contenant à la fois les espaces de noms Storage et Event Hubs vers l’autre région en suivant les étapes similaires à celles décrites dans cet article. 
- Si l’espace de noms Event Hubs se trouve dans un **cluster Event Hubs**, [déplacez le cluster dédié](move-cluster-across-regions.md) vers la **région cible** avant de suivre les étapes de cet article. Vous pouvez également utiliser le [modèle de démarrage rapide sur GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-eventhubs-create-cluster-namespace-eventhub/) pour créer un cluster Event Hubs. Dans le modèle, supprimez la partie Espace de noms du JSON pour créer uniquement le cluster. 

## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager. Ce modèle contient des paramètres qui décrivent votre espace de noms Event Hubs.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Choisissez **Toutes les ressources**, puis sélectionnez votre espace de noms Event Hubs.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Télécharger** dans le panneau **Exporter le modèle**.

    ![Télécharger un modèle Resource Manager](./media/move-across-regions/download-template.png)
5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix.

   Ce fichier zip contient les fichiers .json qui incluent le modèle et des scripts pour le déployer.


## <a name="move"></a>Déplacer

Déployez le modèle pour créer un espace de noms Event Hubs dans la région cible. 


1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Rechercher dans la Place de marché**, tapez **déploiement de modèle**, puis sélectionnez **Déploiement de modèle (déployer à l’aide de modèles personnalisés)** .
5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.
6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.
1. Mettez à jour la valeur de la propriété `location` pour qu’elle pointe vers la nouvelle région. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.
1. Sélectionnez **Enregistrer** pour enregistrer le modèle. 
1. Dans la page **Déploiement personnalisé**, procédez comme suit : 
    1. Sélectionnez un **abonnement** Azure. 
    2. Sélectionnez un **groupe de ressources** ou créez-en un. Si l’espace de noms source figurait dans un cluster Event Hubs, sélectionnez le groupe de ressources contenant le cluster dans la région cible. 
    3. Sélectionnez l’**emplacement** ou la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule. 
    4. Dans la section **SETTINGS** , procédez comme suit :    
        1. Entrez le nouveau **nom de l’espace de noms**. 

            ![Déployer le modèle Azure Resource Manager](./media/move-across-regions/deploy-template.png)
        2. Si votre espace de noms source figurait dans un **cluster Event Hubs**, entrez les noms du **groupe de ressources** et du **cluster Event Hubs** dans l’**ID externe**. 

              ```
              /subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<CLUSTER'S RESOURCE GROUP>/providers/Microsoft.EventHub/clusters/<CLUSTER NAME>
              ```   
        3. Si le hub d’événements de votre espace de noms utilise un compte de stockage pour capturer des événements, spécifiez le nom du groupe de ressources et le compte de stockage pour le champ `StorageAccounts_<original storage account name>_external`. 
            
            ```
            /subscriptions/0000000000-0000-0000-0000-0000000000000/resourceGroups/<STORAGE'S RESOURCE GROUP>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>
            ```    
    5. Au bas de la page, sélectionnez **Examiner et créer**. 
    1. Dans la page **Vérifier + créer**, passez en revue les paramètres, puis sélectionnez **Créer**.   

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Après le déploiement, si vous souhaitez recommencer, vous pouvez supprimer l’**espace de noms Event Hubs cible** et répéter les étapes décrites dans les sections [Préparer](#prepare) et [Déplacer](#move) de cet article.

Pour valider les modifications et terminer le déplacement d’un espace de noms Event Hubs, supprimez l’**espace de noms Event Hubs** dans la région d’origine. Assurez-vous que vous avez traité tous les événements dans l’espace de noms avant de supprimer cet espace de noms. 

Pour supprimer un espace de noms Event Hubs (source ou cible) à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, tapez **Event Hubs**, puis sélectionnez **Event hubs** dans les résultats de la recherche. Les espaces de noms Event Hubs apparaissent dans une liste.
2. Sélectionnez l’espace de noms cible à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 

    ![Supprimer un espace de noms - bouton](./media/move-across-regions/delete-namespace-button.png)
3. Sur la page **supprimer un espace de noms**, confirmez la suppression en tapant le **nom de l’espace de noms**, puis sélectionnez **Supprimer**. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un espace de noms Azure Event Hubs d’une région vers une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)
