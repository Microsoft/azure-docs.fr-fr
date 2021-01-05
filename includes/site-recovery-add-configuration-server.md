---
title: Fichier Include
description: Fichier Include
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: include
ms.date: 02/28/2019
ms.author: mayg
ms.custom: include file
ms.openlocfilehash: f699ffe6d5a91e8ce3ae90c7e12249bbad0fff3e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001550"
---
1. Exécutez le fichier d’installation unifiée.
2. Dans **Avant de commencer**, sélectionnez **Installer le serveur de configuration et le serveur de traitement**.

    ![Capture de l’écran Avant de commencer de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz1.png)

3. Dans **Licence de logiciel tiers**, cliquez sur **J’accepte** pour télécharger et installer MySQL.

    ![Capture de l’écran Licence de logiciel tiers de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz2.png)
4. Dans **Inscription**, sélectionnez la clé d’inscription que vous avez téléchargée à partir du coffre.

    ![Capture de l’écran Inscription de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz3.png)
5. Dans **Paramètres Internet**, indiquez de quelle manière le fournisseur qui s’exécute sur le serveur de configuration doit se connecter à Azure Site Recovery par le biais d’Internet. Vérifiez que vous avez autorisé les URL requises.

    - Si vous voulez vous connecter avec le proxy actuellement configuré sur la machine, sélectionnez **Se connecter à Azure Site Recovery avec un serveur proxy**.
    - Si vous voulez que le fournisseur se connecte directement, sélectionnez **Se connecter directement à Azure Site Recovery sans serveur proxy**.
    - Si le proxy existant nécessite une authentification, ou si vous voulez utiliser un proxy personnalisé pour la connexion au fournisseur, sélectionnez **Se connecter avec des paramètres de proxy personnalisés**, et spécifiez l’adresse, le port et les informations d’authentification.
     ![Capture de l’écran Paramètres Internet de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz4.png)
6. Dans **Vérification de la configuration requise**, le programme d’installation procède à une vérification afin de garantir le bon déroulement de l’installation. Si un avertissement s’affiche à propos de la **vérification de la synchronisation globale**, vérifiez que l’heure de l’horloge système (paramètres **Date et heure**) est identique à celle du fuseau horaire.

    ![Capture de l’écran Vérification de la configuration requise de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz5.png)
7. Dans **Configuration MySQL**, créez des informations d’identification pour vous connecter à l’instance de serveur MySQL installée.

    ![Capture de l’écran Configuration MySQL de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz6.png)
8. Dans **Détails de l'environnement**, sélectionnez Non si vous répliquez des machines virtuelles ou des serveurs physiques Azure Stack. 
9. Dans **Emplacement d’installation**, sélectionnez l’emplacement où vous voulez installer les fichiers binaires et stocker le cache. Le lecteur sélectionné doit présenter au moins 5 Go d’espace disque disponible. Toutefois, nous vous recommandons d’utiliser un lecteur de cache présentant au moins 600 Go d’espace disponible.

    ![Capture de l’écran Emplacement d’installation de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz8.png)
10. Dans **Sélection du réseau**, commencez par sélectionner la carte réseau que le serveur de processus intégré utilise pour l’installation push du service Mobilité sur les machines sources, puis sélectionnez la carte réseau que le serveur de configuration utilise pour la connectivité avec Azure. Le port 9443 est le port utilisé par défaut pour envoyer et recevoir le trafic de réplication, mais vous pouvez le modifier en fonction des exigences de votre environnement. Outre le port 9443, nous ouvrons également le port 443, qui est utilisé par un serveur web pour orchestrer les opérations de réplication. N’utilisez pas le port 443 pour envoyer ou recevoir le trafic de réplication.

    ![Capture de l’écran Sélection du réseau de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz9.png)


11. Dans **Résumé**, passez en revue les informations, puis cliquez sur **Installer**. Une fois l’installation terminée, une phrase secrète est générée. Vous en aurez besoin lorsque vous activerez la réplication. Par conséquent, copiez-la et conservez-la en lieu sûr.

    ![Capture de l’écran Résumé de l’Assistant Installation unifiée](./media/site-recovery-add-configuration-server/combined-wiz10.png)

Une fois l’inscription terminée, le serveur s’affiche sur le panneau **Paramètres** > **Serveurs** du coffre.
