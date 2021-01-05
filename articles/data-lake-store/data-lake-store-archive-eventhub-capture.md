---
title: Capturer des données Event Hubs dans Azure Data Lake Storage Gen1
description: Découvrez comment utiliser Azure Data Lake Storage Gen1 pour capturer les données reçues par Azure Event Hubs. Commencez par vérifier la configuration requise.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 8be242369ecae2c809a38428284c9ddcad440e3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578238"
---
# <a name="use-azure-data-lake-storage-gen1-to-capture-data-from-event-hubs"></a>Utiliser Azure Data Lake Storage Gen1 pour capturer des données Event Hubs

Découvrez comment utiliser Azure Data Lake Storage Gen1 pour capturer les données reçues par Azure Event Hubs.

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Un compte Azure Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).

*  **Un espace de noms Event Hubs**. Pour obtenir des instructions, consultez [Créer un espace de noms Event Hubs](../event-hubs/event-hubs-create.md#create-an-event-hubs-namespace). Vérifiez que le compte Data Lake Storage Gen1 et l’espace de noms Event Hubs appartiennent au même abonnement Azure.


## <a name="assign-permissions-to-event-hubs"></a>Affecter des autorisations à Event Hubs

Dans cette section, vous allez créer un dossier au sein du compte dans lequel vous souhaitez capturer les données Event Hubs. Vous allez également affecter des autorisations à Event Hubs pour qu’il puisse écrire des données dans un compte Data Lake Storage Gen1. 

1. Ouvrez le compte Data Lake Storage Gen1 dans lequel vous souhaitez capturer les données Event Hubs, puis cliquez sur **Explorateur de données**.

    ![Explorateur de données Data Lake Storage Gen1](./media/data-lake-store-archive-eventhub-capture/data-lake-store-open-data-explorer.png "Explorateur de données Data Lake Storage Gen1")

1.  Cliquez sur **Nouveau dossier**, puis entrez un nom pour le dossier dans lequel vous souhaitez capturer les données.

    ![Créer un dossier dans Data Lake Storage Gen1](./media/data-lake-store-archive-eventhub-capture/data-lake-store-create-new-folder.png "Créer un dossier dans Data Lake Storage Gen1")

1. Affectez des autorisations à la racine du compte Data Lake Storage Gen1. 

    a. Cliquez sur **Explorateur de données**, sélectionnez la racine du compte Data Lake Storage Gen1, puis cliquez sur **Accès**.

    ![Screenshot of the Data explorer with the root of the account and the Access option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-permissions-to-root.png "Attribuer des autorisations à la racine de Data Lake Storage Gen1")

    b. Sous **Accès**, cliquez sur **Ajouter**, cliquez sur **Sélectionner un utilisateur ou un groupe**, puis recherchez `Microsoft.EventHubs`. 

    ![Screenshot of the Access page with the Add option, Select User or Group option, and Microsoft Eventhubs option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp.png "Attribuer des autorisations à la racine de Data Lake Storage Gen1")
    
    Cliquez sur **Sélectionner**.

    c. Sous **Affecter des autorisations**, cliquez sur **Sélectionner des autorisations**. Définissez **Autorisations** à **Exécuter**. Définissez **Ajouter à** sur **Ce dossier et tous ses enfants**. Définissez **Ajouter en tant que** sur **Une entrée d’autorisation d’accès et une entrée d’autorisation par défaut**.

    > [!IMPORTANT]
    > Lorsque vous créez une nouvelle hiérarchie de dossiers pour la capture des données reçues par Azure Event Hubs, cela permet d’accéder facilement au dossier de destination.  Toutefois, l’ajout d’autorisations à tous les enfants d’un dossier de niveau supérieur avec de nombreux fichiers et dossiers enfants peut prendre beaucoup de temps.  Si votre dossier racine contient un grand nombre de fichiers et dossiers, il peut être plus rapide d’ajouter des autorisations **Execute** pour `Microsoft.EventHubs` individuellement dans chaque dossier dans le chemin d’accès à votre dossier de destination finale. 

    ![Screenshot of the Assign Permissions section with the Select Permissions option called out. The Select Permissions section is next to it with the Execute option, Add to option, and Add as option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp1.png "Attribuer des autorisations à la racine de Data Lake Storage Gen1")

    Cliquez sur **OK**.

1. Affectez des autorisations pour le dossier situé dans le compte Data Lake Storage Gen1 où vous voulez capturer des données.

    a. Cliquez sur **Explorateur de données**, sélectionnez le dossier du compte Data Lake Storage Gen1, puis cliquez sur **Accès**.

    ![Screenshot of the Data explorer with a folder in the account and the Access option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-permissions-to-folder.png "Attribuer des autorisations au dossier de Data Lake Storage Gen1")

    b. Sous **Accès**, cliquez sur **Ajouter**, cliquez sur **Sélectionner un utilisateur ou un groupe**, puis recherchez `Microsoft.EventHubs`. 

    ![Screenshot of the Data explorer Access page with the Add option, Select User or Group option, and Microsoft Eventhubs option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp.png "Attribuer des autorisations au dossier de Data Lake Storage Gen1")
    
    Cliquez sur **Sélectionner**.

    c. Sous **Affecter des autorisations**, cliquez sur **Sélectionner des autorisations**. Définissez **Autorisations** sur **Lecture, écriture** et **exécution**. Définissez **Ajouter à** sur **Ce dossier et tous ses enfants**. Enfin, définissez **Ajouter en tant que** sur **Une entrée d’autorisation d’accès et une entrée d’autorisation par défaut**.

    ![Screenshot of the Assign Permissions section with the Select Permissions option called out. The Select Permissions section is next to it with the Read, Write, and Execute options, the Add to option, and the Add as option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-assign-eventhub-sp-folder.png "Attribuer des autorisations au dossier de Data Lake Storage Gen1")
    
    Cliquez sur **OK**. 

## <a name="configure-event-hubs-to-capture-data-to-data-lake-storage-gen1"></a>Configurer Event Hubs pour capturer des données et les envoyer vers Data Lake Storage Gen1

Dans cette section, vous allez créer un Event Hub dans un espace de noms Event Hubs. Vous allez également configurer l’Event Hub pour capturer des données et les envoyer vers un compte Azure Data Lake Storage Gen1. Cette section part du principe que vous avez déjà créé un espace de noms Event Hubs.

1. Dans le volet **Vue d’ensemble** de l’espace de noms Event Hubs, cliquez sur **+ Event Hub**.

    ![Screenshot of the Overview pane with the Event Hub option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-create-event-hub.png "Créer un hub d’événements")

1. Fournissez les valeurs suivantes pour configurer Event Hubs de manière à capturer des données et à les envoyer vers Data Lake Storage Gen1.

    ![Screenshot of the Create Event Hub dialog box with the Name text box, the Capture option, the Capture Provider option, the Select Data Lake Store option, and the Data Lake Path option called out.](./media/data-lake-store-archive-eventhub-capture/data-lake-store-configure-eventhub.png "Créer un hub d’événements")

    a. Donnez un nom à l’Event Hub.
    
    b. Pour ce didacticiel, définissez **Nombre de partitions** et **Rétention des messages** sur les valeurs par défaut.
    
    c. Définissez **Capture** sur **Activé**. Définissez la **Fenêtre de temps** (fréquence de capture) et la **Fenêtre de taille** (taille des données à capturer). 
    
    d. Pour **Fournisseur de capture**, sélectionnez **Azure Data Lake Store**, puis sélectionnez le compte Data Lake Storage Gen1 que vous avez créé précédemment. Pour **Chemin de Data Lake**, entrez le nom du dossier que vous avez créé dans le compte Data Lake Storage Gen1. Vous devez uniquement fournir le chemin relatif du dossier.

    e. Pour l’option **Exemples de formats de nom de fichier de capture**, conservez la valeur par défaut. Cette option détermine la structure de dossiers qui est créée sous le dossier de capture.

    f. Cliquez sur **Créer**.

## <a name="test-the-setup"></a>Tester la configuration

Vous pouvez maintenant tester la solution en envoyant des données vers Azure Event Hub. Suivez les instructions de la rubrique [Envoyer des événements vers les hubs d’événements Azure avec .NET Framework](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md). Une fois que vous démarrez l’envoi des données, vous pouvez voir les données dans Data Lake Storage Gen1 avec la structure de dossiers que vous avez spécifiée. Vous pouvez voir une structure de dossiers, telle que celle de la capture d’écran ci-dessous, dans votre compte Data Lake Storage Gen1.

![Exemple de données EventHub dans Data Lake Storage Gen1](./media/data-lake-store-archive-eventhub-capture/data-lake-store-eventhub-data-sample.png "Exemple de données EventHub dans Data Lake Storage Gen1")

> [!NOTE]
> Même si aucun message n’entre dans Event Hubs, celui-ci écrit des fichiers vides avec en-tête dans le compte Data Lake Storage Gen1. Les fichiers sont écrits au même intervalle de temps que celui que vous avez configuré lors de la création des Event Hubs.
> 
>

## <a name="analyze-data-in-data-lake-storage-gen1"></a>Sécuriser les données dans Data Lake Storage Gen1

Une fois les données dans Data Lake Storage Gen1, vous pouvez exécuter des tâches analytiques pour traiter et analyser les données. Pour plus d’informations sur cette procédure à l’aide d’Azure Data Lake Analytics, consultez [Exemple Avro USQL](https://github.com/Azure/usql/tree/master/Examples/AvroExamples).
  

## <a name="see-also"></a>Voir aussi
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Copier des données d’objets blob Stockage Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md)
