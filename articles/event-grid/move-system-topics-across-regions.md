---
title: Déplacer des rubriques système Azure Event Grid vers une autre région
description: Cet article explique comment déplacer des rubriques système Azure Event Grid d’une région à une autre.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/28/2020
ms.openlocfilehash: eb6029b206e7d47789371ee81e75c4e05c69ee65
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89081615"
---
# <a name="move-azure-event-grid-system-topics-to-another-region"></a>Déplacer des rubriques système Azure Event Grid vers une autre région
Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité. 

Voici les étapes générales décrites dans cet article : 

- **Exporter le groupe de ressources** qui contient le compte de stockage Azure et la rubrique système associée vers un modèle Resource Manager. Vous pouvez également exporter un modèle uniquement pour la rubrique système. Si vous choisissez cette approche, n’oubliez pas de déplacer la source de l’événement Azure (dans cet exemple, un compte de stockage Azure) vers l’autre région avant de déplacer la rubrique système. Ensuite, dans le modèle exporté de la rubrique système, mettez à jour l’ID externe du compte de stockage dans la région cible. 
- **Modifier le modèle** pour ajouter la propriété `endpointUrl` afin de pointer vers un webhook qui s’abonne à la rubrique système. Lorsque la rubrique système est exportée, son abonnement (dans le cas présent, il s’agit d’un webhook) est également exporté vers le modèle, mais la propriété `endpointUrl` n’est pas incluse. Par conséquent, vous devez le mettre à jour pour qu’il pointe vers le point de terminaison qui s’abonne à la rubrique. Mettez également à jour la valeur de la propriété `location` avec le nouvel emplacement ou la nouvelle région. Pour les autres types de gestionnaires d’événements, vous devez uniquement mettre à jour l’emplacement. 
- **Utiliser le modèle pour déployer des ressources** dans la région cible. Vous devez spécifier des noms pour le compte de stockage et la rubrique système à créer dans la région cible. 
- **Vérifier le déploiement**. Vérifiez que le webhook est appelé lorsque vous chargez un fichier dans le stockage de blobs dans la région cible. 
- Pour **terminer le déplacement**, supprimez les ressources (source de l’événement et rubrique système) de la région source. 

