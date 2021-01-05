---
title: Présentation de Microsoft Azure StorSimple Virtual Array
description: Décrit StorSimple Virtual Array, une solution de stockage intégrée qui gère les tâches de stockage entre un groupe virtuel local et un stockage cloud Microsoft Azure.
author: alkohli
ms.assetid: 169c639b-1124-46a5-ae69-ba9695525b77
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 32781a83aec996b23f161f5fe695f39a0de38685
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76273866"
---
# <a name="introduction-to-the-storsimple-virtual-array"></a>Présentation de StorSimple Virtual Array

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

Microsoft Azure StorSimple Virtual Array est une solution de stockage intégrée qui gère les tâches de stockage entre un groupe virtuel local exécuté dans un hyperviseur et le stockage cloud Microsoft Azure. Le groupe virtuel est un serveur de fichiers ou une solution de serveur iSCSI efficace, abordable et facile à gérer qui élimine la plupart des problèmes et des frais liés à la protection des données et du stockage d’entreprise. Le groupe virtuel est particulièrement bien adapté pour le stockage des données d’archivage rarement utilisées.

Cet article fournit une vue d’ensemble du groupe virtuel. Voici d’autres ressources :

* Pour parcourir les bonnes pratiques, voir [Bonnes pratiques liées à StorSimple Virtual Array](storsimple-ova-best-practices.md).
* Pour une vue d’ensemble de la gamme StorSimple 8000, consultez la page [Gamme StorSimple 8000 : une solution de stockage de cloud hybride](storsimple-overview.md).
* Pour en savoir plus sur les appareils de la gamme StorSimple 5000/7000, consultez l’[aide en ligne de StorSimple](http://onlinehelp.storsimple.com/).

Virtual Array prend en charge le protocole iSCSI ou le protocole SMB (Server Message Block). Il s'exécute sur l'infrastructure de l'hyperviseur existante et fournit la hiérarchisation du cloud, de la sauvegarde sur le cloud, une restauration rapide, une récupération au niveau de l'élément, et des fonctionnalités de récupération d'urgence.

Le tableau suivant récapitule les fonctionnalités importantes de StorSimple Virtual Array.

| Fonctionnalité | Baie virtuelle StorSimple |
| --- | --- |
| Configuration requise |Utilise l'infrastructure de virtualisation (Hyper-V ou VMware) |
| Disponibilité |Nœud unique |
| Capacité totale (y compris le cloud) |Jusqu’à 64 To de capacité utilisable par groupe virtuel |
| Capacité locale |De 390 Go à 6,4 To de capacité utilisable par groupe virtuel (il est nécessaire de configurer entre 500 Go et 8 To d’espace disque) |
| Protocoles natifs |iSCSI ou SMB |
| Objectif de délai de récupération (RTO) |iSCSI : moins de 2 minutes, quelle que soit la taille |
| Objectif de point de récupération (RPO) |Sauvegardes quotidiennes et sauvegardes à la demande |
| Hiérarchisation du stockage |Utilise le mappage thermique pour déterminer quelles données doivent être hiérarchisées ou non |
| Support |Infrastructure de virtualisation prise en charge par le fournisseur |
| Performances |Varie en fonction de l'infrastructure sous-jacente |
| Mobilité des données |Peut restaurer sur le même appareil ou effectuer une récupération au niveau de l'élément (serveur de fichiers) |
| Niveaux de stockage |Cloud et stockage sur l'hyperviseur local |
| Taille du partage |À plusieurs niveaux : jusqu'à 20 To ; épinglé localement : jusqu'à 2 To |
| Taille du volume |Hiérarchisé : 500 Go à 5 To ; épinglé localement : 50 Go à 200 Go <br> Réservation locale maximale de 200 Go pour les volumes hiérarchisés. |
| Instantanés |Cohérence en cas d’incident |
| Récupération au niveau de l'élément |Oui. Les utilisateurs peuvent restaurer à partir de partages |

## <a name="why-use-storsimple"></a>Pourquoi utiliser StorSimple ?

StorSimple connecte les utilisateurs et les serveurs au stockage Azure en quelques minutes, sans modification d'application.

Le tableau suivant décrit certains des principaux avantages offerts par la solution StorSimple Virtual Array.

| Fonctionnalité | Avantage |
| --- | --- |
| Intégration transparente |Le groupe virtuel prend en charge le protocole iSCSI ou SMB. Le mouvement des données entre le niveau local et le niveau cloud se fait de manière transparente. |
| Coûts de stockage réduits |Avec StorSimple, vous configurez un stockage local suffisant pour répondre aux besoins actuels en termes de données à chaud les plus utilisées. Lorsque les besoins de stockage augmentent, StorSimple fait passer les données brutes vers le stockage cloud économique. Les données sont dédupliquées et compressées avant l’envoi vers le cloud afin de réduire davantage les dépenses en matière de stockage. |
| Gestion simplifiée du stockage |StorSimple fournit une gestion centralisée dans le cloud, à l’aide de StorSimple Device Manager, afin de gérer plusieurs appareils. |
| Récupération d’urgence et conformité améliorées |StorSimple facilite la récupération d’urgence plus rapide en restaurant les métadonnées immédiatement et en restaurant les données en fonction des besoins. Cela signifie que les opérations normales peuvent se poursuivre, avec une interruption minimale. |
| Mobilité des données |Les données hiérarchisées sur le cloud sont accessibles depuis d'autres sites à des fins de récupération et de migration. Notez que vous pouvez uniquement restaurer les données vers la solution Virtual Array d’origine. Toutefois, vous utilisez les fonctionnalités de récupération d’urgence pour restaurer l’intégralité des données Virtual Array vers un autre système Virtual Array. |

## <a name="storsimple-workload-summary"></a>Résumé des charges de travail StorSimple

Voici un tableau résumant les charges de travail StorSimple prises en charge.

|Scénario     |Charge de travail     |Prise en charge      |Restrictions               | Versions applicables|
|-------------|-------------|---------------|---------------------------|--------------------|
|Bureau à distance/Succusale  |Partage de fichiers     |Oui      |Consultez les [limites maximales pour le serveur de fichiers](storsimple-ova-limits.md).<br></br>Consultez la [configuration système requise pour les versions SMB prises en charge](storsimple-ova-system-requirements.md).| Toutes les versions     |
|Archivage cloud  |Partage de fichiers d’archivage     |Oui      |Consultez les [limites maximales pour le serveur de fichiers](storsimple-ova-limits.md).<br></br>Consultez la [configuration système requise pour les versions SMB prises en charge](storsimple-ova-system-requirements.md).| Toutes les versions     |

Le groupe virtuel StorSimple Virtual Array est particulièrement bien adapté pour les données rarement utilisées. Le groupe virtuel a un cache local pour optimiser les performances, mais les utilisateurs doivent prendre en compte que l’appareil traite les fichiers au niveau inférieur du stockage (le cloud). Chaque groupe virtuel peut écrire et lire des données dans le stockage Azure à une vitesse d’environ 100 Mbits/s. Ce lien est partagé entre toutes les demandes entrantes sur l’appareil et peut devenir un goulot d’étranglement comme illustré dans le diagramme ci-dessous.

![Archivage cloud](./media/storsimple-ova-overview/cloud-archiving.png)

Quand plusieurs utilisateurs simultanés accèdent au groupe virtuel, ils partagent la même connexion à Azure, ce qui diminue les performances. Les performances peuvent varier d’un utilisateur à l’autre, et l’appareil traite les demandes individuellement au fur et à mesure de leur réception.

StorSimple Virtual Array n’est pas adapté aux charges de travail qui nécessitent une haute disponibilité. Le groupe virtuel est un appareil à nœud unique qui subit des temps d’arrêt au moment de l’installation des mises à jour logicielles. Les administrateurs doivent planifier une fenêtre de maintenance de 30 minutes trois ou quatre fois par an.

## <a name="workflows"></a>Workflows

StorSimple Virtual Array est particulièrement adapté aux flux de travail suivants :

* [Gestion du stockage sur le cloud](#cloud-based-storage-management)
* [Sauvegarde indépendante de l'emplacement](#location-independent-backup)
* [Récupération d'urgence et protection des données](#data-protection-and-disaster-recovery)

### <a name="cloud-based-storage-management"></a>Gestion du stockage sur le cloud
Vous pouvez utiliser le service StorSimple Device Manager en cours d’exécution dans le portail Azure pour gérer les données stockées sur plusieurs appareils et à plusieurs emplacements. Ceci est particulièrement utile dans les scénarios de succursales distribuées. Notez que vous devez créer des instances distinctes du service StorSimple Device Manager pour gérer les instances Virtual Array et les appareils StorSimple physiques. Notez également que l’instance Virtual Array utilise désormais le nouveau portail Azure au lieu du portail Azure Classic.

![Gestion du stockage sur le cloud](./media/storsimple-ova-overview/cloud-based-storage-management.png)

### <a name="location-independent-backup"></a>Sauvegarde indépendante de l'emplacement
Avec Virtual Array, les instantanés cloud fournissent une copie indépendante de l’emplacement et ponctuelle d’un volume ou d’un partage. Les instantanés cloud sont activés par défaut et ne peuvent pas être désactivés. Tous les volumes et partages sont sauvegardés en même temps selon une stratégie de sauvegarde quotidienne unique, et vous pouvez effectuer des sauvegardes ad hoc supplémentaires chaque fois que c’est nécessaire.

### <a name="data-protection-and-disaster-recovery"></a>Récupération d'urgence et protection des données
Virtual Array prend en charge les scénarios de récupération d’urgence et de protection des données suivants :

* **Restauration de volume ou de partage** : utilisez la restauration en tant que nouveau flux de travail pour récupérer un volume ou un partage. Cette approche permet de récupérer la totalité du volume ou du partage.
* **Récupération au niveau de l’élément** : les partages permettent d’accéder facilement aux sauvegardes récentes. Vous pouvez facilement récupérer un fichier individuel à partir d’un dossier *.backup* spécial, disponible dans le cloud. Cette fonctionnalité de restauration est contrôlée par l'utilisateur et aucune intervention de l'administrateur n'est nécessaire.
* **Récupération d’urgence** : utilisez la fonctionnalité de basculement pour récupérer tous les volumes ou les partages vers un nouveau groupe virtuel. Vous créez l’instance Virtual Array et l’inscrivez auprès du service StorSimple Device Manager, puis vous basculez le système Virtual Array d’origine. Le nouveau Virtual Array prend alors en charge les ressources configurées.

## <a name="storsimple-virtual-array-components"></a>Composants du système StorSimple Virtual Array

Virtual Array comprend les composants suivants :

* [Groupe virtuel](#virtual-array) : appareil de stockage cloud hybride basé sur une machine virtuelle provisionnée dans votre hyperviseur ou environnement virtualisé.
* [Service StorSimple Device Manager](#storsimple-device-manager-service) : extension du portail Azure, qui vous permet de gérer un ou plusieurs appareils StorSimple à partir d’une interface web unique, accessible depuis divers emplacements géographiques. Vous pouvez utiliser le service StorSimple Device Manager pour créer et gérer les services, afficher et gérer les appareils et les alertes, et administrer les volumes, les partages et les instantanés existants.
* [Interface utilisateur web locale](#local-web-user-interface) : interface utilisateur web qui est utilisée pour configurer l’appareil afin qu’il puisse se connecter au réseau local, puis pour inscrire l’appareil auprès du service StorSimple Device Manager. 
* [Interface de ligne de commande](#command-line-interface) : interface Windows PowerShell que vous pouvez utiliser pour démarrer une session de support pour Virtual Array.
  Les sections suivantes décrivent chacun de ces composants plus en détail, et expliquent comment la solution organise les données, alloue le stockage et facilite la gestion du stockage et la protection des données.

### <a name="virtual-array"></a>Virtual Array

Virtual Array est une solution de stockage à nœud unique qui offre un stockage principal, gère la communication avec le stockage cloud et contribue à garantir la sécurité et la confidentialité de toutes les données stockées sur l’appareil.

Cette solution est disponible dans un modèle qui peut être téléchargé. Virtual Array propose une capacité maximale de 6,4 To sur l’appareil (avec une exigence de stockage sous-jacent de 8 To) et 64 To, y compris le stockage cloud.

Virtual Array présente les caractéristiques suivantes :

* Il est économique. Il utilise votre infrastructure de virtualisation existante et peut être déployé sur votre hyperviseur Hyper-V ou VMware existant.
* Il réside dans le centre de données et peut être configuré comme un serveur iSCSI ou un serveur de fichiers.
* Il est intégré au cloud.
* Les sauvegardes sont stockées dans le cloud, ce qui peut faciliter la récupération d'urgence et simplifier la récupération au niveau de l'élément (ILR).
* Vous pouvez appliquer des mises à jour à Virtual Array, comme pour un appareil physique.

> [!NOTE]
> Une instance Virtual Array ne peut pas être développée. Par conséquent, il est important de configurer suffisamment de stockage lorsque vous créez le groupe virtuel.

### <a name="storsimple-device-manager-service"></a>Service StorSimple Device Manager

Microsoft Azure StorSimple fournit une interface utilisateur web (le service StorSimple Device Manager) qui vous permet de gérer le stockage StorSimple de manière centralisée. Vous pouvez utiliser le service StorSimple Device Manager pour effectuer les tâches suivantes :

* Gérer plusieurs StorSimple Virtual Arrays à partir d'un service unique.
* Configurer et gérer les paramètres de sécurité pour les instances StorSimple Virtual Array. (Le chiffrement dans le cloud est dépendant de l'API Microsoft Azure).
* Configurer les propriétés et les informations d'identification du compte de stockage.
* Configurer et gérer des volumes ou des partages.
* Sauvegarder et restaurer des données sur des volumes ou des partages.
* Analyser les performances.
* Passer en revue les paramètres système et identifier les problèmes possibles.

Le service StorSimple Device Manager permet d’effectuer des tâches d’administration quotidiennes sur votre instance Virtual Array.

Pour en savoir plus, voir [Utiliser le service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-virtual-array-manager-service-administration.md).

### <a name="local-web-user-interface"></a>Interface utilisateur web locale

Virtual Array inclut une interface utilisateur web, qui est utilisée pour la configuration à usage unique et l’inscription de l’appareil auprès du service StorSimple Device Manager. Vous pouvez l’utiliser pour arrêter et redémarrer Virtual Array, exécuter des tests de diagnostic, mettre à jour le logiciel, modifier le mot de passe administrateur de l’appareil, afficher les journaux d’activité système et contacter le Support Microsoft pour créer une demande de service.

Pour plus d’informations sur l’utilisation de l’interface utilisateur basée sur le web, voir [Utiliser l’interface utilisateur le web pour gérer votre Groupe virtuel StorSimple](storsimple-ova-web-ui-admin.md).

### <a name="command-line-interface"></a>Interface de ligne de commande

L’interface Windows PowerShell incluse vous permet de lancer une session de support auprès du Support Microsoft afin que l’équipe de support vous aide à résoudre les problèmes que vous rencontrez sur votre groupe virtuel.

## <a name="storage-management-technologies"></a>Technologies de gestion de stockage

En plus de Virtual Array et d’autres composants, la solution StorSimple utilise les technologies logicielles suivantes pour fournir un accès rapide aux données importantes, réduire la consommation du stockage et protéger les données stockées sur votre instance Virtual Array :

* [Hiérarchisation automatique du stockage](#automatic-storage-tiering) 
* [Partages et volumes épinglés localement](#locally-pinned-shares-and-volumes)
* Déduplication et compression des données à plusieurs niveaux ou sauvegardées dans le cloud 
* [Sauvegardes à la demande et planifiées](#scheduled-and-on-demand-backups)

### <a name="automatic-storage-tiering"></a>Hiérarchisation automatique du stockage
Virtual Array utilise un nouveau mécanisme de hiérarchisation pour gérer les données stockées dans Virtual Array et le cloud. Il n’existe que deux niveaux : une instance Virtual Array locale et le stockage cloud Azure. StorSimple Virtual Array organise automatiquement les données dans les niveaux selon une carte thermique, qui effectue le suivi de l'utilisation actuelle, l'âge et relations par rapport à d'autres données. Les données les plus actives (les plus chaudes) sont stockées localement, tandis que les données les moins actives ou inactives sont automatiquement transférées vers le cloud. (Toutes les sauvegardes sont stockées dans le cloud). StorSimple ajuste et réorganise les données et modifie les affectations de stockage conformément aux modifications du schéma d’utilisation. Par exemple, certaines informations peuvent devenir moins actives au fil du temps. Lorsque cela se produit, les données sont hiérarchisées vers le cloud. Si ces mêmes données redeviennent actives, elles sont hiérarchisées vers Storage Array.

Les données d’un partage ou volume hiérarchisé conservent toujours leur propre espace de niveau local (environ 10 % de l’espace total provisionné pour ce partage ou volume). Bien que cela réduise la capacité de stockage disponible sur le groupe virtuel pour ce partage ou volume, cela permet de garantir que la hiérarchisation pour un partage ou un volume ne sera pas affectée par les besoins de hiérarchisation d’autres partages ou volumes. Par conséquent, une charge de travail très occupée sur un partage ou un volume ne peut pas forcer toutes les autres charges de travail vers le cloud.

Les volumes hiérarchisés créés pour iSCSI ont une réservation locale maximale de 200 Go, quelle que soit la taille du volume.

![Hiérarchisation automatique du stockage](./media/storsimple-ova-overview/automatic-storage-tiering.png)

> [!NOTE]
> Vous pouvez spécifier qu’un volume est épinglé localement, auquel cas les données restent sur Virtual Array et ne sont jamais hiérarchisées vers le cloud. Pour plus d’informations, voir [Partages et volumes épinglés localement](#locally-pinned-shares-and-volumes).


### <a name="locally-pinned-shares-and-volumes"></a>Partages et volumes épinglés localement

Vous pouvez créer des partages et volumes appropriés comme étant épinglés localement. Cette fonctionnalité garantit que les données requises par les applications critiques restent dans Virtual Array et ne sont jamais hiérarchisées vers le cloud. Les partages et volumes épinglés localement ont les fonctionnalités suivantes :

* Ils ne sont pas soumis aux latences de cloud ou à des problèmes de connectivité.
* Ils bénéficient toujours des fonctionnalités de récupération d'urgence et de sauvegarde cloud de StorSimple.

Vous pouvez restaurer un partage ou volume épinglé localement sous la forme à plusieurs niveaux, ou restaurer un partage ou volume à plusieurs niveaux sous la forme épinglé localement. 

Pour en savoir plus sur les volumes épinglés localement, voir [Utiliser le service StorSimple Device Manager pour gérer les volumes sur l’instance StorSimple Virtual Array](storsimple-virtual-array-manage-volumes.md).

### <a name="deduplication-and-compression-for-data-tiered-or-backed-up-to-the-cloud"></a>Déduplication et compression des données à plusieurs niveaux ou sauvegardées dans le cloud

StorSimple utilise la compression des données et la déduplication pour réduire encore davantage les besoins de stockage sur le cloud. La déduplication réduit la quantité globale de données stockées en éliminant les redondances dans le jeu de données stocké. Lorsque des données sont modifiées, StorSimple ignore les informations non modifiées et enregistre uniquement les modifications. En outre, StorSimple réduit la quantité de données stockées en identifiant et en supprimant les informations dupliquées.

> [!NOTE]
> Les données stockées sur Virtual Array ne sont ni dédupliquées, ni compressées. La déduplication et la compression se produisent juste avant que les données ne soient envoyées vers le cloud.

### <a name="scheduled-and-on-demand-backups"></a>Sauvegardes à la demande et planifiées

Les fonctionnalités de protection des données de StorSimple permettent de créer des sauvegardes à la demande. En outre, une planification de sauvegarde par défaut garantit que les données sont sauvegardées quotidiennement. Les sauvegardes sont effectuées sous la forme d'instantanés incrémentiels, qui sont stockés dans le cloud. Les instantanés, qui enregistrent uniquement les modifications apportées depuis la dernière sauvegarde, peuvent être créés et restaurés rapidement. Ces instantanés peuvent être d'une importance critique dans les scénarios de récupération d'urgence, car ils remplacent les systèmes de stockage secondaire (comme la sauvegarde sur bande) et vous permettent de restaurer des données sur votre centre de données ou sur d'autres sites si nécessaire.

## <a name="managing-personal-information"></a>Gestion des informations personnelles

StorSimple Device Manager pour les séries virtuelles collecte des informations personnelles principalement dans deux cas :
 - Paramètres utilisateur d’alerte où l’adresse e-mail des utilisateurs est configurée. L’administrateur a la possibilité d’effacer ces données. 
 - Utilisateurs qui ont accès aux données qui se trouvent sur les partages. La liste de ces utilisateurs s’affiche et peut être exportée. Elle est par ailleurs supprimée à la suppression des partages.

Pour plus d’informations, consultez la [Politique de confidentialité Microsoft sur le Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [préparer le portail du groupe virtuel](storsimple-virtual-array-deploy1-portal-prep.md).
