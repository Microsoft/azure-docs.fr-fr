---
title: Présentation de Microsoft Azure Data Box Heavy | Microsoft Docs sur les données
description: Description d’Azure Data Box, une solution hybride qui vous permet de transférer des volumes importants de données dans Azure
services: databox
documentationcenter: NA
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: overview
ms.date: 08/28/2019
ms.author: alkohli
ms.openlocfilehash: 2f1f01a8cfa25c222848e7cc5c86dc3532eba348
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "80437777"
---
# <a name="what-is-azure-data-box-heavy"></a>Qu’est-ce qu’Azure Data Box Heavy ?

Azure Data Box Heavy vous permet d'envoyer des centaines de téraoctets de données vers Azure de manière rapide, économique et fiable. Pour vous permettre de transférer vos données vers Azure, vous recevez un appareil Data Box Heavy d'une capacité de stockage de 1 Po sur lequel vous copiez vos données avant de le renvoyer à Microsoft. L'appareil dispose d'un caisson robuste afin de protéger et de sécuriser vos données pendant le transport.

Une fois l'appareil réceptionné par votre centre de données, configurez-le à l'aide de l'interface utilisateur web locale. Copiez les données à partir de vos serveurs vers l’appareil et réexpédiez l’appareil à Azure. Dans le centre de données Azure, vos données sont chargées sur vos comptes de stockage Azure. Vous pouvez suivre l'ensemble du processus de bout en bout sur le portail Azure.


