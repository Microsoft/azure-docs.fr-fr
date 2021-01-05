---
title: Déplacer un espace de noms Service Bus vers une autre région | Microsoft Docs
description: Cet article explique comment déplacer un espace de noms Service Bus de la région actuelle vers une autre région.
ms.topic: how-to
ms.date: 06/23/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 573ea96437cf6cb76854ffa417fd3ad3fb86138b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88860360"
---
# <a name="move-an-azure-service-bus-namespace-to-another-region"></a>Déplacer un espace de noms Service Bus vers une autre région
Il existe différents scénarios dans lesquels vous pouvez être amené à déplacer votre espace de noms Service Bus existant d’une région à une autre. Par exemple, vous pouvez créer un espace de noms avec la même configuration à des fins de test. Vous pouvez également créer un espace de noms secondaire dans une autre région dans le cadre de la [planification de la reprise d’activité après sinistre](service-bus-geo-dr.md).

Les étapes principales sont les suivantes :

1. Exportez l’espace de noms Service Bus de la région actuelle vers un modèle Azure Resource Manager. 
1. Mettez à jour l’emplacement des ressources dans le modèle. De plus, supprimez le filtre d’abonnement par défaut du modèle, car vous ne pouvez pas créer de règle par défaut, celle-ci étant automatiquement créée pour vous. 
1. Utilisez le modèle pour déployer l’espace de noms Service Bus dans la région cible. 
1. Vérifiez le déploiement pour vous assurer que l’espace de noms, les files d’attente, les rubriques et les abonnements aux rubriques sont tous créés dans la région cible. 
1. Enfin, supprimez l’espace de noms de la région source une fois tous les messages traités. 

## <a name="prerequisites"></a>Prérequis
Vérifiez qu’Azure Service Bus et les fonctionnalités utilisées par votre compte sont pris en charge dans la région cible.
 
## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager. Ce modèle contient des paramètres qui décrivent votre espace de noms Service Bus.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Toutes les ressources**, puis sélectionnez votre espace de noms Service Bus.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Télécharger** dans le panneau **Exporter le modèle**.

    :::image type="content" source="./media/move-across-regions/download-template.png" alt-text="Télécharger un modèle Resource Manager":::
5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix. Ce fichier zip contient les fichiers JSON des modèles et paramètres. 
1. Ouvrez le fichier template.json dans le dossier extrait. 
1. Recherchez `location`et remplacez la valeur de la propriété par le nouveau nom de la région ou de l’emplacement. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.
1. Supprimez les définitions des ressources de type : `Microsoft.ServiceBus/namespaces/topics/subscriptions/rules`. N’oubliez pas de supprimer la virgule (`,`) qui précède cette section pour conserver la validité du JSON.  

    > [!NOTE]
    > Vous ne pouvez pas créer de règle par défaut pour un abonnement à l’aide d’un modèle Resource Manager. La règle par défaut est automatiquement créée lors de la création de l’abonnement dans la région cible. 

## <a name="move"></a>Déplacer
Déployez le modèle pour créer un espace de noms Service Bus dans la région cible. 

1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Rechercher sur la Place de marché**, entrez **template deployment** pour le texte recherché, sélectionnez **Déploiement de modèle (déployer à l’aide de modèles personnalisés)** , puis appuyez sur **Entrée**.

    :::image type="content" source="./media/move-across-regions/new-template-deployment.png" alt-text="Télécharger un modèle Resource Manager":::    
1. Sur la page **Déploiement de modèle**, sélectionnez **Créer**.

    :::image type="content" source="./media/move-across-regions/template-deployment-create-button.png" alt-text="Télécharger un modèle Resource Manager":::        
1. Sur la page **Déploiement personnalisé**, sélectionnez **Créer votre propre modèle dans l’éditeur**.

    :::image type="content" source="./media/move-across-regions/build-template-link.png" alt-text="Télécharger un modèle Resource Manager":::            
1. Sur la page **Modifier un modèle**, sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.

    :::image type="content" source="./media/move-across-regions/select-template.png" alt-text="Télécharger un modèle Resource Manager":::                
1. Sélectionnez **Enregistrer** pour enregistrer le modèle. 

    :::image type="content" source="./media/move-across-regions/save-template.png" alt-text="Télécharger un modèle Resource Manager":::                    
1. Dans la page **Déploiement personnalisé**, procédez comme suit : 
    1. Sélectionnez un **abonnement** Azure. 
    2. Sélectionnez un **groupe de ressources** ou créez-en un. 
    3. Sélectionnez l’**emplacement** ou la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule. 
    4. Entrez un nouveau **nom pour l’espace de noms**.
    1. Sélectionnez **Revoir + créer**. 

        :::image type="content" source="./media/move-across-regions/deploy-template.png" alt-text="Télécharger un modèle Resource Manager":::
    1. En bas de la page **Vérifier + créer**, sélectionnez **Créer**. 
    
## <a name="verify"></a>Vérification
1. Une fois le déploiement terminé, sélectionnez **Accéder au groupe de ressources**.

    :::image type="content" source="./media/move-across-regions/resource-group-navigation-link.png" alt-text="Télécharger un modèle Resource Manager":::    
1. Sur la page **Groupe de ressources**, sélectionnez l’espace de noms Service Bus. 

    :::image type="content" source="./media/move-across-regions/select-namespace.png" alt-text="Télécharger un modèle Resource Manager":::    
1. Sur la page **Espace de noms Service Bus**, vérifiez que les files d’attente, les rubriques et les abonnements sont visibles dans la région source. 
    1. Les **files d’attente** s’affichent dans l’espace de noms en bas du volet droit.         
    
        :::image type="content" source="./media/move-across-regions/queue-namespace.png" alt-text="Télécharger un modèle Resource Manager":::
    2. Basculer vers l’onglet **Rubriques** pour afficher les rubriques de l’espace de noms
    
        :::image type="content" source="./media/move-across-regions/topics-namespace.png" alt-text="Télécharger un modèle Resource Manager":::
    3. Sélectionnez la rubrique pour vérifier que les abonnements sont créés. 

        :::image type="content" source="./media/move-across-regions/topic-subscriptions.png" alt-text="Télécharger un modèle Resource Manager":::      
    
    

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Après le déploiement, si vous souhaitez recommencer, vous pouvez supprimer l’**espace de noms Service Bus cible** et répéter les étapes décrites dans les sections [Préparer](#prepare) et [Déplacer](#move) de cet article.

Pour valider les modifications et terminer le déplacement d’un espace de noms Service Bus, supprimez l’**espace de noms Service Bus source**. Veillez à traiter tous les messages avant de supprimer l’espace de noms. 

Pour supprimer un espace de noms Service Bus (source ou cible) à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, entrez **Service Bus**, puis sélectionnez **Service Bus** dans les résultats de la recherche. Les espaces de noms Service Bus apparaissent dans une liste.
2. Sélectionnez l’espace de noms cible à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 

    ![Supprimer un espace de noms - bouton](./media/move-across-regions/delete-namespace-button.png)
3. Sur la page **Supprimer des ressources**, vérifiez les ressources sélectionnées, puis confirmez la suppression en tapant **Oui**, puis sélectionnez **Supprimer**. 

    Une autre option consiste à supprimer le groupe de ressources contenant l’espace de noms Service Bus. Sur la page **Groupe de ressources**, sélectionnez **Supprimer un groupe de ressources** dans la barre d’outils, puis confirmez la suppression. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un espace de noms Service Bus d’une région vers une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :

- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
