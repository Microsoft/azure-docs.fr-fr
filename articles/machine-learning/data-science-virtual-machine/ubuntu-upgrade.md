---
title: Comment mettre à niveau votre Data Science Virtual Machine vers Ubuntu 18.04
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment mettre à niveau CentOS et Ubuntu 16.04 vers la dernière version d’Ubuntu 18.04 Data Science Virtual Machine.
keywords: formation approfondie, IA, outils de science des données, machine virtuelle de science des données, processus de science des données en équipe
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: samkemp
ms.author: samkemp
ms.topic: conceptual
ms.date: 10/07/2020
ms.openlocfilehash: 5b98384d4d735f4c124c6af40d6edbff896900ce
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93320988"
---
# <a name="upgrade-your-data-science-virtual-machine-to-ubuntu-1804"></a>Mettre à niveau votre Data Science Virtual Machine vers Ubuntu 18.04

Si vous avez une Data Science Virtual Machine exécutant une version antérieure telle que Ubuntu 16.04 ou CentOS, vous devez migrer votre DSVM vers Ubuntu 18.04. La migration vous permet d’obtenir les derniers correctifs du système d’exploitation, des pilotes, des logiciels préinstallés et des versions de bibliothèque. Ce document vous indique comment effectuer une migration à partir de versions antérieures d’Ubuntu ou de CentOS. 

## <a name="prerequisites"></a>Prérequis

- Connaissance de SSH et de la ligne de commande Linux

## <a name="overview"></a>Vue d’ensemble

Il existe deux façons d’effectuer cette migration :

- Migration sur place, également appelée migration « même serveur ». Cette migration met à niveau la machine virtuelle existante sans créer de nouvelle machine virtuelle. La migration sur place est le moyen le plus simple de migrer d’Ubuntu 16.04 vers Ubuntu 18.04.
- Migration côte à côte, également appelée migration « entre serveurs ». Cette migration transfère les données de la machine virtuelle existante vers une nouvelle machine virtuelle. La migration côte à côte est la meilleure façon de migrer de CentOS vers Ubuntu 18.04. Vous pouvez préférer une migration côte à côte pour effectuer une mise à niveau entre des versions Ubuntu si vous pensez que votre ancienne installation est inutilement encombrée.

## <a name="snapshot-your-vm-in-case-you-need-to-roll-back"></a>Faites une capture instantanée de votre machine virtuelle au cas où vous devriez effectuer une restauration

Dans le portail Azure, utilisez la barre de recherche pour rechercher la fonctionnalité d’ **instantanés**. 

:::image type="content" source="media/ubuntu_upgrade/azure-portal-search-bar.png" alt-text="Capture d’écran montrant le Portail Azure et la barre de recherche, avec **instantanés** mis en surbrillance":::

1. Sélectionnez **Ajouter** , ce qui vous permet d’afficher la page **Créer l’instantané**. Sélectionnez l’abonnement et le groupe de ressources de votre machine virtuelle. Pour **Région** , sélectionnez la région où se trouve le stockage cible. Sélectionnez le disque de stockage DSVM et les options de sauvegarde supplémentaires. Le **HDD standard** est un type de stockage approprié pour ce scénario de sauvegarde.

:::image type="content" source="media/ubuntu_upgrade/create-snapshot-options.png" alt-text="Capture d’écran montrant les options de création d’un instantané":::

2. Une fois que tous les détails sont remplis et que les validations sont réussies, sélectionnez **Vérifier + créer** pour valider et créer l’instantané. Une fois l’instantané terminé, un message s’affiche pour vous indiquer que le déploiement est terminé.

## <a name="in-place-migration"></a>Migration sur place

