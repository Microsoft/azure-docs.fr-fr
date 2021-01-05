---
title: Utiliser Application Gateway pour protéger vos applications web sur Azure VMware Solution
description: Configurez Azure Application Gateway pour exposer en toute sécurité vos applications web s’exécutant sur Azure VMware Solution.
ms.topic: how-to
ms.date: 11/13/2020
ms.openlocfilehash: 02e439989c985354dbe06fa3e231d5daf7099d70
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94628990"
---
# <a name="use-azure-application-gateway-to-protect-your-web-apps-on-azure-vmware-solution"></a>Utiliser Application Gateway pour protéger vos applications web sur Azure VMware Solution

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) est un équilibreur de charge de trafic web de couche 7 qui vous permet de gérer le trafic vers vos applications web. Il est proposé dans Azure VMware Solution v1.0 et v2.0. Les deux versions ont été testées avec des applications web s’exécutant sur Azure VMware Solution.

Ces fonctionnalités sont les suivantes : 
- Affinité de session basée sur les cookies
- Routage basé sur des URL
- Pare-feu d’applications web (WAF)

Pour obtenir la liste complète des fonctionnalités, consultez [Fonctionnalités Azure Application Gateway](../application-gateway/features.md). 

Cet article explique comment utiliser Application Gateway devant une batterie de serveurs web pour protéger une application web s’exécutant sur Azure VMware Solution. 

## <a name="topology"></a>Topologie
Le diagramme illustre comment Application Gateway est utilisé pour protéger des machines virtuelles Azure IaaS, des groupes de machines virtuelles identiques Azure ou des serveurs locaux. Application Gateway traite les machines virtuelles Azure VMware Solution comme des serveurs locaux. 

![Diagramme illustrant comment Application Gateway protège des machines virtuelles Azure IaaS, des groupes de machines virtuelles identiques Azure ou des serveurs locaux.](media/protect-azure-vmware-solution-with-application-gateway/app-gateway-protects.png)

> [!IMPORTANT]
> Azure Application Gateway est actuellement la seule méthode prise en charge pour exposer des applications web s’exécutant sur des machines virtuelles Azure VMware Solution.

Le diagramme illustre le scénario de test utilisé pour valider Application Gateway avec des applications web Azure VMware Solution.

:::image type="content" source="media/hub-spoke/azure-vmware-solution-second-level-traffic-segmentation.png" alt-text="Diagramme illustrant le scénario de test utilisé pour valider Application Gateway avec des applications web Azure VMware Solution." border="false":::

L’instance Application Gateway est déployée sur le hub dans un sous-réseau dédié. Elle a une adresse IP publique Azure. L’activation de la protection DDoS Standard pour le réseau virtuel est recommandée. Le serveur web est hébergé sur un cloud privé Azure VMware Solution derrière des routeurs NSX T0 et T1. Azure VMware Solution utilise le service [Global Reach d’ExpressRoute](../expressroute/expressroute-global-reach.md) pour activer la communication avec le hub et les systèmes locaux.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif.
- Cloud privé Azure VMware Solution déployé et en cours d’exécution.

## <a name="deployment-and-configuration"></a>Déploiement et configuration

1. Dans le portail Azure, recherchez **Application Gateway**, puis sélectionnez **Créer une passerelle Application Gateway**.

2. Fournissez les informations de base, comme dans l’illustration suivante, puis sélectionnez **Suivant : Serveurs front-end>** . 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/create-app-gateway.png" alt-text="Capture d’écran montrant la page Créer une passerelle Application Gateway dans le portail Azure.":::

3. Choisissez le type d’adresse IP frontale. Pour l’option Publique, choisissez une adresse IP publique ou créez-en une. Sélectionnez **Suivant : Back-end>** .

    > [!NOTE]
    > Seules les références (SKU) Standard et de pare-feu d’applications web (WAF) sont prises en charge pour des serveurs frontaux privés.

4. Ajoutez un pool back-end des machines virtuelles qui s’exécutent sur l’infrastructure Azure VMware Solution. Fournissez les détails des serveurs web qui s’exécutent sur le cloud privé Azure VMware Solution, puis sélectionnez **Ajouter**.  Ensuite, sélectionnez **Next: Configuration>** .

