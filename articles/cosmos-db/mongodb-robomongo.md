---
title: Utiliser Robo 3T pour se connecter à Azure Cosmos DB
description: Découvrez comment vous connecter à Azure Cosmos DB à l’aide de Robo 3T et de l’API Azure Cosmos DB pour MongoDB
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/23/2020
author: timsander1
ms.author: tisande
ms.openlocfilehash: e096de5eea9bf5bbd47b976976192e62eb6aa33e
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93096408"
---
# <a name="use-robo-3t-with-azure-cosmos-dbs-api-for-mongodb"></a>Utiliser Robo 3T avec l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Pour vous connecter à un compte Cosmos avec Robo 3T, vous devez effectuer les opérations suivantes :

* Télécharger et installer [Robo 3T](https://robomongo.org/)
* Obtenir vos informations de [chaîne de connexion](connect-mongodb-account.md) Cosmos DB

> [!NOTE]
> Actuellement, les versions v1.2 et antérieures de Robo 3T sont prises en charge par l’API Cosmos DB pour MongoDB.

## <a name="connect-using-robo-3t"></a>Vous connecter à l’aide de Robo 3T

Pour ajouter votre compte Cosmos au gestionnaire de connexions Robo 3T, procédez comme suit :

1. Récupérez les informations de connexion pour votre compte Cosmos configuré avec l’API Azure Cosmos DB pour MongoDB en suivant les instructions [ici](connect-mongodb-account.md).

    :::image type="content" source="./media/mongodb-robomongo/connectionstringblade.png" alt-text="Capture d’écran du panneau Chaîne de connexion":::
2. Exécutez l'application *Robomongo*.

3. Cliquez sur le bouton de connexion sous **Fichier** pour gérer vos connexions. Ensuite, cliquez sur **Créer** dans la fenêtre **Connexions MongoDB** , ce qui ouvre la fenêtre **Paramètres de connexion**.

4. Dans la fenêtre **Paramètres de connexion** , choisissez un nom. Ensuite, recherchez **Hôte** et **Port** dans vos informations de connexion à l’étape 1 et entrez-les dans les champs **Adresse** et **Port** , respectivement.

    :::image type="content" source="./media/mongodb-robomongo/manageconnections.png" alt-text="Capture d’écran de la gestion des connexions avec Robomongo":::
5. Dans l’onglet **Authentification** , cliquez sur **Effectuer l’authentification**. Entrez ensuite votre base de données (la valeur par défaut est *Admin* ), le **Nom d’utilisateur** et le **Mot de passe**.
Le **Nom d’utilisateur** et le **Mot de passe** figurent tous deux dans vos informations de connexion à l’étape 1.

    :::image type="content" source="./media/mongodb-robomongo/authentication.png" alt-text="Capture d’écran de l’onglet Authentification de Robomongo":::
6. Dans l’onglet **SSL** , cochez **Utiliser le protocole SSL** , puis modifiez la **Méthode d’authentification** sur **Certificat auto-signé**.

    :::image type="content" source="./media/mongodb-robomongo/SSL.png" alt-text="Capture d’écran de l’onglet SSL de Robomongo":::
7. Enfin, cliquez sur **Test** pour vérifier que vous êtes en mesure de vous connecter, puis sur **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser Studio 3T](mongodb-mongochef.md) avec l’API Azure Cosmos DB pour MongoDB.
- Explorez les [exemples](mongodb-samples.md) MongoDB avec l’API Azure Cosmos DB pour MongoDB.
