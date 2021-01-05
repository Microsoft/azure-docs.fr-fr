---
title: Partager des données en dehors de votre organisation (modèle ARM) - Démarrage rapide Azure Data Share
description: Dans ce guide de démarrage rapide, vous allez découvrir comment partager des données avec des clients et des partenaires à l’aide d’Azure Data Share et d’un modèle Azure Resource Manager (modèle ARM).
author: mumian
ms.author: jgao
ms.service: data-share
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 08/19/2020
ms.openlocfilehash: 5abe92120c8b822ac86ced90658869a0858d4ff4
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92487685"
---
# <a name="quickstart-share-data-using-azure-data-share-and-arm-template"></a>Démarrage rapide : Partager des données à l’aide d’Azure Data Share et d’un modèle ARM

Découvrez comment configurer un nouveau partage de données Azure à partir d’un compte Stockage Azure à l’aide d’un modèle Azure Resource Manager (modèle ARM), et commencez à partager vos données avec des clients et partenaires en dehors de votre organisation Azure. Pour obtenir la liste des magasins de données pris en charge, consultez [Magasins de données pris en charge dans Azure Data Share](./supported-data-stores.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-data-share-share-storage-account/).

:::code language="json" source="~/quickstart-templates/101-data-share-share-storage-account/azuredeploy.json":::

Les ressources suivantes sont définies dans le modèle :

* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) :
* [Microsoft.Storage/storageAccounts/blobServices/containers](/azure/templates/microsoft.storage/storageaccounts/blobservices/containers)
* [Microsoft.DataShare/accounts](/azure/templates/microsoft.datashare/accounts)
* [Microsoft.DataShare/accounts/shares](/azure/templates/microsoft.datashare/accounts/shares)
* [Microsoft.Storage/storageAccounts/providers/roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [Microsoft.DataShare/accounts/shares/dataSets](/azure/templates/microsoft.datashare/accounts/shares/datasets)
* [Microsoft.DataShare/accounts/shares/invitations](/azure/templates/microsoft.datashare/accounts/shares/invitations)
* [Microsoft.DataShare/accounts/shares/synchronizationSettings](/azure/templates/microsoft.datashare/accounts/shares/synchronizationsettings)

Ce modèle effectue les opérations suivantes :

* Il crée un compte de stockage et un conteneur utilisé comme source de données Data Share.
* Il crée un compte et un partage Data Share.
* Il crée une attribution de rôle pour accorder le rôle de Lecteur des données blob du stockage à la ressource de partage de données source. Voir [Rôles et exigences pour Azure Data Share](./concepts-roles-permissions.md).
* Il ajoute un jeu de données au partage Data Share.
* Il ajoute des destinataires au partage Data Share.
* Il active une planification d'instantanés pour votre partage Data Share.

Ce modèle est créé à des fins d'apprentissage. Dans la pratique, vous disposez généralement de données dans un compte de stockage existant. Vous devez créer l'attribution de rôle avant d'exécuter un modèle ou un script pour créer le jeu de données. Le message d'erreur suivant peut apparaître lorsque vous déployez le modèle :

```plaintext
"Missing permissions for DataShareAcccount on resource 'subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Storage/storageAccounts/<STORAGE ACCOUNT NAME>' (Code: 5006)"
```

Cela est dû au fait que le déploiement tente de créer le jeu de données avant la finalisation de l’affectation des rôles Azure. Malgré ce message d'erreur, le déploiement peut aboutir. Vous pourrez toujours parcourir [Vérifier les ressources déployées](#review-deployed-resources).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez l’image ci-après pour vous connecter à Azure et ouvrir le modèle.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-share-share-storage-account%2Fazuredeploy.json)
1. Sélectionnez ou entrez les valeurs suivantes :

    * **Abonnement**  : sélectionnez un abonnement Azure utilisé pour créer le partage de données et les autres ressources.
    * **Groupe de ressources**  : sélectionnez **Créer** pour créer un groupe de ressources ou sélectionnez un groupe de ressources existant.
    * **Emplacement** : sélectionnez un emplacement pour le groupe de ressources.
    * **Nom du projet**  : entrez un nom de projet.  Le nom du projet est utilisé pour générer des noms de ressource.  Consultez les définitions de variables dans le modèle précédent.
    * **Emplacement**  : sélectionnez un emplacement pour les ressources.  Vous pouvez utiliser le même emplacement pour le groupe de ressources.
    * **E-mail d'invitation**  : entrez l'adresse e-mail de connexion Azure du destinataire du partage de données.  L'alias de messagerie ne fonctionne pas.

    Utilisez la valeur par défaut pour les autres paramètres.
1. Sélectionnez **J'accepte les conditions générales mentionnées ci-dessus** , puis **Acheter**.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Ouvrez le compte Data Share que vous avez créé.
1. Dans le menu de gauche, sélectionnez **Envoyer des partages**.  Le compte de stockage doit être répertorié.
1. Sélectionnez le compte de stockage.  Sous **Détails** , vous verrez le paramètre de synchronisation tel que vous l'avez configuré dans le modèle.

    ![Paramètres de synchronisation du compte de stockage Azure Data Share](./media/share-your-data-arm/azure-data-share-storage-account-synchronization-settings.png)
1. Sélectionnez **Invitations** en haut. Vous verrez l'adresse e-mail que vous avez spécifiée lorsque vous avez déployé le modèle. L' **État** doit être **En attente**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment créer un partage Azure Data Share et inviter des destinataires. Pour en savoir plus sur la façon dont un consommateur de données peut accepter et recevoir un partage de données, suivez le tutoriel [​​Accepter et recevoir des données](subscribe-to-data-share.md).
