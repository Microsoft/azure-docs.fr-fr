---
title: Recommandations de sécurité pour les machines virtuelles dans Azure
description: Ces recommandations concernant les machines virtuelles dans Azure vous aideront à respecter les obligations de sécurité décrites dans le modèle de responsabilité partagée et à améliorer la sécurité globale de vos déploiements.
author: msmbaldwin
manager: RKarlin
ms.service: virtual-machines
ms.subservice: security
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: mbaldwin
ms.custom: security-recommendations
ms.reviewer: cynthn
ms.openlocfilehash: 95ba93c4a5d71e6813c1bee144c983e1dfd8c89d
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500323"
---
# <a name="security-recommendations-for-virtual-machines-in-azure"></a>Recommandations de sécurité pour les machines virtuelles dans Azure


Cet article contient des recommandations de sécurité pour les Machines virtuelles Azure. Suivez ces recommandations pour respecter plus facilement les obligations de sécurité décrites dans notre modèle de responsabilité partagée. Ces recommandations vous aideront également à améliorer la sécurité globale de vos solutions d’application web. Pour plus d’informations sur les mesures prises par Microsoft pour assumer ses responsabilités de fournisseur de services, consultez [Responsabilités partagées pour le cloud computing](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91).

Certaines recommandations de cet article peuvent être mises en œuvre automatiquement par Azure Security Center. Azure Security Center est la première ligne de défense de vos ressources dans Azure. Il analyse régulièrement l’état de sécurité de vos ressources Azure pour identifier les vulnérabilités potentielles. Puis, il fournit des recommandations sur la façon de les corriger. Pour plus d’informations, consultez les [recommandations de sécurité dans Azure Security Center](../security-center/security-center-recommendations.md).

Pour obtenir des informations générales sur Azure Security Center, consultez [Qu’est-ce qu’Azure Security Center ?](../security-center/security-center-introduction.md).

## <a name="general"></a>Général

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Quand vous créez des images de machine virtuelle personnalisées, appliquez les dernières mises à jour. | Avant de créer des images, installez les dernières mises à jour du système d’exploitation et de toutes les applications qui feront partie de votre image.  | - |
| veiller à ce que les machines virtuelles soient toujours à jour. | Vous pouvez utiliser la solution [Update Management](../automation/update-management/overview.md) dans Azure Automation pour gérer les mises à jour de vos systèmes d’exploitation Windows et Linux dans Azure. | [Oui](../security-center/asset-inventory.md) |
| Sauvegardez vos machines virtuelles. | La [sauvegarde Azure](../backup/backup-overview.md) vous permet de protéger vos données d’application à des coûts d’exploitation minimes. Les erreurs rencontrées par les applications peuvent endommager vos données et les erreurs humaines peuvent introduire des bogues dans vos applications. La sauvegarde Azure protège vos machines virtuelles exécutant Windows et Linux. | - |
| Utilisez plusieurs machines virtuelles pour plus de résilience et de disponibilité. | Si votre machine virtuelle exécute des applications qui doivent être hautement disponibles, utilisez plusieurs machines virtuelles ou des [groupes à haute disponibilité](./manage-availability.md). | - |
| Adoptez une stratégie BCDR (continuité d’activité et reprise d’activité). | Azure Site Recovery vous permet de choisir parmi différentes options conçues pour la continuité de l’activité. Il prend en charge différents scénarios de réplication et de basculement. Pour plus d’informations, consultez [À propos de Site Recovery](../site-recovery/site-recovery-overview.md). | - |

