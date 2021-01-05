---
title: Configurer les paramètres d’Azure HPC Cache
description: Explique comment configurer des paramètres supplémentaires pour le cache, tels que MTU et no-root-squash, et comment accéder aux instantanés express à partir des cibles de stockage Blob Azure.
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 05/06/2020
ms.author: v-erkel
ms.openlocfilehash: b01c4d896d5ec600e0fe22e3ca7b7816141776a4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86497197"
---
# <a name="configure-additional-azure-hpc-cache-settings"></a>Configurer des paramètres Azure HPC Cache supplémentaires

La page **Configuration** du Portail Azure contient des options permettant de personnaliser plusieurs paramètres. La plupart des utilisateurs n’ont pas besoin de modifier les valeurs par défaut de ces paramètres.

Cet article explique également comment utiliser la fonctionnalité d’instantané pour les cibles de stockage Blob Azure. La fonctionnalité d’instantané n’a pas de paramètres configurables.

Pour afficher les paramètres, ouvrez la page **Configuration** du cache dans le Portail Azure.

![capture d’écran de la page de configuration dans le Portail Azure](media/configuration.png)

> [!TIP]
> La [vidéo relative à la gestion d’Azure HPC Cache](https://azure.microsoft.com/resources/videos/managing-hpc-cache/) présente la page de configuration et ses paramètres.

## <a name="adjust-mtu-value"></a>Ajuster la valeur MTU
<!-- linked from troubleshoot-nas article -->

Vous pouvez sélectionner la taille d’unité de transmission maximale pour le cache à l’aide du menu déroulant intitulé **Taille MTU**.

La valeur par défaut est 1500 octets, mais vous pouvez la remplacer par 1400.

> [!NOTE]
> Si vous réduisez la taille MTU du cache, assurez-vous que les clients et les systèmes de stockage qui communiquent avec le cache ont le même paramètre MTU ou une valeur inférieure.

La réduction de la valeur MTU du cache peut vous aider à contourner les restrictions de taille des paquets dans le reste du réseau du cache. Par exemple, certains VPN ne parviennent pas à transmettre des paquets de 1500 octets en taille réelle. La réduction de la taille des paquets envoyés via le VPN peut éliminer ce problème. Toutefois, notez qu’un paramètre MTU inférieur pour le cache signifie que tout autre composant qui communique avec le cache, y compris les clients et les systèmes de stockage, doit également avoir un paramètre MTU inférieur pour éviter des problèmes de communication.

Si vous ne souhaitez pas modifier les paramètres MTU sur d’autres composants système, vous ne devez pas réduire le paramètre MTU du cache. Il existe d’autres solutions pour contourner les restrictions de taille des paquets VPN. Pour en savoir plus sur le diagnostic et la résolution de ce problème, consultez la section [Ajustement des restrictions de taille des paquets VPN](troubleshoot-nas.md#adjust-vpn-packet-size-restrictions) dans l’article de résolution des problèmes NAS.

Pour plus d’informations sur les paramètres MTU dans les réseaux virtuels Azure, consultez [Optimisation des performances TCP/IP pour les machines virtuelles Azure](../virtual-network/virtual-network-tcpip-performance-tuning.md).

## <a name="configure-root-squash"></a>Configurer le squash racine
<!-- linked from troubleshoot -->

Le paramètre **Activer le squash racine** contrôle la manière dont Azure HPC Cache traite les requêtes de l’utilisateur racine sur les ordinateurs clients.

Lorsque le squash racine est activé, les utilisateurs racine d’un client sont automatiquement mappés à l’utilisateur « nobody » lorsqu’ils envoient des demandes via Azure HPC Cache. Cela empêche également les requêtes de clients d’utiliser les bits d’autorisation set-UID.

Si le squash racine est désactivé, une requête de l’utilisateur racine client (UID 0) est transmise à un système de stockage NFS principal en tant que racine. Cette configuration peut permettre un accès inapproprié aux fichiers.

La définition du squash racine sur le cache peut aider à compenser le paramètre ``no_root_squash`` requis sur les systèmes NAS utilisés comme cibles de stockage. (En savoir plus sur la [configuration requise pour les cibles de stockage NFS](hpc-cache-prerequisites.md#nfs-storage-requirements).) Il peut également améliorer la sécurité lorsqu’il est utilisé avec des cibles de stockage Blob Azure.

Le paramètre par défaut est **Oui**. (Les caches créés avant avril 2020 peuvent avoir le paramètre par défaut **Non**.)

## <a name="view-snapshots-for-blob-storage-targets"></a>Afficher les instantanés pour les cibles de stockage blob

Azure HPC Cache enregistre automatiquement les instantanés de stockage pour les cibles de stockage Blob Azure. Les instantanés fournissent un point de référence rapide pour le contenu du conteneur de stockage back-end.

Les instantanés ne remplacent pas les sauvegardes de données et n’incluent aucune information sur l’état des données en cache.

> [!NOTE]
> Cette fonctionnalité d’instantané est différente de celle incluse dans les logiciels de stockage NetApp ou Isilon. Ces implémentations d’instantanés vident les modifications du cache vers le système de stockage back-end avant de prendre l’instantané.
>
> Par souci d’efficacité, l’instantané d’Azure HPC Cache ne vide pas les modifications en premier et enregistre uniquement les données qui ont été écrites dans le conteneur Blob. Cet instantané ne représente pas l’état des données en cache, il peut donc ne pas inclure les modifications récentes.

Cette fonctionnalité n’est disponible que pour les cibles de stockage Blob Azure, et sa configuration ne peut pas être modifiée.

Les instantanés sont pris toutes les huit heures, à 0:00, 08:00 et 16:00 (UTC).

Azure HPC Cache stocke des instantanés quotidiens, hebdomadaires et mensuels jusqu’à ce qu’ils soient remplacés par de nouveaux. Les limites sont les suivantes :

* jusqu’à 20 instantanés quotidiens
* jusqu’à 8 instantanés hebdomadaires
* jusqu’à 3 instantanés mensuels

Accédez aux instantanés à partir du répertoire `.snapshot` dans l’espace de noms de votre cible de stockage blob.
