---
title: Appliance de réplication Azure Migrate
description: Apprenez-en davantage sur l’appliance de réplication Azure Migrate pour une migration VMWare basée sur un agent.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: conceptual
ms.date: 01/30/2020
ms.openlocfilehash: ec277bcc3e361561f54e72c54526d65487c113b4
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754094"
---
# <a name="replication-appliance"></a>Appliance de réplication

Cet article décrit l’appliance de réplication utilisée par l’outil [Azure Migrate : migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool) lors de la migration de machines virtuelles VMware, de machines physiques et de machines virtuelles de cloud privé/public vers Azure, à l’aide d’une migration basée sur un agent. 


## <a name="overview"></a>Vue d’ensemble

L’appliance de réplication est déployée lorsque vous configurez la migration par agent des machines virtuelles VMware ou des serveurs physiques. Elle est déployée en tant qu’ordinateur local unique, comme machine virtuelle VMware ou comme serveur physique. Il exécute :

- **Appliance de réplication** : L’appliance de réplication coordonne les communications et gère la réplication des données pour les machines virtuelles VMware locales et les serveurs physiques répliqués sur Azure.
- **Serveur de traitement**: Le serveur de processus, qui est installé par défaut sur l’appliance de réplication, effectue les opérations suivantes :
    - **Passerelle de réplication** : Fait office de passerelle de réplication. Il reçoit les données de réplication des machines activées pour la réplication. Optimise les données de réplication grâce à la mise en cache, la compression et le chiffrement et les envoie vers Azure.
    - **Programme d’installation de l’agent** : Effectue une installation d’envoi (push) du service Mobilité. Ce service doit être installé et en cours d’exécution sur chaque ordinateur local que vous souhaitez répliquer pour la migration.

## <a name="appliance-deployment"></a>Étapes de déploiement d’appliance

**Utilisé pour** | **Détails**
--- |  ---
**Migration basée sur un agent de machines virtuelles VMware** | Vous téléchargez le modèle OVA à partir du hub Azure Migrate, puis vous importez sur vCenter Server pour créer la machine virtuelle de l’appliance.
**Migration basée sur un agent de machines physiques** | Si vous ne disposez pas d’une infrastructure VMware ou si vous ne pouvez pas créer une machine virtuelle VMware à l’aide d’un modèle OVA, téléchargez un programme d’installation de logiciel à partir du Hub Azure Migrate et exécutez-le pour configurer l’ordinateur de l’appliance.

> [!NOTE]
> Si vous effectuez un déploiement dans Azure Government, utilisez le fichier d'installation pour déployer l'appliance de réplication.

## <a name="appliance-requirements"></a>Configuration requise de l’appliance

Quand vous configurez l’appliance de réplication en utilisant le modèle OVA fourni dans le hub Azure Migrate, l’appliance exécute Windows Server 2016 et est conforme aux exigences de la prise en charge. Si vous configurez manuellement l’appliance de réplication sur un serveur physique, vérifiez qu’elle est conforme aux exigences.

**Composant** | **Prérequis**
--- | ---
 | **Appliance de machine virtuelle VMware**
PowerCLI | [PowerCLI version 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1) doit être installé si l’appliance de réplication s’exécute sur une machine virtuelle VMware.
Type de carte réseau | VMXNET3 (si l’appliance est une machine virtuelle VMware)
 | **Paramètres matériels**
