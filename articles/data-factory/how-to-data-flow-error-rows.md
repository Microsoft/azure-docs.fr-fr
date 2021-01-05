---
title: Gérer les lignes d’erreur avec un mappage de flux de données dans Azure Data Factory
description: Découvrez comment gérer les erreurs de troncation SQL dans Azure Data Factory à l’aide du mappage de flux de données.
services: data-factory
author: kromerm
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/22/2020
ms.author: makromer
ms.openlocfilehash: c8b0ae7058aecc1813d720a3fbb2a1a1f967cf40
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352597"
---
# <a name="handle-sql-truncation-error-rows-in-data-factory-mapping-data-flows"></a>Gérer les lignes d’erreur de troncation SQL dans Data Factory en mappant les flux de données

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Un scénario courant dans Data Factory lors de l’utilisation de flux de données de mappage consiste à écrire vos données transformées dans Azure SQL Database. Dans ce scénario, une condition d’erreur courante que vous devez éviter est une troncation de colonne possible.

Il existe deux méthodes principales pour gérer correctement les erreurs lors de l’écriture de données vers votre récepteur de base de données dans les flux de données ADF :

* Définissez le récepteur [Gestion des lignes d’erreur](./connector-azure-sql-database.md#error-row-handling) sur « Continuer en cas d’erreur » lors du traitement des données de base de données. Cette méthode d’interception automatique ne nécessite pas de logique personnalisée dans votre flux de données.
* Vous pouvez également procéder comme suit pour fournir la journalisation des colonnes qui ne tiennent pas dans une colonne de chaîne cible. Cela permet à votre flux de données de se poursuivre.

> [!NOTE]
> Lorsque vous activez la gestion automatique des lignes d’erreur, par opposition à la méthode ci-dessous d’écriture de votre propre logique de gestion des erreurs, cela se traduit par une baisse des performances et implique une étape supplémentaire par ADF afin d’effectuer une opération en 2 phases pour intercepter les erreurs.

## <a name="scenario"></a>Scénario

1. Nous disposons d’une table de base de données cible qui comprend une colonne ```nvarchar(5)``` nommée « name ».

2. Dans notre flux de données, nous voulons mapper des titres de films de notre récepteur à cette colonne « name » cible.

    ![Flux de données de film 1](media/data-flow/error4.png)
    
3. Le problème est que le titre du film ne peut pas tenir dans une colonne de récepteur qui ne peut contenir que 5 caractères. Lorsque vous exécutez ce flux de données, vous recevez une erreur semblable à celle-ci : ```"Job failed due to reason: DF-SYS-01 at Sink 'WriteToDatabase': java.sql.BatchUpdateException: String or binary data would be truncated. java.sql.BatchUpdateException: String or binary data would be truncated."```

Cette vidéo présente un exemple de configuration de la logique de gestion des lignes d’erreur dans votre flux de données :
> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4uOHj]

## <a name="how-to-design-around-this-condition"></a>Comment concevoir en tenant compte de cette condition

1. Dans ce scénario, la longueur maximale de la colonne « name » est de cinq caractères. Nous allons donc ajouter une transformation de fractionnement conditionnel qui nous permettra de journaliser des lignes avec des « titres » de plus de cinq caractères, tout en autorisant que le reste des lignes pouvant tenir dans cet espace s’écrive dans la base de données.

    ![fractionnement conditionnel](media/data-flow/error1.png)

2. Cette transformation de fractionnement conditionnel définit la longueur maximale de « title » sur cinq. Toute ligne d’une longueur inférieure ou égale à cinq passe dans le flux ```GoodRows```. Toute ligne d’une longueur supérieure à cinq passe dans le flux ```BadRows```.

3. Nous devons maintenant journaliser les lignes qui ont échoué. Ajoutez une transformation de récepteur au flux de ```BadRows``` pour la journalisation. Ici, nous allons « mapper automatiquement » tous les champs afin d’obtenir la journalisation de l’enregistrement de transaction complet. Il s’agit d’une sortie de fichier CSV délimité par du texte transmis à un fichier unique dans le Stockage Blob Azure. Nous allons appeler le fichier journal « badrows. csv ».

    ![Lignes incorrectes](media/data-flow/error3.png)
    
4. Le flux de données terminé est illustré ci-dessous. Nous sommes maintenant en mesure de fractionner les lignes d’erreur afin d’éviter les erreurs de troncation SQL et de placer ces entrées dans un fichier journal. Pendant ce temps, les lignes réussies peuvent continuer d’écrire dans notre base de données cible.

    ![compléter le flux de données](media/data-flow/error2.png)

5. Si vous choisissez l’option de gestion des lignes d’erreur dans la transformation du récepteur et définissez « Lignes d’erreur de sortie », ADF génère automatiquement une sortie de fichier CSV de vos données de ligne, ainsi que les messages d’erreur signalés par le pilote. Avec cette autre option, vous n’êtes pas tenu d’ajouter manuellement cette logique à votre flux de données. Cette option entraîne une légère baisse des performances pour permettre à ADF d’implémenter une méthodologie en 2 phases afin d’intercepter et de consigner les erreurs.

    ![Flux de données terminé avec lignes d’erreur](media/data-flow/error-row-3.png)

## <a name="next-steps"></a>Étapes suivantes

* Créez le reste de votre logique de flux de données à l’aide de [transformations](concepts-data-flow-overview.md) de flux de données de mappage.