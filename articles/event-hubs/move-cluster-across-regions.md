---
title: Déplacer un cluster dédié Azure Event Hubs vers une autre région | Microsoft Docs
description: Cet article explique comment déplacer un cluster dédié Azure Event Hubs de la région actuelle vers une autre région.
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: 94be44ee8f9442a3a65e899d7a58524b2570f194
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89380362"
---
# <a name="move-an-azure-event-hubs-dedicated-cluster-to-another-region"></a>Déplacer un cluster dédié Azure Event Hubs vers une autre région
Cet article vous explique comment exporter un modèle Azure Resource Manager pour un cluster dédié Event Hubs existant, puis utiliser ce modèle pour créer un cluster avec les mêmes paramètres de configuration dans une autre région. 

Si vous avez d’autres ressources, telles que des espaces de noms et des concentrateurs d’événements, dans le groupe de ressources Azure qui contient le cluster Event Hubs, vous pouvez exporter le modèle au niveau du groupe de ressources afin que toutes les ressources associées puissent être déplacées vers la nouvelle région en une seule étape. Les étapes décrites dans cet article expliquent comment exporter un **cluster Event Hubs** vers le modèle. Les étapes d’exportation d’un **groupe de ressources** vers le modèle sont similaires. 

## <a name="prerequisites"></a>Prérequis
Assurez-vous que le cluster dédié peut être créé dans la région cible. Le moyen le plus simple de s’en assurer est d’utiliser le Portail Azure pour essayer DE [créer un de cluster dédié Event Hubs](event-hubs-dedicated-cluster-create-portal.md). Vous voyez la liste des régions qui sont prises en charge à ce moment-là pour la création du cluster. 

## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager. Ce modèle contient des paramètres qui décrivent votre cluster dédié Event Hubs.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Toutes les ressources**, puis votre cluster dédié Event Hubs.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Télécharger** dans le panneau **Exporter le modèle**.

    :::image type="content" source="./media/move-cluster-across-regions/download-template.png" alt-text="Télécharger un modèle Resource Manager" lightbox="./media/move-cluster-across-regions/download-template.png":::
5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix.

   Ce fichier zip contient les fichiers .json qui incluent le modèle et des scripts pour le déployer.


## <a name="move"></a>Déplacer

Déployez le modèle pour créer un cluster dédié Event Hubs dans la région cible. 


1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Rechercher dans la Place de marché**, tapez **déploiement de modèle**, puis sélectionnez **Déploiement de modèle (déployer à l’aide de modèles personnalisés)** .
5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.
6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.
1. Mettez à jour la valeur de la propriété `location` pour qu’elle pointe vers la nouvelle région. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.
1. Sélectionnez **Enregistrer** pour enregistrer le modèle. 
1. Dans la page **Déploiement personnalisé**, procédez comme suit : 
    1. Sélectionnez un **abonnement** Azure. 
    2. Sélectionnez un **groupe de ressources** ou créez-en un. 
    3. Sélectionnez l’**emplacement** ou la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule. 
    4. Dans la section **SETTINGS** , procédez comme suit :    
        1. Entrez le nouveau **nom de cluster**. 

            :::image type="content" source="./media/move-cluster-across-regions/deploy-template.png" alt-text="Télécharger un modèle Resource Manager":::
    5. Au bas de la page, sélectionnez **Examiner et créer**. 
    1. Dans la page **Vérifier + créer**, passez en revue les paramètres, puis sélectionnez **Créer**.  

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Après le déploiement, si vous souhaitez recommencer, vous pouvez supprimer le **cluster dédié Event Hubs cible** et répéter les étapes décrites dans les sections [Préparer](#prepare) et [Déplacer](#move) de cet article.

Pour valider les modifications et terminer le déplacement d’un cluster Event Hubs, supprimez le **cluster Event Hubs** dans la région d’origine. 

Pour supprimer un cluster Event Hubs (source ou cible) à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, tapez **Clusters Event Hubs**, puis sélectionnez **Clusters Event hubs** dans les résultats de la recherche. Le cluster Event Hubs s’affiche dans une liste.
2. Sélectionnez le cluster à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 
3. Sur la page **supprimer un cluster**, confirmez la suppression en tapant le **nom du cluster**, puis sélectionnez **Supprimer**. 

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à déplacer un cluster dédié Event Hubs d’une région à une autre. 

Pour obtenir des instructions sur le déplacement d’un espace de noms d’une région vers une autre, consultez [Déplacer des espaces de noms Event Hubs entre régions](move-across-regions.md). 

Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :

- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)
