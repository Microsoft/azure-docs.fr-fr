---
title: Utiliser le portail Azure pour utiliser des clés managées par le client ou BYOK avec la plateforme Media Services
description: Dans ce tutoriel, vous allez utiliser le portail Azure pour activer des clés gérées par le client ou BYOK (Bring Your Own Key) avec un compte de stockage Azure Media Services.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: tutorial
ms.date: 10/18/2020
ms.openlocfilehash: 003b8e066a6161baedbc70e9becbca23566813ef
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "96013227"
---
# <a name="tutorial-use-the-azure-portal-to-use-customer-managed-keys-or-byok-with-media-services"></a>Tutoriel : Utiliser le portail Azure pour utiliser des clés gérées par le client ou BYOK avec la plateforme Media Services

Avec l’API 2020-05-01, vous pouvez utiliser une clé RSA gérée par le client avec un compte Azure Media Services disposant d’une identité managée par le système. Ce tutoriel décrit les étapes effectuées dans le portail Azure.

Les services utilisés sont les suivants :

- Stockage Azure
- Azure Key Vault
- Azure Media Services

Ce tutoriel vous apprend à utiliser le portail Azure pour :

> [!div class="checklist"]
> - Créez un groupe de ressources.
> - Créer un compte de stockage avec une identité managée par le système
> - Créer un compte Media Services avec une identité managée affectée par le système
> - Créer un coffre de clés pour stocker une clé RSA gérée par le client

## <a name="prerequisites"></a>Prérequis

Un abonnement Azure.

Si vous n’avez pas d’abonnement Azure, [créez un compte d’essai gratuit](https://azure.microsoft.com/free/).

## <a name="system-managed-keys"></a>Clés managées par le système

<!-- Create a resource group -->
[!INCLUDE [create a resource group in the portal](./includes/task-create-resource-group-portal.md)]

> [!IMPORTANT]
> Pour les étapes de création de compte de stockage suivantes, vous devez sélectionner le choix de clé managée par le système dans les paramètres avancés.

<!-- Create a media services account -->

[!INCLUDE [create a media services account in the portal](./includes/task-create-media-services-account-portal.md)]

<!-- Create a key vault -->

[!INCLUDE [create a key vaultl](./includes/task-create-key-vault-portal.md)]

<!-- Enable CMK BYOK on the account -->
[!INCLUDE [enable CMK](./includes/task-enable-cmk-byok-portal.md)]

> [!IMPORTANT]
> Pour les étapes de chiffrement de stockage suivantes, vous devez sélectionner le **choix de clé gérée par le client**.

<!-- Set encryption for storage account -->
[!INCLUDE [Set encryption for storage account](./includes/task-set-storage-encryption-portal.md)]

## <a name="change-the-key"></a>Modifier la clé

Media Services détecte automatiquement quand la clé est changée. FACULTATIF : Pour tester ce processus, créez une autre version de la même clé. Media Services doit détecter que la clé a été changée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas utiliser les ressources que vous avez créées et *ne souhaitez plus qu’elles vous soient facturées*, vous pouvez les supprimer.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour découvrir comment :
> [!div class="nextstepaction"]
> [Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu avec REST](stream-files-tutorial-with-rest.md)
