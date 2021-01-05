---
title: Références SKU Machines virtuelles prises en charge pour Azure Policy
description: Article décrivant les références SKU de machine virtuelle (par éditeur d’image, offre d’image et SKU d’image) qui sont prises en charge pour les stratégies Azure intégrées que fournit le service Sauvegarde
ms.topic: conceptual
ms.date: 11/08/2019
ms.openlocfilehash: 4658d1beffe707682e173491edea1eac0db9c811
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82183617"
---
# <a name="supported-vm-skus-for-azure-policy"></a>Références SKU Machines virtuelles prises en charge pour Azure Policy

Sauvegarde Azure fournit une stratégie intégrée (via Azure Policy) qui peut être attribuée à **toutes les machines virtuelles Azure dans un emplacement spécifié d’un abonnement ou d’un groupe de ressources**. Quand cette stratégie est affectée à une étendue donnée, toutes les machines virtuelles créées dans cette étendue sont automatiquement configurées pour la sauvegarde dans un **coffre existant au même emplacement et dans le même abonnement**. Le tableau ci-dessous liste toutes les références SKU de machine virtuelle prises en charge par cette stratégie.

## <a name="supported-vms"></a>Machines virtuelles prises en charge*

**Nom de la stratégie :** Configurer la sauvegarde sur les machines virtuelles d’un emplacement dans un coffre central existant au même emplacement

Éditeur de l’image | Offre de l’image | Référence d’image
--- | --- | ---
MicrosoftWindowsServer | WindowsServer | Windows Server 2008 R2 SP1 (2008-R2-SP1)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2008 R2 SP (2008-R2-SP1-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2012 Datacenter (2012-Datacenter)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2012 Datacenter (2012-Datacenter-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2012 R2 Datacenter (2012-R2-Datacenter)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2012 R2 Datacenter (2012-R2-Datacenter-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 Datacenter (2016-Datacenter)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 Datacenter - Server Core (2016-Datacenter-Server-Core)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2016 Datacenter - Server Core (2016-Datacenter-Server-Core-smalldisk)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2016 Datacenter (2016-Datacenter-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core avec Containers (2016-Datacenter-with-Containers)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 Remote Desktop Session Host 2016 (2016-Datacenter-with-RDSH)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter (2019-Datacenter)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core (2019-Datacenter-Core)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2019 Datacenter Server Core (2019-Datacenter-Core-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core avec Containers (2019-Datacenter-Core-with-Containers)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2019 Datacenter Server Core avec Containers (2019-Datacenter-Core-with-Containers-smalldisk)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2019 Datacenter (2019-Datacenter-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter avec Containers (2019-Datacenter-with-Containers)
MicrosoftWindowsServer | WindowsServer | [smalldisk] Windows Server 2019 Datacenter avec Containers (2019-Datacenter-with-Containers-smalldisk)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter (zh-cn) (2019-Datacenter-zhcn)
MicrosoftWindowsServer | WindowsServerSemiAnnual | Datacenter-Core-1709-smalldisk
MicrosoftWindowsServer | WindowsServerSemiAnnual | Datacenter-Core-1709-with-Containers-smalldisk
MicrosoftWindowsServer | WindowsServerSemiAnnual | Datacenter-Core-1803-with-Containers-smalldisk
MicrosoftWindowsServerHPCPack | WindowsServerHPCPack | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2016SP1-WS2016 | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2016-WS2016 | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2016SP1-WS2016-BYOL | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2012SP3-WS2012R2 | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2016-WS2012R2 | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2014SP2-WS2012R2 | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2012SP3-WS2012R2-BYOL | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2014SP1-WS2012R2-BYOL | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2014SP2-WS2012R2-BYOL | Toutes les références SKU d’image
MicrosoftSQLServer | SQL2016-WS2012R2-BYOL | Toutes les références SKU d’image
MicrosoftRServer | MLServer-WS2016 | Toutes les références SKU d’image
MicrosoftVisualStudio | VisualStudio | Toutes les références SKU d’image
MicrosoftVisualStudio |  Windows | Toutes les références SKU d’image
MicrosoftDynamicsAX | Dynamics | Pre-Req-AX7-Onebox-U8
microsoft-ads | windows-data-science-vm | Toutes les références SKU d’image
MicrosoftWindowsDesktop | Windows-10 | Toutes les références SKU d’image
Red Hat | RHEL | 6.7, 6.8, 6.9, 6.10, 7.2, 7.3, 7.4, 7.5, 7.6, 7.7
Red Hat | RHEL-SAP-HANA | 6.7, 7.2, 7.3
SUSE | SLES | 12.X
SUSE | SLES-HPC | 12.X
SUSE | SLES-HPC-Priority | 12.X
SUSE | SLES-SAP | 12.X
SUSE | SLES-SAP-BYOS | 12.X
SUSE | SLES-Priority | 12.X
SUSE | SLES-BYOS | 12.X
SUSE | SLES-SAPCAL | 12.X
SUSE | SLES-Standard | 12.X
Canonical | UbuntuServer | 14.04.0-LTS
Canonical | UbuntuServer | 14.04.1-LTS
Canonical | UbuntuServer | 14.04.2-LTS
Canonical | UbuntuServer | 14.04.3-LTS
Canonical | UbuntuServer | 14.04.4-LTS
Canonical | UbuntuServer | 14.04.5-DAILY-LTS
Canonical | UbuntuServer | 14.04.5-LTS
Canonical | UbuntuServer | 16.04-DAILY-LTS
Canonical | UbuntuServer | 16.04-LTS
Canonical | UbuntuServer | 16.04.0-LTS
Canonical | UbuntuServer | 18.04-DAILY-LTS
Canonical | UbuntuServer | 18.04-LTS
Oracle | Oracle-Linux | 6.8, 6.9, 6.10, 7.3, 7.4, 7.5, 7.6
OpenLogic | CentOS | 6.X, 7.X
OpenLogic | CentOS–LVM | 6.X, 7.X
OpenLogic | CentOS–SRIOV | 6.X, 7.X
cloudera | cloudera-centos-os | 7.X
