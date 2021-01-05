---
title: Traiter des fichiers texte de longueur fixe avec des flux de données de mappage dans Azure Data Factory
description: Découvrez comment traiter des fichiers texte de longueur fixe dans Azure Data Factory à l’aide de flux de données de mappage.
services: data-factory
author: balakreshnan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 8/18/2019
ms.author: makromer
ms.openlocfilehash: 23b812da8c84ebf055ac4eabdc4649828c139a7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89051013"
---
# <a name="process-fixed-length-text-files-by-using-data-factory-mapping-data-flows"></a>Traiter des fichiers texte de longueur fixe à l’aide de flux de données de mappage Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Vous pouvez transformer les données de fichiers texte de longueur fixe en utilisant des flux de données de mappage dans Microsoft Azure Data Factory. Dans le cadre de la tâche suivante, nous allons définir un jeu de données pour un fichier texte sans délimiteur, puis configurer des divisions de sous-chaînes en fonction de la position ordinale.

## <a name="create-a-pipeline"></a>Créer un pipeline

1. Sélectionnez **+Nouveau pipeline** pour créer un pipeline.

2. Ajoutez une activité de flux de données qui sera utilisée pour le traitement des fichiers de longueur fixe :

    ![Pipeline de largeur fixe](media/data-flow/fwpipe.png)

3. Dans l’activité de flux de données, sélectionnez **Nouveau flux de données de mappage**.

4. Ajoutez une transformation de source, de colonne dérivée, de sélection et de récepteur :

    ![Flux de données de largeur fixe](media/data-flow/fw2.png)

5. Configurez la transformation de source pour utiliser un nouveau jeu de données de type Texte délimité.

6. Ne définissez pas d’en-tête ou de délimiteur de colonne.

   Nous allons à présent définir des points de départ et des longueurs de champ pour le contenu de ce fichier :

    ```
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    1234567813572468
    ```

7. Sous l’onglet **Projection** de votre transformation de source, vous devez voir une colonne de chaîne nommée *Column_1*.

8. Créez une colonne dans la colonne dérivée.

9. Nous allons donner des noms simples aux colonnes, tels que *col1*.

10. Dans le générateur d’expressions, tapez la chaîne suivante :

    ```substring(Column_1,1,4)```

    ![Colonne dérivée](media/data-flow/fwderivedcol1.png)

11. Répétez l’étape 10 pour toutes les colonnes que vous devez analyser.

12. Sélectionnez l’onglet **Inspecter** pour afficher les nouvelles colonnes qui seront générées :

    ![Inspecter](media/data-flow/fwinspect.png)

13. Utilisez la transformation de sélection (Select) pour supprimer les colonnes dont vous n’avez pas besoin pour la transformation :

    ![Transformation de sélection](media/data-flow/fwselect.png)

14. Utilisez le récepteur (Sink) pour générer les données dans un dossier :

    ![Récepteur de largeur fixe](media/data-flow/fwsink.png)

    Le résultat se présente ainsi :

    ![Sortie de largeur fixe](media/data-flow/fxdoutput.png)

  Les données de longueur fixe sont maintenant divisées en groupes de quatre caractères et attribuées à Col1, Col2, Col3, Col4, etc. Les données sont divisées en quatre colonnes, conformément à l’exemple précédent.

## <a name="next-steps"></a>Étapes suivantes

* Créez le reste de votre logique de flux de données à l’aide de [transformations](concepts-data-flow-overview.md) de flux de données de mappage.
