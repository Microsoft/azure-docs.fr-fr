---
title: Se connecter à un serveur FTP
description: Automatiser des tâches et des flux de travail qui créent, surveillent et gèrent des fichiers sur un serveur FTP à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 12/15/2019
tags: connectors
ms.openlocfilehash: f4cad2b658547d56d00efdd5e1496110f8e4a5e6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999573"
---
# <a name="create-monitor-and-manage-ftp-files-by-using-azure-logic-apps"></a>Créer, superviser et gérer des fichiers FTP avec Azure Logic Apps

Avec Azure Logic Apps et le connecteur FTP, vous pouvez créer des tâches et des flux de travail automatisés qui créent, supervisent, envoient et reçoivent des fichiers via votre compte sur un serveur FTP, ainsi que d’autres actions, par exemple :

* Superviser à quel moment des fichiers sont ajoutés ou changés.
* Obtenir, créer, copier, mettre à jour, lister et supprimer des fichiers.
* Obtenir les métadonnées et le contenu des fichiers.
* Extraire des archives dans des dossiers.

Vous pouvez utiliser des déclencheurs qui obtiennent des réponses de votre serveur FTP, et mettent la sortie à la disposition d’autres actions. Vous pouvez utiliser des actions d’exécution dans vos applications logiques pour la gestion des fichiers sur votre serveur FTP. Vous pouvez également faire en sorte que des actions utilisent la sortie d’actions FTP. Par exemple, si vous obtenez régulièrement des fichiers à partir de votre serveur FTP, vous pouvez envoyer un e-mail au sujet de ces fichiers et de leur contenu en utilisant le connecteur Office 365 Outlook ou le connecteur Outlook.com. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md).

## <a name="limitations"></a>Limites

* Le connecteur FTP ne prend en charge que FTPS (FTP sur TLS/SSL) en mode explicite ; il n’est pas compatible avec le protocole FTPS implicite.

* Par défaut, les actions FTP peuvent lire ou écrire des fichiers dont la taille est *inférieure ou égale à 50 Mo*. Pour gérer les fichiers dont la taille est supérieure à 50 Mo, les actions FTP prennent en charge la [segmentation des messages](../logic-apps/logic-apps-handle-large-messages.md). L’action **Obtenir le contenu du fichier** utilise implicitement la segmentation.

* Les déclencheurs FTP ne prennent pas en charge la segmentation. Quand ils demandent du contenu de fichiers, les déclencheurs sélectionnent uniquement des fichiers dont la taille est inférieure ou égale à 50 Mo. Pour obtenir des fichiers supérieurs à 50 Mo, suivez ce modèle :

  * Utilisez un déclencheur FTP qui retourne des propriétés de fichier comme **Quand un fichier est ajouté ou modifié (propriétés uniquement)** .

  * Suivez le déclencheur avec l’action FTP **Obtenir le contenu du fichier**, qui lit le fichier complet et utilise implicitement la segmentation.

