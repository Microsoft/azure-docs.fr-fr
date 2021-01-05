---
title: Conversion de modèle
description: Décrit le processus de conversion d’un modèle pour le rendu
author: jakrams
ms.author: jakras
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: e899b249261ea3238695a2e2be6001cb6a9bc763
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91318056"
---
# <a name="convert-models"></a>Convertir des modèles

Azure Remote Rendering vous permet d’afficher des modèles très complexes. Pour obtenir des performances maximales, les données doivent être prétraitées pour être dans un format optimal. Selon la quantité de données, cette étape peut prendre un certain temps. Il serait peu pratique que ce temps soit perdu lors du chargement des modèles. En outre, ce serait dommage de répéter ce processus pour plusieurs sessions. Pour ces raisons, le service ARR fournit un *service de conversion* dédié, que vous pouvez exécuter à l’avance.
Une fois converti, un modèle peut être chargé à partir d’un compte Stockage Azure.

## <a name="supported-source-formats"></a>Formats sources pris en charge

Le service de conversion prend en charge les formats suivants :

- **FBX**  (version 2011 vers la version 2020)
- **GLTF**/**GLB** (version 2.x)

Il existe des différences mineures entre les formats en ce qui concerne la conversion des propriétés de matériau, comme indiqué dans le chapitre [Mappage de matériaux pour les formats de modèle](../../reference/material-mapping.md).

## <a name="the-conversion-process"></a>Processus de conversion

1. [Préparez deux conteneurs Stockage Blob Azure](blob-storage.md) : un pour l’entrée, l’autre pour la sortie
1. Chargez votre modèle dans le conteneur d’entrée (éventuellement sous un sous-tracé)
1. Déclenchez le processus de conversion via [l’API REST de conversion de modèle](conversion-rest-api.md)
1. Interrogez le service pour connaître la progression de la conversion
1. Une fois l’opération terminée, chargez un modèle
    - à partir d’un compte de stockage lié (voir les étapes « Lier des comptes de stockage » dans [Créer un compte](../create-an-account.md#link-storage-accounts) pour lier votre compte de stockage)
    - ou en fournissant une *signature d’accès partagé (SAS)* .

Toutes les données du modèle (entrée et sortie) sont stockées dans le stockage Blob Azure fourni par l’utilisateur. Azure Remote Rendering vous donne un contrôle total sur la gestion de vos ressources.

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs de conversion, consultez la page [Tarification Remote Rendering](https://azure.microsoft.com/pricing/details/remote-rendering).


## <a name="conversion-parameters"></a>Paramètres de conversion

Pour connaître les différentes options de conversion, consultez [ce chapitre](configure-model-conversion.md).

## <a name="examples"></a>Exemples

- [Démarrage rapide : Convertir un modèle pour le rendu](../../quickstarts/convert-model.md) est une présentation pas à pas de la conversion d’un modèle.
- Des [exemples de scripts PowerShell](../../samples/powershell-example-scripts.md), qui illustrent l’utilisation du service de conversion, sont accessibles dans le [référentiel d’exemples ARR](https://github.com/Azure/azure-remote-rendering) dans le dossier *Scripts*.

## <a name="next-steps"></a>Étapes suivantes

- [Utiliser Stockage Blob Azure pour une conversion de modèle](blob-storage.md)
- [API REST de conversion de modèle](conversion-rest-api.md)
- [Configurer la conversion de modèle](configure-model-conversion.md)
- [Disposition des fichiers pour la conversion](layout-files-for-conversion.md)
- [Mappage de matériaux pour les formats de modèle](../../reference/material-mapping.md)
