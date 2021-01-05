---
title: 'Démarrage rapide : Recherche interactive sur une carte avec Azure Maps'
description: 'Démarrage rapide : Apprenez à créer des cartes interactives sur lesquelles des recherches peuvent être effectuées. Apprenez à créer un compte Azure Maps, à vous procurer une clé primaire et à utiliser le kit SDK web pour configurer des applications cartographiques'
author: anastasia-ms
ms.author: v-stharr
ms.date: 7/10/2020
ms.topic: quickstart
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: c017ae8044c14a579190f5f1e76cfb1a73e3ce66
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896189"
---
# <a name="quickstart-create-an-interactive-search-map-with-azure-maps"></a>Démarrage rapide : Créer une carte de recherche interactive avec Azure Maps

Cet article explique comment utiliser Azure Maps pour créer une carte offrant aux utilisateurs une expérience de recherche interactive. Il vous guide pas à pas durant ces étapes de base :

* Créer votre propre compte Azure Maps.
* Obtenir votre clé principale à utiliser dans l’application web de démonstration.
* Téléchargez et ouvrez l’application de carte de démonstration.

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

* Connectez-vous au [portail Azure](https://portal.azure.com).

<a id="createaccount"></a>

## <a name="create-an-azure-maps-account"></a>Créer un compte Azure Maps

Créez un compte Azure Maps en effectuant les étapes suivantes :

1. En haut à gauche du [portail Azure](https://portal.azure.com), cliquez sur **Créer une ressource** .
2. Dans la zone *Rechercher dans la Place de marché* , tapez **Azure Maps** .
3. Dans les *Résultats* , sélectionnez **Azure Maps** . Cliquez sur le bouton **Créer** qui s’affiche sous la carte.
4. Sur la page **Créer un compte Maps** , entrez les valeurs suivantes :
    * *L’Abonnement* à utiliser pour ce compte.
    * Le *Groupe de ressources* pour ce compte. Vous pouvez choisir de *Créer* ou d’utiliser un groupe de ressources *Existant* .
    * Le *Nom* de votre nouveau compte.
    * Le *niveau tarifaire* pour ce compte.
    * Lisez la *Licence* et la *Déclaration de confidentialité* , puis cochez la case pour accepter les conditions.
    * Cliquez sur le bouton **Créer** .

    :::image type="content" source="./media/quick-demo-map-app/create-account.png" alt-text="Créer un compte Maps sur le portail":::

<a id="getkey"></a>

## <a name="get-the-primary-key-for-your-account"></a>Obtenir la clé primaire de votre compte

Une fois le compte Maps créé, récupérez la clé principale qui vous permet d’interroger les API Maps.

1. Ouvrez votre compte Maps dans le portail.
2. Dans la section des paramètres, sélectionnez **Authentification** .
3. Copiez la **Clé primaire** dans le Presse-papiers. Enregistrez-la localement, vous l’utiliserez plus tard dans ce didacticiel.

>[!NOTE]
> Si vous utilisez la clé d’abonnement au lieu de la clé principale, votre mappage ne s’affiche pas correctement. En outre, pour des raisons de sécurité, il est recommandé de permuter vos clés principale et secondaire. Pour permuter les clés, mettez à jour votre application pour utiliser la clé secondaire, déployez, puis appuyez sur le bouton cycle/actualiser en regard de la clé principale pour générer une nouvelle clé principale. L’ancienne clé principale est désactivée. Pour plus d’informations sur la permutation des clés, consultez [Configurer Azure Key Vault avec la permutation des clés et l’audit](../key-vault/secrets/tutorial-rotation-dual.md)

:::image type="content" source="./media/quick-demo-map-app/get-key.png" alt-text="Créer un compte Maps sur le portail":::

## <a name="download-the-demo-application"></a>Télécharger l’application de démonstration

1. Accédez à [interactiveSearch.html](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/interactiveSearch.html). Copiez le contenu du fichier.
2. Enregistrez le contenu de ce fichier localement sous le nom **AzureMapDemo.html** . Ouvrez le fichier dans un éditeur de texte.
3. Recherchez la chaîne `<Your Azure Maps Key>`. Remplacez-la par la valeur de **clé primaire** obtenue dans la section précédente.

## <a name="open-the-demo-application"></a>Ouvrir l’application de démonstration

1. Ouvrez le fichier **AzureMapDemo.html** dans le navigateur de votre choix.
2. Observez la carte de la ville de Los Angeles. Effectuez un zoom avant et arrière : la carte affiche automatiquement plus ou moins d’informations selon le niveau de zoom.
3. Modifiez le centre par défaut de la carte. Dans le fichier **AzureMapDemo.html** , recherchez la variable nommée **center** . Remplacez la paire de valeurs longitude, latitude de cette variable par les nouvelles valeurs : **[-74.0060, 40.7128]** . Enregistrez le fichier et actualisez votre navigateur.
4. Testez l’expérience de recherche interactive. Dans la zone de recherche en haut à gauche de l’application web de démonstration, recherchez **restaurants** .
5. Déplacez votre souris sur la liste des adresses et emplacements qui s’affichent sous la zone de recherche. Le repère correspondant sur la carte affiche des informations à propos de cet emplacement. Pour des raisons de confidentialité des entreprises privées, des adresses et des noms fictifs sont présentés.

    :::image type="content" source="./media/quick-demo-map-app/interactive-search.png" alt-text="Créer un compte Maps sur le portail":::


## <a name="clean-up-resources"></a>Nettoyer les ressources

>[!WARNING]
>Les tutoriels répertoriés dans la section [Étapes suivantes](#next-steps) expliquent en détail comment utiliser et configurer Azure Maps avec votre compte. Si vous souhaitez suivre les tutoriels, ne nettoyez pas les ressources créées dans ce guide de démarrage rapide.

Sinon, effectuez ces étapes pour nettoyer les ressources :

1. Fermez le navigateur qui exécute l’application web **AzureMapDemo.html** .
2. Accédez à la page du portail Azure. Sélectionnez **Toutes les ressources** dans la page principale du portail. Ou cliquez sur l’icône de menu dans le coin supérieur gauche. Sélectionnez **Toutes les ressources** .
3. Cliquez sur votre compte Azure Maps. En haut de la page, cliquez sur **Supprimer** .

Pour obtenir d’autres exemples de code et bénéficier d’une expérience de codage interactive, consultez ces guides :

[Rechercher une adresse avec le service Recherche Azure Maps](how-to-search-for-address.md)

[Utiliser la bibliothèque Map Control d’Azure Maps](how-to-use-map-control.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé votre compte Azure Maps ainsi qu’une application de démonstration. Pour en savoir plus sur Azure Maps, consultez les tutoriels suivants :

> [!div class="nextstepaction"]
> [Rechercher des points d’intérêt de proximité avec Azure Maps](tutorial-search-location.md)