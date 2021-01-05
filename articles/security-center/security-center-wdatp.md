---
title: Utilisation de la licence Microsoft Defender for Endpoint incluse dans Azure Security Center
description: Découvrez Microsoft Defender for Endpoint et déployez-le à partir d’Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/20/2020
ms.author: memildin
ms.openlocfilehash: 5645a2e0449a1929421f9444bf645f08ad0525e9
ms.sourcegitcommit: 192f9233ba42e3cdda2794f4307e6620adba3ff2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96296809"
---
# <a name="protect-your-endpoints-with-security-centers-integrated-edr-solution-microsoft-defender-for-endpoint"></a>Protéger vos points de terminaison avec la solution EDR intégrée de Security Center : Microsoft Defender for Endpoint

Microsoft Defender for Endpoint est une solution holistique de sécurité des points de terminaison dans le cloud. Ses principales fonctionnalités sont les suivantes :

- Gestion et évaluation des vulnérabilités basées sur les risques 
- Réduction de la surface d’attaque
- Protection basée sur le comportement et gérée par le cloud
- Détection de point de terminaison et réponse (EDR)
- Investigation et correction automatiques
- Services de chasse gérés

> [!TIP]
> Lancé à l’origine sous le nom **Windows Defender ATP**, ce produit de détection de point de terminaison et réponse (EDR) a été renommé en 2019 en **Microsoft Defender ATP**.
>
> À l’occasion d’Ignite 2020, nous avons lancé la [suite Microsoft Defender XDR](https://www.microsoft.com/security/business/threat-protection) et ce composant EDR a été renommé **Microsoft Defender for Endpoint**.


## <a name="availability"></a>Disponibilité

| Aspect                          | Détails                                                                                                                                                                                                                                                                                                       |
|---------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| État de sortie :                  | Disponibilité générale (GA)                                                                                                                                                                                                                                                                                      |
| Prix :                        | Nécessite [Azure Defender pour les serveurs](security-center-pricing.md)                                                                                                                                                                                                                                             |
| Plateformes prises en charge :            | ![Oui](./media/icons/yes-icon.png) Machines Azure exécutant Windows<br>![Oui](./media/icons/yes-icon.png) Machines Azure Arc exécutant Windows|
| Versions de Windows prises en charge :  | Defender for Endpoint est intégré à Windows 10 1703 (et versions ultérieures) et Windows Server 2019.<br>Security Center prend en charge la détection sur Windows Server 2016, 2012 R2 et 2008 R2 SP1.<br>La supervision des points de terminaison de serveur qui utilise cette intégration a été désactivée pour les clients Office 365 GCC. |
| Rôles et autorisations obligatoires : | Pour activer/désactiver l'intégration : **Administrateur de la sécurité** ou **Propriétaire**<br>Pour afficher des alertes MDATP dans Security Center : **Lecteur de sécurité**, **Lecteur**, **Contributeur du groupe de ressources**, **Propriétaire du groupe de ressources**, **Administrateur de la sécurité**, **Propriétaire de l’abonnement** ou **Contributeur de l’abonnement**                         |
| Clouds :                         | ![Oui](./media/icons/yes-icon.png) Clouds commerciaux.<br>![Non](./media/icons/no-icon.png) Clients Cloud de la communauté du secteur public exécutant des charges de travail dans des clouds Azure mondiaux<br>![Oui](./media/icons/yes-icon.png) Gouvernement des États-Unis<br>![Non](./media/icons/no-icon.png) Chine Gov, autres Gov                                                        |
|                                 |                                                                                                                                                                                                                                                                                                               |


## <a name="microsoft-defender-for-endpoint-features-in-security-center"></a>Fonctionnalités de Microsoft Defender for Endpoint dans Security Center

Microsoft Defender for Endpoint fournit les éléments suivants :

- **Capteurs de détection des violations avancés**. Les capteurs de Defender for Endpoint pour machines Windows collectent un large éventail de signaux comportementaux.

- **Détection des violations basée sur des analyses dans le cloud**. Defender for Endpoint s’adapte rapidement aux menaces changeantes. Elle utilise l’analytique avancée et le Big Data. Il est amplifié par la puissance d’Intelligent Security Graph avec des signaux à travers Windows, Azure et Office pour détecter les menaces inconnues. Il fournit des alertes actionnables et vous permet de réagir rapidement.

- **Informations sur les menaces**. Defender for Endpoint génère des alertes quand il identifie les outils, les techniques et les procédures de l’attaquant. Il utilise les données générées par les chasseurs de menaces de Microsoft et les équipes de sécurité, complétées par les renseignements fournis par les partenaires.

En intégrant Defender for Endpoint à Security Center, vous bénéficierez des capacités supplémentaires suivantes :

- **Intégration automatisée**. Security Center active automatiquement le capteur Microsoft Defender for Endpoint pour tous les serveurs Windows supervisés par Security Center. Sauf pour ceux qui exécutent Windows Server 2019, et qui doivent être intégrés via un script local, un objet stratégie de groupe (GPO) ou [Microsoft Endpoint Configuration Manager](https://docs.microsoft.com/mem/configmgr/) (anciennement SCCM).

- **Volet unique**. La console Security Center affiche des alertes Microsoft Defender for Endpoint. Pour approfondir vos recherches, utilisez les pages du portail de Microsoft Defender for Endpoint, où vous verrez des informations supplémentaires telles que l’arborescence du processus d’alerte et le graphique d’incident. Vous pouvez également voir une chronologie détaillée de la machine, qui indique tous les comportements pour un historique pouvant s’étendre sur six mois.

    :::image type="content" source="./media/security-center-wdatp/microsoft-defender-security-center.png" alt-text="Centre de sécurité de Microsoft Defender for Endpoint" lightbox="./media/security-center-wdatp/microsoft-defender-security-center.png":::

## <a name="microsoft-defender-for-endpoint-tenant-location"></a>Emplacement du locataire Microsoft Defender for Endpoint

Quand vous utilisez Azure Security Center pour superviser vos serveurs, un locataire Microsoft Defender for Endpoint est créé automatiquement. Les données collectées par Defender for Endpoint sont stockées dans la zone géographique du locataire tel qu’il est identifié lors de l’approvisionnement. Les données des clients, sous forme pseudonymisée, peuvent également être stockées dans des systèmes de stockage et de traitement centralisés aux États-Unis. 

Une fois que vous avez configuré l’emplacement, vous ne pouvez plus le modifier. Si vous avez besoin de déplacer des données vers un autre emplacement, contactez le Support Microsoft pour réinitialiser le locataire.


## <a name="enabling-the-microsoft-defender-for-endpoint-integration"></a>Activation de l’intégration de Microsoft Defender for Endpoint

1. Activez **Azure Defender pour les serveurs**. Voir [Niveaux tarifaires d’Azure Security Center](security-center-pricing.md#enable-azure-defender).

    > [!NOTE]
    > Pour protéger vos machines Azure Arc, suivez les instructions dans [Démarrage rapide : Connecter une machine hybride à l’aide d’un serveur avec Azure Arc](../azure-arc/servers/learn/quick-enable-hybrid-vm.md).

1. Si vous avez déjà acheté et déployé des licences Microsoft Defender for Endpoint sur vos serveurs, supprimez-les à l’aide de la procédure décrite dans [Retirer des serveurs Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints#offboard-windows-servers).
1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement que vous souhaitez modifier.
1. Sélectionnez **Détection des menaces**.
1. Sélectionnez **Autoriser Microsoft Defender for Endpoint à accéder à mes données**, puis **Enregistrer**.

    :::image type="content" source="./media/security-center-wdatp/enable-integration-with-edr.png" alt-text="Activer l’intégration entre Azure Security Center et la solution EDR de Microsoft, Microsoft Defender for Endpoint":::

    Azure Security Center intégrera automatiquement vos serveurs à Microsoft Defender for Endpoint. L’intégration peut prendre jusqu’à 24 heures.


## <a name="access-the-microsoft-defender-for-endpoint-portal"></a>Accéder au portail Microsoft Defender for Endpoint

1. Vérifiez que le compte d’utilisateur dispose des autorisations nécessaires. [Plus d’informations](/windows/security/threat-protection/microsoft-defender-atp/assign-portal-access)

1. Vérifiez si vous avez un proxy ou un pare-feu qui bloque le trafic anonyme. Le capteur Defender for Endpoint se connecte à partir du contexte système, de sorte que le trafic anonyme doit être autorisé. Pour garantir un accès sans entraves au portail Defender for Endpoint, suivez les instructions mentionnées dans [Activer l’accès aux URL du service dans le serveur proxy](/windows/security/threat-protection/microsoft-defender-atp/configure-proxy-internet#enable-access-to-microsoft-defender-atp-service-urls-in-the-proxy-server).

1. Ouvrez le [portail Centre de sécurité Microsoft Defender](https://securitycenter.windows.com/). En savoir plus sur les fonctionnalités et les icônes du portail dans [Vue d’ensemble du portail Centre de sécurité Microsoft Defender](/windows/security/threat-protection/microsoft-defender-atp/portal-overview). 

## <a name="send-a-test-alert"></a>Envoyer une alerte de test

Pour générer une alerte de test Microsoft Defender for Endpoint inoffensive :

1. Créez un dossier « C:\test-MDATP-test ».
1. Utilisez le Bureau à distance pour accéder à une machine virtuelle Windows Server 2012 R2 ou à une machine virtuelle Windows Server 2016.
1. Ouvrez une fenêtre de ligne de commande.
1. A l'invite, copiez et exécutez la commande suivante. La fenêtre d’invite de commandes se ferme automatiquement.

    ```powershell
    powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden (New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-MDATP-test\\invoice.exe'); Start-Process 'C:\\test-MDATP-test\\invoice.exe'
    ```
    :::image type="content" source="./media/security-center-wdatp/generate-edr-alert.png" alt-text="Une fenêtre d’invite de commandes avec la commande pour générer une alerte de test.":::

1. Si la commande réussit, une nouvelle alerte s’affiche sur le tableau de bord Azure Security Center et sur le portail Microsoft Defender for Endpoint. L’alerte peut mettre quelques minutes à s’afficher.
1. Pour examiner l’alerte dans Security Center, accédez à **Alertes de sécurité** > **Ligne de commande PowerShell suspecte**.
1. Dans la fenêtre d’enquête, sélectionnez le lien d’accès au portail Microsoft Defender for Endpoint.


## <a name="faq-for-security-centers-integrated-microsoft-defender-for-endpoint"></a>FAQ relative à la solution Microsoft Defender for Endpoint intégrée de Security Center

### <a name="what-are-the-licensing-requirements-for-microsoft-defender-for-endpoint"></a>Quelles sont les conditions de licence pour Microsoft Defender for Endpoint ?

Defender for Endpoint est inclus sans coût supplémentaire avec **Azure Defender pour les serveurs**. Vous pouvez également l’acheter séparément pour 50 machines ou plus.


### <a name="how-do-i-switch-from-a-third-party-edr-tool"></a>Comment basculer dessus à partir d’un outil EDR tiers ?

Pour plus d’informations sur le basculement à partir d’une solution de point de terminaison non Microsoft, accédez à la documentation Microsoft Defender for Endpoint : [Vue d’ensemble de la migration](/windows/security/threat-protection/microsoft-defender-atp/switch-to-microsoft-defender-migration).
  


## <a name="next-steps"></a>Étapes suivantes

- [Plateformes et fonctionnalités prises en charge par Azure Security Center](security-center-os-coverage.md)
- [Gestion des suggestions de sécurité dans Azure Security Center](security-center-recommendations.md) : découvrez la façon dont les suggestions vous aident à protéger vos ressources Azure.