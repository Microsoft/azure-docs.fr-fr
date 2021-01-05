---
title: Appliance Azure Migrate
description: Fournit un résumé de la prise en charge de l’appliance Azure Migrate.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: f3357fcc070ca4ca2212cf0424e4baf8208a3ca1
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754128"
---
# <a name="azure-migrate-appliance"></a>Appliance Azure Migrate

Cet article récapitule les prérequis et les exigences de la prise en charge de l’appliance Azure Migrate. 

## <a name="deployment-scenarios"></a>Scénarios de déploiement

L’appliance Azure Migrate est utilisée dans les scénarios suivants.

**Scénario** | **Outil** | **Utilisé pour** 
--- | --- | ---
**Évaluation des machines virtuelles VMware** | Azure Migrate : Évaluation de serveurs | Détection des machines virtuelles VMware<br/><br/> Détection des applications et des dépendances de machine<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances pour les évaluations.
**Migration sans agent des machines virtuelles VMware** | Azure Migrate : Migration de serveurs | Détection des machines virtuelles VMware <br/><br/> Répliquer des machines virtuelles VMware avec une migration sans agent.
**Évaluation des machines virtuelles Hyper-V** | Azure Migrate : Évaluation de serveurs | Détection des machines virtuelles Hyper-V<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances pour les évaluations.
**Évaluation des machines physiques** |  Azure Migrate : Évaluation de serveurs |  Découvrez les serveurs physiques (ou les machines virtuelles que vous traitez comme des serveurs physiques).<br/><br/> Collecter les métadonnées des machines et les métadonnées de performances pour les évaluations.

## <a name="deployment-methods"></a>Méthodes de déploiement

L'appliance peut être déployée à l’aide de deux méthodes :

- L'appliance peut être déployée à l’aide d’un modèle pour les machines virtuelles VMware et les machines virtuelles Hyper-V (modèle OVA pour VMware ou VHD pour Hyper-V).
- Si vous ne souhaitez pas utiliser de modèle, vous pouvez déployer l’appliance pour VMware ou Hyper-V à l’aide d’un script PowerShell.
- Dans Azure Government, vous devez déployer l'appliance à l'aide d'un script.
- Pour les serveurs physiques, vous déployez toujours l’appliance à l’aide d’un script.
- Les liens de téléchargement sont disponibles dans les tableaux ci-dessous.


## <a name="appliance---vmware"></a>Appliance - VMware 

Le tableau suivant récapitule les exigences de l’appliance Azure Migrate pour VMware.

