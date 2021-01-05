---
title: Utiliser des clés gérées par le client ou l’API REST BYOK
description: Dans ce tutoriel, vous allez utiliser des clés gérées par le client ou BYOK (Bring Your Own Key) avec un compte de stockage Azure Media Services.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: tutorial
ms.date: 10/18/2020
ms.openlocfilehash: c8a5b682e2ac4879d2181bdb069cf554bad512d9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498283"
---
# <a name="tutorial-use-customer-managed-keys-or-byok-with-media-services-rest-api"></a>Tutoriel : Utiliser des clés gérées par le client ou BYOK avec l’API REST Media Services

Avec l’API 2020-05-01, vous pouvez utiliser une clé RSA gérée par le client avec un compte Azure Media Services disposant d’une identité managée par le système. Ce tutoriel comprend une collection et un environnement Postman permettant d’envoyer des demandes REST aux services Azure. Les services utilisés sont les suivants :

- Inscription d’application Azure Active Directory (Azure AD) pour Postman
- API Microsoft Graph
- Stockage Azure
- Azure Key Vault
- Azure Media Services

Ce tutoriel explique comment utiliser Postman pour effectuer les opérations suivantes :

> [!div class="checklist"]
> - Obtenir les jetons à utiliser avec les services Azure
> - Créer un groupe de ressources et un compte de stockage
> - Créer un compte Media Services avec une identité managée affectée par le système
> - Créer un coffre de clés pour stocker une clé RSA gérée par le client
> - Mettre à jour le compte Media Services pour qu’il utilise la clé RSA avec le compte de stockage
> - Utiliser des variables dans Postman

