---
title: Passerelle de gestion des données pour Data Factory
description: Utilisez la passerelle de gestion des données dans Azure Data Factory pour déplacer vos données.
services: data-factory
documentationcenter: ''
author: nabhishek
manager: anandsub
ms.assetid: b9084537-2e1c-4e96-b5bc-0e2044388ffd
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: 94c1bed8d94f73bc7794037b307618f4c36c4518
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96450607"
---
# <a name="data-management-gateway"></a>Passerelle de gestion de données
> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Guide pratique pour créer et configurer le runtime d’intégration autohébergé dans Data Factory](../create-self-hosted-integration-runtime.md).

> [!NOTE]
> La passerelle de gestion des données a été renommée runtime d’intégration autohébergé.

La passerelle de gestion des données est un agent client que vous devez installer dans votre environnement local pour permettre la copie des données entre les magasins de données cloud et locaux. Les magasins de données locaux pris en charge par Data Factory sont répertoriés dans la section [Sources de données prises en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) .

Cet article vient compléter la procédure pas à pas de l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) . Dans cette procédure pas à pas, vous créez un pipeline qui utilise la passerelle qui déplace les données d’une base de données SQL Server vers un objet blob Azure. Cet article fournit des informations détaillées sur la passerelle de gestion des données.

Vous pouvez effectuer un scale-out d’une passerelle de gestion des données en associant plusieurs machines locales avec la passerelle. Vous pouvez monter en puissance une passerelle en augmentant le nombre de travaux de déplacement des données qui peuvent s’exécuter simultanément sur un nœud. Cette fonctionnalité est également disponible pour une passerelle logique à nœud unique. Consultez l’article [Mise à l’échelle de la passerelle de gestion des données dans Azure Data Factory](data-factory-data-management-gateway-high-availability-scalability.md) pour plus d’informations.

> [!NOTE]
> Actuellement, la passerelle prend en charge uniquement l’activité de copie et l’activité de procédure stockée dans Data Factory. Il n’est pas possible d’utiliser la passerelle à partir d’une activité personnalisée pour accéder à des sources de données locales.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble
### <a name="capabilities-of-data-management-gateway"></a>Fonctionnalités de la passerelle de gestion des données
La passerelle de gestion des données offre les fonctionnalités suivantes :

* Modélisation des sources de données locales et des sources de données sur le cloud au sein de la même fabrique de données, et déplacement des données.
* Un point unique de surveillance et de gestion avec une visibilité de l’état de la passerelle à partir de la page Data Factory.
* Gestion sécurisée de l’accès aux sources de données locales.
  * Aucune modification du pare-feu d’entreprise n’est requise. La passerelle établit uniquement des connexions HTTP sortantes pour l’accès à Internet.
  * Chiffrement des informations d’identification pour vos magasins de données locaux à l’aide de votre certificat.
* Déplacer les données efficacement : les données sont transférées en parallèle et résistent aux problèmes intermittents du réseau, grâce à la logique de nouvelle tentative automatique.

### <a name="command-flow-and-data-flow"></a>Flux de commandes et flux de données
Lorsque vous utilisez une activité de copie pour copier des données entre des machines locales et cloud, l’activité utilise une passerelle pour transférer les données à partir de la source de données locale vers le cloud et vice versa.

Voici le flux de données global et un résumé des étapes pour la copie à l’aide de la passerelle de données : ![Flux de données à l’aide de la passerelle](./media/data-factory-data-management-gateway/data-flow-using-gateway.png)

