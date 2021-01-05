---
title: Créer un test Internet Analyzer avec le portail | Microsoft Docs
description: Dans cet article, vous allez voir comment créer votre premier test Internet Analyzer.
services: internet-analyzer
author: megan-beatty
ms.service: internet-analyzer
ms.topic: tutorial
ms.date: 10/16/2019
ms.author: mebeatty
ms.openlocfilehash: 64bbed9b558d4c20889b28a5247e2113d20daa77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "73509833"
---
# <a name="create-an-internet-analyzer-test-using-portal-preview"></a>Créer un test Internet Analyzer avec le portail (préversion)

Il existe deux façons de créer une ressource Internet Analyzer : avec le portail Azure et avec l’[interface CLI](internet-analyzer-cli.md). Cette section va vous aider à créer une ressource Azure Internet Analyzer à l’aide du portail.

> [!IMPORTANT]
> Cette préversion publique est fournie sans contrat de niveau de service et ne doit pas être utilisée pour les charges de travail de production. Certaines fonctionnalités peuvent ne pas être prises en charge, disposer de capacités limitées ou ne pas être disponibles dans tous les emplacements Azure. Consultez les [Conditions d’utilisation supplémentaires des préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="before-you-begin"></a>Avant de commencer

La préversion publique est disponible dans toutes les régions. Toutefois, le stockage de données est seulement disponible dans la région *USA Ouest 2* dans la préversion.

## <a name="basics"></a>Concepts de base

1. Obtenez un accès à la préversion d’Internet Analyzer en suivant les instructions fournies dans la section **Comment faire pour participer à la préversion ?** des [Questions fréquentes (FAQ) sur Internet Analyzer](internet-analyzer-faq.md).
2. Dans la page d’accueil du [portail Azure](https://preview.portal.azure.com), cliquez sur **+ Créer une ressource**. Internet Analyzer est disponible uniquement dans la préversion du portail Azure.
3. Dans la page **Nouveau**, lancez une recherche sur « Internet Analyzer » à l’aide du champ *Rechercher dans la Place de marché*.
4. Cliquez sur **Internet Analyzer (préversion)** . Assurez-vous que l’éditeur est *Microsoft* et la catégorie *Mise en réseau*.
5. Dans la page **Internet Analyzer (préversion)** , cliquez sur **Créer** pour ouvrir la page **Create an Internet Analyzer** (Créer une ressource Internet Analyzer).
6. Spécifiez les paramètres de configuration suivants pour votre ressource Internet Analyzer :

    * **Abonnement :** Abonnement Azure où est hébergé la nouvelle ressource Internet Analyzer. ***Utilisez le même ID d’abonnement que celui que vous avez utilisé pour demander un accès à la préversion.***
    * **Groupe de ressources :** Groupe de ressources Azure dans lequel la nouvelle ressource Internet Analyzer sera créée. Si vous n’avez pas de groupe de ressources, vous pouvez en créer un.
    * **Nom :** Nom du profil de la nouvelle ressource Internet Analyzer.
    * **Région :** Région publique Azure dans laquelle est créée la ressource. Dans la préversion, seule la région *USA Ouest 2* est disponible.

7. Lorsque vous avez terminé de spécifier les paramètres du profil, cliquez sur **Vérifier + créer**.

## <a name="configuration"></a>Configuration

Il est nécessaire de réaliser les étapes de base avant de configurer un test et d’incorporer le client JavaScript. Une fois que vous avez créé un profil, accédez à **Paramètres > Configuration** pour configurer votre premier test.

1. Donnez un nom à votre test dans la zone **Nom du test**.
2. Ajoutez une description pour votre test dans le champ **Description**.
3. Cliquez sur **Configurer le point de terminaison**. Un onglet va s’afficher sur la droite. Sélectionnez le type de point de terminaison que vous souhaitez configurer : une seule région Azure, plusieurs régions Azure ou un point de terminaison personnalisé.

    >
    ***Points de terminaison préconfigurés : combinaisons de régions seules et de plusieurs régions Azure***
    * Sélectionnez une région ou un ensemble de régions dans une [liste préconfigurée de points de terminaison Azure](internet-analyzer-faq.md).
    * Ensuite, sélectionnez le type d’application ou l’architecture de distribution de contenu que vous souhaitez évaluer.
        * Région Azure seule : Accélération de site ([Azure Front Door](https://azure.microsoft.com/services/frontdoor/)), mise en cache de contenu statique ([Azure CDN pour Microsoft](https://azure.microsoft.com/services/cdn/)) ou aucune.
        * Plusieurs régions Azure : accélération du site ([Azure Front Door](https://azure.microsoft.com/services/frontdoor/)), gestion DNS ([Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/))  

    ***Points de terminaison personnalisés***
    * Suivez les instructions de la page [Créer un point de terminaison personnalisé](internet-analyzer-custom-endpoint.md).
    * Collez l’emplacement de l’URL HTTPS de l’image d’un pixel dans le portail.
    >

4. Cliquez sur **Ajouter** pour ajouter le point de terminaison à votre test.
5. Répétez les étapes 1 à 4 pour configurer votre deuxième point de terminaison. Le point de terminaison B est toujours mesuré par rapport au point de terminaison A. Lorsque vous configurez des points de terminaison, vous devez choisir quel point de terminaison doit être votre contrôle de test.
6. Cliquez sur le bouton **Enregistrer** pour enregistrer votre test. Une fois que vous avez enregistré un test, vous ne pouvez plus modifier ses points de terminaison.
7. Sélectionnez le ou les tests que vous souhaitez démarrer, puis cliquez sur **Démarrer le test**. Cela va faire passer vos tests à l’ ***état*** ***En cours d’exécution***. Vous pouvez démarrer des tests à tout moment. Toutefois, le client JavaScript doit être incorporé pour que le test commence à collecter des mesures.
8. Vous pouvez ajouter d’autres tests à tout moment. Notez que le client JavaScript ne sera pas généré tant qu’un test n’aura pas été créé.

## <a name="embed-client"></a>Incorporer un client

Pour commencer un test, le client JavaScript doit être incorporé dans votre application web. Après avoir configuré au moins un test, cliquez sur **Vérifier + créer**, accédez à **Paramètres > Configuration**, puis copiez le client JavaScript. Pour obtenir des instructions, consultez la page [Incorporer le client Internet Analyzer](internet-analyzer-embed-client.md).  

## <a name="next-steps"></a>Étapes suivantes

* Consultez la [FAQ sur Internet Analyzer](internet-analyzer-faq.md)
* En savoir plus sur l’incorporation du [client Internet Analyzer](internet-analyzer-embed-client.md) et sur la création d’un [point de terminaison personnalisé](internet-analyzer-custom-endpoint.md).
