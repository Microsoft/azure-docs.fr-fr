---
title: Établir une connexion à SQL Server, Azure SQL Database ou Azure SQL Managed Instance
description: Automatiser des tâches pour les bases de données SQL localement ou dans le cloud à l’aide d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: conceptual
ms.date: 10/22/2020
tags: connectors
ms.openlocfilehash: ce7679fff86d2c96588cf2b704d44238535963b3
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130933"
---
# <a name="automate-workflows-for-a-sql-database-by-using-azure-logic-apps"></a>Automatiser les workflows pour une base de données SQL à l’aide d’Azure Logic Apps

Cet article explique comment accéder aux données dans votre base de données SQL depuis une application logique avec le connecteur SQL Server. Vous pouvez ainsi automatiser des tâches, des processus ou des workflows qui gèrent vos données et ressources SQL en créant des applications logiques. Le connecteur SQL Server fonctionne pour [SQL Server](/sql/sql-server/sql-server-technical-documentation), ainsi que [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md) et [Azure SQL Managed instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md).

Vous pouvez créer des applications logiques qui s’exécutent lorsqu’elles sont déclenchées par des événements dans votre base de données SQL ou dans d’autres systèmes, tels que Dynamics CRM Online. Vos applications logiques peuvent également obtenir, insérer et supprimer des données, ainsi qu’exécuter des requêtes SQL et des procédures stockées. Par exemple, vous pouvez concevoir une application logique qui recherche automatiquement les nouveaux enregistrements dans Dynamics CRM Online, ajoute des éléments à votre base de données SQL en cas de nouveaux enregistrements, puis envoie des alertes sur les éléments ajoutés par e-mail.

Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour obtenir les informations techniques, les limitations et les problèmes connus spécifiques aux connecteurs, consultez la page [référence du connecteur SQL Server](/connectors/sql/).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [vous inscrire pour obtenir un compte Azure gratuitement](https://azure.microsoft.com/free/).

* Une [base de données SQL Server](/sql/relational-databases/databases/create-a-database), [Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md) ou [Azure SQL Managed Instance](../azure-sql/managed-instance/instance-create-quickstart.md).

  Vos tables doivent contenir des données afin que votre application logique puisse renvoyer les résultats lors de l’appel des opérations. Si vous utilisez Azure SQL Database, vous pouvez utiliser des échantillons de bases de données inclus.

* Le nom de votre serveur SQL, le nom de la base de données, votre nom d’utilisateur et votre mot de passe. Vous avez besoin de ces informations d’identification pour autoriser votre application logique à accéder à votre serveur SQL.

  * Pour un SQL Server local, vous trouverez ces informations dans la chaîne de connexion :

    `Server={your-server-address};Database={your-database-name};User Id={your-user-name};Password={your-password};`

  * Pour Azure SQL Database, vous trouverez ces informations dans la chaîne de connexion.
  
    Par exemple, pour trouver cette chaîne dans le Portail Azure, ouvrez votre base de données. Dans le menu de la base de données, sélectionnez **Chaînes de connexion** ou **Propriétés**  :

    `Server=tcp:{your-server-name}.database.windows.net,1433;Initial Catalog={your-database-name};Persist Security Info=False;User ID={your-user-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

<a name="multi-tenant-or-ise"></a>

* Selon que vos applications logiques vont être exécutées dans Azure global à plusieurs locataires ou dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), voici d’autres exigences requises pour la connexion à un SQL Server local :

  * Pour les applications logiques dans Azure global à plusieurs locataires qui se connectent à un SQL Server local, vous devez disposer de [ la passerelle de données locale](../logic-apps/logic-apps-gateway-install.md) installée sur un ordinateur local et d’une [ressource de passerelle de données qui est déjà créée dans Azure](../logic-apps/logic-apps-gateway-connection.md).

  * Pour les applications logiques d’un ISE qui se connectent à un SQL Server local et utilisent l’Authentification Windows, le connecteur SQL Server avec version ISE ne prend pas en charge l’Authentification Windows. Vous devez donc toujours utiliser la passerelle de données et le connecteur SQL Server non ISE. Pour les autres types d’authentification, vous n’avez pas besoin d’utiliser la passerelle de données et pouvez utiliser le connecteur avec version ISE.

* L’application logique où vous devez avoir accès à votre base de données SQL. Pour démarrer votre application logique avec un déclencheur SQL, vous avez besoin d’une [application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="create-connection"></a>

## <a name="connect-to-your-database"></a>Connectez-vous à votre base de données

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

Maintenant, effectuez les étapes suivantes :

* [Établissez une connexion à Azure SQL Database ou à Azure SQL Managed Instance basés sur le Cloud](#connect-azure-sql-db)
* [Se connecter à un SQL Server local](#connect-sql-server)

<a name="connect-azure-sql-db"></a>

### <a name="connect-to-azure-sql-database-or-managed-instance"></a>Se connecter à Azure SQL Database ou à Azure SQL Database Managed Instance

Pour accéder à Azure SQL Managed Instance sans utiliser la passerelle de données locale ou l’environnement de service d’intégration, vous devez [configurer le point de terminaison public sur Azure SQL Managed Instance](../azure-sql/managed-instance/public-endpoint-configure.md). Le point de terminaison public utilise le port 3342. Assurez-vous de spécifier ce numéro de port lorsque vous créez la connexion à partir de votre application logique.


La première fois que vous ajoutez un [déclencheur SQL](#add-sql-trigger) ou une [action SQL](#add-sql-action), et que vous n’avez pas préalablement créé de connexion à votre base de données, vous êtes invité à effectuer ces étapes :

1. Pour le **Type d’authentification** , sélectionnez l’authentification requise et activée sur votre base de données dans Azure SQL Database ou Azure SQL Managed Instance :

   | Authentification | Description |
   |----------------|-------------|
   | [**Azure AD Integrated**](../azure-sql/database/authentication-aad-overview.md) | - Prend en charge le connecteur SQL Server non-ISE et ISE. <p><p>- Nécessite une identité valide dans Azure Active Directory (Azure AD) qui a accès à votre base de données. <p>Pour plus d’informations, consultez les rubriques suivantes : <p>- [Vue d’ensemble de la sécurité Azure SQL : authentification](../azure-sql/database/security-overview.md#authentication) <br>- [Autoriser l’accès à la base de données Azure SQL : authentification et autorisation](../azure-sql/database/logins-create-manage.md#authentication-and-authorization) <br>- [Azure SQL : authentification Azure AD Integrated](../azure-sql/database/authentication-aad-overview.md) |
   | [**Authentification SQL Server**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication) | - Prend en charge le connecteur SQL Server non-ISE et ISE. <p><p>- Requiert un nom d’utilisateur et un mot de passe sécurisé valides qui sont créés et stockés dans votre base de données. <p>Pour plus d’informations, consultez les rubriques suivantes : <p>- [Vue d’ensemble de la sécurité Azure SQL : authentification](../azure-sql/database/security-overview.md#authentication) <br>- [Autoriser l’accès à la base de données Azure SQL : authentification et autorisation](../azure-sql/database/logins-create-manage.md#authentication-and-authorization) |
   |||

   Cet exemple se poursuit avec **Azure AD Integrated**  :

   ![Capture d’écran montrant la fenêtre de connexion « SQL Server » avec la liste « Type d’authentification » ouverte et l’option « Azure AD Integrated » sélectionnée.](./media/connectors-create-api-sqlazure/select-azure-ad-authentication.png)

1. Après avoir sélectionné **Azure AD Integrated** , sélectionnez **Se connecter**. Selon que vous utilisez Azure SQL Database ou Azure SQL Managed Instance, sélectionnez vos informations d’identification d’utilisateur pour l’authentification.

1. Sélectionnez ces valeurs pour votre base de données :

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom du serveur** | Oui | L’adresse pour votre SQL Server, par exemple, `Fabrikam-Azure-SQL.database.windows.net` |
   | **Nom de la base de données** | Oui | Le nom de votre base de données SQL, par exemple, `Fabrikam-Azure-SQL-DB` |
   | **Nom de la table** | Oui | La table que vous voulez utiliser, par exemple, `SalesLT.Customer` |
   ||||

   > [!TIP]
   > Pour fournir vos informations de base de données et de table, vous disposez des options suivantes :
   > 
   > * Trouvez ces informations dans la chaîne de connexion de votre base de données. Par exemple, Dans le Portail Azure, recherchez et ouvrez votre base de données. Dans le menu de la base de données, sélectionnez **Chaînes de connexion** ou **Propriétés** , dans lesquelles vous pouvez trouver cette chaîne :
   >
   >   `Server=tcp:{your-server-address}.database.windows.net,1433;Initial Catalog={your-database-name};Persist Security Info=False;User ID={your-user-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`
   >
   > * Par défaut, les tables des bases de données système sont filtrées, elles peuvent donc ne pas s’afficher automatiquement lorsque vous sélectionnez une base de données système. Vous pouvez également indiquer manuellement le nom de la table une fois que vous avez sélectionné **Entrer une valeur personnalisée** dans la liste de base de données.
   >

   Cet exemple montre comment ces valeurs peuvent être présentées :

   ![Créer une connexion à la base de données SQL](./media/connectors-create-api-sqlazure/azure-sql-database-create-connection.png)

1. Maintenant, poursuivez avec les étapes que vous n’avez pas encore effectuées dans [Ajouter un déclencheur SQL](#add-sql-trigger) ou [Ajouter une action SQL](#add-sql-action).

<a name="connect-sql-server"></a>

### <a name="connect-to-on-premises-sql-server"></a>Se connecter à un SQL Server local

La première fois que vous ajoutez un [déclencheur SQL](#add-sql-trigger) ou une [action SQL](#add-sql-action), et que vous n’avez pas préalablement créé de connexion à votre base de données, vous êtes invité à effectuer ces étapes :

1. Pour les connexions à votre SQL Server local qui nécessitent la passerelle de données locale, assurez-vous que vous avez [rempli ces conditions préalables](#multi-tenant-or-ise).

   Sinon, votre ressource de passerelle de données n’apparaîtra pas dans la liste de **Passerelle de connexion** lorsque vous créerez votre connexion.

1. Pour le **Type d’authentification** , sélectionnez l’authentification requise et activée sur votre SQL Server :

   | Authentification | Description |
   |----------------|-------------|
   | [**Authentification Windows**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-windows-authentication) | - Prend en charge uniquement le connecteur SQL Server non-ISE, qui nécessite une ressource de passerelle de données créée préalablement dans Azure pour votre connexion, que vous utilisiez Azure à plusieurs locataires ou un ISE. <p><p>- Requiert un nom d’utilisateur et un mot de passe Windows valides pour confirmer votre identité par le biais de votre compte Windows. <p>Pour plus d'informations, consultez [Authentication Windows](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-windows-authentication) |
   | [**Authentification SQL Server**](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication) | - Prend en charge le connecteur SQL Server non-ISE et ISE. <p><p>- Requiert un nom d’utilisateur et un mot de passe sécurisé valides qui sont créés et stockés dans votre SQL Server. <p>Pour plus d’informations, consultez [Authentification SQL Server](/sql/relational-databases/security/choose-an-authentication-mode#connecting-through-sql-server-authentication). |
   |||

   Cet exemple se poursuit avec l’ **authentification Windows**  :

   ![Sélectionnez le type d’authentification à utiliser](./media/connectors-create-api-sqlazure/select-windows-authentication.png)

1. Sélectionnez ou fournissez les valeurs suivantes pour votre base de données SQL :

   | Propriété | Obligatoire | Description |
   |----------|----------|-------------|
   | **Nom du serveur SQL** | Oui | L’adresse pour votre SQL Server, par exemple, `Fabrikam-Azure-SQL.database.windows.net` |
   | **Nom de la base de données SQL** | Oui | Le nom de votre base de données SQL Server, par exemple, `Fabrikam-Azure-SQL-DB` |
   | **Nom d’utilisateur** | Oui | Votre nom d’utilisateur pour le serveur et la base de données SQL |
   | **Mot de passe** | Oui | Votre mot de passe pour le serveur et la base de données SQL |
   | **Abonnement** |  Oui, pour l’Authentification Windows | L’abonnement Azure pour la ressource de passerelle de données que vous avez créée précédemment dans Azure |
   | **Passerelle de connexion** | Oui, pour l’Authentification Windows | Le nom de la ressource de passerelle de données que vous avez créée précédemment dans Azure <p><p>**Conseil** : Si votre passerelle n’apparaît pas dans la liste, vérifiez que vous avez correctement [configuré votre passerelle](../logic-apps/logic-apps-gateway-connection.md). |
   |||

   > [!TIP]
   > Ces informations se trouvent dans la chaîne de connexion de votre base de données :
   > 
   > * `Server={your-server-address}`
   > * `Database={your-database-name}`
   > * `User ID={your-user-name}`
   > * `Password={your-password}`

   Cet exemple montre comment ces valeurs peuvent être présentées :

   ![Créer une connexion SQL Server terminé](./media/connectors-create-api-sqlazure/sql-server-create-connection-complete.png)

1. Quand vous êtes prêt, sélectionnez **Créer**.

1. Maintenant, poursuivez avec les étapes que vous n’avez pas encore effectuées dans [Ajouter un déclencheur SQL](#add-sql-trigger) ou [Ajouter une action SQL](#add-sql-action).

<a name="add-sql-trigger"></a>

## <a name="add-a-sql-trigger"></a>Ajouter un déclencheur SQL

1. Dans le [Portail Azure](https://portal.azure.com) ou Visual Studio, créez une application logique vide, qui ouvre le Concepteur d’application logique. Cet exemple se poursuit avec le Portail Azure.

1. Dans la zone de recherche du concepteur, entrez `sql server`. Dans la liste des déclencheurs, sélectionnez le déclencheur SQL souhaité. Cet exemple utilise le **Lorsqu'un élément est créé** déclencheur.

   ![Sélectionner le déclencheur « Lorsqu'un élément est créé »](./media/connectors-create-api-sqlazure/select-sql-server-trigger.png)

1. Si vous vous connectez à votre base de données SQL pour la première fois, vous êtes invité à [créer dès à présent votre connexion à la base de données SQL](#create-connection). Après avoir créé cette connexion, vous pouvez passer à l’étape suivante.

1. Dans le déclencheur, spécifiez l’intervalle et la fréquence à laquelle le déclencheur vérifie la table.

1. Pour ajouter d’autres propriétés disponibles pour ce déclencheur, ouvrez la liste **Ajouter un nouveau paramètre**.

   Ce déclencheur ne renvoie qu’une seule ligne de la table sélectionnée. Pour effectuer d’autres tâches, continuez en ajoutant une [action du connecteur SQL](#add-sql-action) ou [une autre action](../connectors/apis-list.md) qui effectue la tâche suivante que vous souhaitez dans votre flux de travail d’application logique.
   
   Par exemple, pour afficher les données de cette ligne, vous pouvez ajouter d’autres actions qui créent un fichier qui comprend les champs de la ligne retournée, puis envoient des alertes par e-mail. Pour en savoir plus sur les autres actions disponibles pour ce connecteur, consultez la [page de référence du connecteur](/connectors/sql/).

1. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

   Bien que cette étape active et publie automatiquement votre application logique dynamique dans Azure, la seule action que votre application logique prend en compte est la vérification de votre base de données en fonction de l’intervalle et de la fréquence spécifiés.

<a name="add-sql-action"></a>

## <a name="add-a-sql-action"></a>Ajouter une action SQL

Dans cet exemple, l’application logique commence par le [déclencheur de périodicité](../connectors/connectors-native-recurrence.md) et appelle une action qui reçoit une ligne d’une base de données SQL.

1. Dans le [Portail Azure](https://portal.azure.com) ou dans Visual Studio, ouvrez votre application logique dans le Concepteur d’application logique. Cet exemple se poursuit avec le Portail Azure.

1. Sous le déclencheur auquel/l’action à laquelle vous souhaitez ajouter l’action SQL, sélectionnez **Nouvelle étape**.

   ![Ajoutez une action à votre application logique](./media/connectors-create-api-sqlazure/select-new-step-logic-app.png)

   Ou, pour ajouter une action entre des étapes, déplacez votre souris sur la flèche de connexion. Cliquez sur le signe ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Sous **Choisir une action** , dans la zone de recherche, entrez `sql server`. Dans la liste des actions, sélectionnez l’action SQL souhaitée. Cet exemple utilise l’action **Obtenir la ligne** , qui obtient un enregistrement unique.

   ![Sélectionnez l’action SQL « Obtenir la ligne »](./media/connectors-create-api-sqlazure/select-sql-get-row-action.png)

1. Si vous vous connectez à votre base de données SQL pour la première fois, vous êtes invité à [créer dès à présent votre connexion à la base de données SQL](#create-connection). Après avoir créé cette connexion, vous pouvez passer à l’étape suivante.

1. Sélectionnez le **nom de la table** , qui est `SalesLT.Customer` dans cet exemple. Entrez l’ **identifiant de la ligne** pour l’enregistrement de votre choix.

   ![Sélectionnez le nom de la table et spécifier l’identifiant de la ligne](./media/connectors-create-api-sqlazure/specify-table-row-id.png)

   Cette action ne renvoie qu’une seule ligne de la table sélectionnée. Ainsi, pour afficher les données dans cette ligne, vous pouvez ajouter d’autres actions qui créent un fichier qui comprend les champs de la ligne retournée, et stocker ce fichier dans un compte de stockage cloud. Pour en savoir plus sur les autres actions disponibles pour ce connecteur, consultez la [page de référence du connecteur](/connectors/sql/).

1. Lorsque c’est chose faite, dans la barre d’outils du concepteur, sélectionnez **Enregistrer**.

   Cette étape active et publie automatiquement votre application logique dans Azure.

<a name="handle-bulk-data"></a>

## <a name="handle-bulk-data"></a>Gérer des données en bloc

Parfois, vous manipulez des jeux de résultats tellement volumineux que le connecteur ne peut pas renvoyer tous les résultats en même temps, ou vous souhaitez mieux contrôler la taille et la structure de vos jeux de résultats. Voici quelques façons de gérer ces grands jeux de résultats :

* Pour mieux gérer les résultats sous forme de jeux plus petits, activez la *pagination*. Pour plus d’informations, voir [Obtenir des données en bloc, des enregistrements et des éléments à l’aide de la pagination](../logic-apps/logic-apps-exceed-default-page-size-with-pagination.md). Pour plus d'informations, consultez [Pagination de SQL pour le transfert de données en bloc avec Logic Apps](https://social.technet.microsoft.com/wiki/contents/articles/40060.sql-pagination-for-bulk-data-transfer-with-logic-apps.aspx)

* Créez une [*procédure stockée*](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) qui trie les résultats comme vous le souhaitez. Le connecteur SQL fournit de nombreuses fonctionnalités back-end auxquelles vous pouvez accéder à l'aide d'Azure Logic Apps afin d'automatiser plus facilement les tâches métier qui utilisent des tables de base de données SQL.

  Lorsque vous procédez à l’extraction ou à l’insertion de plusieurs lignes, votre application logique peut effectuer une itération dans ces lignes en utilisant une [*boucle Until*](../logic-apps/logic-apps-control-flow-loops.md#until-loop) dans ces [limites](../logic-apps/logic-apps-limits-and-config.md). Toutefois, lorsque votre application logique doit manipuler des jeux d’enregistrements si volumineux (plusieurs milliers ou millions de lignes, par exemple), vous devez réduire les coûts liés aux appels à la base de données.

  Pour organiser les résultats à votre convenance, vous pouvez créer une procédure stockée qui s’exécute dans votre instance SQL et utilise l’instruction **SELEC  - ORDER BY**. Cette solution vous permet de déterminer la taille et la structure de vos résultats. Votre application logique appelle la procédure stockée à l’aide de l’action **Exécuter la procédure stockée** du connecteur SQL Server. Pour plus d'informations, consultez [SELECT - Clause ORDER BY](/sql/t-sql/queries/select-order-by-clause-transact-sql).

  > [!NOTE]
  > Le connecteur SQL a une limite de délai d’expiration de procédure stockée qui est [inférieure à 2 minutes](/connectors/sql/#known-issues-and-limitations). Certaines procédures stockées peuvent demander plus de temps que cette limite n’en accorde, entraînant une erreur `504 Timeout`. Vous pouvez contourner ce problème en utilisant un déclencheur d’achèvement SQL, une requête SQL directe native, une table des états et des travaux côté serveur.
  > 
  > Concernant cette tâche, vous pouvez utiliser l’[agent de travail élastique Azure](../azure-sql/database/elastic-jobs-overview.md) pour [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md). Pour ce qui est de [SQL Server local](/sql/sql-server/sql-server-technical-documentation) et d’[Azure SQL Managed Instance](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md), vous pouvez utiliser le service [SQL Server Agent](/sql/ssms/agent/sql-server-agent). Pour plus d’informations, consultez [Gérer les délais d’attente des procédures stockées de longue durée dans le connecteur SQL pour Azure Logic Apps](../logic-apps/handle-long-running-stored-procedures-sql-connector.md).

### <a name="handle-dynamic-bulk-data"></a>Gérer les données en bloc dynamiques

Parfois, lorsque vous effectuez un appel à une procédure stockée au moyen du connecteur SQL Server, la sortie retournée est dynamique. Dans ce scénario, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Affichez le format de sortie en effectuant une série de tests. Copiez et enregistrez votre exemple de sortie.

1. Dans le concepteur, sous l’action où vous appelez la procédure stockée, sélectionnez **Nouvelle étape**.

1. Sous **Choisir une action** , recherchez et sélectionnez l’action [**Analyser JSON**](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action).

1. Dans l’option **Analyser JSON** , sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**.

1. Dans la zone **Entrer ou coller un exemple de charge utile JSON** , collez votre exemple de sortie, puis sélectionnez **Terminé**.

   > [!NOTE]
   > Si vous recevez une erreur indiquant que Logic Apps ne peut pas générer de schéma, vérifiez que la syntaxe de votre exemple de sortie est correctement mise en forme. Si vous ne pouvez toujours pas générer le schéma dans la zone **Schéma** , entrez manuellement le schéma.

1. Dans la barre d’outils du Concepteur, sélectionnez **Enregistrer**.

1. Pour référencer les propriétés de contenu JSON, cliquez à l’intérieur des zones d’édition où vous souhaitez référencer ces propriétés afin que la liste de contenu dynamique s’affiche. Dans la liste, sous l’en-tête [**Analyser JSON**](../logic-apps/logic-apps-perform-data-operations.md#parse-json-action), sélectionnez les jetons de données pour les propriétés de contenu JSON que vous souhaitez.

## <a name="troubleshoot-problems"></a>Résoudre les problèmes

* Des problèmes de connexion peuvent se produire. Par conséquent, pour dépanner et résoudre ces types de problèmes, consultez [Résolution des erreurs de connexion à SQL Server](https://support.microsoft.com/help/4009936/solving-connectivity-errors-to-sql-server). Voici quelques exemples :

  * `A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections.`

  * `(provider: Named Pipes Provider, error: 40 - Could not open a connection to SQL Server) (Microsoft SQL Server, Error: 53)`

  * `(provider: TCP Provider, error: 0 - No such host is known.) (Microsoft SQL Server, Error: 11001)`

## <a name="connector-specific-details"></a>Détails spécifiques du connecteur

Pour obtenir des informations techniques sur les déclencheurs, les actions et les limites de ce connecteur, consultez la [page de référence du connecteur](/connectors/sql/), qui est générée à partir de la description Swagger.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur [tous les autres connecteurs pour Azure Logic apps](../connectors/apis-list.md)
