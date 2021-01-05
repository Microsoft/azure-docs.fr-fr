---
title: Gérer les ressources Azure Cosmos DB à l’aide de l’Explorateur Stockage Azure
description: Apprenez à vous connecter à Azure Cosmos DB et à gérer ses ressources à l’aide de l’Explorateur Stockage Azure.
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/23/2020
ms.author: dech
ms.custom: seodec18, has-adal-ref
ms.openlocfilehash: 5b09ce48226b3c31efce4966ec776c10931cc391
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348651"
---
# <a name="manage-azure-cosmos-db-resources-by-using-azure-storage-explorer"></a>Gérer les ressources Azure Cosmos DB à l’aide de l’Explorateur Stockage Azure
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Vous pouvez utiliser l’Explorateur stockage Azure pour vous connecter à Azure Cosmos DB. Il vous permet de vous connecter à des comptes Azure Cosmos DB hébergés sur Azure et des clouds souverains à partir de Windows, macOS ou Linux.

Utilisez le même outil pour gérer vos différentes entités Azure au même endroit. Vous pouvez gérer des entités Azure Cosmos DB, de manipuler des données, de mettre à jour des procédures stockées et des déclencheurs, ainsi que d’autres entités Azure comme les files d’attente et les objets blob de stockage. L’Explorateur Stockage Azure prend en charge les comptes Cosmos configurés pour les API SQL, MongoDB, Graph et Table.

