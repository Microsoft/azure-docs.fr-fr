---
title: Se connecter à Stockage Blob Azure
description: Créer et gérer des objets Blob dans les comptes Stockage Azure avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 02/21/2020
tags: connectors
ms.openlocfilehash: cd23ff0f5ad9912440d38903a344011b069aaf16
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677721"
---
# <a name="create-and-manage-blobs-in-azure-blob-storage-by-using-azure-logic-apps"></a>Créer et gérer des objets Blob dans Stockage Blob Azure avec Azure Logic Apps

Cet article explique comment vous pouvez accéder et gérer les fichiers stockés en tant qu’objets Blob dans votre compte de stockage Azure à partir d’une application logique avec le connecteur Stockage Blob Azure. De cette façon, vous pouvez créer des applications logiques qui automatisent les tâches et les flux de travail pour gérer vos fichiers. Par exemple, vous pouvez générer des applications logiques qui créent, obtiennent, mettent à jour et suppriment des fichiers dans votre compte de stockage.

Supposons que vous disposez d’un outil qui est mis à jour sur un site web Azure. Celui-ci agit comme le déclencheur de votre application logique. Lorsque cet événement se produit, votre application logique peut mettre à jour un fichier dans votre conteneur de stockage Blob, qui est une action dans votre application logique.

Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md). Pour obtenir des informations techniques spécifiques aux connecteurs, consultez la [référence du connecteur Stockage Blob Azure](/connectors/azureblobconnector/).

