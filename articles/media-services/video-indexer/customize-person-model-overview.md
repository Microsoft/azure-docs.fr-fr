---
title: Personnaliser un modèle de personne dans Video Indexer - Azure
titleSuffix: Azure Media Services
description: Cet article présente les modèles de personne dans Video Indexer et explique comment le personnaliser.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: kumud
ms.openlocfilehash: 613ad2d0ab90bef016e80a33be12b71cba5e2f71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87047055"
---
# <a name="customize-a-person-model-in-video-indexer"></a>Personnaliser un modèle de personne dans Video Indexer

Video Indexer prend en charge la reconnaissance de célébrités dans du contenu vidéo. La fonctionnalité de reconnaissance de célébrités couvre environ un million de visages en fonction de la source de données couramment demandée comme IMDB, Wikipedia et les principaux influenceurs LinkedIn. Les visages qui ne sont pas reconnus par Video Indexer sont tout de même détectés, mais sont laissés sans nom. Les clients peuvent créer des modèles de personne personnalisés et activer Video Indexer pour identifier des visages qui ne sont pas reconnus par défaut. Les clients peuvent créer ces modèles de personne en associant le nom d’une personne avec les fichiers d’image du visage de la personne.  

Si votre compte est adapté à différents scénarios d’utilisation, vous pouvez peut-être tirer parti de la création de plusieurs modèles de personne par compte. Par exemple, si le contenu de votre compte est destiné à être trié dans différents canaux, vous souhaiterez peut-être créer un modèle de personne distinct pour chaque canal. 

> [!NOTE]
> Chaque modèle de personne prend en charge jusqu’à 1 million de personnes, et chaque compte a une limite de 50 modèles de personne. 

Une fois un modèle créé, vous pouvez l’utiliser en fournissant l’ID d’un modèle de personne spécifique lors du chargement /de l’indexation ou de la réindexation d’une vidéo. La formation du modèle sur un nouveau visage dans une vidéo met à jour le modèle personnalisé spécifique auquel la vidéo était associée. 

Si vous n’avez pas besoin de la prise en charge de plusieurs modèles de personne, n’affectez aucun ID de modèle de personne à votre vidéo lors de son chargement/de son indexation ou de sa réindexation. Dans ce cas, Video Indexer utilise le modèle de personne par défaut dans votre compte. 

Vous pouvez utiliser le site web de Video Indexer pour modifier les visages qui ont été détectés dans une vidéo et pour gérer plusieurs modèles de personne personnalisés dans votre compte, comme indiqué dans la section [Personnaliser un modèle de personne avec le site web Video Indexer](customize-person-model-with-website.md). Vous pouvez également utiliser l’API, comme décrit dans la section  [Personnaliser un modèle de personne avec l’API Video Indexer](customize-person-model-with-api.md).
