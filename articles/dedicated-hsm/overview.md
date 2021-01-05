---
title: Présentation d’HSM dédié - HSM dédié Azure | Microsoft Docs
description: Azure Dedicated HSM est un service Azure qui permet le stockage de clés de chiffrement dans Azure.
services: dedicated-hsm
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc, seodec18
ms.date: 12/07/2018
ms.author: mbaldwin
ms.openlocfilehash: 6eb82400b2f598b4b54da4b238f4bce71d83de6b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90970203"
---
# <a name="what-is-azure-dedicated-hsm"></a>Présentation du service HSM dédié Azure

Un HSM dédié Azure est un service Azure qui permet le stockage de clés de chiffrement dans Azure. Un HSM dédié répond aux exigences de sécurité les plus strictes. Il constitue la solution idéale pour les clients qui ont besoin d’appareils certifiés FIPS 140-2 de niveau 3 ainsi que d’un contrôle complet et exclusif des appliances HSM. 

 Les appareils HSM sont déployés mondialement dans plusieurs régions Azure. Ils peuvent être facilement approvisionnés sous forme de paires d’appareils et être configurés pour la haute disponibilité. Les appareils HSM peuvent également être approvisionnés dans différentes régions afin d’offrir une garantie en cas de basculement au niveau régional. Microsoft a publié le service HSM dédié à l’aide de l’appliance [SafeNet Luna Network HSM 7 (Modèle A790)](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/) de Gemalto. Cet appareil offre les niveaux de performances et les options d’intégration de services de chiffrement les plus élevés. 

Une fois approvisionnés, les appareils HSM sont connectés directement au réseau virtuel d’un client. Ils sont également accessibles via des applications et outils de gestion hébergés en local lorsque vous configurez une connectivité VPN point à site ou site à site. Les clients obtiennent les logiciels et la documentation nécessaires pour configurer et gérer les appareils HSM à partir du portail de support technique de Gemalto.

## <a name="why-use-azure-dedicated-hsm"></a>Pourquoi utiliser le service Azure Dedicated HSM ?

### <a name="fips-140-2-level-3-compliance"></a>Conformité FIPS 140-2 de niveau 3

