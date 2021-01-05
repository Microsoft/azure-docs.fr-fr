---
title: Utiliser Stockage Blob Azure pour la conversion de modèle
description: Décrit les étapes courantes de configuration et d’utilisation du stockage Blob pour la conversion de modèle.
author: jakrams
ms.author: jakras
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 96a2fde3e510c6eb7146da9c92d93f69111e8c80
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206541"
---
# <a name="use-azure-blob-storage-for-model-conversion"></a>Utiliser Stockage Blob Azure pour la conversion de modèle

Le service de [conversion de modèle](model-conversion.md) nécessite l’accès au stockage Blob Azure pour pouvoir récupérer les données d’entrée et stocker les données de sortie. Cet article explique comment effectuer les étapes les plus courantes.

## <a name="prepare-azure-storage-accounts"></a>Préparer les comptes de stockage Azure

- Créez un compte de stockage (StorageV2).
- Créez un conteneur d’objets Blob d’entrée dans le compte de stockage (nommé « arrinput », par exemple).
- Créez un conteneur d’objets Blob de sortie dans le compte de stockage (nommé « arroutput », par exemple).

> [!TIP]
> Pour obtenir des instructions pas à pas sur la configuration de votre compte de stockage, consultez [Démarrage rapide : Convertir un modèle pour le rendu](../../quickstarts/convert-model.md).

La création du compte de stockage et des conteneurs d’objets Blob peut être effectuée à l’aide de l’un des outils suivants :

- [Azure portal](https://portal.azure.com)
- [Ligne de commande az](/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)
- SDK (C#, Python, etc.)

## <a name="ensure-azure-remote-rendering-can-access-your-storage-account"></a>Vérifier qu’Azure Remote Rendering peut accéder à votre compte de stockage

Azure Remote Rendering doit pouvoir récupérer les données du modèle à partir de votre compte de stockage et les réécrire dans le compte.

Vous pouvez accorder à Azure Remote Rendering l’accès à votre compte de stockage à l’aide des deux méthodes suivantes :

### <a name="connect-your-azure-storage-account-with-your-azure-remote-rendering-account"></a>Connexion de votre compte de stockage Azure à votre compte Azure Remote Rendering

Suivez les étapes décrites dans la section [Créer un compte](../create-an-account.md#link-storage-accounts).

### <a name="retrieve-sas-for-the-storage-containers"></a>Récupération de SAS pour les conteneurs de stockage

Les signatures d’accès stockées (SAS) sont utilisées afin d’accorder l’accès en lecture pour l’entrée et l’accès en écriture pour la sortie. Nous recommandons de générer de nouveaux URI chaque fois qu’un modèle est converti. Les URI expirent après un certain temps. Votre application peut donc s’arrêter de manière inattendue si vous les conservez plus longtemps.

Pour plus d’informations sur les SAS, consultez la [documentation SAS](../../../storage/common/storage-sas-overview.md).

Vous pouvez générer un URI SAS à l’aide de l’une des méthodes suivantes :

- Module az PowerShell
  - Consultez les [exemples de scripts PowerShell](../../samples/powershell-example-scripts.md).
- [Ligne de commande az](/cli/azure/install-azure-cli?view=azure-cli-latest)
- [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)
  - Cliquez avec le bouton droit sur le conteneur « Obtenir une signature d’accès partagé » (accès en lecture et liste pour le conteneur d’entrée, accès en écriture pour le conteneur de sortie).
- SDK (C#, Python, etc.)

Le fichier Conversion.ps1 des [exemples de scripts PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1) offre un exemple d’utilisation de signatures d’accès partagé pour la conversion des ressources.

## <a name="upload-an-input-model"></a>Charger un modèle d’entrée

Pour commencer à convertir un modèle, vous devez le charger à l’aide de l’une des méthodes suivantes :

- [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) : une interface utilisateur pratique pour charger/télécharger/gérer des fichiers sur un stockage Blob Azure
- [Ligne de commande Azure](../../../storage/blobs/storage-quickstart-blobs-cli.md)
- [Module Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.2.0)
  - Consultez les [exemples de scripts PowerShell](../../samples/powershell-example-scripts.md).
- [Utilisation d’un SDK de stockage (Python, C#, etc.)](../../../storage/index.yml)
- [Utilisation des API REST Stockage Azure](/rest/api/storageservices/blob-service-rest-api)

Pour obtenir un exemple illustrant comment charger des données pour la conversion, reportez-vous au fichier Conversion.ps1 des [exemples de scripts PowerShell](../../samples/powershell-example-scripts.md#script-conversionps1).

## <a name="get-a-sas-uri-for-the-converted-model"></a>Obtenir un URI SAS pour le modèle converti

Cette étape est similaire à celle permettant de [récupérer une SAS pour les conteneurs de stockage](#retrieve-sas-for-the-storage-containers). Cependant, vous devez cette fois récupérer un URI SAS pour le fichier de modèle, qui a été écrit dans le conteneur de sortie.

Par exemple, pour récupérer un URI SAS à l’aide de l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/), cliquez avec le bouton droit sur le fichier de modèle et sélectionnez « Obtenir une signature d’accès partagé ».

Une signature d’accès partagé (SAS) est nécessaire pour charger les modèles si vous n’avez pas connecté votre compte de stockage à votre compte Azure Remote Rendering. Pour savoir comment connecter votre compte, consultez [Créer un compte](../create-an-account.md#link-storage-accounts).

## <a name="next-steps"></a>Étapes suivantes

- [Configurer la conversion de modèle](configure-model-conversion.md)
- [API REST de conversion de modèle](conversion-rest-api.md)