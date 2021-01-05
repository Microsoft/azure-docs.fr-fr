---
title: Surveiller les événements planifiés pour vos machines virtuelles Windows dans Azure
description: Découvrez comment surveiller vos machines virtuelles Azure pour les événements planifiés.
author: mysarn
ms.service: virtual-machines-windows
ms.subservice: monitoring
ms.date: 08/20/2019
ms.author: sarn
ms.topic: how-to
ms.openlocfilehash: 0d1edde5ac1b83feab458eb5d12d524163d3ffb1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483298"
---
# <a name="monitoring-scheduled-events"></a>Surveillance d’événements planifiés

Des mises à jour sont appliquées à différentes parties d’Azure tous les jours, pour assurer la sécurité des services et les mettre à jour. En plus des mises à jour planifiées, des événements non planifiés peuvent également se produire. Par exemple, en cas de dégradation ou de panne du matériel, les services Azure peuvent avoir besoin d’effectuer une maintenance non planifiée. Avec la migration dynamique, les mises à jour de préservation de la mémoire et en appliquant une limite stricte sur l’impact des mises à jour, ces événements sont dans la plupart des cas presque transparents pour les clients et ils n’ont pas d’impact ou causent au maximum un blocage de quelques secondes des machines virtuelles. Toutefois, pour certaines applications, même quelques secondes de blocage des machines virtuelles peuvent avoir un impact. Il est important de connaître à l’avance sur les opérations de maintenance Azure à venir pour garantir une expérience optimale pour ces applications. Le service [Scheduled Events](scheduled-events.md) fournit une interface de programmation qui vous informe des opérations de maintenance à venir et vous permet de gérer les maintenances de manière appropriée. 

Dans cet article, nous allons vous montrer comment vous pouvez utiliser des événements planifiés pour être informé des événements de maintenance susceptibles d’affecter vos machines virtuelles et comment créer une automatisation simple qui peut faciliter la surveillance et l’analyse.


## <a name="routing-scheduled-events-to-log-analytics"></a>Routage des événements planifiés vers Log Analytics

Scheduled Events est disponible dans le cadre d’[Azure Instance Metadata Service](instance-metadata-service.md), qui est disponible sur chaque machine virtuelle Azure. Les clients peuvent écrire l’automatisation pour interroger le point de terminaison de leurs machines virtuelles afin de rechercher les notifications de maintenance planifiées et d’appliquer des mesures d’atténuation comme l’enregistrement de l’état et la mise hors connexion de la machine virtuelle. Nous vous conseillons de créer une automatisation pour enregistrer les événements planifiés afin de disposer d’un journal d’audit des événements de maintenance Azure. 

Dans cet article, nous allons vous guider tout au long de la capture des événements de maintenance planifiés dans Log Analytics. Ensuite, nous allons déclencher des actions de notification de base, comme l’envoi d’un e-mail à votre équipe et l’obtention d’une vue historique de tous les événements qui ont affecté vos machines virtuelles. Pour l’agrégation et l’automatisation des événements, nous allons utiliser [Log Analytics](../../azure-monitor/learn/quick-create-workspace.md), mais vous pouvez utiliser n’importe quelle solution de surveillance pour collecter ces journaux et déclencher l’automatisation.

![Diagramme montrant le cycle de vie des événements](./media/notifications/events.png)

## <a name="prerequisites"></a>Prérequis

