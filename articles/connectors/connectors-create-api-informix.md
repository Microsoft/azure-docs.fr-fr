---
title: Se connecter à une base de données IBM Informix
description: Automatisez les tâches et les workflows qui gèrent des ressources stockées dans IBM Informix à l’aide d’Azure Logic Apps.
services: logic-apps
ms.suite: integration
author: gplarsen
ms.author: daberry
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 01/07/2020
tags: connectors
ms.openlocfilehash: 4995a91783c2302f3bda5cc9409f017248ca29fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88761642"
---
# <a name="manage-ibm-informix-database-resources-by-using-azure-logic-apps"></a>Gérer des ressources de base de données IBM Informix à l’aide d’Azure Logic Apps

Avec [Azure Logic Apps](../logic-apps/logic-apps-overview.md) et le [connecteur Informix](/connectors/informix/), vous pouvez créer des tâches et des workflows automatisés qui gèrent des ressources stockées dans une base de données IBM Informix. Ce connecteur inclut un client Microsoft qui communique avec les ordinateurs serveurs Informix distants par le biais d’un réseau TCP/IP, notamment les bases de données cloud telles qu’IBM Informix pour Windows exécutées dans des bases de données locales et de virtualisation Azure quand vous utilisez la [passerelle de données locale](../logic-apps/logic-apps-gateway-connection.md). Vous pouvez vous connecter à ces plateformes et versions Informix si elles sont configurées pour la prise en charge des connexions client DRDA (Distributed Relational Database Architecture) :

* IBM Informix 12.1
* IBM Informix 11.7

Cette rubrique décrit comment utiliser le connecteur dans une application logique pour traiter les opérations de base de données.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Pour les bases de données locales, [téléchargez et installez la passerelle de données locale](../logic-apps/logic-apps-gateway-install.md) sur un ordinateur local, puis [créez une ressource de passerelle de données Azure dans le portail Azure](../logic-apps/logic-apps-gateway-connection.md).

* L’application logique où vous devez avoir accès à votre base de données Informix. Ce connecteur ne fournit que des actions ; par conséquent, votre application logique doit déjà démarrer avec un déclencheur, par exemple le [déclencheur Périodicité](../connectors/connectors-native-recurrence.md). 

## <a name="add-an-informix-action"></a>Ajouter une action Informix

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques, si elle n’est pas encore ouverte.

1. Sous l’étape à laquelle vous souhaitez ajouter l’action Informix, choisissez **Nouvelle étape**.

   Pour ajouter une action entre des étapes, déplacez votre souris sur la flèche de connexion. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Dans la zone de recherche, entrez `informix` en guise de filtre. Dans la liste des actions, sélectionnez celle qui vous intéresse, par exemple :

   ![Sélectionnez l’action Informix à exécuter](./media/connectors-create-api-informix/select-informix-connector-action.png)

   Le connecteur fournit ces actions, qui exécutent les opérations de base de données correspondantes :

   * Obtenir les tables : lister les tables de base de données à l’aide d’une instruction `CALL`
   * Obtenir les lignes : lire toutes les lignes à l’aide d’une instruction `SELECT *`
   * Obtenir la ligne : lire une ligne à l’aide d’une instruction `SELECT WHERE`
   * Ajouter une ligne à l’aide d’une instruction `INSERT`
   * Modifier une ligne à l’aide d’une instruction `UPDATE`
   * Supprimer une ligne à l’aide d’une instruction `DELETE`