Cœurs d’unité centrale | 8
RAM | 16 Go
Nombre de disques | Trois : Le disque du système d’exploitation, le disque de cache du serveur de processus et le lecteur de conservation.
Espace disque disponible (cache) | 600 Go
Espace disque disponible (disque de rétention) | 600 Go
**Paramètres logiciels** |
Système d’exploitation | Windows Server 2016 ou Windows Server 2012 R2
Licence | L’appliance est fournie avec une licence d’évaluation Windows Server 2016 qui est valide pendant 180 jours.<br/><br/> Si la période d’évaluation est proche de l’expiration, nous vous recommandons de télécharger et de déployer une nouvelle appliance, ou d’activer la licence du système d’exploitation de la machine virtuelle de l’appliance.
Paramètres régionaux du système d’exploitation | Anglais (en-us)
TLS | TLS 1.2 doit être activé.
.NET Framework | .NET Framework 4.6 ou ultérieur doit être installé sur la machine (avec un chiffrement fort activé).
MySQL | MySQL doit être installé sur l’appliance.<br/> MySQL doit être installé. Vous pouvez l’installer manuellement ou laisser Site Recovery le faire lors du déploiement de l’appliance.
Autres applications | N’exécutez pas d’autres applications sur l’appliance de réplication.
Rôles Windows Server | N’activez pas ces rôles : <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V
Stratégies de groupe | N’activez pas ces stratégies de groupe : <br> - Empêcher l’accès à l’invite de commandes <br> - Empêcher l’accès aux outils de modification du Registre <br> - Logique de confiance pour les pièces jointes <br> - Activer l’exécution des scripts <br> [En savoir plus](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))
IIS | - Aucun site web par défaut préexistant <br> - Aucune application/aucun site web préexistants ne doivent écouter le port 443 <br>- Activer [l’authentification anonyme](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br> - Activer le paramètre [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10))
**Paramètres réseau** |
Type d’adresse IP | statique
Ports | 443 (Orchestration du canal de contrôle)<br>9443 (Transport de données)
Type de carte réseau | VMXNET3

## <a name="mysql-installation"></a>Installation de MySQL 

MySQL doit être installé sur l’ordinateur de l’appliance de réplication. Il peut être installé à l’aide de l’une de ces méthodes.

**Méthode** | **Détails**
--- | ---
Télécharger et installer manuellement | Téléchargez l’application MySQL et placez-la dans le dossier C:\Temp\ASRSetup, puis installez-la manuellement.<br/> Quand vous configurez l’appliance, le MySQL apparaît comme déjà installé.
Sans téléchargement en ligne | Placez votre application d’installation de MySQL dans le dossier C:\Temp\ASRSetup. Quand vous installez l’appliance, et que vous cliquez pour télécharger et installer MySQL, le programme d’installation utilise le programme d’installation que vous avez ajouté.
Télécharger et installer dans Azure Migrate | Lorsque vous installez l’appliance et que vous êtes invité à installer MySQL, sélectionnez **Télécharger et installer**.

## <a name="url-access"></a>accès URL

L'appliance de réplication doit avoir accès à ces URL dans le cloud public Azure.

**URL** | **Détails**
--- | ---
\*.backup.windowsazure.com | Élément utilisé pour la coordination et le transfert des données répliquées
\*.store.core.windows.net | Élément utilisé pour la coordination et le transfert des données répliquées
\*.blob.core.windows.net | Utilisé pour l’accès au compte de stockage qui stocke les données répliquées
\*.hypervrecoverymanager.windowsazure.com | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication
https:\//management.azure.com | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication
*.services.visualstudio.com | Utilisé dans le cadre de la télémétrie (facultatif)
time.windows.com | Éléments utilisés pour vérifier la synchronisation horaire entre l’horloge système et l’heure globale.
https:\//login.microsoftonline.com <br/> https:\//secure.aadcdn.microsoftonline-p.com <br/> https:\//login.live.com <br/> https:\//graph.windows.net <br/> https:\//login.windows.net <br/> https:\//www.live.com <br/> https:\//www.microsoft.com  | Le programme d'installation de l'appliance doit avoir accès à ces URL. Azure Active Directory utilise ces adresses pour le contrôle d’accès et la gestion des identités
https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi | Pour effectuer le téléchargement de MySQL. Dans certaines régions, le téléchargement peut être redirigé vers l’URL CDN. Vérifiez que l'URL du CDN est également autorisée, si nécessaire.


## <a name="azure-government-url-access"></a>Accès aux URL Azure Government

L'appliance de réplication doit avoir accès à ces URL dans Azure Government.