Pour cet exemple, vous devez créer une [machine virtuelle Windows dans un groupe à haute disponibilité](tutorial-availability-sets.md). Scheduled Events fournit des notifications sur les modifications qui peuvent affecter les machines virtuelles de votre groupe à haute disponibilité, le service Cloud, le groupe de machines virtuelles identiques ou les machines virtuelles autonomes. Nous allons exécuter un [service](https://github.com/microsoft/AzureScheduledEventsService) qui interroge les événements planifiés sur l’une des machines virtuelles qui jouera le rôle de collecteur, afin de récupérer les événements pour toutes les autres machines virtuelles du groupe à haute disponibilité.    

Ne supprimez pas le groupe de ressources à la fin du tutoriel.

Vous devrez également [créer un espace de travail Log Analytics](../../azure-monitor/learn/quick-create-workspace.md) que nous utiliserons pour agréger les informations à partir des machines virtuelles dans le groupe à haute disponibilité.

## <a name="set-up-the-environment"></a>Configurer l’environnement

Vous devez maintenant avoir 2 machines virtuelles initiales dans un groupe à haute disponibilité. Nous devons maintenant créer une troisième machine virtuelle appelée myCollectorVM dans le même groupe à haute disponibilité. 

```azurepowershell-interactive
New-AzVm `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Name "myCollectorVM" `
   -Location "East US" `
   -VirtualNetworkName "myVnet" `
   -SubnetName "mySubnet" `
   -SecurityGroupName "myNetworkSecurityGroup" `
   -OpenPorts 3389 `
   -PublicIpAddressName "myPublicIpAddress3" `
   -AvailabilitySetName "myAvailabilitySet" `
   -Credential $cred
```
 

Téléchargez le fichier .zip d’installation du projet sur [GitHub](https://github.com/microsoft/AzureScheduledEventsService/archive/master.zip).

Connectez-vous à **myCollectorVM** et copiez le fichier .zip sur la machine virtuelle et décompressez tous les fichiers. Sur votre machine virtuelle, ouvrez une invite PowerShell. Déplacez votre invite dans le dossier contenant `SchService.ps1`, par exemple : `PS C:\Users\azureuser\AzureScheduledEventsService-master\AzureScheduledEventsService-master\Powershell>` et configurez le service.

```powershell
.\SchService.ps1 -Setup
```

Démarrez le service.

```powershell
.\SchService.ps1 -Start
```

Le service commence à interroger toutes les 10 secondes tous les événements planifiés et approuve les événements pour accélérer la maintenance.  Freeze, Reboot, Redeploy et Preempt sont les événements capturés par Scheduled Events.   Notez que vous pouvez étendre le script pour déclencher des atténuations avant d’approuver l’événement.

Validez l’état du service et assurez-vous qu’il est en cours d’exécution.

```powershell
.\SchService.ps1 -status  
```

Le résultat renvoyé doit être `Running`.

Le service commence à interroger toutes les 10 secondes tous les événements planifiés et approuve les événements pour accélérer la maintenance.  Freeze, Reboot, Redeploy et Preempt sont les événements capturés par Scheduled Events. Vous pouvez étendre le script pour déclencher des atténuations avant d’approuver l’événement.

Lorsqu’un des événements ci-dessus est capturé par le service Scheduled Events, il est enregistré dans le journal des événements de l’application, dans Event Status, Event Type, Resources (nom des machines virtuelles) et NotBefore (période de préavis minimale). Vous pouvez retrouver les événements avec l’ID 1234 dans le journal des événements de l’application.

Une fois le service configuré et démarré, les événements sont consignés dans les journaux des applications Windows.   Pour vérifier que tout fonctionne, redémarrez une des machines virtuelles dans le groupe à haute disponibilité. Vous devez voir un événement enregistré dans l’observateur d’événements dans Journaux Windows > Journal des applications, qui indique que la machine virtuelle a redémarré. 

![Capture d’écran de l’Observateur d’événements.](./media/notifications/event-viewer.png)

Lorsque des événements sont capturés par le service Scheduled Events, ils sont enregistrés dans le journal des événements de l’application, avec Event Status, Event Type, Resources (nom de la machine virtuelle) et NotBefore (période de préavis minimale). Vous pouvez retrouver les événements avec l’ID 1234 dans le journal des événements de l’application.

> [!NOTE] 
> Dans cet exemple, les machines virtuelles étaient dans un groupe à haute disponibilité, ce qui nous permettait de désigner une seule machine virtuelle comme collecteur pour écouter et acheminer les événements planifiés vers notre espace de travail Log Analytics. Si vous avez des machines virtuelles autonomes, vous pouvez exécuter le service sur chaque machine virtuelle, puis les connecter individuellement à votre espace de travail Log Analytics.
>
> Dans notre configuration, nous avons choisi Windows, mais vous pouvez concevoir la même solution sur Linux.

À tout moment, vous pouvez arrêter/supprimer le service Scheduled Events à l’aide des commutateurs `–stop` et `–remove`.

## <a name="connect-to-the-workspace"></a>Se connecter à l’espace de travail


Nous souhaitons maintenant connecter un espace de travail Log Analytics à la machine virtuelle qui joue le rôle de collecteur. L’espace de travail Log Analytics fonctionne comme un référentiel et nous allons configurer la collecte des journaux des événements pour capturer les journaux des applications à partir de la machine virtuelle collecteur. 

 Pour acheminer les événements planifiés vers le journal des événements, qui sera enregistré en tant que journal des applications par notre service, vous devrez connecter cette machine virtuelle à votre espace de travail Log Analytics.  
 
1. Ouvrez la page de l’espace de travail que vous avez créé.
1. Sous **Se connecter à une source de données**, sélectionnez **Machines virtuelles Azure**.

    ![Se connecter à une machine virtuelle comme source de données](./media/notifications/connect-to-data-source.png)

1. Recherchez et sélectionnez **MyCollectorVM**. 
1. Sur la nouvelle page de **myCollectorVM**, sélectionnez **Se connecter**.

Cette opération installe l’[agent Microsoft Monitoring](../extensions/oms-windows.md) sur votre machine virtuelle. La connexion de votre machine virtuelle à l’espace de travail et l’installation de l’extension peuvent prendre quelques minutes. 

## <a name="configure-the-workspace"></a>Configurer l’espace de travail

1. Ouvrez la page de votre espace de travail et sélectionnez **Paramètres avancés**.
1. Sélectionnez **Données** dans le menu de gauche, puis **Journaux des événements Windows**.
1. Dans **Collecter dans les journaux des événements suivants**, commencez à taper *application*, puis sélectionnez **Application** dans la liste.

    ![Sélectionnez Paramètres avancés](./media/notifications/advanced.png)

1. Laissez les options **ERREUR**, **AVERTISSEMENT** et **INFORMATIONS** sélectionnées, puis sélectionnez **Enregistrer** pour enregistrer les paramètres.


> [!NOTE]
> Il y aura un certain délai et il peut s’écouler jusqu’à 10 minutes avant que le journal ne soit disponible. 


## <a name="creating-an-alert-rule-with-azure-monitor"></a>Création d’une règle d’alerte avec Azure Monitor 


Une fois les événements envoyés à Log Analytics, vous pouvez exécuter la [requête](../../azure-monitor/log-query/log-analytics-tutorial.md) suivante pour rechercher les événements planifiés.

1. En haut de la page, sélectionnez **Journaux** et collez le code suivant dans la zone de texte :

    ```
    Event
    | where EventLog == "Application" and Source contains "AzureScheduledEvents" and RenderedDescription contains "Scheduled" and RenderedDescription contains "EventStatus" 
    | project TimeGenerated, RenderedDescription
    | extend ReqJson= parse_json(RenderedDescription)
    | extend EventId = ReqJson["EventId"]
    ,EventStatus = ReqJson["EventStatus"]
    ,EventType = ReqJson["EventType"]
    ,NotBefore = ReqJson["NotBefore"]
    ,ResourceType = ReqJson["ResourceType"]
    ,Resources = ReqJson["Resources"]
    | project-away RenderedDescription,ReqJson
    ```

1. Sélectionnez **Enregistrer**, puis tapez le nom *logQuery*, laissez le type **Requête**, tapez la **catégorie** *VMLogs*, puis sélectionnez **Enregistrer**. 

    ![Enregistrer la requête](./media/notifications/save-query.png)

1. Sélectionnez **Nouvelle règle d’alerte**. 
1. Dans la page **Créer une règle**, laissez `collectorworkspace` comme **Ressource**.
1. Sous **Condition**, sélectionnez l’entrée *Chaque fois que la recherche dans les journaux client est <login undefined>* . La page **Configurer la logique du signal** s’ouvre.
1. Sous **Valeur de seuil**, entrez *0*, puis sélectionnez **Terminé**.
1. Sous **Actions**, sélectionnez **Créer un groupe d’actions**. La page **Ajouter un groupe d’actions** s’ouvre.
1. Dans **Nom du groupe d’actions**, tapez *myActionGroup*.
1. Dans **Nom court**, tapez **myActionGroup**.
1. Dans **Groupe de ressources**, sélectionnez **myResourceGroupAvailability**.
1. Sous actions, dans **NOM DE L’ACTION**, tapez **E-mail**, puis sélectionnez **E-mail/SMS/Push/Voix**. La page **E-mail/SMS/Push/Voix** s’ouvre.
1. Sélectionnez **E-mail**, tapez votre adresse de courrier, puis cliquez sur **OK**.
1. Dans la page **Ajouter un groupe d’actions**, sélectionnez **OK**. 
1. Dans la page **Créer une règle**, sous **DÉTAILS DE L’ALERTE**, tapez *myAlert* comme **nom de la règle d’alerte**, puis tapez *Règle d’alerte e-mail* dans **Description**.
1. Lorsque vous avez terminé, cliquez sur **Créer une règle d’alerte**.
1. Redémarrez une des machines virtuelles du groupe à haute disponibilité. Au bout de quelques minutes, vous devez recevoir un courrier électronique indiquant que l’alerte a été déclenchée.

Pour gérer vos règles d’alerte, accédez au groupe de ressources, sélectionnez **Alertes** dans le menu de gauche, puis sélectionnez **Gérer les règles d’alerte** en haut de la page.

     
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez la page [Service Scheduled Events](https://github.com/microsoft/AzureScheduledEventsService) sur GitHub.