De nombreuses organisations doivent se soumettre à des réglementations strictes qui imposent que le stockage des clés de chiffrement respecte les exigences [FIPS 140-2 de niveau 3](https://csrc.nist.gov/publications/detail/fips/140/2/final). Actuellement, le service multilocataire Azure Key Vault de Microsoft fournit uniquement une certification FIPS 140-2 de niveau 2. Azure Dedicated HSM répond à un besoin réel du secteur des services financiers, des agences gouvernementales et d’autres utilisateurs qui doivent répondre aux exigences FIPS 140-2 de niveau 3.

### <a name="single-tenant-devices"></a>Appareils à locataire unique

La plupart de nos clients n’ont besoin que d’un seul locataire pour l’appareil de stockage de chiffrement. Le service Azure Dedicated HSM autorise l’approvisionnement d’un appareil physique à partir de l’un des centres de données mondialement distribués de Microsoft. Une fois le provisionnement effectué pour un client, seul ce client peut accéder à l’appareil.

### <a name="full-administrative-control"></a>Contrôle administratif complet

De nombreux clients exigent un contrôle administratif complet et un accès exclusif sur leur appareil à des fins d’administration. Une fois l’approvisionnement effectué, seul le client dispose d’un accès administratif ou au niveau de l’application à l’appareil.

 Microsoft ne dispose d’aucun contrôle d’administration dès lors que le client accède à l’appareil et modifie son mot de passe. À partir de là, le client est un vrai locataire unique disposant d’un contrôle administratif complet et de capacités de gestion des applications. Microsoft ne conserve aucun accès de surveillance (aucun un rôle d’administrateur) pour la télémétrie via une connexion de port série. Cet accès couvre des analyses matérielles telles que la température, le contrôle d’intégrité de l’alimentation électrique et l’intégrité du ventilateur. 
 
 Le client est libre de désactiver cette surveillance si nécessaire. S’il la désactive, il ne recevra en revanche aucune alerte proactive de contrôle d’intégrité de la part de Microsoft.

### <a name="high-performance"></a>Hautes performances

L’appareil Gemalto a été sélectionné pour ce service pour diverses raisons. Il prend en charge une grande diversité d’algorithmes de chiffrement, de systèmes d’exploitation et d’API. Le modèle spécifique déployé offre d’excellentes performances avec 10 000 opérations par seconde pour RSA-2048. Il prend en charge 10 partitions qui peuvent être utilisées pour des instances d’applications uniques. Il s’agit d’un appareil à faible latence, haute capacité et débit élevé.

### <a name="unique-cloud-based-offering"></a>Offre cloud unique

Microsoft a reconnu un besoin spécifique parmi un ensemble unique de clients. Il est le seul fournisseur cloud à offrir aux nouveaux clients un service HSM dédié conforme à la norme FIPS 140-2 de niveau 3 et une gamme si étendue d’intégration d’applications locales et dans le cloud.

## <a name="is-azure-dedicated-hsm-right-for-you"></a>Azure HSM dédié est-il fait pour vous ?

Azure Dedicated HSM est un service spécialisé ciblant les exigences uniques d’un type spécifique d’organisation à grande échelle. Ainsi, il est probable que la plupart des clients Azure ne présenteront pas le profil d’utilisation pour ce service. Pour eux, le service Azure Key Vault sera plus approprié et économique. Pour vous aider à déterminer l’adéquation à vos besoins, nous avons identifié les critères suivants.

### <a name="best-fit"></a>Adéquation

Le service Azure Dedicated HSM convient plus particulièrement aux scénarios « lift-and-shift » nécessitant un accès direct et unique aux appareils HSM. Voici quelques exemples :

- Migration d'applications locales vers des machines virtuelles Azure
- Migration d’applications à partir d’Amazon AWS EC2 vers des machines virtuelles qui utilisent le service AWS Cloud HSM Classic (Amazon n’offre pas ce service aux nouveaux clients)
- Exécution de logiciels de série, tels qu’Apache/Ngnix SSL Offload, Oracle TDE et ADCS, sur des machines virtuelles Azure 

### <a name="not-a-fit"></a>Inadéquation

Le service Azure Dedicated HSM n’est pas adapté au type de scénario suivant : Les services cloud Microsoft qui prennent en charge le chiffrement avec des clés gérées par le client (tels qu’Azure Information Protection, Azure Disk Encryption, Azure Data Lake Store, Stockage Azure, Azure SQL Database et Clé client Office 365) ne sont pas intégrés à Azure Dedicated HSM.

### <a name="it-depends"></a>Cela dépend

L’adéquation du service Azure Dedicated HSM dépend d’une série potentiellement complexe d’exigences et de compromis que vous pouvez ou non supporter. L’exigence FIPS 140-2 de niveau 3 en est un exemple. Cette exigence est courante et HSM dédié est actuellement la seule option disponible pour y répondre. Si ces obligations légales ne sont pas pertinentes, vous pouvez alors choisir entre Azure Key Vault et Azure Dedicated HSM. Évaluez vos besoins avant de prendre une décision.

Exemples de situations dans lesquelles vous devrez réfléchir à vos options : 

- Nouveau code exécuté sur la machine virtuelle Azure d’un client
- TDE SQL Server sur une machine virtuelle Azure
- Chiffrement Stockage Azure côté client
- SQL Server et Azure SQL DB Always Encrypted

## <a name="next-steps"></a>Étapes suivantes

Il s’agit d’un service hautement spécialisé. Par conséquent, nous vous recommandons de vous familiariser avec les concepts clés décrits dans cette documentation, notamment la tarification, le support et les contrats de niveau de service. 

Les [guides d’intégration Gemalto](https://safenet.gemalto.com/partners/microsoft/) vous aider à faciliter l’approvisionnement des modules HSM dans un environnement de réseau virtuel existant. Des guides pratiques sont également à votre disposition pour vous aider à déterminer la manière de configurer votre architecture de déploiement.

* [Haute disponibilité](high-availability.md)
* [Sécurité physique](physical-security.md)
* [Mise en réseau](networking.md)
* [Prise en charge](supportability.md)
* [Surveillance](monitoring.md)
