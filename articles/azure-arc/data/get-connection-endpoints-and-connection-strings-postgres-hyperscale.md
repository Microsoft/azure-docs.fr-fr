---
title: Obtenir des points de terminaison de connexion et former des chaînes de connexion pour votre groupe de serveurs Arc enabled PostgreSQL Hyperscale
titleSuffix: Azure Arc enabled data services
description: Obtenir des points de terminaison de connexion et former des chaînes de connexion pour votre groupe de serveurs Arc enabled PostgreSQL Hyperscale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 4087d618209ab4db46f89ef4e6db7ac87ca4cf57
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91331010"
---
# <a name="get-connection-endpoints-and-form-connection-strings-for-your-arc-enabled-postgresql-hyperscale-server-group"></a>Obtenir des points de terminaison de connexion et former des chaînes de connexion pour votre groupe de serveurs Arc enabled PostgreSQL Hyperscale

Cet article explique comment récupérer les points de terminaison de connexion pour votre groupe de serveurs et comment former les chaînes de connexion que vous utiliserez avec vos applications et/ou vos outils.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="get-connection-end-points"></a>Obtenez les points de terminaison de connexion :

### <a name="from-cli-with-azdata"></a>Depuis l’interface CLI avec azdata
#### <a name="1-connect-to-your-arc-data-controller"></a>1. Connectez-vous au contrôleur de données Azure Arc :
- Si vous avez déjà une session ouverte sur l’hôte du contrôleur de données Arc : Exécutez la commande suivante :
```console
azdata login
```

- Si aucune session n’est ouverte sur l’hôte du contrôleur de données Arc, exécutez la commande suivante : 
```console
azdata login --endpoint https://<external IP address of host/data controller>:30080
```

#### <a name="2-show-the-connection-endpoints"></a>2. Montrer les points de terminaison de connexion
Exécutez la commande suivante :
```console
azdata arc postgres endpoint list -n <server group name>
```
Elle retourne une sortie comme celle-ci :
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```
Utilisez ces points de terminaison pour :
- Former vos chaînes de connexion et vous connecter avec vos outils clients ou vos applications clientes
- Accéder aux tableaux de bord Grafana et Kibana depuis votre navigateur

Par exemple, vous pouvez utiliser le point de terminaison nommé _Instance PostgreSQL_ pour vous connecter avec psql à votre groupe de serveurs. Exemple :
```console
psql postgresql://postgres:MyPassworkd@12.345.123.456:1234
psql (10.14 (Ubuntu 10.14-0ubuntu0.18.04.1), server 12.4 (Ubuntu 12.4-1.pgdg16.04+1))
WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```
> [!NOTE]
>
> - Le mot de passe de l’utilisateur _postgres_ indiqué dans le point de terminaison nommé « _Instance PostgreSQL_ » est le mot de passe que vous avez choisi lors du déploiement du groupe de serveurs.
> - À propos de azdata : le bail associé à votre connexion dure environ 10 heures. Après cela, vous devez vous reconnecter. Si votre bail a expiré, le message d’erreur suivant s’affiche quand vous essayez d’exécuter une commande avec azdata (autre que azdata login) : _ERREUR : (401)_ 
> _Raison : En-têtes de réponse HTTP_
> _ non autorisés : HTTPHeaderDict({'Date': 'Dim, 06 Sep 2020 16:58:38 GMT', 'Content-Length': '0', 'WWW-Authenticate': '_ 
> _Domaine de base="Informations d’identification_ de connexion requises", Erreur de porteur="invalid_token", error_description="Le jeton est expiré"'})_ Quand ceci se produit, vous devez vous reconnecter avec azdata comme expliqué ci-dessus.

## <a name="from-cli-with-kubectl"></a>Depuis l’interface CLI avec kubectl
- Si votre groupe de serveurs a la version 12 de Postgres (par défaut), utilisez la commande suivante :
```console
kubectl get postgresql-12/<server group name>
```
- Si votre groupe de serveurs a la version 11 de Postgres, utilisez la commande suivante :
```console
kubectl get postgresql-11/<server group name>
```

Ces commandes produisent une sortie similaire à celle-ci. Vous pouvez utiliser ces informations pour former vos chaînes de connexion :
```console
NAME         STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgres01   Ready   3/3          123.456.789.4:31066      5d20h
``` 


## <a name="form-connection-strings"></a>Formez des chaînes de connexion :
Utilisez le tableau de modèles de chaînes de connexion ci-dessous pour votre groupe de serveurs. Vous pouvez ensuite les copier/coller et les personnaliser en fonction des besoins :

### <a name="adonet"></a>ADO.NET

```ado.net
Server=192.168.1.121;Database=postgres;Port=24276;User Id=postgres;Password={your_password_here};Ssl Mode=Require;`
```

### <a name="c-libpq"></a>C++ (libpq)

```cpp
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="jdbc"></a>JDBC

```jdbc
jdbc:postgresql://192.168.1.121:24276/postgres?user=postgres&password={your_password_here}&sslmode=require
```

### <a name="nodejs"></a>Node.js

```node.js
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="php"></a>PHP

```php
host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require
```

### <a name="psql"></a>psql

```psql
psql "host=192.168.1.121 port=24276 dbname=postgres user=postgres password={your_password_here} sslmode=require"
```

### <a name="python"></a>Python

```python
dbname='postgres' user='postgres' host='192.168.1.121' password='{your_password_here}' port='24276' sslmode='true'
```

### <a name="ruby"></a>Ruby

```ruby
host=192.168.1.121; dbname=postgres user=postgres password={your_password_here} port=24276 sslmode=require
```

### <a name="web-app"></a>Application web

```webapp
Database=postgres; Data Source=192.168.1.121; User Id=postgres; Password={your_password_here}
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [scale-out (ajout de nœuds worker)](scale-out-postgresql-hyperscale-server-group.md) de votre groupe de serveurs
- En savoir plus sur le [scale-up ou le scale-down (augmentation/diminution de la mémoire/des vCores)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) de votre groupe de serveurs