> [!IMPORTANT]
> - Pour demander un appareil, inscrivez-vous dans le [portail Azure](https://portal.azure.com).


## <a name="use-cases"></a>Cas d'utilisation

Data Box Heavy a été conçu pour les volumes de données de plusieurs centaines de téraoctets impossibles à charger vers Azure par le biais du réseau pour cause de connectivité insuffisante. Il peut s’agir d’un déplacement de données à usage unique, périodique, ou d’un transfert de données en bloc initial suivi de transferts périodiques. Voici les différents scénarios où Data Box Heavy peut être utilisé pour le transfert de données.

 - **Migration unique** - lorsque de grandes quantités de données locales sont transférées vers Azure.
     - Déplacez une bibliothèque multimédia à partir de bandes hors connexion dans Azure pour créer une bibliothèque multimédia en ligne.
     - Migrez votre batterie de machines virtuelles, d'instances de SQL Server et d'applications vers Azure.
     - Déplacez des données historiques vers Azure pour une analyse et des rapports approfondis à l'aide de HDInsight.

 - **Transfert en bloc initial** : lorsqu’un transfert en bloc initial est effectué à l’aide de Data Box Heavy (seed) suivi de transferts incrémentiels sur le réseau.
     - Par exemple, Data Box Heavy et un partenaire de solutions de sauvegarde sont utilisés pour déplacer d'importantes sauvegardes initiales de données historiques vers Azure. Une fois l’opération terminée, les données incrémentielles sont transférées via le réseau vers le stockage Azure.

 - **Chargements périodiques** - lorsqu’une grande quantité de données est générée régulièrement et doit être déplacée vers Azure. Par exemple, dans le secteur de la production d’énergie, où du contenu vidéo est généré sur des plateformes pétrolières et dans des parcs éoliens.

## <a name="benefits"></a>Avantages

Data Box Heavy est conçu pour déplacer d'importantes quantités de données vers Azure avec peu ou pas d'impact sur votre réseau. La solution offre les avantages suivants :

- **Vitesse** : Data Box Heavy utilise des interfaces réseau hautes performances de 40 Gbits/s.

- **Sécurité** : Data Box Heavy intègre des protections de sécurité pour l'appareil, les données et le service.
    - L’appareil dispose d’un caisson robuste sécurisé par des vis et des autocollants inviolables.
    - Les données sur l’appareil sont sécurisées en permanence à l’aide d’un chiffrement AES 256 bits.
    - L’appareil peut uniquement être déverrouillé à l’aide d’un mot de passe fourni dans le portail Azure.
    - Le service est protégé par les fonctionnalités de sécurité Azure.
    - Une fois vos données chargées dans Azure, les disques de l'appareil sont nettoyés, conformément aux normes 800-88r1 du NIST (National Institute of Standards and Technology).


## <a name="features-and-specifications"></a>Fonctionnalités et spécifications

Dans cette version, l'appareil Data Box Heavy présente les caractéristiques suivantes.

| Spécifications                                          | Description              |
|---------------------------------------------------------|--------------------------|
| Poids                                                  | ~ 500 lbs. <br>Appareil sur roues verrouillées pour le transport|
| Dimensions                                              | Largeur : 26 pouces Hauteur : 28 pouces Longueur : 48 pouces |
| Espace en rack                                              | Ne peut pas être monté en rack|
| Câbles requis                                         | 4 cordons d'alimentation 120 V/10 A mis à la terre (NEMA 5-15) inclus <br> L'appareil prend en charge une alimentation max. de 240 V et est doté de connecteurs C-13 <br> Utilisez des câbles réseau compatibles avec [Mellanox MCX314A-BCCT](https://store.mellanox.com/products/mellanox-mcx314a-bcct-connectx-3-pro-en-network-interface-card-40-56gbe-dual-port-qsfp-pcie3-0-x8-8gt-s-rohs-r6.html)  |
| Power                                                    | 4 blocs d'alimentation intégrés partagés entre les deux nœuds de l'appareil <br> Consommation énergétique standard de 1 200 watts|
| Capacité de stockage                                        | ~ 1 Po brut, 70 disques de 14 To chacun <br> 770 To de capacité utile|
| Nombre de nœuds                                          | 2 nœuds indépendants par appareil (500 To chacun) |
| Interfaces réseau par nœud                             | 4 interfaces réseau par nœud <br><br> MGMT, DATA3 <ul><li> 2 interfaces de 1 GbE </li><li> MGMT est dédiée à la gestion et l'installation initiale, et n’est pas configurable par l'utilisateur </li><li> DATA3 est configurable par l'utilisateur et l’interface DHCP (Dynamic Host Configuration Protocol) par défaut</li></ul>Interfaces de données DATA1, DATA2 <ul><li>2 interfaces de 40 GbE </li><li> Configurable par l’utilisateur pour statique ou DHCP (par défaut)</li></ul>|


## <a name="components"></a>Components

Data Box Heavy inclut les composants suivants :

* **Appareil Data Box Heavy** : appareil physique doté d’une enveloppe extérieure robuste qui stocke les données en toute sécurité. Cet appareil possède une capacité de stockage utilisable de 770 To.
    
* **Service Data Box** : extension du portail Azure qui vous permet de gérer un appareil Data Box Heavy à partir d’une interface web accessible depuis divers emplacements géographiques. Utilisez le service Data Box pour gérer votre appareil Data Box Heavy. Les tâches de service incluent la création et la gestion de commandes, l’affichage et la gestion des alertes ainsi que la gestion des partages.  

* **Interface utilisateur web locale** : interface utilisateur web qui est utilisée pour configurer l’appareil afin qu’il puisse se connecter au réseau local, puis pour inscrire l’appareil auprès du service Data Box. Utilisez l’interface utilisateur web locale pour arrêter et redémarrer l’appareil, afficher les journaux d’activité de copie et contacter le Support Microsoft pour créer une demande de service.


## <a name="the-workflow"></a>Workflow

Un flux type inclut les étapes suivantes :

1. **Commande** - Créez une commande dans le portail Azure et indiquez les informations d’expédition et le compte de stockage de destination Azure de vos données. Si l’appareil est disponible, Azure le prépare et l’expédie avec un ID de suivi d’expédition.

2. **Réception** : une fois l’appareil livré, connectez-le au réseau et à l’alimentation à l’aide des câbles spécifiés. Mettez l’appareil sous tension et connectez-vous à l’appareil. Configurez le réseau de l’appareil et montez les partages sur l’ordinateur hôte à l’emplacement à partir duquel vous souhaitez copier les données.

3. **Copie des données** : copiez les données vers les partages Data Box Heavy.

4. **Retour** : préparez l’appareil, mettez-le hors tension et retournez l’appareil au centre de données Azure.

5. **Chargement** : les données sont automatiquement copiées de l’appareil vers Azure. Les disques de l’appareil sont effacés en toute sécurité conformément aux instructions du National Institute of Standards and Technology (NIST).

Tout au long de ce processus, vous êtes averti par e-mail de toutes les modifications d'état.

## <a name="region-availability"></a>Disponibilité des régions

Data Box Heavy peut transférer des données en fonction de la région dans laquelle le service est déployé, du pays/de la région dans lequel l'appareil est livré et du compte de stockage Azure cible vers lequel vous transférez les données.

- **Disponibilité du service** : pour cette version, Data Box Heavy est disponible dans les régions suivantes :
    - Toutes les régions de cloud public aux États-Unis : USA Centre-Ouest, USA Ouest 2, USA Ouest, USA Centre Sud, USA Centre, USA Centre Nord, USA Est et USA Est 2.
    - Union européenne : Europe Ouest et Europe Nord.
    - Royaume-Uni : Royaume-Uni Sud et Royaume-Uni Ouest.
    - France : France Centre et France Sud.

- **Comptes de stockage de destination** : les comptes de stockage qui stockent les données sont disponibles dans toutes les régions Azure où le service est disponible.

Pour obtenir les informations les plus récentes sur la disponibilité régionale de Data Box Heavy, accédez à [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all).

## <a name="sign-up"></a>Inscription

Procédez comme suit pour vous inscrire à Data Box Heavy :

1. [Connectez-vous au portail Azure](https://portal.azure.com).
2. Cliquez sur **+ Créer une ressource** pour créer une ressource. Recherchez **Azure Data Box**. Sélectionnez le service **Azure Data Box**.
3. Cliquez sur **Créer**.
4. Sélectionnez l’abonnement que vous voulez utiliser pour Data Box Heavy. Sélectionnez la région dans laquelle vous souhaitez déployer la ressource Data Box Heavy. Dans l’option **Data Box Heavy**, cliquez sur **S’inscrire**.
5. Répondez aux questions concernant le pays/la région de résidence des données, le laps de temps, le service Azure cible pour le transfert des données, la bande passante réseau et la fréquence de transfert des données. Consultez le texte Confidentialité et conditions d'utilisation et sélectionnez la case à cocher autorisant Microsoft à utiliser votre adresse de messagerie pour vous contacter.

Une fois inscrit, vous pouvez commander un appareil Data Box Heavy.

    
