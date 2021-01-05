---
title: Fichier Include
description: Fichier Include
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: include
ms.date: 09/03/2018
ms.author: raynew
ms.custom: include file
ms.openlocfilehash: 3a8a7be6f437687a4de31ce8e0ac62588f64e2eb
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96016899"
---
**Configuration requise des serveurs de configuration/traitement pour la réplication de serveurs physiques**

**Composant** | **Prérequis** 
--- | ---
**PARAMÈTRES MATÉRIELS** | 
Cœurs d’unité centrale | 8 
RAM | 16 Go
Nombre de disques | 3, y compris le disque du système d’exploitation, le disque de cache du serveur de processus et le lecteur de conservation pour la restauration automatique 
Espace disque disponible (cache du serveur de traitement) | 600 Go
Espace disque disponible (disque de rétention) | 600 Go
 | 
**PARAMÈTRES LOGICIELS** | 
Système d’exploitation | Windows Server 2012 R2 <br> Windows Server 2016
Paramètres régionaux du système d’exploitation | Anglais (en-us)
Rôles Windows Server | N’activez pas ces rôles : <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V 
Stratégies de groupe | N’activez pas ces stratégies de groupe : <br> - Empêcher l’accès à l’invite de commandes <br> - Empêcher l’accès aux outils de modification du Registre <br> - Logique de confiance pour les pièces jointes <br> - Activer l’exécution des scripts <br> [En savoir plus](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))
IIS | - Aucun site web par défaut préexistant <br> - Aucune application/aucun site web préexistants ne doivent écouter le port 443 <br>- Activer [l’authentification anonyme](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br> - Activer le paramètre [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10))
Type d’adresse IP | statique 
| 
**PARAMÈTRES D’ACCÈS** | 
MYSQL | MySQL doit être installé sur le serveur de configuration. Vous pouvez l’installer manuellement ou laisser Site Recovery le faire au moment du déploiement. Pour permettre l’installation de Site Recovery, vérifiez que la machine a accès à http://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi.
URLs | Le serveur de configuration doit également accéder à ces URL (directement ou par le biais d’un proxy) :<br/><br/> Azure AD : `login.microsoftonline.com` ; `login.microsoftonline.us` ; `*.accesscontrol.windows.net`<br/><br/> Transfert des données de réplication : `*.backup.windowsazure.com` ; `*.backup.windowsazure.us`<br/><br/> Gestion de la réplication : `*.hypervrecoverymanager.windowsazure.com` ; `*.hypervrecoverymanager.windowsazure.us` ; `https://management.azure.com` ; `*.services.visualstudio.com`<br/><br/> Accès au stockage : `*.blob.core.windows.net` ; `*.blob.core.usgovcloudapi.net`<br/><br/> Synchronisation temporelle : `time.nist.gov` ; `time.windows.com`<br/><br/> Données de télémétrie (facultatif) : `dc.services.visualstudio.com`
Pare-feu | Les règles de pare-feu basées sur une adresse IP doivent autoriser les communications vers les URL Azure. Pour simplifier et limiter les plages d’adresses IP, nous vous recommandons d’utiliser le filtrage des URL.<br/><br/>**Pour les adresses IP commerciales :**<br/><br/>- Autorisez les [plages d’adresses IP de centres de données Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653) et le port HTTPS (443).<br/><br/> - Autorisez les plages d’adresses IP de la région USA Ouest (utilisées pour la gestion du contrôle d’accès et des identités).<br/><br/> - Autorisez les plages d’adresses IP de la région Azure de votre abonnement afin de garantir la prise en charge des URL requises pour Azure Active Directory, la sauvegarde, la réplication et le stockage.<br/><br/> **Pour les adresses IP du secteur public :**<br/><br/> - Autorisez les plages d’adresses IP de centres de données Azure Government, ainsi que le port HTTPS (443).<br/><br/> - Autorisez les plages d’adresses IP de toutes les régions Gov US (Virginie, Texas, Arizona et Iowa) afin de garantir la prise en charge des URL requises pour Azure Active Directory, la sauvegarde, la réplication et le stockage.
Ports | Autorisez le port 443 (orchestration du canal de contrôle)<br/><br/> Autorisez le port 9443 (transport de données) 


**Exigences de dimensionnement des serveurs de configuration/de traitement**

**UC** | **Mémoire** | **Disque cache** | **Taux de modification des données** | **Machines répliquées**
--- | --- | --- | --- | ---
8 processeurs virtuels<br/><br/> 2 sockets * 4 cœurs \@ 2,5 GHz | 16 Go | 300 Go | 500 Go ou moins | < 100 machines
12 processeurs virtuels<br/><br/> 2 sockets * 6 cœurs \@ 2,5 GHz | 18 Go | 600 Go | 500 Go à 1 To | 100 à 150 machines
16 processeurs virtuels<br/><br/> 2 sockets * 8 cœurs \@ 2,5 GHz | 32 Go | 1 To | 1 à 2 To | 150 à 200 machines