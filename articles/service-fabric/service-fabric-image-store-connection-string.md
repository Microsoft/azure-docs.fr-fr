---
title: Chaîne de connexion du magasin d’images Azure Service Fabric
description: Découvrez la chaîne de connexion du magasin d’images, notamment ses utilisations et applications dans un cluster Service Fabric.
author: alexwun
ms.topic: conceptual
ms.date: 02/27/2018
ms.author: alexwun
ms.openlocfilehash: 8fc0239dd18fc7071823a129a7dbc4f102023d66
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86246195"
---
# <a name="understand-the-imagestoreconnectionstring-setting"></a>Comprendre le paramètre ImageStoreConnectionString

Dans une partie de notre documentation, nous mentionnons brièvement l’existence d’un paramètre « ImageStoreConnectionString », sans décrire ce qu’il signifie vraiment. Et, après avoir parcouru un article comme [Déployer et supprimer des applications à l’aide de PowerShell][10], il semble que vous vous contentez de copier/coller la valeur telle qu’elle apparaît dans le manifeste du cluster cible. Le paramètre doit donc être configurable par cluster, mais, lorsque vous créez un cluster avec le [portail Azure][11], il n’y a pas d’option permettant de configurer ce paramètre, qui est toujours « fabric:ImageStore ». Dans ce cas, quelle est l’utilité de ce paramètre ?

![Manifeste de cluster][img_cm]

Service Fabric était à l’origine une plateforme destinée à la consommation interne de Microsoft par de nombreuses équipes très diverses ; par conséquent, certains de ses aspects sont hautement personnalisables, et notamment le « magasin d’images ». En substance, le magasin d’images est un référentiel enfichable permettant de stocker des packages d’applications. Lorsque votre application est déployée sur un nœud du cluster, ce nœud télécharge le contenu du package de votre application à partir du magasin d’images. ImageStoreConnectionString est un paramètre qui comprend toutes les informations nécessaires pour que les clients et les nœuds trouvent le bon magasin d’images pour un cluster donné.

Il existe actuellement trois genres possibles de fournisseurs de magasins d’images ; les chaînes de connexion correspondantes sont les suivantes :

1. Service de magasin d’images : « fabric:ImageStore »

2. Système de fichiers : « file:[chemin d’accès du système de fichiers] »

3. Stockage Azure : « xstore:DefaultEndpointsProtocol=https;AccountName=[...];AccountKey=[...];Container=[...] »

Le type de fournisseur utilisé en production est le service de magasin d’images, qui est un service système persistant avec état, visible dans Service Fabric Explorer. 

![Service de magasin d’images][img_is]

Le fait d’héberger le magasin d’images dans un service système au sein même du cluster élimine les dépendances externes du référentiel de packages et nous donne davantage de contrôle sur le lieu de stockage. Les améliorations à venir du magasin d’images cibleront probablement le fournisseur de magasin d’images en premier lieu, si ce n’est exclusivement. La chaîne de connexion du fournisseur de service de magasin d’images ne contient aucune information unique, dans la mesure où le client est déjà connecté au cluster cible. Le client a seulement besoin de savoir que les protocoles ciblant le service système doivent être utilisés.

Le fournisseur du système de fichiers est utilisé à la place du service de magasin d’images pour les clusters à boîtier unique locaux pendant le développement, de façon à démarrer le cluster un peu plus vite. La différence est généralement faible, mais c’est une optimisation utile à la plupart des gens lors du développement. Il est également possible de déployer un cluster à boîtier unique local avec les autres types de fournisseurs de stockage, mais il n’y a généralement aucune raison de le faire dans la mesure où le flux de travail de développement / test reste le même quel que soit le fournisseur. Le fournisseur de stockage Azure n’est disponible que pour la prise en charge héritée des anciens clusters ayant été déployés avant l’arrivée du fournisseur de service du magasin d’images.

En outre, ni le fournisseur du système de fichiers, ni le fournisseur de stockage Azure ne doivent être utilisés pour partager un magasin d’images entre plusieurs clusters. En effet, cela endommagerait les données de configuration de cluster, puisque chaque cluster peut écrire des données conflictuelles dans le magasin d’images. Pour partager des packages d’applications provisionnées entre plusieurs clusters, utilisez des fichiers [sfpkg][12], que vous pouvez charger sur n’importe quel magasin externe ayant un URI de téléchargement.

Par conséquent, si ImageStoreConnectionString est configurable, on utilise simplement le paramètre par défaut. En cas de publication sur Azure avec Visual Studio, le paramètre est automatiquement défini en conséquence. Pour un déploiement par programmation sur des clusters hébergés dans Azure, la chaîne de connexion est toujours « fabric:ImageStore ». En cas de doute, sa valeur peut toujours être vérifiée en récupérant le manifeste de cluster par [PowerShell](/powershell/module/servicefabric/get-servicefabricclustermanifest), [.NET](/previous-versions/azure/reference/mt161375(v=azure.100)) ou [REST](/rest/api/servicefabric/get-a-cluster-manifest). Les clusters de test et de production locaux doivent également être toujours configurés pour utiliser le fournisseur de service de magasin d’images.

### <a name="next-steps"></a>Étapes suivantes
[Déployer et supprimer des applications avec PowerShell][10]

<!--Image references-->
[img_is]: ./media/service-fabric-image-store-connection-string/image_store_service.png
[img_cm]: ./media/service-fabric-image-store-connection-string/cluster_manifest.png

[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-cluster-creation-via-portal.md
[12]: service-fabric-package-apps.md#create-an-sfpkg
