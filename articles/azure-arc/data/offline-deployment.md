---
title: Déploiement hors connexion
description: Le déploiement hors connexion vous permet d’extraire les images conteneur d’un registre de conteneurs privé au lieu de les extraire de Microsoft Container Registry.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 575903654a165bef0d09ac6abf0793af3f6784e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930745"
---
# <a name="offline-deployment-overview"></a>Vue d’ensemble du déploiement hors connexion

En général, les images conteneur utilisées lors de la création du contrôleur de données Azure Arc, de la création des instances managées SQL et des groupes de serveurs PostgreSQL Hyperscale sont directement extraites de Microsoft Container Registry (MCR). Dans certains cas, l’environnement sur lequel vous effectuez le déploiement ne dispose pas d’une connectivité à Microsoft Container Registry.  Dans une telle situation, vous pouvez extraire les images conteneur à l’aide d’un ordinateur qui _a_ accès au Registre de conteneurs Microsoft, puis les étiqueter et les envoyer (push) à un registre de conteneurs privé, qui _est_ connectable depuis l’environnement dans lequel vous souhaitez déployer des services de données activés pour Azure Arc.

Étant donné que les mises à jour mensuelles sont fournies pour les services de données Azure Arc et qu’il existe un grand nombre d’images conteneur, il est préférable d’utiliser un script pour effectuer ces processus d’extraction, d’étiquetage et d’envoi (push) des images conteneur vers un registre de conteneurs privé.  Le script peut être automatisé ou exécuté manuellement.

Un [exemple de script](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/scripts/pull-and-push-arc-data-services-images-to-private-registry.py) se trouve dans le dépôt GitHub Azure Arc.

> [!NOTE]
> Ce script nécessite l’installation de Python et de l’interface [Docker CLI](https://docs.docker.com/install/).

Le script vous invite à entrer les informations suivantes de manière interactive.  Si vous souhaitez que le script s’exécute sans invite interactive, vous pouvez également définir les variables d’environnement correspondantes avant d’exécuter le script.

|Prompt|Variable d’environnement|Notes|
|---|---|---|
|Fournissez le registre de conteneurs source ; appuyez sur Entrée pour utiliser `mcr.microsoft.com`|SOURCE_DOCKER_REGISTRY|En règle générale, vous extrayez les images de Microsoft Container Registry, mais si vous participez à une préversion privée avec un registre différent, vous pouvez utiliser les informations qui vous sont fournies dans le cadre du programme en préversion.|
|Fournissez le référentiel du registre de conteneurs source ; appuyez sur Entrée pour utiliser `arcdata` :|SOURCE_DOCKER_REPOSITORY|Si vous effectuez une extraction à partir de Microsoft Container Registry, le référentiel est `arcdata`.|
|Fournissez le nom d’utilisateur pour le registre de conteneurs source ; appuyez sur Entrée pour n’en utiliser aucun :|SOURCE_DOCKER_USERNAME|Fournissez une valeur uniquement si vous extrayez des images conteneur à partir d’une source qui nécessite une connexion.  Microsoft Container Registry ne nécessite pas de connexion.|
|Fournissez le mot de passe pour le registre de conteneurs source ; appuyez sur Entrée pour n’en utiliser aucun :|SOURCE_DOCKER_PASSWORD|Fournissez une valeur uniquement si vous extrayez des images conteneur à partir d’une source qui nécessite une connexion.  Microsoft Container Registry ne nécessite pas de connexion. Il s’agit d’une invite de mot de passe masqué.  Vous ne voyez pas le mot de passe si vous le tapez ou le collez dedans.|
|Fournissez une étiquette d’image conteneur pour les images à la source ; appuyez sur Entrée pour utiliser `<current monthly release tag>` :|SOURCE_DOCKER_TAG|Le nom d’étiquette par défaut est mis à jour mensuellement pour refléter le mois et l’année de la version actuelle dans Microsoft Container Registry.|
|Fournissez le nom DNS ou l’adresse IP du registre de conteneurs cible :|TARGET_DOCKER_REGISTRY|Nom DNS ou l’adresse IP du registre de conteneurs cible.  Il s’agit du registre _vers_ lequel les images seront envoyées.|
|Fournissez le référentiel du registre de conteneurs cible :|TARGET_DOCKER_REGISTRY|Référentiel dans le registre cible vers lequel envoyer (push) les images.|
|Fournissez le nom d’utilisateur pour le registre de conteneurs cible ; appuyez sur Entrée pour n’en utiliser aucun :|TARGET_DOCKER_USERNAME|Nom d’utilisateur, le cas échéant, qui est utilisé pour se connecter au registre de conteneurs cible.|
|Fournissez le mot de passe pour le registre de conteneurs cible ; appuyez sur Entrée pour n’en utiliser aucun :|TARGET_DOCKER_PASSWORD|Mot de passe, le cas échéant, qui est utilisé pour se connecter au registre de conteneurs cible. Il s’agit d’une invite de mot de passe masqué.  Vous ne voyez pas le mot de passe si vous le tapez ou le collez dedans.|
|Fournissez une étiquette d’image conteneur pour les images dans la cible :|TARGET_DOCKER_TAG|En règle générale, vous utilisez la même étiquette que la source pour éviter toute confusion.|