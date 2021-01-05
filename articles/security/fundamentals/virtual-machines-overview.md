---
title: Fonctionnalités de sécurité utilisées avec les machines virtuelles Azure
titleSuffix: Azure security
description: Cet article fournit une vue d’ensemble des principales fonctionnalités de sécurité Azure pouvant être utilisées avec Azure Virtual Machines.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: 467b2c83-0352-4e9d-9788-c77fb400fe54
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/2/2019
ms.author: terrylan
ms.openlocfilehash: ed2cb967b24ce5abdebadfe29847ae1a16b4e745
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498742"
---
# <a name="azure-virtual-machines-security-overview"></a>Vue d’ensemble de la sécurité des machines virtuelles Azure
Cet article fournit une vue d’ensemble des principales fonctionnalités de sécurité Azure pouvant être utilisées avec les machines virtuelles.

Vous pouvez utiliser des machines virtuelles Azure pour déployer un large éventail de solutions informatiques et ce, en toute flexibilité. Le service prend en charge Microsoft Windows, Linux, Microsoft SQL Server, Oracle, IBM, SAP et Azure BizTalk Services. Vous pouvez ainsi déployer n’importe quelle charge de travail et n’importe quel langage sur quasiment n’importe quel système d’exploitation.

Une machine virtuelle Azure vous donne la flexibilité de la virtualisation sans devoir acheter le matériel physique qui exécute la machine virtuelle ni en assurer la maintenance. Vous pouvez créer et déployer vos applications en ayant la certitude que vos données sont protégées au sein de centres de données hautement sécurisés.

Avec Azure, vous pouvez créer des solutions sécurisées et conformes qui :

* Protègent vos machines virtuelles contre les virus et logiciels malveillants.
* Chiffrent vos données sensibles.
* Sécurisent le trafic réseau.
* Identifient et détectent les menaces.
* Respectent les exigences en matière de conformité.  

## <a name="antimalware"></a>Logiciel anti-programme malveillant

Azure met à votre disposition des logiciels anti-programmes malveillants provenant de fournisseurs de sécurité tels que Microsoft, Symantec, Trend Micro et Kaspersky. Ces logiciels permettent de protéger vos machines virtuelles contre les fichiers malveillants, les logiciels de publicité et d’autres menaces.

Microsoft Antimalware pour Azure Cloud Services et Machines virtuelles offre une fonctionnalité de protection en temps réel qui permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants.  Microsoft Antimalware pour Azure fournit des alertes configurables lorsqu’un logiciel malveillant ou indésirable connu tente de s’installer ou de s’exécuter sur vos systèmes Azure.

Microsoft Antimalware est une solution d’agent unique pour les applications et les environnements client. Elle est conçue pour s’exécuter en arrière-plan sans intervention humaine. Vous pouvez déployer la protection en fonction des besoins de vos charges de travail d’application, avec une configuration de base sécurisée par défaut ou une configuration personnalisée avancée, y compris pour la surveillance anti-programmes malveillants.

En savoir plus sur [Microsoft Antimalware pour Azure](antimalware.md) et les fonctionnalités de base disponibles.

Pour en savoir plus sur les logiciels anti-programme malveillant pour protéger vos machines virtuelles :

