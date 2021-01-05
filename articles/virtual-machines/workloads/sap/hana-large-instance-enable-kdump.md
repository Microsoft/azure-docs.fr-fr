---
title: Script pour activer Kdump dans SAP HANA (grandes instances) | Microsoft Docs
description: Script pour activer Kdump dans SAP HANA (grandes instances), type HLI I, type HLI II
services: virtual-machines-linux
documentationcenter: ''
author: prtyag
manager: hrushib
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 03/30/2020
ms.author: prtyag
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: a799242ecaae7b7152d79b7d341a9cb5cc18d7fe
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654470"
---
# <a name="kdump-for-sap-hana-on-azure-large-instances-hli"></a>Kdump pour SAP HANA sur Azure (grandes instances) [HLI]

La configuration et l’activation de kdump est une étape nécessaire pour résoudre les incidents système qui n’ont pas de cause évidente.
Il arrive parfois qu’un système cesse de fonctionner de manière inattendue sans qu’un problème de matériel ou d’infrastructure ne puisse l’expliquer.
Dans ce cas, il peut s’agir d’un problème de système d’exploitation ou d’application, et kdump permettra à SUSE de déterminer la raison de l’arrêt du système.

## <a name="enable-kdump-service"></a>Activer le service Kdump

Ce document décrit en détail comment activer le service Kdump sur une grande instance Azure HANA (**type I et type II**)

## <a name="supported-skus"></a>Références prises en charge

|  Type de grande instance Hana   |  Fournisseur du système d’exploitation   |  Version du package du système d’exploitation   |  SKU |
|-----------------------------|--------------|-----------------------|-------------|
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S224m      |
|   Type I                    |  SuSE        |   SLES 12 SP4         |  S224m      |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S72        |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S72m       |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S72m       |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S96        |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S96        |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S192       |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S192       |
|   Type I                    |  SuSE        |   SLES 12 SP4         |  S192       |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S192m      |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S192m      |
|   Type I                    |  SuSE        |   SLES 12 SP4         |  S192m      |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S144       |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S144       |
|   Type I                    |  SuSE        |   SLES 12 SP2         |  S144m      |
|   Type I                    |  SuSE        |   SLES 12 SP3         |  S144m      |
|   Type II                   |  SuSE        |   SLES 12 SP2         |  S384       |
|   Type II                   |  SuSE        |   SLES 12 SP3         |  S384       |
|   Type II                   |  SuSE        |   SLES 12 SP4         |  S384       |
|   Type II                   |  SuSE        |   SLES 12 SP2         |  S384xm     |
|   Type II                   |  SuSE        |   SLES 12 SP3         |  S384xm     |
|   Type II                   |  SuSE        |   SLES 12 SP4         |  S384xm     |
|   Type II                   |  SuSE        |   SLES 12 SP2         |  S576m      |
|   Type II                   |  SuSE        |   SLES 12 SP3         |  S576m      |
|   Type II                   |  SuSE        |   SLES 12 SP4         |  S576m      |

## <a name="prerequisites"></a>Prérequis

- Comme le service Kdump utilise le répertoire `/var/crash` pour écrire des vidages, vérifiez que la partition qui correspond à ce répertoire a suffisamment d’espace pour accueillir les vidages.

## <a name="setup-details"></a>Détails de la configuration

- Le script pour activer Kdump est disponible [ici](https://github.com/Azure/sap-hana/blob/master/tools/enable-kdump.sh)
> [!NOTE]
> Ce script est basé sur la configuration de notre laboratoire, et le client doit contacter le fournisseur du système d’exploitation pour tout réglage supplémentaire.
> Un numéro d’unité logique distinct sera configuré pour les serveurs nouveaux et existants aux fins de l’enregistrement des vidages sur incident, et le script se chargera de configurer le système de fichiers à partir du numéro d’unité logique.
> Microsoft n’est pas responsable de l’analyse du vidage sur incident. Le client doit ouvrir un ticket auprès du fournisseur du système d’exploitation pour l’analyser.

- Exécutez ce script sur une grande instance HANA à l’aide de la commande ci-dessous

    > [!NOTE]
    > Privilège sudo requis pour exécuter cette commande.

    ```bash
    sudo bash enable-kdump.sh
    ```

- Si la commande génère le message « Kdump est correctement activé », veillez à redémarrer le système pour appliquer les modifications.

- Si la sortie de la commande est un message de type « Échec de l’exécution d’une opération donnée, sortie », le service Kdump n’est pas activé. Reportez-vous à la section [Problème de support](#support-issue).

## <a name="test-kdump"></a>Tester Kdump

> [!NOTE]
>  L’opération ci-dessous déclenche un plantage du noyau et un redémarrage du système.

- Déclenchez un plantage du noyau

    ```bash
    echo c > /proc/sysrq-trigger
    ```

- Une fois le système redémarré, recherchez les journaux de plantages du noyau dans le répertoire `/var/crash`.

- Si le répertoire `/var/crash` possède un répertoire avec la date du jour, le service Kdump est correctement activé.

## <a name="support-issue"></a>Problème de support

Si le script échoue avec une erreur ou que Kdump n’est pas activé, envoyez une demande de service à l’équipe du support technique Microsoft avec les informations suivantes.

* ID d’abonnement HLI

* Nom du serveur

* Fournisseur du système d’exploitation

* Version du SE

* Version du noyau

## <a name="related-documents"></a>Documents associés
- Pour en savoir plus sur la [configuration de kdump](https://www.suse.com/support/kb/doc/?id=3374462)
