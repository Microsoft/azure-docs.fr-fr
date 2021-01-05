---
title: 'Tutoriel : Diagnostiquer un problème de communication entre réseaux à l’aide du portail Azure'
titleSuffix: Azure Network Watcher
description: Dans ce tutoriel, découvrez comment diagnostiquer un problème de communication entre un réseau virtuel Azure connecté à un réseau local ou à un autre réseau virtuel, via une passerelle de réseau virtuel Azure, à l’aide de la fonctionnalité de diagnostic VPN de Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
Customer intent: I need to determine why resources in a virtual network can't communicate with resources in a different network.
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/27/2018
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: 25c58f99f56a4328527d1dd970e3ade21eb51819
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987169"
---
# <a name="tutorial-diagnose-a-communication-problem-between-networks-using-the-azure-portal"></a>Tutoriel : Diagnostiquer un problème de communication existant entre des réseaux à l’aide du portail Azure

Une passerelle de réseau virtuel connecte un réseau virtuel Azure à un réseau local ou à un autre réseau virtuel. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Diagnostiquer un problème lié à une passerelle de réseau virtuel avec la fonctionnalité de diagnostics VPN de Network Watcher
> * Diagnostiquer un problème lié à une connexion de passerelle
> * Résoudre un problème lié à une passerelle

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour utiliser la fonctionnalité de diagnostics VPN, vous devez disposer d’une passerelle VPN existante en cours d’exécution. Si ce n’est pas le cas, vous pouvez en déployer une à l’aide d’un [script PowerShell](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Vous pouvez exécuter le script PowerShell à partir :
- **Une installation PowerShell locale**. Le script nécessite le module `Az` Azure PowerShell. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.
- **Azure Cloud Shell**. [Azure Cloud Shell](https://shell.azure.com/powershell), qui dispose de la dernière version de PowerShell installée et configurée, vous connecte à Azure.

La création d’une passerelle VPN par le script prend environ une heure. Pour les étapes restantes, il est supposé que la passerelle que vous diagnostiquez est celle qui est déployée par ce script. Si vous préférez diagnostiquer votre passerelle existante, les résultats varient.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="enable-network-watcher"></a>Activer Network Watcher

Si vous avez déjà un observateur réseau activé dans la région USA Est, passez à l’étape [Diagnostiquer une passerelle](#diagnose-a-gateway).

1. Dans le portail Azure, sélectionnez **Tous les services**. Dans la zone **Filtre**, entrez *Network Watcher*. Quand la mention **Network Watcher** apparaît dans les résultats, sélectionnez-la.
2. Sélectionnez la zone **Régions** pour la développer, puis sélectionnez **...** à droite de la région **USA Est**, comme illustré dans l’image suivante :

    ![Activer Network Watcher](./media/diagnose-communication-problem-between-networks/enable-network-watcher.png)

3. Sélectionnez **Activer Network Watcher**.

## <a name="diagnose-a-gateway"></a>Diagnostiquer une passerelle

1. Dans la partie gauche du portail, sélectionnez **Tous les services**.
2. Entrez *network watcher* dans la zone **Filtre**. Quand la mention **Network Watcher** apparaît dans les résultats de recherche, sélectionnez-la.
3. Sous **OUTILS DE DIAGNOSTIC RÉSEAU**, sélectionnez **Diagnostics VPN**.
4. Sélectionnez **Compte de stockage**, puis le compte de stockage dans lequel vous souhaitez écrire les informations de diagnostic.
5. Dans la liste **Comptes de stockage**, sélectionnez le compte de stockage que vous souhaitez utiliser. Si vous ne possédez pas de compte de stockage, sélectionnez **+Compte de stockage**, entrez ou sélectionnez les informations requises, puis sélectionnez **Créer** pour en créer un. Si vous avez créé une passerelle VPN à l’aide du script dans [Prérequis](#prerequisites), vous souhaiterez peut-être créer le compte de stockage dans le même groupe de ressources, *TestRG1*, que la passerelle.
6. Dans la liste **Conteneurs**, sélectionnez le conteneur que vous souhaitez utiliser, puis **Sélectionner**. Si vous ne possédez pas de conteneurs, sélectionnez **+Conteneur**, entrez le nom du conteneur, puis sélectionnez **OK**.
7. Sélectionnez une passerelle, puis **Démarrer le dépannage**. Comme illustré sur l’image suivante, le test est exécuté sur une passerelle nommée **Vnet1GW** :

    ![Diagnostics VPN](./media/diagnose-communication-problem-between-networks/vpn-diagnostics.png)

8. Pendant l’exécution du test, **En cours d’exécution** s’affiche dans la colonne **ÉTAT DU DÉPANNAGE** où la mention **Non démarré** est affichée, comme illustré sur l’image précédente. L’exécution du test peut prendre plusieurs minutes.
9. Affichez l’état d’un test terminé. L’image suivante montre les résultats d’état d’un test de diagnostic terminé :

    ![Capture d’écran montrant les résultats d’un test de diagnostic (dans cet exemple, le test a révélé que le réseau n’est pas sain), comprenant un résumé et une description détaillée.](./media/diagnose-communication-problem-between-networks/status.png)

    Vous pouvez voir que la colonne **ÉTAT DU DÉPANNAGE** indique la mention **Défectueux**, et observer les paramètres **Résumé** et **Détail** du problème dans l’onglet **État**.
10. Lorsque vous sélectionnez l’onglet **Action**, la fonctionnalité de diagnostics VPN fournit des informations supplémentaires. Dans l’exemple illustré dans l’image suivante, la fonctionnalité de diagnostics VPN vous informe que vous devez vérifier l’intégrité de chaque connexion :

    ![Capture d’écran montrant l’onglet Action, qui fournit des informations supplémentaires.](./media/diagnose-communication-problem-between-networks/action.png)

## <a name="diagnose-a-gateway-connection"></a>Effectuer le diagnostic d’une connexion à une passerelle

Une passerelle est connectée à d’autres réseaux via une connexion de passerelle. La passerelle et les connexions de passerelle doivent être intègres pour que la communication entre un réseau virtuel et un réseau connecté aboutisse.

1. Effectuez de nouveau l’étape 7 de la section [Diagnostiquer une passerelle](#diagnose-a-gateway), en sélectionnant cette fois une connexion. Dans l’exemple suivant, une connexion nommée **VNet1toSite1** est testée :

    ![Capture d’écran montrant comment démarrer le dépannage de la connexion sélectionnée.](./media/diagnose-communication-problem-between-networks/connection.png)

    Le test est exécuté pendant plusieurs minutes.
2. À l’issue du test de la connexion, vous recevez des résultats similaires aux résultats présentés dans les images suivantes dans les onglets **État** et **Action** :

    ![État de la connexion](./media/diagnose-communication-problem-between-networks/connection-status.png)

    ![Action de connexion](./media/diagnose-communication-problem-between-networks/connection-action.png)

    La fonctionnalité de diagnostics VPN vous informe du problème dans l’onglet **État** et vous indique plusieurs suggestions relatives à son origine dans l’onglet **Action**.

    Si la passerelle que vous avez testée est celle qui a été déployée par le [script](../vpn-gateway/scripts/vpn-gateway-sample-site-to-site-powershell.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) dans [Prérequis](#prerequisites), le problème indiqué dans l’onglet **État** et les deux premiers éléments de l’onglet **Actions** correspondent exactement au problème. Le script configure une adresse IP d’espace réservé, 23.99.221.164, pour le périphérique de passerelle VPN locale.

    Pour résoudre ce problème, vous devez vous assurer que votre passerelle VPN locale est [correctement configurée](../vpn-gateway/vpn-gateway-about-vpn-devices.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json), puis modifier l’adresse IP configurée par le script pour la passerelle de réseau local pour la définir sur l’adresse publique réelle de votre passerelle VPN locale.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez créé une passerelle VPN à l’aide du script de la section [Prérequis](#prerequisites) uniquement pour effectuer ce didacticiel et que vous n’en avez plus besoin, supprimez le groupe de ressources et toutes les ressources qu’il contient :

1. Entrez *TestRG1* dans la zone **Recherche** en haut du portail. Lorsque le groupe de ressources **TestRG1** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Entrez *TestRG1* dans **TAPER NOM DU GROUPE DE RESSOURCES :** , puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez découvert comment diagnostiquer un problème lié à une passerelle de réseau virtuel. Vous souhaiterez peut-être enregistrer la communication réseau vers et depuis une machine virtuelle afin de pouvoir rechercher des anomalies dans le journal. Passez au didacticiel suivant pour savoir comment procéder.

> [!div class="nextstepaction"]
> [Consigner le trafic réseau vers et à partir d’une machine virtuelle](network-watcher-nsg-flow-logging-portal.md)
