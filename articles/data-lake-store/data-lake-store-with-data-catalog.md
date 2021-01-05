---
title: Intégrer Data Lake Storage Gen1 avec Azure Data Catalog
description: Découvrez comment inscrire des données d’Azure Data Lake Storage Gen1 dans Azure Data Catalog pour les rendre détectables dans votre organisation.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 02544489816f5711ca6e599c2bce03737c747934
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92106618"
---
# <a name="register-data-from-azure-data-lake-storage-gen1-in-azure-data-catalog"></a>Inscrire des données Data Lake Storage Gen1 dans Azure Data Catalog
Dans cet article, vous allez découvrir comment intégrer Azure Data Lake Storage Gen1 à Azure Data Catalog pour rendre vos données détectables au sein d’une organisation en l’intégrant à Data Catalog. Pour plus d’informations sur le catalogage des données, consultez [Azure Data Catalog](../data-catalog/overview.md). Pour comprendre les scénarios dans lesquels vous pouvez utiliser Data Catalog, consultez [Scénarios courants d’Azure Data Catalog](../data-catalog/data-catalog-common-scenarios.md).

## <a name="prerequisites"></a>Prérequis
Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Activez votre abonnement Azure** pour Data Lake Storage Gen1. Consultez les [instructions](data-lake-store-get-started-portal.md).
* **Un compte Data Lake Storage Gen1**. Suivez les instructions de [Prise en main d’Azure Data Lake Storage Gen1 avec le portail Azure](data-lake-store-get-started-portal.md). Pour ce tutoriel, nous allons créer un compte Data Lake Storage Gen1 appelé **datacatalogstore**.

    Une fois que vous avez créé le compte, chargez-y un exemple de jeu de données. Pour ce didacticiel, chargeons tous les fichiers .csv sous le dossier **AmbulanceData** dans le [dépôt Git Azure Data Lake](https://github.com/Azure/usql/tree/master/Examples/Samples/Data/AmbulanceData/). Vous pouvez utiliser différents clients, comme [Explorateur de stockage Azure](https://storageexplorer.com/), pour charger des données dans un conteneur d’objets blob.
* **Azure Data Catalog**. Votre organisation doit déjà avoir un catalogue de données Azure créé pour votre organisation. Un seul catalogue est autorisé pour chaque organisation.

## <a name="register-data-lake-storage-gen1-as-a-source-for-data-catalog"></a>Inscrire Data Lake Storage Gen1 comme source pour Data Catalog

> [!VIDEO https://channel9.msdn.com/Series/AzureDataLake/ADCwithADL/player]

1. Accédez à `https://azure.microsoft.com/services/data-catalog`, puis cliquez sur **Prise en main**.
1. Connectez-vous au portail Azure Data Catalog, puis cliquez sur **Publier des données**.

    ![Inscription d’une source de données](./media/data-lake-store-with-data-catalog/register-data-source.png "Référencer une source de données")
1. Sur la page suivante, cliquez sur **Lancer l’application**. Ceci télécharge le fichier manifeste d’application sur votre ordinateur. Double-cliquez sur le fichier manifeste pour démarrer l’application.
1. Sur la page Bienvenue, cliquez sur **Connexion**, puis entrez vos informations d’identification.

    ![Écran d’accueil](./media/data-lake-store-with-data-catalog/welcome.screen.png "Écran d’accueil")
1. Dans la page Sélectionner une source de données, sélectionnez **Azure Data Lake Store**, puis cliquez sur **Suivant**.

    ![Sélection de la source de données](./media/data-lake-store-with-data-catalog/select-source.png "Sélectionnez la source de données")
1. Sur la page suivante, spécifiez le nom du compte Data Lake Storage Gen1 que vous voulez inscrire dans Data Catalog. Laissez les autres options à leur valeur par défaut, puis cliquez sur **Connexion**.

    ![Connexion à la source de données](./media/data-lake-store-with-data-catalog/connect-to-source.png "Se connecter à une sources de données")
1. La page suivante peut être divisée selon les segments suivants.

    a. La zone **Hiérarchie du serveur** représente la structure des dossiers du compte Data Lake Storage Gen1. **$Root** représente la racine du compte Data Lake Storage Gen1 et **AmbulanceData** représente le dossier créé à la racine du compte Data Lake Storage Gen1.

    b. La zone **Objets disponibles** répertorie les fichiers et les dossiers présents sous le dossier **AmbulanceData**.

    c. La zone **Objets à référencer** répertorie les fichiers et dossiers à référencer dans Azure Data Catalog.

    ![Capture d’écran de la boîte de dialogue Microsoft Azure Data Catalog - compte de stockage.](./media/data-lake-store-with-data-catalog/view-data-structure.png "Afficher la structure des données")
1. Pour ce didacticiel, vous devez référencer tous les fichiers du répertoire. Pour cela, cliquez sur le bouton (![déplacer des objets](./media/data-lake-store-with-data-catalog/move-objects.png "Déplacement d’objets")) pour déplacer tous les fichiers dans la zone **Objets à référencer** .

    Comme les données sont référencées dans un catalogue de données au niveau de l’organisation, il est recommandé d’ajouter des métadonnées que vous pouvez utiliser ultérieurement pour localiser rapidement les données. Par exemple, vous pouvez ajouter une adresse de messagerie pour le propriétaire des données (par exemple une personne qui charge les données) ou ajouter une étiquette pour identifier les données. La capture d’écran ci-dessous montre une étiquette que nous ajoutons aux données.

    ![Capture d’écran de la boîte de dialogue Microsoft Azure Data Catalog - compte de stockage avec la balise ajoutée aux données reprises.](./media/data-lake-store-with-data-catalog/view-selected-data-structure.png "Afficher la structure des données")

    Cliquez sur **S'inscrire**.
1. La capture d’écran suivante montre que les données sont référencées avec succès dans le catalogue de données.

    ![Inscription terminée](./media/data-lake-store-with-data-catalog/registration-complete.png "Afficher la structure des données")
1. Cliquez sur **Afficher le portail** pour revenir au portail Data Catalog et vérifiez que vous pouvez maintenant accéder aux données référencées à partir du portail. Pour rechercher les données, vous pouvez utiliser l’étiquette que vous avez utilisée lors du référencement des données.

     ![Recherche de données dans le catalogue](./media/data-lake-store-with-data-catalog/search-data-in-catalog.png "Rechercher des données dans le catalogue")
1. Vous pouvez maintenant effectuer des opérations comme ajouter des annotations et de la documentation aux données. Pour plus d’informations, consultez les liens suivants :

    * [Annoter des sources de données dans Data Catalog](../data-catalog/data-catalog-how-to-annotate.md)
    * [Documenter des sources de données dans Data Catalog](../data-catalog/data-catalog-how-to-documentation.md)

## <a name="see-also"></a>Voir aussi
* [Annoter des sources de données dans Data Catalog](../data-catalog/data-catalog-how-to-annotate.md)
* [Documenter des sources de données dans Data Catalog](../data-catalog/data-catalog-how-to-documentation.md)
* [Intégrer Data Lake Storage Gen1 avec d’autres services Azure](data-lake-store-integrate-with-other-services.md)