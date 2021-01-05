---
title: Configurer la récupération d’urgence d’une application web IIS avec Azure Site Recovery
description: Découvrez comment répliquer les machines virtuelles d’une batterie de serveurs web IIS avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/27/2018
ms.author: mayg
ms.openlocfilehash: 7a4408b54b663b2cd8abc22772ac1b799ea50de0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87083767"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-iis-based-web-application"></a>Configurer la reprise d’activité pour une application web multiniveau basée sur IIS

Les logiciels d’application sont le moteur de la productivité dans une organisation. Des applications web diverses peuvent répondre aux différents besoins d’une organisation. Certaines applications, telles que les applications utilisées pour le traitement des paies, les applications financières et les sites Web destinés aux clients, peuvent être essentielles à une entreprise. Afin d’éviter une perte de productivité, il est fondamental pour l’entreprise que ces applications soient opérationnelles en permanence. Plus important encore, le maintien de la disponibilité de ces applications peut contribuer à préserver de toute nuisance l’image ou la marque de l’entreprise.

Les applications web critiques sont généralement configurées comme des applications multiniveaux : le web, la base de données et l’application se situent à différents niveaux. Outre leur répartition sur différents niveaux, ces applications peuvent également utiliser plusieurs serveurs de chaque niveau pour équilibrer la charge du trafic. Par ailleurs, les mappages entre les différents niveaux et le serveur web peuvent s’appuyer sur des adresses IP statiques. Lors du basculement, certains de ces mappages doivent être mis à jour, en particulier si plusieurs sites web sont configurés sur le serveur web. Dans le cas d’applications web utilisant TLS, vous devez mettre à jour les liaisons de certificat.

Les méthodes de récupération traditionnelles qui ne reposent pas sur la réplication impliquent la sauvegarde de divers fichiers de configuration, paramètres du Registre, liaisons, composants personnalisés (COM ou .NET), contenus et certificats. Un ensemble d’étapes manuelles permet de récupérer ces fichiers. Les méthodes de récupération traditionnelles avec sauvegarde et récupération manuelle des fichiers sont fastidieuses, sujettes à erreur et non scalables. Par exemple, vous pouvez facilement oublier de sauvegarder des certificats. Après le basculement, vous n’avez plus d’autre choix que celui d’acheter de nouveaux certificats pour le serveur.

Une bonne solution de récupération d’urgence prend en charge la modélisation de plans de récupération pour les architectures d’application complexes. La possibilité d’ajouter des étapes personnalisées au plan de récupération pour la prise en charge des mappages d’application entre les niveaux doit également vous être proposée. En cas d’urgence, les mappages d’application fournissent une solution infaillible en un clic qui permet de déboucher sur un RTO plus faible.

Cet article explique comment protéger une application web qui s’appuie sur Internet Information Services (IIS) à l’aide d’[Azure Site Recovery](site-recovery-overview.md). Il décrit les recommandations à suivre pour répliquer une application web IIS à trois niveaux vers Azure, il indique également comment effectuer un test de récupération d’urgence et basculer l’application vers Azure.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que vous savez accomplir les tâches suivantes :

* [Répliquer une machine virtuelle vers Azure](vmware-azure-tutorial.md)
* [Constituer un réseau de récupération](./concepts-on-premises-to-azure-networking.md)
* [Effectuer un test de basculement vers Azure](site-recovery-test-failover-to-azure.md)
* [Procéder à un basculement vers Azure](site-recovery-failover.md)
* [Répliquer un contrôleur de domaine](site-recovery-active-directory.md)
* [Répliquer SQL Server](site-recovery-sql.md)

## <a name="deployment-patterns"></a>Modèles de déploiement
Une application web IIS suit généralement l’un des modèles de déploiement suivants :

**Modèle de déploiement 1**

Une batterie de serveurs web IIS avec ARR (Application Request Routing), un serveur IIS et SQL Server.

![Schéma d’une batterie de serveurs web IIS à trois niveaux](./media/site-recovery-iis/deployment-pattern1.png)

**Modèle de déploiement 2**

Une batterie de serveurs web IIS avec ARR, un serveur IIS, un serveur d’applications et SQL Server.

![Schéma d’une batterie de serveurs web IIS à quatre niveaux](./media/site-recovery-iis/deployment-pattern2.png)

## <a name="site-recovery-support"></a>Prise en charge de Site Recovery

Pour les exemples de cet article, nous utilisons des machines virtuelles VMware avec IIS 7.5 sur Windows Server 2012 R2 Enterprise. La réplication de Site Recovery n’étant pas spécifique à une application, les recommandations contenues dans cet article sont censées s’appliquer aux scénarios répertoriés dans le tableau suivant, et pour les différentes versions d’IIS.

### <a name="source-and-target"></a>Source et cible

Scénario | Vers un site secondaire | Vers Azure
--- | --- | ---
Hyper-V | Oui | Oui
VMware | Oui | Oui
Serveur physique | Non | Oui
Azure|N/D|Oui

