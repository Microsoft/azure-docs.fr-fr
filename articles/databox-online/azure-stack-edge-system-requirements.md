---
title: Configuration système requise pour Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Découvrez la configuration logicielle et réseau requise pour Azure Stack Edge Pro
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 07/15/2020
ms.author: alkohli
ms.openlocfilehash: 4deeec1b6c9db80c15b001efb7b40bdb4bb9d515
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452651"
---
# <a name="azure-stack-edge-pro-system-requirements"></a>Configuration système requise pour Azure Stack Edge Pro

Cet article décrit la configuration système requise pour votre solution Microsoft Azure Stack Edge Pro et pour les clients se connectant à Azure Stack Edge Pro. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre solution Azure Stack Edge Pro. Reportez-vous aussi souvent que nécessaire à ces informations pendant le déploiement, et après, pour son fonctionnement.

La configuration système requise pour Azure Stack Edge Pro inclut ce qui suit :

- **Configuration logicielle pour les hôtes** : décrit les plateformes prises en charge, les navigateurs pour l’interface utilisateur de configuration locale, les clients SMB et les exigences supplémentaires pour les clients qui accèdent à l’appareil.
- **Configuration réseau pour l’appareil** : fournit des informations sur la configuration réseau nécessaire au fonctionnement de l’appareil physique.

## <a name="supported-os-for-clients-connected-to-device"></a>Systèmes d’exploitation pris en charge pour les clients connectés à l’appareil

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>Protocoles pris en charge pour l’accès des clients à l’appareil

[!INCLUDE [Supported protocols for clients accessing device](../../includes/data-box-edge-gateway-supported-client-protocols.md)]

## <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

[!INCLUDE [Supported storage accounts](../../includes/data-box-edge-gateway-supported-storage-accounts.md)]

## <a name="supported-storage-types"></a>Types de stockage pris en charge

[!INCLUDE [Supported storage types](../../includes/data-box-edge-gateway-supported-storage-types.md)]

## <a name="supported-browsers-for-local-web-ui"></a>Navigateurs pris en charge pour l’interface utilisateur web locale

