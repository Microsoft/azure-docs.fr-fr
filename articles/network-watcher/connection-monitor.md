---
title: 'Tutoriel : Superviser la communication réseau à l’aide du portail Azure'
description: Dans ce tutoriel, découvrez comment superviser la communication réseau entre deux machines virtuelles avec la fonctionnalité de supervision de connexion d’Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to monitor communication between a VM and another VM. If the communication fails, I need to know why, so that I can resolve the problem.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: 2da675f0836dbb10ce5227e7e93e98d706cc5c64
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544800"
---
# <a name="tutorial-monitor-network-communication-between-two-virtual-machines-using-the-azure-portal"></a>Tutoriel : surveiller la communication réseau entre deux machines virtuelles à l’aide du portail Azure

> [!NOTE]
> Ce tutoriel traite du Moniteur de connexion (classique). Essayez le nouveau [Moniteur de connexion](connection-monitor-overview.md), qui offre une meilleure expérience de la supervision de la connectivité.

Il peut être essentiel pour votre organisation que la communication soit efficace entre une machine virtuelle et un point de terminaison comme une autre machine virtuelle. Parfois, des modifications de configuration sont introduites et mettent en péril cette communication. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer deux machines virtuelles
> * Surveiller la communication entre des machines virtuelles avec la fonctionnalité de surveillance de connexion de Network Watcher
> * Générer des alertes pour les métriques de surveillance de connexion
> * Diagnostiquer un problème de communication entre deux machines virtuelles et apprendre à le résoudre

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.



## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-vms"></a>Créer des machines virtuelles

Créez deux machines virtuelles.

### <a name="create-the-first-vm"></a>Créer la première machine virtuelle

1. Sélectionnez **+ Créer une ressource** en haut à gauche du portail Azure.
2. Sélectionnez **Compute**, puis un système d’exploitation. Dans ce tutoriel, **Windows Server 2016 Datacenter** est utilisé.
3. Entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis cliquez sur **OK** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|myVm1|
    |Nom d'utilisateur| Entrez un nom d’utilisateur de votre choix.|
    |Mot de passe| Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Abonnement| Sélectionnez votre abonnement.|
    |Resource group| Sélectionnez **Créer** et entrez **myResourceGroup**.|
    |Emplacement| Sélectionnez **USA Est**.|

4. Choisissez une taille de machine virtuelle, puis cliquez sur **Sélectionner**.
5. Sous **Paramètres**, sélectionnez **Extensions**. Sélectionnez **Ajouter une extension**, puis **Agent Network Watcher pour Windows**, comme l’illustre l’image suivante :

    ![Extension Agent Network Watcher](./media/connection-monitor/nw-agent-extension.png)

6. Sous **Agent Network Watcher pour Windows**, sélectionnez **Créer**, sous **Installer l’extension**, sélectionnez **OK**, puis sous **Extensions**, sélectionnez **OK**.
7. Acceptez les valeurs par défaut des autres **paramètres**, puis sélectionnez **OK**.
8. Sous **Créer** dans le **résumé**, sélectionnez **Créer** pour démarrer le déploiement de la machine virtuelle.

### <a name="create-the-second-vm"></a>Créer la seconde machine virtuelle