* Si vous disposez d’un serveur FTP local, envisagez de créer un [environnement de service d'intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) ou d’utiliser des [connexions hybrides d’Azure App Service](../app-service/app-service-hybrid-connections.md) ; cela vous permet dans les deux cas d’accéder à des sources de données locales sans utiliser de passerelle de données locale.

## <a name="how-ftp-triggers-work"></a>Fonctionnement des déclencheurs FTP

Les déclencheurs FTP fonctionnent en interrogeant le système de fichiers FTP et en recherchant tout fichier changé depuis la dernière interrogation. Certains outils permettent de conserver l’horodatage lorsque les fichiers sont modifiés. Dans ce cas, vous devez désactiver cette fonctionnalité pour permettre l’exécution du déclencheur. Voici quelques paramètres communs :

| Client SFTP | Action |
|-------------|--------|
| Winscp | Accédez à **Options** > **Préférences** > **Transférer** > **Modifier** > **Conserver l’horodatage** > **Désactiver** |
| FileZilla | Accédez à **Transférer** > **Conserver les horodatages des fichiers transférés** > **Désactiver** |
|||

Quand un déclencheur détecte un nouveau fichier, il vérifie que le nouveau fichier est complet et non partiellement écrit. Par exemple, un fichier peut être en cours de modification lorsque le déclencheur vérifie le serveur de fichiers. Pour éviter de retourner un fichier partiellement écrit, le déclencheur note l’horodatage du fichier qui comporte des modifications récentes, mais ne retourne pas immédiatement ce fichier. Le déclencheur retourne le fichier uniquement lors d’une nouvelle interrogation du serveur. Parfois, ce comportement peut entraîner un retard correspondant à jusqu’à deux fois l’intervalle d’interrogation du déclencheur.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Vos informations d’identification de compte et adresse du serveur hôte FTP

  Le connecteur FTP a besoin que votre serveur FTP soit accessible à partir d’Internet, et configuré pour fonctionner en mode *passif*. Vos informations d’identification permettent à votre application logique de créer une connexion et d’accéder à votre compte FTP.

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* L’application logique à partir de laquelle vous souhaitez accéder à votre compte FTP. Pour démarrer avec un déclencheur FTP, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour utiliser une action FTP, démarrez votre application logique avec un autre déclencheur, par exemple le déclencheur **Périodicité**.

## <a name="connect-to-ftp"></a>Se connecter à FTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [Portail Azure](https://portal.azure.com), puis ouvrez votre application logique dans le concepteur d’application logique.

1. Pour les applications logiques vides, dans la zone de recherche, entrez `ftp` comme filtre. Dans la liste des **déclencheurs**, sélectionnez celui qui vous intéresse.

   -ou-

   Pour les applications logiques existantes, sous la dernière étape où vous souhaitez ajouter une action, sélectionnez **Nouvelle étape**, puis **Ajouter une action**. Dans la zone de recherche, entrez `ftp` en guise de filtre. Dans la liste des **actions**, sélectionnez l’action qui vous intéresse.

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Indiquez vos informations de connexion et sélectionnez **Créer**.

1. Fournissez les informations pour le déclencheur ou l’action sélectionnés et continuez à générer le flux de travail de votre application logique.

## <a name="examples"></a>Exemples

<a name="file-added-modified"></a>

### <a name="add-ftp-trigger"></a>Ajouter un déclencheur FTP

Le déclencheur **Quand un fichier est ajouté ou modifié (propriétés uniquement)** démarre un flux de travail d’application logique quand il détecte qu’un fichier est ajouté ou modifié sur un serveur FTP. Par exemple, vous pouvez ajouter une condition qui vérifie le contenu du fichier pour savoir s’il peut être obtenu. Enfin, vous pouvez ajouter une action qui obtient le contenu du fichier et le place dans un dossier différent sur le serveur SFTP.

Par exemple, vous pouvez utiliser ce déclencheur pour superviser l’apparition dans un dossier FTP de nouveaux fichiers qui décrivent les commandes des clients. Vous pouvez ensuite utiliser une action FTP telle que **Obtenir les métadonnées d’un fichier** pour obtenir les propriétés de ce nouveau fichier, puis utiliser **Obtenir le contenu d’un fichier** pour obtenir le contenu de ce fichier en vue d’un traitement ultérieur et de le stocker dans une base de données de commandes.

Voici un exemple qui montre comment utiliser le déclencheur **Quand un fichier est ajouté ou modifié (propriétés uniquement)** .

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Pour les applications logiques vides, dans la zone de recherche, entrez `ftp` comme filtre. Dans la liste des déclencheurs, sélectionnez ce déclencheur : **Quand un fichier est ajouté ou modifié (propriétés uniquement)**

   ![Rechercher et sélectionner le déclencheur FTP](./media/connectors-create-api-ftp/select-ftp-trigger-logic-app.png)

1. Fournissez les informations nécessaires pour votre connexion, puis sélectionnez **Créer**.

   Par défaut, ce connecteur transfère les fichiers au format texte. Pour transférer des fichiers au format binaire, par exemple quand l’encodage est utilisé, sélectionnez **Transport binaire**.

   ![Créer une connexion au serveur FTP](./media/connectors-create-api-ftp/create-ftp-connection-trigger.png)

1. Dans la zone **Dossier**, sélectionnez l’icône de dossier pour faire apparaître une liste. Afin de trouver le dossier que vous souhaitez superviser pour les fichiers nouveaux ou modifiés, sélectionnez le chevron droit ( **>** ), accédez à ce dossier et sélectionnez-le.

   ![Recherche et sélection du dossier à superviser](./media/connectors-create-api-ftp/select-folder-ftp-trigger.png)

   Votre dossier sélectionné s’affiche dans la zone **Dossier**.

   ![Le dossier sélectionné s’affiche dans la propriété « Dossier ».](./media/connectors-create-api-ftp/selected-folder-ftp-trigger.png)

1. Enregistrez votre application logique. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

À présent que votre application logique dispose d’un déclencheur, ajoutez les actions que vous souhaitez exécuter lorsqu’elle trouve un fichier nouveau ou modifié. Pour cet exemple, vous pouvez ajouter une action FTP qui obtient le contenu nouveau ou mis à jour.

<a name="get-content"></a>

### <a name="add-ftp-action"></a>Ajouter une action FTP

L’action **Obtenir les métadonnées du fichier** obtient les propriétés d’un fichier qui se trouve sur votre serveur FTP et l’action **Obtenir le contenu du fichier** obtient le contenu du fichier en fonction des informations relatives à ce fichier sur votre serveur FTP. Par exemple, vous pouvez ajouter le déclencheur de l’exemple précédent et ces actions pour obtenir le contenu du fichier après l’ajout ou la modification de celui-ci.

1. Sous le déclencheur ou n’importe quel autre action, sélectionnez **Nouvelle étape**.

1. Dans la zone de recherche, entrez `ftp` en guise de filtre. Dans la liste des actions, sélectionnez cette action : **Obtenir les métadonnées d’un fichier**

   ![Sélectionner l’action « Obtenir les métadonnées du fichier »](./media/connectors-create-api-ftp/select-get-file-metadata-ftp-action.png)

1. Si vous disposez déjà d’une connexion à vos compte et serveur FTP, passez à l’étape suivante. Sinon, fournissez les informations nécessaires pour cette connexion, puis sélectionnez **Créer**.

   ![Création d’une connexion serveur FTP](./media/connectors-create-api-ftp/create-ftp-connection-action.png)

1. Après l’apparition de l’action **Obtenir les métadonnées du fichier**, cliquez à l’intérieur de la zone **Fichier** pour que la liste de contenu dynamique s’affiche. Vous pouvez à présent sélectionner des propriétés pour les sorties des étapes précédentes. Dans la liste contenu dynamique, sous **Obtenir les métadonnées d’un fichier**, sélectionnez la propriété **List of Files Id** (Id Liste des fichiers), qui fait référence à la collection dans laquelle le fichier a été ajouté ou mis à jour.

   ![Rechercher et sélectionner la propriété « Id Liste des fichiers »](./media/connectors-create-api-ftp/select-list-of-files-id-output.png)

   La propriété **List of Files Id** (Id Liste de fichiers) apparaît maintenant dans la zone **Fichier**.

   ![Propriété « Id Liste des fichiers » sélectionnée](./media/connectors-create-api-ftp/selected-list-file-ids-ftp-action.png)

1. Ajoutez maintenant cette action FTP : **Obtenir le contenu d’un fichier**

   ![Rechercher et sélectionner l’action « Obtenir le contenu du fichier »](./media/connectors-create-api-ftp/select-get-file-content-ftp-action.png)

1. Après l’apparition de l’action **Obtenir le contenu du fichier**, cliquez à l’intérieur de la zone **Fichier** pour que la liste de contenu dynamique s’affiche. Vous pouvez à présent sélectionner des propriétés pour les sorties des étapes précédentes. Dans la liste contenu dynamique, sous **Obtenir les métadonnées d’un fichier**, sélectionnez la propriété **Id**, qui fait référence au fichier qui a été ajouté ou mis à jour.

   ![Rechercher et sélectionner la propriété « Id »](./media/connectors-create-api-ftp/get-file-content-id-output.png)

   La propriété **Id** apparaît maintenant dans la zone **Fichier**.

   ![Propriété « Id » sélectionnée](./media/connectors-create-api-ftp/selected-get-file-content-id-ftp-action.png)

1. Enregistrez votre application logique.

## <a name="test-your-logic-app"></a>Tester votre application logique

Pour vérifier que votre workflow retourne le contenu attendu, ajoutez une autre action qui vous envoie le contenu à partir du fichier téléchargé ou mis à jour.

1. Sous l’action **Obtenir le contenu du fichier**, ajoutez une action qui peut vous envoyer le contenu du fichier. Cet exemple ajoute l’action **Envoyer un e-mail** pour Office 365 Outlook

   ![Ajouter une action pour l’envoi de courrier](./media/connectors-create-api-ftp/select-send-email-action.png)

1. Une fois que l’action s’affiche, fournissez les informations et incluez les propriétés que vous souhaitez tester. Par exemple, incluez la propriété **Contenu du fichier**, qui apparaît dans la liste de contenu dynamique après avoir sélectionné **Voir plus** dans la section **Obtenir le contenu d’un fichier**.

   ![Fournir des informations sur l’action d’e-mail](./media/connectors-create-api-ftp/selected-send-email-action.png)

1. Enregistrez votre application logique. Pour exécuter et déclencher l’application logique, dans la barre d’outils, sélectionnez **Exécuter**, puis ajoutez un fichier au dossier FTP supervisé par votre application logique.

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/ftpconnector/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)

