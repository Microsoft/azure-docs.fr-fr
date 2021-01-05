---
title: 'Tutoriel : Filtrer le trafic Internet entrant avec le Pare-feu Azure DNAT à l’aide du portail'
description: Ce didacticiel vous apprend à déployer et configurer un pare-feu Azure DNAT à l’aide du portail Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 71eda40abd38b4885b8e88085e338667b608902f
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655134"
---
# <a name="tutorial-filter-inbound-internet-traffic-with-azure-firewall-dnat-using-the-azure-portal"></a>Tutoriel : Filtrer le trafic Internet entrant avec le Pare-feu Azure DNAT à l’aide du Portail Azure

Vous pouvez configurer le Pare-feu DNAT (Azure Destination Network Address Translation) pour traduire et filtrer le trafic Internet entrant vers vos sous-réseaux. Lorsque vous configurez le DNAT, l’action de collection de règles NAT est définie sur **DNAT**. Chaque règle de la collection de règles NAT peut ensuite être utilisée pour traduire l’IP et le port publics de votre pare-feu en IP et port privés. Les règles DNAT ajoutent implicitement une règle de réseau correspondante pour autoriser le trafic traduit. Vous pouvez remplacer ce comportement en ajoutant explicitement une collection de règles de réseau avec des règles de refus correspondant au trafic traduit. Pour plus d’informations sur la logique de traitement des règles de Pare-feu Azure, consultez l’article [Logique de traitement des règles du service Pare-feu Azure](rule-processing.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer un environnement réseau de test
> * Déployer un pare-feu
> * Créer un itinéraire par défaut
> * Configurer une règle DNAT
> * Tester le pare-feu

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.



## <a name="create-a-resource-group"></a>Créer un groupe de ressources

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la page d’accueil du portail Azure, sélectionnez **Groupes de ressources**, puis sélectionnez **Ajouter**.
3. Pour **Nom du groupe de ressources**, entrez **RG-DNAT-Test**.
4. Pour **Abonnement**, sélectionnez votre abonnement.
5. Pour **Emplacement du groupe de ressources**, sélectionnez un emplacement. Toutes les ressources suivantes que vous créez doivent se trouver dans le même emplacement.
6. Sélectionnez **Create** (Créer).

## <a name="set-up-the-network-environment"></a>Configurer l’environnement réseau

Pour ce didacticiel, vous créez deux réseaux virtuels appairés :

- **VN-Hub** : le pare-feu est dans ce réseau virtuel.
- **VN-Spoke** : le serveur de la charge de travail est dans ce réseau virtuel.

Tout d’abord, créez les réseaux virtuels, puis appairez-les.

### <a name="create-the-hub-vnet"></a>Créer le réseau virtuel du hub

1. À partir de la page d’accueil du portail Azure, sélectionnez **Tous les services**.
2. Sous **Mise en réseau**, sélectionnez **Réseaux virtuels**.
3. Sélectionnez **Ajouter**.
4. Dans le champ **Nom**, saisissez **VN-Hub**.
5. Pour **Espace d’adressage**, entrez **10.0.0.0/16**.
6. Pour **Abonnement**, sélectionnez votre abonnement.
7. Pour **Groupe de ressources**, sélectionnez **Existant** puis **RG-DNAT-Test**.
8. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
9. Sous **Sous-réseau**, pour **Nom**, entrez **AzureFirewallSubnet**.

     Le pare-feu se trouvera dans ce sous-réseau et le nom du sous-réseau **doit** être AzureFirewallSubnet.
     > [!NOTE]
     > La taille du sous-réseau AzureFirewallSubnet est /26. Pour plus d’informations sur la taille du sous-réseau, consultez le [FAQ Pare-feu Azure](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

10. Pour **Plage d’adresses**, tapez **10.0.1.0/26**.
11. Conservez les autres paramètres par défaut, puis sélectionnez **Créer**.

### <a name="create-a-spoke-vnet"></a>Créer un réseau virtuel spoke

1. À partir de la page d’accueil du portail Azure, sélectionnez **Tous les services**.
2. Sous **Mise en réseau**, sélectionnez **Réseaux virtuels**.
3. Sélectionnez **Ajouter**.
4. Pour **Nom**, tapez **VN-Spoke**.
5. Pour **Espace d’adressage**, entrez **192.168.0.0/16**.
6. Pour **Abonnement**, sélectionnez votre abonnement.
7. Pour **Groupe de ressources**, sélectionnez **Existant** puis **RG-DNAT-Test**.
8. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
9. Sous **Sous-réseau**, tapez **SN-Workload** pour **Nom**.

    Le serveur se trouve dans ce sous-réseau.
10. Pour **Plage d’adresses**, entrez **192.168.1.0/24**.
11. Conservez les autres paramètres par défaut, puis sélectionnez **Créer**.

### <a name="peer-the-vnets"></a>Homologuer les réseaux virtuels

Maintenant, appairez les deux réseaux virtuels.

1. Sélectionnez le réseau virtuel **VN-Hub**.
2. Sous **Paramètres**, sélectionnez **Peerings**.
3. Sélectionnez **Ajouter**.
4. Tapez **Peer-HubSpoke** sous **Nom du peering de VN-Hub à VN-Spoke**.
5. Sélectionnez **VN-Spoke** pour le réseau virtuel.
6. Tapez **Peer-SpokeHub** sous **Nom du peering de VN-Spoke à VN-Hub**.
7. Pour **Autoriser le trafic transféré de VN-Spoke à VN-Hub**, sélectionnez **Activé**.
8. Sélectionnez **OK**.

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle de charge de travail, puis placez-la dans le sous-réseau **SN-Workload**.

1. Dans le menu du Portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Populaire**, sélectionnez **Windows Server 2016 Datacenter**.

**Concepts de base**

1. Pour **Abonnement**, sélectionnez votre abonnement.
1. Pour **Groupe de ressources**, sélectionnez **Existant** puis **RG-DNAT-Test**.
1. Sous **Nom de la machine virtuelle**, tapez **Srv-Workload**.
1. Pour **Région**, sélectionnez le même emplacement que celui utilisé précédemment.
1. Entrez un nom d’utilisateur et un mot de passe.
1. Sélectionnez **Suivant : Disques**.

**Disques**
1. Sélectionnez **Suivant : Mise en réseau**.

**Mise en réseau**

1. Pour **Réseau virtuel**, sélectionnez **VN-Spoke**.
2. Pour **Sous-réseau**, sélectionnez **SN-Workload**.
3. Pour **Adresse IP publique**, sélectionnez **Aucune**.
4. Pour **Ports d’entrée publics**, sélectionnez **Aucun**. 
2. Conservez les autres paramètres par défaut et sélectionnez **Suivant : Gestion**.

**Gestion**

1. Pour **Diagnostics de démarrage**, sélectionnez **Désactivé**.
1. Sélectionnez **Vérifier + créer**.

**Vérifier + créer**.

Passez en revue le récapitulatif, puis sélectionnez **Créer**. L’exécution de cette opération nécessite quelques minutes.

Une fois le déploiement terminé, notez l’adresse IP privée de la machine virtuelle. Elle servira ultérieurement pour configurer le pare-feu. Sélectionnez le nom de la machine virtuelle. Sous **Paramètres**, sélectionnez **Mise en réseau** pour trouver l’adresse IP privée.

## <a name="deploy-the-firewall"></a>Déployer le pare-feu

1. Dans la page d’accueil du portail, sélectionnez **Créer une ressource**.
2. Sélectionnez **Mise en réseau** et après **Recommandés**, sélectionnez **Afficher tout**.
3. Sélectionnez **Pare-feu**, puis **Créer**. 
4. Sur la page **Créer un pare-feu**, utilisez le tableau suivant pour configurer le pare-feu :

   |Paramètre  |Valeur  |
   |---------|---------|
   |Nom     |FW-DNAT-test|
   |Abonnement     |\<your subscription\>|
   |Resource group     |**Utiliser l’existant** : RG-DNAT-Test |
   |Emplacement     |Sélectionnez le même emplacement que celui utilisé précédemment|
   |Choisir un réseau virtuel     |**Utiliser l’existant** : VN-Hub|
   |Adresse IP publique     |**Créer un nouveau**. L’adresse IP publique doit être le type de référence (SKU) Standard.|

5. Sélectionnez **Revoir + créer**.
6. Passez en revue le récapitulatif, puis sélectionnez **Créer** pour créer le pare-feu.

   Le déploiement nécessite quelques minutes.
7. Une fois le déploiement terminé, accédez au groupe de ressources **RG-DNAT-Test**, puis sélectionnez le pare-feu **FW-DNAT-test**.
8. Notez l’adresse IP privée. Vous l’utiliserez plus tard lors de la création de l’itinéraire par défaut.

## <a name="create-a-default-route"></a>Créer un itinéraire par défaut

Pour le sous-réseau **SN-Workload**, vous devez configurer l’itinéraire sortant par défaut pour qu’il traverse le pare-feu.

1. À partir de la page d’accueil du portail Azure, sélectionnez **Tous les services**.
2. Sous **Mise en réseau**, sélectionnez **Tables d’itinéraires**.
3. Sélectionnez **Ajouter**.
4. Dans le champ **Nom**, saisissez **RT-FWroute**.
5. Pour **Abonnement**, sélectionnez votre abonnement.
6. Pour **Groupe de ressources**, sélectionnez **Existant** puis **RG-DNAT-Test**.
7. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
8. Sélectionnez **Create** (Créer).
9. Sélectionnez **Actualiser**, puis la table de route **RT-FWroute**.
10. Sélectionnez **Sous-réseaux**, puis **Associer**.
11. Sélectionnez **Réseau virtuel**, puis sélectionnez **VN-Spoke**.
12. Pour **Sous-réseau**, sélectionnez **SN-Workload**.
13. Sélectionnez **OK**.
14. Sélectionnez **Routes**, puis **Ajouter**.
15. Pour **Nom de l’itinéraire**, entrez **FW-DG**.
16. Pour **Préfixe d’adresse**, entrez **0.0.0.0/0**.
17. Pour **Type de tronçon suivant**, sélectionnez **Appliance virtuelle**.

    Le Pare-feu Azure est en réalité un service managé, mais l’appliance virtuelle fonctionne dans ce cas.
18. Pour **Adresse de tronçon suivant**, entrez l’adresse IP privée pour le pare-feu que vous avez notée précédemment.
19. Sélectionnez **OK**.

## <a name="configure-a-nat-rule"></a>Configurer une règle NAT

1. Ouvrez **RG-DNAT-Test**, puis sélectionnez le pare-feu **FW-DNAT-test**. 
2. Sur la page **FW-DNAT-test**, sous **Paramètres**, sélectionnez **Règles**. 
3. Sélectionnez **Ajouter une collection de règles NAT**. 
4. Dans le champ **Nom**, saisissez **RC-DNAT-01**. 
5. Pour **Priorité**, entrez **200**. 
6. Sous **Règles**, pour **Nom**, entrez **RL-01**.
7. Pour **Protocole**, sélectionnez **TCP**.
8. Pour **Adresses sources**, entrez *. 
9. Dans le champ **Adresses de destination**, tapez l’adresse IP publique du pare-feu. 
10. Pour **Ports de destination**, entrez **3389**. 
11. Dans le champ **Adresse traduite**, saisissez l’adresse IP privée de la machine virtuelle Srv-Workload. 
12. Dans le champ **Port traduit**, tapez **3389**. 
13. Sélectionnez **Ajouter**. 

## <a name="test-the-firewall"></a>Tester le pare-feu

1. Connectez un bureau distant à l’adresse IP publique du pare-feu. Vous devriez être connecté à la machine virtuelle **Srv-Workload**.
2. Fermez le bureau à distance.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez garder vos ressources de pare-feu pour le prochain didacticiel, ou, si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources **RG-DNAT-Test** pour supprimer toutes les ressources associées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Configurer un environnement réseau de test
> * Déployer un pare-feu
> * Créer un itinéraire par défaut
> * Configurer une règle DNAT
> * Tester le pare-feu

Ensuite, vous pouvez surveiller les journaux d’activité de Pare-feu Azure.

> [!div class="nextstepaction"]
> [Tutoriel : Superviser les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)