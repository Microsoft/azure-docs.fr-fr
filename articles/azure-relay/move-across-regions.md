---
title: Déplacer un espace de noms Azure Relay vers une autre région
description: Cet article explique comment déplacer un espace de noms Azure Relay de la région actuelle vers une autre région.
ms.topic: how-to
ms.date: 09/03/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 60a182764639341fcda159356dd9fe6c65cfabd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89463265"
---
# <a name="move-an-azure-relay-namespace-to-another-region"></a>Déplacer un espace de noms Azure Relay vers une autre région
Cet article explique comment déplacer un espace de noms Azure Relay d’une région vers une autre. Les étapes principales sont les suivantes :

1. **Exportez** l’espace de noms Relay vers un modèle Resource Manager.
1. **Mettez à jour l’emplacement (région)** des ressources dans le modèle. Supprimez également les relais WCF **dynamiques** du modèle. 

    Les relais WCF ont deux modes. Dans le premier mode, le relais WCF est créé de manière explicite à l’aide du portail Azure ou d’un Azure Resource Manager. Dans la page **Relais WCF** du portail Azure, la propriété **isDynamic** est définie sur **false** pour un relais dans ce mode. 

    Dans le deuxième mode, le relais WCF est généré automatiquement quand un écouteur (serveur) se connecte pour une adresse de point de terminaison donnée. Tant que l’écouteur est connecté au relais, vous voyez celui-ci dans la liste des relais WCF dans le portail Azure. Pour un relais dans ce mode, la propriété **isDynamic** a la valeur **true**, car elle est générée dynamiquement. Le relais WCF dynamique disparaît lorsque l’écouteur se déconnecte. 
1. **Déployez** des ressources à l’aide du modèle dans la région cible.

## <a name="prerequisites"></a>Prérequis
Vérifiez que le service Azure Relay est disponible dans la région cible. Consultez la [disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=service-bus&regions=all). 
 
## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager. Ce modèle contient des paramètres décrivant votre espace de noms Azure Relay.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Toutes les ressources**, puis votre espace de noms Azure Relay.
3. Sous **Paramètres** dans le menu de gauche, sélectionnez **Exporter le modèle**.
4. Sans la page **Exporter le modèle**, choisissez **Télécharger**.

    :::image type="content" source="./media/move-across-regions/download-template.png" alt-text="Télécharger un modèle Resource Manager":::
5. Localisez le fichier .zip que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix. Ce fichier zip contient les fichiers JSON des modèles et paramètres. 
1. Ouvrez le fichier **template.json** à partir du dossier extrait dans l’éditeur de votre choix.
1. Recherchez `location`, puis remplacez la valeur de la propriété par le nouveau nom de la région. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.
1. Supprimez les définitions des ressources de **relais WCF dynamique** (type : `Microsoft.Relay/namespaces/WcfRelays`). Les relais WCF dynamiques sont ceux dont la propriété **isDynamic** a la valeur **true** sur la page **Relais**. Dans l’exemple suivant, **echoservice** est un relais WCF dynamique et sa définition doit être supprimée du modèle. 

    :::image type="content" source="./media/move-across-regions/dynamic-relays.png" alt-text="Télécharger un modèle Resource Manager":::

## <a name="move"></a>Déplacer
Déployez le modèle pour créer un espace de noms Relais dans la région cible. 

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
1. Dans la page **Groupe de ressources**, sélectionnez l’espace de noms Azure Relay. 

    :::image type="content" source="./media/move-across-regions/select-namespace.png" alt-text="Télécharger un modèle Resource Manager":::    
1. Sur la page **Espace de noms Azure Relay**, dans le menu de gauche, sélectionnez **Connexions hybrides** ou **Relais WCF** pour vérifier que les connexions hybrides et les relais WCF sont créés. Si vous avez oublié de supprimer des définitions pour des relais WCF dynamiques avant d’importer le modèle, supprimez-les de la page **Relais WCF**. Les relais WCF dynamiques sont créés automatiquement lorsque les clients se connectent à l’espace de noms Relais. 

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Après le déploiement, si vous souhaitez recommencer, vous pouvez supprimer l’**espace de noms Azure Relay cible** et répéter les étapes décrites dans les sections [Préparer](#prepare) et [Déplacer](#move) de cet article.

Pour valider les modifications et terminer le déplacement d’un espace de noms, supprimez l’**espace de noms Azure Relay** dans la région source. 

Pour supprimer un espace de noms Azure Relay (source ou cible) à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, tapez **Relais**, puis sélectionnez **Relais** sous **Services** dans les résultats de la recherche. Tous les espaces de noms Azure Relay s’affichent dans une liste.
2. Sélectionnez l’espace de noms cible à supprimer pour ouvrir la page **Relais**. 
1. Sur la page **Relais**, dans la barre d’outils, sélectionnez **Supprimer**. 

    ![Supprimer un espace de noms - bouton](./media/move-across-regions/delete-namespace-button.png)
3. Dans la page **Supprimer l’espace de noms**, tapez le nom de l’espace de noms Azure Relay pour confirmer la suppression, puis sélectionnez **Supprimer**. 

## <a name="next-steps"></a>Étapes suivantes
Dans le cadre de ce tutoriel, vous avez déplacé un espace de noms Azure Relay d’une région vers une autre région. Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :

- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
