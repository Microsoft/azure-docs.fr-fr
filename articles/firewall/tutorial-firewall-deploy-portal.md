---
title: 'Tutoriel : Déployer et configurer un pare-feu Azure à l’aide du portail Azure'
description: Ce didacticiel vous apprend à déployer et configurer un pare-feu Azure à l’aide du portail Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 07/15/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: e73f11ec178c067941ee33e02f37c96605460ee0
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658585"
---
# <a name="tutorial-deploy-and-configure-azure-firewall-using-the-azure-portal"></a>Tutoriel : Déployer et configurer un pare-feu Azure à l’aide du portail Azure

Le contrôle de l’accès réseau sortant est une partie importante d’un plan de sécurité réseau global. Par exemple, vous pouvez souhaiter limiter l’accès aux sites web. Vous pouvez aussi vouloir limiter l’accès à certaines adresses IP et à certains ports sortants.

Vous pouvez contrôler l’accès réseau sortant à partir d’un sous-réseau Azure à l’aide du Pare-feu Azure. Avec le Pare-feu Azure, vous pouvez configurer les éléments suivants :

* Règles d’application qui définissent des noms de domaine complets (FQDN) accessibles depuis un sous-réseau.
* Règles réseau qui définissent l’adresse source, le protocole, le port de destination et l’adresse de destination.

Le trafic réseau est soumis aux règles de pare-feu configurées lorsque vous routez votre trafic réseau vers le pare-feu en tant que sous-réseau de passerelle par défaut.

Pour ce tutoriel, vous devez créer un seul réseau virtuel simplifié avec deux sous-réseaux pour un déploiement facile.

Pour les déploiements de production, un [modèle Hub and Spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), dans lequel le pare-feu est dans son propre réseau virtuel, est recommandé. Les serveurs de la charge de travail se trouvent dans des réseaux virtuels appairés dans la même région avec un ou plusieurs sous-réseaux.

* **AzureFirewallSubnet** : le pare-feu est dans ce sous-réseau.
* **Workload-SN** : le serveur de la charge de travail est dans ce sous-réseau. Le trafic réseau de ce sous-réseau traverse le pare-feu.

