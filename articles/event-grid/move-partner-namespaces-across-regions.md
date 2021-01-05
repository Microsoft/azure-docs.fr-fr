---
title: Déplacer des espaces de noms de partenaires Azure Event Grid vers une autre région
description: Cet article explique comment déplacer des espaces de noms de partenaires Azure Event Grid d’une région à une autre.
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/20/2020
ms.openlocfilehash: 6783db6b9bb1c7d48b308234a179925d6f30e281
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89081620"
---
# <a name="move-azure-event-grid-partner-namespaces-to-another-region"></a>Déplacer des espaces de noms de partenaires Azure Event Grid vers une autre région
Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité. 

Voici les étapes générales décrites dans cet article : 

- **Exporter l’espace de noms de partenaire** vers un modèle Resource Manager. Supprimez les définitions pour les ressources de canal d’événements dans le modèle. Un canal d’événements peut avoir une référence à l’ID Azure Resource Manager de la rubrique du partenaire, qui est détenu par un client. Par conséquent, elles ne peuvent pas être créées à l’aide du modèle dans la région cible.  
- **Utiliser le modèle pour déployer l’espace de noms de partenaire** dans la région cible. Créez ensuite des canaux d’événements dans le nouvel espace de noms de partenaire dans la région cible. 
- Pour **terminer le déplacement**, supprimez l’espace de noms de partenaire de la région source. 

    > [!NOTE]
    > - L’exportation de **rubriques de partenaire** vers un modèle Resource Manager n’est pas prise en charge, car les clients ne peuvent pas créer directement de rubrique de partenaire. 
    > - Les **inscriptions de partenaires** sont des ressources mondiales (non liées à une région spécifique). Par conséquent, les déplacer d’une région à une autre région n’est pas applicable. 

## <a name="prerequisites"></a>Prérequis
- Vérifiez que le service Event Grid est disponible dans la région cible. Consultez la [disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=event-grid&regions=all).

## <a name="prepare"></a>Préparation
Pour commencer, exportez un modèle Resource Manager pour l’espace de noms de partenaire. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche en haut, saisissez **Espaces de noms de partenaires Event Grid**, puis sélectionnez **Espaces de noms de partenaires Event Grid** dans la liste des résultats. 
3. Sélectionnez l’**espace de noms de partenaire** que vous souhaitez exporter vers un modèle Resource Manager. 
4. Dans la page **Espace de noms de partenaire Event Grid**, sélectionnez **Exporter le modèle** sous **Paramètres** dans le menu de gauche, puis sélectionnez **Télécharger** dans la barre d’outils. 

    :::image type="content" source="./media/move-partner-namespaces-across-regions/download-template.png" alt-text="Exporter le modèle -> Télécharger" lightbox="./media/move-partner-namespaces-across-regions/download-template.png":::   
5. Localisez le fichier **.zip** que vous avez téléchargé à partir du portail, puis décompressez-le dans le dossier de votre choix. Ce fichier zip contient les fichiers JSON des modèles et paramètres. 
1. Ouvrez le fichier **template.json** dans l’éditeur de votre choix. 
8. Mettez à jour `location` pour la ressource **rubrique** en spécifiant la région ou l’emplacement cible. Pour obtenir les codes d’emplacement, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région correspond au nom de la région sans espace, par exemple `West US` correspond à `westus`.

    ```json
    {
        "type": "Microsoft.EventGrid/partnerNamespaces",
        "apiVersion": "2020-04-01-preview",
        "name": "[parameters('partnerNamespace_name')]",
        "location": "westus",
        "properties": {
            "partnerRegistrationFullyQualifiedId": "[parameters('partnerRegistrations_ContosoCorpAccount1_externalid')]"
        }
    }
    ``` 
1. **Enregistrez** le modèle. 

## <a name="recreate"></a>Recréer 
Déployez le modèle pour créer un espace de noms de partenaire dans la région cible. 

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
    1. Pour **Emplacement**, sélectionnez la région cible. Si vous avez sélectionné un groupe de ressources existant, ce paramètre est en lecture seule. 
    1. Pour le **nom de l’espace de noms de partenaire**, entrez un nom pour le nouvel espace de noms de partenaire. 
    1. Pour l’ID externe de l’inscription de partenaire, entrez l’ID de ressource de l’inscription de partenaire au format suivant : `/subscriptions/<Azure subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.EventGrid/partnerRegistrations/<Partner registration name>`.
    1. Cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.     
    1. Sélectionnez **Vérifier + créer** pour commencer le processus de déploiement. 
    1. Dans la page **Vérifier + créer**, passez en revue les paramètres, puis sélectionnez **Créer**. 

## <a name="discard-or-clean-up"></a>Ignorer ou nettoyer
Pour terminer le déplacement, supprimez l’espace de noms de partenaire dans la région source.  

Si vous souhaitez recommencer, supprimez l’espace de noms de partenaire dans la région cible, puis répétez les étapes décrites dans les sections [Préparer](#prepare) et [Recréer](#recreate) de cet article.

Pour supprimer un espace de noms de partenaire à l’aide du portail Azure :

1. Dans la fenêtre de recherche en haut du portail Azure, saisissez **Espaces de noms de partenaires Event Grid**, puis sélectionnez **Espaces de noms de partenaires Event Grid** parmi les résultats de la recherche. 
2. Sélectionnez l’espace de noms de partenaire à supprimer, puis choisissez **Supprimer** dans la barre d’outils. 
3. **Confirmez** la suppression pour supprimer l’espace de noms de partenaire. 

## <a name="next-steps"></a>Étapes suivantes
Vous avez appris à déplacer un espace de noms de partenaire Event Grid d’une région à une autre. Consultez les articles suivants pour déplacer des rubriques système, des rubriques personnalisées et des domaines entre différentes régions.

- [Déplacer les rubriques système d’une région à une autre](move-system-topics-across-regions.md). 
- [Déplacer les rubriques personnalisées d’une région à une autre](move-custom-topics-across-regions.md). 
- [Déplacer les domaines d’une région à une autre](move-domains-across-regions.md).

Pour plus d’informations sur le déplacement de ressources entre régions et la récupération d’urgence dans Azure, consultez l’article suivant : [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).