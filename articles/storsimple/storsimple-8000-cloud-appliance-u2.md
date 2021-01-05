---
title: StorSimple Cloud Appliance Update 3| Microsoft Docs
description: Apprenez à créer, déployer et gérer une StorSimple Cloud Appliance dans un réseau virtuel Microsoft Azure. (S'applique à StorSimple Update 3 et versions ultérieures).
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/08/2017
ms.author: alkohli
ms.openlocfilehash: d78051c1a5af82a986152c8244d25b68dd65d552
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968840"
---
# <a name="deploy-and-manage-a-storsimple-cloud-appliance-in-azure-update-3-and-later"></a>Déployer et gérer une appliance cloud StorSimple dans Azure (Update 3 et versions ultérieures)

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

L’appliance cloud StorSimple série 8000 est une fonctionnalité supplémentaire fournie avec votre solution Microsoft Azure StorSimple. StorSimple Cloud Appliance s’exécute sur une machine virtuelle dans un réseau virtuel Microsoft Azure. Vous pouvez l’utiliser pour sauvegarder et cloner les données à partir de vos ordinateurs hôtes.

Cet article décrit le processus pas à pas pour déployer et gérer une appliance cloud StorSimple dans Azure. À la fin de cet article, vous serez capable :

* de différencier l’appliance cloud de l’appareil physique ;
* de créer et de configurer l’appliance cloud ;
* de vous connecter à l’appliance cloud.
* d’utiliser l’appliance cloud.

Ce didacticiel s’applique à toutes les appliances cloud StorSimple exécutant Update 3 ou une version ultérieure.

#### <a name="cloud-appliance-model-comparison"></a>Comparaison des modèles d’appliance cloud

StorSimple Cloud Appliance est disponible en deux modèles, un 8010 Standard (anciennement 1100) et un 8020 Premium (dans Update 2). Le tableau suivant présente une comparaison des deux modèles.

| Modèle de l'appareil | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **Capacité maximale** |30 To |64 To |
| **Microsoft Azure** |Standard_A3 (4 cœurs, 7 Go de mémoire)| Standard_DS3 (4 cœurs, 14 Go de mémoire)|
| **Disponibilité des régions** |Toutes les régions Azure |Régions Azure qui prennent en charge le stockage Premium et les machines virtuelles Azure DS3<br></br>Utilisez [cette liste](https://azure.microsoft.com/regions/services/) afin de déterminer si **Machines virtuelles > Série DS** et **Stockage > Stockage sur disque** sont disponibles dans votre région. |
| **Type de stockage** |Utilise le stockage Azure Standard pour les disques locaux<br></br> Découvrez comment [créer un compte de stockage Standard](../storage/common/storage-account-create.md) |Utilise le stockage Azure Standard pour les disques locaux<sup>2</sup> <br></br> |
| **Aide relative à la charge de travail** |Récupération au niveau des éléments des fichiers à partir de sauvegardes |Scénarios de développement et de test cloud <br></br>Faible latence et charges de travail aux performances plus élevées<br></br>Appareil secondaire pour la récupération d’urgence |

<sup>1</sup> *Anciennement, le modèle 1100*.

<sup>2</sup> *Le modèle 8010 et le modèle 8020 utilisent tous deux le Stockage Standard Azure pour le niveau cloud. La différence existe uniquement au niveau local au sein de l’appareil*.

## <a name="how-the-cloud-appliance-differs-from-the-physical-device"></a>Comment l’appliance cloud se différencie de l’appareil physique

StorSimple Cloud Appliance est une version logicielle de StorSimple qui s’exécute sur un seul nœud dans une machine virtuelle Microsoft Azure. L’appliance cloud prend en charge les scénarios de récupération d’urgence dans lesquels votre appareil physique n’est pas disponible. L’appliance cloud convient à une utilisation dans les scénarios de récupération au niveau des éléments à partir de sauvegardes, dans les scénarios de récupération d’urgence en local et dans les scénarios de développement et de test dans le cloud.

#### <a name="differences-from-the-physical-device"></a>Différences par rapport à l’appareil physique

Le tableau ci-dessous présente quelques différences clés entre StorSimple Cloud Appliance et l’appareil physique StorSimple.

|  | Appareil physique | Appliance cloud |
| --- | --- | --- |
| **Lieu** |Se trouve dans le centre de données. |S'exécute dans Azure. |
| **Interfaces réseau** |A six interfaces réseau : DATA 0 à DATA 5. |A une seule interface réseau : DATA 0. |
| **Inscription** |Inscrit durant l’étape de configuration initiale. |L'inscription est une tâche distincte. |
| **Clé de chiffrement de données du service** |Régénérer la clé sur l’appareil physique, puis mettre à jour l’appliance cloud avec la nouvelle clé. |Ne peut pas régénérer à partir de l’appliance cloud. |
| **Types de volumes pris en charge** |Prend en charge les volumes hiérarchisés et les volumes attachés localement. |Ne prend en charge que les volumes hiérarchisés. |

## <a name="prerequisites-for-the-cloud-appliance"></a>Configuration requise pour l’appliance cloud

Les sections suivantes décrivent les conditions préalables à StorSimple Cloud Appliance. Avant de déployer une appliance cloud, passez en revue les considérations de sécurité pour l’utilisation d’une appliance cloud.

[!INCLUDE [StorSimple Cloud Appliance security](../../includes/storsimple-8000-cloud-appliance-security.md)]

#### <a name="azure-requirements"></a>Conditions requises pour Azure

Avant d’approvisionner l’appliance cloud, vous devez effectuer les préparatifs suivants dans votre environnement Azure :

* Assurez-vous qu’un appareil physique de la série StorSimple 8000 (modèle 8100 ou 8600) est déployé et exécuté dans votre centre de données. Inscrivez cet appareil avec le même service StorSimple Device Manager pour lequel vous souhaitez créer une appliance cloud StorSimple.
* Pour l’appliance cloud, [configurez un réseau virtuel sur Azure](../virtual-network/manage-virtual-network.md#create-a-virtual-network). Si vous utilisez le stockage Premium, vous devez créer un réseau virtuel dans une région Azure qui prend en charge le stockage Premium. Les régions de stockage Premium sont des régions qui correspondent à la ligne Stockage sur disque dans la liste des [services Azure par région](https://azure.microsoft.com/regions/services/).
* Il est recommandé d’utiliser le serveur DNS par défaut fourni par Azure au lieu de spécifier le nom de votre propre serveur DNS. Si le nom de votre serveur DNS n’est pas valide ou si le serveur DNS n’est pas en mesure de résoudre correctement les adresses IP, la création de l’appliance cloud échoue.
* Les options de point à site et de site à site sont facultatives (non obligatoires). Si vous le souhaitez, vous pouvez configurer ces options pour des scénarios plus avancés.
* Vous pouvez créer des [Machines virtuelles Azure](../virtual-machines/windows/quick-create-portal.md) (serveurs hôtes) dans le réseau virtuel qui peut utiliser les volumes exposés par l’appliance cloud. Ces serveurs doivent répondre aux exigences suivantes :

  * Il doit s’agir de machines virtuelles Windows ou Linux sur lesquelles l’initiateur iSCSI est installé.
  * Ils doivent être en cours d’exécution dans le même réseau virtuel que l’appliance cloud.
  * Ils doivent être en mesure de se connecter à la cible iSCSI de l’appliance cloud via l’adresse IP interne de cette dernière.
  * Vérifiez que vous avez configuré la prise en charge de iSCSI et du trafic cloud sur le même réseau virtuel.

#### <a name="storsimple-requirements"></a>Conditions requises pour StorSimple

Apportez les mises à jour suivantes à votre service StorSimple Device Manager avant de créer une appliance cloud :

* Ajoutez des [enregistrements de contrôle d’accès](storsimple-8000-manage-acrs.md) pour les machines virtuelles qui vont être les serveurs hôtes de votre appliance cloud.
* Utilisez un [compte de stockage](storsimple-8000-manage-storage-accounts.md#add-a-storage-account) dans la même région que l’appliance cloud. Des comptes de stockage dans différentes régions peuvent entraîner une dégradation des performances. Vous pouvez utiliser un compte de stockage Standard ou Premium avec l’appliance cloud. Informations supplémentaires sur la création d’un [compte de stockage Standard](../storage/common/storage-account-create.md)
* Utilisez un compte de stockage différent pour la création de l’appliance cloud de celui utilisé pour vos données. L’utilisation du même compte de stockage peut entraîner une dégradation des performances.

Assurez-vous de disposer des informations suivantes avant de commencer :

* Votre compte avec les informations d’identification d’accès au portail Azure.
* Une copie de la clé de chiffrement des données de service issue de l’appareil physique inscrit au service StorSimple Device Manager.

## <a name="create-and-configure-the-cloud-appliance"></a>Créer et configurer l’appliance cloud

Avant d’effectuer ces procédures, assurez-vous que vous avez rempli les [conditions requises relatives à l’appliance cloud](#prerequisites-for-the-cloud-appliance).

Pour créer StorSimple Cloud Appliance, procédez comme suit.

### <a name="step-1-create-a-cloud-appliance"></a>Étape 1 : Créer une appliance cloud

Pour créer StorSimple Cloud Appliance, procédez comme suit.

[!INCLUDE [Create a cloud appliance](../../includes/storsimple-8000-create-cloud-appliance-u2.md)]

Si la création de l’appliance cloud échoue au cours de cette étape, vous ne disposerez peut-être pas d’une connectivité à Internet. Pour plus d’informations, consultez la section [Résolution des problèmes de connectivité Internet](#troubleshoot-internet-connectivity-errors) lors de la création d’une appliance cloud.

### <a name="step-2-configure-and-register-the-cloud-appliance"></a>Étape 2 : Configurer et inscrire l’appliance cloud

Avant de commencer cette procédure, assurez-vous que vous disposez d’une copie de la clé de chiffrement des données de service. Une clé de chiffrement des données de service est générée lorsque vous enregistrez votre premier appareil physique StorSimple auprès du service StorSimple Manager. Nous vous avons demandé à l’enregistrer dans un emplacement sécurisé. Si vous n’avez pas de copie de la clé de chiffrement des données de service, vous devez contacter le support technique de Microsoft pour obtenir de l’aide.

Procédez comme suit pour configurer et inscrire l’appliance cloud StorSimple.

[!INCLUDE [Configure and register a cloud appliance](../../includes/storsimple-8000-configure-register-cloud-appliance.md)]

### <a name="step-3-optional-modify-the-device-configuration-settings"></a>Étape 3 : Modifier les paramètres de configuration de l’appareil (facultatif)

La section suivante décrit les paramètres de configuration d’appareil que vous devez configurer pour l’appliance cloud StorSimple si vous envisagez d’utiliser CHAP, le Gestionnaire d’instantanés StorSimple ou de modifier le mot de passe d’administrateur de l’appareil.

#### <a name="configure-the-chap-initiator"></a>Configuration de l’initiateur CHAP

Ce paramètre contient les informations d’identification que votre appliance cloud (cible) attend des initiateurs (serveurs) qui tentent d’accéder aux volumes. Les initiateurs fournissent un nom d’utilisateur et un mot de passe CHAP pour s’identifier auprès de votre appareil au cours de cette authentification. Pour obtenir la procédure détaillée, consultez la rubrique [Configuration de CHAP pour votre appareil](storsimple-8000-configure-chap.md#unidirectional-or-one-way-authentication).

#### <a name="configure-the-chap-target"></a>Configuration de la cible CHAP

Ce paramètre contient les informations d’identification que votre appliance cloud utilise lorsqu’un initiateur CHAP demande une authentification mutuelle ou bidirectionnelle. Votre appliance cloud utilise un nom d’utilisateur et un mot de passe CHAP inversés pour s’identifier auprès de l’initiateur pendant le processus d’authentification.

> [!NOTE]
> Les paramètres CHAP cibles sont des paramètres globaux. Lorsqu’ils sont appliqués, tous les volumes connectés à l’appliance cloud de stockage utilisent l’authentification CHAP.

Pour obtenir la procédure détaillée, consultez la rubrique [Configuration de CHAP pour votre appareil](storsimple-8000-configure-chap.md#bidirectional-or-mutual-authentication).

#### <a name="configure-the-storsimple-snapshot-manager-password"></a>Configuration du mot de passe du gestionnaire d’instantanés StorSimple

Le Gestionnaire d’instantanés StorSimple réside sur l’ordinateur hôte Windows et permet aux administrateurs de gérer les sauvegardes de votre appareil StorSimple sous la forme d’instantanés cloud ou locaux.

> [!NOTE]
> Pour l’appliance cloud, l’ordinateur hôte Windows est une machine virtuelle Azure.

Lorsque vous configurez un appareil dans le Gestionnaire d’instantanés StorSimple, vous devez fournir l’adresse IP et le mot de passe de l’appareil StorSimple pour authentifier votre appareil de stockage. Pour obtenir la procédure détaillée, consultez la rubrique [Configuration du mot de passe de gestionnaire d'instantanés StorSimple](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password).

#### <a name="change-the-device-administrator-password"></a>Modification du mot de passe d’administrateur de l’appareil

Quand vous utilisez l’interface Windows PowerShell pour accéder à l’appliance cloud, vous devez entrer un mot de passe d’administrateur de l’appareil. Pour la sécurité de vos données, vous êtes obligé de changer ce mot de passe avant de pouvoir utiliser l’appliance cloud. Pour obtenir la procédure détaillée, consultez la rubrique [Configuration du mot de passe d’administrateur de l’appareil](../storsimple/storsimple-8000-change-passwords.md#change-the-device-administrator-password).

## <a name="connect-remotely-to-the-cloud-appliance"></a>Se connecter à distance à l’appliance cloud

L’accès à distance à votre appliance cloud via l’interface Windows PowerShell n’est pas activé par défaut. Vous devez commencer par activer la gestion à distance sur l’appliance cloud, puis sur le client utilisé pour y accéder.

La procédure en deux étapes suivante décrit comment se connecter à distance à votre appliance cloud.

### <a name="step-1-configure-remote-management"></a>Étape 1 : Configuration de la gestion à distance

Procédez comme suit pour configurer la gestion à distance de l’appliance cloud StorSimple.

[!INCLUDE [Configure remote management via HTTP for cloud appliance](../../includes/storsimple-8000-configure-remote-management-http-device.md)]

### <a name="step-2-remotely-access-the-cloud-appliance"></a>Étape 2 : Accéder à distance à l’appliance cloud

Après avoir activé la gestion à distance sur l’appliance cloud, utilisez la communication à distance Windows PowerShell pour vous connecter à l’appliance à partir d’une autre machine virtuelle au sein du même réseau virtuel. Par exemple, vous pouvez vous connecter à partir de la machine virtuelle hôte que vous avez configurée et utilisée pour vous connecter à iSCSI. Dans la plupart des déploiements, vous avez déjà ouvert un point de terminaison public pour accéder à votre machine virtuelle hôte que vous pouvez utiliser pour accéder à l’appliance cloud.

> [!WARNING]
> **Pour renforcer la sécurité, nous vous recommandons d’utiliser le protocole HTTPS lors de la connexion aux points de terminaison et de supprimer ces derniers à la fin de la session à distance PowerShell.**

Vous devez respecter les procédures de la rubrique [Connexion à distance à votre appareil StorSimple](storsimple-8000-remote-connect.md) afin de configurer la communication à distance pour votre appliance cloud.

## <a name="connect-directly-to-the-cloud-appliance"></a>Se connecter directement à l’appliance cloud

Vous pouvez également vous connecter directement à l’appliance cloud. Pour vous connecter directement à l’appliance cloud à partir d’un autre ordinateur en dehors du réseau virtuel ou de l’environnement Microsoft Azure, vous devez créer des points de terminaison supplémentaires.

Procédez comme suit pour créer un point de terminaison public sur l’appliance cloud :

[!INCLUDE [Create public endpoints on a cloud appliance](../../includes/storsimple-8000-create-public-endpoints-cloud-appliance.md)]

Nous vous recommandons de vous connecter à partir d’une autre machine virtuelle à l’intérieur du même réseau virtuel, car cela réduit le nombre de points de terminaison publics sur votre réseau virtuel. Dans ce cas, connectez-vous à la machine virtuelle via une session Bureau à distance, puis configurez cette machine virtuelle pour utilisation, comme vous le feriez avec tout autre client Windows sur un réseau local. Il est inutile d’ajouter le numéro de port public, car le port est déjà connu.

## <a name="get-private-ip-for-the-cloud-appliance"></a>Obtenir une adresse IP privée pour l’appliance cloud

Pour que l’appliance cloud puisse se connecter au serveur hôte dans le même réseau virtuel, vous avez besoin de l’adresse IP interne ou privée de l’appliance cloud. Suivez les étapes ci-dessous pour obtenir l’adresse IP privée de l’appliance cloud

1. Accédez à la machine virtuelle sous-jacente de votre appliance cloud. La machine virtuelle porte le même nom que votre appliance cloud. Accédez à **Toutes les ressources**, indiquez le nom de l’appliance cloud et de l’abonnement, puis sélectionnez le type machines virtuelles. Dans la liste des machines virtuelles présentées, sélectionnez et cliquez sur la machine virtuelle correspondant à l’appliance cloud.

     ![Sélectionner la machine virtuelle de votre appliance cloud](./media/storsimple-8000-cloud-appliance-u2/sca-vm.png)

2. Accédez à **Paramètres > Mise en réseau**. Dans le volet droit, vous obtenez l’adresse IP privée de l’appliance cloud. Notez cette adresse.

    ![Obtenir l’adresse IP privée de votre appliance cloud](./media/storsimple-8000-cloud-appliance-u2/sca-private-ip-vm-networking.png)

## <a name="work-with-the-storsimple-cloud-appliance"></a>Utiliser StorSimple Cloud Appliance

Maintenant que vous avez créé et configuré StorSimple Cloud Appliance, vous êtes prêt à commencer à l’utiliser. Vous pouvez travailler avec des conteneurs de volumes, des volumes et des stratégies de sauvegarde sur une appliance cloud comme vous le feriez sur un appareil physique StorSimple. La seule différence est que vous devez vous assurer que vous sélectionnez l’appliance cloud à partir de votre liste d’appareils. Reportez-vous à la rubrique [Utilisation du service StorSimple Manager pour gérer une appliance cloud](storsimple-8000-manager-service-administration.md) pour connaître les procédures pas à pas des différentes tâches de gestion pour l’appliance cloud.

Les sections suivantes présentent les différences que vous rencontrez lorsque vous utilisez l’appliance cloud.

### <a name="maintain-a-storsimple-cloud-appliance"></a>Assurer la maintenance de StorSimple Cloud Appliance

Comme il s’agit d’un appareil logiciel uniquement, la maintenance de l’appliance cloud est minime par rapport à la maintenance de l’appareil physique.

Vous ne pouvez pas mettre à jour une appliance cloud. Utiliser la dernière version du logiciel pour créer une nouvelle appliance cloud.


### <a name="storage-accounts-for-a-cloud-appliance"></a>Comptes de stockage pour une appliance cloud

Les comptes de stockage sont créés pour une utilisation par le service StorSimple Device Manager, par l’appliance cloud et l’appareil physique. Lorsque vous créez vos comptes de stockage, nous vous recommandons d’utiliser un identificateur de région dans le nom convivial. Cela permet de garantir la cohérence de la région dans tous les composants système. Pour une appliance cloud, il est important que tous les composants soient situés dans la même région afin d’éviter les problèmes de performances.

Pour une procédure pas à pas, consultez la rubrique [Ajout d’un compte de stockage](storsimple-8000-manage-storage-accounts.md#add-a-storage-account).

### <a name="deactivate-a-storsimple-cloud-appliance"></a>Désactiver StorSimple Cloud Appliance

La désactivation d’une appliance cloud supprime la machine virtuelle et les ressources créées lors de son approvisionnement. Une fois l’appliance cloud désactivée, elle ne peut pas être restaurée vers son état précédent. Avant de désactiver l’appliance cloud, arrêtez ou supprimez les clients et ordinateurs hôtes qui en dépendent.

La désactivation d’une appliance cloud entraîne les actions suivantes :

* L’appliance cloud est supprimée.
* Les disques de données et de système d’exploitation de l’appliance cloud sont supprimés.
* Le service hébergé et le réseau virtuel créés lors de l’approvisionnement sont conservés. Si vous ne les utilisez pas, vous devez les supprimer manuellement.
* Les instantanés cloud créés pour l’appliance cloud sont conservés.

Pour une procédure pas à pas, consultez [Désactiver et supprimer votre appareil StorSimple](storsimple-8000-deactivate-and-delete-device.md).

Dès que l’appliance cloud est indiquée comme désactivée dans le panneau du service StorSimple Device Manager, vous pouvez la supprimer de la liste des appareils dans le panneau **Appareils**.

### <a name="start-stop-and-restart-a-cloud-appliance"></a>Démarrer, arrêter et redémarrer une appliance cloud
Contrairement à l’appareil physique StorSimple, aucun bouton d’alimentation ou de mise hors tension n’est présent sur StorSimple Cloud Appliance. Toutefois, il se peut que vous deviez arrêter et redémarrer l’appliance cloud.

Le moyen le plus simple pour vous permettre de démarrer, d’arrêter et de redémarrer une appliance cloud est d’utiliser le panneau de service des machines virtuelles. Accédez au service de la machine virtuelle. Dans la liste des machines virtuelles, identifiez celle correspondant à votre appliance cloud (même nom), puis cliquez sur le nom de la machine virtuelle. Lorsque vous examinez le panneau de la machine virtuelle, l’état de l’appliance cloud est **En cours d’exécution** , car il est démarré par défaut après sa création. Vous pouvez démarrer, arrêter et redémarrer une machine virtuelle à tout moment.

[!INCLUDE [Stop and restart cloud appliance](../../includes/storsimple-8000-stop-restart-cloud-appliance.md)]

### <a name="reset-to-factory-defaults"></a>Rétablir les paramètres d’usine par défaut
Si vous décidez de repartir de zéro avec votre appliance cloud, désactivez-la, supprimez-la, puis créez-en une nouvelle.

## <a name="fail-over-to-the-cloud-appliance"></a>Basculer vers l’appliance cloud
La récupération d’urgence est un des scénarios clés pour lequel StorSimple Cloud Appliance a été conçu. Dans ce scénario, l’appareil physique StorSimple ou un centre de données entier peut ne pas être disponible. Heureusement, vous pouvez utiliser une appliance cloud pour restaurer les opérations dans un autre emplacement. Pendant la récupération d’urgence, la propriété des conteneurs de volumes de l’appareil source change et ces derniers sont transférés vers l’appliance cloud.

Les conditions requises pour la récupération d’urgence sont les suivantes :

* L’appliance cloud est créée et configurée.
* Tous les volumes du conteneur de volumes sont hors connexion.
* Le conteneur de volumes que vous souhaitez basculer est associé à un instantané cloud.

> [!NOTE]
> * Lorsque vous utilisez une appliance cloud en tant qu’appareil secondaire pour la récupération d’urgence, n’oubliez pas que le modèle 8010 dispose de 30 To de stockage Standard et que le modèle 8020 dispose de 64 To de stockage Premium. La capacité de l’appliance cloud 8020, qui est plus importante, peut être plus adaptée à un scénario de récupération d’urgence.

Pour une procédure pas à pas, consultez la rubrique [Basculement vers une appliance cloud](storsimple-8000-device-failover-cloud-appliance.md).

## <a name="delete-the-cloud-appliance"></a>Supprimer l’appliance cloud
Si vous avez précédemment configuré et utilisé StorSimple Cloud Appliance, mais que vous ne voulez plus cumuler de frais de calcul pour son utilisation, vous pouvez arrêter l’appliance cloud. L’arrêt de l’appliance cloud libère la machine virtuelle. Cette action empêche le cumul des frais sur votre abonnement. Les frais de stockage pour les disques de données et de système d’exploitation continuent néanmoins de s’appliquer.

Pour arrêter tous les frais, vous devez supprimer l’appliance cloud. Pour supprimer les sauvegardes créées par l’appliance cloud, vous pouvez désactiver ou supprimer l’appareil. Pour plus d’informations, consultez [Désactiver et supprimer un appareil StorSimple](storsimple-8000-deactivate-and-delete-device.md).

[!INCLUDE [Delete a cloud appliance](../../includes/storsimple-8000-delete-cloud-appliance.md)]

## <a name="troubleshoot-internet-connectivity-errors"></a>Résolution des problèmes de connectivité Internet
La création d’une appliance cloud échoue si vous ne disposez d’aucune connectivité à Internet. Pour résoudre les problèmes de connectivité Internet, procédez comme suit dans le portail Azure :

1. [Création d’une machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/quick-create-portal.md). Cette machine virtuelle doit utiliser le même compte de stockage, le même réseau virtuel et le même sous-réseau que ceux utilisés par votre appliance cloud. S’il existe déjà un hôte Windows Server dans Azure en utilisant le même compte de stockage, le même réseau virtuel et le même sous-réseau, vous pouvez également l’utiliser pour résoudre vos problèmes de connectivité Internet.
2. Connectez-vous à distance à la machine virtuelle créée à l’étape précédente.
3. Ouvrez une fenêtre de commande à l’intérieur de la machine virtuelle (Win + R, puis tapez `cmd`).
4. À l’invite, entrez la commande suivante :

    `nslookup windows.net`
5. Si `nslookup` échoue, l’échec de la connectivité Internet empêche l’appliance cloud de s’inscrire au service StorSimple Manager.
6. Apportez les modifications requises à votre réseau virtuel pour vous assurer que l’appliance cloud est en mesure d’accéder aux sites Azure tels que _windows.net_.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [utiliser le service StorSimple Device Manager pour gérer une appliance cloud](storsimple-8000-manager-service-administration.md).
* Découvrez comment [restaurer un volume StorSimple à partir d’un jeu de sauvegarde](storsimple-8000-restore-from-backup-set-u2.md).