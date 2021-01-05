---
title: Copier facilement des données avec l’Assistant de copie – Azure
description: Découvrez comment utiliser l’Assistant Data Factory Copy pour copier des données à partir de sources de données prises en charge dans des récepteurs.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: ''
editor: ''
ms.assetid: f904972f-cd33-48db-9755-2b3196ae4168
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 28169e43d0e6949a16cc56c7e7d5d91d6db1ef57
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003036"
---
# <a name="copy-or-move-data-easily-with-azure-data-factory-copy-wizard"></a>Copier ou déplacer facilement des données avec l’Assistant de copie Azure Data Factory
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez le [tutoriel sur l’activité de copie](../quickstart-create-data-factory-dot-net.md). 


L’Assistant Azure Data Factory Copy doit faciliter le processus de réception des données, qui est généralement la première étape dans un scénario d’intégration des données de bout en bout. Lorsque vous passez par l’Assistant Azure Data Factory Copy, il est inutile de comprendre les définitions JSON pour les services liés, les jeux de données et les pipelines. Toutefois, une fois que vous avez effectué toutes les étapes de l’Assistant, l’Assistant crée automatiquement un pipeline pour copier les données de la source de données sélectionnée vers la destination sélectionnée. En outre, l’Assistant Azure Data Factory Copy vous permet de valider les données reçues au moment de la création, ce qui vous permet de gagner beaucoup de temps, en particulier lorsque vous recevez des données pour la première fois de la source de données. Pour lancer l’Assistant Data Factory Copy, cliquez sur la mosaïque **Copier les données** sur la page d’accueil de votre fabrique de données.

![Assistant de copie](./media/data-factory-copy-wizard/copy-data-wizard.png)

## <a name="an-intuitive-wizard-for-copying-data"></a>Un Assistant intuitif pour copier des données
Cet Assistant vous permet de déplacer facilement et en quelques minutes des données provenant de nombreuses sources différentes vers des destinations. Une fois que vous avez utilisé l’Assistant, un pipeline avec une activité de copie est automatiquement créé, ainsi que des entités Data Factory dépendantes (services et jeux de données liés). Aucune étape supplémentaire n’est nécessaire pour créer le pipeline.   

![Sélectionnez la source de données](./media/data-factory-copy-wizard/select-data-source-page.png)

> [!NOTE]
> Consultez l’article [Tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Data Factory Copy](data-factory-copy-data-wizard-tutorial.md) pour obtenir les instructions détaillées pour créer un exemple de pipeline afin de copier des données d’un objet blob Azure vers une table Azure SQL Database. 
> 
> 

Dès le début, l’Assistant est conçu pour traiter des données volumineuses (Big Data). Avec l’Assistant Data Factory Copy, vous disposez d’un outil simple et performant pour créer des pipelines Data Factory capables de déplacer des centaines de dossiers, fichiers ou tables. L’Assistant prend en charge les trois fonctionnalités suivantes : aperçu des données automatique, capture et mappage de schéma, et filtrage des données. 

## <a name="automatic-data-preview"></a>Aperçu des données automatique
L’Assistant Data Factory Copy vous permet de consulter une partie des données à partir de la source de données sélectionnée pour que vous puissiez vérifier qu’il s’agit bien des données que vous voulez copier. En outre, si les données sources sont contenues dans un fichier texte, l’Assistant analyse ce fichier pour découvrir automatiquement les délimiteurs de colonnes et de lignes ainsi que le schéma. 

![Paramètres de format de fichier](./media/data-factory-copy-wizard/file-format-settings.png)

## <a name="schema-capture-and-mapping"></a>Mappage et capture de schéma
Le schéma des données d’entrée peut ne pas correspondre au schéma des données de sortie dans certains cas. Le cas échéant, vous devez mapper les colonnes du schéma source aux colonnes du schéma de destination. 

L’Assistant Data Factory Copy mappe automatiquement les colonnes du schéma source aux colonnes du schéma de destination. Vous pouvez ignorer les mappages à l’aide des listes déroulantes (ou) indiquer si une colonne doit être ignorée lors de la copie des données.   

![Mappage de schéma](./media/data-factory-copy-wizard/schema-mapping.png)

