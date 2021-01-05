---
title: Recette d’instance de conteneur Azure
titleSuffix: Azure Cognitive Services
description: Apprendre à déployer des conteneurs Cognitive Services dans une instance de conteneur Azure
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 8ddaed181d017e3167694a9d7edf53c7c09fd5e9
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968517"
---
# <a name="deploy-and-run-container-on-azure-container-instance"></a>Déployer et exécuter un conteneur sur Azure Container Instances

Les étapes suivantes permettent de mettre facilement à l’échelle des applications Azure Cognitive Services dans le cloud avec [Azure Container Instances](../../container-instances/index.yml). La conteneurisation vous aide à vous concentrer sur la création de vos applications au lieu de gérer l’infrastructure. Pour plus d’informations sur l’utilisation de conteneurs, voir [Fonctionnalités et avantages](../cognitive-services-container-support.md#features-and-benefits).

## <a name="prerequisites"></a>Prérequis

Cette recette fonctionne avec n’importe quel conteneur Cognitive Services. Vous devez créer la ressource Cognitive Services sur le portail Azure avant d’utiliser la recette. Chaque service cognitif qui prend en charge les conteneurs propose un document d’aide qui vous explique comment installer et configurer le service pour un conteneur. Certains services nécessitant un fichier ou un ensemble de fichiers en entrée pour le conteneur, avant d’utiliser cette solution, il est important d’avoir compris le fonctionnement du conteneur et de l’avoir déjà utilisé avec succès.

* Une ressource de service cognitif, créée sur le portail Azure.
* Une **URL de point de terminaison** de service cognitif : examinez la procédure d’installation du conteneur propre à votre service de façon à trouver l’emplacement de l’URL du point de terminaison sur le portail Azure ainsi qu’un exemple approprié de présentation de l’URL. Le format exact peut changer d’un service à l’autre.
* **Clé** du service cognitif : les clés se trouvent dans la page **Clés** de la ressource Azure. Vous avez seulement besoin d’une des deux clés. La clé est une chaîne de 32 caractères alphanumériques.
* Un seul conteneur Cognitive Services sur votre hôte local (votre ordinateur). Vérifiez que vous pouvez :
  * Extraire l’image à l’aide d’une commande `docker pull`.
  * Exécuter correctement le conteneur local avec tous les paramètres de configuration nécessaires à l’aide d’une commande `docker run`.
  * Appeler le point de terminaison du conteneur, pour obtenir une réponse de HTTP 2xx et une réponse JSON en retour.

Toutes les variables entre chevrons, `<>`, doivent être remplacées par vos propres valeurs. Ce remplacement inclut les chevrons.

[!INCLUDE [Create a Text Analytics Containers on Azure Container Instances](includes/create-container-instances-resource.md)]

## <a name="use-the-container-instance"></a>Utiliser l’instance de conteneur

1. Sélectionnez **Vue d’ensemble** et copiez l’adresse IP. Il s’agit d’une adresse IP numérique telle que `55.55.55.55`.
1. Ouvrez un nouvel onglet de navigateur et utilisez l’adresse IP, par exemple, `http://<IP-address>:5000 (http://55.55.55.55:5000`). La page d’accueil du conteneur s’affiche en vous informant que le conteneur est en cours d’exécution.

1. Sélectionnez **Description de l’API de service** pour afficher la page swagger pour le conteneur.

1. Sélectionnez une des API **POST**, puis choisissez **Essayer**.  Les paramètres s’affichent, y compris l’entrée. Complétez les paramètres.

1. Sélectionnez **Exécuter** pour envoyer la demande à votre instance de conteneur.

    Vous êtes parvenu à créer et utiliser des conteneurs Cognitive Services dans une instance de conteneur Azure.