[!INCLUDE [Supported browsers for local web UI](../../includes/data-box-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>Configuration requise du port réseau

### <a name="port-requirements-for-azure-stack-edge-pro"></a>Configuration des ports requise pour Azure Stack Edge Pro

Le tableau ci-dessous répertorie les ports qui doivent être ouverts dans votre pare-feu pour autoriser le trafic SMB, cloud ou de gestion. Dans ce tableau, *entrée* ou *entrant* représente la direction à partir de laquelle les requêtes clientes entrantes accèdent à votre appareil. *Sortie* ou *sortant* représente la direction vers laquelle votre appareil Azure Stack Edge Pro envoie des données de façon externe, au-delà du déploiement : par exemple, sortant vers Internet.

[!INCLUDE [Port configuration for device](../../includes/data-box-edge-gateway-port-config.md)]

### <a name="port-requirements-for-iot-edge"></a>Configuration requise du port pour IoT Edge

Azure IoT Edge permet les communications sortantes entre un appareil Edge local et le cloud Azure au moyen des protocoles IoT Hub pris en charge. Les communications entrantes sont uniquement requises pour des scénarios spécifiques où Azure IoT Hub a besoin d’envoyer (push) des messages à l’appareil Azure IoT Edge (par exemple, messagerie cloud vers appareil).

Utilisez le tableau suivant pour configurer les ports des serveurs hébergeant le runtime Azure IoT Edge :

| N° de port | Entrant ou sortant | Étendue de ports | Obligatoire | Assistance |
|----------|-----------|------------|----------|----------|
| TCP 443 (HTTPS)| Sortie       | WAN        | Oui      | Sortie ouverte pour le déploiement de IoT Edge. Cette configuration est requise en cas d’utilisation de scripts manuels ou du service Azure IoT Device Provisioning.|

Pour plus d'informations, consultez [Règles de configuration du pare-feu et des ports pour le déploiement d’IoT Edge](../iot-edge/troubleshoot.md).

## <a name="url-patterns-for-firewall-rules"></a>Modèles d’URL pour règles de pare-feu

Les administrateurs réseau peuvent souvent configurer des règles de pare-feu avancées basées sur des modèles d’URL afin de filtrer le trafic entrant et sortant. Votre appareil Azure Stack Edge Pro et le service dépendent d’autres applications Microsoft comme Azure Service Bus, Azure Active Directory Access Control, des comptes de stockage et des serveurs Microsoft Update. Les modèles d’URL associés à ces applications peuvent être utilisés pour configurer des règles de pare-feu. Il est important de comprendre que les modèles d’URL associés à ces applications peuvent changer. Ces changements demandent que l’administrateur réseau contrôle et mette à jour les règles de pare-feu pour votre appareil Azure Stack Edge Pro, si nécessaire.

Dans la plupart des cas, nous vous recommandons de définir librement les règles de pare-feu pour le trafic sortant en fonction des adresses IP fixes Azure Stack Edge Pro. Toutefois, vous pouvez utiliser les informations ci-dessous pour définir les règles de pare-feu avancées qui sont nécessaires à la création d’environnements sécurisés.

> [!NOTE]
> - Les adresses IP d’appareil (sources) doivent toujours être définies sur l’ensemble des interfaces réseau activées pour le cloud.
> - Les adresses IP de destination doivent être définies sur les [plages d’adresses IP Azure Datacenter](https://www.microsoft.com/download/confirmation.aspx?id=41653).

### <a name="url-patterns-for-gateway-feature"></a>Modèles d’URL pour la fonctionnalité de passerelle

[!INCLUDE [URL patterns for firewall](../../includes/data-box-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-compute-feature"></a>Modèles d’URL pour la fonctionnalité de calcul

| Modèle d’URL                      | Composant ou fonctionnalité                     |   
|----------------------------------|---------------------------------------------|
| https:\//mcr.microsoft.com<br></br>https://\*.cdn.mscr.io | Registre de conteneurs Microsoft (obligatoire)               |
| https://\*.azurecr.io                     | Registres de conteneurs personnels et tiers (facultatif) | 
| https://\*.azure-devices.net              | Accès IoT Hub (obligatoire)                             | 

### <a name="url-patterns-for-gateway-for-azure-government"></a>Modèles d’URL pour la passerelle pour Azure Government

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/data-box-edge-gateway-gov-url-patterns-firewall.md)]

### <a name="url-patterns-for-compute-for-azure-government"></a>Modèles d’URL pour le calcul pour Azure Government

| Modèle d’URL                      | Composant ou fonctionnalité                     |  
|----------------------------------|---------------------------------------------|
| https:\//mcr.microsoft.com<br></br>https://\*.cdn.mscr.com | Registre de conteneurs Microsoft (obligatoire)               |
| https://\*.azure-devices.us              | Accès IoT Hub (obligatoire)           |
| https://\*.azurecr.us                    | Registres de conteneurs personnels et tiers (facultatif) | 

## <a name="internet-bandwidth"></a>Bande passante Internet

[!INCLUDE [Internet bandwidth](../../includes/data-box-edge-gateway-internet-bandwidth.md)]

## <a name="compute-sizing-considerations"></a>Considérations relatives au dimensionnement de la capacité de calcul

Utilisez votre expérience lors des phases de développement et de test de votre solution afin de vous assurer que votre appareil Azure Stack Edge Pro dispose d’une capacité suffisante et que vous en tirez des performances optimales.

Tenez compte des facteurs suivants :

- **Caractéristiques de l’objet conteneur**. Posez-vous les questions suivantes :

    - Combien de conteneurs votre charge de travail compte-t-elle ? Vous pourriez avoir un grand nombre de conteneurs légers ou seulement quelques-uns gourmands en ressources.
    - Quelles sont les ressources allouées à ces conteneurs, et quelles sont les ressources qu’ils consomment ?
    - Combien de couches vos conteneurs partagent-ils ?
    - Existe-t-il des conteneurs inutilisés ? Un conteneur arrêté occupe toujours de l’espace disque.
    - En quels langage vos conteneurs sont-ils écrits ?
- **Taille des données traitées**. Quelle quantité de données vos conteneurs traiteront-ils ? Ces données consommeront-elles de l’espace disque, ou seront-elles traitées en mémoire ?
- **Performances attendues**. Quelles sont les caractéristiques de performances souhaitées de votre solution ? 

Pour comprendre et affiner les performances de votre solution, vous pouvez utiliser :

- Les métriques de calcul disponibles dans le portail Azure. Accédez à votre ressource Azure Stack Edge, puis à **Supervision > Métriques**. Examinez les valeurs **Computing en périphérie - Utilisation de la mémoire** et **Computing en périphérie - Pourcentage du processeur** pour comprendre les ressources disponibles et comment elles sont utilisées.
- Les commandes de supervision disponibles par le biais de l’interface PowerShell de l’appareil comme :

    - Les statistiques `dkr` pour obtenir un flux en direct des statistiques d’utilisation des ressources de conteneurs. La commande prend en charge le processeur, l’utilisation de la mémoire, la limite de mémoire et les métriques d’E/S réseau.
    - `dkr system df` pour obtenir des informations concernant la quantité d’espace disque utilisée. 
    - `dkr image [prune]` pour nettoyer les images inutilisées et libérer de l’espace.
    - `dkr ps --size` pour afficher la taille approximative d’un conteneur en cours d’exécution. 

    Pour plus d’informations sur les commandes disponibles, accédez à [Superviser et dépanner des modules de computing en périphérie](azure-stack-edge-connect-powershell-interface.md#monitor-and-troubleshoot-compute-modules).

Pour finir, veillez à tester votre solution sur votre jeu de données et à quantifier les performances sur Azure Stack Edge Pro avant le déploiement en production.


## <a name="next-step"></a>Étape suivante

- [Déployer votre Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)