**URL** | **Détails**
--- | ---
\*.backup.windowsazure.us | Élément utilisé pour la coordination et le transfert des données répliquées
\*.store.core.windows.net | Élément utilisé pour la coordination et le transfert des données répliquées
\*.blob.core.windows.net | Utilisé pour l’accès au compte de stockage qui stocke les données répliquées
\*.hypervrecoverymanager.windowsazure.us | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication
https:\//management.usgovcloudapi.net | Élément utilisé pour la coordination et l’administration des opérations de gestion de la réplication
*.services.visualstudio.com | Utilisé dans le cadre de la télémétrie (facultatif)
time.nist.gov | Éléments utilisés pour vérifier la synchronisation horaire entre l’horloge système et l’heure globale.
https:\//login.microsoftonline.com <br/> https:\//secure.aadcdn.microsoftonline-p.com <br/> https:\//login.live.com <br/> https:\//graph.windows.net <br/> https:\//login.windows.net <br/> https:\//www.live.com <br/> https:\//www.microsoft.com  | Le programme d'installation de l'appliance avec OVA doit avoir accès à ces URL. Azure Active Directory les utilise pour le contrôle d'accès et la gestion des identités.
https:\//dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi | Pour effectuer le téléchargement de MySQL. Dans certaines régions, le téléchargement peut être redirigé vers l’URL CDN. Vérifiez que l'URL du CDN est également autorisée, si nécessaire.

## <a name="port-access"></a>Accès au port

**Appareil** | **Connection**
--- | ---
Machines virtuelles | Le service Mobility en cours d’exécution sur des machines virtuelles communique avec l'appliance de réplication locale (serveur de configuration) via le port HTTPS 443 entrant, pour la gestion de la réplication.<br/><br/> Les machines virtuelles envoient des données de réplication au serveur de traitement (s’exécutant sur l’ordinateur du serveur de configuration) sur le port HTTPS 9443 entrant. Ce port peut être modifié.
Appliance de réplication | L’appliance de réplication orchestre la réplication avec Azure sur le port HTTPS 443 sortant.
Serveur de traitement | Le serveur de traitement reçoit les données de réplication, les optimise et les chiffre, puis les envoie au stockage Azure via le port 443 sortant.<br/> Le serveur de processus s’exécute par défaut sur l’appliance de réplication.


## <a name="replication-process"></a>Processus de réplication

1. Lorsque vous activez la réplication pour une machine virtuelle, la réplication initiale vers Stockage Azure commence, conformément à la stratégie de réplication spécifiée. 
2. Le trafic est répliqué sur des points de terminaison publics de stockage Azure via Internet. La réplication du trafic à partir d’un site local vers Azure via un réseau VPN de site à site n’est pas prise en charge.
3. Au terme de la réplication initiale, la réplication différentielle commence. Les modifications suivies pour une machine sont journalisées.
4. La communication s’effectue comme suit :
    - Les machines virtuelles communiquent avec l’appliance de réplication sur le port HTTPS 443 entrant, pour la gestion de la réplication.
    - L’appliance de réplication orchestre la réplication avec Azure sur le port HTTPS 443 sortant.
    - Les machines virtuelles envoient des données de réplication au serveur de processus (en cours d’exécution sur l’appliance de réplication) sur le port HTTPS 9443 entrant. Ce port peut être modifié.
    - Le serveur de traitement reçoit les données de réplication, les optimise et les chiffre, puis les envoie au stockage Azure via le port 443 sortant.
5. Les données de réplication se trouvent tout d’abord dans un compte de stockage de cache dans Azure. Ces journaux sont traités et les données sont stockées dans un disque managé Azure.

![Le diagramme illustre l’architecture du processus de réplication.](./media/migrate-replication-appliance/architecture.png)

## <a name="appliance-upgrades"></a>Mises à niveau d’appliance

L’appliance est mise à niveau manuellement à partir du hub Azure Migrate. Nous vous recommandons de toujours exécuter la dernière version.

1. Dans Azure Migrate > Servers > Azure Migrate : Évaluation du serveur, serveurs d’infrastructure, cliquez sur **Serveurs de configuration**.
2. Dans les **Serveurs de configuration**, un lien apparaît dans la **Version de l’agent** lorsqu’une nouvelle version de l’appliance de réplication est disponible. 
3. Téléchargez le programme d’installation sur la machine de l’appliance de réplication et installez la mise à niveau. Le programme d’installation détecte la version actuelle en cours d’exécution sur l’appliance.
 
## <a name="next-steps"></a>Étapes suivantes

- [Découvrez comment](tutorial-migrate-vmware-agent.md#set-up-the-replication-appliance) configurer l’appliance de réplication pour la migration de machines virtuelles VMware basées sur des agents.
- [Découvrez comment](tutorial-migrate-physical-virtual-machines.md#set-up-the-replication-appliance) configurer l’appliance de réplication pour les serveurs physiques.
