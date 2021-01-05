---
title: Déplacer des domaines Azure Event Grid vers une autre région
description: Cet article explique comment déplacer des domaines Azure Event Grid d’une région à une autre.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/20/2020
ms.openlocfilehash: fff8638a819511f84f15c52ad0695cdd5759f971
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89081621"
---
# <a name="move-azure-event-grid-domains-to-another-region"></a>Déplacer des domaines Azure Event Grid vers une autre région
Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité. 

Voici les étapes générales décrites dans cet article : 

- **Exporter la ressource de domaine** vers un modèle Resource Manager. 

    > [!IMPORTANT]
    > La ressource de domaine et les rubriques du domaine sont exportées vers le modèle. Les abonnements aux rubriques relatives au domaine ne sont pas exportés. 
- **Utiliser le modèle pour déployer le domaine** dans la région cible. 
- **Créer manuellement des abonnements pour les rubriques de domaine** dans la région cible. Lorsque vous avez exporté le domaine vers un modèle de la région actuelle, les abonnements pour les rubriques de domaine ne sont pas exportés. Créez-les une fois que le domaine et les rubriques de domaine ont été créés dans la région cible. 
- **Vérifier le déploiement**. Envoyez un événement à une rubrique de domaine dans le domaine et vérifiez que le gestionnaire d’événements associé à l’abonnement est appelé. 
- Pour **terminer le déplacement**, supprimez le domaine de la région source. 

## <a name="prerequisites"></a>Prérequis
- Vérifiez que le service Event Grid est disponible dans la région cible. Consultez la [disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager pour le domaine. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche, saisissez **Domaines Event Grid**, puis sélectionnez **Domaines Event Grid** dans la liste des résultats. 

    :::image type="content" source="./media/move-domains-across-regions/search-domains.png" alt-text="Rechercher et sélectionner des domaines Event Grid":::
3. Sélectionnez le **domaine** que vous souhaitez exporter vers un modèle Resource Manager. 

    :::image type="content" source="./media/move-domains-across-regions/select-domain.png" alt-text="Rechercher et sélectionner des domaines Event Grid":::   
4. Dans la page **Domaine Event Grid**, sélectionnez **Exporter le modèle** sous **Paramètres** dans le menu de gauche, puis sélectionnez **Télécharger** dans la barre d’outils. 

    :::image type="content" source="./media/move-domains-across-regions/export-template-download.png" alt-text="Rechercher et sélectionner des domaines Event Grid" lightbox="./media/move-domains-across-regions/export-template-download.png":::   

    > [!IMPORTANT]
    > Le domaine et les rubriques de domaine sont exportés. Les abonnements aux rubriques de domaine ne sont pas exportés. Par conséquent, vous devez créer des abonnements pour les rubriques de domaine après les avoir déplacées. 
5. Localisez le fichier **.zip** que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix. Ce fichier zip contient les fichiers JSON des modèles et paramètres. 
1. Ouvrez le fichier **template.json** dans l’éditeur de votre choix. 
8. Mettez à jour `location` pour la ressource **domaine** en spécifiant la région ou l’emplacement cible. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.

    ```json
    "type": "Microsoft.EventGrid/domains",
    "apiVersion": "2020-06-01",
    "name": "[parameters('domains_spegriddomain_name')]",
    "location": "westus",
    ```
1. **Enregistrez** le modèle. 

## <a name="recreate"></a>Recréer 
Déployez le modèle pour créer le domaine et les rubriques de domaine dans la région cible. 

1. Dans le portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.
3. Sélectionnez **Déploiement de modèle**.
4. Sélectionnez **Create** (Créer).
5. Sélectionnez **Générer votre propre modèle dans l’éditeur**.
6. Sélectionnez **Charger le fichier**, puis suivez les instructions pour charger le fichier **template.json** que vous avez téléchargé dans la section précédente.
7. Sélectionnez **Enregistrer** pour enregistrer le modèle. 
8. Dans la page **Déploiement personnalisé**, procédez comme suit :
    1. Sélectionnez un **abonnement** Azure. 
    1. Sélectionnez un **groupe de ressources** existant dans la région cible ou créez-en un. 
    1. Pour **Région**, sélectionnez la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule. 
    1. Pour **Nom de domaine**, entrez un nouveau nom pour le domaine. 
    1. Sélectionnez **Revoir + créer**. 
    
        :::image type="content" source="./media/move-domains-across-regions/deploy-template.png" alt-text="Rechercher et sélectionner des domaines Event Grid":::        
    1. Une fois la validation du modèle réussie, sélectionnez **Créer** en bas de la page pour déployer la ressource. 
    1. Une fois le déploiement réussi, sélectionnez **Accéder au groupe de ressources** pour accéder à la page du groupe de ressources. Confirmez qu’il existe un domaine dans le groupe de ressources. Sélectionnez le domaine. Confirmez qu’il existe des rubriques de domaine dans le domaine. 

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Pour terminer le déplacement, supprimez le domaine dans la région source.  

Si vous souhaitez recommencer, supprimez le domaine dans la région cible, puis répétez les étapes décrites dans les sections [Préparer](#prepare) et [Recréer](#recreate) de cet article.

Pour supprimer un domaine à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, saisissez **Domaines Event Grid**, puis sélectionnez **Domaines Event Grid** dans les résultats de la recherche. 
2. Sélectionnez le domaine à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 
3. Dans la page de confirmation, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.  

Pour supprimer le groupe de ressources qui contient le domaine à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, saisissez **Groupes de ressources**, puis sélectionnez **Groupes de ressources** dans les résultats de la recherche. 
2. Sélectionnez le groupe de ressources à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 
3. Dans la page de confirmation, entrez le nom du groupe de ressources, puis sélectionnez **Supprimer**.  

## <a name="next-steps"></a>Étapes suivantes
Vous avez appris à déplacer un domaine Event Grid d’une région à une autre. Consultez les articles suivants pour déplacer des rubriques système, des rubriques personnalisées et des espaces de noms de partenaires entre différentes régions.

- [Déplacer les rubriques système d’une région à une autre](move-system-topics-across-regions.md). 
- [Déplacer les rubriques personnalisées d’une région à une autre](move-custom-topics-across-regions.md). 
- [Déplacer des espaces de noms de partenaires d’une région à une autre](move-partner-namespaces-across-regions.md).

Pour plus d’informations sur le déplacement de ressources entre régions et la récupération d’urgence dans Azure, consultez l’article suivant : [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).