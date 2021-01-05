---
title: Valider des données XML avec des schémas
description: Ajouter des schémas pour valider les documents XML dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 02/06/2019
ms.openlocfilehash: ade29318fdd1510d03ddd208534258a30b256e98
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75979374"
---
# <a name="validate-xml-with-schemas-in-azure-logic-apps-with-enterprise-integration-pack"></a>Valider des documents XML avec des schémas dans Azure Logic Apps avec Enterprise Integration Pack

Pour vérifier que les documents utilisent du code XML valide et qu’ils contiennent les données attendues au format prédéfini pour les scénarios d’intégration entreprise dans Azure Logic Apps, votre application logique peut utiliser des schémas. Un schéma peut également valider des messages que les applications logiques échangent dans les scénarios B2B.

Pour connaître les limites associées aux comptes d’intégration et aux artefacts tels que les schémas, consultez [Limites et informations de configuration pour Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez <a href="https://azure.microsoft.com/free/" target="_blank">vous inscrire pour obtenir un compte Azure gratuitement</a>.

* [Compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) où vous stockez vos schémas et autres artefacts pour les solutions d’intégration d’entreprise et les solutions B2B. 

  Si la taille de votre schéma est [inférieure ou égale à 2 Mo](#smaller-schema), vous pouvez ajouter votre schéma à votre compte d’intégration directement dans le portail Azure. Toutefois, si la taille de votre schéma est supérieure à 2 Mo, mais qu’elle ne dépasse pas la [taille limite des schémas](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits), vous pouvez charger votre schéma dans un compte de stockage Azure. 
  Pour ajouter ce schéma à votre compte d’intégration, vous pouvez associer votre compte de stockage à votre compte d’intégration. 
  Pour cette tâche, voici les éléments dont vous avez besoin : 

  * Un [compte de stockage Azure](../storage/common/storage-account-overview.md) dans lequel vous créez un conteneur d’objets blob pour votre schéma. Découvrez comment [créer un compte de stockage](../storage/common/storage-account-create.md). 

  * Un conteneur d’objets blob pour stocker votre schéma. Découvrez comment [créer un conteneur d’objets blob](../storage/blobs/storage-quickstart-blobs-portal.md). 
  Vous aurez besoin de l’URI du contenu de votre conteneur plus tard, lorsque vous ajouterez le schéma à votre compte d’intégration.

  * L’[Explorateur Stockage Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md), que vous pouvez utiliser pour gérer les comptes de stockage et les conteneurs d’objets blob. 
  Pour utiliser l’Explorateur Stockage, choisissez l’une de ces options :
  
    * Dans le portail Azure, sélectionnez votre compte de stockage. 
    Dans le menu de votre compte de stockage, sélectionnez **Explorateur Stockage**.

    * Pour la version de bureau, [téléchargez et installez l’Explorateur Stockage Azure](https://www.storageexplorer.com/). 
    Ensuite, connectez l’Explorateur Stockage à votre compte de stockage en suivant les étapes décrites dans [Prise en main de l’Explorateur Stockage](../vs-azure-tools-storage-manage-with-storage-explorer.md). 
    Pour en savoir plus, consultez [Démarrage rapide : Créer un objet blob dans le stockage d’objets avec l’Explorateur Stockage Azure](../storage/blobs/storage-quickstart-blobs-storage-explorer.md).

Vous n’avez pas besoin d’une application logique pour créer et ajouter des schémas. Toutefois, pour utiliser un schéma, votre application logique doit être associée au compte d’intégration dans lequel vous stockez ce schéma. Découvrez comment [lier des applications logiques à des comptes d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account). Si vous n’avez pas encore d’application logique, découvrez [comment créer des applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-schemas"></a>Ajouter des schémas

1. Connectez-vous au <a href="https://portal.azure.com" target="_blank">portail Azure</a> avec les informations d’identification de votre compte Azure.

1. Pour rechercher et ouvrir votre compte d’intégration, dans le menu Azure principal, sélectionnez **Tous les services**. Dans la zone de recherche, entrez « compte d’intégration ». Sélectionnez **Comptes d’intégration**.

   ![Chercher le compte d’intégration](./media/logic-apps-enterprise-integration-schemas/find-integration-account.png)

1. Sélectionnez le compte d’intégration dans lequel vous souhaitez ajouter votre schéma, par exemple :

   ![Sélectionner le compte d’intégration](./media/logic-apps-enterprise-integration-schemas/select-integration-account.png)

1. Dans la page **Vue d’ensemble** de votre compte d’intégration, sous **Composants**, sélectionnez la vignette **Schémas**.

   ![Sélectionner « Schémas »](./media/logic-apps-enterprise-integration-schemas/select-schemas.png)

1. Une fois que la page **Schémas** est ouverte, choisissez **Ajouter**.

   ![Choisir « Ajouter »](./media/logic-apps-enterprise-integration-schemas/add-schema.png)

En fonction de la taille de votre schéma (.xsd), suivez les étapes de chargement pour les schémas de taille [inférieure ou égale à 2 Mo](#smaller-schema) ou pour les schémas dont la taille est comprise entre [2 Mo et 8 Mo](#larger-schema).

<a name="smaller-schema"></a>

### <a name="add-schemas-up-to-2-mb"></a>Ajouter des schémas de taille inférieure ou égale à 2 Mo

1. Sous **Ajouter un schéma**, saisissez un nom pour votre schéma. 
   L’option **Fichier de petite taille** doit rester cochée. À côté de la zone **Schéma**, choisissez l’icône de dossier. Recherchez puis sélectionnez le schéma à charger, par exemple :

   ![Charger un schéma plus petit](./media/logic-apps-enterprise-integration-schemas/upload-smaller-schema-file.png)

1. Lorsque vous êtes prêt, choisissez **OK**.

   Lorsque le chargement de votre schéma est terminé, celui-ci s’affiche dans la liste **Schémas**.

<a name="larger-schema"></a>

### <a name="add-schemas-more-than-2-mb"></a>Ajouter des schémas de taille supérieure à 2 Mo

Pour ajouter des schémas plus volumineux, vous pouvez charger votre schéma dans un conteneur d’objets blob Azure de votre compte de stockage Azure. Les étapes pour ajouter des schémas varient selon que votre conteneur d’objets blob dispose ou non d’un accès en lecture public. Effectuez donc les étapes suivantes pour vérifier si votre conteneur d’objets blob dispose d’un accès en lecture public : [Définir le niveau d’accès public pour le conteneur d’objets blob](../vs-azure-tools-storage-explorer-blobs.md#set-the-public-access-level-for-a-blob-container)

#### <a name="check-container-access-level"></a>Vérifier le niveau d’accès du conteneur

1. Ouvrez l’Explorateur de stockage Azure. Dans la fenêtre de l’Explorateur, développez votre abonnement Azure, si ce n’est déjà fait.

1. Développez **Comptes de stockage** > {*votre compte de stockage*} > **Conteneurs d’objets blob**. Sélectionnez votre conteneur blob.

1. Dans le menu contextuel de votre conteneur d’objets blob, sélectionnez **Définir le niveau d’accès public**.

   * Si votre conteneur d’objets blob dispose au minimum d’un accès public, choisissez **Annuler**, puis suivez les étapes fournies plus loin dans cette page : [Charger dans les conteneurs avec accès public](#public-access)

     ![Accès public](media/logic-apps-enterprise-integration-schemas/azure-blob-container-public-access.png)

   * Si votre conteneur d’objets blob ne dispose pas d’un accès public, choisissez **Annuler**, puis suivez les étapes fournies plus loin dans cette page : [Charger dans les conteneurs sans accès public](#public-access)

     ![Aucun accès public](media/logic-apps-enterprise-integration-schemas/azure-blob-container-no-public-access.png)

<a name="public-access"></a>

#### <a name="upload-to-containers-with-public-access"></a>Charger dans les conteneurs avec accès public

1. Téléchargez le schéma vers votre compte de stockage. 
   Dans la fenêtre de droite, choisissez **Charger**.

1. Une fois son chargement terminé, sélectionnez votre schéma. Dans la barre d’outils, choisissez **Copier l’URL** pour copier l’URL du schéma.

1. Retournez au portail Azure où le volet **Ajouter un schéma** est ouvert. 
   Entrez un nom pour votre assembly. 
   Choisissez **Fichier de grande taille (plus de 2 Mo)** . 

   La zone **URI de contenu** s’affiche à la place de la zone **Schéma**.

1. Dans la zone **URI de contenu**, collez l’URL de votre schéma. 
   Terminez l’ajout de votre schéma.

Lorsque le chargement de votre schéma est terminé, celui-ci s’affiche dans la liste **Schémas**. Dans la page **Vue d’ensemble** de votre compte d’intégration, sous **Composants**, la vignette **Schémas** doit afficher le nombre de schémas chargés.

<a name="no-public-access"></a>

#### <a name="upload-to-containers-without-public-access"></a>Charger dans les conteneurs sans accès public

1. Téléchargez le schéma vers votre compte de stockage. 
   Dans la fenêtre de droite, choisissez **Charger**.

1. Une fois le chargement terminé, générez une signature d’accès partagé (SAP) pour votre schéma. 
   Dans le menu contextuel de votre schéma, sélectionnez **Obtenir une signature d’accès partagé**.

1. Dans le volet **Signature d’accès partagé**, sélectionnez **Generate container-level shared access signature URI (Générer un URI de signature d’accès partagé au niveau du conteneur)**  > **Créer**. 
   Une fois que vous avez généré l’URI de signature d’accès partagé, à côté de la zone **URL**, sélectionnez **Copier**.

1. Retournez au portail Azure où le volet **Ajouter un schéma** est ouvert. Choisissez **Fichier de grande taille**.

   La zone **URI de contenu** s’affiche à la place de la zone **Schéma**.

1. Dans la zone **URI de contenu**, collez l’URI de signature d’accès partagé généré précédemment. Terminez l’ajout de votre schéma.

Lorsque le chargement de votre schéma est terminé, celui-ci s’affiche dans la liste **Schémas**. Dans la page **Vue d’ensemble** de votre compte d’intégration, sous **Composants**, la vignette **Schémas** doit afficher le nombre de schémas chargés.

## <a name="edit-schemas"></a>Modification de schémas

Pour mettre à jour un schéma existant, vous devez charger un nouveau fichier de schéma qui comporte les modifications souhaitées. Cependant, vous pouvez d’abord télécharger le schéma existant pour le modifier.

1. Dans le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, ouvrez votre compte d’intégration, s’il n’est pas déjà ouvert.

1. Dans le menu principal Azure, sélectionnez **Tous les services**. 
   Dans la zone de recherche, entrez « compte d’intégration ». 
   Sélectionnez **Comptes d’intégration**.

1. Sélectionnez le compte d’intégration dans lequel vous souhaitez mettre à jour le schéma.

1. Dans la page **Vue d’ensemble** de votre compte d’intégration, sous **Composants**, sélectionnez la vignette **Schémas**.

1. Une fois que la page **Schémas** est ouverte, sélectionnez votre schéma. 
   Pour d’abord télécharger et modifier le schéma, choisissez **Télécharger**, puis enregistrez le schéma.

1. Lorsque vous êtes prêt à charger le schéma mis à jour, dans la page **Schémas**, sélectionnez le schéma que vous souhaitez mettre à jour, puis choisissez **Mettre à jour**.

1. Recherchez puis sélectionnez le schéma mis à jour que vous souhaitez charger. 
   Lorsque le chargement du schéma mis à jour est terminé, celui-ci s’affiche dans la liste **Schémas**.

## <a name="delete-schemas"></a>Suppression de schémas

1. Dans le <a href="https://portal.azure.com" target="_blank">portail Azure</a>, ouvrez votre compte d’intégration, s’il n’est pas déjà ouvert.

1. Dans le menu principal Azure, sélectionnez **Tous les services**. 
   Dans la zone de recherche, entrez « compte d’intégration ». 
   Sélectionnez **Comptes d’intégration**.

1. Sélectionnez le compte d’intégration dans lequel vous souhaitez supprimer le schéma.

1. Dans la page **Vue d’ensemble** de votre compte d’intégration, sous **Composants**, sélectionnez la vignette **Schémas**.

1. Une fois que la page **Schémas** est ouverte, sélectionnez votre schéma, puis choisissez **Supprimer**.

1. Pour confirmer que vous souhaitez supprimer le schéma sélectionné, choisissez **Oui**.

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)
* [En savoir plus sur les mappages](../logic-apps/logic-apps-enterprise-integration-maps.md)
* [En savoir plus sur les transformations](../logic-apps/logic-apps-enterprise-integration-transform.md)