1. Le développeur des données crée une passerelle pour une fabrique de données Azure à l’aide du [portail Azure](https://portal.azure.com)ou d’une [applet de commande PowerShell](/powershell/module/az.datafactory/).
2. Le développeur de données crée un service lié pour un magasin de données local en spécifiant la passerelle. Dans le cadre de la configuration du service lié, le développeur des données utilise l’application de configuration des informations d’identification pour spécifier les types d’authentification et les informations d’identification. La boîte de dialogue de l’application de configuration des informations d’identification communique avec le magasin de données pour tester la connexion et la passerelle afin d’enregistrer les informations d’identification.
3. La passerelle chiffre les informations d’identification avec le certificat associé à la passerelle (fourni par le développeur des données) avant d’enregistrer les informations d’identification dans le cloud.
4. Le service Data Factory communique avec la passerelle pour la planification et la gestion des tâches via un canal de contrôle qui utilise une file d’attente Azure Service Bus partagée. Lorsqu’une tâche de l’activité de copie doit être lancée, Data Factory place en file d’attente la requête ainsi que les informations d’identification. La passerelle lance la tâche après avoir interrogé la file d'attente.
5. La passerelle déchiffre les informations d’identification avec le même certificat puis se connecte au magasin de données local avec le type d’authentification et les informations d’identification appropriés.
6. La passerelle copie les données d’un magasin local vers un stockage cloud, ou vice versa selon la configuration de l'activité de copie dans le pipeline de données. Pour cette étape, la passerelle communique directement avec le service de stockage basé sur le cloud comme le stockage d'objets blob Azure via un canal sécurisé (HTTPS).

### <a name="considerations-for-using-gateway"></a>Considérations relatives à l’utilisation de la passerelle
* Une seule instance de passerelle de gestion des données peut être utilisée pour plusieurs sources de données locales. Toutefois, **une seule instance de passerelle est liée à une instance d’Azure Data Factory** et ne peut pas être partagée avec une autre instance d’Azure Data Factory.
* Vous ne pouvez installer qu’**une seule instance de la passerelle de gestion de données** sur un même ordinateur. Si deux fabriques de données doivent accéder aux sources de données locales, vous devez installer des passerelles sur deux ordinateurs locaux. En d’autres termes, une passerelle est liée à une instance d’Azure Data Factory spécifique
* La **passerelle n’a pas besoin d’être sur la même machine que la source de données**. Toutefois, avoir une passerelle plus proche de la source de données réduit le temps de connexion de la passerelle à la source de données. Nous vous recommandons d’installer la passerelle sur un ordinateur différent de celui qui héberge la source de données locale. Lorsque la source de données et la passerelle se trouvent sur des machines différentes, la passerelle ne demande pas de ressources de la source de données.
* Vous pouvez avoir **plusieurs passerelles sur différents ordinateurs connectés à la même source de données locale**. Par exemple, vous pouvez avoir deux passerelles desservant deux fabriques de données, alors que la même source de données locale est enregistrée auprès des deux fabriques de données.
* Si une passerelle est déjà installée sur votre ordinateur desservant un scénario **Power BI**, installez une **passerelle distincte pour Azure Data Factory** sur un autre ordinateur.
* Vous devez utiliser la passerelle même lorsque vous utilisez **ExpressRoute**.
* Considérez votre source de données comme une source de données locale (derrière un pare-feu), même lorsque vous utilisez **ExpressRoute**. Utilisez la passerelle pour établir la connectivité entre le service et la source de données.
* Vous devez **utiliser la passerelle** même si la banque de données se trouve dans le cloud sur une **machine virtuelle IaaS Azure**.

## <a name="installation"></a>Installation
### <a name="prerequisites"></a>Prérequis
* Les versions de **système d’exploitation** prises en charge sont Windows 7, Windows 8/8.1, Windows 10, Windows Server 2008 R2, Windows Server 2012 et Windows Server 2012 R2. L’installation de la passerelle de gestion des données sur un contrôleur de domaine n’est pas prise en charge.
* .NET framework 4.5.1 ou version ultérieure est requis. Si vous installez la passerelle sur un ordinateur Windows 7, installez .NET Framework 4.5 ou une version ultérieure. Consultez [Configuration système requise pour .NET Framework](/dotnet/framework/get-started/system-requirements) pour plus d’informations.
* La **configuration** recommandée pour l’ordinateur de passerelle est la suivante : au moins 2 GHz, 4 cœurs, 8 Go de RAM et 80 Go d’espace disque.
* Si l’ordinateur hôte est en veille prolongée, la passerelle ne répond pas à la demande de données. Vous devez donc configurer un **plan de gestion de l’alimentation** approprié sur l’ordinateur avant d’installer la passerelle. L’installation de la passerelle ouvre un message si l’ordinateur est configuré pour la mise en veille prolongée.
* Vous devez être administrateur sur la machine pour installer et configurer la passerelle de gestion des données avec succès. Vous pouvez ajouter des utilisateurs supplémentaires au groupe Windows local **Utilisateurs de la passerelle de gestion des données**. Les membres de ce groupe sont en mesure d’utiliser l’outil **Gestionnaire de configuration de passerelle de gestion des données** pour configurer la passerelle.

Étant donné que l’activité de copie s’exécute selon une fréquence spécifique, l’utilisation des ressources (processeur, mémoire) sur l’ordinateur suit également le même modèle avec des pics et des baisses d’inactivité. L'utilisation des ressources dépend également en grande partie de la quantité de données déplacées. Lorsque plusieurs travaux sont en cours, vous constaterez une augmentation des ressources utilisées pendant les heures de pointe.

### <a name="installation-options"></a>Options d'installation
La passerelle de gestion des données peut être installée comme suit :

* En téléchargeant un package d’installation MSI à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=39717). Le fichier MSI peut également servir à mettre à niveau la passerelle de gestion des données existante vers la version la plus récente, en conservant tous les paramètres.
* En cliquant sur le lien **Télécharger et installer la passerelle de données** sous INSTALLATION MANUELLE ou **Installer directement sur cet ordinateur** sous INSTALLATION RAPIDE. Pour des instructions pas à pas sur l’utilisation de l’installation rapide, consultez l’article [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) . L’étape manuelle vous amène au centre de téléchargement. Les instructions pour télécharger et installer la passerelle à partir du centre de téléchargement se trouvent dans la section suivante.

### <a name="installation-best-practices"></a>Meilleures pratiques d’installation :
1. Définissez un plan d'alimentation sur l'ordinateur hôte de la passerelle afin d’empêcher la mise en veille prolongée. Si l’ordinateur hôte est en veille prolongée, la passerelle ne répond pas à la demande de données.
2. Sauvegardez le certificat associé à la passerelle.

