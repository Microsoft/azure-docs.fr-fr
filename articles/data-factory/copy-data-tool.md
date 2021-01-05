---
title: Outil Copier des données dans Azure Data Factory
description: Fournit des informations sur l’outil Copier des données dans l’interface utilisateur d’Azure Data Factory
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: yexu
ms.openlocfilehash: b0ba14a05ac42e9fccc1a146ad5943716652104d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89434195"
---
# <a name="copy-data-tool-in-azure-data-factory"></a>Outil Copier des données dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’outil Copier des données d’Azure Data Factory facilite et optimise le processus de réception des données dans Azure Data Lake, qui est généralement la première étape dans un scénario d’intégration des données de bout en bout.  Il permet de gagner du temps, surtout lorsque vous utilisez Azure Data Factory pour réceptionner des données à partir d’une source de données pour la première fois. Voici certains des avantages de l’utilisation de cet outil :

- Lorsque vous utilisez l’outil Copier des données Azure Data Factory, vous n’avez pas besoin de comprendre les définitions Data Factory pour les services, jeux de données, pipelines, activités et déclencheurs liés. 
- Le flux de l’outil Copier des données est intuitif pour le chargement des données dans Data Lake. L’outil crée automatiquement toutes les ressources Data Factory nécessaires pour copier des données depuis le magasin de données source sélectionné vers le magasin de données de destination/récepteur sélectionné. 
- L’outil Copier des données vous permet de valider les données en cours de réception au moment de la création, et d’éviter ainsi des erreurs potentielles en amont.
- Si vous devez implémenter une logique métier complexe pour charger des données dans Data Lake, vous pouvez toujours modifier les ressources Data Factory créées par l’outil Copier des données à l’aide de la création par activité dans l’interface utilisateur Data Factory. 

Le tableau suivant fournit des instructions concernant l’utilisation de Copier des données par rapport à la création par activité dans l’interface utilisateur Data Factory : 

| Outil Copier des données | Création par activité (activité de copie) |
| -------------- | -------------------------------------- |
| Vous souhaitez créer facilement une tâche de chargement des données sans connaître en détail les entités Azure Data Factory (services, jeux de données, pipelines liés, etc.). | Vous souhaitez implémenter une logique complexe et flexible pour charger des données dans Data Lake. |
| Vous souhaitez charger rapidement un grand nombre d’artefacts de données dans Data Lake. | Vous souhaitez lier l’activité de copie à d’autres activités suivantes pour le nettoyage ou le traitement des données. |

Pour lancer l’outil Copier des données, cliquez sur la mosaïque **Copier des données** sur la page d’accueil de votre fabrique de données.

![Page de prise en main - lien vers l’outil Copier des données](./media/doc-common-process/get-started-page.png)


## <a name="intuitive-flow-for-loading-data-into-a-data-lake"></a>Flux intuitif pour le chargement des données dans Data Lake
Cet outil vous permet de déplacer facilement et en quelques minutes des données provenant de nombreuses sources différentes vers des destinations, à l’aide d’un flux intuitif :  

1. Configurez les paramètres pour la **source**.
2. Configurez les paramètres pour la **destination**. 
3. Configurez les **paramètres avancés** pour l’opération de copie, notamment le mappage de colonnes, les paramètres de performances et les paramètres de tolérance de panne. 
4. Spécifiez une **planification** pour les tâche de chargement des données. 
5. Examinez le **résumé** des entités Data Factory à créer. 
6. **Modifiez** le pipeline pour mettre à jour les paramètres de l’activité de copie, selon les besoins. 

   La conception de l’outil est particulièrement adaptée au Big Data, avec la prise en charge de divers types d’objet et de données. Vous pouvez l’utiliser pour déplacer des centaines de dossiers, fichiers ou tables. L’outil prend en charge l’aperçu des données automatique, la capture et le mappage automatique de schéma, ainsi que le filtrage des données.

![Outil Copier des données](./media/copy-data-tool/copy-data-tool.png)

## <a name="automatic-data-preview"></a>Aperçu des données automatique
Vous pouvez afficher un aperçu d’une partie des données à partir du magasin de données source sélectionné et valider ainsi les données copiées. En outre, si les données sources sont contenues dans un fichier texte, l’outil Copier des données analyse ce fichier pour détecter les délimiteurs de colonnes et de lignes ainsi que le schéma.

![Paramètres du fichier](./media/copy-data-tool/file-format-settings.png)

Après la détection :

![Paramètres du fichier détecté et aperçu](./media/copy-data-tool/after-detection.png)

## <a name="schema-capture-and-automatic-mapping"></a>Capture et mappage automatique du schéma
Souvent, le schéma de la source de données n’est peut-être pas identique au schéma de la destination de données. Le cas échéant, vous devez mapper les colonnes du schéma source aux colonnes du schéma de destination.

