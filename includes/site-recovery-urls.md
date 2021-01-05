---
title: Fichier Include
description: Fichier Include
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 09/12/2018
ms.author: raynew
ms.openlocfilehash: f4cd9cad3813378fcdc3f06e8ab1c28eced4f93c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999181"
---
Name | URL commerciale | URL du gouvernement | Description
--- | --- | --- | ---
Azure Active Directory | ``login.microsoftonline.com`` | ``login.microsoftonline.us`` | Azure Active Directory utilise ces adresses pour le contrôle d’accès et la gestion des identités. 
Backup | ``*.backup.windowsazure.com`` | ``*.backup.windowsazure.us`` | Élément utilisé pour la coordination et le transfert des données de réplication.
Réplication | ``*.hypervrecoverymanager.windowsazure.com`` | ``*.hypervrecoverymanager.windowsazure.us``  | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication.
Stockage | ``*.blob.core.windows.net`` | ``*.blob.core.usgovcloudapi.net``  | Élément utilisé pour l’accès au compte de stockage qui stocke les données répliquées.
Données de télémétrie (facultatif) | ``dc.services.visualstudio.com`` | ``dc.services.visualstudio.com`` | Adresses utilisées pour la télémétrie.
Synchronisation temporelle | ``time.windows.com`` | ``time.nist.gov`` | Adresses utilisées pour vérifier la synchronisation horaire entre l’horloge système et l’heure globale lors de tous les déploiements.


