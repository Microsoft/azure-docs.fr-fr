---
title: Sauvegarder une batterie de serveurs SharePoint dans Azure avec DPM
description: Cet article fournit une vue d’ensemble de la protection du serveur DPM/Azure Backup Server d’une batterie de serveurs SharePoint sur Azure
ms.topic: conceptual
ms.date: 03/09/2020
ms.openlocfilehash: 7661d64e487c8b8badca240852d17bcf736ba8cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91254429"
---
# <a name="back-up-a-sharepoint-farm-to-azure-with-dpm"></a>Sauvegarder une batterie de serveurs SharePoint dans Azure avec DPM

Vous sauvegardez une batterie de serveurs SharePoint sur Microsoft Azure à l’aide de System Center Data Protection Manager (DPM) de la même façon que vous sauvegardez d’autres sources de données. Azure Backup offre une flexibilité dans la planification d’une sauvegarde pour créer des points de sauvegarde quotidiens, hebdomadaires, mensuels ou annuels, et vous offre des options de stratégie de conservation pour les différents points de sauvegarde. DPM vous permet de stocker des copies sur disque local pour bénéficier d’objectifs de temps de récupération (RTO) rapides, ainsi que de stocker des copies sur Azure pour une conservation à long terme économique.

La sauvegarde de SharePoint dans Azure avec DPM est un processus très similaire à la sauvegarde locale de SharePoint dans DPM. Cet article comprend des points particuliers à prendre en compte pour Azure.

## <a name="sharepoint-supported-versions-and-related-protection-scenarios"></a>Versions SharePoint prises en charge et scénarios de protection associés

Pour obtenir la liste des versions de SharePoint prises en charge et des versions de DPM nécessaires pour les sauvegarder, consultez les [applications que peut sauvegarder DPM](/system-center/dpm/dpm-protection-matrix#applications-backup)

## <a name="before-you-start"></a>Avant de commencer

Quelques points doivent être confirmés avant de sauvegarder une batterie de serveurs SharePoint sur Azure.

### <a name="prerequisites"></a>Prérequis

Avant de continuer, vérifiez que toutes les [conditions préalables](backup-azure-dpm-introduction.md#prerequisites-and-limitations) à l’utilisation de Microsoft Azure Backup pour protéger les charges de travail ont été remplies. Certaines tâches liées aux conditions préalables incluent : créer un coffre de sauvegarde, télécharger les informations d’identification du coffre, installer l’Agent Azure Backup et inscrire le serveur DPM/Azure Backup Server auprès du coffre.

D’autres prérequis et limitations sont disponibles dans l’article [Sauvegarder SharePoint avec DPM](/system-center/dpm/back-up-sharepoint#prerequisites-and-limitations).

## <a name="configure-backup"></a>Configurer une sauvegarde

Pour sauvegarder la batterie de serveurs SharePoint, vous configurez la protection pour SharePoint à l’aide de ConfigureSharePoint.exe, puis créez un groupe de protection dans DPM. Pour obtenir des instructions, consultez [Configurer la sauvegarde](/system-center/dpm/back-up-sharepoint#configure-backup) dans la documentation DPM.

## <a name="monitoring"></a>Surveillance

Pour superviser le travail de sauvegarde, suivez les instructions de [Supervision de la sauvegarde DPM](/system-center/dpm/back-up-sharepoint#monitoring)

## <a name="restore-sharepoint-data"></a>Restaurer des données SharePoint

Pour savoir comment restaurer un élément SharePoint à partir d’un disque avec DPM, consultez [Restaurer des données SharePoint](/system-center/dpm/back-up-sharepoint#restore-sharepoint-data).

## <a name="restore-a-sharepoint-database-from-azure-by-using-dpm"></a>Restauration d’une base de données SharePoint à partir d’Azure à l'aide de DPM

1. Pour récupérer une base de données de contenu SharePoint, parcourez les différents points de récupération (comme indiqué ci-dessus), puis sélectionnez le point de récupération à restaurer.

    ![DPM SharePoint Protection8](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection9.png)
2. Double-cliquez sur le point de récupération SharePoint pour afficher les informations du catalogue SharePoint disponibles.

   > [!NOTE]
   > Comme la batterie de serveurs SharePoint est protégée pour une rétention à long terme dans Azure, aucune information du catalogue (métadonnées) n’est disponible sur le serveur DPM. Par conséquent, chaque fois qu’une base de données contenu SharePoint ponctuelle doit être récupérée, vous devez de nouveau cataloguer la batterie de serveurs SharePoint.
   >
   >
3. Sélectionnez **Recataloguer**.

    ![DPM SharePoint Protection10](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection12.png)

    La fenêtre d’état **Recataloguer le cloud** s’ouvre.

    ![DPM SharePoint Protection11](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection13.png)

    Une fois le catalogage terminé, l’état passe à *Succès*. Sélectionnez **Fermer**.

    ![DPM SharePoint Protection12](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection14.png)
4. Sélectionnez l’objet SharePoint affiché dans l’onglet **Récupération** DPM pour obtenir la structure de base de données de contenu. Cliquez avec le bouton droit sur l’élément, puis sélectionnez **Récupérer**.

    ![DPM SharePoint Protection13](./media/backup-azure-backup-sharepoint/dpm-sharepoint-protection15.png)
5. À ce stade, suivez les étapes de récupération mentionnées précédemment dans cet article pour la récupération d’une base de données de contenu SharePoint à partir du disque.

## <a name="switching-the-front-end-web-server"></a>Changement de serveur web front-end

Si vous avez plusieurs serveurs web front-end et que vous voulez changer le serveur que DPM utilise pour protéger la batterie de serveurs, suivez les instructions de [Changement de serveur web front-end](/system-center/dpm/back-up-sharepoint#switching-the-front-end-web-server).

## <a name="next-steps"></a>Étapes suivantes

* [Serveur de sauvegarde Azure et DPM - FAQ](backup-azure-dpm-azure-server-faq.md)
* [Résoudre les problèmes liés à System Center Data Protection Manager](backup-azure-scdpm-troubleshooting.md)
