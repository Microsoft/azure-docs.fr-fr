---
title: Bien démarrer avec Azure Data Lake Storage Gen1 - Portail
description: Utiliser le portail Azure pour créer un compte Data Lake Storage Gen1 et effectuer des opérations de base dans ce compte.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 06/27/2018
ms.author: twooley
ms.openlocfilehash: 428ed96c3223e644b0c78712723231a5fabbdc77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578578"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-the-azure-portal"></a>Bien démarrer avec Azure Data Lake Storage Gen1 à l’aide du Portail Azure

> [!div class="op_single_selector"]
> * [Portail](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
>

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Découvrez comment utiliser le portail Azure pour créer un compte Data Lake Storage Gen1 et effectuer des opérations de base telles que la création de dossiers, le chargement et le téléchargement de fichiers de données, la suppression de votre compte, etc. Pour plus d’informations, consultez [Présentation d’Azure Data Lake Storage Gen1](data-lake-store-overview.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="create-a-data-lake-storage-gen1-account"></a>Créer un compte Data Lake Storage Gen1

1. Inscrivez-vous au nouveau [portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource > Stockage > Data Lake Storage Gen1**.
3. Dans le panneau **Nouveau magasin Data Lake Storage Gen1**, fournissez les valeurs indiquées dans la capture d’écran suivante :

    ![Créer un compte Data Lake Storage Gen1](./media/data-lake-store-get-started-portal/ADL.Create.New.Account.png "Créer un compte Data Lake Storage Gen1")

   * **Nom**. Entrez un nom unique pour le compte Data Lake Storage Gen1.
   * **Abonnement**. Sélectionnez l’abonnement sous lequel vous souhaitez créer un compte Data Lake Storage Gen1.
   * **Groupe de ressources**. Sélectionnez un groupe de ressources existant ou sélectionnez l’option **Créer** pour en créer un. Un groupe de ressources est un conteneur réunissant les ressources associées d’une application. Pour plus d’informations, consultez [Groupes de ressources dans Azure](../azure-resource-manager/management/overview.md#resource-groups).
   * **Emplacement** : sélectionnez l'emplacement où vous souhaitez créer le compte Data Lake Storage Gen1.
   * **Paramètres de chiffrement**. Vous disposez de trois options :

     * **Ne pas activer le chiffrement**.
     * **Utiliser les clés gérées par Data Storage Gen1**, si vous souhaitez que Data Lake Storage Gen1 gère vos clés de chiffrement.
     * **Utiliser les clés de votre propre coffre de clés**. Vous pouvez sélectionner un coffre Azure Key Vault existant ou en créer un. Pour utiliser les clés d’un coffre de clés, vous devez attribuer des autorisations permettant au compte Data Lake Storage Gen1 d’accéder à Azure Key Vault. Pour obtenir les instructions correspondantes, reportez-vous à la section [Attribuer des autorisations à Azure Key Vault](#assign-permissions-to-azure-key-vault).

        ![Capture d’écran du panneau Nouveau compte Data Lake Storage Gen1 et du panneau Paramètres de chiffrement.](./media/data-lake-store-get-started-portal/adls-encryption-2.png "Chiffrement Data Lake Storage Gen1")

        Cliquez sur **OK** dans le panneau **Paramètres de chiffrement**.

        Pour plus d’informations, consultez [Chiffrement des données dans Azure Data Lake Storage Gen1](./data-lake-store-encryption.md).

4. Cliquez sur **Créer**. Si vous choisissez d’épingler le compte au tableau de bord, vous serez renvoyé au tableau de bord et vous pourrez voir la progression du provisionnement de votre compte Data Lake Storage Gen1. Une fois le compte Data Lake Storage Gen1 provisionné, le panneau du compte s’affiche.

## <a name="assign-permissions-to-azure-key-vault"></a><a name="assign-permissions-to-azure-key-vault"></a>Attribuer des autorisations à Azure Key Vault

Si vous avez utilisé des clés d’un coffre de clés Azure pour configurer le chiffrement sur le compte Data Lake Storage Gen1, vous devez configurer l’accès entre le compte Data Lake Storage Gen1 et le compte Azure Key Vault. Pour ce faire, procédez comme suit.

1. Si vous avez utilisé des clés du coffre de clés Azure, un avertissement s’affiche en haut du panneau du compte Data Lake Storage Gen1. Cliquez sur l’avertissement pour ouvrir la zone **Chiffrement**.

    ![Capture d’écran du panneau Compte Data Lake Storage Gen1 montrant l’avertissement indiquant « Vous devez configurer les autorisations du coffre de clés. Cliquez ici pour les configurer. »](./media/data-lake-store-get-started-portal/adls-encryption-3.png "Chiffrement Data Lake Storage Gen1")
2. Le panneau affiche deux options possibles pour configurer l’accès.

    ![Capture d’écran du panneau Chiffrement.](./media/data-lake-store-get-started-portal/adls-encryption-4.png "Chiffrement Data Lake Storage Gen1")

   * Dans la première option, cliquez sur **Accorder des autorisations** pour configurer l’accès. La première option est activée uniquement quand l’utilisateur qui a créé le compte Data Lake Storage Gen1 est également administrateur du coffre Azure Key Vault.
   * L’autre option consiste à exécuter l’applet de commande PowerShell qui s’affiche dans le panneau. Vous devez être propriétaire de coffre Azure Key Vault ou avoir le droit d’accorder des autorisations sur le coffre Azure Key Vault. Après avoir exécuté l’applet de commande, revenez au panneau et cliquez sur **Activer** pour configurer l’accès.

> [!NOTE]
> Vous pouvez également créer un compte Data Lake Storage Gen1 à l’aide de modèles Azure Resource Manager. Ces modèles sont accessibles à partir de la page [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?term=data+lake+store) :
> * Sans chiffrement des données : [Déployer un compte Azure Data Lake Storage Gen1 sans chiffrement des données](https://azure.microsoft.com/resources/templates/101-data-lake-store-no-encryption/).
> * Avec chiffrement des données en utilisant Data Lake Storage Gen1 : [Déployer un compte Data Lake Storage Gen1 sans chiffrement des données (Data Lake)](https://azure.microsoft.com/resources/templates/101-data-lake-store-encryption-adls/).
> * Avec chiffrement des données en utilisant Azure Key Vault : [Déployer un compte Data Lake Storage Gen1 sans chiffrement des données (Key Vault)](https://azure.microsoft.com/resources/templates/101-data-lake-store-encryption-key-vault/).
>
>

## <a name="create-folders"></a><a name="createfolder"></a>Créer des dossiers

Vous pouvez créer des dossiers dans votre compte Data Lake Storage Gen1 pour gérer et stocker des données.

1. Ouvrez le compte Data Lake Storage Gen1 que vous avez créé. Dans le volet gauche, cliquez sur **Toutes les ressources**, puis dans le volet **Toutes les ressources**, cliquez sur le nom du compte sous lequel vous souhaitez créer des dossiers. Si vous avez épinglé le compte au tableau d'accueil, cliquez sur la vignette de ce compte.
2. Dans le panneau de votre compte Data Lake Storage Gen1, cliquez sur **Explorateur de données**.

    ![Capture d’écran du panneau Compte Data Lake Storage Gen1 avec l’option Explorateur de données entourée.](./media/data-lake-store-get-started-portal/ADL.Create.Folder.png "Créer des dossiers dans un compte Data Lake Storage Gen1")
3. Dans le panneau Explorateur de données, cliquez sur **Nouveau dossier**, saisissez un nom pour le nouveau dossier, puis cliquez sur **OK**.

    ![Capture d’écran du panneau Explorateur de données avec l’option Nouveau dossier et la zone de texte Créer un dossier entourées.](./media/data-lake-store-get-started-portal/ADL.Folder.Name.png "Créer des dossiers dans un compte Data Lake Storage Gen1")

    Le dossier que vous venez de créer apparaît dans le panneau **Explorateur de données**. Vous pouvez créer des dossiers imbriqués jusqu'au niveau de votre choix.

    ![Créer des dossiers dans un compte Data Lake](./media/data-lake-store-get-started-portal/ADL.New.Directory.png "Créer des dossiers dans un compte Data Lake")

## <a name="upload-data"></a><a name="uploaddata"></a>Charger des données

Vous pouvez charger vos données sur un compte Data Lake Storage Gen1 directement à la racine ou dans un dossier que vous avez créé dans le compte.

1. À partir du panneau **Explorateur de données**, cliquez sur **Télécharger**.
2. Dans le panneau **Charger des fichiers**, accédez aux fichiers à charger, puis cliquez sur **Ajouter les fichiers sélectionnés**. Vous pouvez également sélectionner plusieurs fichiers à charger.

    ![Charger des données](./media/data-lake-store-get-started-portal/ADL.New.Upload.File.png "Télécharger des données")

Si vous recherchez des exemples de données à charger, vous pouvez récupérer le dossier **Données Ambulance** dans le [Référentiel Git Azure Data Lake](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData).

## <a name="actions-available-on-the-stored-data"></a><a name="properties"></a>Actions disponibles sur les données stockées

Cliquez sur l’icône représentant des points de suspension en regard d’un fichier et, dans le menu contextuel, cliquez sur l’action que vous souhaitez exécuter sur les données.

![Propriétés des données](./media/data-lake-store-get-started-portal/ADL.File.Properties.png "Propriétés des données")

## <a name="secure-your-data"></a>Sécurisez vos données

Vous pouvez sécuriser les données stockées dans votre compte Data Lake Storage Gen1 à l’aide d’Azure Active Directory et de listes de contrôle d’accès (ACL). Pour savoir comment procéder, consultez [Sécurisation des données dans Azure Data Lake Storage Gen1](data-lake-store-secure-data.md).

## <a name="delete-your-account"></a>Supprimer votre compte

Pour supprimer un compte Data Lake Storage Gen1, dans votre panneau Data Lake Storage Gen1, cliquez sur **Supprimer**. Pour confirmer l'action, vous devrez entrer le nom du compte que vous souhaitez supprimer. Entrez le nom du compte, puis cliquez sur **Supprimer**.

![Supprimer un compte Data Lake Storage Gen1](./media/data-lake-store-get-started-portal/ADL.Delete.Account.png "Supprimer un compte Data Lake")

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser Azure Data Lake Storage Gen1 pour le Big Data](data-lake-store-data-scenarios.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