## <a name="replicate-virtual-machines"></a>Répliquer des machines virtuelles

Pour démarrer la réplication de toutes les machines virtuelles de la batterie de serveurs IIS web vers Azure, suivez les instructions dans [Tester le basculement vers Azure dans Site Recovery](site-recovery-test-failover-to-azure.md).

Si vous utilisez une adresse IP statique, vous pouvez spécifier l’adresse IP que vous souhaitez attribuer à la machine virtuelle. Pour définir l’adresse IP, accédez à **Paramètres Calcul et réseau** > **ADRESSE IP CIBLE**.

![Capture d’écran illustrant la définition de l’adresse IP cible dans le volet Calcul et réseau de Site Recovery](./media/site-recovery-active-directory/dns-target-ip.png)

## <a name="create-a-recovery-plan"></a>Créer un plan de récupération
Lors d’un basculement, un plan de récupération prend en charge le séquencement des différents niveaux d’une application multiniveau. La mise en séquence permet d’assurer la cohérence de l’application. Lorsque vous créez un plan de récupération pour une application web multiniveau, suivez les étapes décrites dans [Créer un plan de récupération à l’aide de Site Recovery](site-recovery-create-recovery-plans.md).

### <a name="add-virtual-machines-to-failover-groups"></a>Ajouter des machines virtuelles à des groupes de basculement
Une application web IIS multiniveau classique comprend les composants suivants :
* Une couche Base de données qui a les machines virtuelles SQL
* La couche Web, qui se compose d’un serveur IIS, et une couche Application 

Ajouter des machines virtuelles à différents groupes en fonction du niveau :

1. Créez un plan de récupération. Ajoutez les machines virtuelles de la couche Base de données sous le groupe 1. De cette façon, les machines virtuelles de la couche Base de données sont à coup sûr arrêtées en dernier, et affichées en premier.
1. Ajoutez les machines virtuelles de la couche Application sous le groupe 2. De cette façon, les machines virtuelles de la couche Application sont à coup sûr affichées après la couche Base de données.
1. Ajoutez les machines virtuelles de la couche Web dans le groupe 3. De cette façon, les machines virtuelles de la couche Web sont à coup sûr affichées après la couche Application.
1. Ajoutez les machines virtuelles de l’équilibrage de charge au groupe 4. De cette façon, les machines virtuelles de l’équilibrage de charge sont à coup sûr affichées après la couche Web.

