---
title: Présentation d’Azure Stack Edge Mini R | Microsoft Docs
description: Décrit Azure Stack Edge Mini R, solution de stockage destinée aux applications militaires qui utilise un appareil physique portable avec une batterie à des fins de transfert Wi-Fi vers Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 09/23/2020
ms.author: alkohli
ms.openlocfilehash: 23ec79b7e3989c9749edf98d930ad8305324c9da
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454007"
---
# <a name="what-is-the-azure-stack-edge-mini-r"></a>Qu’est-ce qu’Azure Stack Edge Mini R ?

Azure Stack Edge Mini R est un appareil informatique en périphérie ultra portable et robuste, conçu pour une utilisation dans des environnements difficiles. Azure Stack Edge Mini R est fourni en tant que solution de matériel en tant que service. Microsoft fournit un appareil managé dans le cloud qui fait office de passerelle de stockage réseau et intègre une VPU (Vision Processing Unit) permettant une inférence accélérée de l’IA.

Cet article vous fournit une vue d’ensemble de la solution Azure Stack Edge Mini R, ses principales fonctionnalités et les scénarios dans lesquels vous pouvez déployer cet appareil.


## <a name="key-capabilities"></a>Fonctionnalités clés

Azure Stack Edge Mini R offre les fonctionnalités suivantes :

|Fonctionnalité |Description  |
|---------|---------|
|Matériel robuste| Matériel robuste conçu pour les environnements difficiles.|
|Ultra portable| Facteur de forme ultra portable, fonctionnant sur batterie.|
|Géré par le cloud|L’appareil et le service sont gérés via le portail Azure.|
|Charges de travail Edge Computing|Permet l’analyse, le traitement et le filtrage des données.<br>Prend en charge les machines virtuelles et les charges de travail conteneurisées. |
|Inférence accélérée avec l’intelligence artificielle| Activé par Intel Movidius Myriad X VPU.|
|Avec et sans fil | Autorise les transferts de données avec et sans fil.|
|Accès aux données     | Accès direct aux données à partir d’objets blob Stockage Azure et Azure Files à l’aide des API cloud pour le traitement d’autres données dans le cloud. Un cache local sur l’appareil est utilisé pour accéder rapidement aux fichiers les plus récemment utilisés.|
|Mode déconnecté|  L’appareil et le service peuvent être gérés par le biais d’Azure Stack Hub. Déployez, exécutez et gérez des applications en mode hors connexion. <br> Un mode déconnecté prend en charge les scénarios de chargement hors connexion.|
|Protocoles de transfert de fichiers pris en charge      |Prend en charge les protocoles SMB, NFS et REST standard pour l’ingestion des données. <br> Pour plus d’informations sur les versions prises en charge, consultez [Conditions requises pour Azure Stack Edge Mini R](azure-stack-edge-gpu-system-requirements.md).|
|Actualisation des données     | Possibilité d’actualiser les fichiers locaux avec la dernière version issue du cloud.|
|Double chiffrement    | L’utilisation d’un lecteur à chiffrement automatique fournit la première couche de chiffrement. Le VPN fournit la deuxième couche de chiffrement. Prise en charge de BitLocker pour chiffrer les données localement et transférer de façon sécurisée des données vers le cloud via *https*.|
|Limitation de bande passante| Limiter l’utilisation de la bande passante pendant les heures de pointe.|

## <a name="use-cases"></a>Cas d'utilisation

Voici différents scénarios d’utilisation d’Azure Stack Edge Mini R pour l’inférence rapide de Machine Learning (ML) à la périphérie et le prétraitement des données avant leur envoi à Azure.

- **Inférence avec Azure Machine Learning** : Azure Stack Edge Mini R vous permet d’exécuter des modèles Machine Learning (ML) pour obtenir des résultats rapides qui peuvent être traités avant l’envoi des données vers le cloud. Le jeu de données complet peut également être transféré pour continuer à reformer et améliorer vos modèles ML. Pour plus d’informations sur l’utilisation des modèles accélérés matériellement d’Azure ML sur un appareil Azure Stack Edge Mini R, consultez [Déployer des modèles accélérés matériellement Azure ML sur Azure Stack Edge Mini R](../machine-learning/how-to-deploy-fpga-web-service.md#deploy-to-a-local-edge-server).

- **Prétraiter les données** : transformez les données via des options de calcul telles que des conteneurs ou des machines virtuelles avant de les envoyer à Azure pour créer un jeu de données plus actionnable. Le prétraitement peut servir à :

    - Agréger les données.
    - Modifiez les données, par exemple, pour supprimer des données personnelles.
    - Fractionnez les données pour optimiser le stockage et la bande passante, ainsi que pour les analyser de façon plus approfondie.
    - Analyser et réagir aux événements IoT.

- **Transférer des données vers Azure via le réseau** : utilisez Azure Stack Edge Mini R pour transférer rapidement et facilement des données vers Azure pour le calcul et l’analyse à des fins d’archivage.

## <a name="components"></a>Composants

La solution Azure Stack Edge Mini R se compose d’une ressource Azure Stack Edge, d’un appareil physique Azure Stack Edge robuste et ultra portable ainsi que d’une interface utilisateur web locale.

* **Appareil physique Azure Stack Edge Mini R** : appareil ultra portable, robuste, de calcul et de stockage fourni par Microsoft. L’appareil dispose d’une batterie intégrée et pèse moins de 3 kg.

    ![Appareil Azure Stack Edge Mini R](media/azure-stack-edge-k-series-overview/perspective-view-1.png)

* **Ressource Azure Stack Edge** : ressource du portail Azure, qui vous permet de gérer un appareil robuste Azure Stack Edge Mini R à partir d’une interface web accessible depuis divers emplacements géographiques. Utilisez la ressource Azure Stack Edge pour créer et gérer des ressources, afficher et gérer les appareils et les alertes, et gérer les partages.  

* **Interface utilisateur web locale Azure Stack Edge Mini R** : une interface utilisateur locale basée sur un navigateur sur votre appareil Azure Stack Edge Mini R, principalement conçue pour la configuration initiale de l’appareil. Utilisez également l’interface utilisateur web locale pour exécuter des diagnostics, arrêter et redémarrer l’appareil Azure Stack Edge Pro, afficher les journaux d’activité de copie, et contactez le support Microsoft pour créer une demande de service.


## <a name="region-availability"></a>Disponibilité des régions

L’appareil physique Azure Stack Edge Mini R, la ressource Azure et le compte de stockage cible vers lequel vous transférez des données n’ont pas besoin de se trouver dans la même région.

- **Disponibilité des ressources** : pour obtenir la liste de toutes les régions où la ressource Azure Stack Edge est disponible, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). 

- **Disponibilité de l’appareil** : pour obtenir la liste de tous les pays où l’appareil Azure Stack Edge Mini R est disponible, accédez à la section Disponibilité sous l’onglet Azure Stack Edge Mini R, qui indique les [tarifs d’Azure Stack Edge Mini R](https://azure.microsoft.com/pricing/details/azure-stack/edge/#azureStackEdgeMiniR).

- **Comptes de stockage de destination** : les comptes de stockage qui stockent les données sont disponibles dans toutes les régions Azure. Pour garantir des performances optimales, les régions dans lesquelles des comptes de stockage stockent les données Azure Stack Edge Mini R doivent se trouver à proximité de l’appareil. Un compte de stockage situé dans un emplacement éloigné entraîne des temps de latence longs et une baisse des performances.


## <a name="next-steps"></a>Étapes suivantes

- Relire les [Conditions requises pour le système Azure Stack Edge Mini R](azure-stack-edge-gpu-system-requirements.md).