1. Sous l’onglet **Configuration**, sélectionnez **Ajouter une règle d’acheminement**.

6. Sous l’onglet **Écouteur**, entrez les détails de l’écouteur. Si le protocole HTTPS est sélectionné, vous devez fournir un certificat, soit à partir d’un fichier PFX, soit à partir d’un certificat Azure Key Vault existant. 

7. Sélectionnez l’onglet **Cibles de back-end**, puis le pool principal créé précédemment. Pour le champ **Paramètres HTTP**, sélectionnez **Ajouter nouveau**.

8. Configurez les paramètres HTTP. Sélectionnez **Ajouter**.

9. Si vous souhaitez configurer des règles basées sur le chemin d’accès, sélectionnez **Ajouter plusieurs cibles pour créer une règle basée sur le chemin**. 

10. Ajoutez une règle basée sur un chemin d’accès, et sélectionnez **Ajouter**. Répétez cette opération pour ajouter d’autres règles basées sur un chemin d’accès. 

11. Lorsque vous avez fini d’ajouter des règles basées sur un chemin d’accès, sélectionnez de nouveau **Ajouter**. Ensuite, sélectionnez **Suivant : Étiquettes>** . 

12. Ajoutez des étiquettes, puis sélectionnez **Suivant : Vérifier + créer>** .

13. Une validation est exécutée sur votre passerelle Application Gateway. En cas de réussite, sélectionnez **Créer** pour déployer.

## <a name="configuration-examples"></a>Exemples de configuration

Dans cette section, vous allez apprendre à configurer Application Gateway avec des machines virtuelles Azure VMware Solution comme pools back-end pour les cas d’utilisation suivants : 

