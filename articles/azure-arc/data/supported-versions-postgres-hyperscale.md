---
title: Versions prises en charge de Postgres avec PostgreSQL Hyperscale activé pour Azure Arc
description: Versions prises en charge de Postgres avec PostgreSQL Hyperscale activé pour Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0f3f9e7f4566800bdea6871cae1c5bafb3b37b95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930196"
---
# <a name="supported-versions-of-postgres-with-azure-arc-enabled-postgresql-hyperscale"></a>Versions prises en charge de Postgres avec PostgreSQL Hyperscale activé pour Azure Arc

Cet article explique quelles versions de Postgres sont disponibles avec PostgreSQL Hyperscale activé pour Azure Arc.
La liste des versions prises en charge évolue au fil du temps. Aujourd’hui, les versions principales prises en charge sont les suivantes :
- Postgres 12 (valeur par défaut)
- Postgres 11

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="how-to-chose-between-versions"></a>Comment choisir entre les versions ?
Il est recommandé de consulter les versions pour lesquelles vos applications ont été conçues et quelles sont les fonctionnalités de chacune des versions. Pour en savoir plus, consultez les informations relatives à chaque version sur le site officiel Postgres :
- [Postgres 12 (valeur par défaut)](https://www.postgresql.org/docs/12/index.html)
- [Postgres 11](https://www.postgresql.org/docs/11/index.html)

## <a name="how-to-create-a-particular-version-in-azure-arc-enabled-postgresql-hyperscale"></a>Comment créer une version particulière dans PostgreSQL Hyperscale avec Azure Arc ?
Au moment de la création, vous avez la possibilité d’indiquer la version à créer en passant le paramètre _--engine-version_. Si vous n’indiquez pas d’informations sur la version, par défaut, un groupe de serveurs avec la version 12 de Postgres sera créé.

## <a name="how-do-be-notified-when-other-versions-are-available"></a>Comment être averti lorsque d’autres versions sont disponibles ?
Revenez en arrière et lisez cet article. Il sera mis à jour selon les besoins. Vous pouvez également répertorier les types de définitions de ressources personnalisées (CRD) dans le contrôleur de données Arc de votre cluster Kubernetes.
Exécutez la commande suivante :
```console
kubectl get crds
```

Elle renverra une sortie comme suit :
```console
NAME                                            CREATED AT
datacontrollers.arcdata.microsoft.com           2020-08-31T20:15:16Z
postgresql-11s.arcdata.microsoft.com            2020-08-31T20:15:16Z
postgresql-12s.arcdata.microsoft.com            2020-08-31T20:15:16Z
sqlmanagedinstances.sql.arcdata.microsoft.com   2020-08-31T20:15:16Z
```

Dans cet exemple, cette sortie indique qu’il existe deux types de CRD liés à Postgres :
- postgresql-12s.arcdata.microsoft.com est le CRD pour Postgres 12
- postgresql-11s.arcdata.microsoft.com est le CRD pour Postgres 11

Il s’agit de CRD et non de groupes de serveurs. La présence d’un CRD n’indique pas que vous avez ou non créé un groupe de serveurs de cette version.
Le CRD est une indication du type de ressources qui peuvent être créées.

## <a name="next-steps"></a>Étapes suivantes :
- [Découvrir la création de PostgreSQL Hyperscale avec Azure Arc](create-postgresql-hyperscale-server-group.md)
- [En savoir plus sur la façon d’obtenir une liste des groupes de serveurs PostgreSQL Hyperscale avec Azure Arc créés dans un contrôleur de données Arc](list-server-groups-postgres-hyperscale.md)
