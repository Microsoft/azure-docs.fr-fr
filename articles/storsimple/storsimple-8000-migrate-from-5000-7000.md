---
title: Migrer les données d’un appareil StorSimple 5000-7000 vers un appareil StorSimple 8000 | Microsoft Docs
description: Découvrez comment migrer des données sur un appareil de la série StorSimple 5000-7000 vers un appareil de la série 8000, et quelles sont les conditions préalables au processus de migration.
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/25/2020
ms.author: alkohli
ms.openlocfilehash: f6fffadd3c53f67af2e4c833a6a1d442c18efa0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91398184"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-8000-series-device"></a>Migrer les données d’un appareil StorSimple 5000-7000 vers un appareil StorSimple 8000

> [!IMPORTANT]
> - La fin du support de la série StorSimple 5000/7000 interviendra le 31 juillet 2019. Nous recommandons aux clients de la série StorSimple 5000/7000 de migrer vers l’une des alternatives décrites dans le document.
> - La migration est actuellement une opération assistée. Si vous envisagez de migrer les données de votre appareil StorSimple 5000-7000 vers un appareil StorSimple 8000, vous devez planifier la migration avec le Support Microsoft. Le Support Microsoft se chargera d’activer votre abonnement pour la migration. Pour plus d’informations, vérifiez comment [ouvrir un ticket de support](storsimple-8000-contact-microsoft-support.md).
> - Une fois soumise la demande de service, l’exécution du plan de migration et le démarrage de la migration peut prendre jusqu’à deux semaines.
> - Avant de contacter le Support Microsoft, assurez-vous que les [prérequis pour la migration](#migration-prerequisites) indiqués dans cet article sont satisfaits.

## <a name="overview"></a>Vue d’ensemble

Cet article présente la fonctionnalité de migration qui permet aux utilisateurs d’appareils StorSimple 5000-7000 de migrer leurs données vers des appareils physiques StorSimple 8000 ou des appliances cloud 8010/8020. Il renvoie également à une procédure pas à pas téléchargeable qui décrit les étapes nécessaires pour migrer des données d’un appareil d’ancienne génération 5000-7000 vers un appareil physique 8000 ou une appliance cloud.

Cet article concerne à la fois les appareils 8000 locaux et les appliances cloud StorSimple.


## <a name="migration-feature-versus-host-side-migration"></a>Fonctionnalité de migration et migration côté hôte

Vous pouvez déplacer vos données à l’aide de la fonctionnalité de migration ou en procédant à une migration côté hôte. Cette section décrit les caractéristiques de chaque méthode, y compris les avantages et les inconvénients. Utilisez ces informations pour déterminer la méthode à utiliser pour migrer vos données.

La fonctionnalité de migration simule un processus de récupération d’urgence d’un appareil 5000/7000 sur un appareil 8000. Cette fonctionnalité vous permet de migrer les données d’un appareil 5000/7000 vers un appareil 8000 sur Azure. Le processus de migration est lancé à l’aide de l’outil de migration StorSimple. L’outil démarre le téléchargement et la conversion de métadonnées de sauvegarde sur l’appareil 8000, puis utilise la dernière sauvegarde pour exposer les volumes sur l’appareil.

| Avantages                                                                                                                                     |Inconvénients                                                                                                                                                              |
|-------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Le processus de migration conserve l’historique des sauvegardes qui ont été effectuées sur les appareils 5000/7000.                                               | Quand les utilisateurs tentent d’accéder aux données, la fonctionnalité de migration télécharge les données à partir d’Azure, ce qui implique des frais de téléchargement de données.                                     |
| Aucune donnée n’est migrée côté hôte.                                                                                                     | Le processus nécessite un temps d’arrêt entre le début de la sauvegarde et la dernière sauvegarde surfacée sur l’appareil 8000 (il est possible de l’estimer à l’aide de l’outil de migration). |
| Ce processus conserve toutes les stratégies, les modèles de bande passante, le chiffrement et autres paramètres sur les appareils 8000.                      | L’accès utilisateur renvoie uniquement aux données accessibles par les utilisateurs, sans rafraîchir l’ensemble du jeu de données.                                                  |
| Ce processus nécessite plus de temps pour convertir toutes les anciennes sauvegardes dans Azure de manière asynchrone sans affecter la production. | La migration peut uniquement être effectuée à un niveau de configuration cloud.  Les volumes individuels dans une configuration cloud ne peuvent pas être migrés séparément.                       |

Une migration côté hôte permet de configurer indépendamment les appareils 8000 et de copier les données d’un appareil 5000/7000 sur un appareil 8000. Cela équivaut à migrer des données d’un appareil de stockage vers un autre. Différents outils, tels que Diskboss ou robocopy, sont utilisés pour copier les données.

| Avantages                                                                                                                      |Inconvénients                                                                                                                                                                                                      |
|---------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| La migration peut être réalisée en plusieurs phases, volume-par-volume.                                               | Les sauvegardes précédentes (effectuées sur des appareils 5000/7000) ne seront pas disponibles sur les appareils 8000.                                                                                                       |
| Permet de consolider les données dans un même compte de stockage dans Azure.                                                       | La première sauvegarde sur le cloud d’un appareil 8000 prend plus de temps, car toutes les données de l’appareil 8000 doivent être sauvegardées dans Azure.                                                                     |
| Après une migration réussie, toutes les données sont locales sur l’appliance. Il n’existe pas de latence durant l’accès aux données. | La consommation du stockage Azure augmente jusqu'à ce que les données soient supprimées de l’appareil 5000/7000.                                                                                                        |
|                                                                                                                           | Si l’appareil 5000/7000 héberge une grande quantité de données, durant la migration, ces données doivent être téléchargées à partir d’Azure, ce qui implique des frais et des latences liés au téléchargement de données à partir d’Azure. |

Cet article aborde uniquement la fonctionnalité de migration d’un appareil 5000/7000 vers un appareil 8000. Pour plus d’informations sur la migration côté hôte, consultez [Migration à partir d’autres appareils de stockage](https://download.microsoft.com/download/9/4/A/94AB8165-CCC4-430B-801B-9FD40C8DA340/Migrating%20Data%20to%20StorSimple%20Volumes_09-02-15.pdf).

## <a name="migration-prerequisites"></a>Prérequis pour la migration

Vous trouverez ci-dessous les prérequis pour la migration d’appareils d’ancienne génération StorSimple 5000 ou 7000 vers des appareils StorSimple 8000.

> [!IMPORTANT]
> Assurez-vous que les prérequis pour la migration sont satisfaits avant de soumettre une demande de service au Support Microsoft.

### <a name="for-the-50007000-series-device-source"></a>Pour les appareils 5000/7000 (sources)

Avant d’effectuer la migration, assurez-vous de satisfaire les exigences suivantes :

* Vous possédez un appareil source 5000 ou 7000 (actif ou inactif).

    > [!IMPORTANT]
    > Nous vous recommandons de disposer d’un accès série à cet appareil durant tout le processus de migration. En cas de problèmes avec votre appareil, l’accès série peut en faciliter la résolution.

* Votre appareil source 5000 ou 7000 exécute la version logicielle v2.1.1.518 ou une version ultérieure. Les versions antérieures ne sont pas prises en charge.
* Pour vérifier la version exécutée sur votre appareil 5000 ou 7000, regardez dans l’angle supérieur droit de votre interface utilisateur Web. Il devrait afficher la version logicielle de votre appareil. Pour la migration, les appareils 5000 ou 7000 doivent exécuter la version v2.1.1.518.

    ![Vérification de la version logicielle sur les appareils d’ancienne génération](media/storsimple-8000-migrate-from-5000-7000/check-version-legacy-device1.png)

    * Si votre appareil actif n’exécute pas la version v2.1.1.518 ou une version ultérieure, mettez à niveau votre système vers la version minimale requise. Vous devrez peut-être contacter le Support Microsoft afin de solliciter son assistance pour la mise à niveau.
    * Si vous exécutez la version v2.1.1.518, accédez à l’interface utilisateur Web pour vérifier s’il existe des notifications relatives à des échecs de restauration du Registre. Si la restauration du Registre a échoué, exécutez d’abord celle-ci. Vous devrez peut-être contacter le Support Microsoft afin de solliciter son assistance pour la restauration de votre registre.
    * Si vous disposez d’un appareil inactif qui n’exécutait pas la version v2.1.1.518, effectuez un basculement vers un appareil de remplacement exécutant la version v2.1.1.518. Pour plus d’informations, reportez-vous aux instructions détaillées de récupération d’urgence de votre appareil StorSimple 5000/7000.
    * Sauvegardez les données de votre appareil en prenant un instantané cloud.
    * Vérifiez s’il existe d’autres tâches de sauvegarde actives en cours d’exécution sur l’appareil source. Cela inclut notamment les tâches sur l’hôte de la console de protection des données StorSimple. Patientez jusqu’à ce que les tâches en cours soient entièrement terminées.


### <a name="for-the-8000-series-physical-device-target"></a>Pour les appareils physiques 8000 (cibles)

Avant d’effectuer la migration, assurez-vous de satisfaire les exigences suivantes :

* Votre appareil 8000 cible est inscrit et en cours d’exécution. Pour plus d’informations, vérifiez comment [déployer votre appareil StorSimple avec le service StorSimple Manager](storsimple-8000-deployment-walkthrough-u2.md).
* Votre appareil 8000 dispose de la dernière mise à jour 5 de StorSimple 8000 et exécute la version 6.3.9600.17845 ou une version ultérieure. Si votre appareil ne dispose pas des dernières mises à jour, vous devez les installer avant de pouvoir procéder à la migration. Pour en savoir plus, vérifiez comment [installer la dernière mise à jour sur votre appareil 8000](storsimple-8000-install-update-5.md).
* Votre abonnement Azure est activé pour la migration. Si ce n’est pas le cas, contactez le Support Microsoft pour l’activer.

### <a name="for-the-80108020-cloud-appliance-target"></a>Pour les appliances cloud 8010/8020 (cibles)

Avant d’effectuer la migration, assurez-vous de satisfaire les exigences suivantes :

* Votre appliance cloud cible est inscrite et en cours d’exécution. Pour plus d’informations, vérifiez comment [déployer et gérer une appliance cloud StorSimple](storsimple-8000-cloud-appliance-u2.md).
* Votre appliance cloud exécute la dernière mise à jour 5 de StorSimple 8000 et la version logicielle 6.3.9600.17845. Si elle n’exécute pas la mise à jour 5, créez une appliance cloud avec la mise à jour 5 avant de procéder à la migration. Pour plus d'informations, vérifiez comment [créer une appliance cloud 8010/8020](storsimple-8000-cloud-appliance-u2.md).

### <a name="for-the-computer-running-storsimple-migration-tool"></a>Pour l’ordinateur exécutant l’outil de migration StorSimple

L’outil de migration StorSimple est un outil basé sur l’interface utilisateur qui permet de migrer des données d’un appareil StorSimple 5000-7000 vers un appareil 8000. Pour installer l’outil de migration StorSimple, utilisez un ordinateur répondant aux exigences ci-dessous.

L’ordinateur possède une connexion Internet et :

* Exécute le système d’exploitation suivant :
    * Windows 10.
    * Windows Server 2012 R2 (ou version supérieure) pour installer l’outil de migration StorSimple.
* Dispose de .NET 4.5.2.
* Dispose d’au moins 5 Go d’espace libre pour l’installation et l’utilisation de l’outil.

> [!TIP]
> Si votre appareil StorSimple est connecté à un hôte Windows Server, vous pouvez installer l’outil de migration sur l’ordinateur hôte Windows Server.

#### <a name="to-install-storsimple-migration-tool"></a>Installer l’outil de migration StorSimple

Procédez comme suit pour installer l’outil de migration StorSimple sur votre ordinateur.

1. Copiez le dossier _StorSimple8000SeriesMigrationTool_ sur votre ordinateur Windows. Assurez-vous que le lecteur sur lequel est copié le logiciel possède suffisamment d’espace.

    Ouvrez le fichier config de l’outil _StorSimple8000SeriesMigrationTool.exe.config_ dans le dossier. Voici l’extrait de code du fichier.
    
    ```xml
        <add key="UserName" value="username@xyz.com" />
        <add key="SubscriptionName" value="YourSubscriptionName" />
        <add key="SubscriptionId" value="YourSubscriptionId" />
        <add key="TenantId" value="YourTenantId" />
        <add key="ResourceName" value="YourResourceName" />
        <add key="ResourceGroupName" value="YourResourceGroupName" />

    ```
2. Modifiez les valeurs correspondant aux clés comme suit :

    * `UserName` – Nom d’utilisateur pour vous connecter au portail Azure.
    * `SubscriptionName and SubscriptionId` – Nom et ID de votre abonnement Azure. Dans la page d’accueil de votre service StorSimple Device Manager, sous **Général**, cliquez sur **Propriétés**. Copiez le nom d’abonnement et l’ID d’abonnement associés à votre service.
    * `ResourceName` – Nom de votre service StorSimple Device Manager dans le portail Azure. Il est également affiché dans les propriétés du service.
    * `ResourceGroup`– Nom du groupe de ressources associé à votre service StorSimple Device Manager dans le portail Azure. Il est également affiché dans les propriétés du service.
    ![Vérification des propriétés du service pour l’appareil cible](media/storsimple-8000-migrate-from-5000-7000/check-service-properties1.png)
    * `TenantId` – ID de locataire Azure Active Directory dans le portail Azure. Connectez-vous à Microsoft Azure en tant qu’administrateur. Dans le portail Microsoft Azure, cliquez sur  **Azure Active Directory**. Sous **Gérer**, cliquez sur **Propriétés**. L’ID de locataire est indiqué dans la zone **ID de répertoire**.
    ![Vérification de l’ID de locataire pour Azure Active Directory](media/storsimple-8000-migrate-from-5000-7000/check-tenantid-aad.png)

3.  Enregistrez les modifications apportées au fichier config.
4.  Exécutez le fichier _StorSimple8000SeriesMigrationTool.exe_ pour lancer l’outil. Quand vous y êtes invité, fournissez les informations d’identification associées à votre abonnement dans le portail Azure. 
5.  L’interface utilisateur de l’outil de migration StorSimple s’affiche.
  

## <a name="next-steps"></a>Étapes suivantes
Téléchargez les instructions pas à pas relatives à la [migration de données d’un appareil StorSimple 5000-7000 vers un appareil 8000](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b).