L’outil Copier des données surveille votre comportement et s’y adapte lorsque vous mappez des colonnes entre des magasins source et destination. Après avoir sélectionné une ou plusieurs colonnes à partir du magasin de données source et les avoir mappées au schéma de destination, l’outil Copier des données commence à analyser le modèle pour les paires de colonnes que vous avez choisies des deux côtés. Il applique ensuite le même modèle aux autres colonnes. Par conséquent, quelques clics suffisent pour afficher comme vous le souhaitez toutes les colonnes mappées vers la destination.  Si vous n’êtes pas satisfait du choix du mappage de colonnes fourni par l’outil Copier des données, vous pouvez l’ignorer et continuer en mappant manuellement les colonnes. Pendant ce temps, l’outil Copier des données assimile et met à jour continuellement le modèle jusqu’à obtenir le modèle adapté au mappage de colonnes que vous voulez. 

> [!NOTE]
> Lors de la copie de données depuis SQL Server ou Azure SQL Database vers Azure Synapse Analytics (anciennement SQL Data Warehouse), si la table n’existe pas dans le magasin de destination, l’outil Copier des données prend en charge la création automatique de la table à l’aide du schéma source. 

## <a name="filter-data"></a>Filtrer les données
Vous pouvez filtrer les données sources pour sélectionner uniquement celles qui doivent être copiées vers le magasin de données de récepteur. Le filtrage réduit le volume des données à copier vers le magasin de données de récepteur et améliore de ce fait le débit de la copie. L’outil Copier des données fournit un moyen souple de filtrer les données dans une base de données relationnelle à l’aide du langage de requête SQL ou les fichiers d’un dossier d’objets blob Azure. 

### <a name="filter-data-in-a-database"></a>Filtrer les données d’une base de données
La capture d’écran suivante montre une requête SQL permettant de filtrer les données.

![Filtrer les données d’une base de données](./media/copy-data-tool/filter-data-in-database.png)

### <a name="filter-data-in-an-azure-blob-folder"></a>Filtrer les données dans un dossier d’objets blob Azure
Vous pouvez utiliser des variables dans le chemin d’accès au dossier pour copier des données à partir d’un dossier. Les variables prises en charge sont les suivantes : **{year}** , **{month}** , **{day}** , **{hour}** et **{minute}** . Exemple : inputfolder/{year}/{month}/{day}. 

Supposons que vos dossiers d’entrée présentent le format suivant : 

```
2016/03/01/01
2016/03/01/02
2016/03/01/03
...
```

Cliquez sur le bouton **Parcourir** à côté de **Fichier ou dossier**, accédez à l’un de ces dossiers (par exemple, 2016->03->01->02), puis cliquez sur **Choisir**. Vous devez voir 2016/03/01/02 dans la zone de texte. 

Puis remplacez **2016** par **{year}** , **03** par **{month}** , **01** par **{day}** et **02** par **{hour}** , puis appuyez sur la touche de **tabulation**. Vous devez maintenant voir des listes déroulantes pour sélectionner le format de ces quatre variables :

![Filtrer un fichier ou un dossier](./media/copy-data-tool/filter-file-or-folder.png)

L’outil Copier des données génère des paramètres avec des expressions, fonctions et variables système qui peuvent être utilisés pour représenter les variables {year}, {month}, {day}, {hour} et {minute} lors de la création du pipeline.

## <a name="scheduling-options"></a>Options de planification
Vous pouvez effectuer l’opération de copie une seule fois ou la répéter selon une planification établie (horaire, quotidienne et ainsi de suite.). Ces options peuvent être utilisées pour les connecteurs entre différents environnements, notamment les environnements locaux, dans le cloud ou sur un bureau local. 

Une opération de copie unique ne permet le déplacement des données à d’une source vers une destination qu’une seule fois. Elle s’applique aux données de toute taille et de n’importe quel format pris en charge. La copie planifiée vous permet de copier des données selon la récurrence de votre choix. Vous pouvez utiliser les paramètres avancés (nouvelle tentative, délai d’attente, alertes, etc.) pour configurer la copie planifiée.

![Options de planification](./media/copy-data-tool/scheduling-options.png)


## <a name="next-steps"></a>Étapes suivantes
Essayez ces didacticiels qui utilisent l’outil Copier des données :

- [Démarrage rapide : créer une fabrique de données à l’aide de l’outil Copier des données](quickstart-create-data-factory-copy-data-tool.md)
- [Didacticiel : copier des données dans Azure à l’aide de l’outil Copier des données](tutorial-copy-data-tool.md) 
- [Didacticiel : copier des données locales dans Azure à l’aide de l’outil Copier des données](tutorial-hybrid-copy-data-tool.md)