## <a name="data-security"></a>Sécurité des données

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Chiffrez les disques de système d’exploitation. | [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) vous permet de chiffrer vos disques de machines virtuelles IaaS Windows et Linux. Sans les clés nécessaires, le contenu des disques chiffrés est illisible. Le chiffrement des disques protège les données stockées contre tout accès non autorisé qui serait rendu possible par la copie du disque.| [Oui](../security-center/asset-inventory.md) |
| Chiffrez les disques de données. | [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) vous permet de chiffrer vos disques de machines virtuelles IaaS Windows et Linux. Sans les clés nécessaires, le contenu des disques chiffrés est illisible. Le chiffrement des disques protège les données stockées contre tout accès non autorisé qui serait rendu possible par la copie du disque.| -  |
| Limitez le nombre de logiciels installés. | Installez uniquement les logiciels nécessaires à l’application correcte de votre solution. Cette règle permet de réduire la surface d’attaque de votre solution. | - |
| Utilisez des logiciels antivirus ou anti-programmes malveillants. | Azure vous permet d’utiliser des logiciels anti-programmes malveillants provenant de fournisseurs de sécurité tels que Microsoft, Symantec, Trend Micro et Kaspersky. Ces logiciels permettent de protéger vos machines virtuelles contre les fichiers malveillants, les logiciels de publicité et d’autres menaces. Vous pouvez déployer Microsoft Antimalware en fonction des charges de travail de votre application. Microsoft Antimalware n’est disponible que pour les ordinateurs Windows. Utilisez une configuration de base sécurisée par défaut ou une configuration personnalisée avancée. Pour plus d’informations, consultez [Microsoft Antimalware pour les services cloud Azure et les machines virtuelles](../security/fundamentals/antimalware.md). | - |
| Stockez les secrets et les clés de façon sécurisée. | Simplifiez la gestion de vos secrets et de vos clés en fournissant aux propriétaires d’applications une option sécurisée et gérée de manière centralisée. Cette gestion réduit le risque de compromissions ou de fuites accidentelles. Azure Key Vault permet de stocker les clés dans des modules de sécurité matériels (HSM) certifiés conformes aux normes FIPS 140-2 de niveau 2. Si vous avez besoin de stocker vos clés et vos secrets à l’aide de la norme FIPS 140.2 de niveau 3, vous pouvez utiliser [Azure Dedicated HSM](../dedicated-hsm/overview.md). | - |

## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès 

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Centralisez l’authentification des machines virtuelles. | Vous pouvez centraliser l’authentification de vos machines virtuelles Windows et Linux à l’aide de l’[authentification Azure Active Directory](../active-directory/develop/authentication-vs-authorization.md). | - |

## <a name="monitoring"></a>Surveillance

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Supervisez vos machines virtuelles. | Vous pouvez utiliser [Azure Monitor pour machines virtuelles](../azure-monitor/insights/vminsights-overview.md) pour superviser l’état des machines virtuelles et des groupes de machines virtuelles identiques Azure. Une machine virtuelle insuffisamment performante peut entraîner une interruption de service, réduisant ainsi la disponibilité. | - |

## <a name="networking"></a>Mise en réseau

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Limitez l’accès aux ports de gestion. | Les attaquants analysent les plages d’adresses IP de cloud public pour détecter les ports de gestion ouverts et tenter des attaques « faciles », en utilisant des mots de passe courants ou en exploitant des vulnérabilités non corrigées connues. Vous pouvez utiliser l’[accès juste-à-temps (JAT) aux machines virtuelles](../security-center/security-center-just-in-time.md) pour verrouiller le trafic entrant vers vos machines virtuelles Azure, ce qui réduit l’exposition aux attaques et facilite la connexion aux machines virtuelles en cas de besoin. | - |
| Limitez l’accès réseau. | Les groupes de sécurité réseau vous permettent de restreindre l’accès réseau et de contrôler le nombre de points de terminaison exposés. Pour plus d’informations, consultez [Créer, changer ou supprimer un groupe de sécurité réseau](../virtual-network/manage-network-security-group.md). | - |

## <a name="next-steps"></a>Étapes suivantes

Consultez votre fournisseur d’applications pour en savoir plus sur les exigences de sécurité supplémentaires. Pour plus d’informations sur le développement d’applications sécurisées, consultez [Documentation sur le développement sécurisé](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/).