Si vous n’avez pas d’abonnement Azure, [créez un compte d’essai gratuit](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Prérequis

1. Inscrire un principal de service avec les autorisations nécessaires
1. Installer [Postman](https://www.postman.com)
1. Téléchargez la collection Postman de ce tutoriel à l’adresse [Azure Samples: media-services-customer-managed-keys-byok](https://github.com/Azure-Samples/media-services-customer-managed-keys-byok).

### <a name="register-a-service-principal-with-the-needed-permissions"></a>Inscrire un principal de service avec les autorisations nécessaires

1. [Créez un principal du service](../../active-directory/develop/howto-create-service-principal-portal.md).
1. Accédez à [Option 2 : Créer un secret d’application](../../active-directory/develop/howto-create-service-principal-portal.md#authentication-two-options) pour obtenir le secret du principal du service.

   > [!IMPORTANT]
   >Copiez et enregistrez le secret pour une utilisation ultérieure. Vous ne pouvez plus accéder au secret une fois que vous avez quitté la page de secret dans le portail.

1. Affectez des autorisations au principal du service, comme illustré dans la capture d’écran suivante :

   :::image type="complex" source="./media/tutorial-byok/service-principal-permissions-1.png" alt-text="Capture d’écran montrant les autorisations nécessaires pour le principal du service.":::
   Les autorisations sont listées par service, nom d’autorisation, type, puis description. Azure Key Vault : emprunt d’identité de l’utilisateur, délégué, accès complet à Azure Key Vault. Gestion des services Azure : emprunt d’identité de l’utilisateur, délégué, accès à la gestion des services Azure en tant qu’utilisateur de l’organisation. Stockage Azure : emprunt d’identité de l’utilisateur, délégué, accès au stockage Azure. Media Services : emprunt d’identité de l’utilisateur, délégué, accès aux services multimédias. Microsoft Graph : utilisateur.lecture, délégué, se connecter et lire le profil utilisateur.
   :::image-end:::

### <a name="install-postman"></a>Installer Postman

Si vous n’avez pas déjà installé Postman afin de l’utiliser avec Azure, vous pouvez l’obtenir sur [postman.com](https://www.postman.com/).

### <a name="download-and-import-the-collection"></a>Télécharger et importer la collection

Téléchargez la collection Postman de ce tutoriel à l’adresse [Azure Samples: media-services-customer-managed-keys-byok](https://github.com/Azure-Samples/media-services-customer-managed-keys-byok).

## <a name="install-the-postman-collection-and-environment"></a>Installer la collection Postman et l’environnement

1. Exécutez Postman.
1. Sélectionnez **Importer**.
1. Sélectionnez **Charger des fichiers**.
1. Accédez à l’emplacement où vous avez enregistré les fichiers de collection et d’environnement.
1. Sélectionnez les fichier d’environnement et de collection.
1. Sélectionnez **Ouvrir**. Un avertissement s’affiche pour indiquer que les fichiers ne seront pas importés en tant qu’API, mais en tant que collections. C’est tout à fait normal. C’est ce que nous voulons.

La collection apparaît désormais dans vos collections en tant que BYOK. De plus, les variables d’environnement apparaissent dans vos environnements.

### <a name="understand-the-rest-api-requests-in-the-collection"></a>Comprendre les requêtes d’API REST de la collection

La collection fournit les requêtes d’API REST suivantes.

> [!NOTE]
>
>- Les requêtes doivent être envoyées dans la séquence fournie.
>- La plupart des requêtes ont des scripts de test qui créent dynamiquement des variables globales pour la requête suivante dans la séquence.
>- Vous n’avez pas besoin de créer manuellement des variables globales.

Dans Postman, vous verrez que ces variables sont placées entre crochets. Par exemple : `{{bearerToken}}`.

1. Obtenir un jeton Azure AD : Le test définit la variable globale **bearerToken**.
2. Obtenir un jeton Microsoft Graph : Le test définit la variable globale **graphToken**.
3. Obtenir les détails du principal du service : Le test définit la variable globale **servicePrincipalObjectId**.
4. Créer un compte de stockage : Le test définit la variable globale **storageAccountId**.
5. Créer un compte Media Services avec une identité managée affectée par le système : Le test définit la variable globale **principalId**.
6. Créer un coffre de clés pour accorder l’accès au principal du service : Le test définit la variable globale **keyVaultId**.
7. Obtenir un jeton Key Vault : Le test définit la variable globale **keyVaultToken**.
8. Créer la clé RSA dans le coffre de clés : Le test définit la variable globale **keyId**.
9. Mettre à jour le compte Media Services pour qu’il utilise la clé avec le compte de stockage : Il n’y a aucun script de test pour cette requête.

## <a name="define-environment-variables"></a>Définir des variables d’environnement

1. Sélectionnez la liste déroulante de l’environnement pour basculer vers l’environnement que vous avez téléchargé.
1. Établissez vos variables d’environnement dans Postman. Elles sont également utilisées comme des variables placées entre crochets. Par exemple : `{{tenantId}}`.

    - **tenantId** : Votre ID de client.
    - **servicePrincipalId** : ID du principal du service que vous établissez avec la méthode de votre choix, telle que le portail ou l’interface CLI.
    - **servicePrincipalSecret** : secret créé pour le principal du service.
    - **Abonnement** : Votre ID d’abonnement.
    - **storageName** : nom que vous souhaitez attribuer à votre stockage.
    - **accountName** : nom du compte Media Services que vous souhaitez utiliser.
    - **keyVaultName** : nom que vous souhaitez utiliser pour le coffre de clés.
    - **resourceLocation** : emplacement **CentralUs** ou tout autre endroit où placer vos ressources. Cette collection a été testée uniquement avec **CentralUs**.
    - **resourceGroup** : Nom du groupe de ressources.

    Les variables suivantes sont des variables standard pour l’utilisation des ressources Azure. Vous n’avez donc pas besoin de les modifier.

    - **armResource** : `https://management.core.windows.net`
    - **graphResource** : `https://graph.windows.net/`
    - **keyVaultResource** : `https://vault.azure.net`
    - **armEndpoint** : `management.azure.com`
    - **graphEndpoint** : `graph.windows.net`
    - **aadEndpoint** : `login.microsoftonline.com`
    - **keyVaultDomainSuffix** : `vault.azure.net`

## <a name="send-the-requests"></a>Envoyer les requêtes

Une fois que vous avez défini vos variables d’environnement, vous pouvez exécuter les requêtes une à la fois dans la séquence précédente. Vous pouvez également utiliser l’exécuteur de Postman pour exécuter la collection.

## <a name="change-the-key"></a>Modifier la clé

Media Services détecte automatiquement quand la clé est changée. Pour tester ce processus, créez une autre version de la même clé. Media Services doit détecter la clé en moins de 15 minutes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas utiliser les ressources que vous avez créées et *ne souhaitez plus qu’elles vous soient facturées*, vous pouvez les supprimer.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour découvrir comment :
> [!div class="nextstepaction"]
> [Encoder un fichier distant basé sur une URL et diffuser la vidéo en continu avec REST](stream-files-tutorial-with-rest.md)