### <a name="install-the-gateway-from-download-center"></a>Installer la passerelle à partir du Centre de téléchargement
1. Accédez à la [page de téléchargement de la passerelle de gestion des données Microsoft](https://www.microsoft.com/download/details.aspx?id=39717).
2. Cliquez sur **Télécharger**, sélectionnez la version **64 bits** (la version 32 bits n’est plus prise en charge), puis cliquez sur **Suivant**.
3. Exécutez le **MSI** directement ou enregistrez-le sur votre disque dur avant de l’exécuter.
4. Dans la page **Bienvenue**, sélectionnez une **langue** et cliquez sur **Suivant**.
5. **Acceptez** le Contrat de Licence Utilisateur Final et cliquez sur **Suivant**.
6. Sélectionnez le **dossier** d’installation de la passerelle, puis cliquez sur **Suivant**.
7. Dans la page **Prêt pour l’installation**, cliquez sur **Installer**.
8. Cliquez sur **Terminer** pour terminer l’installation.
9. Obtenez la clé à partir du portail Azure. Pour des instructions pas à pas, consultez la section suivante.
10. Sur la page **Enregistrer la passerelle** du **Gestionnaire de configuration de passerelle de gestion de données** en cours d’exécution sur votre machine, procédez comme suit :
    1. Collez la clé dans le texte.
    2. Éventuellement, cliquez sur **Afficher la clé de passerelle** pour afficher le texte de la clé.
    3. Cliquez sur **S'inscrire**.

### <a name="register-gateway-using-key"></a>Inscrire la passerelle à l’aide de la clé
#### <a name="if-you-havent-already-created-a-logical-gateway-in-the-portal"></a>Si vous n’avez pas déjà créé une passerelle logique dans le portail
Pour créer une passerelle dans le portail et obtenir la clé à partir de la page **Configurer**, suivez les étapes de la procédure pas à pas de l’article [Déplacement de données entre des sources locales et le cloud](data-factory-move-data-between-onprem-and-cloud.md).

#### <a name="if-you-have-already-created-the-logical-gateway-in-the-portal"></a>Si vous avez déjà créé la passerelle logique dans le portail
1. Dans le portail Azure, accédez à la page **Data Factory**, puis cliquez sur la vignette **Services liés**.

    ![Page Data Factory](media/data-factory-data-management-gateway/data-factory-blade.png)
2. Dans la page **Services liés**, sélectionnez la **passerelle** logique que vous avez créée dans le portail.

    ![passerelle logique](media/data-factory-data-management-gateway/data-factory-select-gateway.png)
3. Dans la page **Passerelle de données**, cliquez sur **Télécharger et installer la passerelle de données**.

    ![Lien de téléchargement dans le portail](media/data-factory-data-management-gateway/download-and-install-link-on-portal.png)
4. Dans la page **Configurer**, cliquez sur **Recréer une clé**. Dans le message d’avertissement, cliquez sur Oui après l’avoir lu attentivement.

    ![Bouton Recréer une clé](media/data-factory-data-management-gateway/recreate-key-button.png)
5. Cliquez sur le bouton Copier en regard de la clé. La clé est copiée dans le Presse-papiers.

    ![Copier la clé](media/data-factory-data-management-gateway/copy-gateway-key.png)

### <a name="system-tray-icons-notifications"></a>Icônes de la barre d’état système/notifications
L’illustration suivante représente certaines des icônes de barre d’état qui s’affichent.

![Icônes de la barre d’état système](./media/data-factory-data-management-gateway/gateway-tray-icons.png)

Si vous déplacez le curseur sur les icônes/messages de notification de la barre d’état système, vous afficherez des informations supplémentaires sur l’état d’opération de la passerelle/la progression de la mise à jour dans une fenêtre contextuelle.

### <a name="ports-and-firewall"></a>Ports et pare-feu
Vous devez porter votre attention sur deux pare-feu : le **pare-feu d’entreprise**, exécuté sur le routeur central de l’organisation et le **Pare-feu Windows**, configuré en tant que démon sur l’ordinateur local sur lequel la passerelle est installée.

![Pare-feu](./media/data-factory-data-management-gateway/firewalls2.png)

Au niveau du pare-feu d’entreprise, vous devez configurer les domaines et ports de sortie suivants :

| Noms de domaine | Ports | Description |
| --- | --- | --- |
| *.servicebus.windows.net |443 |Utilisé pour la communication avec le serveur principal du service Déplacement des données |
| *.core.windows.net |443 |Utilisé pour une copie intermédiaire à l’aide d’objets Blob Azure (si configuré)|
| *.frontend.clouddatahub.net |443 |Utilisé pour la communication avec le serveur principal du service Déplacement des données |
| *.servicebus.windows.net |9350-9354, 5671 |Service Bus Relay facultatif sur TCP utilisé par l’Assistant de copie |

Au niveau du pare-feu Windows, ces ports de sortie sont normalement activés. Sinon, vous pouvez configurer en conséquence les domaines et les ports sur l’ordinateur de passerelle.

> [!NOTE]
> 1. Selon votre source/vos récepteurs, vous devrez peut-être autoriser des domaines et des ports de sortie supplémentaires dans votre pare-feu d’entreprise/Windows.
> 2. Pour certaines bases de données cloud (par exemple : [Azure SQL Database](../../azure-sql/database/firewall-configure.md), [Azure Data Lake](../../data-lake-store/data-lake-store-secure-data.md#set-ip-address-range-for-data-access), etc.), vous aurez besoin d’autoriser l’adresse IP de la machine passerelle dans la configuration du pare-feu.
>
>

#### <a name="copy-data-from-a-source-data-store-to-a-sink-data-store"></a>Copier des données d’une banque de données source vers une banque de données de récepteur
Assurez-vous que les règles de pare-feu sont correctement activées sur le pare-feu d’entreprise, sur le pare-feu Windows de l’ordinateur de passerelle, ainsi que sur le magasin de données lui-même. Activer ces règles permet à la passerelle de se connecter correctement à la source et au récepteur. Activez les règles pour chaque magasin de données impliqué dans l’opération de copie.

Par exemple, pour copier à partir d’une **banque de données locale vers un récepteur Azure SQL Database ou un récepteur Azure Synapse Analytics**, effectuez les opérations suivantes :

* Autorisez le trafic **TCP** sortant sur le port **1433** pour le pare-feu Windows et le pare-feu d’entreprise.
* Configurez les paramètres de pare-feu du serveur SQL logique pour ajouter l’adresse IP de l’ordinateur de passerelle à la liste d’adresses IP autorisées.

> [!NOTE]
> Si votre pare-feu n’autorise pas le port de sortie 1433, la passerelle ne peut pas accéder directement à Azure SQL. Dans ce cas, vous pouvez utiliser la [copie intermédiaire](./data-factory-copy-activity-performance.md#staged-copy) sur SQL Database/SQL Managed Instance/SQL Azure DW. Dans ce scénario, vous auriez uniquement besoin du protocole HTTPS (port 443) pour le déplacement des données.
>
>

### <a name="proxy-server-considerations"></a>Considérations relatives aux serveurs proxy
Si votre environnement de réseau d’entreprise utilise un serveur proxy pour accéder à Internet, configurez la passerelle de gestion des données pour utiliser les bons paramètres de proxy. Vous pouvez définir le proxy lors de la phase initiale de l’enregistrement.

![Définir le serveur proxy lors de l’inscription](media/data-factory-data-management-gateway/SetProxyDuringRegistration.png)

La passerelle utilise le serveur proxy pour se connecter au service cloud. Cliquez sur le lien **Modifier** pendant l’installation initiale. La boîte de dialogue **Paramètre proxy** s’affiche.

![Définir le proxy avec le gestionnaire de configuration 1](media/data-factory-data-management-gateway/SetProxySettings.png)

Il existe trois options de configuration :

* **Ne pas utiliser de proxy** : la passerelle n’utilise pas explicitement de proxy pour se connecter aux services cloud.
* **Utiliser le proxy système** : la passerelle utilise le paramètre de proxy configuré dans diahost.exe.config et diawp.exe.config. Si aucun proxy n’est configuré dans diahost.exe.config et diawp.exe.config, la passerelle se connecte au service cloud directement sans passer par le proxy.
* **Utiliser un proxy personnalisé** : configurez les paramètres du proxy HTTP à utiliser pour la passerelle, au lieu d’utiliser les configurations dans diahost.exe.config et diawp.exe.config. L’adresse et le port sont requis. Le nom d’utilisateur et le mot de passe sont facultatifs, en fonction du paramètre d’authentification de votre proxy. Tous les paramètres sont chiffrés avec le certificat d’informations d’identification de la passerelle et stockés localement sur la machine hôte de passerelle.

Le service hôte de la passerelle de gestion des données redémarre automatiquement après avoir enregistré les paramètres de proxy mis à jour.

Une fois la passerelle enregistrée avec succès, si vous souhaitez afficher ou mettre à jour les paramètres de proxy, utilisez le gestionnaire de configuration de la passerelle de gestion des données.

1. Lancez le **Gestionnaire de configuration de la passerelle de gestion des données**.
2. Basculez vers l’onglet **Paramètres** .
3. Cliquez sur le lien **Modifier** dans la section **Serveur proxy HTTP** pour lancer la boîte de dialogue **Définir le proxy HTTP**.
4. Après avoir cliqué sur le bouton **Suivant** , vous verrez une boîte de dialogue d’avertissement demandant l’autorisation d’enregistrer les paramètres de proxy et de redémarrer le service hôte de la passerelle.

Vous pouvez afficher et mettre à jour le proxy HTTP à l’aide de l’outil Gestionnaire de Configuration.

![Définir le proxy avec le gestionnaire de configuration 2](media/data-factory-data-management-gateway/SetProxyConfigManager.png)

> [!NOTE]
> Si vous configurez un serveur proxy avec l’authentification NTLM, le service hôte de la passerelle s’exécute sous le compte du domaine. Si vous modifiez le mot de passe du compte du domaine ultérieurement, veillez à mettre à jour les paramètres de configuration pour le service et à redémarrer ce dernier en conséquence. En raison de cette exigence, nous vous conseillons d’utiliser un compte de domaine dédié qui ne nécessite pas de mettre à jour le mot de passe fréquemment pour accéder au serveur proxy.
>
>

### <a name="configure-proxy-server-settings"></a>Configurer les paramètres du serveur proxy
Si vous sélectionnez le paramètre **Utiliser le proxy système** pour le proxy HTTP, la passerelle utilise le paramètre du proxy dans diahost.exe.config et diawp.exe.config. Si aucun proxy n’est spécifié dans diahost.exe.config et diawp.exe.config, la passerelle se connecte au service cloud directement sans passer par le proxy. La procédure suivante fournit des instructions pour mettre à jour le fichier de configuration diahost.exe.config.

1. Dans l’Explorateur de fichiers, effectuez une copie de sauvegarde de *C:\\\\Program Files\\Microsoft Data Management Gateway\\2.0\\Shared\\diahost.exe.config* pour sauvegarder le fichier d’origine.
2. Lancez Notepad.exe en tant qu’administrateur, puis ouvrez le fichier texte *C:\\\\Program Files\\Microsoft Data Management Gateway\\2.0\\Shared\\diahost.exe.config*. La balise par défaut pour system.net apparaît dans le code suivant :

    ```
    <system.net>
        <defaultProxy useDefaultCredentials="true" />
    </system.net>
    ```

    Vous pouvez ensuite ajouter les détails du serveur proxy comme illustré dans l’exemple suivant :

    ```
    <system.net>
        <defaultProxy enabled="true">
            <proxy bypassonlocal="true" proxyaddress="http://proxy.domain.org:8888/" />
        </defaultProxy>
    </system.net>
    ```

    Vous pouvez ajouter des propriétés supplémentaires à l’intérieur de la balise de proxy pour spécifier les paramètres requis comme scriptLocation. Reportez-vous à la page de syntaxe [&lt;proxy&gt;, élément (paramètres réseau)](/dotnet/framework/configure-apps/file-schema/network/proxy-element-network-settings) .

    ```
    <proxy autoDetect="true|false|unspecified" bypassonlocal="true|false|unspecified" proxyaddress="uriString" scriptLocation="uriString" usesystemdefault="true|false|unspecified "/>
    ```
3. Enregistrez le fichier de configuration à l’emplacement d’origine, puis redémarrez le service hôte de passerelle de gestion des données, qui relève les modifications. Pour redémarrer le service, utilisez l’applet de services du panneau de configuration, ou allez dans le **Gestionnaire de configuration de passerelle de gestion des données**, cliquez sur le bouton **Arrêter le service**, puis sur **Démarrer le service**. Si le service ne démarre pas, il est probable qu’une syntaxe de balise XML incorrecte ait été ajoutée dans le fichier de configuration d’application que vous avez modifié.

> [!IMPORTANT]
> N’oubliez pas de mettre à jour diahost.exe.config **et** diawp.exe.config.

Outre ces points, vous devez également vérifier que Microsoft Azure figure dans la liste autorisée de votre entreprise. Vous pouvez télécharger la liste des adresses IP Microsoft Azure valides à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=41653).

#### <a name="possible-symptoms-for-firewall-and-proxy-server-related-issues"></a>Symptômes possibles des erreurs liées au pare-feu et au serveur proxy
Si vous rencontrez l’une des erreurs suivantes, cela signifie que vous avez probablement mal configuré le serveur proxy ou le pare-feu, et que la passerelle ne peut pas se connecter à Data Factory pour s’authentifier. Reportez-vous à la section précédente pour vous assurer que votre pare-feu et votre serveur proxy sont correctement configurés.

1. Quand vous tentez d’inscrire la passerelle, vous recevez le message d’erreur suivant : « Nous n’avons pas pu enregistrer la clé de passerelle. Avant de réessayer d’enregistrer la clé de passerelle, vérifiez que la passerelle de gestion des données est connectée et que le service d’hébergement de la passerelle de gestion des données est en cours d’exécution. »
2. Quand vous ouvrez le Gestionnaire de configuration, l’état indiqué est « Déconnecté » ou « En cours de connexion ». Quand vous affichez les journaux des événements Windows, sous « Observateur d’événements » > « Journaux des applications et services » > « Passerelle de gestion des données », des messages d’erreur tels que le suivant s’affichent : `Unable to connect to the remote server`
   `A component of Data Management Gateway has become unresponsive and restarts automatically. Component name: Gateway.`

### <a name="open-port-8050-for-credential-encryption"></a>Ouvrir le port 8050 pour le chiffrement des informations d’identification
Le port de trafic entrant **8050** est utilisé par l’application **Définition des informations d’identification** pour relayer les informations d’identification à la passerelle lorsque vous configurez un service lié local dans le portail Azure. Lors de l’installation de la passerelle, l’installation de la passerelle ouvre cette dernière par défaut sur l’ordinateur de passerelle.

Si vous utilisez un pare-feu tiers, vous pouvez ouvrir manuellement le port 8050. Si vous rencontrez des problèmes de pare-feu lors de l’installation de la passerelle, vous pouvez essayer d’utiliser la commande suivante pour installer la passerelle sans configurer le pare-feu.

```cmd
msiexec /q /i DataManagementGateway.msi NOFIREWALL=1
```

Si vous préférez ne pas ouvrir le port 8050 sur l’ordinateur passerelle, utilisez d’autres mécanismes que l’application **Définition des informations d’identification** pour configurer les informations d’identification de la banque de données. Vous pouvez par exemple utiliser la cmdlet PowerShell [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue). Consultez la section Configuration des informations d’identification et de la sécurité pour savoir comment configurer les informations d’identification de la banque de données.

## <a name="update"></a>Update
Par défaut, la passerelle de gestion des données est automatiquement mise à jour lorsqu’une version plus récente est disponible. La passerelle n’est pas mise à jour tant que toutes les tâches planifiées ne sont pas terminées. Aucune autre tâche n’est traitée par la passerelle avant la fin de l’opération de mise à jour. Si la mise à jour échoue, la passerelle est restaurée vers son ancienne version.

L’heure de mise à jour planifiée s’affiche aux emplacements suivants :

* Page Propriétés de la passerelle dans le portail Azure.
* Page d’accueil du gestionnaire de configuration de la passerelle de gestion des données
* Message de notification de la barre d’état du système.

L’onglet Accueil du Gestionnaire de configuration de passerelle de gestion des données affiche la planification de la mise à jour, ainsi que la date de la dernière mise à jour/installation de la passerelle.

![Planifier les mises à jour](media/data-factory-data-management-gateway/UpdateSection.png)

Vous pouvez installer la mise à jour immédiatement ou attendre que la passerelle soit mise à jour automatiquement à l’heure planifiée. Par exemple, l’image suivante montre le message de notification affiché dans le Gestionnaire de configuration de passerelle, ainsi que le bouton de mise à jour qui vous permet d’installer cette dernière immédiatement.

![Mise à jour dans DMG Configuration Manager](./media/data-factory-data-management-gateway/gateway-auto-update-config-manager.png)

Le message de notification dans la barre d’état système se présenterait comme l’image suivante :

![Message de barre d’état système](./media/data-factory-data-management-gateway/gateway-auto-update-tray-message.png)

Vous voyez s’afficher la progression de l’opération de mise à jour (manuelle ou automatique) dans la barre d’état système. À la prochaine ouverture du Gestionnaire de configuration de passerelle, un message s’affiche dans la barre de notification, vous indiquant que la passerelle a été mise à jour, et contenant un lien vers la rubrique relative aux [nouveautés](data-factory-gateway-release-notes.md).

### <a name="to-disableenable-auto-update-feature"></a>Pour activer/désactiver une fonctionnalité de mise à jour automatique
Vous pouvez désactiver/activer la fonctionnalité de mise à jour automatique comme suit :

[Pour une passerelle à nœud unique]
1. Lancez Windows PowerShell sur l’ordinateur de passerelle.
2. Accédez au dossier *C:\\\\Program Files\\Microsoft Integration Runtime\\3.0\\PowerShellScript\\* .
3. Exécutez la commande suivante pour désactiver la fonctionnalité de mise à jour automatique.

    ```powershell
    .\IntegrationRuntimeAutoUpdateToggle.ps1 -off
    ```
4. Pour la réactiver :

    ```powershell
    .\IntegrationRuntimeAutoUpdateToggle.ps1 -on
    ```
   [Pour une passerelle multinœud à haute disponibilité et scalable](data-factory-data-management-gateway-high-availability-scalability.md)
1. Lancez Windows PowerShell sur l’ordinateur de passerelle.
2. Accédez au dossier *C:\\\\Program Files\\Microsoft Integration Runtime\\3.0\\PowerShellScript\\* .
3. Exécutez la commande suivante pour désactiver la fonctionnalité de mise à jour automatique.

    Pour une passerelle avec une fonctionnalité de haute disponibilité, un paramètre AuthKey supplémentaire est nécessaire.
    ```powershell
    .\IntegrationRuntimeAutoUpdateToggle.ps1 -off -AuthKey <your auth key>
    ```
4. Pour la réactiver :

    ```powershell
    .\IntegrationRuntimeAutoUpdateToggle.ps1 -on -AuthKey <your auth key>
    ```

## <a name="configuration-manager"></a>Gestionnaire de configuration
Une fois la passerelle installée, vous pouvez lancer le Gestionnaire de configuration de passerelle de gestion des données de l’une des manières suivantes :

1. Dans la fenêtre **Rechercher**, saisissez **passerelle de gestion de données** pour accéder à cet utilitaire.
2. Exécutez le fichier exécutable *ConfigManager.exe* dans le dossier : *C:\\\\Program Files\\Microsoft Data Management Gateway\\2.0\\Shared*.

### <a name="home-page"></a>page d'accueil
La page d’accueil permet d’effectuer les actions suivantes :

* Afficher l’état de la passerelle (connectée au service cloud ou non, etc.).
* **Inscrire** à l’aide d’une clé à partir du portail.
* **Arrêter** et démarrer le **service hôte de passerelle de gestion des données** sur l’ordinateur de passerelle.
* **Planifier les mises à jour** à un moment spécifique de la journée.
* Afficher la date de la **dernière mise à jour** de la passerelle.

### <a name="settings-page"></a>Page Paramètres
La page Paramètres permet d’effectuer les actions suivantes :

* Afficher, modifier et exporter le **certificat** utilisé par la passerelle. Ce certificat est utilisé pour chiffrer les informations d’identification de la source de données.
* Modifier le **port HTTPS** du point de terminaison. La passerelle ouvre un port pour définir les informations d’identification de la source de données.
* **l’état** du point de terminaison.
* Afficher le **Certificat SSL** qui est utilisé pour la communication TLS/SSL entre le portail et la passerelle pour définir les informations d’identification pour les sources de données.

### <a name="remote-access-from-intranet"></a>Accès à distance à partir de l’intranet
Cette fonctionnalité sera activée plus tard. Dans les futures mises à jour (version 3.4 ou ultérieure), nous vous autoriserons à activer / désactiver une connexion à distance qui se fait via le port 8050 (voir la section ci-dessus) tout en utilisant l’application Gestionnaire des informations d’identification ou PowerShell pour chiffrer les informations d’identification.

### <a name="diagnostics-page"></a>Page Diagnostics
La page Diagnostics permet d’effectuer les actions suivantes :

* Activer la **journalisation** détaillée, afficher les journaux d’activité dans l’observateur d’événements et envoyer des journaux d’activité à Microsoft en cas de défaillance.
* **Tester la connexion** à une source de données.

### <a name="help-page"></a>Help page
Cette page d’aide affiche les informations suivantes :

* Brève description de la passerelle
* Numéro de version
* Liens vers l’aide en ligne, la déclaration de confidentialité et le contrat de licence.

## <a name="monitor-gateway-in-the-portal"></a>Passerelle de surveillance dans le portail
Dans le portail Azure, vous pouvez afficher un instantané en quasi temps réel de l’utilisation des ressources (processeur, mémoire, réseau (entrée/sortie), etc.) sur un ordinateur de passerelle.

1. Dans le portail Azure, accédez à la page d’accueil de votre fabrique de données, puis cliquez sur la vignette **Services liés**.

    ![Page d’accueil Data Factory](./media/data-factory-data-management-gateway/monitor-data-factory-home-page.png)
2. Sélectionnez la **passerelle** dans la page **Services liés**.

    ![Page Services liés](./media/data-factory-data-management-gateway/monitor-linked-services-blade.png)
3. Dans la page **Passerelle**, vous pouvez consulter l’utilisation de la mémoire et du processeur de la passerelle.

    ![Utilisation du processeur et de la mémoire de la passerelle](./media/data-factory-data-management-gateway/gateway-simple-monitoring.png)
4. Activez **Paramètres avancés** pour plus d’informations, notamment sur l’utilisation du réseau.
    
    ![Surveillance avancée de la passerelle](./media/data-factory-data-management-gateway/gateway-advanced-monitoring.png)

Le tableau suivant fournit les descriptions des colonnes utilisées dans la liste **Nœuds de passerelle** :

Propriété de surveillance | Description
:------------------ | :----------
Nom | Nom de la passerelle logique et nœuds associés à la passerelle. Le nœud est un ordinateur Windows local sur lequel la passerelle est installée. Pour plus d’informations sur la multitude de nœuds (jusqu’à quatre) dans une seule passerelle logique, consultez [Passerelle de gestion des données - Haute disponibilité et scalabilité](data-factory-data-management-gateway-high-availability-scalability.md).
Statut | État de la passerelle logique et des nœuds de passerelle. Exemple : En ligne/Hors connexion/Limité/etc. Pour plus d’informations sur ces états, consultez la section [État de la passerelle](#gateway-status).
Version | Indique la version de la passerelle logique et de chaque nœud de passerelle. La version de la passerelle logique est déterminée selon la version de la majorité des nœuds dans le groupe. S’il existe des nœuds de différentes versions dans l’installation de la passerelle logique, seuls les nœuds dont le numéro de version est identique à celui de la passerelle logique fonctionnent correctement. Les autres sont en mode limité et ont besoin d’une mise à jour manuelle (uniquement si la mise à jour automatique échoue).
Mémoire disponible | Mémoire disponible sur un nœud de passerelle. Cette valeur est un instantané en quasi temps réel.
Utilisation du processeur | Utilisation du processeur d’un nœud de passerelle. Cette valeur est un instantané en quasi temps réel.
Réseau (entrée/sortie) | Utilisation du réseau d’un nœud de passerelle. Cette valeur est un instantané en quasi temps réel.
Tâches simultanées (en cours d’exécution/limite) | Nombre de travaux ou tâches qui s’exécutent sur chaque nœud. Cette valeur est un instantané en quasi temps réel. La limite correspond au nombre maximal de travaux simultanés pour chaque nœud. Cette valeur est définie selon la taille de l’ordinateur. Vous pouvez augmenter la limite pour monter en puissance l’exécution de tâches simultanées dans les scénarios avancés, où le processeur/la mémoire /le réseau sont sous-utilisés, alors que les activités expirent. Cette fonctionnalité est également disponible avec une passerelle à nœud unique (même quand la fonctionnalité Haute disponibilité et scalabilité n’est pas activée).
Role | Il existe deux types de rôles dans une passerelle à plusieurs nœuds : répartiteur et rôle de travail. Tous les nœuds sont des rôles de travail, ce qui signifie qu’ils peuvent tous être utilisés pour exécuter des tâches. Il n’existe qu’un seul nœud répartiteur, utilisé pour extraire des tâches/travaux auprès de services cloud et les répartir entre les différents nœuds rôles de travail (y compris lui-même).

Dans cette page figurent des paramètres plus significatifs en présence de deux nœuds ou plus (scénario scale-out) dans la passerelle. Consultez [Passerelle de gestion des données - Haute disponibilité et scalabilité](data-factory-data-management-gateway-high-availability-scalability.md) pour plus d’informations sur la configuration d’une passerelle à plusieurs nœud.

### <a name="gateway-status"></a>État de la passerelle
Le tableau suivant indique les états possibles d’un **nœud de passerelle** :

Statut  | Commentaires/Scénarios
:------- | :------------------
En ligne | Nœud connecté au service Data Factory.
Hors connexion | Le nœud est hors connexion.
Mise à niveau | Le nœud est en cours de mise à jour automatique.
Limité | Dû à un problème de connectivité. Éventuellement dû à un problème de port HTTP 8050, à un problème de connectivité du bus de service ou à un problème de synchronisation des informations d’identification.
Inactif | La configuration du nœud est différente de celle de la majorité des autres nœuds.<br/><br/> Un nœud peut être inactif quand il ne parvient pas à se connecter à d’autres nœuds.

Le tableau suivant indique les états possibles d’une **passerelle logique**. L’état de la passerelle dépend des états des nœuds de passerelle.

Statut | Commentaires
:----- | :-------
Doit être inscrite | Aucun nœud n’est encore inscrit sur cette passerelle logique.
En ligne | Les nœuds de passerelle sont en ligne.
Hors connexion | Aucun nœud n’est en ligne.
Limité | Tous les nœuds inclus dans cette passerelle ne sont pas dans un état intègre. Cet état est un avertissement pouvant indiquer que certains nœuds sont en panne ! <br/><br/>Peut être dû à un problème de synchronisation des informations d’identification sur le nœud répartiteur/rôle de travail.

## <a name="scale-up-gateway"></a>Monter en puissance la passerelle
Vous pouvez configurer le nombre de **travaux de déplacement de données simultanés** qui peuvent s’exécuter sur un nœud pour augmenter la capacité de déplacement des données entre les magasins de données locaux et dans le cloud.

Quand la mémoire disponible et le processeur ne sont pas correctement utilisés, mais que la capacité inactive s’élève à 0, vous devez augmenter la taille des instances en augmentant le nombre de travaux simultanés pouvant s’exécuter sur un nœud. Vous pouvez également augmenter la taille des instances quand les activités expirent parce que la passerelle est surchargée. Dans les paramètres avancés d’un nœud de passerelle, vous pouvez augmenter la capacité maximale d’un nœud.

## <a name="troubleshooting-gateway-issues"></a>Résolution des problèmes de la passerelle
Consultez l’article [Résolution des problèmes de la passerelle](data-factory-troubleshoot-gateway-issues.md) pour plus d’informations/de conseils pour résoudre les problèmes liés à la passerelle de gestion des données.

## <a name="move-gateway-from-one-machine-to-another"></a>Déplacer la passerelle d’une machine vers une autre
Cette section décrit les opérations pour déplacer une passerelle client d’une machine vers une autre.

1. Dans le portail, accédez à la **page d’accueil de Data Factory**, puis cliquez sur la vignette **Services liés**.

    ![Lien de passerelles de données](./media/data-factory-data-management-gateway/DataGatewaysLink.png)
2. Sélectionnez votre passerelle dans la section **PASSERELLES DE DONNÉES** de la page **Services liés**.

    ![Page Services liés avec une passerelle sélectionnée](./media/data-factory-data-management-gateway/LinkedServiceBladeWithGateway.png)
3. Dans la page **Passerelle de données**, cliquez sur **Télécharger et installer la passerelle de données**.

    ![Télécharger le lien de passerelle](./media/data-factory-data-management-gateway/DownloadGatewayLink.png)
4. Dans la page **Configurer**, cliquez sur **Télécharger et installer la passerelle de données**, puis suivez les instructions pour installer la passerelle de données sur la machine.

    ![Page Configurer](./media/data-factory-data-management-gateway/ConfigureBlade.png)
5. Laissez le **Gestionnaire de configuration de la passerelle de gestion des données** ouvert.

    ![Gestionnaire de configuration](./media/data-factory-data-management-gateway/ConfigurationManager.png)
6. Dans la page **Configurer** du portail, cliquez sur **Recréer une clé** dans la barre de commandes, puis, dans le message d’avertissement, cliquez sur **Oui**. Cliquez sur le **bouton de copie** en regard du texte de la clé qui copie la clé dans le presse-papiers. La passerelle de l’ancienne machine cesse de fonctionner dès que vous recréez la clé.

    ![Recréer une clé 2](./media/data-factory-data-management-gateway/RecreateKey.png)
7. Collez la **clé** dans la zone de texte sur la page **Inscrire la passerelle** du **Gestionnaire de configuration de passerelle de gestion des données** sur votre machine. (Facultatif) Cochez la case **Afficher la clé de passerelle** pour afficher le texte de la clé.

    ![Copier la clé et s’inscrire](./media/data-factory-data-management-gateway/CopyKeyAndRegister.png)
8. Cliquez sur **S’inscrire** pour enregistrer la passerelle auprès du service cloud.
9. Dans l’onglet **Paramètres**, cliquez sur **Modifier** pour sélectionner le certificat utilisé avec l’ancienne passerelle, entrez le **mot de passe**, puis cliquez sur **Terminer**.

   ![Spécifier le certificat](./media/data-factory-data-management-gateway/SpecifyCertificate.png)

   Vous pouvez exporter un certificat à partir de l’ancienne passerelle en procédant comme suit : lancez le Gestionnaire de configuration de passerelle de gestion des données sur l’ancien panneau, basculez vers l’onglet **Certificat**, cliquez sur le bouton **Exporter** et suivez les instructions.
10. Une fois l’inscription de la passerelle terminée, vous devez voir **Inscription** définie sur **Inscrit** et **État** défini sur la valeur **Démarré** sur la page d’accueil du Gestionnaire de configuration de passerelle.

## <a name="encrypting-credentials"></a>Chiffrement des informations d’identification
Pour chiffrer les informations d’identification dans Data Factory Editor, procédez comme suit :

1. Lancez le navigateur web sur **l’ordinateur passerelle**, accédez au [portail Azure](https://portal.azure.com). Recherchez votre fabrique de données si nécessaire, ouvrez-la dans la page **DATA FACTORY**, puis cliquez sur **Créer et déployer** pour démarrer Data Factory Editor.
2. Cliquez sur un **service lié** existant dans l’arborescence pour afficher sa définition JSON, ou créez un autre service lié qui nécessite une passerelle de gestion des données (par exemple, SQL Server ou Oracle).
3. Dans l’éditeur JSON, entrez le nom de la passerelle pour la propriété **gatewayName** .
4. Entrez le nom du serveur pour la propriété **Data Source** dans **connectionString**.
5. Entrez le nom de la base de données pour la propriété **Initial Catalog** dans **connectionString**.
6. Cliquez sur le bouton **Chiffrer** dans la barre de commandes qui lance l’application ClickOnce **Gestionnaire d’informations d’identification**. La boîte de dialogue **Définition des informations d’identification** doit s’afficher.

    ![Boîte de dialogue de définition des informations d’identification](./media/data-factory-data-management-gateway/setting-credentials-dialog.png)
7. Dans la boîte de dialogue **Configuration des informations d’identification** , procédez comme suit :
   1. Sélectionnez l’ **authentification** que le service de Data Factory doit utiliser pour se connecter à la base de données.
   2. Entrez le nom de l’utilisateur ayant accès à la base de données dans le paramètre **USERNAME** .
   3. Entrez le mot de passe de l’utilisateur dans le paramètre **PASSWORD** .
   4. Cliquez sur **OK** pour chiffrer les informations d’identification et fermer la boîte de dialogue.
8. Vous devez maintenant voir une propriété **encryptedCredential** dans **connectionString**.

    ```JSON
    {
        "name": "SqlServerLinkedService",
        "properties": {
            "type": "OnPremisesSqlServer",
            "description": "",
            "typeProperties": {
                "connectionString": "data source=myserver;initial catalog=mydatabase;Integrated Security=False;EncryptedCredential=eyJDb25uZWN0aW9uU3R",
                "gatewayName": "adftutorialgateway"
            }
        }
    }
    ```
   Si vous accédez au portail à partir d’un ordinateur différent de l’ordinateur de passerelle, vous devrez peut-être vous assurer que l’application Gestionnaire d’informations d’identification peut se connecter à l’ordinateur de passerelle. Sinon, vous ne pourrez pas définir les informations d’identification de la source de données, ni tester la connexion à la source de données.

Quand vous utilisez l’application **Définition des informations d’identification**, le portail chiffre les informations d’identification avec le certificat que vous avez spécifié dans l’onglet **Certificat** du **Gestionnaire de configuration de passerelle** sur l’ordinateur de passerelle.

Si vous recherchez une approche basée sur une API pour chiffrer les informations d’identification, vous pouvez utiliser la cmdlet PowerShell [New-AzDataFactoryEncryptValue](/powershell/module/az.datafactory/new-azdatafactoryencryptvalue) pour chiffrer les informations d’identification. L'applet de commande utilise le certificat qui a servi à configurer la passerelle pour chiffrer les informations d'identification. Vous ajoutez des informations d’identification chiffrées pour l’élément **EncryptedCredential** de **connectionString** dans JSON. Vous utilisez JSON avec la cmdlet [New-AzDataFactoryLinkedService](/powershell/module/az.datafactory/new-azdatafactorylinkedservice) ou dans Data Factory Editor.

```JSON
"connectionString": "Data Source=<servername>;Initial Catalog=<databasename>;Integrated Security=True;EncryptedCredential=<encrypted credential>",
```

Il existe une autre approche pour définir les informations d’identification à l’aide de Data Factory Editor. Si vous créez un service SQL Server lié à l’aide de l’éditeur et entrez les informations d’identification en texte brut, ces informations d’identification sont chiffrées à l’aide d’un certificat appartenant au service Data Factory. Il n’utilise PAS le certificat que passerelle est configurée pour utiliser. Bien que cette approche puisse être un peu plus rapide dans certains cas, elle reste moins sécurisée. Par conséquent, nous vous recommandons de suivre cette approche uniquement à des fins de développement/test.

## <a name="powershell-cmdlets"></a>Applets de commande PowerShell
Cette section décrit comment créer et enregistrer une passerelle à l’aide des applets de commande Azure PowerShell.

1. Lancez **Azure PowerShell** en mode administrateur.
2. Connectez-vous à votre compte Azure en exécutant la commande suivante et en entrant vos informations d’identification Azure.

    ```powershell
    Connect-AzAccount
    ```
3. Utilisez la cmdlet **New-AzDataFactoryGateway** pour créer une passerelle logique, comme suit :

    ```powershell
    $MyDMG = New-AzDataFactoryGateway -Name <gatewayName> -DataFactoryName <dataFactoryName> -ResourceGroupName ADF –Description <desc>
    ```
    **Exemple de commande et de sortie**:

    ```
    PS C:\> $MyDMG = New-AzDataFactoryGateway -Name MyGateway -DataFactoryName $df -ResourceGroupName ADF –Description "gateway for walkthrough"

    Name              : MyGateway
    Description       : gateway for walkthrough
    Version           :
    Status            : NeedRegistration
    VersionStatus     : None
    CreateTime        : 9/28/2014 10:58:22
    RegisterTime      :
    LastConnectTime   :
    ExpiryTime        :
    ProvisioningState : Succeeded
    Key               : ADF#00000000-0000-4fb8-a867-947877aef6cb@fda06d87-f446-43b1-9485-78af26b8bab0@4707262b-dc25-4fe5-881c-c8a7c3c569fe@wu#nfU4aBlq/heRyYFZ2Xt/CD+7i73PEO521Sj2AFOCmiI
    ```

1. Dans Azure PowerShell, accédez au dossier : *C:\\\\Program Files\\Microsoft Integration Runtime\\3.0\\PowerShellScript\\* . Exécutez le script *RegisterGateway.ps1* associé à la variable locale **$Key**, comme indiqué dans la commande suivante. Ce script enregistre l’agent client installé sur votre ordinateur avec la passerelle logique que vous avez créée précédemment.

    ```powershell
    PS C:\> .\RegisterGateway.ps1 $MyDMG.Key
    ```
    ```
    Agent registration is successful!
    ```
    Vous pouvez inscrire la passerelle sur un ordinateur distant en utilisant le paramètre IsRegisterOnRemoteMachine. Exemple :

    ```powershell
    .\RegisterGateway.ps1 $MyDMG.Key -IsRegisterOnRemoteMachine true
    ```
2. Vous pouvez utiliser la cmdlet **Get-AzDataFactoryGateway** pour obtenir la liste des passerelles dans votre fabrique de données. Lorsque **l’état** est **online**, cela signifie que votre passerelle est prête.

    ```powershell        
    Get-AzDataFactoryGateway -DataFactoryName <dataFactoryName> -ResourceGroupName ADF
    ```
   Vous pouvez supprimer une passerelle à l’aide de la cmdlet **Remove-AzDataFactoryGateway** et mettre à jour la description de la passerelle en utilisant les cmdlets **Set-AzDataFactoryGateway**. Pour obtenir la syntaxe et d’autres détails sur ces applets de commande, consultez la rubrique Référence des applets de commande Azure Data Factory.  

### <a name="list-gateways-using-powershell"></a>Répertorier les passerelles à l’aide de PowerShell

```powershell
Get-AzDataFactoryGateway -DataFactoryName jasoncopyusingstoredprocedure -ResourceGroupName ADF_ResourceGroup
```

### <a name="remove-gateway-using-powershell"></a>Supprimer une passerelle à l’aide de PowerShell

```powershell
Remove-AzDataFactoryGateway -Name JasonHDMG_byPSRemote -ResourceGroupName ADF_ResourceGroup -DataFactoryName jasoncopyusingstoredprocedure -Force
```

## <a name="next-steps"></a>Étapes suivantes
* Consultez la page [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) . Dans cette procédure pas à pas, vous créez un pipeline qui utilise la passerelle qui déplace les données d’une base de données SQL Server vers un objet blob Azure.