## <a name="filtering-data"></a>Filtrage des données
L’Assistant vous permet de filtrer les données sources pour sélectionner uniquement celles qui doivent être copiées vers le magasin de données de destination/récepteur. Le filtrage réduit le volume des données à copier vers le magasin de données de récepteur et améliore de ce fait le débit de la copie. Il fournit un moyen souple de filtrer les données dans une base de données relationnelle à l’aide du langage de requête SQL (ou) les fichiers d’un dossier d’objets blob Azure à l’aide de [variables et fonctions Data Factory](data-factory-functions-variables.md).   

### <a name="filtering-of-data-in-a-database"></a>Filtrage des données dans une base de données
Dans cet exemple, la requête SQL utilise la fonction `Text.Format` et la variable `WindowStart`. 

![Valider des expressions](./media/data-factory-copy-wizard/validate-expressions.png)

### <a name="filtering-of-data-in-an-azure-blob-folder"></a>Filtrage des données dans un dossier d’objets blob Azure
Vous pouvez utiliser des variables dans le chemin du dossier pour copier des données à partir d’un dossier qui est déterminé au moment de l’exécution par les [variables système](data-factory-functions-variables.md#data-factory-system-variables). Les variables prises en charge sont les suivantes : **{year}** , **{month}** , **{day}** , **{hour}** , **{minute}** et **{custom}** . Exemple : inputfolder/{year}/{month}/{day}.

Supposons que vos dossiers d’entrée présentent le format suivant :

```text
2016/03/01/01
2016/03/01/02
2016/03/01/03
...
```

Cliquez sur le bouton **Parcourir** à côté de **Fichier ou dossier**, accédez à l’un de ces dossiers (par exemple, 2016->03->01->02), puis cliquez sur **Choisir**. Vous devez voir `2016/03/01/02` dans la zone de texte. À présent, remplacez **2016** par **{year}** , **03** par **{month}** , **01** par **{day}** et **02** par **{hour}** , puis appuyez sur la touche de tabulation. Vous devez maintenant voir des listes déroulantes pour sélectionner le format de ces quatre variables :

![Utilisation de variables système](./media/data-factory-copy-wizard/blob-standard-variables-in-folder-path.png)   

Comme la capture d’écran suivante le montre, vous pouvez également configurer une variable **custom** pour utiliser l’une des [chaînes de format prises en charge](/dotnet/standard/base-types/custom-date-and-time-format-strings). Pour sélectionner un dossier avec cette structure, utilisez d’abord le bouton **Parcourir** . Remplacez alors une valeur avec **{custom}** , puis appuyez sur Tab pour afficher la zone de texte dans laquelle vous pouvez taper la chaîne de format.     

![Utilisation de la variable custom](./media/data-factory-copy-wizard/blob-custom-variables-in-folder-path.png)

## <a name="support-for-diverse-data-and-object-types"></a>Prise en charge de divers types d’objet et de données
À l’aide de l’Assistant de copie, vous pouvez déplacer efficacement des centaines de tables, fichiers ou dossiers.

![Sélectionnez les tables à partir desquelles copier les données](./media/data-factory-copy-wizard/select-tables-to-copy-data.png)

## <a name="scheduling-options"></a>Options de planification
Vous pouvez effectuer l’opération de copie une seule fois ou la répéter selon une planification établie (horaire, quotidienne et ainsi de suite.). Ces deux options peuvent être définies pour l’ensemble des connecteurs utilisés pour la copie locale, dans le cloud ou sur un Bureau local.

Une opération de copie unique ne permet le déplacement des données à d’une source vers une destination qu’une seule fois. Elle s’applique aux données de toute taille et de n’importe quel format pris en charge. La copie planifiée vous permet de copier des données selon une périodicité définie. Vous pouvez utiliser les paramètres avancés (nouvelle tentative, délai d’attente, alertes, etc.) pour configurer la copie planifiée.

![Propriétés de planification](./media/data-factory-copy-wizard/scheduling-properties.png)

## <a name="next-steps"></a>Étapes suivantes
Pour découvrir une procédure pas à pas rapide sur l’utilisation de l’Assistant Copie de Data Factory afin de créer un pipeline avec une activité de copie, consultez le [Tutoriel : Créez un pipeline à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md).