---
title: Surveiller Azure Site Recovery | Microsoft Docs
description: Surveiller et résoudre les opérations et les problèmes de réplication d’Azure Site Recovery à l’aide du portail
author: raynew
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/30/2019
ms.author: raynew
ms.openlocfilehash: d441284b265ab11dd5ece42ec3737e455d662435
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023565"
---
# <a name="monitor-site-recovery"></a>Superviser Site Recovery

Cet article explique comment surveiller [Azure Site Recovery](site-recovery-overview.md) à l’aide de la surveillance intégrée de Site Recovery.  Vous pouvez surveiller les aspects suivants :

- Intégrité et état des machines répliquées par Site Recovery
- État de test de basculement des machines
- Problèmes et erreurs affectant la configuration et la réplication
- Composants d’infrastructure tels que des serveurs locaux


## <a name="before-you-start"></a>Avant de commencer

Avant de commencer, vous pouvez consulter les [questions courantes concernant la surveillance](monitoring-common-questions.md).

## <a name="monitor-in-the-dashboard"></a>Surveiller sur le tableau de bord

1. Dans le coffre, cliquez sur **Vue d’ensemble**. Le tableau de bord Recovery Services regroupe toutes les informations de surveillance pour le coffre dans un emplacement unique. Il comprend des pages pour les services Site Recovery et Sauvegarde Azure, entre lesquels vous pouvez basculer.

    ![Tableau de bord Site Recovery](./media/site-recovery-monitor-and-troubleshoot/dashboard.png)

2. À partir du tableau de bord, faites défiler vers le bas dans différentes zones. 

    ![Capture d’écran montrant les zones du tableau de bord où vous pouvez descendre dans la hiérarchie.](./media/site-recovery-monitor-and-troubleshoot/site-recovery-overview-page.png).

3. Dans **Éléments répliqués**, cliquez sur **Afficher tout** pour voir tous les serveurs dans le coffre.
4. Cliquez sur les informations d’état de chaque section pour faire défiler vers le bas.
5. Sous **Affichage de l’infrastructure**, triez les informations de surveillance par type de machines que vous répliquez.

## <a name="monitor-replicated-items"></a>Surveiller les éléments répliqués

Dans **Éléments répliqués**, surveillez l’intégrité de toutes les machines dans le coffre pour lesquelles la réplication est activée.

**State** | **Détails**
--- | ---
Healthy | La réplication s’exécute normalement. Aucun symptôme d’erreur ou d’avertissement n’est détecté.
Avertissement | Un ou plusieurs symptômes d’avertissement qui peuvent avoir un impact sur la réplication sont détectés.
Critique | Un ou plusieurs symptômes d’erreur de réplication critique ont été détectés.<br/><br/> Ces erreurs indiquent que la réplication est soit bloquée, soit évolue moins vite que le rythme de modification des données.
Non applicable | Serveurs ne devant pas être répliqués pour le moment. Cela peut inclure des machines qui ont été basculées.

## <a name="monitor-test-failovers"></a>Surveiller les tests de basculement

Dans **Réussite du test de basculement**, surveillez l’état du basculement des ordinateurs figurant dans le coffre.

- Nous vous recommandons d’exécuter un test de basculement sur les machines répliquées au moins une fois tous les six mois. Cela vous permet de vérifier que le basculement fonctionne comme prévu, sans interrompre votre environnement de production. 
- Un test de basculement est considéré comme réussi uniquement une fois que le basculement et le nettoyage après basculement sont terminés.

**State** | **Détails**
--- | ---
Test recommandé | Machines qui n’ont pas fait l’objet d’un test de basculement depuis l’activation de la protection.
Effectué | Machines avec un ou plusieurs tests de basculement réussis.
Non applicable | Machines actuellement non éligibles à un test de basculement. Par exemple, pour les machines ayant basculé, une réplication initiale/un test de basculement/un basculement est en cours.

## <a name="monitor-configuration-issues"></a>Surveiller les problèmes de configuration

Dans **Problèmes de configuration**, surveillez tous les problèmes susceptibles d’avoir une incidence sur votre capacité à opérer correctement le basculement.

- Les problèmes de configuration (autres que la disponibilité des mises à jour logicielles) sont détectés par une validation périodique qui s’exécute toutes les 12 heures par défaut. Vous pouvez forcer l’exécution immédiate de l’opération de validation en cliquant sur l’icône d’actualisation en regard de l’intitulé de la section **Problèmes de configuration**.
- Cliquez sur les liens pour obtenir plus d’informations. Pour les problèmes affectant des machines spécifiques, dans la colonne **Configurations cibles**, cliquez sur **Doit être surveillé**. Les détails incluent des recommandations de correction.

**State** | **Détails**
--- | ---
Configurations manquantes | Un paramètre nécessaire est manquant, par exemple un réseau de récupération ou un groupe de ressources.
Ressources manquantes | Une ressource spécifiée est introuvable ou n’est pas disponible dans l’abonnement. Par exemple, la ressource a été supprimée ou migrée. Les ressources surveillées incluent le groupe de ressources cible, le sous-réseau/réseau virtuel cible, le compte de stockage cible/des journaux, le groupe à haute disponibilité cible et l’adresse IP cible.
Quota d’abonnement |  Le quota de ressources disponibles dans l’abonnement est comparé au solde requis pour faire basculer toutes les machines dans le coffre.<br/><br/> Si les ressources sont insuffisantes, un solde de quota insuffisant est signalé.<br/><br/> Pour les quotas, le système surveille le nombre de cœurs de machine virtuelle, le nombre de cœurs de famille de machines virtuelles et le nombre de cartes d’interface réseau.
Mises à jour logicielles | La disponibilité des nouvelles mises à jour logicielles et des informations sur les versions logicielles arrivant à expiration.


## <a name="monitor-errors"></a>Surveiller les erreurs

Dans **Résumé des erreurs**, surveillez les symptômes d’erreurs actives susceptibles d’avoir une incidence sur la réplication des serveurs dans le coffre, ainsi que le nombre de machines concernées.

- Les erreurs ayant une incidence sur les composants de l’infrastructure locale sont présentées au début de cette section. Par exemple, la non-réception d’une pulsation en provenance du fournisseur Azure Site Recovery sur le serveur de configuration local, ou l’hôte Hyper-V.
- Les erreurs de réplication ayant un impact sur les serveurs répliqués apparaissent ensuite.
- Le tableau est trié par ordre décroissant de la gravité des erreurs, puis par ordre décroissant du nombre de machines concernées.
- Le nombre de serveurs concernés est une information qui permet de comprendre si un même problème sous-jacent peut avoir une incidence sur plusieurs machines. Par exemple, un problème réseau peut avoir une incidence sur toutes les machines répliquant sur Azure. 
- Plusieurs erreurs de réplication peuvent se produire sur un seul serveur. Dans ce cas, chaque erreur inclut ce serveur dans la liste de ses serveurs affectés. Une fois le problème est résolu, les paramètres de réplication sont optimisés et l’erreur est résolue à partir de la machine.

## <a name="monitor-the-infrastructure"></a>Surveiller l’infrastructure

Dans **Affichage de l’infrastructure**, surveillez les composants d’infrastructure impliqués dans la réplication, ainsi que l’intégrité de la connectivité entre les serveurs et les services Azure.

- Une ligne verte indique que la connexion est opérationnelle.
- Une ligne rouge sur laquelle est superposée une icône d’erreur indique l’existence d’une ou de plusieurs erreurs qui affectent la connectivité.
-  Placez le pointeur de la souris sur l’icône d’erreur pour afficher l’erreur et le nombre d’entités concernées. Cliquez sur l’icône pour afficher une liste filtrée des entités concernées.

    ![Vue d’infrastructure de Site Recovery (coffre)](./media/site-recovery-monitor-and-troubleshoot/site-recovery-vault-infra-view.png)

### <a name="tips-for-monitoring-the-infrastructure"></a>Conseils relatifs à la surveillance de l’infrastructure

- Assurez-vous que les composants d’infrastructure locaux (serveur de configuration, serveurs de traitement, serveurs VMM, hôtes Hyper-V, machines virtuelles VMware) exécutent les dernières versions du fournisseur et/ou des agents Site Recovery.
- Pour utiliser toutes les fonctionnalités de la vue d’infrastructure, vous devez exécuter le [Correctif cumulatif 22](https://support.microsoft.com/help/4072852) pour ces composants.
- Pour utiliser la vue d’infrastructure, sélectionnez le scénario de réplication approprié dans votre environnement. Vous pouvez effectuer un zoom avant pour obtenir plus d’informations. Le tableau suivant affiche les scénarios qui sont représentés.

    **Scénario** | **State**  | **Vue disponible ?**
    --- |--- | ---
    **Réplication entre des sites locaux** | Tous les états | Non 
    **Réplication de machines virtuelles Azure entre les régions Azure**  | Réplication activée/réplication initiale en cours | Oui
    **Réplication de machines virtuelles Azure entre les régions Azure** | Basculement/restauration automatique | Non   
    **Réplication VMware vers Azure** | Réplication activée/réplication initiale en cours | Oui     
    **Réplication VMware vers Azure** | Basculement/restauration automatique | Non      
    **Réplication Hyper-V vers Azure** | Basculement/restauration automatique | Non

- Pour afficher la vue d’infrastructure pour une seule machine en cours de réplication, dans le menu du coffre, cliquez sur **Éléments répliqués**, puis sélectionnez un serveur.  




## <a name="monitor-recovery-plans"></a>Surveiller les plans de récupération

Dans **Plans de récupération**, surveillez le nombre de plans, créer des plans et modifiez des plans existants.  

## <a name="monitor-jobs"></a>Surveiller des travaux

La section **Travaux** affiche l’état des opérations Site Recovery.

- La plupart des opérations dans Azure Site Recovery s’exécutent de façon asynchrone, avec la création d’un travail de suivi qui est utilisé pour suivre l’état d’avancement de chaque opération. 
- L’objet de travail dispose de toutes les informations nécessaires pour suivre l’état et la progression des opérations. 

Pour surveiller les travaux, procédez comme suit :

1. Dans le tableau de bord > section **Travaux**, vous pouvez voir un résumé des travaux terminés, en cours d’exécution ou en attente pour l’entrée au cours des dernières 24 heures. Vous pouvez cliquer sur n’importe quel état pour obtenir plus d’informations sur les travaux correspondants.
2. Cliquez sur **Afficher tout** pour afficher tous les travaux des dernières 24 heures.

    > [!NOTE]
    > Vous pouvez également accéder aux informations sur les travaux à partir du menu du coffre > **Travaux Site Recovery**. 

2. La liste **Travaux Site Recovery** affiche tous les travaux. Dans le menu supérieur, vous pouvez afficher les détails d’erreur pour des travaux spécifiques, filtrer la liste des travaux selon des critères spécifiques et exporter les détails d’un travail sélectionné vers Excel.
3. Cliquez sur un travail pour en afficher les détails. 

## <a name="monitor-virtual-machines"></a>Surveillance des machines virtuelles

Dans **Éléments répliqués**, obtenez la liste des machines répliquées. 
    ![Vue de la liste des éléments répliqués dans Azure Site Recovery](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-list-view.png)

2. Vous pouvez afficher et filtrer les informations. Dans le menu d’action supérieur, vous pouvez effectuer des actions pour une machine donnée, y compris exécuter un test de basculement ou afficher des erreurs spécifiques.
3. Cliquez sur **Colonnes** pour afficher des colonnes supplémentaires, par exemple pour afficher le RPO, les problèmes de configuration cible et les erreurs de réplication.
4. Cliquez sur **Filtre** pour afficher les informations en fonction de paramètres spécifiques telles que l’intégrité de la réplication, ou une stratégie de réplication particulière.
5. Cliquez avec le bouton droit sur une machine pour y lancer des opérations telles qu’un test de basculement, ou pour afficher les détails des erreurs qui lui sont associées.
6. Cliquez sur une machine pour en afficher les détails. Les détails sont les suivants :
   - **Informations de réplication** : état et intégrité actuels de la machine.
   - **RPO** (objectif de point de récupération) : RPO actuel de la machine virtuelle et heure du dernier calcul du RPO.
   - **Point de récupération** : derniers points de récupération disponibles pour la machine.
   - **Disponibilité du basculement** : indique si un test de basculement a été exécuté pour la machine, la version de l’agent en cours d’exécution sur la machine (pour les machines exécutant le service Mobilité) et les éventuels problèmes de configuration.
   - **Erreurs** : liste des symptômes d’erreur de réplication actuellement observés sur la machine, ainsi que les causes/actions possibles.
   - **Événements** : liste chronologique des événements récents ayant un impact sur la machine. La colonne Détails de l’erreur indique les erreurs actuellement observables sur la machine, tandis que la colonne Événements est un enregistrement historique des problèmes qui ont eu un impact sur la machine.
   - **Vue d’infrastructure** : affiche l’état de l’infrastructure pour le scénario de réplication des machines vers Azure.

     ![Vue d’ensemble/détails des éléments répliqués d’Azure Site Recovery](./media/site-recovery-monitor-and-troubleshoot/site-recovery-virtual-machine-details.png)

## <a name="subscribe-to-email-notifications"></a>S’abonner aux notifications par courrier électronique

Vous pouvez vous abonner pour recevoir des notifications par courrier électronique pour ces événements critiques :
 
- État critique de la machine répliquée.
- Aucune connectivité entre les composants d’infrastructure locaux et le service Site Recovery. La connectivité entre Site Recovery et les serveurs locaux enregistrés dans un coffre est détectée à l’aide d’un mécanisme de pulsation.
- Échecs de basculement.

Pour vous abonner, procédez comme suit :

Dans le coffre > section **Supervision**, cliquez sur **Événements Site Recovery**.
1. Cliquez sur **Notifications par e-mail**.
1. Sous **Notifications par e-mail**, activez les notifications et spécifiez le destinataire des notifications. Vous pouvez choisir d’envoyer les notifications à tous les administrateurs de l’abonnement ou à des adresses e-mail spécifiques.

    ![Notifications par e-mail](./media/site-recovery-monitor-and-troubleshoot/email.png)

## <a name="next-steps"></a>Étapes suivantes

[Découvrez](monitor-log-analytics.md) la surveillance de Site Recovery avec Azure Monitor.
