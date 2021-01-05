---
title: Se connecter à Oracle Database
description: Insérer et gérer des enregistrements avec les API REST Oracle Database et Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 05/20/2020
tags: connectors
ms.openlocfilehash: 91873a2d6a498712773bfe721653e64c3364666f
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674822"
---
# <a name="get-started-with-the-oracle-database-connector"></a>Prise en main du connecteur de base de données Oracle

À l’aide du connecteur de base de données Oracle, vous créez des workflows d’organisation qui utilisent des données de votre base de données existante. Ce connecteur peut se connecter à une base de données Oracle locale ou à une machine virtuelle sur laquelle la base de données Oracle est installée. Avec ce connecteur, vous pouvez :

* Créez votre flux de travail en ajoutant un nouveau client à une base de données clients ou en mettant à jour une commande dans une base de données de commandes.
* Utilisez des actions pour obtenir une ligne de données, insérer une nouvelle ligne ou en supprimer. Par exemple, quand un enregistrement est créé dans Dynamics CRM Online (déclencheur), insérez une ligne dans une base de données Oracle (action). 

Ce connecteur ne prend pas en charge les éléments suivants :

* Les vues 
* Toute table avec des clés composites
* Types d’objet imbriqués dans des tables
* Fonctions de base de données avec des valeurs non scalaires

Cet article décrit comment utiliser le connecteur de base de données Oracle dans une application logique.

## <a name="prerequisites"></a>Prérequis

* Versions d’Oracle prises en charge : 
    * Oracle 9 et versions ultérieures
    * Logiciel client Oracle 8.1.7 et versions ultérieures

* Installez la passerelle de données locale. [Connexion à des données locales à partir d’applications logiques](../logic-apps/logic-apps-gateway-connection.md) répertorie les étapes à suivre. La passerelle vous permet de vous connecter à une base de données Oracle locale ou à une machine virtuelle Azure sur laquelle la base de données Oracle est installée. 

    > [!NOTE]
    > La passerelle de données locale agit comme un pont et fournit un transfert de données sécurisé entre les données locales (qui ne sont pas sur le cloud) et vos applications logiques. La même passerelle peut être utilisée avec plusieurs services et plusieurs sources de données.  Par conséquent, il vous suffira peut-être d’installer la passerelle une seule fois.

