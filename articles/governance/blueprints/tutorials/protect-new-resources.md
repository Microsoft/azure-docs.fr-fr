---
title: 'Tutoriel : Protéger des nouvelles ressources avec des verrous'
description: Dans ce tutoriel, vous utilisez les options de verrous de ressources Azure Blueprints « Lecture seule » et « Ne pas supprimer » pour protéger les ressources nouvellement déployées.
ms.date: 08/27/2020
ms.topic: tutorial
ms.openlocfilehash: 3ed75a1dee925f2a55ac46705a171bec5fc1d30e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89048568"
---
# <a name="tutorial-protect-new-resources-with-azure-blueprints-resource-locks"></a>Tutoriel : Protéger les nouvelles ressources avec des verrous de ressource Azure Blueprints

Les [verrous de ressources](../concepts/resource-locking.md) Azure Blueprints permettent de protéger les ressources nouvellement déployées contre toute manipulation, même par un compte disposant du rôle _Propriétaire_. Vous pouvez ajouter cette protection dans les définitions de blueprint des ressources qui ont été créées par un artefact de modèle Azure Resource Manager (modèle ARM).

Dans ce tutoriel, vous allez effectuer les étapes suivantes :

> [!div class="checklist"]
> - Créer une définition de blueprint
> - Affecter l’état **Publié** à votre définition de blueprint
> - Affecter votre définition de blueprint à un abonnement existant
> - Inspecter le nouveau groupe de ressources
> - Annuler l’affectation du blueprint afin de retirer les verrous

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

## <a name="create-a-blueprint-definition"></a>Créer une définition de blueprint

Tout d’abord, créez la définition de blueprint.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage** située à gauche, sélectionnez **Créer** sous **Créer un blueprint**.

1. Accédez à l’exemple de blueprint **Blueprint vide** situé en haut de la page. Sélectionnez **Commencer par un blueprint vide**.

1. Entrez les informations suivantes sous l’onglet **Fonctions de base** :

   - **Nom du blueprint** : Entrez le nom de votre copie de l’exemple de blueprint. Pour ce didacticiel, nous utiliserons le nom **locked-storageaccount**.
   - **Description du blueprint** : Ajoutez une description pour la définition de blueprint. Utilisez **For testing blueprint resource locking on deployed resources** (« Pour tester le verrouillage de ressources de blueprint sur les ressources déployées »).
   - **Emplacement de la définition** : Sélectionnez le bouton représentant des points de suspension (...), puis sélectionnez le groupe d’abonnement ou l’abonnement dans lequel enregistrer votre définition de blueprint.

1. Sélectionnez l’onglet **Artefacts** situé en haut de la page, ou sélectionnez **Suivant : Artefacts** dans le bas de la page.

1. Ajoutez un groupe de ressources au niveau de l’abonnement :
   1. Sélectionnez la ligne **Ajouter un artefact** située sous **Abonnement**.
   1. Sélectionnez **Groupe de ressources** sous **Type d’artefact**.
   1. Définissez le **Nom complet de l'artefact** sur **RGtoLock**.
   1. Laissez les champs **Nom du groupe de ressources** et **Emplacement** vides, mais vérifiez que la case de chaque propriété est cochée pour en faire des **paramètres dynamiques**.
   1. Sélectionnez **Ajouter** pour ajouter l’artefact au blueprint.