1. Si vous êtes invité à fournir des informations de connexion pour votre base de données Informix, suivez les [étapes pour créer la connexion](#create-connection), puis passez à l’étape suivante.

1. Fournissez les informations relatives à l’action sélectionnée :

   | Action | Description | Propriétés et descriptions |
   |--------|-------------|-----------------------------|
   | **Obtenir les tables** | Lister les tables de base de données en exécutant une instruction CALL Informix. | None |
   | **Obtenir les lignes** | Récupérer toutes les lignes de la table spécifiée en exécutant une instruction `SELECT *` Informix. | **Nom de la table** : nom de la table Informix souhaitée <p><p>Pour ajouter d’autres propriétés à cette action, sélectionnez-les dans la liste **Ajouter un nouveau paramètre**. Pour plus d’informations, consultez la [rubrique de référence du connecteur](/connectors/informix/). |
   | **Obtenir la ligne** | Récupérer une ligne de la table spécifiée en exécutant une instruction `SELECT WHERE` Informix. | - **Nom de la table** : nom de la table Informix souhaitée <br>- **ID de ligne** : ID unique de la ligne, par exemple `9999` |
   | **Insérer une ligne** | Ajouter une ligne à la table Informix spécifiée en exécutant une instruction `INSERT` Informix. | - **Nom de la table** : nom de la table Informix souhaitée <br>- **élément** : ligne avec les valeurs à ajouter |
   | **Mettre à jour la ligne** | Modifier une ligne dans la table Informix spécifiée en exécutant une instruction `UPDATE` Informix. | - **Nom de la table** : nom de la table Informix souhaitée <br>- **ID de ligne** : ID unique de la ligne à mettre à jour, par exemple `9999` <br>- **Ligne** : ligne avec les valeurs mises à jour, par exemple `102` |
   | **Supprimer la ligne** | Supprimer une ligne de la table Informix spécifiée en exécutant une instruction `DELETE` Informix. | - **Nom de la table** : nom de la table Informix souhaitée <br>- **ID de ligne** : ID unique de la ligne à supprimer, par exemple `9999` |
   ||||

1. Enregistrez votre application logique. Maintenant, [testez votre application logique](#test-logic-app) ou poursuivez sa génération.

<a name="create-connection"></a>

## <a name="connect-to-informix"></a>Se connecter à Informix

1. Si votre application logique se connecte à une base de données locale, sélectionnez **Se connecter via une passerelle de données locale**.

1. Fournissez ces informations de connexion, puis sélectionnez **Créer**.

   | Propriété | Propriété JSON | Obligatoire | Valeur d'exemple | Description |
   |----------|---------------|----------|---------------|-------------|
   | Nom de la connexion | `name` | Oui | `informix-demo-connection` | Nom à utiliser pour la connexion à votre base de données Informix |
   | Serveur | `server` | Oui | - Cloud : `informixdemo.cloudapp.net:9089` <br>- Local : `informixdemo:9089` | L’adresse TCP/IP ou l’alias au format IPv4 ou IPv6, suivi de deux-points et d’un numéro de port TCP/IP |
   | Base de données | `database` | Oui | `nwind` | Nom de la base de données relationnelle DRDA (RDBNAM) ou nom de la base de données Informix (dbname). Informix accepte une chaîne de 128 octets. |
   | Authentification | `authentication` | Local uniquement | **De base** ou **Windows** (Kerberos) | Type d’authentification requis par votre base de données Informix. Cette propriété s’affiche uniquement quand vous sélectionnez **Se connecter via une passerelle de données locale**. |
   | Nom d’utilisateur | `username` | Non | <*nom_utilisateur_base_de_données*> | Nom d’utilisateur pour la base de données |
   | Mot de passe | `password` | Non | <*mot-de_passe_base_de_données*> | Mot de passe pour la base de données |
   | Passerelle | `gateway` | Local uniquement | - <*abonnement_Azure*> <br>- <*ressource_passerelle_de_données_locale_Azure*> | Abonnement Azure et nom de la ressource Azure pour la passerelle de données locale que vous avez créée dans le portail Azure. La propriété **Passerelle** et les sous-propriétés s’affichent uniquement quand vous sélectionnez **Se connecter via une passerelle de données locale**. |
   ||||||

   Par exemple :

   * **Base de données cloud**

     ![Informations de connexion à la base de données cloud](./media/connectors-create-api-informix/informix-cloud-connection.png)

   * **Base de données locale**

     ![Informations de connexion à la base de données locale](./media/connectors-create-api-informix/informix-on-premises-connection.png)

1. Enregistrez votre application logique.

<a name="test-logic-app"></a>

## <a name="test-your-logic-app"></a>Tester votre application logique

1. Dans la barre d’outils du Concepteur d’application logique, sélectionnez **Exécuter**. Après l’exécution de votre application logique, vous pouvez afficher les sorties de cette exécution.

1. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. Dans le volet Vue d’ensemble, sous **Résumé** > **Historique des exécutions**, sélectionnez la dernière exécution.

1. Sous **Exécution de l’application logique**, sélectionnez **Détails de l’exécution**.

1. Dans la liste des actions, sélectionnez celle dont vous souhaitez afficher les sorties, par exemple **Obtenir les tables**.

   Si l’action a réussi, leur propriété **État** est marquée comme **Réussite**.

1. Pour afficher les entrées, sous **Lien d’entrées**, sélectionnez le lien URL. Pour afficher les sorties, sous **Lien de sorties**, sélectionnez le lien URL. Voici quelques exemples de sorties :

   * **Obtenir les tables** affiche une liste de tables :

     ![Sorties de l’action « Obtenir les tables »](./media/connectors-create-api-informix/InformixconnectorGetTablesLogicAppRunOutputs.png)

   * **Obtenir les lignes** affiche une liste de lignes :

     ![Sorties de l’action « Obtenir les lignes »](./media/connectors-create-api-informix/InformixconnectorGetRowsOutputs.png)

   * **Obtenir la ligne** affiche la ligne spécifiée :

     ![Sorties de l’action « Obtenir la ligne »](./media/connectors-create-api-informix/InformixconnectorGetRowOutputs.png)

   * **Insérer une ligne** affiche la nouvelle ligne :

     ![Sorties de l’action « Insérer une ligne »](./media/connectors-create-api-informix/InformixconnectorInsertRowOutputs.png)

   * **Mettre à jour la ligne** affiche la ligne mise à jour :

     ![Sorties de l’action « Mettre à jour la ligne »](./media/connectors-create-api-informix/InformixconnectorUpdateRowOutputs.png)

   * **Supprimer la ligne** affiche la ligne supprimée :

     ![Sorties de l’action « Supprimer la ligne »](./media/connectors-create-api-informix/InformixconnectorDeleteRowOutputs.png)

## <a name="connector-specific-details"></a>Détails spécifiques du connecteur

Pour obtenir des détails techniques sur les déclencheurs, les actions et les limites, qui sont fournis par la description OpenAPI du connecteur (anciennement Swagger), consultez la [page de référence du connecteur](/connectors/informix/).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](apis-list.md)
