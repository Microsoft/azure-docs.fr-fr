---
title: Fichier include
description: Fichier include
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: 472129be5baa865365b49894b705d84c23e9cd04
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97506301"
---
## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7, 3.5 ou ultérieur.
- Une ressource Communication Services active et la chaîne de connexion. [Créez une ressource Communication Services](../create-communication-resource.md).

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-python-application"></a>Créer une application Python

1. Ouvrez votre fenêtre de terminal ou de commande, créez un répertoire pour votre application, puis accédez-y.

   ```console
   mkdir access-tokens-quickstart && cd access-tokens-quickstart
   ```

1. Utilisez un éditeur de texte pour créer un fichier appelé **issue-access-tokens.py** dans le répertoire racine du projet, puis ajoutez la structure du programme, notamment la gestion des exceptions de base. Dans les sections suivantes, vous ajouterez l’ensemble du code source de ce guide de démarrage rapide dans ce fichier.

   ```python
   import os
   from azure.communication.administration import CommunicationIdentityClient

   try:
      print('Azure Communication Services - Access Tokens Quickstart')
      # Quickstart code goes here
   except Exception as ex:
      print('Exception:')
      print(ex)
   ```

### <a name="install-the-package"></a>Installer le package

Toujours dans le répertoire de l’application, installez le package de la bibliothèque de client Azure Communication Services Administration pour Python à l’aide de la commande `pip install`.

```console
pip install azure-communication-administration
```

## <a name="authenticate-the-client"></a>Authentifier le client

Instanciez un `CommunicationIdentityClient` avec votre chaîne de connexion. Le code ci-dessous récupère la chaîne de connexion de la ressource à partir d’une variable d’environnement nommée `COMMUNICATION_SERVICES_CONNECTION_STRING`. Découvrez comment [gérer la chaîne de connexion de la ressource](../create-communication-resource.md#store-your-connection-string).

Ajoutez ce code dans le bloc `try` :

```python
# This code demonstrates how to fetch your connection string
# from an environment variable.
connection_string = os.environ['COMMUNICATION_SERVICES_CONNECTION_STRING']

# Instantiate the identity client
client = CommunicationIdentityClient.from_connection_string(connection_string)
```

## <a name="create-an-identity"></a>Créer une identité

Azure Communication Services gère un répertoire LID (Lightweight Identity Directory). Utilisez la méthode `create_user` pour créer une entrée, avec une valeur `Id` unique, dans le répertoire. Stockez l’identité reçue avec un mappage aux utilisateurs de votre application. Par exemple, en les stockant dans la base de données de votre serveur d’applications. L’identité sera demandée ultérieurement pour émettre des jetons d’accès.

```python
identity = client.create_user()
print("\nCreated an identity with ID: " + identity.identifier + ":")
```

## <a name="issue-access-tokens"></a>Émettre des jetons d’accès

Avec la méthode `issue_token`, émettez un jeton d’accès pour une identité Communication Services existant déjà. Le paramètre `scopes` définit un ensemble de primitives, qui autorise ce jeton d’accès. Consultez [la liste des actions prises en charge](../../concepts/authentication.md). Une nouvelle instance du paramètre `communicationUser` peut être construite en fonction de la représentation sous forme de chaîne de l’identité Azure Communication Service.

```python
# Issue an access token with the "voip" scope for an identity
token_result = client.issue_token(identity, ["voip"])
expires_on = token_result.expires_on.strftime('%d/%m/%y %I:%M %S %p')
print("\nIssued an access token with 'voip' scope that expires at " + expires_on + ":")
print(token_result.token)
```

Les jetons d’accès sont des informations d’identification à durée de vie courte, qui doivent être réémises. Ne pas le faire peut entraîner une interruption expérimentée par les utilisateurs de votre application. La propriété de réponse `expires_on` indique la durée de vie du jeton d’accès.

## <a name="refresh-access-tokens"></a>Actualiser des jetons d’accès

Pour actualiser un jeton d’accès, utilisez l’objet `CommunicationUser` permettant d’émettre à nouveau :

```python  
# Value existingIdentity represents identity of Azure Communication Services stored during identity creation
identity = CommunicationUser(existingIdentity)
token_result = client.issue_token( identity, ["voip"])
```

## <a name="revoke-access-tokens"></a>Révoquer des jetons d’accès

Dans certains cas, vous pouvez révoquer explicitement des jetons d’accès. Par exemple, lorsqu’un utilisateur d’une application modifie le mot de passe qu’il utilise pour s’authentifier auprès de votre service. La méthode `revoke_tokens` invalide tous les jetons d’accès actifs qui ont été émis pour l’identité.

```python  
client.revoke_tokens(identity)
print("\nSuccessfully revoked all access tokens for identity with ID: " + identity.identifier)
```

## <a name="delete-an-identity"></a>Supprimer une identité

Quand vous supprimez une identité, vous supprimez aussi tous les jetons d’accès actifs et vous ne pouvez plus émettre de jetons d’accès pour l’identité. Tout le contenu persistant associé à l’identité est également supprimé.

```python
client.delete_user(identity)
print("\nDeleted the identity with ID: " + identity.identifier)
```

## <a name="run-the-code"></a>Exécuter le code

À partir d’une invite de console, accédez au répertoire contenant le fichier *issue-access-token.py*, puis exécutez la commande `python` suivante pour exécuter l’application.

```console
python ./issue-access-token.py
```