1. Ajoutez un modèle sous le groupe de ressources :
   1. Sélectionnez la ligne **Ajouter un artefact** sous l’entrée **RGtoLock**.
   1. Sélectionnez **Modèle Azure Resource Manager** sous **Type d’artefact**, définissez **Nom complet de l’artefact** sur la valeur **StorageAccount**, et laissez le champ **Description** vide.
   1. Sous l’onglet **Modèle**, collez le modèle ARM suivant dans la zone de l’éditeur. Après avoir collé le modèle, sélectionnez **Ajouter** pour ajouter cet artefact au blueprint.

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
           "storageAccountType": {
               "type": "string",
               "defaultValue": "Standard_LRS",
               "allowedValues": [
                   "Standard_LRS",
                   "Standard_GRS",
                   "Standard_ZRS",
                   "Premium_LRS"
               ],
               "metadata": {
                   "description": "Storage Account type"
               }
           }
       },
       "variables": {
           "storageAccountName": "[concat('store', uniquestring(resourceGroup().id))]"
       },
       "resources": [{
           "type": "Microsoft.Storage/storageAccounts",
           "name": "[variables('storageAccountName')]",
           "location": "[resourceGroup().location]",
           "apiVersion": "2018-07-01",
           "sku": {
               "name": "[parameters('storageAccountType')]"
           },
           "kind": "StorageV2",
           "properties": {}
       }],
       "outputs": {
           "storageAccountName": {
               "type": "string",
               "value": "[variables('storageAccountName')]"
           }
       }
   }
   ```

1. Cliquez sur **Enregistrer le brouillon** en bas de la page.

Cette étape crée la définition de blueprint dans le groupe d’administration ou l’abonnement sélectionné.

Dès que la notification **Définition de blueprint enregistrée** s’affiche dans le portail, passez à l’étape suivante.

## <a name="publish-the-blueprint-definition"></a>Publier la définition de blueprint

Votre définition de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être publiée avant de pouvoir être attribuée et déployée.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour trouver la définition de blueprint **locked-storageaccount**, puis sélectionnez celle-ci.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans le nouveau volet situé sur la droite, entrez **1.0** pour la **version**. Cette propriété est utile si vous devez faire une modification ultérieurement. Sous **Change notes**, entrez des remarques sur les modifications, par exemple, **Première version publiée pour le verrouillage des ressources déployées du blueprint**. Sélectionnez ensuite **Publier** en bas de la page.

Cette étape permet d’affecter le plan à un abonnement. Vous pouvez modifier la définition de blueprint, même après sa publication. Si vous la modifiez, vous devez la publier avec une nouvelle valeur de version, de manière à pouvoir voir les différences qui existent entre les versions d’une même définition de blueprint.

Dès que la notification **Définition de blueprint publiée** s’affiche dans le portail, passez à l’étape suivante.

## <a name="assign-the-blueprint-definition"></a>Affecter la définition de blueprint

Une fois que la définition de blueprint est publiée, vous pouvez l’affecter à un abonnement dans le groupe d’administration où vous avez enregistré la définition. Dans cette étape, vous allez fournir des paramètres pour que chaque déploiement de la définition de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour trouver la définition de blueprint **locked-storageaccount**, puis sélectionnez celle-ci.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - **Concepts de base**

     - **Abonnements** : Sélectionnez un ou plusieurs des abonnements du groupe d’administration où vous avez enregistré votre définition de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation** : Le nom est prérempli automatiquement en fonction du nom de la définition du blueprint. Cette affectation représente le verrouillage du nouveau groupe de ressources. Renommez celle-ci en **assignment-locked-storageaccount-TestingBPLocks**.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
       Pour ce didacticiel, sélectionnez **USA Est 2**.
     - **Version de définition du blueprint** : Sélectionnez la version publiée **1.0** de la définition de blueprint.

   - **Verrouiller l'affectation**

     Sélectionnez le mode de verrouillage **Lecture seule**. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

   - **Identité gérée**

     Utilisez l’option par défaut : **Affecté(e) par le système**. Pour plus d’informations, voir [Identités managées](../../../active-directory/managed-identities-azure-resources/overview.md).

   - **Paramètres d'artefact**

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel ils sont définis. Ces paramètres sont des [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour chaque artefact, définissez la valeur du paramètre en vous référant à la colonne **Valeur**.

     |Nom de l’artefact|Type d’artefact|Nom du paramètre|Valeur|Description|
     |-|-|-|-|-|
     |Groupe de ressources RGtoLock|Resource group|Nom|TestingBPLocks|Définit le nom du premier groupe de ressources auquel appliquer les verrous de blueprint.|
     |Groupe de ressources RGtoLock|Resource group|Emplacement|USA Ouest 2|Définit l’emplacement du nouveau groupe de ressources auquel appliquer les verrous de blueprint.|
     |StorageAccount|Modèle Resource Manager|storageAccountType (StorageAccount)|Standard_GRS|Référence SKU du stockage. La valeur par défaut est _Standard_LRS_.|

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page.

Cette étape déploie les ressources définies et configure l’option **Verrouiller l’affectation** sélectionnée. L’application des verrous de blueprint peut prendre jusqu’à 30 minutes.

Dès que la notification **Définition de blueprint affectée** s’affiche dans le portail, passez à l’étape suivante.

## <a name="inspect-resources-deployed-by-the-assignment"></a>Examiner les ressources déployées par l’affectation

L’affectation crée le groupe de ressources _TestingBPLocks_ et le compte de stockage déployé par l’artefact du modèle ARM. Le nouveau groupe de ressources et l’état de verrouillage sélectionné sont affichés dans la page de détails de l’affectation.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Blueprints affectés** à gauche. Utilisez les filtres pour trouver l’affectation de blueprint **assignment-locked-storageaccount-TestingBPLocks**, puis sélectionnez celle-ci.

   Dans cette page, nous voyons que l’affectation a réussi et que les ressources ont été déployées avec le nouvel état de verrouillage de blueprint. Si l’affectation est mise à jour, le menu déroulant **Opération d’affectation** affiche des détails sur le déploiement de chaque version de définition. Vous pouvez sélectionner le groupe de ressources pour ouvrir la page de propriétés.

1. Sélectionnez le groupe de ressources **TestingBPLocks**.

1. Sélectionnez le volet **Contrôle d’accès (IAM)** situé sur la gauche. Ensuite, sélectionnez l’onglet **Attributions de rôles**.

   Nous pouvons constater que l’affectation de blueprint _assignment-locked-storageaccount-TestingBPLocks_ dispose du rôle _Propriétaire_. Elle dispose de ce rôle, car celui-ci a été utilisé pour déployer et verrouiller le groupe de ressources.

1. Sélectionnez l’onglet **Affectations de refus**.

   L’affectation du blueprint a créé une instance [Refuser l’affectation](../../../role-based-access-control/deny-assignments.md) sur le groupe de ressources déployé pour appliquer le mode **Lecture seule** de verrouillage du blueprint. Cette affectation de refus empêche un utilisateur disposant de droits appropriés dans l’onglet **Attributions de rôle** d’effectuer des actions particulières. L’affectation de refus concerne _Tous les principaux_.

   Pour plus d’informations sur l’exclusion d’un principal à partir d’une affectation de refus, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md#exclude-a-principal-from-a-deny-assignment).

1. Sélectionnez l’affectation Refuser, puis sélectionnez la page **Autorisations refusées** située sur la gauche.

   L’affectation Refuser empêche toutes les opérations ayant une configuration **\*** et **Action**, mais autorise l’accès en lecture, en excluant **\*/read** via **NotActions**.

1. Dans la barre de navigation du portail Azure, sélectionnez **TestingBPLocks - Contrôle d’accès (IAM)** . Sélectionnez ensuite la page **Vue d’ensemble** à gauche, puis le bouton **Supprimer le groupe de ressources**. Entrez le nom **TestingBPLocks** pour confirmer la suppression, puis sélectionnez **Supprimer** au bas du volet.

   Le message de notification du portail **Échec de la suppression du groupe de ressources TestingBPLocks**  s’affiche. L’erreur indique que, même si votre compte est autorisé à supprimer le groupe de ressources, l’accès est refusé par l’affectation du blueprint. N’oubliez pas que nous avons sélectionné le mode **Lecture seule** de verrouillage du blueprint pendant l’affectation du blueprint. Le verrou de blueprint empêche un compte disposant d’autorisations, y compris _Propriétaire_, de supprimer la ressource. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

Ces étapes montrent que nos ressources déployées sont à présent protégées par les verrous du blueprint qui empêchent toute suppression non souhaitée, même à partir d’un compte autorisé à supprimer les ressources.

## <a name="unassign-the-blueprint"></a>Annuler l’affectation du blueprint

La dernière étape consiste à supprimer l’affectation de la définition de blueprint. Le fait de supprimer l’affectation ne supprime pas les artefacts associés.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Blueprints affectés** à gauche. Utilisez les filtres pour trouver l’affectation de blueprint **assignment-locked-storageaccount-TestingBPLocks**, puis sélectionnez celle-ci.

1. Sélectionnez **Annuler l'affectation du blueprint** en haut de la page. Lisez l’avertissement de la boîte de dialogue de confirmation, puis sélectionnez **OK**.

   La suppression du blueprint entraîne la suppression de ses verrous. Les ressources peuvent à nouveau être supprimées par un compte disposant des autorisations nécessaires.

1. Sélectionnez **Groupes de ressources** dans le menu Azure, puis sélectionnez **TestingBPLocks**.

1. Sélectionnez la page **Contrôle d’accès (IAM)** située sur la gauche, puis sélectionnez l’onglet **Attributions de rôles**.

Du point de vue de la sécurité du groupe de ressources, l’affectation du blueprint n’a plus d’accès _Propriétaire_.

Dès que la notification **Affectation de blueprint supprimée** s’affiche dans le portail, passez à l’étape suivante.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé le tutoriel, supprimez les ressources suivantes :

- Groupe de ressources _TestingBPLocks_
- Définition de blueprint _locked-storageaccount_

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à protéger de nouvelles ressources déployées avec Azure Blueprints. Pour plus d’informations sur Azure Blueprints, consultez l’article concernant le cycle de vie des blueprints.

> [!div class="nextstepaction"]
> [En savoir plus sur le cycle de vie des blueprints](../concepts/lifecycle.md)