> [!NOTE]
> L’intégration d’Azure Cosmos DB avec Explorateur Stockage est déconseillée. Les fonctionnalités existantes ne seront pas supprimées avant au minimum un an après cette mise en production. Vous devez utiliser à la place le [portail Azure](https://portal.azure.com/), l’[application de bureau du portail Azure](https://portal.azure.com/App/Download) ou l’[explorateur Azure Cosmos DB](data-explorer.md) autonome. Les autres options contiennent de nombreuses nouvelles fonctionnalités qui ne sont pas actuellement prises en charge dans Explorateur Stockage.

## <a name="prerequisites"></a>Prérequis

Un compte Cosmos avec l’API SQL ou l’API pour MongoDB d’Azure Cosmos DB. Si vous ne possédez pas de compte, vous pouvez en créer un compte dans le portail Azure. Voir [Azure Cosmos DB : Développer une application web API SQL avec .NET et le portail Azure](create-sql-api-dotnet.md) pour plus d’informations.

## <a name="installation"></a>Installation

Pour installer la dernière version de l’Explorateur Stockage Azure, consultez [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/). Nous prenons en charge les versions Windows, Linux et macOS.

## <a name="connect-to-an-azure-subscription"></a>Connexion à un abonnement Azure

1. Après avoir installé l’**Explorateur de stockage Azure**, sélectionnez l’icône du **plug-in** dans le volet de gauche.

   :::image type="content" source="./media/storage-explorer/plug-in-icon.png" alt-text="Capture d’écran montrant l’icône de plug-in dans le volet de gauche.":::

1. Sélectionnez **Ajouter un compte Azure**, puis **Connexion**.

   :::image type="content" source="./media/storage-explorer/connect-to-azure-subscription.png" alt-text="Capture d’écran de la fenêtre Se connecter au stockage Azure montrant la case d’option Ajouter un compte Azure sélectionnée et le menu déroulant Environnement Azure.":::

1. Dans la boîte de dialogue **Connexion à Azure**, sélectionnez **Se connecter**, puis entrez vos informations d’identification Azure.

    :::image type="content" source="./media/storage-explorer/sign-in.png" alt-text="Capture d’écran de la fenêtre de connexion montrant où entrer vos informations d’identification pour votre abonnement Azure.":::

1. Sélectionnez votre abonnement dans la liste, puis **Appliquer**.

    :::image type="content" source="./media/storage-explorer/apply-subscription.png" alt-text="Capture d’écran du volet gestion des comptes, montrant une liste d’abonnements et le bouton Appliquer.":::

    Le volet Explorateur se met à jour et affiche les comptes de l’abonnement sélectionné.

    :::image type="content" source="./media/storage-explorer/account-list.png" alt-text="Capture d’écran du volet de l’explorateur, mise à jour pour afficher les comptes de l’abonnement sélectionné.":::

    Votre **compte Cosmos DB** est connecté à votre abonnement Azure.

## <a name="use-a-connection-string-to-connect-to-azure-cosmos-db"></a>Utiliser une chaîne de connexion pour se connecter à Azure Cosmos DB

Vous pouvez utiliser une chaîne de connexion pour vous connecter à Azure Cosmos DB. Cette méthode ne prend en charge que les API SQL et Table. Pour vous connecter avec une chaîne de connexion, procédez comme suit :

1. Recherchez **Locaux et joints** dans l’arborescence de gauche, cliquez avec le bouton droit sur **Comptes Cosmos DB**, puis sélectionnez **Connect to Cosmos DB** (Se connecter à Cosmos DB).

    :::image type="content" source="./media/storage-explorer/connect-to-db-by-connection-string.png" alt-text="Capture d’écran montrant le menu déroulant après un clic droit, avec la mise en surbrillance de la connexion à Azure Cosmos DB.":::

2. Dans la fenêtre **Se connecter à Cosmos DB** :
   1. Sélectionnez l’API dans le menu déroulant.
   1. Chaîne de connexion interne dans la zone **Chaîne de connexion**. Pour plus d’informations sur la récupération de la chaîne de connexion principale, consultez [Obtenir la chaîne de connexion](manage-with-powershell.md#list-keys).
   1. Entrez un **Libellé de compte**, puis sélectionnez **Suivant** pour vérifier le résumé.
   1. Sélectionnez **Se connecter** pour connecter le compte Azure Cosmos DB.

      :::image type="content" source="./media/storage-explorer/connection-string.png" alt-text="Capture d’écran de la fenêtre Se connecter à Cosmos DB, qui présente le menu déroulant API, la zone Chaîne de connexion et la zone Libellé de compte.":::

> [!NOTE]
> Si l’Explorateur Stockage Azure indique que le format de la chaîne de connexion Azure Cosmos DB n’est pas valide, vérifiez que la chaîne de connexion se termine par un point-virgule (`;`). Voici un exemple de chaîne de connexion Azure Cosmos DB valide : `AccountEndpoint=https://accountname.documents.azure.com:443;AccountKey=accountkey==;`

## <a name="use-a-local-emulator-to-connect-to-azure-cosmos-db"></a>Utiliser un émulateur local pour se connecter à Azure Cosmos DB

Procédez comme suit pour vous connecter à Azure Cosmos DB avec un émulateur. Cette méthode prend en charge uniquement les comptes SQL.

1. Installez l’émulateur Cosmos DB, puis ouvrez-le. Pour savoir comment installer l’émulateur, consultez [Utilisation de l’émulateur Azure Cosmos DB pour le développement local et le test](./local-emulator.md).

1. Recherchez **Locaux et joints** dans l’arborescence de gauche, cliquez avec le bouton droit sur **Comptes Cosmos DB**, puis sélectionnez **Connect to Cosmos DB Emulator** (Se connecter à l’émulateur Cosmos DB).

    :::image type="content" source="./media/storage-explorer/emulator-entry.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de la connexion à l’émulateur Azure Cosmos DB.":::

1. Dans la fenêtre **Se connecter à Cosmos DB** :
   1. Chaîne de connexion interne dans la zone **Chaîne de connexion**. Pour plus d’informations sur la récupération de la chaîne de connexion principale, voir [Obtenir la chaîne de connexion](manage-with-powershell.md#list-keys).
   1. Entrez un **Libellé de compte**, puis sélectionnez **Suivant** pour vérifier le résumé.
   1. Sélectionnez **Se connecter** pour connecter le compte Azure Cosmos DB.

      :::image type="content" source="./media/storage-explorer/emulator-dialog.png" alt-text="Capture d’écran de la fenêtre Se connecter à Cosmos DB, qui présente la zone Chaîne de connexion et la zone Libellé de compte.":::

## <a name="azure-cosmos-db-resource-management"></a>Gestion des ressources Azure Cosmos DB

Utilisez les opérations suivantes pour gérer un compte Azure Cosmos DB :

* Ouvrez le compte dans le portail Azure.
* Ajoutez la ressource à la liste d’accès rapide.
* Recherchez et actualisez les ressources.
* Créez et supprimez des bases de données.
* Créez et supprimez des collections.
* Créez, modifiez, supprimez et filtrez des documents.
* Gérer les procédures stockées, les déclencheurs et les fonctions définies par l’utilisateur.

### <a name="quick-access-tasks"></a>Tâches d’accès rapide

Vous pouvez cliquer avec le bouton droit sur un abonnement dans le volet Explorateur pour effectuer de nombreuses tâches d’action rapide, par exemple :

* Cliquez avec le bouton droit sur un compte ou une base de données Azure Cosmos DB, puis sélectionnez **Ouvrir dans le portail** pour gérer la ressource dans le navigateur sur le portail Azure.

  :::image type="content" source="./media/storage-explorer/open-in-portal.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance d’Ouvrir dans le portail.":::

* Cliquez avec le bouton droit sur un compte, une base de données ou une collection Azure Cosmos DB, puis sélectionnez **Ajouter à Accès rapide** pour l’ajouter au menu Accès rapide.

* Sélectionnez **Rechercher à partir d’ici** pour effectuer une recherche par mot clé dans le chemin sélectionné.

    :::image type="content" source="./media/storage-explorer/search-from-here.png" alt-text="Capture d’écran montrant la zone de recherche en surbrillance.":::

### <a name="database-and-collection-management"></a>Gestion des bases de données et des collections

#### <a name="create-a-database"></a>Création d'une base de données

1. Cliquez avec le bouton droit sur le compte Azure Cosmos DB, puis sélectionnez **Créer une de base de données**.

   :::image type="content" source="./media/storage-explorer/create-database.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de Créer une base de données.":::

1. Entrez le nom de la base de données, puis appuyez sur **Entrée** pour terminer.

#### <a name="delete-a-database"></a>Supprimer une base de données

1. Cliquez avec le bouton de droite sur la base de données et sélectionnez **Supprimer la base de données**. 

   :::image type="content" source="./media/storage-explorer/delete-database1.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de Supprimer une base de données.":::

1. Dans la fenêtre indépendante qui s’affiche, sélectionnez **Oui**. Le nœud de base de données est supprimé et le compte Azure Cosmos DB s’actualise automatiquement.

   :::image type="content" source="./media/storage-explorer/delete-database2.png" alt-text="Capture d’écran de la fenêtre de confirmation avec le bouton Oui mis en surbrillance.":::

#### <a name="create-a-collection"></a>Création d'une collection

1. Cliquez avec le bouton droit sur votre base de données, puis sélectionnez **Créer une collection**.

   :::image type="content" source="./media/storage-explorer/create-collection.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de Créer une collection.":::

1. Dans la fenêtre créer une collection, entrez les informations demandées, par exemple **ID de collection**, **Capacité de stockage**, et ainsi de suite. Sélectionnez **OK** pour terminer.

   :::image type="content" source="./media/storage-explorer/create-collection2.png" alt-text="Capture d’écran de la fenêtre Créer une collection, montrant la zone de ID de collection et les boutons de capacité de stockage.":::

1. Sélectionnez **Illimité** pour pouvoir spécifier la clé de partition, puis sélectionnez **OK** pour terminer.

   > [!NOTE]
   > Si une clé de partition est utilisée pendant la création d’une collection, une fois la création terminée, vous ne pouvez pas modifier la valeur de la clé de partition sur la collection.

    :::image type="content" source="./media/storage-explorer/partitionkey.png" alt-text="Capture d’écran de la fenêtre Créer une collection, avec Illimité sélectionné pour la capacité de stockage et la zone Clé de partition mise en surbrillance.":::

#### <a name="delete-a-collection"></a>Supprimer une collection

- Cliquez avec le bouton droit sur la collection, sélectionnez **Supprimer la collection**, puis **Oui** dans la fenêtre contextuelle.

    Le nœud de collection est supprimé et la base de données s’actualise automatiquement.

    :::image type="content" source="./media/storage-explorer/delete-collection.png" alt-text="Capture d’écran montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de Supprimer une collection.":::

### <a name="document-management"></a>Gestion de documents

#### <a name="create-and-modify-documents"></a>Créer et modifier des documents

- Ouvrez **Documents** dans le volet de gauche, sélectionnez **Nouveau document**, modifiez le contenu dans le volet droit, puis sélectionnez **Enregistrer**.
- Vous pouvez également mettre à jour un document existant, puis sélectionner **Enregistrer**. Pour ignorer les modifications, sélectionnez **Ignorer**.

  :::image type="content" source="./media/storage-explorer/document.png" alt-text="Capture d’écran montrant les documents mis en surbrillance dans le volet de gauche. Dans le volet droit, Nouveau document, Enregistrer et Ignorer sont mis en surbrillance.":::

#### <a name="delete-a-document"></a>Supprimer un document

* Sélectionnez le bouton **Supprimer** pour supprimer le document sélectionné.

#### <a name="query-for-documents"></a>Rechercher des documents

* Pour modifier le filtre de document, entrez une [requête SQL](./sql-query-getting-started.md), puis sélectionnez **Appliquer**.

  :::image type="content" source="./media/storage-explorer/document-filter.png" alt-text="Capture d’écran du volet droit, montrant les boutons Filtrer et Appliquer, le numéro d’ID et la zone de requête en surbrillance.":::

### <a name="graph-management"></a>Gestion des graphiques

#### <a name="create-and-modify-a-vertex"></a>Créer et modifier un sommet

* Pour créer un sommet, ouvrez **Graphe** dans le volet de gauche, sélectionnez **Nouveau sommet**, modifiez le contenu, puis sélectionnez **OK**.
* Pour modifier un sommet existant, sélectionnez l’icône de crayon dans le volet droit.

   :::image type="content" source="./media/storage-explorer/vertex.png" alt-text="Capture d’écran montrant le graphique sélectionné dans le volet de gauche et le nouveau sommet et l’icône de stylet en surbrillance dans le volet droit.":::

#### <a name="delete-a-graph"></a>Supprimer un graphique

* Pour supprimer un sommet, sélectionnez l’icône de corbeille à côté du nom du sommet.

#### <a name="filter-for-graph"></a>Filtrer un graphique

* Pour modifier le filtre de graphe, entrez une [requête Gremlin](gremlin-support.md), puis sélectionnez **Appliquer un filtre**.

   :::image type="content" source="./media/storage-explorer/graph-filter.png" alt-text="Capture d’écran montrant le graphique sélectionné dans le volet de gauche et le bouton Appliquer le filtre et la zone de requête en surbrillance dans le volet droit.":::

### <a name="table-management"></a>Gestion des tables

#### <a name="create-and-modify-a-table"></a>Créer et modifier une table

* Pour créer une table :
   1. Dans le volet de gauche, ouvrez **Entités**, puis sélectionnez **Ajouter**.
   1. Dans la boîte de dialogue **Ajouter une entité**, modifiez le contenu.
   1. Sélectionnez le bouton **Ajouter une propriété** pour ajouter une propriété.
   1. Sélectionnez **Insérer**.

      :::image type="content" source="./media/storage-explorer/table.png" alt-text="Capture d’écran montrant Entités en surbrillance dans le volet de gauche, et Ajouter, Modifier, Ajouter une propriété et Insérer en surbrillance dans le volet droit.":::

* Pour modifier une table, sélectionnez **Modifier**, modifiez le contenu, puis sélectionnez **Mettre à jour**.

   

#### <a name="import-and-export-table"></a>Importer et exporter une table

* Pour importer, sélectionnez le bouton **Importer** et choisissez une table existante.
* Pour exporter, sélectionnez le bouton **Exporter** et choisissez une destination.

   :::image type="content" source="./media/storage-explorer/table-import-export.png" alt-text="Capture d’écran montrant les boutons Importer et Exporter mis en surbrillance dans le volet droit.":::

#### <a name="delete-entities"></a>Supprimer des entités

* Sélectionnez les entités, puis sélectionnez le bouton **Supprimer**.

  :::image type="content" source="./media/storage-explorer/table-delete.png" alt-text="Capture d’écran montrant le bouton Supprimer mis en surbrillance dans le volet droit et une fenêtre contextuelle de confirmation avec l’option Oui en surbrillance.":::

#### <a name="query-a-table"></a>Interrogation d’une table

- Sélectionnez le bouton **Requête**, entrez une condition de requête, puis sélectionnez le bouton **Exécuter la requête**. Pour fermer le volet de requête, sélectionnez le bouton **Fermer la requête**.

  :::image type="content" source="./media/storage-explorer/table-query.png" alt-text="Capture d’écran du volet droit, montrant le bouton Exécuter la requête et le bouton Fermer la requête en surbrillance.":::

### <a name="manage-stored-procedures-triggers-and-udfs"></a>Gérer les procédures stockées, les déclencheurs et les fonctions définies par l'utilisateur

* Pour créer une procédure stockée :
  1. Dans l’arborescence de gauche, cliquez avec le bouton droit sur **Procédures stockées**, puis sélectionnez **Créer une procédure stockée**.
  
     :::image type="content" source="./media/storage-explorer/stored-procedure.png" alt-text="Capture d’écran du volet de gauche montrant le menu qui s’affiche après un clic droit, avec la mise en surbrillance de Créer une procédure stockée.":::
  
  1. Entrez un nom dans la partie gauche, entrez les scripts de procédure stockée dans le volet droit, puis sélectionnez **Créer**.
  
* Pour modifier une procédure stockée existante, double-cliquez sur la procédure, effectuez la mise à jour, puis sélectionnez **Mettre à jour** pour enregistrer. Vous pouvez également sélectionner **Ignorer** pour annuler la modification.

* Les opérations pour les **déclencheurs** et les **fonctions définies par l’utilisateur** sont similaires à celles des **procédures stockées**.

## <a name="troubleshooting"></a>Dépannage

Vous trouverez ci-dessous des solutions aux problèmes courants qui surviennent lorsque vous utilisez Azure Cosmos DB dans l’Explorateur Stockage.

### <a name="sign-in-issues"></a>Problèmes de connexion

Tout d’abord, redémarrez votre application pour voir si cela résout le problème. Si le problème persiste, poursuivez la résolution des problèmes.

#### <a name="self-signed-certificate-in-certificate-chain"></a>Certificat auto-signé dans la chaîne d’approbation

Vous pouvez voir cette erreur pour plusieurs raisons. Les deux plus courantes sont :

* Vous êtes derrière un *proxy transparent*. Quelqu’un, comme votre service informatique, intercepte le trafic HTTPS, le déchiffre, puis le chiffre à l’aide d’un certificat auto-signé.

* Vous exécutez un logiciel, comme un logiciel antivirus. Le logiciel injecte un certificat TLS/SSL auto-signé dans les messages HTTPS que vous recevez.

Lorsque l’Explorateur Stockage trouve un certificat auto-signé, il ne sait pas si le message HTTPS qu’il reçoit est falsifié. Si vous avez une copie du certificat auto-signé, vous pouvez indiquer à l’Explorateur de stockage de lui faire confiance. Si vous n’êtes pas sûr de savoir qui a injecté le certificat, vous pouvez suivre ces étapes pour essayer de le déterminer :

1. Installez OpenSSL :

     - [Windows](https://slproweb.com/products/Win32OpenSSL.html) : Toutes les versions légères conviennent.
     - macOS et Linux : normalement fourni avec votre système d’exploitation.

1. Exécuter OpenSSL :
    * Windows : accédez au répertoire d’installation, puis **/bin/** , puis double-cliquez sur **openssl.exe**.
    * Mac et Linux : Exécutez **openssl** à partir d’un terminal.
1. Exécutez `s_client -showcerts -connect microsoft.com:443`.
1. Recherchez les certificats auto-signés. Si vous ne savez pas lesquels sont auto-signés, recherchez ceux dont le sujet (« s: ») et l’émetteur (« i: ») sont identiques.
1. Si vous avez trouvé les certificats auto-signés, pour chacun d’eux copiez et collez tout depuis **-----BEGIN CERTIFICATE-----** jusqu’à **-----END CERTIFICATE-----** (les deux inclus) dans un nouveau fichier .CER.
1. Ouvrez l’Explorateur de stockage puis accédez à **Modifier** > **Certificats SSL** > **Importer des certificats**. Servez-vous du sélecteur de fichiers pour rechercher, sélectionner et ouvrir les fichiers .CER que vous avez créés.

Si vous ne trouvez pas de certificats auto-signés, vous pouvez envoyer des commentaires pour obtenir de l’aide.

#### <a name="unable-to-retrieve-subscriptions"></a>Impossible de récupérer les abonnements

Si vous ne parvenez pas à récupérer vos abonnements après vous être connecté, essayez les suggestions suivantes :

* Vérifiez que votre compte a accès aux abonnements. Pour cela, connectez-vous au [portail Azure](https://portal.azure.com/).
* Vérifiez que vous êtes connecté à l’environnement approprié :
  * [Microsoft Azure](https://portal.azure.com/)
  * [Azure Chine](https://portal.azure.cn/)
  * [Azure Allemagne](https://portal.microsoftazure.de/)
  * [Azure US Government](https://portal.azure.us/)
  * Environnement personnalisé/Azure Stack
* Si vous vous trouvez derrière un proxy, vérifiez que le proxy de l’Explorateur Stockage est correctement configuré.
* Supprimez le compte, puis rajoutez-le.
* Supprimez les fichiers suivants de votre répertoire de base (par exemple : C:\Users\ContosoUser), puis rajoutez le compte :
  * .adalcache
  * .devaccounts
  * .extaccounts
* Appuyez sur la touche F12 pour ouvrir la console de développeur. Vérifiez la console pour tout message d’erreur lorsque vous vous connectez.

   :::image type="content" source="./media/storage-explorer/console.png" alt-text="Capture d’écran de la console d’outils de développement, montrant la console en surbrillance.":::

#### <a name="unable-to-see-the-authentication-page"></a>Impossible de voir la page d’authentification

Si vous ne parvenez pas à voir la page d’authentification :

* Selon la vitesse de votre connexion, le chargement de la page de connexion peut prendre un certain temps. Attendez au moins une minute avant de fermer la boîte de dialogue d’authentification.
* Si vous vous trouvez derrière un proxy, vérifiez que le proxy de l’Explorateur Stockage est correctement configuré.
* Dans la console d’outils de développement (F12), regardez les réponses pour voir si vous pouvez trouver des indices pour la raison pour laquelle l’authentification ne fonctionne pas.

#### <a name="cant-remove-an-account"></a>Impossible de supprimer un compte

Si vous ne pouvez pas supprimer un compte, ou que le lien de réauthentification est inopérant :

* Supprimez les fichiers suivants de votre répertoire de base, puis rajoutez le compte :
  * .adalcache
  * .devaccounts
  * .extaccounts

* Si vous souhaitez supprimer des ressources de stockage jointes SAP, supprimez :
  * Dossier %AppData%/StorageExplorer pour Windows
  * /Users/<votre_nom>/Library/Application SUpport/StorageExplorer pour macOS
  * ~/.config/StorageExplorer pour Linux
  
  > [!NOTE]
  > Si vous supprimez ces fichiers, **vous devrez entrer à nouveau toutes vos informations d’identification**.

### <a name="httphttps-proxy-issue"></a>Problème de proxy HTTP/HTTPS

Vous ne pouvez pas afficher les nœuds Azure Cosmos DB dans l’arborescence de gauche lorsque vous configurez un proxy HTTP/HTTPS dans ASE. Vous pouvez utiliser l’Explorateur de données Azure Cosmos DB dans le portail Azure pour contourner ce problème.

### <a name="development-node-under-local-and-attached-node-issue"></a>Problème de nœud de « Développement » sous le nœud « Local et attaché »

Il n’y a pas de réponse après avoir sélectionné le nœud **Développement** sous le nœud **Local et attaché** dans l’arborescence de gauche. Ce comportement est attendu.

:::image type="content" source="./media/storage-explorer/development.png" alt-text="Capture d’écran montrant le nœud de développement sélectionné.":::

### <a name="attach-an-azure-cosmos-db-account-in-the-local-and-attached-node-error"></a>Erreur lors de l’attachement d’un compte Azure Cosmos DB dans le nœud **Local et attaché**

Si l’erreur suivante s’affiche après l’attachement d’un compte Azure Cosmos DB dans le nœud **Local et attaché**, vérifiez que vous utilisez la bonne chaîne de connexion.

:::image type="content" source="./media/storage-explorer/attached-error.png" alt-text="Capture d’écran de la fenêtre contextuelle Impossible de récupérer les ressources enfants, indiquant getaddrinfo ENOTFOUND.":::

### <a name="expand-azure-cosmos-db-node-error"></a>Erreur de développement du nœud Azure Cosmos DB

L’erreur suivante peut s’afficher lorsque vous tentez de développer des nœuds dans l’arborescence de gauche.

:::image type="content" source="./media/storage-explorer/expand-error.png" alt-text="Capture d’écran de la fenêtre contextuelle Impossible de récupérer les ressources enfants, indiquant Impossible de se connecter à ce compte Cosmos DB.":::

Essayez les suggestions suivantes :

* Vérifiez si le compte Azure Cosmos DB est en cours d’approvisionnement. Réessayez lorsque le compte est créé avec succès.
* Si le compte se trouve sous les nœuds **Accès rapide** ou **Accès rapide**, vérifiez si le compte est supprimé. Si c’est le cas, vous devez supprimer le nœud manuellement.

## <a name="next-steps"></a>Étapes suivantes

* Visionnez cette vidéo pour voir comment utiliser Azure Cosmos DB dans l’Explorateur Stockage Azure : [Utiliser Azure Cosmos DB dans l’Explorateur Stockage Azure](https://www.youtube.com/watch?v=iNIbg1DLgWo&feature=youtu.be).
* Pour en savoir plus sur l’Explorateur Stockage et sur comment connecter plus de services, consultez [Prise en main de l’Explorateur Stockage (préversion)](../vs-azure-tools-storage-manage-with-storage-explorer.md).
