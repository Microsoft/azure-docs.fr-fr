---
title: Mettre à niveau l’agent Microsoft Azure Recovery Services (MARS)
description: Découvrez comment mettre à niveau l’agent Microsoft Azure Recovery Services (MARS).
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: a1ee26db962781643e9599069282647658301bac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89181474"
---
# <a name="upgrade-the-microsoft-azure-recovery-services-mars-agent"></a>Mettre à niveau l’agent Microsoft Azure Recovery Services (MARS)

Cet article porte sur les points suivants :

* Identifier les serveurs ayant des versions antérieures de l’agent MARS
* Mettre à jour les installations MARS sur ces serveurs

## <a name="identify-servers-with-earlier-versions-of-the-mars-agent"></a>Identifier les serveurs ayant des versions antérieures de l’agent MARS

Pour les installations de l’agent Sauvegarde Azure et du serveur de sauvegarde Azure :

1. Accédez au coffre Recovery Services dans lequel vous avez inscrit des serveurs qui sont potentiellement sauvegardés par des versions antérieures de l’agent. Vous trouverez une liste représentative des coffres avec des agents de sauvegarde Azure plus anciens dans les alertes de mise à jour de la sauvegarde Azure à partir d’Azure.
1. Dans la section **Paramètres** à gauche du coffre Recovery Services, sélectionnez **Infrastructure de sauvegarde** sous la section **Gérer**.
1. Pour découvrir les agents Sauvegarde Azure installés dans le cadre des installations de serveur de sauvegarde Azure, accédez à **Serveurs de gestion de sauvegarde** sous **Serveurs d’administration**. Cette opération répertorie les serveurs qui ont des installations de serveur Sauvegarde Azure, ainsi que le numéro de version de l’agent de sauvegarde Azure associé.

    ![Liste des agents MARS installés dans le cadre des installations de serveur Sauvegarde Azure](./media/upgrade-mars-agent/backup-management-servers.png)

1. Pour vérifier les versions d’agent installées pour l’agent Microsoft Azure Recovery Services (MARS) ou l’agent de sauvegarde Azure, accédez à **Serveurs protégés** sous **Serveurs d’administration**. Sélectionnez ensuite **Agent de sauvegarde Azure** sous Type de gestion des sauvegardes. Cette opération répertorie les serveurs qui ont des installations de l’agent de sauvegarde Azure, ainsi que le numéro de version de l’installation.

    ![Liste des serveurs sur lesquels l’agent MARS est installé](./media/upgrade-mars-agent/protected-servers.png)

1. Triez la colonne des versions de l’agent Sauvegarde Azure en sélectionnant la colonne **Version de l’agent** pour les installations de l’agent MARS ou la colonne **Version de l’agent Sauvegarde Azure** pour les installations de serveur de sauvegarde Azure.

1. L’étape précédente vous donnera la liste des serveurs ayant des agents de sauvegarde Azure dont les versions sont inférieures à 2.0.9083.0 ou dont les versions d’agent sont répertoriées comme vides. Il s’agit des serveurs sur lesquels les agents de sauvegarde Azure doivent être mis à jour.

## <a name="update-the-mars-agent-installation-on-the-server"></a>Mettre à jour l’installation de l’agent MARS sur le serveur

Une fois que vous avez identifié les serveurs qui nécessitent une mise à jour de l’agent Sauvegarde Azure, procédez comme suit pour chaque serveur identifié (à l’aide du serveur de sauvegarde Azure ou de l’agent MARS). [Téléchargez la version la plus récente de l’agent Sauvegarde Azure](https://aka.ms/azurebackup_agent) avant de suivre les étapes ci-dessous.

1. Sélectionnez une ligne avec l’agent de sauvegarde Azure inférieur à 2.0.9083.0 ou vide. L’écran de détails du serveur s’ouvre.

    ![Serveurs protégés avec des versions obsolètes de l’agent](./media/upgrade-mars-agent/old-agent-version.png)

    ![Serveurs de sauvegarde Azure avec des versions obsolètes de l’agent](./media/upgrade-mars-agent/backup-management-servers-old-versions.png)

1. Sélectionnez **Se connecter** pour recevoir un fichier de connexion Bureau à distance pour vous connecter au serveur ou connectez-vous directement au serveur via la connexion Bureau à distance sur votre serveur.

    ![Se connecter au serveur via une connexion Bureau à distance](./media/upgrade-mars-agent/connect-to-server.png)

    >[!NOTE]
    > Si le serveur indiqué n’existe pas ou a été mis hors service, vous pouvez ignorer les étapes restantes ci-dessous et passer au serveur suivant.

1. Entrez vos informations de connexion d’administration et connectez-vous.

1. Si votre serveur ou le proxy de votre serveur dispose d’un accès limité à Internet, assurez-vous que les paramètres du pare-feu sur le serveur/proxy sont configurés de sorte à autoriser l’URL appropriée vers le cloud Azure que vous utilisez :

    Cloud Azure | URL
    --- | ---
    Cloud Azure (public) |   `https://login.windows.net`
    Cloud Azure China 21Vianet   | `https://login.chinacloudapi.cn`
    Cloud Azure US Government |   `https://login.microsoftonline.us`
    Cloud Azure – Allemagne  |  `https://login.microsoftonline.de`

1. Copiez le programme d’installation de la mise à jour de l’agent Sauvegarde Azure sur le serveur.

    ![Copier le programme d’installation de la mise à jour de l’agent Sauvegarde Azure sur le serveur](./media/upgrade-mars-agent/copy-agent-installer.png)

1. Exécutez le programme d’installation. L’Assistant Mise à niveau de l’agent Microsoft Azure Recovery Services s’ouvre.

    ![Assistant Mise à niveau de l’agent Microsoft Azure Recovery Services](./media/upgrade-mars-agent/agent-upgrade-wizard.png)

1. Sélectionnez **Suivant**.

1. Sélectionnez **Mettre à niveau**.

    ![Installation de l’agent Microsoft Azure Recovery Services](./media/upgrade-mars-agent/upgrade-installation.png)

1. L’écran de confirmation final indique que l’agent de sauvegarde Azure a été correctement mis à jour.

## <a name="for-system-center-data-protection-manager-sc-dpm-customers"></a>Pour les clients System Center Data Protection Manager (SC DPM)

Si vous avez installé les agents Sauvegarde Azure sur les serveurs System Center Data Protection Manager (SC DPM), vous devez suivre les étapes ci-dessous pour déterminer si vos serveurs DPM ont besoin d’une mise à jour de l’agent Sauvegarde Azure :

1. Connectez-vous à votre serveur SC DPM en tant qu’administrateur.
2. Ouvrez la console DPM.
3. Sélectionnez **Gestion** dans le volet de navigation inférieur gauche de la console.
4. Dans les informations qui s’affichent dans le volet de navigation gauche, recherchez les informations de version de l’agent de sauvegarde Azure.
5. Si la version est inférieure à 2.0.9083.0, téléchargez la version la plus récente du programme d’installation de l’agent Sauvegarde Azure et exécutez le programme d’installation sur le serveur DPM pour mettre à jour l’agent Sauvegarde Azure.

Répétez les étapes ci-dessus pour tous les serveurs DPM de votre environnement.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [sauvegarder des ordinateurs Windows à l’aide de l’agent MARS de Sauvegarde Azure](backup-windows-with-mars-agent.md)
