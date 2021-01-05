---
title: Paramétrage du cluster Avere vFXT - Azure
description: Découvrez quelques réglages personnalisés que vous pouvez appliquer aux clusters vFXT dans Average vFXT pour Azure, en collaboration avec un représentant du support technique.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 12/19/2019
ms.author: rohogue
ms.openlocfilehash: 5d9f81c9438cb992f81bd3e6319532d67db75552
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88272380"
---
# <a name="cluster-tuning"></a>Paramétrage du cluster

La plupart des clusters vFXT peuvent bénéficier de paramètres de performances personnalisés. Ces paramètres permettent au cluster de mieux fonctionner avec votre flux de travail particulier, le jeu de données et les outils.

Cette personnalisation doit être effectuée avec un technicien du support technique, car elle peut impliquer la configuration de fonctionnalités qui ne sont pas disponibles à partir du Panneau de configuration Avere.

Cette section décrit certaines opérations de paramétrage personnalisé qui peuvent être effectuées.

## <a name="general-optimizations"></a>Optimisations générales

Ces modifications peuvent être recommandées selon les qualités du jeu de données ou le style de flux de travail.

* Si la charge de travail est lourde en écriture, augmentez la taille par défaut du cache en écriture de 20 %.
* Si le jeu de données implique beaucoup de petits fichiers, augmentez la limite du nombre de fichiers du cache du cluster.
* Si le travail implique la copie ou le déplacement de données entre deux dépôts, ajustez le nombre de threads utilisés pour le déplacement des données :
  * Pour améliorer la vitesse, vous pouvez augmenter le nombre de threads parallèles utilisés.
  * Si le volume de stockage back-end est surchargé, vous devrez peut-être réduire le nombre de threads parallèles utilisés.
* Si le cluster met en cache des données pour un système de stockage principal (core filer) qui utilise des listes de contrôle d’accès (ACL) NFSv4, activez la mise en cache du mode d’accès afin de simplifier l’autorisation de fichier pour les clients particuliers.

## <a name="cloud-nas-or-cloud-gateway-optimizations"></a>Optimisations de serveur NAS ou de passerelle cloud

Dans un scénario de serveur NAS ou de passerelle cloud, le cluster vFXT fournit un accès de type NAS vers un conteneur cloud. Pour tirer parti des vitesses de données plus élevées entre le cluster vFXT et le stockage cloud, le technicien peut vous conseiller de modifier des paramètres pour effectuer de manière plus intense une transmission des données de type push vers le volume de stockage à partir du cache. Par exemple :

* Augmenter le nombre de connexions TCP entre le cluster et le conteneur de stockage

## <a name="cloud-bursting-or-hybrid-wan-optimizations"></a>Optimisations de réseau étendu hybride ou de cloud bursting

Dans un scénario de cloud bursting ou d’optimisation de réseau étendu de stockage hybride, le cluster vFXT assure une intégration entre le cloud et le stockage matériel local. Ces modifications peuvent être utiles pour ce qui suit :

* Augmenter le nombre de connexions TCP autorisées entre le cluster et le système de stockage principal
* Activer le paramètre d’optimisation de réseau étendu pour le système de stockage principal à distance (ce paramètre peut être utilisé pour un système de stockage local à distance ou un système de stockage principal cloud dans une autre région Azure).
* Augmenter la taille de la mémoire tampon du socket TCP<sup>*</sup>
* Activer le paramètre de transfert systématique afin de réduire les fichiers mis en cache de manière redondante<sup>*</sup>

<sup>*</sup>Ces ajustements peuvent ne pas s’appliquer à tous les systèmes, selon les besoins en charges de travail et performances.

## <a name="help-optimizing-your-avere-vfxt-for-azure"></a>Optimisation de votre système Avere vFXT pour Azure

Pour contacter le personnel du support technique au sujet de ces optimisations, suivez la procédure décrite dans [Obtenir de l’aide avec votre système](avere-vfxt-open-ticket.md).