* [Déploiement de solutions anti-programmes malveillants sur des machines virtuelles Azure (en anglais)](https://azure.microsoft.com/blog/deploying-antimalware-solutions-on-azure-virtual-machines/)
* [Installation et configuration de Trend Micro Deep Security comme service sur une machine virtuelle Windows](../../virtual-machines/extensions/trend.md)
* [Installation et configuration de Symantec Endpoint Protection sur une machine virtuelle Windows](../../virtual-machines/extensions/symantec.md)
* [Solutions de sécurité dans Azure Marketplace](https://azure.microsoft.com/marketplace/?term=security)

Pour une protection encore plus puissante, utilisez [Windows Defender - Protection avancée contre les menaces](/windows/security/threat-protection/windows-defender-atp/windows-defender-advanced-threat-protection). Windows Defender ATP offre les fonctionnalités suivantes :

* [Réduction de la surface d’attaque](/windows/security/threat-protection/windows-defender-atp/overview-attack-surface-reduction)  
* [Protection de nouvelle génération](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10)  
* [Protection et réponse du point de terminaison](/windows/security/threat-protection/windows-defender-atp/overview-endpoint-detection-response)
* [Investigation et correction automatisées](/windows/security/threat-protection/windows-defender-atp/automated-investigations-windows-defender-advanced-threat-protection)
* [Degré de sécurisation](/windows/security/threat-protection/microsoft-defender-atp/tvm-microsoft-secure-score-devices)
* [Recherche avancée](/windows/security/threat-protection/windows-defender-atp/overview-hunting-windows-defender-advanced-threat-protection)
* [Gestion et API](/windows/security/threat-protection/windows-defender-atp/management-apis)
* [Protection contre les menaces Microsoft](/windows/security/threat-protection/windows-defender-atp/threat-protection-integration)

En savoir plus :

* [Prise en main de WDATP](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)  
* [Vue d’ensemble des fonctionnalités de WDATP](/windows/security/threat-protection/windows-defender-atp/overview)  

## <a name="hardware-security-module"></a>Module de sécurité matériel

Une meilleure clé de sécurité permet d’améliorer les protections par chiffrement et authentification. Vous pouvez simplifier la gestion et la sécurité de vos clés et secrets critiques en les stockant dans Azure Key Vault.

Key Vault permet de stocker les clés dans des modules de sécurité matériels (HSM) certifiés conformes aux normes FIPS 140-2 de niveau 2. Vos clés de chiffrement SQL Server pour la sauvegarde ou le [chiffrement transparent des données](/sql/relational-databases/security/encryption/transparent-data-encryption) peuvent toutes être stockées dans Key Vault avec les clés ou secrets de vos applications. Les autorisations et l’accès à ces éléments protégés sont gérés via [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/).

En savoir plus :

* [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)
* [Blog Azure Key Vault](/archive/blogs/kv/)

## <a name="virtual-machine-disk-encryption"></a>Chiffrement du disque de machine virtuelle

Azure Disk Encryption est une nouvelle fonctionnalité de chiffrement de vos disques de machine virtuelle Windows et Linux. Azure Disk Encryption utilise la fonctionnalité standard [BitLocker](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc732774(v=ws.11)) Windows et la fonctionnalité [dm-crypt](https://en.wikipedia.org/wiki/Dm-crypt) de Linux pour fournir le chiffrement de volume du système d’exploitation et des disques de données.

La solution est intégrée à Azure Key Vault, ce qui vous permet de contrôler et de gérer les clés et les secrets de chiffrement de disque dans votre abonnement Key Vault. Elle garantit que toutes les données sur les disques de vos machines virtuelles sont chiffrées au repos dans le stockage Azure.

En savoir plus :

* [Azure Disk Encryption pour machines virtuelles Iaas](./azure-disk-encryption-vms-vmss.md)
* [Démarrage rapide : Chiffrer une machine virtuelle IaaS Windows avec Azure PowerShell](../../virtual-machines/linux/disk-encryption-powershell-quickstart.md)

## <a name="virtual-machine-backup"></a>Sauvegarde de machine virtuelle

Sauvegarde Azure est une solution évolutive qui permet de protéger les données de vos applications, sans investissement en capital et avec des frais de fonctionnement minimaux. Les erreurs rencontrées par les applications peuvent endommager vos données et les erreurs humaines peuvent introduire des bogues dans vos applications. Avec Sauvegarde Azure, vos machines virtuelles exécutant Windows et Linux sont protégées.

En savoir plus :

* [Qu’est-ce que Sauvegarde Azure ?](../../backup/backup-overview.md)
* [Service Sauvegarde Azure – Forum aux questions](../../backup/backup-azure-backup-faq.md)

## <a name="azure-site-recovery"></a>Azure Site Recovery

Une partie importante de la stratégie de continuité des activités et de récupération d’urgence de votre organisation consiste à savoir comment maintenir les charges de travail et les applications d’entreprise opérationnelles lorsque des interruptions planifiées et non planifiées se produisent. Azure Site Recovery aide à coordonner la réplication, le basculement et la récupération des charges de travail et des applications afin qu’elles soient disponibles à partir d’un site secondaire si votre site principal tombe en panne.

Site Recovery :

* **Simplifie votre stratégie de continuité d'activité et de reprise d'activité** : Site Recovery facilite la gestion de la réplication, du basculement et de la récupération de plusieurs charges de travail et applications métier à partir d’un emplacement unique. Site Recovery orchestre la réplication et le basculement, sans intercepter les données de vos applications ni posséder des informations à leur sujet.
* **Fournit une réplication flexible** : À l’aide de Site Recovery, vous pouvez répliquer des charges de travail s’exécutant sur des machines virtuelles Hyper-V et VMware, et des serveurs physiques Windows/Linux.
* **Prend en charge le basculement et la récupération** : Site Recovery fournit des tests de basculement pour prendre en charge des exercices de récupération d’urgence sans affecter les environnements de production. Vous pouvez également exécuter des basculements planifiés avec une perte de données zéro pour les interruptions attendues, ou des basculements non planifiés avec une perte de données minimale (en fonction de la fréquence de réplication) pour les incidents inattendus. Après le basculement, vous pouvez procéder à une restauration automatique sur vos sites principaux. Site Recovery fournit des plans de récupération qui peuvent inclure des scripts et des classeurs Azure Automation afin que vous puissiez personnaliser le basculement et la récupération d’applications multiniveaux.
* **Élimine les centres de données secondaires** : Vous pouvez répliquer vos éléments vers un site local secondaire ou sur Azure. L’utilisation d’Azure comme destination de la récupération d’urgence élimine le coût et la complexité associés à la gestion d’un site secondaire. Les données répliquées sont stockées dans Azure Storage.
* **S’intègre aux technologies BCDR existantes** : Site Recovery est compatible avec les fonctionnalités BCDR d’autres applications. Par exemple, vous pouvez utiliser Site Recovery pour aider à protéger le serveur principal SQL Server des charges de travail d’entreprise. Cela inclut la prise en charge native de SQL Server AlwaysOn pour gérer le basculement des groupes de disponibilité.

En savoir plus :

* [Qu’est-ce que le service Azure Site Recovery ?](../../site-recovery/site-recovery-overview.md)
* [Comment Azure Site Recovery fonctionne-t-il ?](../../site-recovery/azure-to-azure-architecture.md)
* [Quelles sont les charges de travail protégées par Azure Site Recovery ?](../../site-recovery/site-recovery-workload.md)

## <a name="virtual-networking"></a>Réseau virtuel

Les machines virtuelles nécessitent une connectivité réseau. Pour cela, les machines virtuelles doivent être connectées à un réseau virtuel Azure.

Un réseau virtuel Azure est une construction logique basée sur le réseau physique Azure. Chaque réseau virtuel logique Azure est isolé des autres réseaux virtuels Azure. Cet isolement permet de s’assurer que le trafic réseau dans votre déploiement n’est pas accessible aux autres clients Microsoft Azure.

En savoir plus :

* [Vue d’ensemble de la sécurité réseau d’Azure](network-overview.md)
* [Présentation du réseau virtuel](../../virtual-network/virtual-networks-overview.md)
* [Fonctionnalités de mise en réseau et partenariats pour les scénarios d’entreprise](https://azure.microsoft.com/blog/networking-enterprise/)

## <a name="security-policy-management-and-reporting"></a>Gestion des stratégies de sécurité et création de rapports

Azure Security Center vous aide à vous empêcher, détecter et répondre aux menaces. Il offre une visibilité et un contrôle complets sur la sécurité de vos ressources Azure. Il fournit des fonctions intégrées de surveillance de la sécurité et de gestion des stratégies sur vos abonnements Azure. Il permet de détecter les menaces qui pourraient passer inaperçues et fonctionne avec un vaste écosystème de solutions de sécurité.

Security Center vous permet d’optimiser et de surveiller la sécurité de vos machines virtuelles grâce aux opérations suivantes :

* Proposition de [suggestions de sécurité](../../security-center/security-center-recommendations.md) pour les machines virtuelles. Les suggestions de sécurité incluent l’application des mises à jour système, la configuration des points de terminaison ACL, l’activation des logiciels anti-programme malveillant, l’activation des groupes de sécurité réseau et l’application du chiffrement de disque.
* Surveillance de l’état de vos machines virtuelles.

En savoir plus :

* [Présentation d’Azure Security Center](../../security-center/security-center-introduction.md)
* [FAQ d’Azure Security Center](../../security-center/faq-general.md)
* [Guide des opérations et de planification d’Azure Security Center](../../security-center/security-center-planning-and-operations-guide.md)

## <a name="compliance"></a>Conformité

Azure Virtual Machines bénéficie des certifications FISMA, FedRAMP, HIPAA, PCI DSS Level 1 et de celles d’autres programmes majeurs de conformité. Cette certification facilite le respect des exigences de conformité pour vos propres applications Azure, et donne à votre entreprise la possibilité de se conformer à un large éventail d’exigences réglementaires internationales et locales.

En savoir plus :

* [Centre de gestion de la confidentialité Microsoft : Conformité](https://www.microsoft.com/en-us/trustcenter/compliance)
* [Cloud de confiance : Sécurité, confidentialité et conformité dans Microsoft Azure](https://download.microsoft.com/download/1/6/0/160216AA-8445-480B-B60F-5C8EC8067FCA/WindowsAzure-SecurityPrivacyCompliance.pdf)

## <a name="confidential-computing"></a>Calcul confidentiel

Même si l’informatique confidentielle ne fait pas techniquement partie de la sécurité des machines virtuelles, le concept de sécurité de la machine virtuelle s’inscrit dans le niveau supérieur de sécurité du « calcul ». L’informatique confidentielle appartient à la catégorie de sécurité « calcul ».

L’informatique confidentielle garantit que lorsque les données sont affichées « en clair », ce qui est requis pour un traitement efficace, les données sont protégées à l’intérieur d’un environnement d’exécution approuvé https://en.wikipedia.org/wiki/Trusted_execution_environment (TEE, également appelé une enclave), dont voici un exemple dans la figure ci-dessous.  

Les environnements TEE s’assurent qu’il n’existe aucun moyen d’afficher les données ou les opérations de l’extérieur, même avec un débogueur. Elles garantissent même que seul le code autorisé peut accéder aux données. Si le code est modifié ou altéré, les opérations sont refusées et l’environnement est désactivé. L’environnement TEE met en œuvre ces protections tout au long de l’exécution du code qu’il contient.

En savoir plus :

* [Présentation de l’informatique confidentielle Azure](https://azure.microsoft.com/blog/introducing-azure-confidential-computing/)  
* [Informatique confidentielle Azure](https://azure.microsoft.com/blog/azure-confidential-computing/)  

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [meilleures pratiques en matière de sécurité](iaas.md) pour les machines virtuelles et les systèmes d’exploitation.