**Prérequis** | **VMware** 
--- | ---
**autorisations** | Pour accéder à l’application web de l’appliance localement ou à distance, vous devez être un administrateur de domaine ou un administrateur local sur l’ordinateur de l’appliance.
**Composants de l’appliance** | L’appliance a les composants suivants :<br/><br/> - **Application de gestion** : Il s’agit d’une application web pour l’entrée des utilisateurs pendant le déploiement de l’appareil. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/> - **Agent de découverte** : L’agent collecte les données de configuration de la machine. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Agent collecteur** : l’agent collecte les données de performances. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Service de mise à jour automatique** : Met à jour les composants de l’appliance (s’exécute toutes les 24 heures).<br/>- **Agent DRA** : Orchestre la réplication des machines virtuelles et coordonne la communication entre les machines répliquées et Azure. Utilisé seulement lors de la réplication de machines virtuelles VMware sur Azure avec la migration sans agent.<br/>- **Passerelle** : envoie des données répliquées vers Azure. Utilisé seulement lors de la réplication de machines virtuelles VMware sur Azure avec la migration sans agent.
**Déploiement pris en charge** | Déployez en tant que machine virtuelle VMware à en utilisant un modèle OVA.<br/><br/> Déployez en tant que machine virtuelle VMware ou que machine physique avec un script d’installation PowerShell.
**Prise en charge de projet** |  Une appliance peut être associée à un seul projet. <br/> Un nombre quelconque d’appliances peut être associé à un même projet.<br/> 
**Limites de la découverte** | Une appliance peut découvrir jusqu’à 10 000 machines virtuelles VMware sur un serveur vCenter.<br/> Une appliance ne peut se connecter qu'à un seul vCenter Server.
**Modèle OVA** | Téléchargez-le sur le portail ou [ici](https://go.microsoft.com/fwlink/?linkid=2140333).<br/><br/> La taille du téléchargement est de 11,9 Go.<br/><br/> Le modèle d’appliance téléchargé est fourni avec une licence d’évaluation Windows Server 2016 qui est valide pendant 180 jours. Si la période d’évaluation est proche de l’expiration, nous vous recommandons de télécharger et de déployer une nouvelle appliance, ou d’activer la licence du système d’exploitation de la machine virtuelle de l’appliance.
**Script PowerShell** | Consultez cet [article](./deploy-appliance-script.md#set-up-the-appliance-for-vmware).<br/><br/> 
**Logiciel/matériel** |  L’appliance doit s’exécuter sur une machine avec Windows Server 2016, 32 Go de RAM, 8 processeurs virtuels, environ 80 Go de stockage sur disque et un commutateur virtuel externe.<br/> L’appliance nécessite un accès à Internet, directement ou via un proxy.<br/><br/> Si vous exécutez l’appliance sur une machine virtuelle VMware, vous devez disposer de suffisamment de ressources sur le serveur vCenter pour allouer une machine virtuelle qui répond à la configuration requise.<br/><br/> Si vous exécutez l’appliance sur une machine physique, vérifiez qu’elle exécute Windows Server 2016 et qu’elle est conforme à la configuration matérielle requise.
**Conditions requises VMware** | Si vous déployez l’appliance en tant que machine virtuelle VMware, elle doit être déployée sur un hôte ESXi exécutant la version 5.5 ou ultérieure.<br/><br/> vCenter Server exécutant les versions 5,5, 6,0, 6,5 ou 6,7.
**VDDK (migration sans agent)** | Si vous déployez l’appliance en tant que machine virtuelle VMware et que vous effectuez une migration sans agent, le VDDK VMware vSphere doit être installé sur la machine virtuelle de l’appliance.
**Valeur de hachage - OVA** | [Vérifiez](tutorial-discover-vmware.md#verify-security) les valeurs de hachage du modèle OVA.
**Valeur de hachage - Script PowerShell** | [Vérifiez](deploy-appliance-script.md#verify-file-security) les valeurs de hachage du script PowerShell.




## <a name="appliance---hyper-v"></a>Appliance - Hyper-V

**Prérequis** | **Hyper-V** 
--- | ---
**autorisations** | Pour accéder à l’application web de l’appliance localement ou à distance, vous devez être un administrateur de domaine ou un administrateur local sur l’ordinateur de l’appliance.
**Composants de l’appliance** | L’appliance a les composants suivants :<br/><br/>- **Application de gestion** : Il s’agit d’une application web pour l’entrée des utilisateurs pendant le déploiement de l’appareil. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/> - **Agent de découverte** : L’agent collecte les données de configuration de la machine. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Agent collecteur** : l’agent collecte les données de performances. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Service de mise à jour automatique** : Met à jour les composants de l’appliance (s’exécute toutes les 24 heures).
**Déploiement pris en charge** | Déployez en tant que machine virtuelle Hyper-V en utilisant un modèle de disque dur virtuel.<br/><br/> Déployez en tant que machine virtuelle Hyper-V ou que machine physique avec un script d’installation PowerShell.
**Prise en charge de projet** |  Une appliance peut être associée à un seul projet. <br/> Un nombre quelconque d’appliances peut être associé à un même projet.<br/> 
**Limites de la découverte** | Une appliance peut détecter jusqu’à 5 000 machines virtuelles Hyper-V.<br/> Une appliance peut connecter à jusqu’à 300 hôtes Hyper-V.
**Modèle de disque dur virtuel** | Dossier compressé incluant le disque dur virtuel. Téléchargez-le sur le portail ou [ici](https://go.microsoft.com/fwlink/?linkid=2140422).<br/><br/> La taille du téléchargement est de 8,91 Go.<br/><br/> Le modèle d’appliance téléchargé est fourni avec une licence d’évaluation Windows Server 2016 qui est valide pendant 180 jours. Si la période d’évaluation est proche de l’expiration, nous vous recommandons de télécharger et de déployer une nouvelle appliance, ou d’activer la licence du système d’exploitation de la machine virtuelle de l’appliance.
**Script PowerShell** | Consultez cet [article](./deploy-appliance-script.md#set-up-the-appliance-for-hyper-v).<br/><br/> 
**Logiciel/matériel** _   |  L’appliance doit s’exécuter sur une machine avec Windows Server 2016, 16 Go de RAM, 8 processeurs virtuels, environ 80 Go de stockage sur disque et un commutateur virtuel externe.<br/> L’appliance nécessite une adresse IP statique ou dynamique et un accès à Internet, directement ou via un proxy.<br/><br/> Si vous exécutez l'appliance en tant que machine virtuelle Hyper-V, vous devez disposer de suffisamment de ressources sur l'hôte Hyper-V pour allouer la configuration matérielle requise.<br/><br/> Si vous exécutez l’appliance sur une machine physique, vérifiez qu’elle exécute Windows Server 2016 et qu’elle est conforme à la configuration matérielle requise. 
_ *Configuration requise pour Hyper-V** | Si vous déployez l’appliance avec le modèle de disque dur virtuel, la machine virtuelle de l’appliance fournie par Azure Migrate est une machine virtuelle Hyper-V version 5.0.<br/><br/> L’hôte Hyper-V doit exécuter Windows Server 2012 R2 ou une version ultérieure. 
**Valeur de hachage - Disque dur virtuel** | [Vérifiez](tutorial-discover-hyper-v.md#verify-security) les valeurs de hachage du modèle VHD.
**Valeur de hachage - Script PowerShell** | [Vérifiez](deploy-appliance-script.md#verify-file-security) les valeurs de hachage du script PowerShell.


## <a name="appliance---physical"></a>Appliance - Physique

**Prérequis** | **Physique** 
--- | ---
**autorisations** | Pour accéder à l’application web de l’appliance localement ou à distance, vous devez être un administrateur de domaine ou un administrateur local sur l’ordinateur de l’appliance.
**Composants de l’appliance** | L’appliance a les composants suivants : <br/><br/> - **Application de gestion** : Il s’agit d’une application web pour l’entrée des utilisateurs pendant le déploiement de l’appareil. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/> - **Agent de découverte** : L’agent collecte les données de configuration de la machine. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Agent collecteur** : l’agent collecte les données de performances. Utilisé lors de l’évaluation des machines pour la migration vers Azure.<br/>- **Service de mise à jour automatique** : Met à jour les composants de l’appliance (s’exécute toutes les 24 heures).
**Déploiement pris en charge** | Déployez en tant que machine dédiée ou en tant que machine virtuelle avec un script d’installation PowerShell. Le script est disponible en téléchargement à partir du portail.
**Prise en charge de projet** |  Une appliance peut être associée à un seul projet. <br/> Un nombre quelconque d’appliances peut être associé à un même projet.<br/> 
**Limites de la découverte** | Une appliance peut découvrir jusqu’à 1 000 serveurs physiques.
**Script PowerShell** | Téléchargez le script (AzureMigrateInstaller.ps1) dans un dossier compressé à partir du portail ou [ici](https://go.microsoft.com/fwlink/?linkid=2140334). [Plus d’informations](tutorial-discover-physical.md)<br/><br/> La taille du téléchargement est de 85,8 Mo.
**Logiciel/matériel** |  L'appliance doit s'exécuter sur une machine dotée de Windows Server 2016, de 16 Go de RAM, de 8 processeurs virtuels et d'environ 80 Go de stockage sur disque.<br/> L’appliance nécessite une adresse IP statique ou dynamique et un accès à Internet, directement ou via un proxy.<br/><br/> Si vous exécutez l’appliance sur une machine physique, vérifiez qu’elle exécute Windows Server 2016 et qu’elle est conforme à la configuration matérielle requise.<br/>_(Actuellement, le déploiement de l’appliance est pris en charge uniquement sur Windows Server 2016.)_
**Valeur de hachage** | [Vérifiez](tutorial-discover-physical.md#verify-security) les valeurs de hachage du script PowerShell.

## <a name="url-access"></a>accès URL

L’appliance Azure Migrate a besoin d’une connectivité Internet.

- Lorsque vous déployez l’appliance, Azure Migrate effectue une vérification de la connectivité aux URL requises.
- Vous devez autoriser l’accès à toutes les URL de la liste. Si vous effectuez uniquement une évaluation, vous pouvez ignorer les URL marquées comme requises pour la migration sans agent VMware uniquement.
-  Si vous utilisez un proxy basé sur les URL pour vous connecter à Internet, vérifiez que le proxy résout tous les enregistrements CNAME reçus lors de la recherche des URL.

### <a name="public-cloud-urls"></a>URL de cloud public

**URL** | **Détails**  
--- | --- |
*. portal.azure.com  | Accédez au portail Azure.
\* .windows.net <br/> *.msftauth.net <br/> *.msauth.net <br/> *.microsoft.com <br/> *.live.com <br/> *.office.com | Connectez-vous à votre abonnement Azure.
*.microsoftonline.com <br/> *.microsoftonline-p.com | Créez des applications Azure Active Directory (AD) pour que l’appliance communique avec Azure Migrate.
management.azure.com | Créez des applications Azure AD pour que l’appliance communique avec le service Azure Migrate.
*.services.visualstudio.com | Chargez les journaux d’applications utilisés pour la supervision interne.
*.vault.azure.net | Gérez les secrets dans Azure Key Vault. Remarque : Vérifiez que les machines à répliquer y ont accès.
aka.ms/* | Autorisez l’accès à des liens aka. Utilisé pour les mises à jour de l’appliance Azure Migrate.
download.microsoft.com/download | Autoriser les téléchargements à partir du téléchargement Microsoft.
*.servicebus.windows.net | Communication entre l’appliance et le service Azure Migrate.
*.discoverysrv.windowsazure.com <br/> *.migration.windowsazure.com | Connectez-vous aux URL du service Azure Migrate.
*.hypervrecoverymanager.windowsazure.com | **Utilisé pour la migration sans agent VMware**<br/><br/> Connectez-vous aux URL du service Azure Migrate.
*.blob.core.windows.net |  **Utilisé pour la migration sans agent VMware**<br/><br/>Chargez les données vers le stockage pour la migration.

### <a name="government-cloud-urls"></a>URL cloud Government

**URL** | **Détails**  
--- | --- |
*.portal.azure.us  | Accédez au portail Azure.
graph.windows.net | Connectez-vous à votre abonnement Azure.
login.microsoftonline.us  | Créez des applications Azure Active Directory (AD) pour que l’appliance communique avec Azure Migrate.
management.usgovcloudapi.net | Créez des applications Azure AD pour que l’appliance communique avec le service Azure Migrate.
*.services.visualstudio.com | Chargez les journaux d’applications utilisés pour la supervision interne.
*.vault.usgovcloudapi.net | Gérez les secrets dans Azure Key Vault.
aka.ms/* | Autorisez l’accès à des liens aka. Utilisé pour les mises à jour de l’appliance Azure Migrate.
download.microsoft.com/download | Autoriser les téléchargements à partir du téléchargement Microsoft.
*.servicebus.usgovcloudapi.net  | Communication entre l’appliance et le service Azure Migrate.
*.discoverysrv.windowsazure.us <br/> *.migration.windowsazure.us | Connectez-vous aux URL du service Azure Migrate.
*.hypervrecoverymanager.windowsazure.us | **Utilisé pour la migration sans agent VMware**<br/><br/> Connectez-vous aux URL du service Azure Migrate.
*.blob.core.usgovcloudapi.net  |  **Utilisé pour la migration sans agent VMware**<br/><br/>Chargez les données vers le stockage pour la migration.
*.applicationinsights.us | Chargez les journaux d’applications utilisés pour la supervision interne.





## <a name="collected-data---vmware"></a>Données collectées - VMware

L’appliance collecte les métadonnées, les données de performance et les données d’analyse des dépendances (si l’[analyse des dépendances](concepts-dependency-visualization.md) sans agent est utilisée).

### <a name="metadata"></a>Métadonnées

Les métadonnées découvertes par l’appliance Azure Migrate vous aident à déterminer si les machines et les applications sont prêtes pour la migration vers Azure : dimensionnement approprié des machines et des applications, coûts des plans et analyse des dépendances des applications. Microsoft n’utilise pas ces données dans le cadre des audits de conformité des licences.

Voici la liste complète des métadonnées des machines virtuelles VMware que l’appliance collecte et envoie à Azure.

**DONNÉES** | **COMPTEUR**
--- | --- 
**Détails de la machine** | 
ID de l'ordinateur virtuel | vm.Config.InstanceUuid 
nom de la machine virtuelle | vm.Config.Name
ID du vCenter Server | VMwareClient.Instance.Uuid
Description de la machine virtuelle | vm.Summary.Config.Annotation
Nom de produit de la licence | vm.Client.ServiceContent.About.LicenseProductName
Type de système d'exploitation | vm.SummaryConfig.GuestFullName
Type de démarrage | vm.Config.Firmware
Nombre de mémoires à tores magnétiques | vm.Config.Hardware.NumCPU
Mémoire (Mo) | vm.Config.Hardware.MemoryMB
Nombre de disques | vm.Config.Hardware.Device.ToList().FindAll(x => is VirtualDisk).count
Liste des tailles de disque | vm.Config.Hardware.Device.ToList().FindAll(x => is VirtualDisk)
Liste des adaptateurs réseau | vm.Config.Hardware.Device.ToList().FindAll(x => is VirtualEthernet).count
Utilisation du processeur | cpu.usage.average
Utilisation de la mémoire |mem.usage.average
**Détails par disque** | 
Valeur de la clé du disque | disk.Key
Numéro du disque | disk.UnitNumber
Valeur de clé de contrôleur de disque | disk.ControllerKey.Value
Gigaoctets approvisionnés | virtualDisk.DeviceInfo.Summary
Nom du disque | Valeur générée à l’aide de disk.UnitNumber, disk.Key, disk.ControllerKey.VAlue
Opérations de lecture par seconde | virtualDisk.numberReadAveraged.average
Opérations d’écriture par seconde | virtualDisk.numberWriteAveraged.average
Débit de lecture (Mo par seconde) | virtualDisk.read.average
Débit d’écriture (Mo par seconde) | virtualDisk.write.average
**Détails par carte d’interface réseau** | 
Nom de la carte réseau | nic.Key
Adresse MAC | ((VirtualEthernetCard)nic).MacAddress
Adresses IPv4 | vm.Guest.Net
Adresses IPv6 | vm.Guest.Net
Débit de lecture (Mo par seconde) | net.received.average
Débit d’écriture (Mo par seconde) | net.transmitted.average
**Détails de chemin d’accès d’inventaire** | 
Nom | container.GetType().Name
Type d’objet enfant | container.ChildType
Détails de référence | container.MoRef
Détails du parent | Container.Parent
Détails du dossier par machine virtuelle | ((Folder)container).ChildEntity.Type
Détails du centre de données par machine virtuelle | ((Datacenter)container).VmFolder
Détails du centre de données par dossier hôte | ((Datacenter)container).HostFolder
Détails du cluster par hôte | ((ClusterComputeResource)container).Host
Détails de l’hôte par machine virtuelle | ((HostSystem)container).VM

### <a name="performance-data"></a>Données de performances


Voici les données de performances des machines virtuelles VMware que l’appliance collecte et envoie à Azure.

**Données** | **Compteur** | **Évaluation de l'impact**
--- | --- | ---
Utilisation du processeur | cpu.usage.average | Taille de machine virtuelle recommandée/coût
Utilisation de la mémoire | mem.usage.average | Taille de machine virtuelle recommandée/coût
Débit de lecture du disque (Mo par seconde) | virtualDisk.read.average | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Débits d’écriture du disque (Mo par seconde) | virtualDisk.write.average | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Opérations de lecture du disque par seconde | virtualDisk.numberReadAveraged.average | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Opérations d’écriture du disque par seconde | virtualDisk.numberWriteAveraged.average  | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Débit de lecture de la carte d’interface réseau (Mo par seconde) | net.received.average | Calcul de la taille de machine virtuelle
Débit des écritures de la carte réseau (Mo par seconde) | net.transmitted.average  |Calcul de la taille de machine virtuelle


### <a name="installed-apps-metadata"></a>Métadonnées des applications installées

La découverte des applications collecte les données des applications et du système d’exploitation installées.

#### <a name="windows-vm-apps-data"></a>Données des applications de machine virtuelle Windows

Voici les données des applications installées que l’appliance collecte auprès de chaque machine virtuelle activée pour la détection des applications. Ces données sont envoyées à Azure.

**Données** | **Emplacement du registre** | **Clé**
--- | --- | ---
Nom de l’application  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayName
Version  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | DisplayVersion 
Fournisseur  | HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*  <br/> HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*  | Serveur de publication

#### <a name="windows-vm-features-data"></a>Données des fonctionnalités de machine virtuelle Windows

Voici les données des fonctionnalités que l’appliance collecte auprès de chaque machine virtuelle activée pour la détection des applications. Ces données sont envoyées à Azure.

**Données**  | **Applet de commande PowerShell** | **Propriété**
--- | --- | ---
Nom  | Get-WindowsFeature  | Nom
Type de fonctionnalité | Get-WindowsFeature  | FeatureType
Parent  | Get-WindowsFeature  | Parent

#### <a name="windows-vm-sql-server-metadata"></a>Métadonnées SQL Server de machine virtuelle Windows

Voici les métadonnées de serveur SQL Server que l’appliance collecte auprès des machines virtuelles exécutant un serveur Microsoft SQL Server activé pour la détection des applications. Ces données sont envoyées à Azure.

**Données**  | **Emplacement du registre**  | **Clé**
--- | --- | ---
Nom  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL  | installedInstance
Édition  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Édition 
Service Pack  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | SP
Version  | HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\\\<InstanceName>\Setup  | Version 

#### <a name="windows-vm-operating-system-data"></a>Données de système d’exploitation de machine virtuelle Windows

Voici les données de système d’exploitation que l’appliance collecte auprès de chaque machine virtuelle activée pour la détection des applications. Ces données sont envoyées à Azure.

Données  | Classe WMI  | Propriété de classe WMI
--- | --- | ---
Nom  | Win32_operatingsystem  | Caption
Version  | Win32_operatingsystem  | Version
Architecture  | Win32_operatingsystem  | OSArchitecture

#### <a name="linux-vm-apps-data"></a>Données des applications de machine virtuelle Linux

Voici les données des applications installées que l’appliance collecte auprès de chaque machine virtuelle activée pour la détection des applications. En fonction du système d’exploitation de la machine virtuelle, une ou plusieurs des commandes sont exécutées. Ces données sont envoyées à Azure.

Données  | Commande
--- | --- 
Nom | rpm, dpkg-query, snap
Version | rpm, dpkg-query, snap
Fournisseur | rpm, dpkg-query, snap

#### <a name="linux-vm-operating-system-data"></a>Données de système d’exploitation de machine virtuelle Linux

Voici les données de système d’exploitation que l’appliance collecte auprès de chaque machine virtuelle activée pour la détection des applications. Ces données sont envoyées à Azure.

**Données**  | **Commande** 
--- | --- | ---
Nom <br/> version | Collectées dans un ou plusieurs des fichiers suivants :<br/> <br/>/etc/os-release  <br> /usr/lib/os-release  <br> /etc/enterprise-release  <br> /etc/redhat-release  <br> /etc/oracle-release  <br> /etc/SuSE-release  <br> /etc/lsb-release  <br> /etc/debian_version 
Architecture | uname


### <a name="app-dependencies-metadata"></a>Métadonnées des dépendances d’application

L’analyse des dépendances sans agent collecte les données de connexion et de processus.

#### <a name="windows-vm-app-dependencies-data"></a>Données des dépendances des applications de machine virtuelle Windows

Voici les données de connexion que l’appliance collecte auprès de chaque machine virtuelle activée pour l’analyse des dépendances sans agent. Ces données sont envoyées à Azure.

**Données** | **Commande utilisée** 
--- | --- 
Port local | netstat
Adresse IP locale | netstat
Port distant | netstat
Adresse IP distante | netstat
État de la connexion TCP | netstat
ID du processus | netstat
Nombre de connexions actives | netstat


Voici les données de processus que l’appliance collecte auprès de chaque machine virtuelle activée pour l’analyse des dépendances sans agent. Ces données sont envoyées à Azure.

**Données** | **Classe WMI** | **Propriété de classe WMI**
--- | --- | ---
Nom du processus | Win32_Process | ExecutablePath
Arguments de processus | Win32_Process | CommandLine
Nom de l'application | Win32_Process | Paramètre VersionInfo.ProductName de la propriété ExecutablePath

#### <a name="linux-vm-app-dependencies-data"></a>Données des dépendances des applications de machine virtuelle Linux

Voici les données de connexion et de processus que l’appliance collecte auprès de chaque machine virtuelle Linux activée pour l’analyse des dépendances sans agent. Ces données sont envoyées à Azure.

**Données** | **Commande utilisée** 
--- | ---
Port local | netstat 
Adresse IP locale | netstat 
Port distant | netstat 
Adresse IP distante | netstat 
État de la connexion TCP | netstat 
Nombre de connexions actives | netstat
ID du processus  | netstat 
Nom du processus | ps
Arguments de processus | ps
Nom de l'application | dpkg ou rpm



## <a name="collected-data---hyper-v"></a>Données collectées - Hyper-V

L’appliance collecte les métadonnées, les données de performance et les données d’analyse des dépendances (si l’[analyse des dépendances](concepts-dependency-visualization.md) sans agent est utilisée).

### <a name="metadata"></a>Métadonnées
Les métadonnées découvertes par l’appliance Azure Migrate vous aident à déterminer si les machines et les applications sont prêtes pour la migration vers Azure : dimensionnement approprié des machines et des applications, coûts des plans et analyse des dépendances des applications. Microsoft n’utilise pas ces données dans le cadre des audits de conformité des licences.

Voici la liste complète des métadonnées des machines virtuelles Hyper-V que l’appliance collecte et envoie à Azure.

**DONNÉES** | **CLASSE WMI** | **PROPRIÉTÉ DE LA CLASSE WMI**
--- | --- | ---
**Détails de la machine** | 
Numéro de série de BIOS _ Msvm_BIOSElement | BIOSSerialNumber
Type de machine virtuelle (Gen1 ou 2) | Msvm_VirtualSystemSettingData | VirtualSystemSubType
Nom d'affichage de la machine virtuelle | Msvm_VirtualSystemSettingData | ElementName
Version de la machine virtuelle | Msvm_ProcessorSettingData | VirtualQuantity
Mémoire (octets) | Msvm_MemorySettingData | VirtualQuantity
Mémoire maximale pouvant être consommée par machine virtuelle | Msvm_MemorySettingData | Limite
Mémoire dynamique activée | Msvm_MemorySettingData | DynamicMemoryEnabled
Nom/version/nom de domaine complet du système d'exploitation | Msvm_KvpExchangeComponent | Données de nom GuestIntrinsicExchangeItems
États d’alimentation de la machine virtuelle | Msvm_ComputerSystem | EnabledState
**Détails par disque** | 
Identificateur de disque | Msvm_VirtualHardDiskSettingData | VirtualDiskId
Type de disque dur virtuel | Msvm_VirtualHardDiskSettingData | Type
Taille de disque dur virtuel | Msvm_VirtualHardDiskSettingData | MaxInternalSize
Parent de disque dur virtuel | Msvm_VirtualHardDiskSettingData | ParentPath
**Détails par carte d’interface réseau** | 
Adresses IP (cartes réseau synthétiques) | Msvm_GuestNetworkAdapterConfiguration | IPAddresses
DHCP activé (cartes réseau synthétiques) | Msvm_GuestNetworkAdapterConfiguration | DHCPEnabled
ID de carte réseau (cartes réseau synthétiques) | Msvm_SyntheticEthernetPortSettingData | InstanceID
Adresse MAC de la carte réseau (cartes réseau synthétiques) | Msvm_SyntheticEthernetPortSettingData | Adresse
ID de la carte réseau (cartes réseau héritées) | MsvmEmulatedEthernetPortSetting Data | InstanceID
ID MAC de la carte réseau (cartes réseau héritées) | MsvmEmulatedEthernetPortSetting Data | Adresse

### <a name="performance-data"></a>Données de performances

Voici les données de performances des machines virtuelles Hyper-V que l’appliance collecte et envoie à Azure.

**Classe du compteur de performances** | **Compteur** | **Évaluation de l'impact**
--- | --- | ---
Processeur virtuel de l’hyperviseur Hyper-V | % de temps d'exécution de l’invité | Taille de machine virtuelle recommandée/coût
Machine virtuelle de la mémoire dynamique Hyper-V | Pression actuelle (%)<br/> Mémoire physique visible de l’invité (Mo) | Taille de machine virtuelle recommandée/coût
Dispositif de stockage virtuel Hyper-V | Octets lus/seconde | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Dispositif de stockage virtuel Hyper-V | Octets écrits/seconde | Calcul de la taille du disque, du coût de stockage et de la taille de la machine virtuelle
Carte réseau virtuelle Hyper-V | Octets reçus/seconde | Calcul de la taille de machine virtuelle
Carte réseau virtuelle Hyper-V | Octets envoyés/seconde | Calcul de la taille de machine virtuelle

- L'utilisation du processeur représente la somme de toutes les utilisations pour tous les processeurs virtuels connectés à une machine virtuelle.
- L'utilisation de la mémoire correspond à (Pression actuelle * Mémoire physique visible de l'invité) / 100.
- Les valeurs d'utilisation du disque et du réseau sont collectées à partir des compteurs de performances Hyper-V répertoriés.


## <a name="collected-data---physical"></a>Données collectées – Physique

L’appliance collecte les métadonnées, les données de performance et les données d’analyse des dépendances (si l’[analyse des dépendances](concepts-dependency-visualization.md) sans agent est utilisée).

### <a name="windows-metadata"></a>Métadonnées Windows

Les métadonnées découvertes par l’appliance Azure Migrate vous aident à déterminer si les machines et les applications sont prêtes pour la migration vers Azure : dimensionnement approprié des machines et des applications, coûts des plans et analyse des dépendances des applications. Microsoft n’utilise pas ces données dans le cadre des audits de conformité des licences.

Voici la liste complète des métadonnées des serveurs Windows que l’appliance collecte et envoie à Azure.

**DONNÉES** | **CLASSE WMI** | **PROPRIÉTÉ DE LA CLASSE WMI**
--- | --- | ---
FQDN | Win32_ComputerSystem | Domain, Name, PartOfDomain
Nombre de cœurs de processeur | Win32_PRocessor | NumberOfCores
Mémoire allouée | Win32_ComputerSystem | TotalPhysicalMemory
Numéro de série du BIOS | Win32_ComputerSystemProduct | IdentifyingNumber
GUID du BIOS | Win32_ComputerSystemProduct | UUID
Type de démarrage | Win32_DiskPartition | Recherchez une partition dont Type = **GPT:System** pour EFI/BIOS
Nom du système d’exploitation | Win32_OperatingSystem | Caption
Version du SE |Win32_OperatingSystem | Version
Architecture du système d’exploitation | Win32_OperatingSystem | OSArchitecture
Nombre de disques | Win32_DiskDrive | Model, Size, DeviceID, MediaType, Name
Taille du disque | Win32_DiskDrive | Taille
Liste de cartes réseau | Win32_NetworkAdapterConfiguration | Description, Index
Adresse IP de la carte réseau | Win32_NetworkAdapterConfiguration | IPAddress
Adresse MAC de la carte réseau | Win32_NetworkAdapterConfiguration | MACAddress

### <a name="linux-metadata"></a>Métadonnées Linux

Voici la liste complète des métadonnées des serveurs Linux que l’appliance collecte et envoie à Azure.

**DONNÉES** | **LINUX** 
--- | --- 
FQDN | cat /proc/sys/kernel/hostname, hostname -f
Nombre de cœurs de processeur |  /proc/cpuinfo \| awk '/^processor/{print $3}' \| wc -l
Mémoire allouée | cat /proc/meminfo \| grep MemTotal \| awk '{printf "%.0f", $2/1024}'
Numéro de série du BIOS | lshw \| grep "serial:" \| head -n1 \| awk '{print $2}' <br/> /usr/sbin/dmidecode -t 1 \| grep 'Serial' \| awk '{ $1="" ; $2=""; print}'
GUID du BIOS | cat /sys/class/dmi/id/product_uuid
Type de démarrage | [ -d /sys/firmware/efi ] && echo EFI \|\| echo BIOS
Nom/version du système d’exploitation | Nous accédons à ces fichiers pour déterminer la version et le nom du système d’exploitation :<br/><br/> /etc/os-release<br/> /usr/lib/os-release <br/> /etc/enterprise-release <br/> /etc/redhat-release<br/> /etc/oracle-release<br/>  /etc/SuSE-release<br/>  /etc/lsb-release  <br/> /etc/debian_version
Architecture du système d’exploitation | Uname -m
Nombre de disques | fdisk -l \| egrep 'Disk.*bytes' \| awk '{print $2}' \| cut -f1 -d ':'
Disque de démarrage | df /boot \| sed -n 2p \| awk '{print $1}'
Taille du disque | fdisk -l \| egrep 'Disk.*bytes' \| egrep $disk: \| awk '{print $5}'
Liste de cartes réseau | ip -o -4 addr show \| awk '{print $2}'
Adresse IP de la carte réseau | ip addr show $nic \| grep inet \| awk '{print $2}' \| cut -f1 -d "/" 
Adresse MAC de la carte réseau | ip addr show $nic \| grep ether \| awk '{print $2}'

### <a name="windows-performance-data"></a>Données de performances Windows

Voici les données de performances des serveurs Windows que l’appliance collecte et envoie à Azure.

**Données** | **Classe WMI** | **Propriété de classe WMI**
--- | --- | ---
Utilisation de l’UC | Win32_PerfFormattedData_PerfOS_Processor | PercentIdleTime
Utilisation de la mémoire | Win32_PerfFormattedData_PerfOS_Memory | AvailableMBytes
Nombre de cartes réseau | Win32_PerfFormattedData_Tcpip_NetworkInterface | Obtient le nombre de périphériques réseau.
Données reçues par la carte réseau | Win32_PerfFormattedData_Tcpip_NetworkInterface  | BytesReceivedPerSec
Données transmises par la carte réseau | BWin32_PerfFormattedData_Tcpip_NetworkInterface | BytesSentPersec
Nombre de disques | BWin32_PerfFormattedData_PerfDisk_PhysicalDisk | Nombre de disques
Détails du disque | Win32_PerfFormattedData_PerfDisk_PhysicalDisk | DiskWritesPerSec, DiskWriteBytesPerSec, DiskReadsPerSec, DiskReadBytesPerSec.

### <a name="linux-performance-data"></a>Données de performances Linux

Voici les données de performances des serveurs Linux que l’appliance collecte et envoie à Azure.

**Données** | **Linux** 
--- | --- 
Utilisation de l’UC | cat /proc/stat/| grep 'cpu' /proc/stat
Utilisation de la mémoire | free \| grep Mem \| awk '{print $3/$2 * 100.0}'
Nombre de cartes réseau | lshw -class network \| grep eth[0-60] \| wc -l
Données reçues par la carte réseau | cat /sys/class/net/eth$nic/statistics/rx_bytes
Données transmises par la carte réseau | cat /sys/class/net/eth$nic/statistics/tx_bytes
Nombre de disques | fdisk -l \| egrep 'Disk.*bytes' \| awk '{print $2}' \| cut -f1 -d ':'
Détails du disque | cat /proc/diskstats


## <a name="appliance-upgrades"></a>Mises à niveau d’appliance

L'appliance est mise à niveau à mesure que les agents Azure Migrate fonctionnant sur l'appliance sont mis à jour. Ceci se produit automatiquement, car la mise à jour automatique est activée par défaut sur l’appliance. Vous pouvez modifier ce paramètre par défaut pour mettre à jour les services de l’appliance manuellement.

### <a name="turn-off-auto-update"></a>Désactiver la mise à jour automatique

1. Sur l’ordinateur exécutant l’appliance, ouvrez l’Éditeur du Registre.
2. Accédez à **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Pour désactiver la mise à jour automatique, créez une clé de Registre **AutoUpdate** avec la valeur DWORD 0.

    ![Définir la clé de Registre](./media/migrate-appliance/registry-key.png)


### <a name="turn-on-auto-update"></a>Activer la mise à jour automatique

Vous pouvez activer la mise à jour automatique à l’aide de l’une des méthodes suivantes :

- En supprimant la clé de registre AutoUpdate de HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance.
- Cliquez sur **Afficher les services de l’appliance** résultant de la dernière vérification des mises à jour dans le panneau **Configurer les prérequis** pour activer la mise à jour automatique.

Pour supprimer la clé de Registre :

1. Sur l’ordinateur exécutant l’appliance, ouvrez l’Éditeur du Registre.
2. Accédez à **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureAppliance**.
3. Supprimez la clé de Registre **AutoUpdate** précédemment créée pour désactiver la mise à jour automatique.

Pour activer à partir d’Appliance Configuration Manager, une fois la découverte terminée :

1. Dans le gestionnaire de configuration de l’appliance, accédez au panneau **Configurer les prérequis**.
2. Dans la dernière vérification des mises à jour, cliquez sur **Afficher les services de l’appliance**, puis cliquez sur le lien pour activer la mise à jour automatique.

    ![Activer les mises à jour automatiques](./media/migrate-appliance/autoupdate-off.png)

### <a name="check-the-appliance-services-version"></a>Vérifier la version des services de l’appliance

Vous pouvez vérifier la version des services de l’appliance à l’aide de l’une des méthodes suivantes :

- Dans le gestionnaire de configuration de l’appliance, accédez au panneau **Configurer les prérequis**.
- Sur l’ordinateur de l’appliance, dans le **Panneau de configuration** > **Programmes et fonctionnalités**.

Pour vérifier dans le gestionnaire de configuration de l’appliance :

1. Dans le gestionnaire de configuration de l’appliance, accédez au panneau **Configurer les prérequis**.
2. Dans la dernière vérification des mises à jour, cliquez sur **Afficher les services de l’appliance**.

    ![Vérifier la version](./media/migrate-appliance/versions.png)

Pour vérifier dans le Panneau de configuration :

1. Sur l’appliance, cliquez sur **Démarrer** > **Panneau de configuration** > **Programmes et fonctionnalités**
2. Vérifiez les versions des services de l’appliance dans la liste.

    ![Vérifier la version dans le Panneau de configuration](./media/migrate-appliance/programs-features.png)

### <a name="manually-update-an-older-version"></a>Mise à jour manuelle d'une ancienne version

Si vous utilisez une ancienne version pour l'un des composants, vous devez désinstaller le service puis effectuer une mise à jour manuelle avec la dernière version.

1. Pour vérifier les dernières versions d’un service d’appliance, [téléchargez](https://aka.ms/latestapplianceservices) le fichier LatestComponents.json.
2.    Après le téléchargement, ouvrez le fichier LatestComponents.json dans le Bloc-notes.
3. Vous trouverez dans ce fichier la dernière version du service ainsi que le lien de téléchargement correspondant. Par exemple :

    "Name": "ASRMigrationWebApp", "DownloadLink": "https://download.microsoft.com/download/f/3/4/f34b2eb9-cc8d-4978-9ffb-17321ad9b7ed/MicrosoftAzureApplianceConfigurationManager.msi", "Version": "6.0.211.2", "Md5Hash": "e00a742acc35e78a64a6a81e75469b84"

4.    Téléchargez la dernière version d'un service obsolète en utilisant le lien de téléchargement figurant dans le fichier.
5. Après le téléchargement, exécutez la commande suivante dans une fenêtre de commande d'administrateur pour vérifier l'intégrité du fichier msi téléchargé.

    ``` C:\>Get-FileHash -Path <file_location> -Algorithm [Hashing Algorithm] ``` Par exemple :  C:\>CertUtil -HashFile C:\Users\public\downloads\MicrosoftAzureApplianceConfigurationManager.MSI MD5

5. Vérifiez que la sortie de la commande correspond à l'entrée de la valeur de hachage pour le service dans le fichier (par exemple, la valeur de hachage MD5 ci-dessus).
6. Exécutez maintenant le fichier msi pour installer le service. L’installation est silencieuse et la fenêtre d'installation se ferme une fois l’opération terminée.
7. Une fois l'installation terminée, vérifiez la version du service dans **Panneau de configuration** > **Programmes et fonctionnalités**. La version du service devrait maintenant être mise à jour avec la dernière version indiquée dans le fichier json.



## <a name="next-steps"></a>Étapes suivantes

- [Découvrez comment](how-to-set-up-appliance-vmware.md) configurer l’appliance pour VMware.
- [Découvrez comment](how-to-set-up-appliance-hyper-v.md) configurer l’appliance pour Hyper-V.
- [Découvrez comment](how-to-set-up-appliance-physical.md) configurer l’appliance pour des serveurs physiques.

