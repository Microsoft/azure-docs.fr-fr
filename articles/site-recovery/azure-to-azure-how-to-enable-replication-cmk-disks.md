---
title: Activer la réplication des machines virtuelles Azure chiffrées dans Azure Site Recovery
description: Cet article décrit comment configurer la réplication pour les machines virtuelles avec des disques activés par les clés gérées par le client d’une région Azure à l’autre à l’aide de Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/10/2020
ms.author: mayg
ms.openlocfilehash: 9f9052f51c5bab0ea738e9fd15d8f62f45ff0c9b
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146533"
---
# <a name="replicate-machines-with-customer-managed-keys-cmk-enabled-disks"></a>Répliquer des machines avec des disques activées par les clés gérées par le client

Cet article explique comment répliquer des machines virtuelles Azure avec des disques managés activés par les clés gérées par le client d’une région Azure à l’autre.

## <a name="prerequisite"></a>Configuration requise
Vous devez créer le ou les jeux de chiffrement de disque dans la région cible de l’abonnement en question avant d’activer la réplication de vos machines virtuelles disposant de disques managés activés par des clés gérées par le client.

## <a name="enable-replication"></a>Activer la réplication

Pour cet exemple, la région principale Azure est Asie Est et la région secondaire Asie Sud-Est.

1. Dans le coffre, sélectionnez **+Répliquer**.
2. Notez les champs suivants.
    - **Source** : point d’origine des machines virtuelles, qui dans ce cas est **Azure**.
    - **Emplacement source**  : région Azure où vous souhaitez protéger vos machines virtuelles. Dans cet exemple, l’emplacement source est « Asie Est ».
    - **Modèle de déploiement**  : modèle de déploiement Azure des machines sources.
    - **Abonnement source**  : abonnement auquel vos machines virtuelles sources appartiennent. Il peut s’agir de n’importe quel abonnement au sein du même locataire Azure Active Directory où se trouve votre coffre Recovery services.
    - **Groupe de ressources** : groupe de ressources auquel appartiennent vos machines virtuelles sources. Toutes les machines virtuelles du groupe de ressources sélectionné sont listées pour la protection à l’étape suivante.

3. Dans **Machines virtuelles** > **Sélectionner les machines virtuelles** , sélectionnez chaque machine à répliquer. Vous pouvez uniquement sélectionner les machines pour lesquelles la réplication peut être activée. Ensuite, sélectionnez **OK**.

4. Dans **Paramètres** , vous pouvez configurer les paramètres de site/cible suivants.

    - **Emplacement cible** : Emplacement où vos données de machines virtuelles sources sont répliquées. Site Recovery fournit la liste des régions cibles appropriées selon l’emplacement de la machine sélectionnée. Nous vous recommandons d’utiliser le même emplacement que celui du coffre Recovery Services.
    - **Abonnement cible**  : abonnement cible utilisé pour la reprise d’activité. Par défaut, l’abonnement cible est identique à l’abonnement source.
    - **Groupe de ressources cible**  : groupe de ressources auquel appartiennent toutes vos machines virtuelles répliquées. Par défaut, Site Recovery crée un groupe de ressources dans la région cible. Son nom porte le suffixe `asr`. S’il existe déjà un groupe de ressources créé par Azure Site Recovery, il est réutilisé. Vous pouvez également choisir de le personnaliser, comme indiqué dans la section suivante. L’emplacement du groupe de ressources cible peut être n’importe quelle région Azure à l’exception de la région dans laquelle les machines virtuelles sources sont hébergées.
    - **Réseau virtuel cible** : Par défaut, Site Recovery crée un réseau virtuel dans la région cible. Son nom porte le suffixe `asr`. Il est mappé à votre réseau source et utilisé pour toute protection ultérieure. [En savoir plus](./azure-to-azure-network-mapping.md) sur le mappage réseau.
    - **Comptes de stockage cibles (si votre machine virtuelle source n’utilise pas de disques managés)**  : par défaut, Site Recovery crée un compte de stockage cible avec la même configuration que celle du compte de stockage de machines virtuelles source. S’il existe déjà un compte de stockage, il est réutilisé.
    - **Disques managés de réplica (si votre machine virtuelle source utilise des disques managés)**  : Site Recovery crée des disques managés de réplica dans la région cible pour refléter les disques managés de la machine virtuelle source du même type de stockage (Standard ou Premium).
    - **Comptes de stockage de cache** : Site Recovery a besoin d’un compte de stockage supplémentaire appelé *stockage de cache* dans la région source. Toutes les modifications apportées sur les machines virtuelles sources sont suivies et envoyées au compte de stockage de cache. Elles sont ensuite répliquées sur l’emplacement cible.
    - **Groupe à haute disponibilité**  : Par défaut, Site Recovery crée un groupe à haute disponibilité dans la région cible. Son nom porte le suffixe `asr`. S’il existe déjà un groupe à haute disponibilité créé par Site Recovery, il est réutilisé.
    - **Jeux de chiffrements de disque (DES)**  : Site Recovery a besoin que les jeux de chiffrement de disque soient utilisés pour le réplica et les disques managés cibles. Avant d’activer la réplication, vous devez préalablement créer des jeux de chiffrement de disque dans l’abonnement et la région cibles. Par défaut, aucun jeu de chiffrement de disque n’est sélectionné. Cliquez sur « Personnaliser » pour choisir un jeu de chiffrement de disque par disque source.
    - **Stratégie de réplication**  : définit les paramètres de l’historique de conservation des points de récupération et la fréquence des instantanés de cohérence d’application. Par défaut, Site Recovery crée une stratégie de réplication avec des paramètres par défaut de *24 heures* pour la rétention des points de récupération et *60 minutes* pour la fréquence des instantanés de cohérence d’application.

    ![Activer la réplication pour les machines avec des disques activés par la clé gérée par le client](./media/azure-to-azure-how-to-enable-replication-cmk-disks/cmk-enable-dr.png)

