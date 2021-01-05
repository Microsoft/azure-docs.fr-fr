---
title: Fichier include
description: Fichier include
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 07/08/2020
ms.author: raynew
ms.custom: include file
ms.openlocfilehash: d09cb92055c69b6585fa3b55609144c0e680964a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997803"
---
**Exigences des serveur de configuration et de traitement**


## <a name="hardware-requirements"></a>Configuration matérielle requise

**Composant** | **Prérequis** 
--- | ---
Cœurs d’unité centrale | 8 
RAM | 16 Go
Nombre de disques | 3, y compris le disque du système d’exploitation, le disque de cache du serveur de processus et le lecteur de conservation pour la restauration automatique 
Espace disque disponible (cache du serveur de traitement) | 600 Go
Espace disque disponible (disque de rétention) | 600 Go
 | 

## <a name="software-requirements"></a>Configuration logicielle requise

**Composant** | **Prérequis** 
--- | ---
Système d’exploitation | Windows Server 2012 R2 <br> Windows Server 2016
Paramètres régionaux du système d’exploitation | Anglais (en-*)
Rôles Windows Server | N’activez pas ces rôles : <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V 
Stratégies de groupe | N’activez pas ces stratégies de groupe : <br> - Empêcher l’accès à l’invite de commandes <br> - Empêcher l’accès aux outils de modification du Registre <br> - Logique de confiance pour les pièces jointes <br> - Activer l’exécution des scripts <br> [En savoir plus](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))
IIS | - Aucun site web par défaut préexistant <br> - Aucune application/aucun site web préexistants ne doivent écouter le port 443 <br>- Activer [l’authentification anonyme](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br> - Activer le paramètre [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10)) 
FIPS (Federal Information Processing Standard) | Ne pas activer le mode FIPS
|

## <a name="network-requirements"></a>Configuration requise pour le réseau

**Composant** | **Prérequis** 
--- | --- 
Type d’adresse IP | statique 
Ports | 443 (Orchestration du canal de contrôle)<br>9443 (Transport de données) 
Type de carte réseau | VMXNET3 (si le serveur de configuration est une machine virtuelle VMware)
 |
**Accès Internet** (le serveur doit également accéder aux URL suivantes, directement ou par le biais d’un proxy) :|
\*.backup.windowsazure.com | Élément utilisé pour la coordination et le transfert des données répliquées
\*.blob.core.windows.net | Élément utilisé pour l’accès au compte de stockage qui stocke les données répliquées Vous pouvez fournir l’URL spécifique de votre compte de stockage de cache.
\*.hypervrecoverymanager.windowsazure.com | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication
https:\//login.microsoftonline.com | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication 
time.nist.gov | Éléments utilisés pour vérifier la synchronisation horaire entre l’horloge système et l’heure globale
time.windows.com | Éléments utilisés pour vérifier la synchronisation horaire entre l’horloge système et l’heure globale
| <ul> <li> https:\//management.azure.com </li><li> https:\//secure.aadcdn.microsoftonline-p.com </li><li> https:\//login.live.com </li><li> https:\//graph.windows.net </li><li> https:\//login.windows.net </li><li> *.services.visualstudio.com (facultatif) </li><li> https:\//www.live.com </li><li> https:\//www.microsoft.com </li></ul> | L’installation OVF nécessite l’accès à ces URL supplémentaires. Azure Active Directory utilise ces adresses pour le contrôle d’accès et la gestion des identités.
https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi  | Pour effectuer le téléchargement de MySQL. </br> Dans certaines régions, le téléchargement peut être redirigé vers l’URL CDN. Vérifiez que l’URL CDN est également approuvée, si nécessaire.
|

> [!NOTE]
> Si vous avez une [connexion de liaisons privées](../articles/site-recovery/hybrid-how-to-enable-replication-private-endpoints.md) au coffre Site Recovery, vous n’avez pas besoin d’un autre accès Internet pour le serveur de configuration. Exception : lors de la configuration de l’ordinateur CS à l’aide du modèle OVA, vous devez accéder aux URL suivantes vers et au-delà de l’accès de liaison privée - https://management.azure.com, https://www.live.com et https://www.microsoft.com. Si vous ne souhaitez pas autoriser l’accès à ces URL, configurez l’ordinateur CS avec le programme d’installation unifiée.

## <a name="required-software"></a>Logiciels requis

**Composant** | **Prérequis** 
--- | ---
VMware vSphere PowerCLI | [PowerCLI version 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1) doit être installé si le serveur de configuration est en cours d’exécution sur une machine virtuelle VMware.
MYSQL | MySQL doit être installé. Vous pouvez l’installer manuellement ou laisser Site Recovery le faire. (Reportez-vous à la [configuration des paramètres](../articles/site-recovery/vmware-azure-deploy-configuration-server.md#configure-settings) pour plus d’informations)
|

## <a name="sizing-and-capacity-requirements"></a>Besoins en capacité et dimensionnement

Le tableau suivant récapitule les besoins en capacité du serveur de configuration. Si vous répliquez plusieurs machines virtuelles VMware, consultez les [considérations sur la planification de la capacité](../articles/site-recovery/site-recovery-plan-capacity-vmware.md) et exécutez l’outil [Planificateur de déploiement Azure Site Recovery](../articles/site-recovery/site-recovery-deployment-planner.md).


**UC** | **Mémoire** | **Disque cache** | **Taux de modification des données** | **Machines répliquées**
--- | --- | --- | --- | ---
8 processeurs virtuels<br/><br/> 2 sockets * 4 cœurs \@ 2,5 GHz | 16 Go | 300 Go | 500 Go ou moins | < 100 machines
12 processeurs virtuels<br/><br/> 2 sockets * 6 cœurs \@ 2,5 GHz | 18 Go | 600 Go | 500 Go à 1 To | 100 à 150 machines
16 processeurs virtuels<br/><br/> 2 sockets * 8 cœurs \@ 2,5 GHz | 32 Go | 1 To | 1 à 2 To | 150 à 200 machines
|