---
title: Enregistrer des recherches et épingler des ressources de données dans Azure Data Catalog
description: Article de procédure abordant les fonctionnalités d’Azure Data Catalog permettant d’enregistrer des sources de données et des ressources de données en vue d’une utilisation ultérieure.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 2d0baed53441893ec294784f0f8092fe89a347c9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86523704"
---
# <a name="save-searches-and-pin-data-assets-in-azure-data-catalog"></a>Enregistrer des recherches et épingler des ressources de données dans Azure Data Catalog
## <a name="introduction"></a>Introduction
Azure Data Catalog fournit des fonctionnalités de détection des sources de données. Vous pouvez rapidement rechercher et filtrer le catalogue pour localiser des sources de données et comprendre leur rôle respectif. Vous pouvez ainsi trouver plus facilement les données dont vous avez besoin pour le travail en cours.

Mais qu’en est-il lorsque vous devez utiliser régulièrement les mêmes données ? Ou lorsque les utilisateurs mettent régulièrement à jour une même source de données du catalogue ? Dans de telles situations, le fait d’effectuer toujours les mêmes recherches peut représenter une perte de temps. C’est là que l’enregistrement des recherches et l’épinglage des ressources de données peuvent vous aider.

## <a name="saved-searches"></a>Recherches enregistrées
Dans Data Catalog, une recherche enregistrée est une définition de recherche réutilisable spécifique à un utilisateur. Vous pouvez définir une recherche (termes, balises et autres filtres), puis l’enregistrer. Ainsi, vous pourrez la réexécuter plus tard pour retourner les ressources de données qui correspondent aux critères de recherche.

### <a name="create-a-saved-search"></a>Créer une recherche enregistrée
Pour créer une recherche enregistrée, effectuez les étapes suivantes :
1. Dans le portail Azure Data Catalog, dans la fenêtre **Recherche actuelle**, cliquez sur **Enregistrer**. 

    ![Bouton Enregistrer dans les paramètres Recherche actuelle](./media/data-catalog-how-to-save-pin/01-save-option.png) 

2. Entrez les critères de recherche que vous souhaitez réutiliser, puis cliquez sur **Enregistrer**.

    ![Attribution d’un nom à la recherche enregistrée dans les paramètres Recherche actuelle](./media/data-catalog-how-to-save-pin/02-name.png)

3. Lorsque vous y êtes invité, donnez un nom à la recherche enregistrée. Choisissez un nom significatif décrivant bien les ressources de données qui seront retournées par la recherche.

### <a name="manage-saved-searches"></a>Gérer les recherches enregistrées
Lorsque vous enregistrez une recherche, l’option **Recherches enregistrées** s’affiche sous la zone de texte **Recherche actuelle**. Lorsque la liste est développée, toutes les recherches enregistrées sont affichées.

 ![Data Catalog - Liste des recherches enregistrées](./media/data-catalog-how-to-save-pin/03-list.png)

Faites ce qui suit :

* Pour exécuter une recherche, sélectionnez une recherche enregistrée dans la liste.

* Pour afficher la liste des options de gestion d’une recherche enregistrée, cliquez sur la flèche bas en regard du nom de la recherche.

    ![Options de gestion des recherches enregistrées](./media/data-catalog-how-to-save-pin/04-managing.png)

* Pour attribuer un nouveau nom à une recherche enregistrée, sélectionnez **Renommer**. La définition de la recherche reste inchangée.

* Pour supprimer la recherche enregistrée de votre liste, sélectionnez **Supprimer**, puis confirmez la suppression.

* Pour marquer la recherche enregistrée comme recherche par défaut, sélectionnez **Enregistrer par défaut**. Si l’utilisateur effectue une recherche « vide » dans la page d’accueil Azure Data Catalog, la recherche par défaut est exécutée. En outre, la recherche marquée comme recherche par défaut s’affiche en haut de la liste **Recherches enregistrées**.

### <a name="organizational-saved-searches"></a>Recherches enregistrées organisationnelles
Tous les utilisateurs de votre organisation peuvent enregistrer des recherches pour leur propre usage. Les administrateurs de Data Catalog peuvent également enregistrer des recherches à destination de tous les utilisateurs de l’organisation. Lorsque les administrateurs enregistrent une recherche, l’option **Partager au sein de l’entreprise** leur est proposée. Cette option permet de partager la recherche enregistrée avec tous les utilisateurs de l’organisation.

 ![Data Catalog - Recherches enregistrées organisationnelles](./media/data-catalog-how-to-save-pin/08-organizational-saved-search.png)

## <a name="pinned-data-assets"></a>Ressources de données épinglées
Les recherches enregistrées vous permettent d’enregistrer et de réutiliser les définitions de recherche. Les ressources de données qui sont retournées par la recherche changent en fonction des modifications apportées au catalogue. L’épinglage des ressources de données vous permet d’identifier de manière explicite certaines ressources de données afin de faciliter leur accès, sans avoir besoin d’utiliser une recherche.

L’épinglage d’une ressource de données est simple. Pour ajouter une ressource de données à votre liste de ressources épinglées, il vous suffit de cliquer sur l’icône représentant une **épingle**. Cette icône s’affiche dans l’angle de la vignette de la ressource, dans l’affichage en mosaïque, ainsi que dans la colonne la plus à gauche de la liste, dans le portail Azure Data Catalog.

![Data Catalog - Icône permettant d’épingler une ressource de données](./media/data-catalog-how-to-save-pin/05-pinning.png)

Le désépinglage d’une ressource de données est tout aussi simple. Il vous suffit de recliquer sur l’icône représentant une **épingle** pour désépingler la ressource.

![Data Catalog - Icône permettant de désépingler une ressource de données](./media/data-catalog-how-to-save-pin/06-unpinning.png)

## <a name="the-my-assets-section"></a>Section Mes ressources
La page d’accueil du portail Data Catalog comprend une section intitulée **Mes ressources** qui affiche les actifs d’intérêt pour l’utilisateur actuel. Cette section inclut à la fois les ressources épinglées et les recherches enregistrées.

![Section Mes actifs dans la page d’accueil](./media/data-catalog-how-to-save-pin/07-my-assets.png)

## <a name="summary"></a>Résumé
Azure Data Catalog offre aux utilisateurs des fonctionnalités qui leur permettent de mieux détecter les sources de données dont ils ont besoin, et ainsi de passer moins de temps à rechercher les données et plus de temps à les utiliser. Les recherches enregistrées et l’épinglage des ressources de données viennent s’ajouter à ces fonctionnalités pour que les utilisateurs puissent identifier facilement les sources de données qu’ils utilisent le plus souvent.
