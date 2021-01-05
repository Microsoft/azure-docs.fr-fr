---
title: 'Référence : Problèmes connus et résolutions'
titleSuffix: Azure Data Science Virtual  Machine
description: Obtenez la liste des problèmes connus, des solutions de contournement et des résolutions pour Azure Data Science Virtual Machine
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: gvashishtha
ms.author: gopalv
ms.topic: reference
ms.date: 10/10/2019
ms.openlocfilehash: 864b5e519875029149e93df248aa5953f62ec51e
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322951"
---
# <a name="known-issues-and-troubleshooting-the-azure-data-science-virtual-machine"></a>Problèmes connus et dépannage d’Azure Data Science Virtual Machine

Cet article vous aide à rechercher et corriger les erreurs ou les défaillances que vous pouvez rencontrer lors de l’utilisation d’Azure Data Science Virtual Machine.

## <a name="python-package-installation-issues"></a>Problèmes d’installation des packages Python

### <a name="installing-packages-with-pip-breaks-dependencies-on-linux"></a>L’installation de packages avec PIP rompt les dépendances sur Linux

Utilisez `sudo pip install` à la place de `pip install` lors de l’installation des packages.

## <a name="disk-encryption-issues"></a>Problèmes de chiffrement de disque

### <a name="disk-encryption-fails-on-the-ubuntu-dsvm"></a>Le chiffrement de disque échoue sur la machine virtuelle DSVM Ubuntu

Azure Disk Encryption (ADE) n’est actuellement pas pris en charge sur la machine virtuelle DSVM Ubuntu. Pour contourner ce problème, envisagez de configurer le [chiffrement côté serveur des disques managés Azure](../../virtual-machines/disk-encryption.md).

## <a name="tool-appears-disabled"></a>Un outil apparaît désactivé

### <a name="hyper-v-does-not-work-on-the-windows-dsvm"></a>Hyper-V ne fonctionne pas sur la machine virtuelle DSVM Windows

Pour le moment, Hyper-V ne fonctionne pas sur Windows. Pour garantir les performances au démarrage, nous avons désactivé certains services. Pour activer Hyper-V :

1. Ouvrez la barre de recherche sur votre DSVM Windows.
1. Saisissez « Services ».
1. Définissez tous les services Hyper-V sur « Manual ».
1. Définissez « Hyper-V Virtual Machine Management » sur « Automatic ».

Votre écran final doit ressembler à ceci :

   ![Activer Hyper-V](./media/workaround/hyperv-enable-dsvm.png)