---
title: Créer une fabrique de données Azure avec un modèle Azure Resource Manager (modèle ARM)
description: Créez un exemple de pipeline Azure Data Factory en utilisant un modèle Azure Resource Manager (modèle ARM).
services: data-factory
ms.service: data-factory
tags: azure-resource-manager
ms.workload: data-services
author: dcstwh
ms.author: weetok
ms.reviewer: maghan, jingwang
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 07/16/2020
ms.openlocfilehash: 0a9c4f6d9e98e721370f54dfbe261025cf186b6c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497484"
---
# <a name="quickstart-create-an-azure-data-factory-using-arm-template"></a>Démarrage rapide : Créer une fabrique de données Azure à l’aide d’un modèle ARM

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-build-your-first-pipeline-using-arm.md)
> * [Version actuelle](quickstart-create-data-factory-resource-manager-template.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Ce démarrage rapide décrit comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une fabrique de données Azure. Le pipeline que vous créez dans cette fabrique de données **copie** les données d’un dossier vers un autre dossier dans un stockage Blob Azure. Pour suivre un tutoriel sur la **transformation** des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Transformer des données à l’aide de Spark](transform-data-using-spark.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

> [!NOTE]
> Cet article ne fournit pas de présentation détaillée du service Data Factory. Pour une présentation du service Azure Data Factory, consultez [Présentation d’Azure Data Factory](introduction.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

### <a name="azure-subscription"></a>Abonnement Azure

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

### <a name="create-a-file"></a>Créer un fichier

Ouvrez un éditeur de texte tel que **Bloc-notes**, puis créez un fichier nommé **emp.txt** avec le contenu suivant :

```emp.txt
John, Doe
Jane, Doe
```

Enregistrez-le dans le dossier **C:\ADFv2QuickStartPSH** (si le dossier n’existe pas, créez-le).

## <a name="review-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-data-factory-v2-blob-to-blob-copy/).

:::code language="json" source="~/quickstart-templates/101-data-factory-v2-blob-to-blob-copy/azuredeploy.json":::

Des ressources sont définies dans le modèle :

- [Microsoft.Storage/storageAccounts](/azure/templates/Microsoft.Storage/storageAccounts) : définit un compte de stockage.
- [Microsoft.DataFactory/factories](/azure/templates/microsoft.datafactory/factories) : crée une fabrique de données Azure.
- [Microsoft.DataFactory/factories/linkedServices](/azure/templates/microsoft.datafactory/factories/linkedservices) : crée un service lié Azure Data Factory.
- [Microsoft.DataFactory/factories/datasets](/azure/templates/microsoft.datafactory/factories/datasets) : crée un jeu de données Azure Data Factory.
- [Microsoft.DataFactory/factories/pipelines](/azure/templates/microsoft.datafactory/factories/pipelines) : crée un pipeline Azure Data Factory.

Vous trouverez d’autres exemples de modèles Azure Data Factory dans la [galerie de modèles de démarrage rapide](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée un compte Azure Data Factory, un compte de stockage et un conteneur d’objets blob.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-deploy-arm-template.png" alt-text="Déployer un modèle ARM ADF":::

    Sauf aucune valeur n’est spécifiée, utilisez les valeurs par défaut pour créer les ressources Azure Data Factory :

    - **Abonnement**: Sélectionnez un abonnement Azure.
    - **Groupe de ressources** : Sélectionnez **Créer**, entrez un nom unique pour le groupe de ressources, puis sélectionnez **OK**.
    - **Région** : Sélectionnez un emplacement.  Par exemple, *USA Est*.
    - **Nom de la fabrique de données** : Utilisez la valeur par défaut.
    - **Emplacement** : Utilisez la valeur par défaut.
    - **Nom du compte de stockage** : Utilisez la valeur par défaut.
    - **Conteneur d’objets blob** : Utilisez la valeur par défaut.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

1. Sélectionnez **Accéder au groupe de ressources**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-go-to-resource-group.png" alt-text="Groupe de ressources":::

2.  Vérifiez que votre Azure Data Factory est créé.
    1. Le nom de votre Azure Data Factory est au format datafactory\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-sample.png" alt-text="Exemple de Data Factory":::

2. Vérifiez que votre compte de stockage est créé.
    1. Le nom du compte de stockage est au format storage\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-storage-account.png" alt-text="Compte de stockage":::

3. Sélectionnez le compte de stockage créé, puis **Conteneurs**.
    1. Sur la page **Conteneurs**, sélectionnez le conteneur d’objets blob que vous avez créé.
        1. Le nom du conteneur d’objets blob est au format blob\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-blob-container.png" alt-text="Conteneur d’objets blob":::

### <a name="upload-a-file"></a>Charger un fichier

1. Sur la page **Conteneurs**, sélectionnez **Charger**.

2. Dans le volet de droite, activez la zone **Fichiers**, puis recherchez et sélectionnez le fichier **emp.txt** que vous avez créé précédemment.

3. Développez le titre **Avancé**.

4. Dans la zone **Charger dans le dossier**, entrez *input*.

5. Cliquez sur le bouton **Charger**. Vous devriez voir le fichier **emp.txt** et l’état du chargement dans la liste.

6. Sélectionnez l’icône **Fermer** (**X**) pour fermer la page **Charger l’objet blob**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-upload-blob-file.png" alt-text="Charger le fichier dans le dossier d’entrée":::

Laissez la page du conteneur ouverte, car elle vous permettra de vérifier la sortie à l’issue de ce démarrage rapide.

### <a name="start-trigger"></a>Démarrer le déclencheur

1. Accédez à la page **fabriques de données**, puis sélectionnez la fabrique de données que vous avez créée. 

2. Sélectionnez la vignette **Créer et surveiller**. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author-monitor-tile.png" alt-text="Auteur et moniteur":::

2. Sélectionnez l’onglet **Auteur** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author.png" border="false":::.

3. Sélectionnez le pipeline créé ArmtemplateSampleCopyPipeline.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-pipelines.png" alt-text="Pipeline de modèle ARM":::

4. Sélectionnez **Ajouter un déclencheur** > **Déclencher maintenant**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-trigger-now.png" alt-text="Déclencheur":::

5. Dans le volet droit, sous **Exécuter le pipeline**, sélectionnez **OK**.

### <a name="monitor-the-pipeline"></a>Surveiller le pipeline

1. Sélectionnez l’onglet **Moniteur** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-monitor.png" border="false":::.

2. Vous voyez les exécutions d’activités associées avec l’exécution du pipeline. Dans ce guide de démarrage rapide, le pipeline n’a qu’une seule activité de type : Copier. Par conséquent, vous voyez une exécution pour cette activité.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-successful-run.png" alt-text="Exécution réussie":::

### <a name="verify-the-output-file"></a>Vérifier le fichier de sortie

Le pipeline crée automatiquement un dossier de sortie dans le conteneur d’objets blob. Ensuite, il copie le fichier emp.txt à partir du dossier d’entrée dans le dossier de sortie. 

1. Dans le portail Azure, dans la page **Conteneurs**, sélectionnez **Actualiser** pour voir le dossier de sortie. 

2. Dans la liste des dossiers, sélectionnez **output**.

3. Vérifiez que le fichier **emp.txt** a été copié dans le dossier de sortie. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-output.png" alt-text="Sortie":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous disposez de deux moyens de supprimer les ressources que vous avez créées dans le guide de démarrage rapide. Vous pouvez [supprimer le groupe de ressources Azure](../azure-resource-manager/management/delete-resource-group.md) qui inclut toutes les ressources du groupe de ressources. Si vous souhaitez conserver les autres ressources, supprimez uniquement la fabrique de données créée dans ce tutoriel.

Si vous supprimez un groupe de ressources, toutes les ressources qu’il contient, y compris les fabriques de données, seront supprimées. Exécutez la commande suivante pour supprimer l’intégralité du groupe de ressources : 

```azurepowershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

Si vous souhaitez supprimer uniquement la fabrique de données, et non pas le groupe de ressources entier, exécutez la commande suivante : 

```azurepowershell-interactive
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé une fabrique de données Azure à l’aide d’un modèle ARM, et avez validé le déploiement. Pour en savoir plus sur Azure Data Factory et Azure Resource Manager, passez aux articles ci-dessous.

- [Documentation Azure Data Factory](index.yml)
- En savoir plus sur [Azure Resource Manager](../azure-resource-manager/management/overview.md)
- Obtenez d’autres [modèles ARM Azure Data Factory](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular)