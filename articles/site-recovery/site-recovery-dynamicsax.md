---
title: Reprise d’activité de Dynamics AX avec Azure Site Recovery
description: Découvrez comment configurer la reprise d’activité pour Dynamics AX avec Azure Site Recovery.
author: sideeksh
manager: rochakm
ms.topic: how-to
ms.date: 11/27/2018
ms.openlocfilehash: dfa3c108d00aeba9c7d42e96e7a40736a087a508
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86133814"
---
# <a name="set-up-disaster-recovery-for-a-multitier-dynamics-ax-application"></a>Configurer la reprise d’activité pour une application Dynamics AX multiniveau   




 Dynamics AX figure parmi les solutions ERP les plus populaires au sein des entreprises pour standardiser les processus, gérer les ressources et simplifier la conformité sur l’ensemble des sites. En cas de défaillance, étant donné le rôle critique qu’elle joue au sein de l’entreprise, l’application doit pouvoir être restaurée le plus rapidement possible.

À l’heure actuelle, Dynamics AX ne fournit pas de fonctionnalités de récupération d’urgence prêtes à l’emploi. Dynamics AX comprend de nombreux composants serveur comme le serveur d’objets d’application Windows, Azure Active Directory, Azure SQL Database, SharePoint Server et Reporting Services. La gestion manuelle de la récupération d’urgence de chacun de ces composants est non seulement coûteuse, mais elle favorise également les erreurs.

Cet article explique comment créer une solution de récupération d’urgence pour votre application Dynamics AX avec [Azure Site Recovery](site-recovery-overview.md). Sont également couverts : les basculements de test planifiés et non planifiés à l’aide d’un plan de récupération en un seul clic, les configurations prises en charge et les prérequis.



## <a name="prerequisites"></a>Prérequis

L’implémentation de la récupération d’urgence pour l’application Dynamics AX à l’aide de Site Recovery nécessite les prérequis suivants :

• La configuration d’un déploiement local de Dynamics AX

• La création d’un coffre Site Recovery dans un abonnement Azure

• Si Azure est votre site de récupération, exécutez l’outil Azure Virtual Machine Readiness Assessment sur les machines virtuelles. Elles doivent être compatibles avec les services Machines virtuelles Microsoft Azure et Site Recovery.

## <a name="site-recovery-support"></a>Prise en charge de Site Recovery

Pour les besoins de cet article, des machines virtuelles VMware avec Dynamics AX 2012 R3 sur Windows Server 2012 R2 Enterprise ont été utilisées. Comme la réplication Site Recovery est indépendante des applications, les recommandations indiquées ici sont censées s’appliquer aux scénarios suivants.

### <a name="source-and-target"></a>Source et cible

**Scénario** | **Vers un site secondaire** | **Vers Azure**
--- | --- | ---
**Hyper-V** | Oui | Oui
**VMware** | Oui | Oui
**Serveur physique** | Oui | Oui

## <a name="enable-disaster-recovery-of-the-dynamics-ax-application-by-using-site-recovery"></a>Permettre la récupération d’urgence de l’application Dynamics AX à l’aide de Site Recovery
### <a name="protect-your-dynamics-ax-application"></a>Protéger votre application Dynamics AX
Pour permettre la réplication et la récupération complètes de l’application, chaque composant de Dynamics AX doit être protégé.

### <a name="1-set-up-active-directory-and-dns-replication"></a>1. Configurer la réplication Active Directory et DNS

Active Directory doit se trouver sur le site de récupération d’urgence pour que l’application Dynamics AX fonctionne. En fonction de la complexité de l’environnement local du client, deux choix vous sont recommandés.

**Option 1 :**

Le client dispose d’un petit nombre d’applications et d’un seul contrôleur de domaine pour l’ensemble du site local, et prévoit de faire basculer l’ensemble du site. Il est conseillé d’utiliser la réplication Site Recovery pour répliquer l’ordinateur contrôleur de domaine sur un site secondaire (ceci est applicable pour les scénarios de site à site et les scénarios de site vers Azure).

**Option 2 :**

Le client dispose d’un grand nombre d’applications, exécute une forêt Active Directory et prévoit de faire basculer quelques applications à la fois. Il est conseillé de configurer un contrôleur de domaine supplémentaire sur le site de récupération d’urgence (un site secondaire ou Azure).

 Pour plus d’informations, consultez [Rendre un contrôleur de domaine disponible sur un site de récupération d’urgence](site-recovery-active-directory.md). Dans la suite de ce document, nous partirons du principe qu’un contrôleur de domaine est disponible sur le site de récupération d’urgence.

### <a name="2-set-up-sql-server-replication"></a>2. Configurer la réplication SQL Server
Pour obtenir des conseils techniques sur l’option recommandée pour la protection de la couche SQL, consultez [Répliquer des applications avec SQL Server et Azure Site Recovery](site-recovery-sql.md).

### <a name="3-enable-protection-for-the-dynamics-ax-client-and-application-object-server-vms"></a>3. Activer la protection du client Dynamics AX et des machines virtuelles du serveur d’objets d’application
Effectuez la configuration Site Recovery appropriée selon que les machines virtuelles sont déployées sur [Hyper-V](./hyper-v-azure-tutorial.md) ou sur [VMware](./vmware-azure-tutorial.md).

> [!TIP]
> Il est conseillé de définir la fréquence de cohérence en cas d’incident sur 15 minutes.
>

La capture instantanée qui suit montre l’état de la protection des machines virtuelles avec composants Dynamics dans un scénario de protection de type « site VMware vers Azure ».

![Éléments protégés](./media/site-recovery-dynamics-ax/protecteditems.png)

### <a name="4-configure-networking"></a>4. Configurer la mise en réseau
**Configurer les paramètres de calcul et de réseau des machines virtuelles**

Pour le client Dynamics AX et les machines virtuelles de serveur d’objets d’application, configurez les paramètres réseau dans Site Recovery, afin que les réseaux des machines virtuelles soient associés au bon réseau de récupération d’urgence après le basculement. Vérifiez que le réseau de récupération d’urgence de ces couches est routable vers la couche SQL.

Vous pouvez sélectionner la machine virtuelle dans les éléments répliqués afin de configurer les paramètres réseau, comme indiqué dans la capture instantanée ci-dessous :

* Pour les serveurs d’objets d’application, sélectionnez le groupe à haute disponibilité qui convient.

* Si vous utilisez une adresse IP statique, spécifiez l’adresse IP que vous souhaitez attribuer à la machine virtuelle dans la zone de texte **Adresse IP cible**

    ![Paramètres réseau](./media/site-recovery-dynamics-ax/vmpropertiesaos1.png)


### <a name="5-create-a-recovery-plan"></a>5. Créer un plan de récupération

Vous pouvez créer un plan de récupération dans Site Recovery pour automatiser le processus de basculement. Ajoutez une couche Application et une couche Web au plan de récupération. Organisez-les en différents groupes afin que le front-end s’arrête avant la couche Application.

1. Sélectionnez le coffre Site Recovery dans votre abonnement, puis cliquez sur la vignette **Plans de récupération**.

2. Cliquez sur **+ Plan de récupération**, puis spécifiez un nom.

3. Sélectionnez la **Source** et la **Cible**. La cible peut être Azure ou un site secondaire. Si vous choisissez Azure, vous devez spécifier le modèle de déploiement.

    ![Créer un plan de récupération](./media/site-recovery-dynamics-ax/recoveryplancreation1.png)

4. Sélectionnez le serveur d’objets d’application et les machines virtuelles clientes pour le plan de récupération, puis sélectionnez la coche ✓.

    ![Sélectionner des éléments](./media/site-recovery-dynamics-ax/selectvms.png)

    Exemple de plan de récupération :

    ![Détails du plan de récupération](./media/site-recovery-dynamics-ax/recoveryplan.png)

Vous pouvez personnaliser le plan de récupération pour l’application Dynamics AX en ajoutant les étapes ci-dessous. La capture instantanée ci-dessus montre l’intégralité du plan de récupération après l’ajout de toutes ces étapes.


* **Étapes de basculement SQL Server** : pour plus d’informations sur les étapes de récupération spécifiques à SQL Server, consultez [Répliquer des applications avec SQL Server et Azure Site Recovery](site-recovery-sql.md).

* **Groupe de basculement 1** : faites basculer les machines virtuelles du serveur d’objets d’application.
Vérifiez que le point de récupération sélectionné est aussi proche que possible du PIT de la base de données, mais pas antérieur à lui.

* **Script** : ajoutez l’équilibreur de charge (E-A uniquement).
Ajoutez un script (via Azure Automation) après l’affichage du groupe de machines virtuelles du serveur d’objets d’application pour lui ajouter un équilibreur de charge. Vous pouvez utiliser un script pour effectuer cette tâche. Pour plus d’informations, consultez [Comment ajouter un équilibreur de charge pour la récupération d’urgence des applications multicouches](https://azure.microsoft.com/blog/cloud-migration-and-disaster-recovery-of-load-balanced-multi-tier-applications-using-azure-site-recovery/).

* **Groupe de basculement 2** : faites basculer les machines virtuelles du client Dynamics AX. Basculez les machines virtuelles de niveau web dans le cadre du plan de récupération.


### <a name="perform-a-test-failover"></a>Exécution d’un test de basculement

Pour plus d’informations sur Active Directory pendant le test de basculement, consultez le guide d’accompagnement « Solution de récupération d’urgence d’Active Directory ».

Pour plus d’informations sur SQL Server pendant le test de basculement, consultez [Répliquer des applications avec SQL Server et Azure Site Recovery](site-recovery-sql.md).

1. Accédez au portail Azure, puis sélectionnez votre coffre Site Recovery.

2. Sélectionnez le plan de récupération créé pour Dynamics AX.

3. Sélectionnez **Test de basculement**.

4. Sélectionnez le réseau virtuel pour démarrer le processus de test de basculement.

5. Lorsque l’environnement secondaire est opérationnel, vous pouvez effectuer vos validations.

6. Une fois les validations terminées, vous pouvez sélectionner **Validations terminées**. L’environnement de test de basculement est alors nettoyé.

Pour plus d’informations sur l’exécution d’un basculement de test, consultez [Basculement de test sur Azure dans Site Recovery](site-recovery-test-failover-to-azure.md).

### <a name="perform-a-failover"></a>Effectuer un basculement

1. Accédez au portail Azure, puis sélectionnez votre coffre Site Recovery.

2. Sélectionnez le plan de récupération créé pour Dynamics AX.

3. Cliquez sur **Basculement**, puis de nouveau sur **Basculement**.

4. Sélectionnez le réseau cible, puis la coche **✓** pour démarrer le processus de basculement.

Pour plus d’informations sur le processus de basculement, consultez [Basculement dans Site Recovery](site-recovery-failover.md).

### <a name="perform-a-failback"></a>Effectuer une restauration automatique

Pour plus d’informations sur SQL Server pendant une restauration automatique, consultez [Répliquer des applications avec SQL Server et Azure Site Recovery](site-recovery-sql.md).

1. Accédez au portail Azure, puis sélectionnez votre coffre Site Recovery.

2. Sélectionnez le plan de récupération créé pour Dynamics AX.

3. Cliquez sur **Basculement**, puis de nouveau sur **Basculement**.

4. Sélectionnez **Changer de direction**.

5. Sélectionnez les options de synchronisation de données et de création de machines virtuelles appropriées.

6. Sélectionnez la coche **✓** pour démarrer le processus de basculement.


Pour plus d’informations sur le processus de restauration automatique, consultez [Restaurer automatiquement des machines virtuelles VMware d’Azure vers un site local](./vmware-azure-failback.md).

## <a name="summary"></a>Résumé
À l’aide de Site Recovery, vous pouvez créer un plan de récupération d’urgence automatisée complet pour votre application Dynamics AX. En cas d’interruption du service, vous pouvez démarrer le basculement en quelques secondes, où que vous soyez, et bénéficier d’une application opérationnelle en quelques minutes.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la protection des charges de travail d’entreprise avec Site Recovery, consultez [Quelles charges de travail puis-je protéger ?](site-recovery-workload.md).
