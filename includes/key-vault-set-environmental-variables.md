---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: 6dfc96df3623e44a4ba513c238f21482c2a2989d
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87013098"
---
La méthode DefaultAzureCredential dans notre application s’appuie sur trois variables d’environnement : `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` et `AZURE_TENANT_ID`. Définissez ces variables sur les valeurs clientId, clientSecret et tenantId qui ont été retournées à l’étape « Créer un principal de service » au format `export VARNAME=VALUE`. (Cette méthode définit seulement les variables pour votre interpréteur de commandes actuel et les processus créés depuis celui-ci ; pour ajouter ces variables de façon permanente à votre environnement, modifiez votre fichier `/etc/environment `.) 

Vous devrez également enregistrer votre nom de coffre de clés en tant que variable d’environnement appelée `KEY_VAULT_NAME`.

```console
export AZURE_CLIENT_ID=<your-clientID>

export AZURE_CLIENT_SECRET=<your-clientSecret>

export AZURE_TENANT_ID=<your-tenantId>

export KEY_VAULT_NAME=<your-key-vault-name>
````