Pour plus d’informations, consultez [Personnaliser le plan de récupération](site-recovery-runbook-automation.md#customize-the-recovery-plan).


### <a name="add-a-script-to-the-recovery-plan"></a>Ajouter un script au plan de récupération
Pour que la batterie de serveurs web IIS fonctionne correctement, vous devrez peut-être effectuer certaines opérations sur les machines virtuelles Azure, après le basculement ou lors d’un test de basculement. Vous pouvez automatiser certaines opérations après le basculement. Par exemple, vous pouvez mettre à jour l’entrée DNS, modifier une liaison de site ou une chaîne de connexion en ajoutant au plan de récupération les scripts correspondants. L’article [Ajouter un script VMM à un plan de récupération](./hyper-v-vmm-recovery-script.md) explique comment configurer les tâches automatisées à l’aide d’un script.

#### <a name="dns-update"></a>Mise à jour DNS
Si le système DNS est configuré pour la mise à jour DNS dynamique, les machines virtuelles le mettent généralement à jour avec la nouvelle adresse IP dès leur démarrage. Si vous voulez ajouter une étape explicite afin de mettre à jour le DNS avec les nouvelles adresses IP des machines virtuelles, ajoutez un [script pour mettre à jour les adresses IP dans DNS](https://aka.ms/asr-dns-update) en tant qu’action post-basculement dans les groupes de plan de récupération.  

#### <a name="connection-string-in-an-applications-webconfig"></a>Chaîne de connexion dans le fichier web.config d’une application
La chaîne de connexion spécifie la base de données avec laquelle le site web communique. Si la chaîne de connexion porte le nom d’une machine virtuelle de base de données, aucune étape postérieure au basculement n’est nécessaire. L’application peut communiquer automatiquement avec la base de données. De plus, si l’adresse IP de la machine virtuelle base de données est conservée, il n’est pas nécessaire de mettre à jour la chaîne de connexion. 

Si la chaîne de connexion désigne la machine virtuelle base de données à l’aide d’une adresse IP, elle doit être mise à jour après le basculement. Par exemple, la chaîne de connexion suivante pointe vers la base de données avec l’adresse IP 127.0.1.2 :

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<connectionStrings>
<add name="ConnStringDb1" connectionString="Data Source= 127.0.1.2\SqlExpress; Initial Catalog=TestDB1;Integrated Security=False;" />
</connectionStrings>
</configuration>
```

Pour mettre à jour la chaîne de connexion dans la couche Web, ajoutez un [script de mise à jour de la connexion IIS](https://gallery.technet.microsoft.com/Update-IIS-connection-2579aadc) après le groupe 3, dans le plan de récupération.

#### <a name="site-bindings-for-the-application"></a>Liaisons de site pour l’application
Chaque site comporte des informations de liaison. Ces informations de liaison incluent le type de liaison, l’adresse IP à laquelle le serveur IIS écoute les requêtes correspondant au site, le numéro de port et les noms d’hôte du site. Lors d’un basculement, vous pouvez être amené à mettre à jour ces liaisons en cas de modification de l’adresse IP qui leur est associée.

> [!NOTE]
>
> Si vous définissez la liaison de site sur **Toutes non attribuées**, vous n’avez pas besoin de mettre à jour cette liaison suite au basculement. De même, si l’adresse IP associée à un site n’est pas modifiée après le basculement, vous n’avez pas besoin de mettre à jour la liaison de site. (La rétention de l’adresse IP dépend de l’architecture réseau et des sous-réseaux affectés aux sites principaux et de restauration. Leur mise à jour peut s’avérer impossible pour votre organisation.)

![Capture d’écran qui illustre le paramétrage de la liaison TLS/SSL](./media/site-recovery-iis/sslbinding.png)

Si vous avez associé l’adresse IP à un site, mettez à jour toutes les liaisons de site avec la nouvelle adresse IP. Pour modifier les liaisons de site, ajoutez un [script de mise à jour de la couche Web IIS](https://aka.ms/asr-web-tier-update-runbook-classic) après le groupe 3, dans le plan de récupération.

#### <a name="update-the-load-balancer-ip-address"></a>Mettre à jour l’adresse IP de l’équilibreur de charge
Si vous avez une machine virtuelle ARR, ajoutez un [script de basculement ARR IIS](https://aka.ms/asr-iis-arrtier-failover-script-classic) après le groupe 4 pour mettre à jour l’adresse IP.

#### <a name="tlsssl-certificate-binding-for-an-https-connection"></a>Liaison de certificat TLS/SSL d’une connexion HTTPS
Le cas échéant, un site web peut posséder un certificat TLS/SSL associé qui garantit une communication sécurisée entre le serveur web et le navigateur de l’utilisateur. Si le site web dispose d’une connexion HTTPS et d’une liaison de site HTTPS associée à l’adresse IP du serveur IIS avec une liaison de certificat TLS/SSL, vous devez ajouter une nouvelle liaison de site pour le certificat avec l’adresse IP de la machine virtuelle IIS après le basculement.

Le certificat TLS/SSL peut être émis en incluant ces composants :

* Le nom de domaine complet du site web
* Le nom du serveur
* Un certificat générique pour le nom de domaine  
* Une adresse IP Si le certificat TLS/SSL est émis en incluant l’adresse IP du serveur IIS, un autre certificat TLS/SSL doit être émis en incluant l’adresse IP du serveur IIS sur le site Azure. Vous devez créer une liaison TLS supplémentaire pour ce certificat. Nous vous recommandons par conséquent de ne pas utiliser de certificat TLS/SSL émis en incluant l’adresse IP. Cette option est moins couramment utilisée et sera bientôt dépréciée conformément aux nouvelles modifications du CA/Browser Forum.

#### <a name="update-the-dependency-between-the-web-tier-and-the-application-tier"></a>Mettre à jour la dépendance entre la couche Web et la couche Application
Si une dépendance propre à une application est basée sur l’adresse IP des machines virtuelles, vous devez mettre à jour cette dépendance après le basculement.

## <a name="run-a-test-failover"></a>Exécuter un test de basculement

1. Dans le portail Azure, sélectionnez votre coffre Recovery Services.
2. Sélectionnez le plan de récupération que vous avez créé pour la batterie de serveurs web IIS.
3. Sélectionnez **Test de basculement**.
4. Pour démarrer le test de basculement, sélectionnez le point de récupération et le réseau virtuel Azure.
5. Lorsque l’environnement secondaire est opérationnel, vous pouvez procéder aux validations.
6. Pour nettoyer l’environnement du test de basculement lorsque ces validations sont terminées, sélectionnez **Validations terminées**.

Pour plus d’informations, consultez [Tester le basculement vers Azure dans Site Recovery](site-recovery-test-failover-to-azure.md).

## <a name="run-a-failover"></a>Exécuter un basculement

1. Dans le portail Azure, sélectionnez votre coffre Recovery Services.
1. Sélectionnez le plan de récupération que vous avez créé pour la batterie de serveurs web IIS.
1. Sélectionnez **Basculement**.
1. Pour démarrer le processus de basculement, sélectionnez le point de récupération.

Pour plus d’informations, consultez [Basculement dans Site Recovery](site-recovery-failover.md).

## <a name="next-steps"></a>Étapes suivantes
* Approfondissez vos connaissances sur la [réplication d’autres applications](site-recovery-workload.md) à l’aide de Site Recovery.
