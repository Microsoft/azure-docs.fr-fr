---
title: Accéder aux fichiers et les gérer dans Microsoft OneDrive
description: Charger et gérer des fichiers dans OneDrive en créant des flux de travail automatisés dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/18/2016
tags: connectors
ms.openlocfilehash: 9fe4988b5499943f4b11ec5d640209ceb68e84ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87040250"
---
# <a name="access-and-manage-files-in-onedrive-connector-by-using-azure-logic-apps"></a>Accéder aux fichiers et les gérer dans le connecteur OneDrive à l’aide d'Azure Logic Apps

[Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le connecteur [OneDrive](/connectors/onedriveconnector/) vous permettent de créer des tâches et des flux de travail automatisés afin de gérer vos fichiers, notamment de charger, d'obtenir, de supprimer des fichiers et bien plus. Avec OneDrive, vous pouvez effectuer ces tâches :

* Créer votre flux de travail en stockant des fichiers dans OneDrive, ou mettre à jour des fichiers existants dans OneDrive. 
* Utiliser des déclencheurs pour lancer votre flux de travail lorsqu’un fichier est créé ou mis à jour dans votre OneDrive.
* Utiliser des actions pour créer un fichier, supprimer un fichier et bien plus encore. Par exemple, lorsqu’un nouveau courrier électronique Office 365 est reçu avec une pièce jointe (déclencheur), créer un nouveau fichier dans OneDrive (action).

Cette rubrique décrit comment utiliser le connecteur OneDrive dans une application logique et répertorie les déclencheurs et les actions.

Pour plus d’informations sur Logic Apps, voir [Qu’est-ce qu’une application logique ?](../logic-apps/logic-apps-overview.md) et [Créer une application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="connect-to-onedrive"></a>Connexion à OneDrive

Pour que votre application logique puisse accéder à un service, vous devez d’abord créer une *connexion* à celui-ci. Une connexion permet d’assurer la connectivité entre une application logique et un autre service. Par exemple, pour vous connecter à OneDrive, vous devez préalablement disposer d’une *connexion* OneDrive. Pour créer une connexion, entrez les informations d’identification que vous utilisez généralement pour accéder au service auquel vous souhaitez vous connecter. Ensuite, dans OneDrive, entrez les informations d’identification de votre compte OneDrive pour créer la connexion.

### <a name="create-the-connection"></a>Créer la connexion

[!INCLUDE [Steps to create a connection to OneDrive](../../includes/connectors-create-api-onedrive.md)]

## <a name="use-a-trigger"></a>Utilisation d’un déclencheur

Un déclencheur est un événement qui peut être utilisé pour lancer le flux de travail défini dans une application logique. Les déclencheurs « interrogent » le service à l’intervalle et à la fréquence de votre choix. [En savoir plus sur les déclencheurs](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. Dans le Concepteur d'application logique, saisissez `onedrive` pour obtenir la liste des déclencheurs :  

   ![Une boîte de dialogue intitulée « Afficher les API gérées par Microsoft » comporte une zone qui contient « onedrive ». Vous trouverez ci-dessous une liste de quatre déclencheurs. Le premier est « OneDrive – Quand un fichier est créé ». Le second, « OneDrive – Quand un fichier est modifié », a été sélectionné.](./media/connectors-create-api-onedrive/onedrive-1.png)

2. Sélectionnez **Quand un fichier est modifié**. Si une connexion existe déjà, sélectionnez le bouton Afficher le sélecteur pour sélectionner un dossier.

   ![Une boîte de dialogue intitulée « Quand un fichier est modifié » contient une zone intitulée « DOSSIER » avec un bouton Parcourir associé.](./media/connectors-create-api-onedrive/sample-folder.png)

   Si vous êtes invité à vous connecter, entrez les informations de connexion pour créer la connexion. [Créer la connexion](connectors-create-api-onedrive.md#create-the-connection) dans cet article répertorie les étapes.

   Dans cet exemple, l’application logique s’exécute lorsqu’un fichier est mis à jour dans le dossier que vous avez choisi. Pour consulter les résultats de ce déclencheur, ajoutez une autre action qui vous envoie un courrier électronique. Par exemple, ajoutez l’action Office 365 Outlook *Envoyer un courrier électronique* qui vous avertit par e-mail quand un fichier est mis à jour.

3. Sélectionnez le bouton **Modifier**, puis renseignez les valeurs **Fréquence** et **Intervalle**. Par exemple, si vous souhaitez que le déclencheur interroge le service toutes les 15 minutes, définissez le champ **Fréquence** sur **Minute**, et le champ **Intervalle** sur **15**. 

   ![Une boîte de dialogue intitulée « Quand un fichier est modifié » présente cinq cases nommées : « DOSSIER », « FRÉQUENCE », « INTERVALLE », « FUSEAU HORAIRE » et « HEURE DE DÉBUT ». Il existe des listes déroulantes pour les champs « FRÉQUENCE » et « FUSEAU HORAIRE ».](./media/connectors-create-api-onedrive/trigger-properties.png)

4. **Enregistrez** vos modifications (dans le coin supérieur gauche de la barre d’outils). Votre application logique est enregistrée et peut être activée automatiquement.

## <a name="use-an-action"></a>Utilisation d’une action

Une action est une opération effectuée par le flux de travail défini dans une application logique. [En savoir plus sur les actions](../logic-apps/logic-apps-overview.md#logic-app-concepts).

1. Sélectionnez le signe plus. Vous disposez de plusieurs options : **Ajouter une action**, **Ajouter une condition** ou l’une des options **Plus**.

   ![Une capture d’écran illustre quatre boutons : « + Nouvelle étape », « Ajouter une action », « Ajouter une condition » et « … Plus ».](./media/connectors-create-api-onedrive/add-action.png)

2. Choisissez **Ajouter une action**.

3. Dans la zone de recherche, saisissez `onedrive` pour obtenir la liste de toutes les actions disponibles.

   ![Une boîte de dialogue intitulée « Afficher les API gérées par Microsoft » comporte une zone qui contient « onedrive ». Vous trouverez ci-dessous une liste de huit actions. La première est « OneDrive – Créer un fichier » et elle est sélectionnée.](./media/connectors-create-api-onedrive/onedrive-actions.png) 

4. Dans notre exemple, choisissez **OneDrive - Créer un fichier**. Si une connexion existe déjà, sélectionnez le **chemin du dossier** où placer le fichier, entrez le **nom de fichier** et choisissez le **contenu du fichier** souhaité :  

   ![Une boîte de dialogue intitulée « Créer un fichier » affiche trois zones nommées « CHEMIN DU DOSSIER », « NOM DE FICHIER » et « CONTENU DU DOSSIER ». Il y a un bouton Parcourir de répertoire à côté de la zone « CHEMIN DU DOSSIER ».](./media/connectors-create-api-onedrive/sample-action.png)

   Si vous êtes invité à indiquer les informations de connexion, entrez les informations requises pour [créer la connexion comme décrit](#create-the-connection) dans cette rubrique.

   Dans cet exemple, vous créez un fichier dans un dossier OneDrive. Vous pouvez utiliser les résultats d’un autre déclencheur pour créer le fichier OneDrive. Par exemple, ajoutez le déclencheur Outlook Office 365 *Lorsqu’un nouveau courrier électronique arrive*. Puis ajoutez l’action OneDrive *Créer un fichier* qui utilise les champs Pièces jointes et Type de contenu d’une instruction ForEach pour créer le fichier dans OneDrive.

   ![Une boîte de dialogue intitulée « Pour chaque » comporte une zone nommée « SÉLECTIONNER UNE SORTIE PARMI LES ÉTAPES PRÉCÉDENTES » qui contient « Pièces jointes ». La boîte de dialogue « Créer un fichier » recouvre le reste de la zone « Pour chaque », avec les cases intitulées « CHEMIN DU DOSSIER », « NOM DE FICHIER » et « CONTENU DU FICHIER ». ](./media/connectors-create-api-onedrive/foreach-action.png)

5. **Enregistrez** vos modifications (dans le coin supérieur gauche de la barre d’outils). Votre application logique est enregistrée et peut être activée automatiquement.

## <a name="connector-specific-details"></a>Détails spécifiques du connecteur

Consultez l’ensemble des déclencheurs et actions définis dans le swagger, ainsi que les éventuelles limites dans les [détails des connecteurs](/connectors/onedriveconnector/).

## <a name="next-steps"></a>Étapes suivantes

* [Connecteurs pour Azure Logic Apps](apis-list.md)