Réeffectuez les étapes indiquées dans [Créer la première machine virtuelle](#create-the-first-vm), avec les modifications suivantes :

|Étape|Paramètre|Valeur|
|---|---|---|
| 1 | Sélectionner une version de **Ubuntu Server** |                                                                         |
| 3 | Nom                                  | myVm2                                                                   |
| 3 | Type d'authentification                   | Collez votre clé publique SSH ou sélectionnez **Mot de passe**, puis entrez un mot de passe. |
| 3 | Resource group                        | Sélectionnez **Utiliser l’existant**, puis **myResourceGroup**.                 |
| 6 | Extensions                            | **Agent Network Watcher pour Linux**                                             |

Le déploiement de la machine virtuelle ne nécessite que quelques minutes. Attendez la fin du déploiement de la machine virtuelle avant d’effectuer les étapes restantes.

## <a name="create-a-connection-monitor"></a>Créer un moniteur de connexion

Créez un moniteur de connexion pour surveiller la communication sur le port TCP 22 entre *myVm1* et *myVm2*.

1. Dans la partie gauche du portail, sélectionnez **Tous les services**.
2. Entrez *network watcher* dans la zone **Filtre**. Quand la mention **Network Watcher** apparaît dans les résultats de recherche, sélectionnez-la.
3. Sous **SURVEILLANCE**, sélectionnez **Contrôle de la conversion**.
4. Sélectionnez **Ajouter**.
5. Entrez ou sélectionnez les informations appropriées à la connexion à contrôler, puis sélectionnez **Ajouter**. Dans l’exemple illustré par l’image suivante, la connexion surveillée est celle entre la machine virtuelle *myVm1* et la machine virtuelle *myVm2* sur le port 22 :

    | Paramètre                  | Valeur               |
    | ---------                | ---------           |
    | Nom                     | myVm1-myVm2(22)     |
    | Source                   |                     |
    | Machine virtuelle          | myVm1               |
    | Destination              |                     |
    | Sélectionnez une machine virtuelle |                     |
    | Machine virtuelle          | myVm2               |
    | Port                     | 22                  |

    ![Ajouter un contrôle de la connexion](./media/connection-monitor/add-connection-monitor.png)

## <a name="view-a-connection-monitor"></a>Afficher un moniteur de connexion

1. Effectuez les étapes 1 à 3 dans [Créer un contrôle de la connexion](#create-a-connection-monitor) pour afficher le contrôle de la connexion. La liste des moniteurs de connexion existants apparaît comme l’illustre l’image suivante :

    ![Moniteurs de connexion](./media/connection-monitor/connection-monitors.png)

2. Sélectionnez le moniteur qui porte le nom **myVm1-myVm2(22)**, comme l’illustre l’image précédente, pour en voir les détails, comme l’illustre l’image suivante :

    ![Détails du moniteur](./media/connection-monitor/vm-monitor.png)

    Notez les informations suivantes :

    | Élément                     | Valeur                      | Détails                                                     |
    | ---------                | ---------                  |--------                                                     |
    | Statut                   | Accessible                  | Vous permet de savoir si le point de terminaison est accessible ou non.|
    | DURÉE MOY. DES BOUCLES          | Vous indique la durée des boucles pour établir la connexion, en millisecondes. Le moniteur de connexion sonde la connexion toutes les 60 secondes, pour que vous puissiez surveiller la latence au fil du temps.                                         |
    | Hops                     | Le moniteur de connexion vous indique les tronçons entre les deux points de terminaison. Dans cet exemple, la connexion est établie entre deux machines virtuelles du même réseau virtuel, donc il n’y a qu’un seul tronçon, à l’adresse IP 10.0.0.5. S’il existe un système ou des itinéraires personnalisés qui acheminent le trafic entre les machines virtuelles par le biais d’une passerelle VPN ou d’une appliance de réseau virtuel, par exemple, d’autres tronçons sont répertoriés.                                                                                                                         |
    | STATUT                   | Les coches vertes de chaque point de terminaison vous indiquent qu’il est sain.    ||

## <a name="generate-alerts"></a>Générer des alertes

Les alertes sont créées par des règles d’alerte dans Azure Monitor et peuvent exécuter automatiquement des requêtes enregistrées ou des recherches personnalisées dans les journaux à intervalles réguliers. Une alerte générée peut exécuter automatiquement une ou plusieurs actions, par exemple avertir un utilisateur ou démarrer un processus. Lorsque vous définissez une règle d’alerte, en fonction de la ressource que vous ciblez, vous disposez d’une liste de métriques disponibles pour générer des alertes.

1. Dans le portail Azure, accédez au service **Surveiller**, puis sélectionnez **Alertes** > **Nouvelle règle d’alerte**.
2. Cliquez sur **Sélectionner une cible**, puis sélectionnez les ressources que vous souhaitez cibler. Sélectionnez l’**abonnement** et définissez le **type de ressource** pour déterminer la surveillance de connexion que vous voulez utiliser.

    ![écran d’alerte avec la cible sélectionnée](./media/connection-monitor/set-alert-rule.png)
1. Une fois que vous avez sélectionné une ressource à cibler, sélectionnez **Ajouter des critères**. Network Watcher dispose de [métriques à partir desquelles vous pouvez créer des alertes](../azure-monitor/platform/alerts-metric-near-real-time.md#metrics-and-dimensions-supported). Attribuez des **signaux disponibles** aux paramètres ProbesFailedPercent et AverageRoundtripMs des métriques :

    ![page d’alerte avec les signaux sélectionnés](./media/connection-monitor/set-alert-signals.png)
1. Renseignez les détails de l’alerte, comme le nom de règle d’alerte, la description et la gravité. Vous pouvez également ajouter un groupe d’actions à l’alerte afin d’automatiser et de personnaliser la réponse à l’alerte.

## <a name="view-a-problem"></a>Voir un problème

Par défaut, Azure permet la communication sur tous les ports entre les machines virtuelles du même réseau virtuel. Au fil du temps, une personne de votre organisation ou vous-même pouvez remplacer les règles par défaut d’Azure, par inadvertance, et ainsi entraîner un échec de communication. Procédez comme suit pour créer un problème de communication, puis reconsultez le moniteur de connexion :

1. Dans la zone de recherche située en haut du portail, entrez *myResourceGroup*. Quand le groupe de ressources **myResourceGroup** apparaît dans les résultats de recherche, sélectionnez-le.
2. Sélectionnez le groupe de sécurité réseau **myVm2-nsg**.
3. Sélectionnez **Règles de sécurité de trafic entrant**, puis **Ajouter**, comme l’illustre l’image suivante :

    ![Règles de sécurité de trafic entrant](./media/connection-monitor/inbound-security-rules.png)

4. La règle par défaut qui autorise la communication entre toutes les machines virtuelles d’un réseau virtuel porte le nom **AllowVnetInBound**. Créez une règle avec une priorité supérieure (numéro inférieur) à la règle **AllowVnetInBound** qui refuse les communications entrantes sur le port 22. Sélectionnez ou entrez les informations suivantes, acceptez les valeurs par défaut restantes, puis sélectionnez **Ajouter** :

    | Paramètre                 | Valeur          |
    | ---                     | ---            |
    | Plages de ports de destination | 22             |
    | Action                  | Deny           |
    | Priority                | 100            |
    | Nom                    | DenySshInbound |

5. Étant donné que le moniteur de connexion effectue un sondage toutes les 60 secondes, patientez quelques minutes, puis dans la partie gauche du portail, sélectionnez **Network Watcher**, puis **Moniteur de connexion**, puis resélectionnez le moniteur **myVm1-myVm2(22)**. Les résultats sont à présent différents, comme l’illustre l’image suivante :

    ![Détails de l’erreur moniteur](./media/connection-monitor/vm-monitor-fault.png)

    Vous pouvez voir que la colonne d’état contient un point d’exclamation rouge pour l’interface réseau **myvm2529**.

6. Pour découvrir pourquoi l’état a changé, sélectionnez 10.0.0.5, dans l’image précédente. Le moniteur de connexion vous informe que la raison de l’échec de communication est : *Trafic bloqué en raison de la règle de groupe de sécurité réseau suivante : UserRule_DenySshInbound*.

    Si vous ne saviez pas que quelqu’un avait implémenté la règle de sécurité que vous avez créée à l’étape 4, vous apprendriez du moniteur de connexion que cette règle est la cause du problème de communication. Vous pourriez alors la modifier, la remplacer ou la supprimer pour rétablir la communication entre les machines virtuelles.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, supprimez-le, ainsi que toutes les ressources qu’il contient :

1. Entrez *myResourceGroup* dans le champ **Recherche** en haut du portail. Quand **myResourceGroup** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Entrez *myResourceGroup* dans **TAPER NOM DU GROUPE DE RESSOURCES :** puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à surveiller une connexion entre deux machines virtuelles. Vous avez appris qu’une règle de groupe de sécurité réseau empêchait la communication avec une machine virtuelle. Pour connaître toutes les différentes réponses qu’un moniteur de connexion peut retourner, consultez les [types de réponse](network-watcher-connectivity-overview.md#response). Vous pouvez également surveiller une connexion entre une machine virtuelle, un nom de domaine complet, un URI ou une adresse IP.

À un moment donné, vous constaterez peut-être que les ressources d’un réseau virtuel ne sont pas en mesure de communiquer avec celles d’autres réseaux connectés par une passerelle de réseau virtuel Azure. Passez au tutoriel suivant pour apprendre à diagnostiquer un problème lié à une passerelle de réseau virtuel.

> [!div class="nextstepaction"]
> [Diagnostiquer des problèmes de communication entre réseaux](diagnose-communication-problem-between-networks.md)