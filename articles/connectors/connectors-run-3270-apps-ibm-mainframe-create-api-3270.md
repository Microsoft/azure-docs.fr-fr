---
title: Se connecter à des applications 3270 sur des mainframes IBM
description: Intégrer et automatiser des applications avec écran 3270 avec Azure à l’aide d’Azure Logic Apps et du connecteur IBM 3270
services: logic-apps
ms.suite: integration
author: ChristopherHouser
ms.author: chrishou
ms.reviewer: estfan, valthom
ms.topic: article
ms.date: 03/06/2019
tags: connectors
ms.openlocfilehash: ae99e3fa287cc9012e317142cc1e6aef36ce90d6
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93094997"
---
# <a name="integrate-3270-screen-driven-apps-on-ibm-mainframes-with-azure-by-using-azure-logic-apps-and-ibm-3270-connector"></a>Intégrer des applications avec écran 3270 sur des mainframes IBM avec Azure à l’aide d’Azure Logic Apps et du connecteur IBM 3270

> [!NOTE]
> Ce connecteur est en [*préversion publique*](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 

Avec Azure Logic Apps et le connecteur IBM 3270, vous pouvez atteindre et exécuter des applications mainframe IBM que vous pilotez généralement en parcourant les écrans de l’émulateur 3270. De cette façon, vous pouvez intégrer vos applications mainframe IBM avec Azure, Microsoft et d’autres applications, services et systèmes en créant des workflows automatisés avec Azure Logic Apps. Le connecteur communique avec les mainframes IBM en utilisant le protocole TN3270. De plus, il est disponible dans toutes les régions Azure Logic Apps à l’exception d’Azure Government et Azure China 21Vianet. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md)

Cet article décrit les aspects suivants relatifs à l’utilisation du connecteur 3270 : 

* Pourquoi utiliser le connecteur IBM 3270 dans Azure Logic Apps et comment le connecteur exécute des applications avec écran 3270

* Prérequis et configuration lors de l’utilisation du connecteur 3270

* Étapes pour l’ajout d’actions de connecteur 3270 à votre application logique

## <a name="why-use-this-connector"></a>Pourquoi utiliser ce connecteur

Pour accéder aux applications sur des mainframes IBM, vous utilisez généralement un émulateur de terminal 3270, souvent appelé « écran vert ». Cette méthode est un moyen éprouvé mais qui a des limitations. Bien que HIS (Host Integration Server) vous permette de travailler directement avec ces applications, il n’est parfois pas possible de séparer l’écran et la logique métier. Ou il se peut que vous n’ayez plus d’informations sur le fonctionnement des applications hôtes.

Pour étendre ces scénarios, le connecteur IBM 3270 dans Azure Logic Apps fonctionne avec l’outil de conception 3270, que vous utilisez pour enregistrer, ou « capturer », les écrans d’hôte utilisés pour une tâche spécifique, définir le flux de navigation pour cette tâche via votre application mainframe et définir les méthodes avec des paramètres d’entrée et de sortie pour cette tâche. L’outil de conception convertit ces informations en métadonnées que le connecteur 3270 utilise lors de l’appel d’une action qui représente cette tâche à partir de votre application logique.