Si vous migrez depuis une ancienne version d’Ubuntu, vous pouvez choisir d’effectuer une migration sur place. Cette migration ne crée pas de nouvelle machine virtuelle et comporte moins d’étapes qu’une migration côte à côte.  Si vous souhaitez effectuer une migration côte à côte parce que vous souhaitez davantage de contrôle ou parce que vous effectuez une migration à partir d’une autre distribution, par exemple CentOS, passez à la section sur la [migration côte à côte](#side-by-side-migration). 

1. Dans le portail Azure, démarrez votre DSVM et connectez-vous à l’aide de SSH. Pour ce faire, sélectionnez **Connecter** et **SSH** , et suivez les instructions de connexion. 

1. Une fois connecté à une session terminal sur votre DSVM, exécutez la commande de mise à niveau suivante :

    ```bash
    sudo do-release-upgrade
    ```

Le processus de mise à niveau va prendre un certain temps. Lorsqu’il est terminé, le programme demande l’autorisation de redémarrer la machine virtuelle. Répondez **Oui**. Vous serez déconnecté de la session SSH au redémarrage du système.

### <a name="if-necessary-regenerate-ssh-keys"></a>Si nécessaire, régénérer les clés SSH

> [!IMPORTANT] 
> Après la mise à niveau et le redémarrage, vous devrez peut-être régénérer vos clés SSH.

Une fois que votre machine virtuelle a été mise à niveau et redémarrée, tentez d’y accéder à nouveau par le biais de SSH. L’adresse IP a peut-être été modifiée lors du redémarrage. Vérifiez-la avant d’essayer de vous connecter.

Si vous recevez l’erreur **L’IDENTIFICATION DE L’HÔTE DISTANT A CHANGÉ** , vous devez régénérer vos informations d’identification SSH.

:::image type="content" source="media/ubuntu_upgrade/remote-host-warning.png" alt-text="Capture d’écran PowerShell montrant l’avertissement de changement d’identification de l’hôte distant":::

Sur votre ordinateur local, exécutez la commande :

```bash
ssh-keygen -R "your server hostname or ip"
```

Vous devez être maintenant en mesure de vous connecter avec SSH. Si vous rencontrez toujours des problèmes, dans la page **Se connecter** suivez le lien pour **résoudre les problèmes de connectivité SSH**.

## <a name="side-by-side-migration"></a>Migration côte à côte

Si vous effectuez une migration à partir de CentOS ou que vous souhaitez une installation propre du système d’exploitation, vous pouvez effectuer une migration côte à côte. Ce type de migration comporte plus d’étapes, mais elle vous permet de contrôler exactement les fichiers qui sont transférés.

Les migrations d’autres systèmes basés sur le même ensemble de packages sources en amont sont généralement relativement simples, par exemple [FAQ/CentOS3](https://wiki.centos.org/FAQ/CentOS3).

Vous pouvez choisir de mettre à niveau les parties du système d’exploitation du système de fichiers et de conserver les répertoires de l’utilisateur, par exemple `/home`. Si vous laissez les anciens répertoires de démarrage de l’utilisateur en place, vous pouvez rencontrer des problèmes avec les menus GNOME/KDE et d’autres éléments du bureau. Il peut être plus facile de créer des comptes d’utilisateur et de monter les anciens répertoires ailleurs dans le système de fichiers pour référence, puis de copier ou lier les documents de l’utilisateur après la migration.

### <a name="migration-at-a-glance"></a>La migration en un clin d’œil

1.  Créer un instantané de votre machine virtuelle existante comme décrit précédemment

1.  Créer un disque à partir de cette capture instantanée

1.  Créer une instance Ubuntu de Data Science Virtual Machine

1.  Recréer un ou plusieurs comptes d’utilisateur sur la nouvelle machine virtuelle

1.  Monter le disque de la machine virtuelle instantanée en tant que disque de données sur votre nouveau Data Science Virtual Machine

1.  Copier manuellement les données souhaitées

### <a name="create-a-disk-from-your-vm-snapshot"></a>Créer un disque à partir de l’instantané de votre machine virtuelle

Si vous n’avez pas encore créé d’instantané de machine virtuelle comme décrit précédemment, faites-le. 

1. Dans le portail Azure, recherchez **Disques** et sélectionnez **Ajouter** , ce qui ouvre la page **Disque**.

:::image type="content" source="media/ubuntu_upgrade/portal-disks-search.png" alt-text="Capture d’écran du portail Azure montrant la page de recherche des disques et le bouton Ajouter":::

2. Définissez les valeurs **Abonnement** , **Groupe de ressources** et **Région** sur les valeurs de votre instantané de machine virtuelle. Choisissez un **nom** pour le disque à créer.

3. Sélectionnez **Type de source** comme **Capture instantanée** et sélectionnez l’instantané de machine virtuelle en tant qu’ **instantané source**. Vérifiez et créez le disque. 

:::image type="content" source="media/ubuntu_upgrade/disk-create-options.png" alt-text="Capture d’écran de la boîte de dialogue de création de disque montrant les options":::

### <a name="create-a-new-ubuntu-data-science-virtual-machine"></a>Créer une instance Ubuntu de Data Science Virtual Machine

Créez une machine virtuelle Data Science Virtual Machine Ubuntu à l’aide du [portail Azure](https://portal.azure.com) ou d’un [modèle ARM](./dsvm-tutorial-resource-manager.md). 

### <a name="recreate-user-accounts-on-your-new-data-science-virtual-machine"></a>Recréer un ou plusieurs comptes d’utilisateur sur votre nouvelle Data Science Virtual Machine

Étant donné que vous allez simplement copier des données à partir de votre ancien ordinateur, vous devez recréer les comptes utilisateur et les environnements logiciels que vous souhaitez utiliser sur le nouvel ordinateur.

Linux est suffisamment flexible pour vous permettre de personnaliser les répertoires et les chemins d’accès sur votre nouvelle installation pour qu’ils correspondent à ceux de votre ancienne machine. En général, cependant, il est plus facile d’utiliser la disposition moderne par défaut d’Ubuntu et de modifier votre environnement utilisateur et vos scripts pour les adapter.

Pour plus d’informations, consultez [Démarrage rapide : Configurer la machine virtuelle Science des données pour Linux (Ubuntu)](./dsvm-ubuntu-intro.md).

### <a name="mount-the-disk-of-the-snapshotted-vm-as-a-data-disk-on-your-new-data-science-virtual-machine"></a>Monter le disque de la machine virtuelle instantanée en tant que disque de données sur votre nouveau Data Science Virtual Machine

1. Dans le portail Azure, assurez-vous que votre Data Science Virtual Machine est en cours d’exécution.

2. Dans le portail Azure, accédez à la page de votre machine DSVM (Data Science Virtual Machine). Choisissez le panneau **des disques** sur le rail de gauche. Choisissez **Attacher un disque existant**.

3. Dans la liste déroulante **Nom du disque** , sélectionnez le disque que vous avez créé à partir de l’instantané de l’ancien ordinateur virtuel.

:::image type="content" source="media/ubuntu_upgrade/attach-data-disk.png" alt-text="Capture d’écran de la page d’options DSVM montrant les options d’attachement de disque":::

4. Sélectionnez **Enregistrer** pour mettre à jour votre machine virtuelle.

> [!Important]
> Votre machine virtuelle doit être en cours d’exécution au moment où vous attachez le disque de données. Si la machine virtuelle n’est pas en cours d’exécution, les disques peuvent ne pas être ajoutés dans le bon ordre, conduisant à un système confus et pouvant éventuellement refuser de démarrer. Si vous ajoutez le disque de données lorsque la machine virtuelle est désactivée, choisissez le **X** en regard du disque de données, démarrez la machine virtuelle, puis rattachez-le.

### <a name="manually-copy-the-wanted-data"></a>Copier manuellement les données souhaitées 

1. Connectez-vous à votre machine virtuelle en cours d’exécution à l’aide de SSH.

2. Vérifiez que vous avez attaché le disque créé à partir de l’instantané de l’ancienne machine virtuelle en exécutant la commande suivante :

    ```bash
    lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i 'sd'
    ```
    
    Les résultats doivent ressembler à cette image. Dans l’image, le disque `sda1` est monté à la racine et `sdb2` est le disque de travail `/mnt`. Le disque de données créé à partir de la capture instantanée de votre ancienne machine virtuelle est identifié comme `sdc1`, mais n’est pas encore disponible, comme le démontre l’absence d’un emplacement de montage. Vos résultats peuvent utiliser des identificateurs différents, mais vous devriez voir une situation similaire.
    
    :::image type="content" source="media/ubuntu_upgrade/lsblk-results.png" alt-text="Capture d’écran de la sortie lsblk montrant le lecteur de données non monté":::
    
3. Pour accéder au lecteur de données, créez un emplacement et montez-le. Remplacez `/dev/sdc1` par la valeur appropriée retournée par `lsblk` :

    ```bash
    sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive
    ```
    
4. À présent, `/datadrive` contient les répertoires et les fichiers de votre ancienne machine Data Science Virtual Machine. Déplacez ou copiez les répertoires ou fichiers que vous souhaitez du lecteur de données vers la nouvelle machine virtuelle comme vous le souhaitez.

Pour plus d’informations, consultez [Utiliser le portail pour attacher un disque de données à une machine virtuelle Linux](../../virtual-machines/linux/attach-disk-portal.md#connect-to-the-linux-vm-to-mount-the-new-disk).

## <a name="connect-and-confirm-version-upgrade"></a>Se connecter et confirmer la mise à niveau de la version

Que vous ayez effectué une migration sur place ou côte à côte, vérifiez que vous avez effectué la mise à niveau. Depuis une session de terminal, exécutez : 

```bash
cat /etc/os-release
```

Vous devriez voir que vous exécutez Ubuntu 18.04.

:::image type="content" source="media/ubuntu_upgrade/ssh-os-release.png" alt-text="Capture d’écran du terminal Ubuntu montrant les données de version du système d’exploitation":::

Le changement de version est également indiqué dans le portail Azure.

:::image type="content" source="media/ubuntu_upgrade/portal-showing-os-version.png" alt-text="Capture d’écran du portail montrant les propriétés DSVM, y compris la version du système d’exploitation":::

## <a name="next-steps"></a>Étapes suivantes

- [Science des données avec une image Data Science Machine Ubuntu sur Azure](./linux-dsvm-walkthrough.md)
- [Quels sont les outils qui sont inclus dans Azure Data Science Virtual Machine ?](./tools-included.md)