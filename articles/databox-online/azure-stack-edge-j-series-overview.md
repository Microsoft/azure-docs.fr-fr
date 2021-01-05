---
title: Présentation de Microsoft Azure Stack Edge Pro R | Microsoft Docs
description: Décrit les appareils Azure Stack Edge Pro R, solution de stockage destinée aux applications militaires qui utilise un appareil physique pour le transfert vers Azure via le réseau.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/23/2020
ms.author: alkohli
ms.openlocfilehash: d95c4bb0345051cc79dadca5fa7cc11932a9512a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454044"
---
# <a name="what-is-the-azure-stack-edge-pro-r"></a>Qu’est-ce qu’Azure Stack Edge Pro R ?

Azure Stack Edge Pro R est un appareil informatique en périphérie robuste conçu pour une utilisation dans des environnements difficiles. Azure Stack Edge Pro R est fourni en tant que solution de matériel en tant que service. Microsoft fournit un appareil managé dans le cloud qui fait office de passerelle de stockage réseau et intègre un processeur graphique (GPU) permettant une inférence accélérée de l’IA.

Cet article vous fournit une vue d’ensemble de la solution Azure Stack Edge Pro R, ses principales fonctionnalités et les scénarios dans lesquels vous pouvez déployer cet appareil.


## <a name="key-capabilities"></a>Fonctionnalités clés

Azure Stack Edge Pro R intègre les fonctionnalités suivantes :

|Fonctionnalité |Description  |
|---------|---------|
|Matériel robuste| Matériel de classe serveur robuste conçu pour les environnements exigeants. Appareil contenu dans une mallette de transport portable. |
|Géré par le cloud     |L’appareil et le service sont gérés via le portail Azure.|
|Charges de travail Edge Computing   |Permet l’analyse, le traitement et le filtrage des données. Prend en charge les machines virtuelles et les charges de travail conteneurisées.|
|Inférence accélérée avec l’intelligence artificielle| Activé par GPU Nvidia T4.|
|Accès aux données     | Accès direct aux données à partir d’objets blob Stockage Azure et Azure Files à l’aide des API cloud pour le traitement d’autres données dans le cloud. Un cache local sur l’appareil est utilisé pour accéder rapidement aux fichiers les plus récemment utilisés.|
|Mode déconnecté| L’appareil et le service peuvent être gérés par le biais d’Azure Stack Hub. Déployez, exécutez et gérez des applications en mode hors connexion. <br> Un mode déconnecté prend en charge les scénarios de chargement hors connexion.|
|Protocoles de transfert de fichiers pris en charge     |Prise en charge des protocoles SMB, NFS et REST standard pour l’ingestion des données. <br> Pour plus d’informations sur les versions prises en charge, consultez [Conditions requises pour Azure Stack Edge Pro R](azure-stack-edge-gpu-system-requirements.md).|
|Actualisation des données     | Possibilité d’actualiser les fichiers locaux avec la dernière version issue du cloud.|
|Double chiffrement    | L’utilisation de lecteurs à chiffrement automatique fournit la première couche de chiffrement. Le VPN fournit la deuxième couche de chiffrement. Prise en charge de BitLocker pour chiffrer les données localement et transférer de façon sécurisée des données vers le cloud via *https*.|
|Limitation de bande passante| Limiter l’utilisation de la bande passante pendant les heures de pointe.|

<!--|Scale out file server| Available as 1-node and 4-node cluster configurations|-->

## <a name="use-cases"></a>Cas d'utilisation

Voici différents scénarios d’utilisation d’Azure Stack Edge Pro R pour l’inférence rapide de Machine Learning (ML) à la périphérie et le prétraitement des données avant leur envoi à Azure.

- **Inférence avec Azure Machine Learning** : Azure Stack Edge Pro R vous permet d’exécuter des modèles Machine Learning (ML) pour obtenir des résultats rapides qui peuvent être traités avant l’envoi des données vers le cloud. Le jeu de données complet peut également être transféré pour continuer à reformer et améliorer vos modèles ML. Pour plus d’informations sur l’utilisation des modèles accélérés matériellement d’Azure ML sur un appareil Azure Stack Edge Pro R, consultez [Déployer des modèles accélérés matériellement Azure ML sur Azure Stack Edge Pro R](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Prétraiter les données** : transformez les données avant de les envoyer à Azure pour créer un jeu de données plus actionnable. Le prétraitement peut servir à :

    - Agréger les données.
    - Modifiez les données, par exemple, pour supprimer des données personnelles.
    - Fractionnez les données pour optimiser le stockage et la bande passante, ainsi que pour les analyser de façon plus approfondie.
    - Analyser et réagir aux événements IoT.

- **Transférer des données vers Azure via le réseau** : utilisez Azure Stack Edge Pro R pour transférer rapidement et facilement des données vers Azure pour le calcul et l’analyse à des fins d’archivage.

## <a name="components"></a>Composants

La solution Azure Stack Edge Pro R se compose d’une ressource Azure Stack Edge, d’un appareil physique Azure Stack Edge Pro R robuste et d’une interface utilisateur web locale.

- **Appareil physique Azure Stack Edge Pro R** : appareil de calcul et de stockage à un nœud contenu dans une mallette de transport robuste. Un onduleur facultatif est également disponible.

    ![Appareil Azure Stack Edge Pro R à un nœud](media/azure-stack-edge-j-series-overview/device-image-1.png)

- **Ressource Azure Stack Edge** : ressource du portail Azure, qui vous permet de gérer un appareil robuste Azure Stack Edge Pro R à partir d’une interface web accessible depuis divers emplacements géographiques. Utilisez la ressource Azure Stack Edge pour créer et gérer des ressources, afficher et gérer les appareils et les alertes, et gérer les partages.  

- **Interface utilisateur web locale Azure Stack Edge Pro R** : interface utilisateur locale basée sur un navigateur sur votre appareil Azure Stack Edge Pro R, principalement conçue pour la configuration initiale de l’appareil. Utilisez également l’interface utilisateur web locale pour exécuter des diagnostics, arrêter et redémarrer l’appareil Azure Stack Edge Pro, afficher les journaux d’activité de copie, et contactez le support Microsoft pour créer une demande de service.


## <a name="region-availability"></a>Disponibilité des régions

L’appareil physique Azure Stack Edge Pro R, la ressource Azure et le compte de stockage cible vers lequel vous transférez des données n’ont pas besoin de se trouver dans la même région.

- **Disponibilité des ressources** : pour obtenir la liste de toutes les régions où la ressource Azure Stack Edge est disponible, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). 

- **Disponibilité de l’appareil** : pour obtenir la liste de tous les pays où l’appareil Azure Stack Edge Pro R est disponible, accédez à la section **Disponibilité** sous l’onglet **Azure Stack Edge Pro R**, qui indique les [tarifs d’Azure Stack Edge Pro R](https://azure.microsoft.com/pricing/details/azure-stack/edge/#azureStackEdgeProR).

- **Comptes de stockage de destination** : les comptes de stockage qui stockent les données sont disponibles dans toutes les régions Azure. Pour garantir des performances optimales, les régions où des comptes de stockage stockent les données Azure Stack Edge Pro R doivent se trouver à proximité de l’appareil. Un compte de stockage situé dans un emplacement éloigné entraîne des temps de latence longs et une baisse des performances.

## <a name="next-steps"></a>Étapes suivantes

- Relire les [Conditions requises pour le système Azure Stack Edge Pro R](azure-stack-edge-gpu-system-requirements.md).
<!--- Understand the [Azure Stack Edge Pro R limits](azure-stack-edge-limits.md).-->