## <a name="prerequisites"></a>Prérequis
- Suivez le [Démarrage rapide : Acheminer des événements de stockage Blob vers un point de terminaison web avec le portail Azure](blob-event-quickstart-portal.md) dans la région source. Cette étape est **facultative**. Faites-le pour tester les étapes de cet article. Conservez le compte de stockage dans un groupe de ressources distinct d’App Service et du plan App Service. 
- Vérifiez que le service Event Grid est disponible dans la région cible. Consultez la [disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager pour le groupe de ressources qui contient la source de l’événement système (compte de stockage Azure) et la rubrique système qui lui est associée. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Groupes de ressources** dans le menu de gauche. Ensuite, sélectionnez le groupe de ressources qui contient la source de l’événement pour laquelle la rubrique système a été créée. Dans l’exemple suivant, il s’agit du compte **Stockage Azure**. Le groupe de ressources contient le compte de stockage et la rubrique système qui lui est associée. 

    :::image type="content" source="./media/move-system-topics-across-regions/resource-group-page.png" alt-text="Page du groupe de ressources":::        
3. Dans le menu de gauche, sélectionnez **Exporter le modèle** sous **Paramètres**, puis sélectionnez **Télécharger** dans la barre d’outils. 

    :::image type="content" source="./media/move-system-topics-across-regions/export-template-menu.png" alt-text="Page du groupe de ressources"
        }
        ```

        > [!NOTE]
        > Pour les autres types de gestionnaires d’événements, toutes les propriétés sont exportées vers le modèle. Vous devez uniquement mettre à jour la propriété `location` vers la région cible, comme indiqué à l’étape suivante. 
7. Mettez à jour `location` pour la ressource **compte de stockage** en spécifiant la région ou l’emplacement cible. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.

    ```json
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-06-01",
    "name": "[parameters('storageAccounts_spegridstorage080420_name')]",
    "location": "westus",
    ```
8. Répétez l’étape pour mettre à jour `location` pour la ressource **rubrique système** dans le modèle. 

    ```json
    "type": "Microsoft.EventGrid/systemTopics",
    "apiVersion": "2020-04-01-preview",
    "name": "[parameters('systemTopics_spegridsystopic080420_name')]",
    "location": "westus",
    ```
1. **Enregistrez** le modèle. 

## <a name="recreate"></a>Recréer 
Déployez le modèle pour créer un compte de stockage et une rubrique système pour le compte de stockage dans la région cible. 

1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.
3. Sélectionnez **Déploiement de modèle**.
4. Sélectionnez **Create** (Créer).
5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.
6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.
7. Sélectionnez **Enregistrer** pour enregistrer le modèle. 
8. Dans la page **Déploiement personnalisé**, procédez comme suit. 
    1. Sélectionnez un **abonnement** Azure. 
    1. Sélectionnez un **groupe de ressources** existant dans la région cible ou créez-en un. 
    1. Pour **Région**, sélectionnez la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule.
    1. Pour le **nom de la rubrique système**, entrez un nom pour la rubrique système qui sera associée au compte de stockage.  
    1. Pour le **nom du compte de stockage**, entrez un nom pour le compte de stockage qui sera créé dans la région cible. 

        :::image type="content" source="./media/move-system-topics-across-regions/deploy-template.png" alt-text="Page du groupe de ressources":::
    5. Au bas de la page, sélectionnez **Examiner et créer**. 
    1. Dans la page **Vérifier + créer**, passez en revue les paramètres, puis sélectionnez **Créer**. 

## <a name="verify"></a>Vérification
1. Une fois le déploiement réussi, sélectionnez **Accéder au groupe de ressources**. 
1. Sur la page **Groupe de ressources**, vérifiez que la source de l’événement (dans cet exemple, compte de stockage Azure) et la rubrique système sont créées. 
1. Chargez un fichier dans un conteneur dans le stockage Blob Azure et vérifiez que le webhook a reçu l’événement. Pour plus d’informations, consultez [Envoyer un événement à votre point de terminaison](blob-event-quickstart-portal.md#send-an-event-to-your-endpoint).

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Pour terminer le déplacement, supprimez le groupe de ressources qui contient le compte de stockage et sa rubrique système associée dans la région source.  

Si vous souhaitez recommencer, supprimez le groupe de ressources dans la région cible, puis répétez les étapes décrites dans les sections [Préparer](#prepare) et [Recréer](#recreate) de cet article.

Pour supprimer un groupe de ressources (source ou cible) à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, saisissez **Groupes de ressources**, puis sélectionnez **Groupes de ressources** dans les résultats de la recherche. 
2. Sélectionnez le groupe de ressources à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 

    :::image type="content" source="./media/move-system-topics-across-regions/delete-resource-group-button.png" alt-text="Page du groupe de ressources":::
3. Dans la page de confirmation, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.  

## <a name="next-steps"></a>Étapes suivantes
Vous avez appris à déplacer une source d’événement Azure et sa rubrique système associée d’une région à une autre. Consultez les articles suivants pour déplacer des rubriques personnalisées, des domaines et des espaces de noms de partenaires entre différentes régions.

- [Déplacer les rubriques personnalisées d’une région à une autre](move-custom-topics-across-regions.md). 
- [Déplacer les domaines d’une région à une autre](move-domains-across-regions.md). 
- [Déplacer des espaces de noms de partenaires d’une région à une autre](move-partner-namespaces-across-regions.md). 

Pour plus d’informations sur le déplacement de ressources entre régions et la récupération d’urgence dans Azure, consultez l’article suivant : [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).
