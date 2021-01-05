---
title: Déplacer une instance vers une autre région Azure
titleSuffix: Azure Digital Twins
description: Découvrez comment déplacer une instance Azure Digital Twins d’une région Azure à une autre.
author: baanders
ms.author: baanders
ms.date: 08/26/2020
ms.topic: how-to
ms.custom: subject-moving-resources
ms.service: digital-twins
ms.openlocfilehash: 6393b0b8d794345fded95718a2581ae9b929ad49
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94381148"
---
# <a name="move-an-azure-digital-twins-instance-to-a-different-azure-region"></a>Déplacer une instance Azure Digital Twins vers une autre région Azure

Si vous devez déplacer votre instance Azure Digital Twins d’une région à l’autre, la démarche actuelle consiste à recréer vos ressources dans la nouvelle région, puis à supprimer les ressources d’origine. À l’issue de ce processus, vous utiliserez une nouvelle instance Azure Digital Twins identique à la première, à l’exception de l’emplacement mis à jour.

Cet article fournit des conseils sur la mise en œuvre d’un déplacement complet et d’une copie de tout ce dont vous avez besoin pour que la nouvelle instance corresponde à celle d’origine.

Ce processus comprend les étapes suivantes :

1. Préparation : télécharger vos modèles, jumeaux et graphe d’origine.
1. Déplacement : créer une instance Azure Digital Twins dans une nouvelle région.
1. Déplacement : remplir à nouveau l’instance Azure Digital Twins.
    - Charger les modèles, jumeaux et graphe d’origine.
    - Recréer les points de terminaison et les routes.
    - Relier les ressources connectées.
1. Nettoyer les ressources sources : Supprimez l'instance d'origine.

## <a name="prerequisites"></a>Prérequis

Avant d’essayer de recréer votre instance Azure Digital Twins, passez en revue les composants de votre instance d’origine afin de vous faire une idée claire de tous les éléments qui doivent être recréés.

Voici quelques questions à considérer :

* Quels sont les *modèles* chargés dans mon instance ? Combien y en a-t-il ?
* Quels sont les *jumeaux* dans mon instance ? Combien y en a-t-il ?
* Quelle est la forme générale du *graphe* dans mon instance ? Combien de relations y a-t-il ?
* De quels *points de terminaison* est-ce que je dispose dans mon instance ?
* Quelles *routes* figurent dans mon instance ? Sont-elles équipées de filtres ?
* Où mon instance *se connecte-elle aux autres services Azure* ? Certains points d’intégration courants comprennent :

    - Azure Event Grid, Azure Event Hubs ou Azure Service Bus
    - Azure Functions
    - Azure Logic Apps
    - Azure Time Series Insights
    - Azure Maps
    - Service Azure IoT Hub Device Provisioning
* Quelles autres *applications personnelles ou d’entreprise* se connectent à mon instance ?

