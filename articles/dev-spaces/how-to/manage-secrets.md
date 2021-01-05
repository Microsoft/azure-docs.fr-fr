---
title: Guide pratique pour gérer les secrets en utilisant un espace Azure Dev Spaces
services: azure-dev-spaces
ms.date: 12/03/2019
ms.topic: conceptual
description: Découvrez comment utiliser des secrets Kubernetes au moment de l’exécution ou de la génération quand vous développez des applications avec Azure Dev Spaces
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, conteneurs
ms.custom: devx-track-js
ms.openlocfilehash: 8791480f420dfd76d5291ce82e8ebf7412a41326
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91972966"
---
# <a name="how-to-manage-secrets-when-working-with-an-azure-dev-space"></a>Guide pratique pour gérer les secrets en utilisant un espace Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Vos services peuvent exiger certains mots de passe, chaînes de connexion et autres secrets, comme c’est le cas avec les bases de données ou d’autres services Azure sécurisés. En définissant les valeurs de ces secrets dans des fichiers de configuration, vous pouvez les rendre accessibles dans votre code en tant que variables d’environnement.  Ces fichiers de configuration doivent être traités avec soin pour éviter de compromettre la sécurité des secrets.

## <a name="storing-and-using-runtime-secrets"></a>Stockage et utilisation des secrets au moment de l’exécution

Azure Dev Spaces propose deux options rationalisées recommandées pour le stockage des secrets dans les graphiques Helm générés par les outils client Azure Dev Spaces : dans le fichier `values.dev.yaml` ou directement inclus dans `azds.yaml`. Il est déconseillé de stocker les secrets dans `values.yaml`.

> [!NOTE]
> Les approches suivantes vous montrent comment stocker et utiliser des secrets pour les graphiques Helm générés par les outils clients. Si vous créez votre propre graphique Helm, vous pouvez utiliser le graphique Helm directement pour gérer et stocker les secrets.

### <a name="using-valuesdevyaml"></a>Utilisation de values.dev.yaml

Dans un projet que vous avez déjà préparé avec Azure Dev Spaces, créez un fichier `values.dev.yaml` dans le même dossier que `azds.yaml` pour définir vos clés et valeurs secrètes. Par exemple :

```yaml
secrets:
  redis:
    port: "6380"
    host: "contosodevredis.redis.cache.windows.net"
    key: "secretkeyhere"
```

Vérifiez que le fichier `azds.yaml` référence `values.dev.yaml` comme facultatif à l’aide d’un `?`. Par exemple :

```yaml
install:
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
```

Si vous avez des fichiers de secret supplémentaires, vous pouvez également les ajouter ici.

Mettez à jour ou vérifiez que votre service référence vos secrets en tant que variables d’environnement. Par exemple :

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Exécutez vos services mis à jour à l’aide de `azds up`.

```console
azds up
```
 
Utilisez `kubectl` pour vérifier que vos secrets ont été créés.

```console
kubectl get secret --namespace default -o yaml 
```

> [!IMPORTANT]
> Il n’est pas recommandé de stocker les secrets dans le contrôle de code source. Si vous utilisez Git, ajoutez `values.dev.yaml` dans le fichier `.gitignore` pour éviter de valider les secrets dans le contrôle de code source.

### <a name="using-azdsyaml"></a>Utilisation d’azds.yaml

Dans un projet que vous avez déjà préparé avec Azure Dev Spaces, ajoutez une valeur et des clés secrètes à l’aide de la syntaxe *$PLACEHOLDER* sous *configurations.develop.install.set* dans `azds.yaml`. Par exemple :

```yaml
configurations:
  develop:
    ...
    install:
      set:
        secrets:
          redis:
            port: "$REDIS_PORT"
            host: "$REDIS_HOST"
            key: "$REDIS_KEY"
```

> [!NOTE]
> Vous pouvez entrer des valeurs secrètes directement sans utiliser la syntaxe *$PLACEHOLDER* dans `azds.yaml`. Toutefois, cette approche n’est pas recommandée, car `azds.yaml` est stocké dans le contrôle de code source.
     
Créez un fichier `.env` dans le même dossier que `azds.yaml` pour définir vos valeurs *$PLACEHOLDER*. Par exemple :

```
REDIS_PORT=3333
REDIS_HOST=myredishost
REDIS_KEY=myrediskey
```

> [!IMPORTANT]
> Il n’est pas recommandé de stocker les secrets dans le contrôle de code source. Si vous utilisez Git, ajoutez `.env` dans le fichier `.gitignore` pour éviter de valider les secrets dans le contrôle de code source.

Mettez à jour ou vérifiez que votre service référence vos secrets en tant que variables d’environnement. Par exemple :

```javascript
var redisPort = process.env.REDIS_PORT
var host = process.env.REDIS_HOST
var theKey = process.env.REDIS_KEY
```
    
Exécutez vos services mis à jour à l’aide de `azds up`.

```console
azds up
```
 
Utilisez `kubectl` pour vérifier que vos secrets ont été créés.

```console
kubectl get secret --namespace default -o yaml 
```

## <a name="using-secrets-as-build-arguments"></a>Utilisation de secrets en tant qu’arguments de build

La section précédente a montré comment stocker et utiliser des secrets au moment de l’exécution du conteneur. Vous pouvez également utiliser un secret quelconque au moment de la génération du conteneur, tel qu’un mot de passe pour un NuGet privé, en utilisant `azds.yaml`.

Dans `azds.yaml`, définissez les secrets au moment de la génération dans *configurations.develop.build.args* à l’aide de la syntaxe `<variable name>: ${secret.<secret name>.<secret key>}`. Par exemple :

```yaml
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
      useGitIgnore: true
      args:
        BUILD_CONFIGURATION: ${BUILD_CONFIGURATION:-Debug}
        MYTOKEN: ${secret.mynugetsecret.pattoken}
```

Dans l’exemple ci-dessus, *mynugetsecret* est un secret existant et *pattoken* est une clé existante.

>[!NOTE]
> Les noms de secrets et les clés peuvent contenir le caractère `.`. Utilisez `\` pour échapper `.` lors du passage de secrets en tant qu’arguments de build. Par exemple, pour passer un secret nommé *foo.bar* avec la clé du *jeton* : `MYTOKEN: ${secret.foo\.bar.token}`. En outre, les secrets peuvent être évalués avec du texte de préfixe et suffixe. Par exemple : `MYURL: eus-${secret.foo\.bar.token}-version1`. De plus, les secrets disponibles dans les espaces parents et grands parents peuvent être passés en tant qu’arguments de build.

Dans votre Dockerfile, utilisez la directive *ARG* pour consommer le secret, puis utilisez la même variable ultérieurement dans le Dockerfile. Par exemple :

```dockerfile
...
ARG MYTOKEN
...
ARG NUGET_EXTERNAL_FEED_ENDPOINTS="{'endpointCredentials': [{'endpoint':'PRIVATE_NUGET_ENDPOINT', 'password':'${MYTOKEN}'}]}"
...
```

Mettez à jour les services en cours d’exécution dans votre cluster avec ces modifications. Sur la ligne de commande, exécutez la commande suivante :

```
azds up
```

## <a name="next-steps"></a>Étapes suivantes

Grâce à ces méthodes, vous pouvez maintenant vous connecter de manière sécurisée à une base de données, un cache Azure pour Redis ou accéder à des services Azure sécurisés.
 
