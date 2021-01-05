---
title: Présentation de Microsoft Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit Azure Stack Edge Pro avec GPU, solution de stockage qui utilise un appareil physique pour le transfert vers Azure via le réseau.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/23/2020
ms.author: alkohli
ms.openlocfilehash: 059aad8b45aa449f5cd28c97f4ba55e2204c04ce
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937064"
---
# <a name="what-is-azure-stack-edge-pro-with-gpu"></a>Qu’est-ce qu’Azure Stack Edge Pro avec GPU ?

Azure Stack Edge Pro avec GPU est un appareil de computing en périphérie qui prend en charge l’IA et qui présente des fonctionnalités de transfert de données via le réseau. Cet article vous fournit une vue d’ensemble de la solution Azure Stack Edge Pro, ses avantages, ses principales fonctionnalités et les scénarios dans lesquels vous pouvez déployer cet appareil.

Azure Stack Edge Pro avec GPU est une solution de matériel en tant que service. Microsoft fournit un appareil managé dans le cloud qui fait office de passerelle de stockage réseau et intègre un processeur graphique (GPU) permettant une inférence accélérée de l’IA. 

## <a name="use-cases"></a>Cas d'utilisation

Voici différents scénarios d’utilisation d’Azure Stack Edge Pro pour l’inférence rapide de Machine Learning (ML) à la périphérie et le prétraitement des données avant leur envoi à Azure.

- **Inférence avec Azure Machine Learning** : Azure Stack Edge Pro vous permet d’exécuter des modèles Machine Learning (ML) pour obtenir des résultats rapides qui peuvent être traités avant l’envoi des données vers le cloud. Le jeu de données complet peut également être transféré pour continuer à reformer et améliorer vos modèles ML. Pour plus d’informations sur l’utilisation des modèles accélérés matériellement d’Azure ML sur un appareil Azure Stack Edge Pro, consultez [Déployer des modèles accélérés matériellement Azure ML sur Azure Stack Edge Pro](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Prétraiter les données** : transformez les données avant de les envoyer à Azure via des options de calcul telles que des charges de travail conteneurisées et des machines virtuelles pour créer un jeu de données plus actionnable. Le prétraitement peut servir à : 

    - Agréger les données.
    - Modifiez les données, par exemple, pour supprimer des données personnelles.
    - Fractionnez les données pour optimiser le stockage et la bande passante, ainsi que pour les analyser de façon plus approfondie.
    - Analyser et réagir aux événements IoT. 

- **Transférer des données vers Azure via le réseau** : utilisez Azure Stack Edge Pro pour transférer rapidement et facilement des données vers Azure pour le calcul et l’analyse à des fins d’archivage. 

## <a name="key-capabilities"></a>Fonctionnalités clés

Azure Stack Edge Pro intègre les fonctionnalités suivantes :

|Fonctionnalité |Description  |
|---------|---------|
|Inférence accélérée avec l’intelligence artificielle| Activé par le GPU intégré (un ou deux selon le modèle).|
|Edge Computing      |Prend en charge les charges de travail de machine virtuelle et conteneurisées pour permettre l’analyse, le traitement et le filtrage des données. |
|Accès aux données     | Accès direct aux données à partir d’objets blob Stockage Azure et Azure Files à l’aide des API cloud pour le traitement d’autres données dans le cloud. Un cache local sur l’appareil est utilisé pour accéder rapidement aux fichiers les plus récemment utilisés.|
|Géré par le cloud     |L’appareil et le service sont gérés via le portail Azure.  |
|Chargement hors connexion     | Un mode déconnecté prend en charge les scénarios de chargement hors connexion.|
|Protocoles de transfert de fichiers pris en charge      | Prise en charge des protocoles SMB, NFS et REST standard pour l’ingestion des données. <br> Pour plus d’informations sur les versions prises en charge, consultez [Conditions requises pour Azure Stack Edge Pro](azure-stack-edge-system-requirements.md).|
|Actualisation des données     | Possibilité d’actualiser les fichiers locaux avec la dernière version issue du cloud.|
|Chiffrement    | Prise en charge de BitLocker pour chiffrer les données localement et transférer de façon sécurisée des données vers le cloud via *https*.|
|Limitation de bande passante| Limiter l’utilisation de la bande passante pendant les heures de pointe.|
<!--|ExpressRoute | Sécurité accrue avec ExpressRoute. Utilisez la configuration de peering avec laquelle le trafic des appareils locaux vers les points de terminaison de stockage cloud transite par ExpressRoute. Pour plus d’informations, voir [Appairage ExpressRoute](../expressroute/expressroute-introduction.md).|-->

## <a name="components"></a>Components

La solution Azure Stack Edge Pro se compose d’une ressource Azure Stack Edge, d’un appareil physique Azure Stack Edge Pro et d’une interface utilisateur web locale.

* **Appareil physique Azure Stack Edge Pro** : un serveur 1U monté en rack, fourni par Microsoft qui peut être configuré pour envoyer des données vers Azure.
    
* **Ressource Azure Stack Edge** : ressource du portail Azure, qui vous permet de gérer un appareil Azure Stack Edge Pro à partir d’une interface web accessible depuis divers emplacements géographiques. Utilisez la ressource Azure Stack Edge pour créer et gérer des ressources, afficher et gérer les appareils et les alertes, et gérer les partages.  

    Pour plus d’informations, accédez à la section vous expliquant comment [Créer une commande pour votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md#create-a-new-resource).

* **Interface utilisateur web locale Azure Stack Edge Pro** - une interface utilisateur locale basée sur un navigateur sur votre appareil Azure Stack Edge Pro, principalement conçue pour la configuration initiale de l’appareil. Utilisez également l’interface utilisateur web locale pour exécuter des diagnostics, arrêter et redémarrer l’appareil Azure Stack Edge Pro, afficher les journaux d’activité de copie, et contactez le support Microsoft pour créer une demande de service.

    Pour plus d’informations sur l’utilisation de l’interface utilisateur basée sur le web, consultez [Utiliser l’interface utilisateur basée sur le web pour gérer votre Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilité des régions

L’appareil physique Azure Stack Edge Pro, la ressource Azure et le compte de stockage cible vers lequel vous transférez des données n’ont pas besoin de se trouver dans la même région.

- **Disponibilité de la ressource** : pour cette préversion, la ressource est disponible dans les régions USA Est, Europe Ouest et Asie Sud-Est.

- **Disponibilité de l’appareil** : pour obtenir la liste de tous les pays/régions où l’appareil Azure Stack Edge Pro est disponible, accédez à la section **Disponibilité** sous l’onglet **Azure Stack Edge Pro** pour la [tarification d’Azure Stack Edge Pro](https://azure.microsoft.com/pricing/details/azure-stack/edge/#azureStackEdgePro).
    
- **Comptes de stockage de destination** : les comptes de stockage qui stockent les données sont disponibles dans toutes les régions Azure. Pour garantir des performances optimales, les régions où des comptes de stockage stockent les données Azure Stack Edge Pro doivent se trouver à proximité de l’appareil. Un compte de stockage situé dans un emplacement éloigné entraîne des temps de latence longs et une baisse des performances.

## <a name="next-steps"></a>Étapes suivantes

- Relire les [Conditions requises pour le système Azure Stack Edge Pro](azure-stack-edge-gpu-system-requirements.md).

- Comprendre les [limites d’Azure Stack Edge Pro](azure-stack-edge-limits.md).
- Déployer [Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md) sur le portail Azure.