Vous pouvez collecter ces informations à l’aide du [portail Azure](https://portal.azure.com), des [API et SDK Azure Digital Twins](how-to-use-apis-sdks.md), des [commandes CLI d’Azure Digital Twins](how-to-use-cli.md) ou de l’exemple [Azure Digital Twins (ADT) Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/).

## <a name="prepare"></a>Préparation

Dans cette section, vous allez préparer la recréation de votre instance en téléchargeant vos modèles, jumeaux et graphe d’origine à partir de votre instance de départ. Cet article utilise l’exemple [ADT Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) pour cette tâche.

>[!NOTE]
>Vous avez peut-être déjà des fichiers qui contiennent les modèles ou le graphe dans votre instance. Si c’est le cas, vous n’avez pas besoin de tout retélécharger, mais uniquement ce qui vous manque ou ce qui a été modifié depuis le chargement initial de ces fichiers. Par exemple, vous pouvez avoir des jumeaux qui ont été mis à jour avec de nouvelles données.

### <a name="limitations-of-adt-explorer"></a>Limites d’ADT Explorer

L’[exemple ADT Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/) est un exemple d’application cliente qui prend en charge une représentation visuelle de votre graphe et fournit une interaction visuelle avec votre instance. Cet article explique comment l’utiliser pour télécharger, puis recharger ultérieurement vos modèles, jumeaux et graphes.

Cet exemple n’est pas un outil complet. Il n’a fait l’objet d’aucun test de contrainte et n’est pas conçu pour gérer des graphes de grande taille. Par conséquent, n’oubliez pas les limites suivantes de cet exemple prêt à l’emploi :

* Pour l’instant, l’exemple a été testé uniquement sur des tailles de graphes allant jusqu’à 1 000 nœuds et 2 000 relations.
* L’exemple ne prend pas en charge les nouvelles tentatives en cas de défaillance intermittente.
* L’exemple n’avertit pas nécessairement l’utilisateur si les données chargées sont incomplètes.
* L’exemple ne gère pas les erreurs dues au dépassement des ressources disponibles comme la mémoire par les graphes très volumineux.

Si l’exemple n’est pas en mesure de gérer la taille de votre graphe, vous pouvez exporter et importer ce dernier à l’aide d’autres outils de développement Azure Digital Twins :

* [Commandes CLI Azure Digital Twins](how-to-use-cli.md)
* [API et SDK Azure Digital Twins](how-to-use-apis-sdks.md)

### <a name="set-up-the-adt-explorer-application"></a>Configurer l’application ADT Explorer

Pour continuer avec ADT Explorer, commencez par télécharger l’exemple de code d’application, puis configurez-le pour qu’il s’exécute sur votre machine.

Pour obtenir l’exemple, consultez [ADT Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). Sélectionnez le bouton **Télécharger le zip** pour télécharger sur votre ordinateur un fichier .zip de cet exemple de code nommé **Azure_Digital_Twins__ADT__explorer.zip**. Décompressez le fichier.

Ensuite, définissez et configurez les autorisations pour ADT Explorer. Suivez les instructions de la section [Configurer Azure Digital Twins et ADT Explorer](quickstart-adt-explorer.md#set-up-azure-digital-twins-and-adt-explorer) du guide de démarrage rapide Azure Digital Twins. Cette section vous permet de vous familiariser avec les étapes suivantes :

1. Configurer une instance Azure Digital Twins. Vous pouvez ignorer cette partie, car vous avez déjà une instance.
1. Configurer des informations d’identification Azure locales pour fournir l’accès à votre instance.
1. Exécuter ADT Explorer et le configurer pour qu’il se connecte à votre instance. Vous allez utiliser le *nom d’hôte* de l’instance Azure Digital Twins d’origine que vous déplacez.

L’exemple de l’application ADT Explorer doit désormais s’exécuter dans un navigateur sur votre machine. L’exemple doit être connecté à votre instance Azure Digital Twins d’origine.

:::image type="content" source="media/how-to-move-regions/explorer-blank.png" alt-text="Fenêtre de navigateur présentant une application exécutée sur localhost:3000. L’application s’appelle ADT Explorer. Elle comprend les sections suivantes : Explorateur de requêtes, Vue du modèle, Vue du graphe et Explorateur de propriétés. Il n’y a pas encore de données à l’écran." lightbox="media/how-to-move-regions/explorer-blank.png":::

Afin de vérifier la connexion, vous pouvez sélectionner le bouton **Exécuter la requête** pour exécuter la requête par défaut qui affiche tous les jumeaux et les relations du graphe dans la zone **EXPLORATEUR DU GRAPHE**.

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Un bouton nommé Exécuter la requête dans le coin supérieur droit de la fenêtre est mis en évidence." lightbox="media/how-to-move-regions/run-query.png":::

Vous pouvez laisser ADT Explorer s’exécuter, car vous allez le réutiliser plus loin dans cet article pour recharger ces éléments sur votre nouvelle instance dans la région cible.

### <a name="download-models-twins-and-graph"></a>Télécharger les modèles, les jumeaux et le graphe

Ensuite, téléchargez les modèles, les jumeaux et le graphe dans votre solution sur votre machine.

Pour télécharger tous ces éléments en même temps, vérifiez d’abord que le graphe complet apparaît dans la section **VUE DU GRAPHE**. Si le graphe complet n’apparaît pas déjà, réexécutez la requête par défaut de `SELECT * FROM digitaltwins` dans la section **EXPLORATEUR DE REQUÊTES**.
 
Ensuite, sélectionnez l’icône **Exporter le graphe** dans la section **VUE DU GRAPHE**.

:::image type="content" source="media/how-to-move-regions/export-graph.png" alt-text="Dans la section Vue du graphe, une icône est mise en évidence. On peut y voir une flèche pointant vers le bas, à partir d’un nuage." lightbox="media/how-to-move-regions/export-graph.png":::

Cette action active un lien **Télécharger** dans la section **VUE DU GRAPHE**. Sélectionnez-le pour télécharger une représentation JSON du résultat de la requête incluant vos modèles, jumeaux et relations. Cette action doit permettre de télécharger un fichier .json sur votre machine.

>[!NOTE]
>Si le fichier téléchargé semble présenter une extension de fichier différente, essayez de modifier l’extension directement en la remplaçant par .json.

## <a name="move"></a>Déplacer

Ensuite, vous allez finaliser le déplacement de votre instance en créant une instance dans la région cible. Vous allez ensuite la renseigner avec les données et composants de votre instance d’origine.

### <a name="create-a-new-instance"></a>Créer une instance

Tout d’abord, créez une instance d’Azure Digital Twins dans votre région cible. Effectuez les étapes indiquées dans le [guide pratique Configurer une instance et l’authentification](how-to-set-up-instance-portal.md). Gardez à l’esprit les conseils suivants :

* Vous pouvez conserver le même nom pour la nouvelle instance *si* elle se trouve dans un groupe de ressources différent. Si vous devez utiliser le groupe de ressources contenant votre instance d’origine, la nouvelle instance doit porter son propre nom distinct.
* Lorsque vous êtes invité à indiquer une localisation, entrez la nouvelle région cible.

Cette étape terminée, vous allez avoir besoin du nom d’hôte de votre nouvelle instance pour continuer sa configuration avec vos données. Si vous n’avez pas noté le nom d’hôte pendant la configuration, suivez [ces instructions](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values) pour le récupérer maintenant à partir du portail Azure.

### <a name="repopulate-the-old-instance"></a>Reremplir l’ancienne instance

À présent, vous allez configurer la nouvelle instance pour en faire une copie de l’original.

#### <a name="upload-the-original-models-twins-and-graph-by-using-adt-explorer"></a>Charger les modèles, les jumeaux et le graphe d’origine à l’aide d’ADT Explorer

Dans cette section, vous pouvez recharger vos modèles, jumeaux et graphe dans la nouvelle instance. Si votre instance d’origine ne comporte aucun modèle, jumeau ou graphe, ou si vous ne voulez pas les déplacer vers la nouvelle instance, passez à la [section suivante](#re-create-endpoints-and-routes).

Sinon, revenez à la fenêtre du navigateur qui exécute ADT Explorer et suivez ces étapes.

##### <a name="connect-to-the-new-instance"></a>Se connecter à la nouvelle instance

Pour le moment, ADT Explorer est connecté à votre instance Azure Digital Twins d’origine. Changez la connexion pour pointer vers votre nouvelle instance en sélectionnant le bouton **Se connecter** dans le coin supérieur droit de la fenêtre.

:::image type="content" source="media/how-to-move-regions/sign-in.png" alt-text="ADT Explorer avec mise en évidence de l’icône Se connecter dans le coin supérieur droit de la fenêtre. L’icône représente la silhouette d’une personne à laquelle est superposée une clé." lightbox="media/how-to-move-regions/sign-in.png":::

Remplacez l’**URL ADT** de façon à refléter votre nouvelle instance. Remplacez cette valeur par *https://{nom d’hôte de la nouvelle instance}* .

Sélectionnez **Connecter**. Vous serez peut-être amené à vous reconnecter à l’aide de vos informations d’identification Azure ou à accorder ce consentement d’application pour votre instance.

##### <a name="upload-models-twins-and-graph"></a>Charger les modèles, les jumeaux et le graphe

À présent, dans votre nouvelle instance, chargez les composants de la solution que vous avez téléchargés précédemment.

Pour charger vos modèles, jumeaux et graphe, sélectionnez l’icône **Importer le graphe** dans la section **VUE DU GRAPHE**. Cette option permet de charger les trois composants en même temps. Elle permet même de charger les modèles qui ne sont pas en cours d’utilisation dans le graphe.

:::image type="content" source="media/how-to-move-regions/import-graph.png" alt-text="Dans la section Vue du graphe, une icône est mise en évidence. On peut voir une flèche pointant vers un nuage." lightbox="media/how-to-move-regions/import-graph.png":::

Dans la zone de sélection des fichiers, accédez à votre graphe téléchargé. Sélectionnez le fichier **.json** du graphe, puis **Ouvrir**.

Après quelques secondes, ADT Explorer ouvre la vue **Importer** dans laquelle vous voyez un aperçu du graphe à charger.

Pour confirmer le chargement du graphe, sélectionnez l’icône **Enregistrer** située en haut à droite de la section **VUE DU GRAPHE**.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/graph-preview-save.png" alt-text="Icône Enregistrer mise en évidence dans le volet Aperçu du graphe." lightbox="media/how-to-move-regions/graph-preview-save.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

ADT Explorer charge à présent vos modèles et votre graphe (y compris les jumeaux et les relations) dans votre nouvelle instance Azure Digital Twins. Vous devez voir un message de réussite indiquant le nombre de modèles, de jumeaux et de relations qui ont été chargés.

:::row:::
    :::column:::
        :::image type="content" source="media/how-to-move-regions/import-success.png" alt-text="Boîte de dialogue indiquant la réussite de l’importation du graphe. Le message dit « Importation réussie. 2 modèles ont été importés. 4 jumeaux ont été importés. 2 relations ont été importées. »" lightbox="media/how-to-move-regions/import-success.png":::
    :::column-end:::
    :::column:::
    :::column-end:::
    :::column:::
    :::column-end:::
:::row-end:::

Pour vérifier que tout a été correctement chargé, sélectionnez le bouton **Exécuter la requête** dans la section **EXPLORATEUR DU GRAPHE** pour exécuter la requête par défaut qui affiche l’intégralité des jumeaux et des relations dans le graphe. Cette action actualise aussi la liste des modèles dans la zone **VUE DU MODÈLE**.

:::image type="content" source="media/how-to-move-regions/run-query.png" alt-text="Le bouton Exécuter la requête est mis en évidence dans le coin supérieur droit de la fenêtre." lightbox="media/how-to-move-regions/run-query.png":::

Vous devez voir votre graphe, avec tous ses jumeaux et ses relations affichés dans la zone  **EXPLORATEUR DU GRAPHE**. Vous devez également voir vos modèles listés dans la zone **VUE DU MODÈLE**.

:::image type="content" source="media/how-to-move-regions/post-upload.png" alt-text="Vue d’ADT Explorer mettant en exergue deux modèles dans la zone Vue du modèle et un graphe dans la zone Explorateur du graphe." lightbox="media/how-to-move-regions/post-upload.png":::

Ces vues confirment que vos modèles, jumeaux et graphe ont été rechargés dans la nouvelle instance de la région cible.

#### <a name="re-create-endpoints-and-routes"></a>Recréer les points de terminaison et les routes

Si vous avez des points de terminaison ou des routes dans votre instance d’origine, vous devez les recréer dans votre nouvelle instance. Si votre instance d’origine ne présente aucun point de terminaison et aucune route, ou si vous ne souhaitez pas les déplacer vers la nouvelle instance, vous pouvez passer à la [section suivante](#relink-connected-resources).

Sinon, suivez les étapes décrites dans le [guide pratique Gérer les points de terminaison et les routes](how-to-manage-routes-portal.md) en utilisant la nouvelle instance. Gardez à l’esprit les conseils suivants :

* Vous n’avez *pas* besoin de recréer la ressource Event Grid, Event Hubs ou Service Bus que vous utilisez pour le point de terminaison. Pour plus d’informations, consultez la section « Prérequis » dans les instructions relatives au point de terminaison. Il vous suffit de recréer le point de terminaison sur l’instance Azure Digital Twins.
* Vous pouvez réutiliser les noms des points de terminaison et des routes, car ils sont étendus à une autre instance.
* N’oubliez pas d’ajouter les filtres nécessaires pour les routes que vous créez.

#### <a name="relink-connected-resources"></a>Relier les ressources connectées

Si vous avez d’autres applications ou d’autres ressources Azure connectées à votre instance Azure Digital Twins d’origine, vous devez modifier la connexion afin qu’elles joignent plutôt votre nouvelle instance. Ces ressources peuvent inclure d’autres services Azure, d’autres applications personnelles ou d’entreprise que vous avez configurés pour fonctionner avec Azure Digital Twins.

Si vous n’avez pas d’autres ressources connectées à votre instance d’origine, ou si vous ne souhaitez pas les déplacer vers la nouvelle instance, vous pouvez passer à la [section suivante](#verify).

Sinon, tenez compte des ressources connectées dans votre scénario. Vous n’avez pas besoin de supprimer puis de recréer les ressources connectées. Il vous suffit plutôt de modifier les points où elles se connectent à une instance Azure Digital Twins par le biais de son nom d’hôte. Ensuite, vous mettez à jour ces points pour utiliser le nom d’hôte de la nouvelle instance au lieu de l’original.

Les ressources exactes que vous devez modifier varient en fonction de votre scénario, mais il existe quelques points d’intégration courants que voici :

* Azure Functions. Si vous disposez d’une fonction Azure dont le code comprend le nom d’hôte de l’instance d’origine, vous devez mettre à jour cette valeur avec le nom d’hôte de la nouvelle instance, puis republier la fonction.
* Event Grid, Event Hubs ou Service Bus.
* Logic Apps.
* Time Series Insights.
* Azure Maps.
* Service IoT Hub Device Provisioning.
* Applications personnelles ou d’entreprise en dehors d’Azure, comme l’application cliente créée dans le [Tutoriel : Coder une application cliente](tutorial-code.md), qui se connecte à l’instance et appelle les API Azure Digital Twins.
* Les inscriptions d’applications Azure AD n’ont *pas* besoin d’être recréées. Si vous utilisez une [inscription d’application](how-to-create-app-registration.md) pour vous connecter aux API Azure Digital Twins, vous pouvez réutiliser la même inscription d’application avec votre nouvelle instance.

À l’issue de cette étape, votre nouvelle instance située dans la région cible doit être une copie de l’instance d’origine.

## <a name="verify"></a>Vérification

Pour confirmer que votre nouvelle instance a été correctement configurée, utilisez les outils suivants :

* [Portail Azure](https://portal.azure.com). Le portail est une bonne solution pour vérifier que votre nouvelle instance existe et qu’elle se trouve dans la bonne région cible. Il permet aussi de vérifier les points de terminaison, les routes et les connexions à d’autres services Azure.
* [Commandes CLI Azure Digital Twins](how-to-use-cli.md). Ces commandes sont une bonne solution pour vérifier que votre nouvelle instance existe et qu’elle se trouve dans la bonne région cible. Elles peuvent aussi servir à vérifier les données d’instance.
* [ADT Explorer](/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/). ADT Explorer s’avère pratique pour vérifier des données d’instance, comme les modèles, les jumeaux et les graphes.
* [API et SDK Azure Digital Twins](how-to-use-apis-sdks.md). Ces ressources s’avèrent pratiques pour vérifier des données d’instance, comme les modèles, les jumeaux et les graphes. Elles s’avèrent aussi utiles pour vérifier les points de terminaison et les routes.

Vous pouvez aussi essayer d’exécuter les applications personnalisées ou les flux de bout en bout que vous avez exécutés avec votre instance d’origine, pour mieux vérifier que leur fonctionnement est correct avec la nouvelle instance.

## <a name="clean-up-source-resources"></a>Nettoyer les ressources sources

Maintenant que votre nouvelle instance est configurée dans la région cible, avec une copie des données et des connexions de l’instance originale, vous pouvez supprimer l’instance d’origine.

Vous pouvez utiliser le [portail Azure](https://portal.azure.com), [Azure CLI](how-to-use-cli.md) ou les [API du plan de contrôle](how-to-use-apis-sdks.md#overview-control-plane-apis).

Pour supprimer l’instance à l’aide du portail Azure, [ouvrez le portail](https://portal.azure.com) dans une fenêtre de navigateur et accédez à votre instance Azure Digital Twins d’origine en recherchant son nom dans la barre de recherche du portail.

Sélectionnez le bouton **Supprimer** et suivez les invites pour finaliser la suppression.

:::image type="content" source="media/how-to-move-regions/delete-instance.png" alt-text="Affichage des détails de l’instance Azure Digital Twins dans l’onglet Vue d’ensemble du portail Azure. Le bouton Supprimer est mis en évidence.":::