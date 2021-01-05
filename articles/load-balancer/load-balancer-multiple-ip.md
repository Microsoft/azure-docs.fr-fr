---
title: Équilibrage de charge sur plusieurs configurations IP - Portail Azure
titleSuffix: Azure Load Balancer
description: Dans cet article, vous en apprendrez plus sur l’équilibrage de charge sur des configurations IP principales et secondaires à l’aide du Portail Azure.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: se0dec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: 5a896d3fbe2d191473b10655ccb19c5759762131
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84803632"
---
# <a name="load-balancing-on-multiple-ip-configurations-by-using-the-azure-portal"></a>Équilibrage de charge sur plusieurs configurations IP dans le portail Azure

> [!div class="op_single_selector"]
> * [Portail](load-balancer-multiple-ip.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)
> * [INTERFACE DE LIGNE DE COMMANDE](load-balancer-multiple-ip-cli.md)


Dans cet article, nous allons vous montrer comment utiliser Azure Load Balancer avec plusieurs adresses IP sur un contrôleur d’interface réseau (carte réseau) secondaire. Le diagramme suivant illustre notre scénario :

![Scénario d’équilibreur de charge](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

Dans ce scénario, nous utilisons la configuration suivante :

- Deux machines virtuelles qui exécutent Windows.
- Chaque machine virtuelle possède une carte réseau principale et une carte réseau secondaire.
- Chacune des cartes réseau secondaires dispose de deux configurations IP.
- Chaque machine virtuelle héberge deux sites web : contoso.com et fabrikam.com.
- Chaque site web est lié à une configuration IP sur la carte réseau secondaire.
- Azure Load Balancer est utilisé pour exposer deux adresses IP frontales, une pour chaque site web. Les adresses frontales sont utilisées pour répartir le trafic sur la configuration IP respective de chaque site web.
- Le même numéro de port est utilisé pour les adresses IP frontales et pour celles du pool principal.

## <a name="prerequisites"></a>Conditions préalables requises

Notre exemple de scénario suppose que vous disposez d’un groupe de ressources nommé **contosofabrikam** configuré de la manière suivante :

- Le groupe de ressources inclut un réseau virtuel nommé **myVNet**.
- Le réseau **myVNet** comprend deux machines virtuelles nommées **VM1** et **VM2**.
- VM1 et VM2 se trouvent dans le même groupe à haute disponibilité nommé **myAvailset**. 
- VM1 et VM2 disposent chacune d’une carte réseau principale nommée **VM1NIC1** et **VM2NIC1**, respectivement. 
- VM1 et VM2 disposent chacune d’une carte réseau secondaire nommée **VM1NIC2** et **VM2NIC2**, respectivement.

Pour plus d’informations sur la création de machines virtuelles avec plusieurs cartes réseau, consultez [Créer une machine virtuelle avec plusieurs cartes réseau à l’aide de PowerShell](../virtual-machines/windows/multiple-nics.md).

## <a name="perform-load-balancing-on-multiple-ip-configurations"></a>Effectuer un équilibrage de charge sur plusieurs configurations IP

Suivez les étapes ci-dessous pour accomplir le scénario décrit dans cet article.

### <a name="step-1-configure-the-secondary-nics"></a>Étape 1 : Configurer les cartes réseau secondaires

Pour chaque machine virtuelle dans votre réseau virtuel, ajoutez la configuration IP définie pour la carte réseau secondaire :  

1. Accédez au portail Azure à l’adresse https://portal.azure.com. Connectez-vous à votre compte Azure.

2. Dans le coin supérieur gauche de l’écran, sélectionnez l’icône **Groupe de ressources**. Sélectionnez le groupe de ressources dans lequel se trouvent vos machines virtuelles (par exemple, **contosofabrikam**). Le volet **Groupes de ressources** affiche toutes les ressources et les cartes réseau pour les machines virtuelles.

3. Ajoutez la configuration IP à la carte réseau secondaire de chaque machine virtuelle :

    1. Sélectionnez la carte réseau secondaire que vous souhaitez configurer.
    
    2. Sélectionnez **Configurations IP**. Dans le volet suivant, en haut, sélectionnez **Ajouter**.

    3. Dans **Add IP configurations (Ajouter des configurations IP)** , ajoutez une seconde configuration IP à la carte réseau : 

        1. Entrez un nom pour la configuration IP secondaire. (Par exemple, pour VM1 et VM2, nommez la configuration IP **VM1NIC2-ipconfig2** et **VM2NIC2-ipconfig2**, respectivement.)

        2. Dans **Adresse IP privée**, pour **Allocation**, sélectionnez **Statique**.

        3. Sélectionnez **OK**.

Lorsque la seconde configuration IP de la carte réseau secondaire est terminée, elle s’affiche dans les paramètres **Configurations IP** de la carte réseau donnée.

### <a name="step-2-create-the-load-balancer"></a>Étape 2 : Créer l’équilibreur de charge

Créez votre équilibreur de charge pour la configuration :

1. Accédez au portail Azure à l’adresse https://portal.azure.com. Connectez-vous à votre compte Azure.

2. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**. Ensuite, sélectionnez **Créer**.

3. Dans **Créer un équilibreur de charge**, tapez le nom de votre équilibreur de charge. Dans ce scénario, nous utilisons le nom **mylb**.

4. Dans **Adresse IP publique**, créez une adresse IP publique nommée **PublicIP1**.

5. Dans **Groupe de ressources**, sélectionnez le groupe de ressources de vos machines virtuelles (par exemple, **contosofabrikam**). Sélectionnez l’emplacement de déploiement de votre équilibreur de charge, puis sélectionnez **OK**.

Le déploiement de l’équilibreur de charge commence. Cette opération peut prendre plusieurs minutes. Une fois déployé, l’équilibreur de charge s’affiche en tant que ressource dans votre groupe de ressources.

### <a name="step-3-configure-the-front-end-ip-pool"></a>Étape 3 : Configurer le pool d’adresses IP frontal

Pour chaque site web (contoso.com et fabrikam.com), configurez le pool d’adresses IP frontal de votre équilibreur de charge :

1. Dans le portail, cliquez sur **Plus de services**. Dans la zone de filtre, tapez **adresse IP publique**, puis sélectionnez **Adresses IP publiques**. Dans le volet suivant, en haut, sélectionnez **Ajouter**.

2. Configurez deux adresses IP publiques (**PublicIP1** et **PublicIP2**) pour les deux sites web (contoso et fabrikam) :

   1. Entrez le nom de votre adresse IP frontale.

   2. Dans **Groupe de ressources**, sélectionnez le groupe de ressources de vos machines virtuelles (par exemple, **contosofabrikam**).

   3. Dans **Emplacement**, sélectionnez le même emplacement que les machines virtuelles.

   4. Sélectionnez **OK**.

      Une fois les deux adresses IP publiques créées, elles s’affichent dans **Adresses IP publiques**.

3. <a name="step3-3"></a>Dans le portail, cliquez sur **Plus de services**. Dans la zone de filtre, tapez **équilibreur de charge**, puis sélectionnez **Équilibreur de charge**. 

4. Sélectionnez l’équilibreur de charge (**mylb**) auquel vous souhaitez ajouter le pool d’adresses IP frontal.

5. Sous **Paramètres**, sélectionnez **Configuration d’adresses IP frontales**. Dans le volet suivant, en haut, sélectionnez **Ajouter**.

6. Entrez le nom de votre adresse IP frontale (par exemple, **contosofe** ou **fabrikamfe**).

7. <a name="step3-7"></a>Sélectionnez **Adresse IP**. Dans **Choisir une adresse IP publique**, sélectionnez les adresses IP de votre serveur frontal (**PublicIP1** ou **PublicIP2**).

8. Créez la deuxième adresse IP frontale en répétant les <a href="#step3-3">étapes 3</a> à <a href="#step3-7">7</a> de cette section.

Une fois le pool frontal configuré, les adresses IP s’affichent dans les paramètres **de configuration IP du frontal** de votre équilibreur de charge. 
    
### <a name="step-4-configure-the-back-end-pool"></a>Étape 4 : Configurer le pool principal

Pour chaque site web (contoso.com et fabrikam.com), configurez le pool d’adresses IP principal de votre équilibreur de charge :
        
1. Dans le portail, cliquez sur **Plus de services**. Dans la zone de filtre, tapez **équilibreur de charge**, puis sélectionnez **Équilibreur de charge**.

2. Sélectionnez l’équilibreur de charge (**mylb**) auquel vous souhaitez ajouter le pool principal.

3. Sous **Paramètres**, sélectionnez **Backend Pools (Pools principaux)** . Entrez le nom de votre pool principal (par exemple, **contosopool** ou **fabrikampool**). Dans le volet suivant, en haut, sélectionnez **Ajouter**. 

4. Dans **Associated to (Associé à)** , sélectionnez **Availability set (Groupe à haute disponibilité)** .

5. Dans **Availability set (Groupe à haute disponibilité)** , sélectionnez **myAvailset**.

6. Ajoutez les configurations IP réseau cibles pour les deux machines virtuelles : 

    ![Configuration des pools principaux de l’équilibreur de charge](./media/load-balancer-multiple-ip/lb-backendpool.PNG)
    
    1. Dans **Machine virtuelle cible**, sélectionnez la machine virtuelle que vous souhaitez ajouter au pool principal (par exemple, **VM1** ou **VM2**).

    2. Dans **Configuration IP réseau**, sélectionnez la configuration IP de la carte réseau secondaire de la machine virtuelle que vous avez sélectionnée lors de l’étape précédente (par exemple, **VM1NIC2-ipconfig2** ou **VM2NIC2-ipconfig2**).

7. Sélectionnez **OK**.

Une fois le pool principal configuré, les adresses s’affichent dans les paramètres **Pool principal** de votre équilibreur de charge.

### <a name="step-5-configure-the-health-probe"></a>Étape 5 : Configurer la sonde d’intégrité

Configurez une sonde d’intégrité pour votre équilibreur de charge :

1. Dans le portail, cliquez sur **Plus de services**. Dans la zone de filtre, tapez **équilibreur de charge**, puis sélectionnez **Équilibreur de charge**.

2. Sélectionnez l’équilibreur de charge (**mylb**) auquel vous souhaitez ajouter la sonde d’intégrité.

3. Sous **Paramètres**, sélectionnez **Health Probe (Sonde d’intégrité)** . Dans le volet suivant, en haut, sélectionnez **Ajouter**. 

4. Entrez le nom de la sonde d’intégrité (par exemple, **HTTP**). Sélectionnez **OK**.

### <a name="step-6-configure-load-balancing-rules"></a>Étape 6 : Configurer les règles d’équilibrage de charge

Pour chaque site web (contoso.com et fabrikam.com), configurez les règles d’équilibrage de charge :
    
1. <a name="step6-1"></a>Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**. Dans le volet suivant, en haut, sélectionnez **Ajouter**. 

2. Dans **Nom**, entrez le nom de la règle d’équilibrage de charge (par exemple, **HTTPc** pour contoso.com ou **HTTPf** pour fabrikam.com).

3. Dans **Adresse IP du serveur frontal**, sélectionnez l’adresse IP frontale que vous avez précédemment créée (par exemple, **contosofe** ou **fabrikamfe**).

4. Dans **Port** et **Backend port (Port principal)** , conservez la valeur par défaut (**80**).

5. Dans **IP flottante (retour direct du serveur)** , sélectionnez **Désactivée**.

6. <a name="step6-6"></a>Sélectionnez **OK**.

7. Créez la deuxième règle d’équilibrage de charge en répétant les <a href="#step6-1">étapes 1</a> à <a href="#step6-6">6</a> de cette section.

Une fois les règles configurées, elles s’affichent dans les paramètres **Règles d’équilibrage de charge** de votre équilibreur de charge.

### <a name="step-7-configure-dns-records"></a>Étape 7 : Configurer les enregistrements DNS

La dernière étape consiste à configurer vos enregistrements de ressource DNS pour qu’ils pointent sur les adresses IP frontales respectives de votre équilibreur de charge. Vous pouvez héberger vos domaines dans Azure DNS. Pour plus d’informations sur l’utilisation d’Azure DNS avec un équilibrage de charge, voir [Utiliser Azure DNS avec d’autres services Azure](../dns/dns-for-azure-services.md).

## <a name="next-steps"></a>Étapes suivantes
- Pour en savoir plus sur la combinaison de services d’équilibrage de charge dans Azure, consultez [Utilisation des services d’équilibrage de charge dans Azure](../traffic-manager/traffic-manager-load-balancing-azure.md).
- Pour savoir comment gérer et dépanner l’équilibrage de charge à l’aide de différents types de journaux d’activité, consultez [Journaux d’activité Azure Monitor pour Azure Load Balancer](../load-balancer/load-balancer-monitor-log.md).