- [Hébergement de plusieurs sites](#hosting-multiple-sites)
- [Routage par URL](#routing-by-url)

### <a name="hosting-multiple-sites"></a>Hébergement de plusieurs sites

Cette procédure vous montre comment définir des pools d’adresses back-end à l’aide de machines virtuelles s’exécutant sur un cloud privé Azure VMware Solution sur une passerelle Application Gateway existante. 

>[!NOTE]
>Cette procédure part du principe que vous avez plusieurs domaines. Donc, nous allons utiliser les exemples www.contoso.com et www.fabrikam.com.


1. Dans votre cloud privé, créez deux pools de machines virtuelles différents. L’un représente Contoso et le deuxième Fabrikam. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool.png" alt-text="Capture d’écran montrant le résumé des détails d’un serveur web dans VSphere Client.":::

    Pour illustrer ce tutoriel, nous avons utilisé Windows Server 2016 avec le rôle Internet Information Services (IIS) installé. Une fois les machines virtuelles installées, exécutez les commandes PowerShell suivantes pour configurer IIS sur chaque machine virtuelle. 

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

2. Dans une instance Application Gateway existante, sélectionnez **Pools back-end** dans le menu à gauche, sélectionnez **Ajouter** et entrez les détails des nouveaux pools. Sélectionnez **Ajouter** dans le volet droit.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-02.png" alt-text="Capture d’écran de la page Pools back-end pour l’ajout de pools back-end." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-02.png":::

3. Dans la section **Écouteurs**, créez un écouteur pour chaque site web. Entrez les détails de chaque écouteur, puis sélectionnez **Ajouter**.

4. À gauche, sélectionnez **Paramètres HTTP**, puis choisissez **Ajouter** dans le volet gauche. Renseignez les détails pour créer un paramètre HTTP, puis sélectionnez **Enregistrer**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-03.png" alt-text="Capture d’écran de la page Paramètres HTTP pour créer un nouveau paramètre HTTP." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-03.png":::

5. Créez les règles dans la section **Règles** du menu à gauche. Associez chaque règle à l’écouteur correspondant. Sélectionnez **Ajouter**.

6. Configurez le pool principal et les paramètres HTTP correspondants. Sélectionnez **Ajouter**.

7. Testez la connexion. Ouvrez votre navigateur préféré et accédez aux différents sites web hébergés dans votre environnement Azure VMware Solution, par exemple http://www.fabrikam.com.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-multi-backend-pool-07.png" alt-text="Capture d’écran de la page du navigateur montrant la réussite du test de la connexion.":::

### <a name="routing-by-url"></a>Routage par URL

Cette procédure vous montre comment définir des pools d’adresses back-end à l’aide de machines virtuelles s’exécutant sur un cloud privé Azure VMware Solution sur une passerelle Application Gateway existante. Il permet ensuite de définir des règles d’acheminement qui font en sorte que le trafic web arrive sur les serveurs voulus dans les pools.

1. Dans votre cloud privé, créez un pool de machines virtuelles pour représenter la batterie de serveurs web. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool.png" alt-text="Capture d’écran de la page dans VMSphere Client montrant le résumé d’une autre machine virtuelle.":::

    Le rôle Windows Server 2016 avec IIS installé a été utilisé pour illustrer ce tutoriel. Une fois les machines virtuelles installées, exécutez les commandes PowerShell suivantes pour configurer IIS pour chaque machine virtuelle (tutoriel). 

    La première machine virtuelle, contoso-web-01, hébergera le site web principal.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    Add-Content -Path C:\inetpub\wwwroot\Default.htm -Value $($env:computername)
    ```

    La deuxième machine virtuelle, contoso-web-02, hébergera le site d’images.
 
    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "images" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\images\test.htm -Value $($env:computername)
    ```

    La troisième machine virtuelle, contoso-web-03, hébergera le site de vidéos.

    ```powershell
    Install-WindowsFeature -Name Web-Server
    New-Item -Path "C:\inetpub\wwwroot\" -Name "video" -ItemType "directory"
    Add-Content -Path C:\inetpub\wwwroot\video\test.htm -Value $($env:computername)
    ```

2. Ajoutez trois nouveaux pools back-end dans une instance Application Gateway existante. 

   1. Sélectionnez **Pools principaux** dans le menu de gauche. 
   1. Sélectionnez **Ajouter** et entrez les détails du premier pool, **contoso-web**. 
   1. Ajoutez une machine virtuelle en tant que cible. 
   1. Sélectionnez **Ajouter**. 
   1. Répétez ce processus pour **contoso-images** et **contoso-video**, en ajoutant une seule machine virtuelle comme cible. 

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-02.png" alt-text="Capture d’écran de la page Pools back-end montrant l’ajout de trois nouveaux pools back-end." lightbox="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-02.png":::

3. Dans la section **Écouteurs**, créez un écouteur de type De base utilisant le port 8080.

4. Dans le volet de navigation à gauche, sélectionnez **Paramètres HTTP**, puis choisissez **Ajouter** dans le volet gauche. Renseignez les détails pour créer un paramètre HTTP, puis sélectionnez **Enregistrer**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-04.png" alt-text="Capture d’écran de la page Ajouter un paramètre HTTP montrant la configuration des paramètres HTTP.":::

5. Créez les règles dans la section **Règles** du menu à gauche. Associez chaque règle à l’écouteur créé précédemment. Configurez ensuite le pool principal principal et les paramètres HTTP. Sélectionnez **Ajouter**.

    :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-07.png" alt-text="Capture d’écran de la page Ajouter une règle d’acheminement pour configurer des règles d’acheminement pour une cible back-end.":::

6. Testez la configuration. Accédez à la passerelle d’application sur le portail Azure, puis, dans la section **Vue d’ensemble**, copiez l’adresse IP publique. 

   1. Ouvrez une nouvelle fenêtre de navigateur et entrez l’URL `http://<app-gw-ip-address>:8080`. 

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-08.png" alt-text="Capture d’écran de la page du navigateur montrant la réussite du test de la configuration.":::

   1. Définissez l’URL sur `http://<app-gw-ip-address>:8080/images/test.htm`.

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-09.png" alt-text="Capture d’écran d’un autre test réussi avec la nouvelle URL.":::

   1. Définissez de nouveau l’URL sur `http://<app-gw-ip-address>:8080/video/test.htm`.

      :::image type="content" source="media/protect-azure-vmware-solution-with-application-gateway/app-gateway-url-route-backend-pool-10.png" alt-text="Capture d’écran d’un test réussi avec l’URL finale.":::

## <a name="next-steps"></a>Étapes suivantes

Pour d’autres exemples de configuration, consultez la [documentation d’Azure Application Gateway](../application-gateway/index.yml).