Une fois que vous avez généré le fichier de métadonnées à partir de l’outil de conception, vous ajoutez ce fichier à un compte d’intégration dans Azure. De cette façon, votre application logique peut accéder aux métadonnées de votre application lorsque vous ajoutez une action de connecteur 3270. Le connecteur lit le fichier de métadonnées à partir de votre compte d’intégration, gère la navigation à travers les écrans 3270 et présente dynamiquement les paramètres pour l’action de connecteur 3270. Vous pouvez ensuite fournir des données à l’application hôte et le connecteur retourne les résultats à votre application logique. De cette façon, vous pouvez intégrer vos applications héritées avec Azure, Microsoft et d’autres applications, services et systèmes pris en charge par Azure Logic Apps.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Des connaissances de base en [création d’applications logiques](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* Recommandé : [Environnement de service d’intégration](../logic-apps/connect-virtual-network-vnet-isolated-environment.md) 

  Vous pouvez sélectionner cet environnement en tant qu’emplacement de création et d’exécution de votre application logique. Une environnement de service d’intégration permet d’accéder à partir de votre application logique à des ressources qui sont protégées à l’intérieur des réseaux virtuels Azure.

* Application logique à utiliser pour l’automatisation et l’exécution de votre application contrôlée par écran 3270

  Comme le connecteur IBM 3270 n’a pas de déclencheur, utilisez un autre déclencheur pour démarrer votre application logique, tel que le déclencheur de **périodicité**. Vous pouvez ensuite ajouter des actions de connecteur 3270. Pour commencer, [créez une application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md). 
  Si vous utilisez un environnement de service d’intégration, sélectionnez ce dernier en tant qu’emplacement de votre application logique.

* [Téléchargez et installez l’outil de conception 3270](https://aka.ms/3270-design-tool-download).
Le seul prérequis est [Microsoft .NET Framework 4.6.1](https://aka.ms/net-framework-download).

  Cet outil vous permet d’enregistrer les écrans, les chemins de navigation, les méthodes et les paramètres pour les tâches dans votre application que vous ajoutez et exécutez en tant qu’actions du connecteur 3270. L’outil génère un fichier HIDX (Host Integration Designer XML) qui fournit les métadonnées nécessaires que le connecteur utilise pour le pilotage de votre application mainframe.
  
  Après le téléchargement et l’installation de cet outil, procédez comme suit pour la connexion à votre hôte :

  1. Ouvrez l’outil de conception 3270. À partir du menu **Session** , sélectionnez **Host Sessions (Sessions hôte)** .
  
  1. Fournissez les informations relatives au serveur hôte TN3270.

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md), qui est l’endroit où vous stockez votre fichier HIDX sous forme de carte afin que votre application logique puisse accéder aux définitions de métadonnées et de méthode dans ce fichier. 

  Assurez-vous que votre compte d’intégration est lié à l’application logique que vous utilisez. De plus, si vous utilisez un environnement de service d’intégration, veillez à ce que l’emplacement de votre compte d’intégration soit dans le même environnement de service d’intégration que celui que votre application logique utilise.

* L’accès au serveur TN3270 qui héberge votre application mainframe.

<a name="define-app-metadata"></a>

## <a name="create-metadata-overview"></a>Créer la vue d’ensemble des métadonnées

Dans une application 3270 pilotée par écran, les écrans et les champs de données sont uniques à vos scénarios. Par conséquent, le connecteur 3270 a besoin des informations relatives à votre application, que vous pouvez fournir en tant que métadonnées. Ces métadonnées décrivent les informations qui aident votre application logique à identifier et à reconnaître les écrans, expliquent comment naviguer entre les écrans, où entrer les données et où attendre des résultats. Pour spécifier et générer ces métadonnées, vous utilisez l’outil de conception 3270, qui vous guide à travers ces *modes* spécifiques, ou étapes, comme décrit plus loin plus en détails :

* **Capture**  : dans ce mode, vous enregistrez les écrans requis pour effectuer une tâche spécifique avec votre application mainframe, par exemple, l’obtention d’un solde de compte.

* **Navigation**  : dans ce mode, vous spécifiez le plan ou le chemin d’accès pour la navigation dans les écrans de votre application mainframe pour la tâche spécifique.

* **Méthodes**  : dans ce mode, vous définissez la méthode, par exemple `GetBalance`, qui décrit le chemin de navigation des écrans. Vous choisissez également les champs de chaque écran qui deviennent les paramètres d’entrée et de sortie de la méthode.

### <a name="unsupported-elements"></a>Éléments non pris en charge

L’outil de conception ne prend pas en charge les éléments suivants :

* Cartes IBM BMS (Basic Mapping Support) partielles : si vous importez une carte BMS, l’outil de conception ignore les définitions d’écran partielles.
* Paramètres d’entrée et de sortie : vous ne pouvez pas définir de paramètres d’entrée et de sortie.
* Traitement des menus : non pris en charge pendant la préversion
* Traitement des groupes : non pris en charge pendant la préversion

<a name="capture-screens"></a>

## <a name="capture-screens"></a>Capturer les écrans

Dans ce mode, vous marquez un élément sur chaque écran 3270 qui identifie de façon unique cet écran. Par exemple, vous pouvez spécifier une ligne de texte ou un ensemble plus complexe de conditions, comme un texte spécifique et un champ non vide. Vous pouvez enregistrer ces écrans sur une connexion active au serveur hôte, ou importer ces informations à partir d’une carte IBM BMS (Basic Mapping Support). La connexion active utilise un émulateur TN3270 pour la connexion à l’hôte. Chaque action de connecteur doit mapper à une seule tâche qui démarre avec la connexion à votre session et se termine avec la déconnexion de votre session.

1. Si ce n’est pas déjà fait, ouvrez l’outil de conception 3270. Dans la barre d’outils, choisissez **Capture** pour passer au mode Capture.

1. Pour démarrer l’enregistrement, appuyez sur la touche F5, ou à partir du menu **Recording (Enregistrement)** , sélectionnez **Start Recording (Démarrer l’enregistrement)** . 

1. Dans le menu **Session** , sélectionnez **Connect (Se connecter)** .

1. Dans le volet **Capture** , à partir du premier écran de votre application, pilotez votre application pour la tâche spécifique que vous enregistrez.

1. Une fois que vous avez terminé la tâche, déconnectez-vous de votre application comme vous le faites habituellement.

1. À partir du menu **Session** , sélectionnez **Disconnect (Se déconnecter)** .

1. Pour arrêter l’enregistrement, appuyez sur les touches Maj+F5, ou à partir du menu **Recording (Enregistrement)** , sélectionnez **Stop Recording (Arrêter l’enregistrement)** .

   Une fois que vous avez capturé les écrans d’une tâche, l’outil de concepteur affiche des miniatures qui représentent ces écrans. Quelques remarques concernant ces miniatures :

   * Parmi vos écrans capturés, vous disposez d’un écran qui est nommé « Empty Vide » (Vide).

     Lorsque vous vous connectez à [CICS](https://www.ibm.com/it-infrastructure/z/cics) pour la première fois, vous devez envoyer la clé « Clear » (Vierge) avant de pouvoir entrer le nom de la transaction que vous souhaitez exécuter. L’écran où vous envoyez la clé « Clear » ne possède aucun *attribut de reconnaissance* , par exemple un titre d’écran, que vous pouvez ajouter à l’aide de l’éditeur de reconnaissance de l’écran. Pour représenter cet écran, la miniature inclut un écran nommé « Empty » (Vide). Vous pouvez utiliser cet écran ultérieurement pour représenter l’écran où vous entrez le nom de la transaction.

   * Par défaut, le nom d’un écran capturé utilise le premier mot à l’écran. Si ce nom existe déjà, l’outil de conception ajoute au nom un trait de soulignement et un nombre, par exemple, « WBGB » et « WBGB_1 ».

1. Pour donner un nom plus significatif à un écran capturé, procédez comme suit :

   1. Dans le volet **Host Screens (Écrans de l’hôte)** , sélectionnez l’écran que vous souhaitez renommer.

   1. Dans le même volet, en bas, recherchez la propriété **Screen Name (Nom de l’écran)** .

   1. Remplacez le nom d’écran actuel par un nom plus descriptif.

1. Maintenant, spécifiez les champs d’identification de chaque écran.

   Avec le flux de données 3270, les écrans n’ont pas d’identificateurs par défaut. Vous devez donc sélectionner du texte unique sur chaque écran. Pour des scénarios complexes, vous pouvez spécifier plusieurs conditions, par exemple, un texte unique et un champ avec une condition spécifique.

Une fois que vous avez terminé de sélectionner les champs de reconnaissance, passez au mode suivant.

### <a name="conditions-for-identifying-repeated-screens"></a>Conditions de l’identification des écrans répétés

Pour que le connecteur parcourt et différencie les écrans, vous trouvez généralement un texte unique sur un écran que vous pouvez utiliser en tant qu’identificateur parmi les écrans capturés. Pour les écrans répétés, vous aurez peut-être besoin de plusieurs méthodes d’identification. Par exemple, supposons que vous avez 2 écrans qui se ressemblent, à ceci près qu’un écran retourne une valeur valide tandis que l’autre écran retourne un message d’erreur.

Dans l’outil de conception, vous pouvez ajouter des *attributs de reconnaissance* , par exemple un titre d’écran tel que « Obtenir le solde du compte », à l’aide de l’éditeur de reconnaissance d’écran. Si vous avez un chemin comprenant des branches et que les deux branches retournent le même écran, mais avec des résultats différents, vous avez besoin d’autres attributs de reconnaissance. Au moment de l’exécution, le connecteur utilise ces attributs pour déterminer la branche actuelle et la branche dupliquée. Voici les conditions utilisables :

* Specific value (Valeur spécifique) : cette valeur correspond à la chaîne spécifiée à l’emplacement spécifié.
* NOT a specific value (PAS une valeur spécifique) : cette valeur ne correspond pas à la chaîne spécifiée à l’emplacement spécifié.
* Empty (Vide) : ce champ est vide.
* NOT empty (NON vide) : ce champ n’est pas vide.

Pour plus d’informations, consultez l’[exemple de plan de navigation](#example-plan) plus loin dans cette rubrique.

<a name="define-navigation"></a>

## <a name="define-navigation-plans"></a>Définir des plans de navigation

Dans ce mode, vous définissez le flux ou les étapes pour parcourir les écrans de votre application mainframe pour votre tâche spécifique. Par exemple, vous pouvez parfois avoir plusieurs chemins que votre application peut suivre, où un chemin produit le résultat correct tandis que l’autre chemin génère une erreur. Pour chaque écran, spécifiez les séquences de touches nécessaires pour le passage à l’écran suivant, telles que `CICSPROD <enter>`.

> [!TIP]
> Si vous automatisez plusieurs tâches qui utilisent les mêmes écrans de connexion et de déconnection, l’outil de conception fournit les types de plan spéciaux Connect (Se connecter) et Disconnect (Se déconnecter). Lorsque vous définissez ces plans, vous pouvez les ajouter au début et à la fin de votre plan de navigation.

### <a name="guidelines-for-plan-definitions"></a>Instructions pour les définitions de plan

* Incluez tous les écrans, en commençant par la connexion et en terminant par la déconnexion.

* Vous pouvez créer un plan autonome ou utiliser les plans Se connecter et Se déconnecter, qui vous permettent de réutiliser une série d’écrans communs à toutes vos opérations.

  * Le dernier écran de votre plan de connexion doit être le même écran que le premier écran de votre plan de navigation.

  * Le premier écran de votre plan de déconnexion doit être le même écran que le dernier écran de votre plan de navigation.

* Vos écrans capturés peuvent contenir de nombreux écrans répétés. Aussi, sélectionnez et utilisez une seule instance des écrans répétés dans votre plan. Voici quelques exemples d’écrans répétés :

  * L'écran de connexion, par exemple l'écran **MSG-10**
  * L’écran d’accueil pour CICS
  * L’écran « Clear » ou **Empty (Vide)**

<a name="create-plans"></a>

### <a name="create-plans"></a>Créer des plans

1. Dans la barre d’outils de l’outil de conception 3270, choisissez **Navigation** afin de passer au mode Navigation.

1. Pour démarrer votre plan, dans le volet **Navigation** , choisissez **New Plan (Nouveau plan)** .

1. Sous **Choose New Plan Name (Choisir un nouveau nom de plan)** , entrez un nom pour votre plan. Dans la liste **Type** , sélectionnez le type de plan :

   | Type de plan | Description |
   |-----------|-------------|
   | **Processus** | Pour des plans autonomes ou combinés |
   | **Connexion** | Pour des plans de connexion |
   | **Déconnexion** | Pour des plans de déconnexion |
   |||

1. Dans le volet **Host Screens (Écrans de l’hôte)** , faites glisser les miniatures capturées dans la surface du plan de navigation dans le volet **Navigation**.

   Pour représenter l’écran vide où vous entrez le nom de la transaction, utilisez l’écran « Vide ».

1. Réorganisez les écrans dans l’ordre qui décrit la tâche que vous définissez.

1. Pour définir le chemin entre des écrans, notamment les branches et les jointures, sur la barre d’outils de l’outil de conception, choisissez **Flow (Flux)** .

1. Choisissez le premier écran dans le flux. Faites glisser et dessinez une connexion à l’écran suivant dans le flux.

1. Pour chaque écran, indiquez les valeurs pour la propriété **AID Key (Clé AID)** (identificateur d’attention) et pour la propriété **Fixed Text (Texte fixe)** , qui déplace le flux à l’écran suivant.

   Vous pouvez avoir uniquement la clé AID, ou à la fois la clé AID et le texte fixe.

Une fois que vous avez terminé votre plan de navigation, vous pouvez [définir des méthodes dans le mode suivant](#define-method).

<a name="example-plan"></a>

### <a name="example"></a>Exemple

Dans cet exemple, supposons que vous exécutez une transaction CICS nommée « WBGB » qui inclut les étapes suivantes : 

* Dans le premier écran, vous entrez un nom et un compte.
* Dans le deuxième écran, vous obtenez le solde du compte.
* Vous quittez l’écran « Vide ».
* Vous vous déconnectez de CICS pour vous connecter à l’écran « MSG-10 ».

Supposons également que vous répétez ces étapes, mais que vous entrez des données incorrectes, capturant ainsi l’écran qui affiche l’erreur. Voici les écrans que vous capturez :

* MSG-10
* CICS Welcome (Bienvenue dans CICS)
* Vide
* WBGB_1 (input) (WBGB_1 (entrée))
* WBGB_2 (error) (WBGB_2 (erreur))
* Empty_1 (Vide_1)
* MSG-10_1

Bien que de nombreux écrans ici aient des noms uniques, certains écrans sont identiques, par exemple « MSG-10 » et « Vide ». Pour un écran répété, utilisez une seule instance de cet écran dans votre plan. Voici des exemples qui illustrent la façon dont un plan autonome, un plan de connexion, un plan de déconnexion et un plan combiné peuvent se présenter :

* Plan autonome

  ![Plan de navigation autonome](./media/connectors-create-api-3270/standalone-plan.png)

* Plan de connexion

  ![Plan de connexion](./media/connectors-create-api-3270/connect-plan.png)

* Plan de déconnexion

  ![Plan de déconnexion](./media/connectors-create-api-3270/disconnect-plan.png)

* Plan combiné

  ![Plan combiné](./media/connectors-create-api-3270/combined-plan.png)

#### <a name="example-identify-repeated-screens"></a>Exemple : Identifier les écrans répétés

Pour que le connecteur parcourt et différencie les écrans, vous trouvez généralement un texte unique sur un écran que vous pouvez utiliser en tant qu’identificateur parmi les écrans capturés. Pour les écrans répétés, vous aurez peut-être besoin de plusieurs méthodes d’identification. L’exemple de plan a une branche où vous pouvez avoir des écrans qui se ressemblent. Un des écrans retourne un solde de compte, tandis que l’autre écran retourne un message d’erreur.

Dans l’outil de conception, vous pouvez ajouter des attributs de reconnaissance, par exemple un titre d’écran tel que « Obtenir le solde du compte », à l’aide de l’éditeur de reconnaissance d’écran. Dans le cas présentant des écrans similaires, vous avez besoin d’autres attributs. Au moment de l’exécution, le connecteur utilise ces attributs pour déterminer la branche et la branche dupliquée.

* Dans la branche qui retourne une entrée valide, qui est l’écran avec le solde du compte, vous pouvez ajouter un champ qui a une condition « non vide ».

* Dans la branche qui retourne une erreur, vous pouvez ajouter un champ qui a une condition « vide ».

<a name="define-method"></a>

## <a name="define-methods"></a>Définir des méthodes

Dans ce mode, vous définissez une méthode qui est associée à votre plan de navigation. Pour chaque paramètre de méthode, vous spécifiez le type de données (chaîne, entier, date ou heure et ainsi de suite). Lorsque vous avez terminé, vous pouvez tester votre méthode sur l’hôte actif et vérifier que la méthode fonctionne comme prévu. Vous générez ensuite le fichier de métadonnées, ou le fichier Host Integration Designer XML (HIDX), qui comporte à présent les définitions de méthode à utiliser pour la création et l’exécution d’une action pour le connecteur IBM 3270.

1. Dans la barre d’outils de l’outil de conception 3270, choisissez **Méthodes** afin de passer au mode Méthodes. 

1. Dans le volet **Navigation** , sélectionnez l’écran qui a les champs d’entrée souhaités.

1. Pour ajouter le premier paramètre d’entrée pour votre méthode, procédez comme suit :

   1. Dans le volet **Capture** , sur l’écran de l’émulateur 3270, choisissez le champ entier, pas uniquement le texte dans le champ, que vous souhaitez comme première entrée.

      > [!TIP]
      > Pour afficher tous les champs et vous assurer que vous sélectionnez le champ complet, dans le menu **View (Vue)** , sélectionnez **All Fields (Tous les champs)** .

   1. Dans la barre d’outils de l’outil de conception, choisissez **Input Field (Champ d’entrée)** . 

   Pour ajouter plus de paramètres d’entrée, répétez les étapes précédentes pour chaque paramètre.

1. Pour ajouter le premier paramètre de sortie pour votre méthode, procédez comme suit :

   1. Dans le volet **Capture** , sur l’écran de l’émulateur 3270, choisissez le champ entier, pas uniquement le texte dans le champ, que vous souhaitez comme première sortie.

      > [!TIP]
      > Pour afficher tous les champs et vous assurer que vous sélectionnez le champ complet, dans le menu **View (Vue)** , sélectionnez **All Fields (Tous les champs)** .

   1. Dans la barre d’outils de l’outil de conception, choisissez **Output Field (Champ de sortie)** .

   Pour ajouter plus de paramètres de sortie, répétez les étapes précédentes pour chaque paramètre.

1. Une fois que vous avez ajouté tous les paramètres de la méthode, définissez ces propriétés pour chaque paramètre :

   | Nom de la propriété | Valeurs possibles | 
   |---------------|-----------------|
   | **Type de données** | Byte, Date Time, Decimal, Int, Long, Short, String |
   | **Field Fill Technique (Technique de remplissage de champ)** | Les paramètres prennent en charge ces types de remplissage, en ajoutant des espaces vides si nécessaire : <p><p>- **Type**  : entrez les caractères séquentiellement dans le champ. <p>- **Fill (Remplir)**  : remplacez le contenu du champ par des caractères, en ajoutant des espaces vides si nécessaire. <p>- **EraseEofType**  : effacez le champ, puis tapez les caractères de manière séquentielle dans le champ. |
   | **Format String (Chaîne de format)** | Certains types de données de paramètre utilisent une chaîne de format, qui indique au connecteur 3270 comment convertir le texte de l’écran en type de données .NET : <p><p>- **DateTime**  : la chaîne de format DateTime suit le [format des chaînes de date et heure personnalisées .NET](/dotnet/standard/base-types/custom-date-and-time-format-strings). Par exemple, la date `06/30/2019` utilise la chaîne de format `MM/dd/yyyy`. <p>- **Decimal**  : la chaîne de format décimal utilise la [clause COBOL Picture](https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_73/rzasb/picture.htm). Par exemple, le nombre `100.35` utilise la chaîne de format `999V99`. |
   |||

## <a name="save-and-view-metadata"></a>Enregistrer et afficher les métadonnées

Une fois que vous avez défini votre méthode, mais avant de la tester, enregistrez toutes les informations que vous avez définies jusqu’à présent dans un fichier RAP (.rap).
Vous pouvez enregistrer ce fichier RAP à tout moment et quel que soit le mode. L’outil de conception inclut également un exemple de fichier de stratégie que vous pouvez ouvrir et consulter en accédant au dossier d’installation de l’outil de conception à cet emplacement et en ouvrant le fichier « WoodgroveBank.rap » :

`..\Program Files\Microsoft Host Integration Server - 3270 Design Tool\SDK\WoodgroveBank.rap`

Toutefois, si vous essayez d’enregistrer les modifications apportées à l’exemple de fichier RAP ou de générer un fichier HIDX à partir de l’exemple de fichier RAP tandis que le fichier reste dans le dossier d’installation de l’outil de conception, vous pouvez obtenir une erreur « accès refusé ». Par défaut, l’outil de conception est installé dans votre dossier Program Files sans autorisations élevées. Si vous obtenez une erreur, essayez l’une de ces solutions :

* Copiez l’exemple de fichier vers un autre emplacement.
* Exécutez l’outil de conception en tant qu’administrateur.
* Définissez-vous comme propriétaire du dossier du SDK.

## <a name="test-your-method"></a>Tester votre méthode

1. Pour exécuter votre méthode sur l’hôte actif, tout en restant dans le mode Méthodes, appuyez sur la touche F5, ou dans la barre d’outils de l’outil de conception, choisissez **Run (Exécuter)** .

   > [!TIP]
   > Vous pouvez changer de mode à tout moment. Dans le menu **File (Fichier)** , sélectionnez **Mode** , puis le mode souhaité.

1. Entrez les valeurs de vos paramètres, puis choisissez **OK**.

1. Pour passer à l’écran suivant, choisissez **Next (Suivant)** .

1. Lorsque vous avez terminé, choisissez **Done (Terminé)** , ce qui a pour effet d’afficher vos valeurs de paramètre de sortie.

<a name="add-metadata-integration-account"></a>

## <a name="generate-and-upload-hidx-file"></a>Générer et charger un fichier HIDX

Lorsque vous êtes prêt, générez le fichier HIDX afin que vous puissiez le charger dans votre compte d’intégration. L’outil de conception 3270 crée le fichier HIDX dans un nouveau sous-dossier où vous avez enregistré votre fichier RAP.

1. Dans la barre d’outils de l’outil de conception 3270, choisissez **Generate Code (Générer le code)** .

1. Accédez au dossier qui contient votre fichier RAP, puis ouvrez le sous-dossier créé par l’outil après la génération de votre fichier HIDX. Vérifiez que l’outil a créé le fichier HIDX.

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis trouvez votre compte d’intégration.

1. Ajoutez votre fichier HIDX sous forme de carte à votre compte d’intégration en [suivant ces étapes pour ajouter des cartes similaires](../logic-apps/logic-apps-enterprise-integration-liquid-transform.md), mais lorsque vous sélectionnez le type de carte, sélectionnez **HIDX**.

Plus loin dans cette rubrique, lorsque vous ajoutez une action IBM 3270 à votre application logique pour la première fois, vous êtes invité à créer une connexion entre votre application logique et le serveur hôte en fournissant des informations de connexion, telles que le nom de votre compte d’intégration et du serveur hôte. Une fois que vous avez créé la connexion, vous pouvez sélectionner votre fichier HIDX précédemment ajouté, la méthode à exécuter et les paramètres à utiliser.

Lorsque vous avez terminé toutes ces étapes, vous pouvez utiliser l’action que vous créez dans votre application logique pour la connexion à votre mainframe IBM, piloter les écrans de votre application, entrer des données, retourner des résultats et ainsi de suite. Vous pouvez également continuer à ajouter d’autres actions à votre application logique pour l’intégration avec d’autres applications, services et systèmes.

<a name="run-action"></a>

## <a name="run-ibm-3270-actions"></a>Exécuter des actions IBM 3270

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com) et ouvrez votre application logique dans le concepteur d’application logique, si elle n’est pas déjà ouverte.

1. Sous la dernière étape où vous souhaitez ajouter une action, choisissez **New step (Nouvelle étape)** , puis sélectionnez **Add an action (Ajouter une action)** . 

1. Sous la zone de recherche, choisissez **Enterprise**. Dans la zone de recherche, entrez 3270 comme filtre. Dans la liste des actions, sélectionnez cette action : **Exécute un programme mainframe via une connexion TN3270**

   ![Sélectionner une action 3270](./media/connectors-create-api-3270/select-3270-action.png)

   Pour ajouter une action entre des étapes, placez votre pointeur au-dessus de la flèche qui les sépare. 
   Cliquez sur le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action**.

1. Si aucune connexion n’existe encore, fournissez les informations nécessaires pour votre connexion, puis choisissez **Create (Créer)** .

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom de connexion** | Oui | <*connection-name*> | Nom de votre connexion |
   | **ID de compte d’intégration** | Oui | <*integration-account-name*> | Nom de votre compte d’intégration |
   | **URL SAP du compte d’intégration** | Oui | <*integration-account-SAS-URL*> | URL de signature d’accès partagé (SAP) de votre compte d’intégration, que vous pouvez générer à partir des paramètres de votre compte d’intégration dans le portail Azure. <p>1. Dans le menu du compte d’intégration, sous **Paramètres** , sélectionnez **URL de rappel**. <br>2. Dans le volet de droite, copiez la valeur **URL de rappel générée**. |
   | **Serveur** | Oui | <*TN3270-server-name*> | Nom du serveur pour votre service TN3270 |
   | **Port** | Non | <*TN3270-server-port*> | Port utilisé par votre serveur TN3270. Si ce champ est vide, le connecteur utilise `23` comme valeur par défaut. |
   | **Type d’appareil** | Non | <*IBM-terminal-model*> | Numéro ou nom de modèle du terminal IBM à émuler. Si ce champ est vide, le connecteur utilise les valeurs par défaut. |
   | **Page de codes** | Non | <*code-page-number*> | Numéro de page de codes pour l’hôte. Si ce champ est vide, le connecteur utilise `37` comme valeur par défaut. |
   | **Nom d’unité logique** | Non | <*logical-unit-name*> | Nom d’unité logique spécifique à demander à partir de l’hôte |
   | **Activer le protocole SSL ?** | Non | Activé ou désactivé | Activez ou désactivez le chiffrement TLS. |
   | **Valider le certificat SSL de l’hôte ?** | Non | Activé ou désactivé | Activez ou désactivez la validation du certificat du serveur. |
   ||||

   Par exemple :

   ![Propriétés de connexion](./media/connectors-create-api-3270/connection-properties.png)

1. Entrez les informations nécessaires pour l’action :

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom d’un fichier Hidx** | Oui | <*HIDX-file-name*> | Sélectionnez le fichier HIDX 3270 à utiliser. |
   | **Nom de la méthode** | Oui | <*method-name*> | Sélectionnez la méthode dans le fichier HIDX à utiliser. Une fois que vous sélectionnez une méthode, la liste **Add new parameter (Ajouter un nouveau paramètre)** s’affiche et vous pouvez alors sélectionner les paramètres à utiliser avec cette méthode. |
   ||||

   Par exemple :

   **Sélectionner le fichier HIDX**

   ![Sélectionner le fichier HIDX](./media/connectors-create-api-3270/select-hidx-file.png)

   **Sélectionner la méthode**

   ![Sélectionner la méthode](./media/connectors-create-api-3270/select-method.png)

   **Sélectionner les paramètres**

   ![Sélectionner les paramètres](./media/connectors-create-api-3270/add-parameters.png)

1. Lorsque vous avez terminé, enregistrez et exécutez votre application logique.

   Lorsque l’exécution de votre application logique est terminée, les étapes de l’exécution apparaissent. 
   Les étapes réussies affichent des coches, alors que les étapes ayant échoué affichent la lettre « X ».

1. Pour réviser les entrées et les sorties de chaque étape, développez l’étape à vérifier.

1. Pour réviser les sorties, choisissez **See raw outputs (Afficher les sorties brutes)** .

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/si3270/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)

