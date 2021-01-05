---
title: Redondance régionale et récupération après un basculement régional avec Azure HPC Cache
description: Techniques fournissant des fonctionnalités de basculement pour la reprise d’activité avec Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: 9159807f55ae52393b8fccec339fcc94c3e4ebb0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87061371"
---
# <a name="use-multiple-caches-for-regional-failover-recovery"></a>Utiliser plusieurs caches pour la récupération après un basculement régional

Chaque instance Azure HPC Cache s’exécute dans un abonnement et dans une région. Cela signifie que votre workflow de cache peut être interrompu si la région connaît une panne totale.

Cet article décrit une stratégie permettant de réduire les risques d’interruption de travail à l’aide d’une deuxième région pour le basculement du cache.

La clé utilise un stockage back-end accessible à partir de plusieurs régions. Ce stockage peut être soit un système NAS local avec une prise en charge DNS appropriée, soit un Stockage Blob Azure qui réside dans une autre région que celle du cache.

À mesure que votre workflow progresse dans votre région primaire, les données sont enregistrées dans le stockage à long terme en dehors de la région. Si la zone de cache n’est plus disponible, vous pouvez dupliquer une instance Azure HPC Cache dans une région secondaire, vous connecter au même stockage et reprendre le travail à partir du nouveau cache.

## <a name="planning-for-regional-failover"></a>Planification du basculement régional

Pour préparer un cache à l’éventualité d’un basculement, effectuez les étapes suivantes :

1. Vérifiez que votre stockage back-end est bien accessible dans une deuxième région.
1. Si vous prévoyez de créer l’instance de cache primaire, vous devez également vous préparer à répliquer ce processus de configuration dans la deuxième région. Vous devez y inclure les éléments suivants :

   1. Un réseau virtuel et une structure de sous-réseau
   1. La capacité du cache
   1. Les informations, les noms et les chemins d’espace de noms de la cible de stockage
   1. Les informations concernant les ordinateurs clients, si ceux-ci se trouvent dans la même région que le cache
   1. Commande de montage devant être utilisée par les clients de cache

   > [!NOTE]
   > Azure HPC Cache peut être créé par programmation, soit par le biais d’un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md), soit en accédant directement à son API. Pour plus d’informations, contactez l’équipe Azure HPC Cache.

## <a name="failover-example"></a>Exemple de basculement

Par exemple, imaginons que vous souhaitiez placer votre instance Azure HPC Cache dans la région Azure USA Est. Elle accédera aux données stockées dans votre centre de données local.

Vous pouvez utiliser un cache dans la région USA Ouest 2 comme sauvegarde de basculement.

Lorsque vous créez le cache dans la région USA Est, préparez un deuxième cache pour le déploiement dans la région USA Ouest 2. Vous pouvez utiliser des scripts ou des modèles pour automatiser cette préparation.

En cas de défaillance de l’ensemble de la région USA Est, créez le cache que vous avez préparé dans la région USA Ouest 2.

Une fois le cache créé, ajoutez des cibles de stockage qui pointent vers les mêmes magasins de données locaux, et utilisez les mêmes chemins d’espace de noms agrégés que les cibles de stockage de l’ancien cache.

Si les clients d’origine sont affectés, créez de nouveaux clients dans la région USA Ouest 2 afin de les utiliser avec le nouveau cache.

Tous les clients doivent monter le nouveau cache, même ceux qui ne sont pas affectés par la panne régionale. Le nouveau cache a des adresses de montage différentes de celles de l’ancien cache.

## <a name="learn-more"></a>En savoir plus

Le Guide de l’architecture des applications Azure contient des informations supplémentaires sur la [reprise d’activité après une interruption de service régionale](<https://docs.microsoft.com/azure/architecture/resiliency/recovery-loss-azure-region>).