![Tutoriel relatif à l’infrastructure réseau](media/tutorial-firewall-deploy-portal/tutorial-network.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer un environnement réseau de test
> * Déployer un pare-feu
> * Créer un itinéraire par défaut
> * Configurer une règle d’application pour autoriser l’accès à www.google.com
> * Configurer une règle de réseau pour autoriser l’accès aux serveurs DNS externes
> * Configurer une règle NAT pour autoriser la connexion d’un Bureau à distance au serveur de test
> * Tester le pare-feu

Si vous préférez, vous pouvez effectuer ce didacticiel en utilisant [Azure PowerShell](deploy-ps.md).

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="set-up-the-network"></a>Configurer le réseau

Tout d’abord, créez un groupe de ressources qui contiendra les ressources nécessaires pour déployer le pare-feu. Ensuite, créez un réseau virtuel, des sous-réseaux et un serveur de test.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Le groupe de ressources contient toutes les ressources utilisées dans ce didacticiel.

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans le menu du Portail Azure, sélectionnez **Groupes de ressources** ou recherchez et sélectionnez *Groupes de ressources* dans n’importe quelle page. Sélectionnez ensuite **Ajouter**.
3. Dans **Nom du groupe de ressources**, entrez *Test-FW-RG*.
4. Pour **Abonnement**, sélectionnez votre abonnement.
5. Pour **Emplacement du groupe de ressources**, sélectionnez un emplacement. Toutes les autres ressources que vous créez doivent se trouver dans le même emplacement.
6. Sélectionnez **Create** (Créer).

### <a name="create-a-vnet"></a>Créer un réseau virtuel

Ce réseau virtuel contient trois sous-réseaux.

> [!NOTE]
> La taille du sous-réseau AzureFirewallSubnet est /26. Pour plus d’informations sur la taille du sous-réseau, consultez le [FAQ Pare-feu Azure](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.
1. Sélectionnez **Mise en réseau** > **Réseau virtuel**.
2. Pour **Abonnement**, sélectionnez votre abonnement.
3. Pour **Groupe de ressources**, sélectionnez **Test-FW-RG**.
4. Pour **Nom**, entrez **Test-FW-VN**.
5. Pour **Région**, sélectionnez le même emplacement que celui utilisé précédemment.
6. Sélectionnez **Suivant : adresses IP**.
7. Pour **Espace d’adressage IPv4**, entrez **10.0.0.0/16**.
8. Sous **Sous-réseau**, sélectionnez **Par défaut**.
9. Sous **Nom du sous-réseau**, entrez **AzureFirewallSubnet**. Le pare-feu se trouvera dans ce sous-réseau et le nom du sous-réseau **doit** être AzureFirewallSubnet.
10. Pour **Plage d’adresses**, tapez **10.0.1.0/26**.
11. Sélectionnez **Enregistrer**.

   Créez ensuite un sous-réseau pour le serveur de la charge de travail.

1. Sélectionnez **Ajouter un sous-réseau**.
4. Pour **Nom du sous-réseau**, entrez **Workload-SN**.
5. Pour **Plage d’adresses du sous-réseau**, entrez **10.0.2.0/24**.
6. Sélectionnez **Ajouter**.
7. Sélectionnez **Vérifier + créer**.
8. Sélectionnez **Create** (Créer).

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

À présent, créez la machine virtuelle de charge de travail et placez-la dans le sous-réseau **Workload-SN**.

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.
2. Sélectionnez **Compute**, puis **Machine virtuelle**.
3. Sélectionnez **Windows Server 2016 Datacenter** dans la liste Proposé.
4. Entrez ces valeurs pour la machine virtuelle :

   |Paramètre  |Valeur  |
   |---------|---------|
   |Resource group     |**Test-FW-RG**|
   |Nom de la machine virtuelle     |**Srv-Work**|
   |Région     |Identique au précédent|
   |Image|Windows Server 2019 Datacenter|
   |Nom d’utilisateur de l’administrateur     |Entrez un nom d’utilisateur.|
   |Mot de passe     |Entrez un mot de passe.|

4. Sous **Règles des ports d’entrée**, pour **Ports d’entrée publics**, sélectionnez **Aucune**.
6. Acceptez les autres valeurs par défaut, puis sélectionnez **Suivant : Disques**.
7. Acceptez les disques par défaut, puis sélectionnez **Suivant : Mise en réseau**.
8. Assurez-vous que **Test-FW-VN** est sélectionné pour le réseau virtuel et que le sous-réseau est **Workload-SN**.
9. Pour **Adresse IP publique**, sélectionnez **Aucune**.
11. Acceptez les autres valeurs par défaut, puis sélectionnez **Suivant : Gestion**.
12. Sélectionnez **Désactivé** pour désactiver les diagnostics de démarrage. Acceptez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer**.
13. Vérifiez les paramètres sur la page de récapitulatif, puis sélectionnez **Créer**.

## <a name="deploy-the-firewall"></a>Déployer le pare-feu

Déployez le pare-feu dans le réseau virtuel.

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.
2. Tapez **pare-feu** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Pare-feu**, puis **Créer**.
4. Sur la page **Créer un pare-feu**, utilisez le tableau suivant pour configurer le pare-feu :

   |Paramètre  |Valeur  |
   |---------|---------|
   |Abonnement     |\<your subscription\>|
   |Resource group     |**Test-FW-RG** |
   |Nom     |**Test-FW01**|
   |Emplacement     |Sélectionnez le même emplacement que celui utilisé précédemment|
   |Choisir un réseau virtuel     |**Utiliser l’existant** : **Test-FW-VN**|
   |Adresse IP publique     |**Ajouter nouveau**<br>**Nom** : **fw-pip**|

5. Sélectionnez **Revoir + créer**.
6. Passez en revue le récapitulatif, puis sélectionnez **Créer** pour créer le pare-feu.

   Le déploiement nécessite quelques minutes.
7. Une fois le déploiement terminé, accédez au groupe de ressources **Test-FW-RG**, puis sélectionnez le pare-feu **Test-FW01**.
8. Notez les adresses IP privée et publique du pare-feu. Vous les utiliserez plus tard.

## <a name="create-a-default-route"></a>Créer un itinéraire par défaut

Pour le sous-réseau **Workload-SN**, configurez l’itinéraire sortant par défaut pour qu’il traverse le pare-feu.

1. Dans le menu du Portail Azure, sélectionnez **Tous les services** ou recherchez et sélectionnez *Tous les services* dans n’importe quelle page.
2. Sous **Mise en réseau**, sélectionnez **Tables d’itinéraires**.
3. Sélectionnez **Ajouter**.
4. Pour **Nom**, entrez **Firewall-route**.
5. Pour **Abonnement**, sélectionnez votre abonnement.
6. Pour **Groupe de ressources**, sélectionnez **Test-FW-RG**.
7. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
8. Sélectionnez **Create** (Créer).
9. Sélectionnez **Actualiser**, puis la table d’itinéraires **Firewall-route**.
10. Sélectionnez **Sous-réseaux**, puis **Associer**.
11. Sélectionnez **Réseau virtuel** > **Test-FW-VN**.
12. Pour **Sous-réseau**, sélectionnez **Workload-SN**. Veillez à ne sélectionner que le sous-réseau **Workload-SN** pour cette route, sinon votre pare-feu ne fonctionnera pas correctement.

13. Sélectionnez **OK**.
14. Sélectionnez **Itinéraires**, puis **Ajouter**.
15. Pour **Nom de l’itinéraire**, tapez **fw-dg**.
16. Pour **Préfixe d’adresse**, entrez **0.0.0.0/0**.
17. Pour **Type de tronçon suivant**, sélectionnez **Appliance virtuelle**.

    Le Pare-feu Azure est en réalité un service managé, mais l’appliance virtuelle fonctionne dans ce cas.
18. Pour **Adresse de tronçon suivant**, entrez l’adresse IP privée pour le pare-feu que vous avez notée précédemment.
19. Sélectionnez **OK**.

## <a name="configure-an-application-rule"></a>Configurer une règle d’application

Il s’agit de la règle d’application qui autorise un accès sortant à `www.google.com`.

1. Ouvrez **Test-FW-RG**, puis sélectionnez le pare-feu **Test-FW01**.
2. Sur la page **Test-FW01**, sous **Paramètres**, sélectionnez **Règles**.
3. Sélectionnez l’onglet **Collection de règles d’application**.
4. Sélectionnez **Ajouter une collection de règles d’application**.
5. Pour **Nom**, entrez **App-Coll01**.
6. Pour **Priorité**, entrez **200**.
7. Pour **Action**, sélectionnez **Autoriser**.
8. Sous **Règles**, **Noms de domaine complets cibles**, pour **Nom**, tapez **Allow-Google**.
9. Pour **Type de source**, sélectionnez **Adresse IP**.
10. Pour **Source**, tapez **10.0.2.0/24**.
11. Pour **Protocol:port**, entrez **http, https**.
12. Pour **Noms de domaine complets cibles**, entrez **`www.google.com`** .
13. Sélectionnez **Ajouter**.

Le Pare-feu Azure comprend un regroupement de règles intégré pour les noms de domaine complets d’infrastructure qui sont autorisés par défaut. Ces noms de domaine complets sont spécifiques à la plateforme et ne peuvent pas être utilisés à d’autres fins. Pour plus d’informations, consultez [Noms de domaine complets d’infrastructure](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Configurer une règle de réseau

Il s’agit de la règle de réseau qui autorise un accès sortant à deux adresses IP sur le port 53 (DNS).

1. Sélectionnez l’onglet **Collection de règles de réseau**.
2. Sélectionnez **Ajouter une collection de règles de réseau**.
3. Pour **Nom**, entrez **Net-Coll01**.
4. Pour **Priorité**, entrez **200**.
5. Pour **Action**, sélectionnez **Autoriser**.
6. Sous **Règles**, **Adresses IP**, pour **Nom**, tapez **Allow-DNS**.
7. Pour **Protocole**, sélectionnez **UDP**.
9. Pour **Type de source**, sélectionnez **Adresse IP**.
1. Pour **Source**, tapez **10.0.2.0/24**.
2. Pour **Type de destination**, sélectionnez **Adresse IP**.
3. Pour **Adresse de destination**, tapez **209.244.0.3,209.244.0.4**

   Il s’agit de serveurs DNS publics gérés par CenturyLink.
1. Pour **Ports de destination**, entrez **53**.
2. Sélectionnez **Ajouter**.

## <a name="configure-a-dnat-rule"></a>Configurer une règle DNAT

Cette règle vous permet de connecter un Bureau à distance à la machine virtuelle Srv-Work par le biais du pare-feu.

1. Sélectionnez l’onglet **Collection de règles NAT**.
2. Sélectionnez **Ajouter une collection de règles NAT**.
3. Pour **Nom**, entrez **rdp**.
4. Pour **Priorité**, entrez **200**.
5. Sous **Règles**, pour **Nom**, entrez **rdp-nat**.
6. Pour **Protocole**, sélectionnez **TCP**.
7. Pour **Type de source**, sélectionnez **Adresse IP**.
8. Pour **Source**, tapez * *\** _.
9. Pour _*Adresse de destination**, tapez l’adresse IP publique du pare-feu.
10. Pour **Ports de destination**, tapez **3389**.
11. Pou **Adresse traduite**, saisissez l’adresse IP privée de **Srv-work**.
12. Dans le champ **Port traduit**, tapez **3389**.
13. Sélectionnez **Ajouter**.


### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>Modifier les adresses DNS principales et secondaires de l’interface réseau **Srv-Work**

À des fins de test dans ce tutoriel, configurez les adresses DNS principales et secondaires du serveur. Ceci n’est pas obligatoire pour le pare-feu Azure.

1. Dans le menu du Portail Azure, sélectionnez **Groupes de ressources** ou recherchez et sélectionnez *Groupes de ressources* dans n’importe quelle page. Sélectionnez le groupe de ressources **Test-FW-RG**.
2. Sélectionnez l’interface réseau de la machine virtuelle **Srv-Work**.
3. Sous **Paramètres**, sélectionnez **Serveurs DNS**.
4. Sous **Serveurs DNS**, sélectionnez **Personnalisé**.
5. Entrez **209.244.0.3** dans la zone de texte **Ajouter un serveur DNS**, et **209.244.0.4** dans la zone de texte suivante.
6. Sélectionnez **Enregistrer**.
7. Redémarrez la machine virtuelle **Srv-Work**.

## <a name="test-the-firewall"></a>Tester le pare-feu

Testez maintenant le pare-feu pour vérifier qu’il fonctionne comme prévu.

1. Connectez un Bureau à distance à l’adresse IP publique du pare-feu et connectez-vous à la machine virtuelle **Srv-Work**. 
3. Ouvrez Internet Explorer et accédez à `https://www.google.com`.
4. Sélectionnez **OK** > **Fermer** sur les alertes de sécurité d’Internet Explorer.

   La page d’accueil Google doit s’afficher.

5. Accédez à `https://www.microsoft.com`.

   Vous devriez être bloqué par le pare-feu.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

* Vous pouvez accéder au nom de domaine complet autorisé, mais pas à d’autres.
* Vous pouvez résoudre les noms DNS à l’aide du serveur DNS externe configuré.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez garder vos ressources de pare-feu pour le prochain didacticiel, ou, si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources **Test-FW-RG** pour supprimer toutes les ressources associées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Superviser les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)