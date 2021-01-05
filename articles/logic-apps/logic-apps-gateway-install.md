---
title: Installer la passerelle de données locale
description: Télécharger et installer la passerelle de données locale pour pouvoir accéder aux données locales avec Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: arthii, logicappspm
ms.topic: article
ms.date: 05/15/2020
ms.openlocfilehash: a36b9d20fa20df56ec53e090976ea86e689ac74b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91322510"
---
# <a name="install-on-premises-data-gateway-for-azure-logic-apps"></a>Installer une passerelle de données locale pour Azure Logic Apps

Pour pouvoir vous [connecter à des sources de données locales à partir d’Azure Logic Apps](../logic-apps/logic-apps-gateway-connection.md), téléchargez et installez la [passerelle de données locale](https://aka.ms/on-premises-data-gateway-installer) sur un ordinateur local. La passerelle fonctionne comme un pont permettant un transfert et un chiffrement de données rapides entre les sources de données locales et vos applications logiques. Vous pouvez utiliser la même installation de passerelle pour d’autres services cloud, comme Power BI, Power Automate, Power Apps et Azure Analysis Services. Pour plus d’informations sur l’utilisation de la passerelle avec ces services, consultez ces articles :

* [Passerelle de données locale Microsoft Power Automate](/power-automate/gateway-reference)
* [Passerelle de données locale Microsoft Power BI](/power-bi/service-gateway-onprem)
* [Passerelle de données locale Microsoft Power Apps](/powerapps/maker/canvas-apps/gateway-reference)
* [Passerelle de données locale Azure Analysis Services](../analysis-services/analysis-services-gateway.md)

Cet article explique comment télécharger, installer et configurer votre passerelle de données locale afin de pouvoir accéder à des sources de données locales à partir d’Azure Logic Apps. Vous pouvez également en savoir plus sur le [fonctionnement de la passerelle de données](#gateway-cloud-service) plus loin dans cette rubrique. Pour obtenir des informations détaillées sur la passerelle, voir [Qu’est-ce qu’une passerelle de données locale ?](/data-integration/gateway/service-gateway-onprem). Pour automatiser les tâches d'installation et de gestion de la passerelle, visitez la galerie PowerShell pour accéder aux [cmdlets DataGateway PowerShell](https://www.powershellgallery.com/packages/DataGateway/3000.15.15).

<a name="requirements"></a>

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Si vous n’avez pas de compte Azure avec un abonnement, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

  * Votre compte Azure doit être un compte professionnel ou scolaire, du type `username@contoso.com`. Vous ne pouvez pas utiliser des comptes Azure B2B (invité) ou des comptes Microsoft personnels, comme @hotmail.com ou @outlook.com.

    > [!NOTE]
    > Si vous avez souscrit une offre Microsoft 365 sans fournir votre adresse e-mail professionnelle, votre adresse de connexion peut se présenter comme ceci : `username@domain.onmicrosoft.com`. Votre compte est stocké dans un locataire Azure AD. Dans la plupart des cas, le nom d’utilisateur principal (UPN) de votre compte Azure est identique à votre adresse e-mail.

    Pour utiliser un [abonnement standard Visual Studio](https://visualstudio.microsoft.com/vs/pricing/) associé à un compte Microsoft, commencez par [créer un locataire Azure AD](../active-directory/develop/quickstart-create-new-tenant.md) ou utilisez l’annuaire par défaut. Ajoutez un utilisateur avec un mot de passe à l’annuaire, puis donnez-lui accès à votre abonnement Azure. Vous pourrez alors vous connecter au cours de l’installation de la passerelle avec ce nom d’utilisateur et ce mot de passe.

  * Votre compte Azure ne doit appartenir qu’à un seul [locataire ou annuaire Azure Active Directory (Azure AD)](../active-directory/fundamentals/active-directory-whatis.md#terminology). Vous devez utiliser le même compte Azure pour installer et administrer la passerelle sur votre ordinateur local.

  * Lorsque vous installez la passerelle, vous vous connectez avec votre compte Azure, qui relie l’installation de votre passerelle à votre compte Azure et uniquement à celui-ci. Il n’est pas possible de lier la même installation de passerelle à plusieurs comptes Azure ou locataires Azure AD.

  * Plus tard, sur le Portail Azure, vous devrez utiliser le même compte Azure pour créer une ressource de passerelle Azure reliée à votre installation de passerelle. Vous ne pouvez lier qu’une seule installation de passerelle et une seule ressource de passerelle Azure l’une à l’autre. Toutefois, votre compte Azure peut être lié à différentes installations de passerelle qui sont chacune associées à une ressource de passerelle Azure. Vos applications logiques peuvent alors utiliser cette ressource de passerelle dans les déclencheurs et les actions qui ont accès à des sources de données locales.

* Voici la configuration requise pour votre ordinateur local :

  **Configuration minimale requise**

  * .NET Framework 4.7.2
  * Version 64 bits de Windows 7 ou Windows Server 2008 R2 (ou version ultérieure)

  **Configuration recommandée**

  * Processeur 8 cœurs
  * 8 Go de mémoire
  * Version 64 bits de Windows Server 2012 R2 ou ultérieure
  * Stockage SSD pour la mise en file d’attente

  > [!NOTE]
  > La passerelle ne prend pas en charge Windows Server Core.

* **Considérations associées**

  * Installez la passerelle de données locale sur un ordinateur local, et non sur un contrôleur de domaine. Vous ne devez pas nécessairement installer la passerelle sur le même ordinateur que votre source de données. Vous n’avez besoin que d’une seule passerelle pour toutes vos sources de données : il n’est donc pas nécessaire d’installer une passerelle pour chaque source de données.

    > [!TIP]
    > Pour réduire la latence, vous pouvez installer la passerelle le plus près possible de votre source de données, ou sur le même ordinateur, en supposant que vous disposiez des autorisations nécessaires.

  * Installez la passerelle sur un ordinateur local branché à un réseau câblé, connecté à Internet, toujours allumé et qui ne se met pas en veille. Sinon, la passerelle ne peut pas s’exécuter et les performances peuvent se dégrader sur un réseau sans fil.

  * Si vous prévoyez d’utiliser l’authentification Windows, veillez à installer la passerelle sur un ordinateur membre du même environnement Active Directory que vos sources de données.

  * La région que vous sélectionnez pour l’installation de votre passerelle est le même emplacement que celui que vous devez sélectionner quand vous créez plus tard la ressource de passerelle Azure pour votre application logique. Par défaut, cette région est le même emplacement que votre locataire Azure AD qui gère votre compte Azure. Vous pouvez cependant changer l’emplacement lors de l’installation de la passerelle.

  * Si vous mettez à jour votre installation de passerelle, commencez par désinstaller votre passerelle actuelle pour obtenir une interface plus propre.

    Nous vous recommandons de vérifier que vous utilisez une version prise en charge. Microsoft publie une nouvelle mise à jour de la passerelle de données locale chaque mois, et prend actuellement en charge uniquement les six dernières versions de la passerelle de données locale. Si vous rencontrez des problèmes avec la version que vous utilisez, essayez d’opérer une [mise à niveau vers la version la plus récente](https://aka.ms/on-premises-data-gateway-installer), car celle-ci résoudra peut-être votre problème.

  * La passerelle a deux modes : le mode standard et le mode personnel, qui s’applique seulement à Power BI. Vous ne pouvez pas avoir plusieurs passerelles s’exécutant dans le même mode sur le même ordinateur.

  * Azure Logic Apps prend en charge les opérations de lecture et d’écriture par le biais de la passerelle. Toutefois, ces opérations ont des [limites quant à la taille de leur charge utile](/data-integration/gateway/service-gateway-onprem#considerations).

<a name="install-gateway"></a>

## <a name="install-data-gateway"></a>Installer une passerelle de données

1. [Téléchargez et exécutez le programme d’installation de passerelle sur un ordinateur local](https://aka.ms/on-premises-data-gateway-installer).

1. Passez en revue la configuration minimale requise, conservez le chemin d’installation par défaut, acceptez les conditions d’utilisation, puis sélectionnez **Installer**.

   ![Passer en revue les conditions requises et accepter les conditions d’utilisation](./media/logic-apps-gateway-install/review-and-accept-terms-of-use.png)

1. Une fois la passerelle installée avec succès, fournissez l’adresse e-mail pour votre compte Azure, puis choisissez **Se connecter**, par exemple :

   ![Connectez-vous avec un compte professionnel ou scolaire](./media/logic-apps-gateway-install/sign-in-gateway-install.png)

   L’installation de votre passerelle ne peut être liée qu’à un seul compte Azure.

1. Sélectionnez **Inscrivez une nouvelle passerelle sur cet ordinateur** > **Suivant**. Cette étape inscrit l’installation de votre passerelle auprès du [service cloud de passerelle](#gateway-cloud-service).

   ![Inscrire la passerelle sur l’ordinateur local](./media/logic-apps-gateway-install/register-gateway-local-computer.png)

1. Fournissez les informations suivantes pour votre installation de passerelle :

   * Un nom de passerelle unique sur votre locataire Azure AD
   * La clé de récupération (qui doit comporter au moins huit caractères) que vous voulez utiliser
   * Confirmation de votre clé de récupération

   ![Fournir des informations pour l’installation de la passerelle](./media/logic-apps-gateway-install/gateway-name-recovery-key.png)

   > [!IMPORTANT]
   > Sauvegardez votre clé de récupération dans un endroit sûr. Vous avez besoin de cette clé si vous voulez changer l’emplacement, déplacer, récupérer ou reprendre l’installation d’une passerelle.

   Notez l’option pour **Ajouter à un cluster de passerelle existant**, que vous sélectionnez quand vous installez des passerelles supplémentaires pour des [scénarios de haute disponibilité](#high-availability).

1. Vérifiez la région du service cloud de passerelle et [l’instance de messagerie Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) qui est utilisée par votre installation de passerelle. Par défaut, cette région est le même emplacement que votre locataire Azure AD pour votre compte Azure.

   ![Confirmer la région pour le service de passerelle et le service bus](./media/logic-apps-gateway-install/confirm-gateway-region.png)

1. Pour accepter la région par défaut, sélectionnez **Configurer**. Cependant, si la région par défaut n’est pas celle qui est la plus proche de vous, vous pouvez changer la région.

   *Pourquoi modifier la région de votre programme d’installation de passerelle ?*

   Par exemple, pour réduire la latence, vous pouvez modifier la région de votre passerelle vers la même région que votre application logique. Ou bien, vous pouvez sélectionner la région la plus proche de votre source de données locale. Votre *ressource de passerelle sur Azure* et votre application logique peuvent se trouver dans des emplacements différents.

   1. À côté de la région actuelle, sélectionnez **Changer la région**.

      ![Modifier la région de passerelle actuelle](./media/logic-apps-gateway-install/change-gateway-service-region.png)

   1. Sur la page suivante, ouvrez la liste **Sélectionner une région**, sélectionnez la région de votre choix, puis sélectionnez **Terminé**.

      ![Sélectionner une autre région pour le service de passerelle](./media/logic-apps-gateway-install/select-region-gateway-install.png)

1. Passez en revue les informations de la fenêtre de confirmation finale. Cet exemple utilise le même compte pour Logic Apps, Power BI, Power Apps et Power Automate : la passerelle est donc disponible pour tous ces services. Quand vous êtes prêt, sélectionnez **Fermer**.

   ![Confirmer les informations de la passerelle de données](./media/logic-apps-gateway-install/finished-gateway-default-location.png)

1. Maintenant, [créez la ressource Azure pour votre installation de passerelle](../logic-apps/logic-apps-gateway-connection.md).

## <a name="check-or-adjust-communication-settings"></a>Vérifier ou ajuster les paramètres de communication

La passerelle de données locale dépend de la [messagerie Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md) pour la connectivité au cloud et établit les connexions sortantes correspondantes à la région Azure associée de la passerelle. Si votre environnement de travail nécessite que le trafic passe par un proxy ou un pare-feu pour accéder à Internet, cette restriction peut empêcher la passerelle de données locale de se connecter au service cloud de passerelle et à la messagerie Azure Service Bus. La passerelle a plusieurs paramètres de communication que vous pouvez ajuster. Pour plus d’informations, consultez les rubriques suivantes :

* [Ajuster les paramètres de communication pour la passerelle de données locale](/data-integration/gateway/service-gateway-communication)
* [Configurer les paramètres de proxy pour la passerelle de données locale](/data-integration/gateway/service-gateway-proxy)

<a name="high-availability"></a>

## <a name="high-availability-support"></a>Prise en charge de la haute disponibilité

Pour éviter les points de défaillance uniques pour l’accès aux données locales, vous pouvez avoir plusieurs installations de passerelle (mode standard uniquement) avec chacune sur un ordinateur différent, et les configurer en tant que cluster ou en tant que groupe. Ainsi, si la passerelle principale n’est pas disponible, les demandes de données sont routées vers la deuxième passerelle, etc. Comme vous ne pouvez installer qu’une seule passerelle standard sur un ordinateur, vous devez installer chaque passerelle supplémentaire qui se trouve dans le cluster sur un autre ordinateur. Tous les connecteurs qui fonctionnent avec la passerelle de données locale prennent en charge la haute disponibilité.

* Vous devez déjà disposer d’au moins une passerelle d’installation avec le même compte Azure servant de passerelle principale et de clé de récupération pour cette installation.

* Votre passerelle principale doit fonctionner sous la mise à jour de passerelle datant de novembre 2017 ou une version plus récente.

Une fois que vous avez configuré votre passerelle principale, quand vous accédez à installer une autre passerelle, sélectionnez **Ajouter à un cluster de passerelle existant**, sélectionnez la passerelle principale, qui est la première passerelle que vous avez installée, et spécifiez la clé de récupération pour cette passerelle. Pour plus d’informations, consultez [Clusters à haute disponibilité pour la passerelle de données locale](/data-integration/gateway/service-gateway-install#add-another-gateway-to-create-a-cluster).

<a name="update-gateway-installation"></a>

## <a name="change-location-migrate-restore-or-take-over-existing-gateway"></a>Modifier l’emplacement, migrer, restaurer ou contrôler une passerelle existante

Si vous devez modifier l’emplacement de votre passerelle, déplacer votre programme d’installation de passerelle vers un nouvel ordinateur, récupérer une passerelle endommagée ou contrôler une passerelle existante, vous avez besoin de la clé de récupération qui a été fournie lors de l’installation de la passerelle.

> [!NOTE]
> Avant de restaurer la passerelle sur l’ordinateur où la passerelle d’origine est installée, vous devez d’abord désinstaller la passerelle sur cet ordinateur. Cette action déconnecte l’ancienne passerelle.
> Si vous supprimez un cluster de passerelle pour un service cloud, vous ne pouvez pas restaurer ce cluster.

1. Exécutez le programme d’installation de passerelle sur l’ordinateur où se trouve la passerelle existante.

1. Une fois le programme d’installation ouvert, connectez-vous avec le même compte Azure que celui utilisé pour installer la passerelle.

1. Sélectionnez **Migrez, restaurez ou remplacez une passerelle existante** > **Suivant**, par exemple :

   ![Sélectionnez « Migrer, restaurer ou contrôler une passerelle existante »](./media/logic-apps-gateway-install/migrate-recover-take-over-gateway.png)

1. Sélectionnez parmi les clusters et les passerelles disponibles, puis entrez la clé de récupération pour la passerelle sélectionnée, par exemple :

   ![Sélectionner une passerelle et fournir une clé de récupération](./media/logic-apps-gateway-install/select-existing-gateway.png)

1. Pour changer la région, sélectionnez **Changer la région** et sélectionnez la nouvelle région.

1. Quand vous êtes prêt, sélectionnez **Configurer** pour pouvoir terminer votre tâche.

## <a name="tenant-level-administration"></a>Administration au niveau du locataire

Pour obtenir une visibilité de toutes les passerelles de données locales dans un locataire Azure AD, les administrateurs généraux de ce locataire peuvent se connecter au [Centre d’administration Power Platform](https://powerplatform.microsoft.com) en tant qu’administrateur du locataire et sélectionner l’option **Passerelles de données**. Pour plus d’informations, consultez [Administration au niveau du locataire pour la passerelle de données locale](/data-integration/gateway/service-gateway-tenant-level-admin).

<a name="restart-gateway"></a>

## <a name="restart-gateway"></a>Redémarrer la passerelle

Par défaut, l’installation de la passerelle sur votre ordinateur local s’exécute avec un compte de service Windows nommé « Service de passerelle de données locale ». Cependant, l’installation de la passerelle utilise le nom `NT SERVICE\PBIEgwService` pour les informations d’identification de son compte « Ouvrir une session en tant que » et dispose des autorisations « Ouvrir une session en tant que service ».

> [!NOTE]
> Votre compte de service Windows diffère du compte utilisé pour la connexion à des sources de données locales, et du compte Azure que vous utilisez quand vous vous connectez à des services cloud.

Comme tout autre service Windows, vous pouvez démarrer et arrêter la passerelle de plusieurs façons. Pour plus d’informations, consultez [Redémarrer une passerelle de données locale](/data-integration/gateway/service-gateway-restart).

<a name="gateway-cloud-service"></a>

## <a name="how-the-gateway-works"></a>Fonctionnement de la passerelle

Les utilisateurs au sein de votre organisation peuvent accéder aux données locales pour lesquelles ils disposent déjà d’un accès autorisé. Toutefois, pour que ces utilisateurs puissent se connecter à votre source de données locale, vous devez installer et configurer une passerelle de données locale. En règle générale, la personne qui installe et configure une passerelle est un administrateur. Ces actions peuvent nécessiter des autorisations d’administrateur du serveur ou des connaissances spéciales sur vos serveurs locaux.

La passerelle facilite une communication plus rapides et plus sûre en coulisses. Cette communication circule entre un utilisateur dans le cloud, le service cloud de passerelle et votre source de données locale. Le service cloud de passerelle chiffre et stocke les informations d’identification de votre source de données et les détails de la passerelle. Le service achemine également les requêtes et leurs résultats entre l’utilisateur, la passerelle et votre source de données locale.

La passerelle fonctionne avec les pare-feux et utilise uniquement des connexions sortantes. L’ensemble du trafic est généré sous forme de trafic sortant sécurisé en provenance de l’agent de passerelle. La passerelle envoie les données des sources locales sur des canaux chiffrés via la [messagerie Azure Service Bus](../service-bus-messaging/service-bus-messaging-overview.md). Ce service bus crée un canal entre la passerelle et le service d’appel, mais ne stocke aucune donnée. Toutes les données qui transitent via la passerelle sont chiffrées.

![Architecture pour la passerelle de données locale](./media/logic-apps-gateway-install/how-on-premises-data-gateway-works-flow-diagram.png)

> [!NOTE]
> Selon le service cloud, il se peut que vous deviez configurer une source de données pour la passerelle.

Ces étapes décrivent ce qu’il se produit quand vous interagissez avec un élément connecté à une source de données locale :

1. Le service cloud crée une requête, ainsi que les informations d’identification chiffrées pour la source de données. Le service envoie ensuite la requête et les informations d’identification à la file d’attente de passerelle pour traitement.

1. Le service cloud de passerelle analyse la requête et envoie (push) celle-ci à la messagerie Azure Service Bus.

1. La messagerie Azure Service Bus envoie les requêtes en attente à la passerelle.

1. La passerelle reçoit la requête, déchiffre les informations d’identification et les utilise pour se connecter à une ou plusieurs sources de données.

1. La passerelle envoie la requête à la source de données pour exécution.

1. Les résultats sont envoyés de la source de données vers la passerelle, puis au service cloud de passerelle. Le service cloud de passerelle utilise ensuite les résultats.

### <a name="authentication-to-on-premises-data-sources"></a>Authentification auprès de sources de données locales

Des informations d’identification stockées sont utilisées pour la connexion de la passerelle aux sources de données locales. Quel que soit l’utilisateur, la passerelle utilise les informations d’identification stockées pour se connecter. Il peut y avoir des exceptions d’authentification pour des services spécifiques, tels que DirectQuery et LiveConnect pour Analysis Services dans Power BI.

### <a name="azure-active-directory-azure-ad"></a>Azure Active Directory (Azure AD)

Les services de cloud computing Microsoft utilisent [Azure AD](../active-directory/fundamentals/active-directory-whatis.md) pour authentifier les utilisateurs. Un locataire Azure AD contient des noms d’utilisateur et des groupes de sécurité. En règle générale, l’adresse e-mail que vous utilisez pour la connexion est celle du nom d’utilisateur principal (UPN) de votre compte.

### <a name="what-is-my-upn"></a>Qu’est-ce que mon UPN ?

Si vous n’êtes pas administrateur de domaine, vous ne connaissez peut-être pas votre UPN. Pour trouver l’UPN de votre compte, exécutez la commande `whoami /upn` à partir de votre station de travail. Le résultat qui ressemble à une adresse e-mail est l’UPN de votre compte de domaine local.

### <a name="synchronize-an-on-premises-active-directory-with-azure-ad"></a>Synchroniser un annuaire Active Directory local avec Azure AD

L’UPN de vos comptes Active Directory locaux doit être identique à celui des comptes Azure AD. Par conséquent, assurez-vous que chaque compte Active Directory local correspond à votre compte Azure AD. Les services cloud ne connaissent que des comptes faisant partie d’Azure AD. Vous n’avez donc pas besoin d’ajouter de compte à votre Active Directory local. Si le compte n’existe pas dans Azure AD, vous ne pouvez pas l’utiliser.

Voici comment faire correspondre vos comptes Active Directory locaux avec Azure AD.

* Ajoutez des comptes manuellement à Azure AD.

  Créez un compte dans le Portail Azure ou dans le Centre d’administration Microsoft 365. Assurez-vous que le nom du compte correspond à l’UPN du compte Active Directory local.

* Synchronisez les comptes locaux avec votre locataire Azure AD à l’aide de l’outil Azure Active Directory Connect.

  L’outil Azure AD Connect fournit des options pour la synchronisation d’annuaires et la configuration de l’authentification. Ces options incluent la synchronisation du hachage de mot de passe, l’authentification directe et la fédération. Si vous n’êtes pas administrateur de locataire ou administrateur de domaine local, demandez à votre administrateur informatique de configurer Azure AD Connect. Azure AD Connect s’assure que votre UPN Azure AD correspond à votre UPN Active Directory local. Cette correspondance est utile si vous utilisez des connexions actives Analysis Services avec Power BI ou des fonctionnalités d’authentification unique (SSO).

  > [!NOTE]
  > La synchronisation des comptes avec l’outil Azure AD Connect a pour effet de créer des comptes dans votre locataire Azure AD.

<a name="faq"></a>

## <a name="faq-and-troubleshooting"></a>FAQ et résolution des problèmes

* [Questions fréquentes (FAQ) sur la passerelle de données locale](/data-integration/gateway/service-gateway-onprem-faq)
* [Résoudre des problèmes de passerelle de données locale](/data-integration/gateway/service-gateway-tshoot)
* [Surveiller et optimiser les performances de la passerelle](/data-integration/gateway/service-gateway-performance)

## <a name="next-steps"></a>Étapes suivantes

* [Connexion à des données locales à partir d’applications logiques](../logic-apps/logic-apps-gateway-connection.md)
* [Fonctionnalités d’intégration d'entreprise](../logic-apps/logic-apps-enterprise-integration-overview.md)
* [Connecteurs pour Azure Logic Apps](../connectors/apis-list.md)