* Installez le client Oracle sur l’ordinateur où vous avez installé la passerelle de données locale.  Veillez à installer le fournisseur de données Oracle 64 bits pour .NET à partir d’Oracle :  

  [ODAC 12C version 4 (12.1.0.2.4) 64 bits pour Windows x64](https://www.oracle.com/technetwork/database/windows/downloads/index-090165.html)

    > [!TIP]
    > Si le client Oracle n’est pas installé, une erreur se produit lorsque vous tentez de créer ou d’utiliser la connexion. Consultez les erreurs courantes dans cet article.


## <a name="add-the-connector"></a>Ajouter le connecteur

> [!IMPORTANT]
> Ce connecteur ne possède aucun déclencheur. Il possède uniquement des actions. Par conséquent, lorsque vous créez votre application logique, ajoutez un autre déclencheur pour démarrer votre application logique, comme **Planification - Périodicité** ou **Requête/réponse - Réponse** . 

1. Dans le [portail Azure](https://portal.azure.com), créez une application logique vide.

2. Au lancement de votre application logique, sélectionnez le déclencheur **Requête/réponse - Requête**  : 

    ![Une boîte de dialogue comporte une zone permettant d’effectuer une recherche dans tous les déclencheurs. Il y a également un seul déclencheur, « Requête / Réponse-Requête », avec un bouton de sélection.](./media/connectors-create-api-oracledatabase/request-trigger.png)

3. Sélectionnez **Enregistrer** . Au moment de l’enregistrement, une URL de requête est générée automatiquement. 

4. Sélectionnez **Nouvelle étape** , puis sélectionnez **Ajouter une action** . Entrez `oracle` pour afficher les actions disponibles : 

    ![Une zone de recherche contient « Oracle ». La recherche produit un accès intitulé « Oracle Database ». Il y a une page avec onglets, un onglet indiquant « DÉCLENCHEURS (0) » et un autre indiquant « ACTIONS (6) ». Six actions sont répertoriées. La première consiste à « obtenir l’aperçu de la ligne ».](./media/connectors-create-api-oracledatabase/oracledb-actions.png)

    > [!TIP]
    > C’est également le moyen le plus rapide de voir les déclencheurs et les actions disponibles pour n’importe quel connecteur. Entrez une partie du nom du connecteur, comme `oracle`. Le concepteur répertorie tous les déclencheurs et toutes les actions. 

5. Sélectionnez l’une des actions, comme **Base de données Oracle - Obtenir une ligne** . Sélectionnez l’option **Se connecter via la passerelle de données locale** . Entrez le nom du serveur Oracle, la méthode d’authentification, le nom d’utilisateur et le mot de passe, puis sélectionnez la passerelle :

    ![La boîte de dialogue est intitulée « Oracle Database - Obtenir une ligne ». Une case intitulée « Se connecter via une passerelle de données locale » est cochée. Vous trouverez ci-dessous les cinq autres zones de texte.](./media/connectors-create-api-oracledatabase/create-oracle-connection.png)

6. Une fois connecté, sélectionnez une table dans la liste et entrez l’ID de ligne pour votre table. Vous devez connaître l’identificateur de la table. Si vous ne le connaissez pas, contactez votre administrateur de base de données Oracle et récupérez la sortie à partir de `select * from yourTableName`. Vous disposerez ainsi des informations d’identification nécessaires pour continuer.

    Dans l’exemple suivant, les données de travail sont renvoyées à partir d’une base de données Ressources humaines : 

    ![La boîte de dialogue intitulée « Obtenir une ligne (Aperçu) » comporte deux zones de texte : « Nom de la table », qui contient « HRJOBS » et présente une liste déroulante, et « ID de la ligne », qui contient « SA_REP ».](./media/connectors-create-api-oracledatabase/table-rowid.png)

7. Dans l’étape suivante, vous pouvez utiliser l’un des autres connecteurs pour créer votre workflow. Si vous souhaitez tester l’obtention de données à partir d’Oracle, envoyez-vous un e-mail contenant les données Oracle à l’aide de l’un des connecteurs d’envoi d’e-mails, comme Office 365 ou Outlook. Utilisez les jetons dynamiques de la table Oracle pour créer les éléments `Subject` et `Body` de votre message électronique :

    ![Il existe deux boîtes de dialogue. La boîte de dialogue « Envoyer un e-mail » contient des cases permettant de spécifier le corps, l’objet et l’adresse du destinataire de l’e-mail. La boîte de dialogue « Ajouter du contenu dynamique » permet de rechercher du contenu dynamique à partir des applications et des services du flux.](./media/connectors-create-api-oracledatabase/oracle-send-email.png)

8. **Enregistrez** votre application logique, puis sélectionnez **Exécuter** . Fermez le concepteur et consultez l’état de l’historique des exécutions. En cas d’échec, sélectionnez la ligne du message ayant échoué. Le concepteur s’ouvre et affiche l’étape qui a échoué ainsi que les informations d’erreur. En cas de réussite, vous devez recevoir un courrier électronique contenant les informations que vous avez ajoutées.


### <a name="workflow-ideas"></a>Idées de workflow

* Vous souhaitez surveiller le hashtag #oracle et inclure les tweets dans une base de données afin de pouvoir les consulter et les utiliser dans d’autres applications. Dans une application logique, ajoutez le déclencheur `Twitter - When a new tweet is posted`, puis entrez le hashtag **#oracle** . Ajoutez ensuite l’action `Oracle Database - Insert row`, puis sélectionnez votre table :

    ![La boîte de dialogue « Lorsqu’un nouveau tweet est publié » affiche « mot-dièse Oracle » comme texte de recherche et vous permet de spécifier la fréquence de vérification. Cette boîte de dialogue ouvre la boîte de dialogue « Oracle Database » qui vous permet de sélectionner l’action.](./media/connectors-create-api-oracledatabase/twitter-oracledb.png)

* Les messages sont envoyés à une file d’attente Service Bus. Vous souhaitez récupérer ces messages et les inclure dans une base de données. Dans une application logique, ajoutez le déclencheur `Service Bus - when a message is received in a queue`, puis sélectionnez la file d’attente. Ajoutez ensuite l’action `Oracle Database - Insert row`, puis sélectionnez votre table :

    ![La boîte de dialogue « Lorsqu’un message est reçu... » affiche des « ordres » comme « Nom de la file d’attente » et vous permet de spécifier la fréquence de vérification. Cette boîte de dialogue ouvre la boîte de dialogue « Insérer une ligne (Aperçu) » qui vous permet de sélectionner « Nom de la table ».](./media/connectors-create-api-oracledatabase/sbqueue-oracledb.png)

## <a name="common-errors"></a>Erreurs courantes

#### <a name="error-cannot-reach-the-gateway"></a>**Erreur**  : La passerelle est injoignable.

**Cause**  : La passerelle de données locale n’est pas en mesure de se connecter au cloud. 

**Atténuation**  : Assurez-vous que votre passerelle s’exécute sur l’ordinateur local où vous l’avez installée, et qu’elle peut se connecter à Internet.    Nous vous recommandons ne pas installer la passerelle sur un ordinateur susceptible d’être mis hors tension ou en veille.  Vous pouvez également redémarrer le service de passerelle de données locale (PBIEgwService).

#### <a name="error-the-provider-being-used-is-deprecated-systemdataoracleclient-requires-oracle-client-software-version-817-or-greater-see-httpsgomicrosoftcomfwlinkplinkid272376-to-install-the-official-provider"></a>**Erreur**  : Le fournisseur utilisé est déconseillé : System.Data.OracleClient requiert le logiciel client Oracle version 8.1.7 ou supérieure. Consultez [https://go.microsoft.com/fwlink/p/?LinkID=272376](/power-bi/connect-data/desktop-connect-oracle-database) pour installer le fournisseur officiel.

**Cause**  : Le kit de développement du client Oracle n’est pas installé sur l’ordinateur qui exécute la passerelle de données locale.  

**Résolution**  : Téléchargez et installez le kit de développement logiciel du client Oracle sur le même ordinateur que la passerelle de données locale.

#### <a name="error-table-tablename-does-not-define-any-key-columns"></a>**Erreur**  : La table « [Tablename] » ne définit aucune colonne clé.

**Cause**  : La table n’a pas de clé primaire.  

**Résolution**  : Le connecteur Oracle Database requiert une table avec une colonne clé primaire.
 
## <a name="connector-specific-details"></a>Détails spécifiques du connecteur

Consultez l’ensemble des déclencheurs et actions définis dans le swagger, ainsi que les éventuelles limites dans les [détails des connecteurs](/connectors/oracle/). 

## <a name="get-some-help"></a>Obtenir de l’aide

La [page de questions Microsoft Q&A pour Azure Logic Apps](/answers/topics/azure-logic-apps.html) est l’endroit idéal pour poser des questions, répondre aux questions posées et découvrir ce que font les autres utilisateurs de Logic Apps. 

Vous pouvez améliorer Logic Apps et les connecteurs en votant et en soumettant vos idées sur [https://aka.ms/logicapps-wish](https://aka.ms/logicapps-wish). 


## <a name="next-steps"></a>Étapes suivantes
[Créer une application logique](apis-list.md) et explorez les connecteurs disponibles dans Logic Apps sur la [liste d’API](../logic-apps/quickstart-create-first-logic-app-workflow.md).