> [!IMPORTANT]
> Les applications logiques n’ont pas directement accès aux comptes de stockage qui sont derrière des pare-feu dans la même région qu’elles. Pour contourner ce problème, placez vos applications logiques et votre compte de stockage dans des régions différentes. Pour plus d’informations sur l’activation de l’accès à partir d’Azure Logic Apps aux comptes de stockage derrière des pare-feu, consultez la section [Accéder aux comptes de stockage derrière des pare-feu](#storage-firewalls) plus loin dans cette rubrique.

<a name="blob-storage-limits"></a>

## <a name="limits"></a>limites

* Par défaut, les actions de Stockage Blob Azure peuvent lire ou écrire des fichiers dont la taille est *inférieure ou égale à 50 Mo* . Pour gérer les fichiers d’une taille supérieure à 50 Mo, mais n’excédant pas 1 024 Mo, les actions de Stockage Blob Azure prennent en charge la [segmentation du message](../logic-apps/logic-apps-handle-large-messages.md). L’action **Obtenir le contenu de l’objet blob** utilise implicitement la segmentation.

* Les déclencheurs de Stockage Blob Azure ne prennent pas en charge la segmentation. Quand ils demandent du contenu de fichiers, les déclencheurs sélectionnent uniquement des fichiers dont la taille est inférieure ou égale à 50 Mo. Pour obtenir des fichiers supérieurs à 50 Mo, suivez ce modèle :

  * Utilisez un déclencheur Stockage Blob Azure qui retourne des propriétés de fichier, comme **Quand un blob est ajouté ou modifié (propriétés uniquement)** .

  * Suivez le déclencheur avec l’action Stockage Blob Azure **Obtenir le contenu de l’objet blob** , qui lit le fichier complet et utilise implicitement la segmentation.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un [compte de stockage Azure et un conteneur de stockage](../storage/blobs/storage-quickstart-blobs-portal.md)

* L’application logique où vous devez avoir accès à votre compte de stockage d’objets Blob Azure. Pour démarrer votre application logique avec un déclencheur Stockage Blob Azure, vous avez besoin d’une [application logique vide](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="add-trigger"></a>

## <a name="add-blob-storage-trigger"></a>Ajouter un déclencheur de stockage d’objets blob

Dans Azure Logic Apps, chaque application logique doit démarrer avec un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts), qui s’active lorsqu’un événement spécifique se produit ou lorsqu’une condition particulière est remplie. Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une instance d’application logique et lance l’exécution du flux de travail de votre application.

Cet exemple montre comment vous pouvez démarrer un flux de travail d’application logique avec le déclencheur **Quand un blob est ajouté ou modifié (propriétés uniquement)** lorsque les propriétés d’un objet Blob sont ajoutées ou mises à jour dans votre conteneur de stockage.

1. Dans le [portail Azure](https://portal.azure.com) ou Visual Studio, créez une application logique vide, qui ouvre le Concepteur d’applications logiques. Cet exemple utilise le portail Azure.

2. Dans la zone de recherche, saisissez le filtre « blob azure ». Dans la liste des déclencheurs, sélectionnez le déclencheur souhaité.

   Cet exemple utilise ce déclencheur : **Quand un blob est ajouté ou modifié (propriétés uniquement)**

   ![Sélectionner un déclencheur Stockage Blob Azure](./media/connectors-create-api-azureblobstorage/add-azure-blob-storage-trigger.png)

3. Si vous êtes invité à entrer les informations de connexion, [créez votre connexion de stockage d’objets Blob maintenant](#create-connection). Ou bien, si votre connexion existe déjà, fournissez les informations nécessaires pour le déclencheur.

   Dans cet exemple, sélectionnez le conteneur et le dossier que vous souhaitez analyser.

   1. Dans la zone **Conteneur** , sélectionnez l’icône de dossier.

   2. Dans la liste des dossiers, cliquez sur le chevron droit ( **>** ), puis naviguez jusqu’au dossier et sélectionnez-le.

      ![Sélectionner le dossier de stockage à utiliser avec le déclencheur](./media/connectors-create-api-azureblobstorage/trigger-select-folder.png)

   3. Sélectionnez l’intervalle et la fréquence à laquelle vous souhaitez que le déclencheur recherche des modifications dans le dossier.

4. Une fois terminé, dans la barre d’outils du concepteur, choisissez **Enregistrer** .

5. Continuez maintenant à ajouter une ou plusieurs actions à votre application logique pour les tâches à effectuer avec les résultats du déclencheur.

<a name="add-action"></a>

## <a name="add-blob-storage-action"></a>Ajouter une action de stockage d’objets Blob

Dans Azure Logic Apps, une [action](../logic-apps/logic-apps-overview.md#logic-app-concepts) est une étape de votre flux de travail qui suit un déclencheur ou une autre action. Dans cet exemple, l’application logique commence avec le [déclencheur de périodicité](../connectors/connectors-native-recurrence.md).

1. Dans le [portail Azure](https://portal.azure.com) ou dans Visual Studio, ouvrez votre application logique dans le Concepteur d’application logique. Cet exemple utilise le portail Azure.

2. Dans le Concepteur d’application logique, sous le déclencheur ou l’action, sélectionnez **Nouvelle étape** .

   ![Ajouter une nouvelle étape au workflow de l’application logique](./media/connectors-create-api-azureblobstorage/add-new-step-logic-app-workflow.png) 

   Pour ajouter une action entre des étapes, déplacez votre souris sur la flèche de connexion. Choisissez le signe plus ( **+** ) qui s’affiche, puis sélectionnez **Ajouter une action** .

3. Dans la zone de recherche, saisissez le filtre « blob azure ». Dans la liste des actions, sélectionnez l’action souhaitée.

   Cet exemple utilise cette action : **Obtenir le contenu de l’objet blob**

   ![Sélectionner une action Stockage Blob Azure](./media/connectors-create-api-azureblobstorage/add-azure-blob-storage-action.png)

4. Si vous êtes invité à entrer les informations de connexion, [créez votre connexion Stockage Blob Azure maintenant](#create-connection).
Ou bien, si votre connexion existe déjà, fournissez les informations nécessaires pour l’action.

   Dans cet exemple, sélectionnez le fichier souhaité.

   1. Dans la zone **Blob** , sélectionnez l’icône de dossier.
  
      ![Sélectionner le dossier de stockage à utiliser avec l’action](./media/connectors-create-api-azureblobstorage/action-select-folder.png)

   2. Recherchez et sélectionnez le fichier souhaité en fonction de l’ **ID** d’objet Blob. Vous pouvez retrouver cet **ID** dans les métadonnées de l’objet blob qui sont retournées par le déclencheur de stockage d’objets blob décrit précédemment.

5. Une fois terminé, dans la barre d’outils du concepteur, choisissez **Enregistrer** .
Pour tester votre application logique, assurez-vous que le dossier sélectionné contient un objet Blob.

Cet exemple obtient uniquement le contenu d’un objet Blob. Pour afficher le contenu, ajoutez une autre action qui crée un fichier avec l’objet Blob à l’aide d’un autre connecteur. Par exemple, ajoutez une action OneDrive qui crée un fichier basé sur le contenu de l’objet Blob.

<a name="create-connection"></a>

## <a name="connect-to-storage-account"></a>Se connecter à un compte de stockage

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Lorsque vous êtes invité à créer la connexion, fournissez les informations suivantes :

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom de connexion** | Oui | <*connection-name*> | Nom à créer pour votre connexion |
   | **Compte de stockage** | Oui | <*storage-account*> | Sélectionnez votre compte de stockage dans la liste. |
   ||||

   Par exemple :

   ![Créer une connexion de compte de stockage d’objets blob Azure](./media/connectors-create-api-azureblobstorage/create-storage-account-connection.png) 

1. Quand vous êtes prêt, sélectionnez **Créer**

1. Après avoir créé votre connexion, poursuivez avec [Ajouter un déclencheur Stockage Blob](#add-trigger) ou [Ajouter une action Stockage Blob](#add-action).

## <a name="connector-reference"></a>Référence de connecteur

Pour plus d’informations techniques sur ce connecteur, notamment au sujet des déclencheurs, des actions et des limites décrits dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/azureblobconnector/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages de l’ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits) à la place.

<a name="storage-firewalls"></a>

## <a name="access-storage-accounts-behind-firewalls"></a>Accéder à des comptes de stockage derrière des pare-feu

Vous pouvez ajouter la sécurité réseau à un compte de stockage Azure en restreignant l’accès avec un [pare-feu et des règles de pare-feu](../storage/common/storage-network-security.md). Toutefois, cette configuration crée une difficulté pour Azure et d’autres services Microsoft qui ont besoin d’accéder au compte de stockage. La communication locale dans le centre de données fait abstraction des adresses IP internes, vous ne pouvez donc pas configurer de règles de pare-feu avec des restrictions d’adresse IP. Pour plus d’informations, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../storage/common/storage-network-security.md).

Voici différentes options pour accéder aux comptes de stockage derrière des pare-feu à partir d’Azure Logic Apps à l’aide du connecteur Stockage Blob Azure ou d’autres solutions :

* Connecteur d’objet blob de stockage Azure

  * [Accéder aux comptes de stockage dans d’autres régions](#access-other-regions)
  * [Accéder aux comptes de stockage via un réseau virtuel approuvé](#access-trusted-virtual-network)

* Autres solutions

  * [Accéder aux comptes de stockage en tant que service approuvé avec des identités gérées](#access-trusted-service)
  * [Accéder aux comptes de stockage par le biais de la gestion des API Azure](#access-api-management)

<a name="access-other-regions"></a>

### <a name="problems-accessing-storage-accounts-in-the-same-region"></a>Problèmes d’accès aux comptes de stockage situés dans la même région

Les applications logiques n’ont pas directement accès aux comptes de stockage qui sont derrière des pare-feu dans la même région qu’elles. Pour contourner ce problème, placez vos applications logiques dans une région différente de celle de votre compte de stockage et accordez l’accès aux [adresses IP sortantes pour les connecteurs managés dans votre région](../logic-apps/logic-apps-limits-and-config.md#outbound).

> [!NOTE]
> Cette solution ne s’applique pas aux connecteurs Stockage Table Azure et Stockage File d’attente Azure. À la place, pour accéder à votre connecteur Stockage Table ou Stockage File d’attente, utilisez le déclencheur et les actions HTTP intégrés.

<a name="access-trusted-virtual-network"></a>

### <a name="access-storage-accounts-through-a-trusted-virtual-network"></a>Accéder aux comptes de stockage via un réseau virtuel approuvé

Vous pouvez placer le compte de stockage dans un réseau virtuel Azure que vous gérez, puis ajouter ce réseau virtuel à la liste des réseaux virtuels approuvés. Pour que votre application logique accède au compte de stockage via un [réseau virtuel approuvé](../virtual-network/virtual-networks-overview.md), vous devez déployer cette application logique sur un [environnement de service d'intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), qui peut se connecter aux ressources d’un réseau virtuel. Vous pouvez ensuite ajouter les sous-réseaux de cet ISE à la liste approuvée. Les connecteurs de stockage Azure, tels que le connecteur Stockage Blob, peuvent accéder directement au conteneur de stockage. Cette configuration est la même que l’utilisation de points de terminaison de service à partir d’un ISE.

<a name="access-trusted-service"></a>

### <a name="access-storage-accounts-as-a-trusted-service-with-managed-identities"></a>Accéder aux comptes de stockage en tant que service approuvé avec des identités gérées

Pour accorder aux services approuvés Microsoft un accès à un compte de stockage via un pare-feu, vous pouvez définir une exception sur ce compte de stockage pour ces services. Cette solution permet aux services Azure qui prennent en charge les [identités gérées pour l’authentification](../active-directory/managed-identities-azure-resources/overview.md) d’accéder à des comptes de stockage derrière des pare-feu en tant que services approuvés. Plus précisément, pour qu’une application logique dans une instance Azure multi-locataire globale accède à ces comptes de stockage, vous devez d’abord [activer la prise en charge des identités gérées](../logic-apps/create-managed-service-identity.md) sur l’application logique. Ensuite, vous utilisez l’action ou le déclencheur HTTP dans votre application logique et [définissez leur type d’authentification pour utiliser l’identité gérée de votre application logique](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity). Pour ce scénario, vous pouvez utiliser *uniquement* l’action ou le déclencheur HTTP.

Pour configurer l’exception et la prise en charge des identités gérées, suivez ces étapes générales :

1. Sur votre compte de stockage, sous **Paramètres** , sélectionnez **Pare-feu et réseaux virtuels** . Sous **Autoriser l’accès à partir de** , sélectionnez l’option **Réseaux sélectionnés** afin que les paramètres associés s’affichent.

1. Sous **Exceptions** , sélectionnez **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** , puis sélectionnez **Enregistrer** .

   ![Sélectionner une exception qui autorise les services de confiance Microsoft](./media/connectors-create-api-azureblobstorage/allow-trusted-services-firewall.png)

1. Dans les paramètres de votre application logique, [activez la prise en charge de l'identité gérée](../logic-apps/create-managed-service-identity.md).

1. Dans le flux de travail de votre application logique, ajoutez et configurez l’action ou le déclencheur HTTP pour accéder au compte de stockage ou à l’entité.

   > [!IMPORTANT]
   > Pour les appels de déclencheur ou d’action HTTP sortants aux comptes de stockage Azure, assurez-vous que l’en-tête de la demande comprend la propriété `x-ms-version` et la version de l’API pour l’opération que vous souhaitez exécuter sur le compte de stockage. Pour plus d’informations, consultez [Authentifier l’accès avec une identité managée](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity) et [Gestion des versions pour les services Stockage Azure](/rest/api/storageservices/versioning-for-the-azure-storage-services#specifying-service-versions-in-requests).

1. Sur cette action, [sélectionnez l’identité gérée](../logic-apps/create-managed-service-identity.md#authenticate-access-with-managed-identity) à utiliser pour l’authentification.

<a name="access-api-management"></a>

### <a name="access-storage-accounts-through-azure-api-management"></a>Accéder aux comptes de stockage par le biais de la gestion des API Azure

Si vous utilisez un niveau dédié pour la [Gestion des API](../api-management/api-management-key-concepts.md), vous pouvez avoir recours à l’API de stockage en utilisant le service Gestion des API et en autorisant les adresses IP de ce dernier via le pare-feu. En gros, ajoutez le réseau virtuel Azure utilisé par le service Gestion des API au paramètre de pare-feu du compte de stockage. Vous pouvez ensuite utiliser l’action Gestion des API ou l’action HTTP pour appeler les API Stockage Azure. Cela dit, si vous choisissez cette option, vous devez gérer vous-même le processus d’authentification. Pour plus d’informations, consultez [Architecture d’intégration d’entreprise simple](/azure/architecture/reference-architectures/enterprise-integration/basic-enterprise-integration).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les autres [connecteurs d’applications logiques](../connectors/apis-list.md)