## <a name="customize-target-resources"></a>Personnaliser les ressources cibles

Suivez ces étapes pour modifier les paramètres cibles par défaut de Site Recovery.

1. Sélectionnez **Personnaliser** en regard d’« Abonnement cible » pour modifier l’abonnement cible par défaut. Sélectionnez l’abonnement dans la liste des abonnements disponibles dans le locataire Azure AD.

2. Sélectionnez **Personnaliser** en regard de « Groupe de ressources, Réseau, Stockage et Groupes à haute disponibilité » pour modifier les paramètres par défaut suivants :
    - Pour **Groupe de ressources cible** , sélectionnez le groupe de ressources dans la liste des groupes de ressources à l’emplacement cible de l’abonnement.
    - Pour **Réseau virtuel cible** , sélectionnez le réseau dans la liste des réseaux virtuels à l’emplacement cible.
    - Pour **Groupe à haute disponibilité** , vous pouvez ajouter les paramètres de groupe à haute disponibilité de la machine virtuelle, s’ils font partie d’un groupe à haute disponibilité dans la zone source.
    - Pour **Comptes de stockage cible** , sélectionnez le compte à utiliser.

3. Sélectionnez **Personnaliser** à côté des « paramètres de chiffrement de stockage » afin de sélectionner le jeu de chiffrement de disque cible pour chaque disque source managé activé par une clé gérée par le client. Lors de la sélection, vous pouvez également voir le coffre de clés cible auquel le jeu de chiffrement de disque est associé.

4. Sélectionnez **Créer une ressource cible** > **Activer la réplication**.
5. Une fois que les machines virtuelles sont activées pour la réplication, vous pouvez vérifier leur état d’intégrité sous **Éléments répliqués**.

![Capture d’écran montrant où vérifier l’état d’intégrité des machines virtuelles.](./media/azure-to-azure-how-to-enable-replication-cmk-disks/cmk-customize-target-disk-properties.png)

>[!NOTE]
>Pendant la réplication initiale, l’état peut prendre un certain temps à s’actualiser et sa progression peut ne pas apparaître. Cliquez sur **Actualiser** pour obtenir l’état le plus récent.

## <a name="faqs"></a>FAQ

* J’ai activé la clé gérée par le client sur un élément répliqué existant. Comment puis-je vérifier que la clé gérée par le client est également appliquée à la région cible ?

    Vous pouvez rechercher le nom du disque managé de réplica (créé par Azure Site Recovery dans la région cible) et joindre le jeu de chiffrement de disque à ce disque de réplica. Toutefois, vous ne pourrez pas voir les détails du jeu de chiffrement de disque dans le panneau Disques une fois que vous l’aurez joint. Sinon, vous pouvez également désactiver la réplication de la machine virtuelle et la réactiver. Cela vous permettra de voir les détails du jeu de chiffrement de disque et du coffre de clés dans le panneau Disques pour l’élément répliqué.

* J’ai ajouté un nouveau disque activé par une clé gérée par le client à l’élément répliqué. Comment puis-je répliquer ce disque à l’aide d’Azure Site Recovery ?

    L’ajout d’un nouveau disque activé par une clé gérée par le client à un élément répliqué existant n’est pas pris en charge. Désactivez et réactivez la réplication pour la machine virtuelle.

* J’ai activé des clés gérées par la plateforme et celles par le client. Comment puis-je protéger mes disques ?

    L’activation du double chiffrement avec les clés gérées par la plateforme et celles gérées par le client est prise en charge par Site Recovery. Suivez les instructions de cet article pour protéger votre machine. Vous devez créer à l’avance un DES pour lequel le double chiffrement est activé dans la région cible. Au moment d’activer la réplication pour une machine virtuelle de ce type, vous pouvez fournir ce DES à Site Recovery.

