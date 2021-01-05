---
title: FAQ Microsoft Azure Data Box | Microsoft Docs sur les données
description: Contient les questions fréquentes et leurs réponses au sujet d’Azure Data Box, une solution cloud qui vous permet de transférer de grandes quantités de données dans Azure.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 12/14/2020
ms.author: alkohli
ms.openlocfilehash: 18a1fc7f42884dcc194256e1f51ddecb62d2708c
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516394"
---
# <a name="azure-data-box-frequently-asked-questions"></a>Azure Data Box : Forum Aux Questions (FAQ)

La solution hybride Microsoft Azure Data Box Disk vous permet d’envoyer des téraoctets de données vers Azure de façon rapide, économique et fiable, via un appareil de transfert. Cette FAQ contient des réponses aux questions que vous pouvez vous poser lorsque vous utilisez Data Box sur le portail Azure.

Les questions/réponses sont classées dans les catégories suivantes :

- À propos du service
- Commander un appareil
- Configuration et connexion 
- Suivi de l'état
- Copier des données 
- Expédier un appareil
- Vérification et chargement de données 
- Prise en charge de la chaîne de responsabilité

## <a name="about-the-service"></a>À propos du service

### <a name="q-what-is-azure-data-box-service"></a>Q. Qu’est-ce que le service Azure Data Box ? 
R.  Le service Azure Data Box a été conçu pour l’ingestion des données hors connexion. Ce service gère une gamme de produits offrant différentes capacités de stockage, tous conçus pour le transport des données. 

### <a name="q-what-is-azure-data-box"></a>Q. Qu’est-ce qu’Azure Data Box ?
R. Azure Data Box permet un transfert rapide, économique et sécurisé de plusieurs téraoctets de données dans Azure. Vous commandez l’appareil Data Box via le portail Azure. Microsoft fait appel à un transporteur régional pour vous faire parvenir un appareil de stockage d’une capacité utile de 80 To. 

Une fois l’appareil reçu, vous le configurez rapidement à l’aide de l’interface utilisateur web locale. Copiez les données à partir de vos serveurs vers l’appareil ou vice versa et réexpédiez l’appareil à Azure. Pour une commande d’importation, dans le centre de données Azure, vos données sont automatiquement envoyées de l’appareil vers Azure. L’ensemble du processus est suivi de bout en bout par le service Data Box dans le portail Azure.

### <a name="q-when-should-i-use-data-box"></a>Q. Quand dois-je utiliser Data Box ?
R. Si vous avez 40 à 500 To de données que vous souhaitez transférer vers ou depuis Azure, il serait préférable d’utiliser Data Box. Pour les tailles de données < 40 To, utilisez Data Box Disk, et pour les tailles de données > 500 To, optez pour [Data Box Heavy](data-box-heavy-overview.md).

### <a name="q-what-is-the-price-of-data-box"></a>Q. Combien coûte Data Box ?
R. Data Box est disponible à un tarif réduit pendant 10 jours. Lorsque vous sélectionnez le modèle de produit lors de la création d’une commande dans le portail Azure, les frais pour l’appareil sont affichés. Les frais d’expédition standard et les frais liés au stockage Azure s’appliquent également. Les commandes d’exportation suivent un modèle de tarification similaire à celui des commandes d’importation, même si des frais de sortie supplémentaires peuvent s’appliquer. 

Pour plus d’informations, consultez [Tarification Azure Data Box](https://azure.microsoft.com/pricing/details/storage/databox/) et [Frais de sortie](https://azure.microsoft.com/pricing/details/bandwidth/). 

### <a name="q-what-is-the-maximum-amount-of-data-i-can-transfer-with-data-box-in-one-instance"></a>Q. Quelle est la quantité maximale de données que je peux transférer avec Data Box dans une seule instance ?
R. Data Box offre une capacité brute de 100 To et une capacité utile de 80 To. Vous pouvez transférer jusqu’à 80 To de données avec Data Box. Pour transférer davantage de données, vous devez commander davantage d’appareils.

### <a name="q-how-can-i-check-if-data-box-is-available-in-my-region"></a>Q. Comment puis-je vérifier si Data Box est disponible dans ma région ? 
R.  Pour plus d’informations sur les pays/régions dans lesquels Data Box est disponible, accédez à la [disponibilité par région](data-box-overview.md#region-availability).  

### <a name="q-which-regions-can-i-store-data-in-with-data-box"></a>Q. Dans quelles régions puis-je stocker des données avec Data Box ?
R. La solution Data Box est prise en charge dans toutes les régions des États-Unis, en Europe Ouest, en Europe Nord, en France, au Royaume-Uni, au Japon, en Australie et au Canada. Pour plus d’informations, consultez [Disponibilité des régions](data-box-overview.md#region-availability).

### <a name="q-how-can-i-import-source-data-at-my-location-in-a-particular-country-to-an-azure-region-in-a-different-countryregion-or-export-data-from-an-azure-region-in-one-country-to-a-different-countryregion"></a>Q. Comment puis-je importer des données sources à mon emplacement dans un pays vers une région Azure située dans un autre pays/une autre région ou exporter des données à partir d’une région Azure d’un pays vers un autre pays/une autre région ?

Data Box prend en charge l’ingestion ou la sortie de données, mais uniquement dans le même pays/la même région que leur destination. Aucune frontière internationale ne peut être franchie. La seule exception concerne les commandes au sein de l’Union européenne (UE), où les Data Box peuvent être expédiés depuis et vers n’importe quel pays ou n’importe quelle région de l’UE.

Par exemple, dans le scénario d’importation, si vous souhaitez transférer des données sources se trouvant au Canada vers un compte de stockage Azure USA Ouest, vous pouvez procéder comme suit :

1. Commandez un appareil Data Box au Canada en choisissant un compte de stockage au Canada. L’appareil est expédié d’un centre de données Azure situé au Canada à l'adresse de livraison (au Canada) fournie lors de la création de la commande.

2. Une fois la copie de données locales vers le Data Box terminée, retournez l’appareil au centre de données Azure au Canada. Les données présentes sur le Data Box sont ensuite chargées sur le compte de stockage de destination dans la région canadienne Azure choisie lors de la création de la commande.

3. Vous pouvez ensuite utiliser un outil comme AzCopy pour copier les données dans un compte de stockage situé dans la région USA Ouest. Cette étape entraîne des frais de [stockage standard](https://azure.microsoft.com/pricing/details/storage/) ainsi que des frais de [bande passante](https://azure.microsoft.com/pricing/details/bandwidth/) qui ne sont pas inclus dans la facturation de Data Box.

### <a name="q-who-should-i-contact-if-i-come-across-any-issues-with-data-box"></a>Q. Qui dois-je contacter si je rencontre des problèmes avec Data Box ?
R. Si vous rencontrez des problèmes avec Data Box, contactez [Support Microsoft](data-box-disk-contact-microsoft-support.md).

### <a name="q-i-lost-my-data-box-is-there-a-lost-device-charge"></a>Q. J’ai perdu mon appareil Data Box. La perte d’un appareil occasionne-t-elle des frais supplémentaires ?
R. Oui. La perte ou l’endommagement d’un appareil est facturé. Ces frais sont décrits sur la [page Tarification](https://azure.microsoft.com/pricing/details/storage/databox/) ainsi que dans les [conditions d’utilisation du produit](https://www.microsoft.com/licensing/product-licensing/products).


## <a name="order-device"></a>Commander un appareil

### <a name="q-how-do-i-get-data-box"></a>Q. Comment faire pour obtenir Data Box ? 
R.  Pour obtenir Azure Data Box, connectez-vous au portail Azure et créez une commande Data Box. Indiquez vos coordonnées et vos paramètres de notification. Une fois que vous avez passé une commande, un appareil Data Box vous sera expédié sous 10 jours, en fonction de la disponibilité. Pour plus d’informations, accédez à [Commander une Data Box](data-box-deploy-ordered.md).

### <a name="q-i-couldnt-create-a-data-box-order-in-the-azure-portal-why"></a>Q. Je n’ai pas pu créer de commande Data Box dans le portail Azure. Pourquoi ?
R. Si vous ne pouvez pas créer de commande Data Box, le problème vient de votre type d’abonnement ou de votre accès.

Vérifiez votre abonnement. Data Box est disponible uniquement pour les offres d’abonnement Contrat Entreprise et Fournisseur de solutions Cloud. Si vous ne disposez pas de l’un de ces types d’abonnement, contactez Support Microsoft pour mettre à niveau votre abonnement.

Si vous disposez d’un type d’offre pris en charge pour l’abonnement, vérifiez le niveau d’accès de votre abonnement. Vous devez être contributeur ou propriétaire de votre abonnement pour créer une commande.

### <a name="q-how-long-will-my-order-take-from-order-creation-to-data-uploaded-to-azure"></a>Q. Combien de temps s’écoulera entre la création de ma commande et le chargement des données vers Azure ?

R. Les délais estimés suivants pour chaque phase du traitement des commandes vous donneront une bonne idée de ce à quoi vous pouvez vous attendre.  

Ces délais sont des *estimations*. Le temps nécessaire à chaque étape du traitement des commandes est déterminé par la charge du centre de données, les commandes simultanées et d’autres conditions environnementales.

**Délais estimés pour une commande Data Box :**

1. Commande d’un appareil Data Box : quelques minutes, à partir du portail
2. Allocation et préparation des appareils : 1 à 2 jours ouvrables
3. Expédition : 2 à 3 jours ouvrables
4. Copie des données sur le site du client : dépend de la nature des données, de la taille et du nombre de fichiers
5. Réexpédition : 2 à 3 jours ouvrables
6. Traitement de l’appareil au centre de données : 1 à 2 jours ouvrables
7. Charger les données sur Azure : commence dès que le traitement est terminé et que l’appareil est connecté. Le temps de chargement dépend de la nature des données, de la taille et du nombre de fichiers.

### <a name="q-i-ordered-a-couple-of-data-box-devices-i-cant-create-any-additional-orders-why"></a>Q. J’ai commandé deux appareils Data Box. Mais je ne peux pas créer d’autres commandes. Pourquoi ?
R. Nous autorisons un maximum de cinq commandes actives par abonnement et par zone commerciale (paire pays/région sélectionnée). Si vous avez besoin de commander un appareil supplémentaire, contactez le support Microsoft pour augmenter la limite de votre abonnement.

### <a name="q-when-i-try-to-create-an-order-i-receive-a-notification-that-the-data-box-service-is-not-available-what-does-this-mean"></a>Q. Lorsque j’essaie de créer une commande, je reçois une notification indiquant que le service Data Box n’est pas disponible. Qu’est-ce que cela signifie ?
R. Le service Data Box n’est pas disponible pour la paire pays/région que vous avez sélectionnée. Modifier cette paire vous permettra sans doute de bénéficier du service Data Box. Pour obtenir la liste des régions où le service est disponible, accédez à [Disponibilité des régions pour Data Box](data-box-overview.md#region-availability).

### <a name="q-i-placed-my-data-box-order-few-days-back-when-will-i-receive-my-data-box"></a>Q. J’ai passé ma commande Data Box il y a quelques jours. Quand recevrai-je ma Data Box ?
R. Lorsque vous passez commande, nous vérifions si l’appareil commandé est disponible. Si l’appareil est disponible, nous vous l’expédierons sous 10 jours. Mais nous sommes parfois confrontés à des périodes de forte demande. Dans ce cas, votre commande est mise en attente et vous pouvez suivre son statut dans le portail Azure. La commande est automatiquement annulée si elle n’est pas finalisée sous 90 jours.

### <a name="q-i-have-filled-up-my-data-box-with-data-and-need-to-order-another-one-is-there-a-way-to-quickly-place-the-order"></a>Q. J’ai rempli mon appareil Data Box de données et je dois en commander un autre. Existe-t-il un moyen de passer rapidement une commande ?
R. Vous pouvez cloner votre commande précédente. Ainsi, vous créez une commande identique à la précédente et vous pouvez en modifier les détails sans avoir à entrer votre adresse, vos informations de contact et vos préférences de notification. Le clonage n’est autorisé que pour les commandes d’importation.

## <a name="configure-and-connect"></a>Configuration et connexion

### <a name="q-how-do-i-unlock-the-data-box"></a>Q. Comment faire pour déverrouiller la Data Box ? 
R.  Dans le portail Azure, affichez votre commande Data Box, puis accédez à **Informations sur l’appareil**. Copiez le mot de passe de déverrouillage. Utilisez ce mot de passe pour vous connecter à l’interface utilisateur web locale sur votre Data Box. Pour plus d'informations, consultez [Tutoriel : Déballer, brancher et connecter votre Azure Data Box](data-box-deploy-set-up.md).

### <a name="q-can-i-use-a-linux-host-computer-to-connect-and-copy-the-data-on-to-the-data-box"></a>Q. Puis-je utiliser un ordinateur hôte Linux pour me connecter et copier les données sur Data Box ?
R.  Oui. Vous pouvez utiliser Data Box pour vous connecter à des clients SMB et NFS. Pour plus d’informations, accédez à la liste des [systèmes d’exploitation pris en charge](data-box-system-requirements.md) pour votre ordinateur hôte.

### <a name="q-my-data-box-is-dispatched-but-now-i-want-to-cancel-this-order-why-is-the-cancel-button-not-available"></a>Q. Ma Data Box a été expédié, mais je souhaite maintenant annuler ma commande. Pourquoi le bouton Annuler n’est-il pas disponible ?
R.  Vous pouvez annuler votre commande Data Box uniquement si celle-ci n’a pas encore été traitée. Une fois la commande Data Box traitée, vous ne pouvez plus l’annuler. 

### <a name="q-can-i-connect-a-data-box-at-the-same-to-multiple-host-computers-to-transfer-data"></a>Q. Puis-je connecter une Data Box à plusieurs ordinateurs hôtes simultanément pour transférer des données ?
R. Oui. Plusieurs ordinateurs hôtes peuvent se connecter à Data Box pour transférer des données, et plusieurs travaux de copie peuvent être exécutés en parallèle. Pour plus d'informations, consultez [Tutoriel : Copier des données sur Azure Data Box](data-box-deploy-copy-data.md).

### <a name="q-can-i-connect-to-both-the-10-gbe-interfaces-on-the-data-box-to-transfer-data"></a>Q. Puis-je me connecter aux deux interfaces 10 GbE sur la Data Box pour transférer des données ?
R. Oui. Il est possible de connecter les deux interfaces 10 GbE sur la Data Box pour copier des données en même temps. Pour plus d’informations sur la copie de données, accédez au [Tutoriel : Copier des données sur Azure Data Box](data-box-deploy-copy-data.md).

<!--### Q. The network interface on my Data Box is not working. What should I do? 
A. 

### Q. I could not set up Data Box using a Dynamic (DHCP) IP address. Why?
A.

### Q. I could not set up Data Box using a Static IP address. Why?
A.

### Q. I could not set up Data Box on a private network. Why?
A.-->

### <a name="q-the-system-fault-indicator-led-on-the-front-operating-panel-is-on-what-should-i-do"></a>Q. Le voyant LED indiquant une erreur système est allumé sur le panneau de commande avant. Que dois-je faire ?
R. Deux voyants LED se trouvent sous le bouton d’alimentation situé à l’avant d’un appareil Data Box. Le voyant le plus bas est l’indicateur d’une erreur système, qui indique si votre système est sain.

Un voyant rouge signalant une erreur système peut indiquer l’un des problèmes suivants :
- Panne de ventilateur
- Température élevée de l’UC
- Température élevée de la carte mère
- Erreur du code de correction d’erreur (ECC) du DIMM (Dual Inline Memory Module)

Suivez ces étapes :
1. Vérifiez si le ventilateur fonctionne.
2. Déplacez l’appareil dans un endroit où le flux d’air est plus important.

Si le voyant d’erreur système est allumé, [contact Support Microsoft](data-box-disk-contact-microsoft-support.md).

### <a name="q-i-cant-access-the-data-box-unlock-password-in-the-azure-portal-why"></a>Q. Impossible d’accéder au mode de passe de déverrouillage Data Box dans le portail Azure. Pourquoi ?
R. Si vous ne pouvez pas accéder au mot de passe de déverrouillage dans le portail Azure, vérifiez les autorisations de votre abonnement et de votre compte de stockage. Assurez-vous de disposer des autorisations de collaborateur ou de propriétaire au niveau du groupe de ressources. Vous devez avoir au moins une autorisation de rôle Opérateur Data Box pour voir les identifiants d’accès.

### <a name="q-is-port-channel-configuration-supported-on-data-box-how-about-mpio"></a>Q. La configuration du canal de port est-elle prise en charge sur Data Box ? Qu’en est-il de MPIO ?
R. Nous ne prenons pas en charge la configuration du canal de port, la configuration MPIO (Multipath I/O) ou la configuration du réseau local virtuel (VLAN) sur Data Box.

## <a name="track-status"></a>Suivi de l'état

### <a name="q-how-do-i-track-the-data-box-from-when-i-placed-the-order-to-shipping-the-device-back"></a>Q. Comment faire pour suivre la Data Box depuis sa commande jusqu’à son renvoi ? 
R.  Vous pouvez suivre l’état de la commande Data Box dans le portail Azure. Lorsque vous créez la commande, vous êtes invité à fournir une adresse e-mail pour recevoir des notifications. Si vous en avez fourni une, vous êtes averti par e-mail de tous les changements d’état de la commande. Plus d’informations sur la manière de [configurer les notifications par e-mail](data-box-portal-ui-admin.md#edit-notification-details).

### <a name="q-how-do-i-return-the-device"></a>Q. Comment faire pour renvoyer l’appareil ? 
R.  Microsoft affiche une étiquette d’expédition sur l’écran E-ink. Si l’étiquette d’expédition n’apparaît pas sur l’écran E-ink, accédez à **Vue d’ensemble > Télécharger une étiquette d’expédition**. Téléchargez et imprimez l’étiquette, insérez-la dans l’onglet en plastique transparent fixé sur l’appareil, puis déposez l’appareil au point de dépôt de votre transporteur. 

### <a name="q-i-received-an-email-notification-that-my-device-has-reached-the-azure-datacenter-how-do-i-find-out-if-the-data-upload-is-in-progress"></a>Q. J’ai reçu une notification par e-mail que mon appareil est arrivé au centre de données Azure. Comment savoir si le chargement des données est en cours ?
R. Vous pouvez accéder à votre commande Data Box dans la **vue d’ensemble** du portail Azure. Si le chargement des données vers Azure a démarré, vous verrez la progression de la copie dans le volet droit. 

## <a name="migrate-data"></a>Migration des données

### <a name="q-what-is-the-maximum-data-size-that-can-be-used-with-data-box"></a>Q. Quel volume de données puis-je utiliser au maximum avec Data Box ?  
R.  Data Box a une capacité de stockage utile de 80 To. Vous pouvez utiliser un seul appareil Data Box pour les données dont la taille est comprise entre 40 To et 80 To. Pour les tailles de données supérieures à 500 To, vous pouvez commander plusieurs appareils Data Box. Pour les tailles de données supérieures à 500 To, optez pour Data Box Heavy.  

### <a name="q-what-are-the-maximum-block-blob-and-page-blob-sizes-supported-by-data-box"></a>Q. Quelle est la taille maximale des objets blob de blocs et des objets blob de pages prise en charge par Data Box ? 
R.  Les tailles maximales dépendent des limites du Stockage Azure. La taille maximale pour un objet blob de blocs est d’environ 4,768 Tio et la taille maximale pour un objet blob de page est de 8 Tio. Pour plus d’informations, consultez [Objectifs de performance et de scalabilité pour le stockage d’objets blob](../storage/blobs/scalability-targets.md).

### <a name="q-how-do-i-know-that-my-data-is-secure-during-transit"></a>Q. Comment faire pour savoir si mes données sont sécurisées pendant le transfert ? 
R. Plusieurs fonctionnalités de sécurité ont été implémentées pour garantir la sécurité de votre Data Box pendant le transit. Certaines d'entre elles incluent des scellés inviolables, une protection contre la falsification du matériel et des logiciels, ainsi qu’un mot de passe de déverrouillage de l’appareil. Pour plus d’informations, accédez à [Azure Data Box security and data protection](data-box-security.md) (Protection des données et sécurité Azure Data Box).

### <a name="q-how-do-i-copy-the-data-to-the-data-box"></a>Q. Comment faire pour copier des données sur la Data Box ? 
R.  Si vous utilisez un client SMB, vous pouvez exploiter un outil de copie SMB tel que `Robocopy`, `Diskboss`, voire la fonction glisser-déplacer d’Explorateur de fichiers Windows pour copier des données sur l’appareil. 

Si vous utilisez un client NFS, vous pouvez utiliser [rsync](https://rsync.samba.org/), [FreeFileSync](https://www.freefilesync.org/), [Unison](https://www.cis.upenn.edu/~bcpierce/unison/) ou [Ultracopier](https://ultracopier.first-world.info/). 

Pour plus d'informations, consultez [Tutoriel : Copier des données sur Azure Data Box](data-box-deploy-copy-data.md).

### <a name="q-are-there-any-tips-to-speed-up-the-data-copy"></a>Q. Y a-t-il des astuces permettant d’accélérer la copie des données ?
R.  Pour accélérer le processus de copie :

- Utilisez plusieurs flux de copie des données. Par exemple, avec `Robocopy`, utilisez l’option multithread. Pour plus d’informations sur la commande exacte utilisée, accédez à [Didacticiel : Copier des données sur Azure Data Box et procéder à une vérification](data-box-deploy-copy-data.md).
- Utilisez plusieurs sessions.
- Au lieu de copier sur un partage réseau (où les vitesses réseau peuvent limiter la vitesse de copie), stockez les données localement sur l’ordinateur auquel l’appareil Data Box est connecté.
- Évaluez les performances de l’ordinateur utilisé pour copier les données. Téléchargez et utilisez l’[outil FIO `Bluestop`](https://ci.appveyor.com/project/axboe/fio) pour tester les performances du matériel du serveur. Sélectionnez la build x86 ou x64 la plus récente, l’onglet **Artefacts**, puis téléchargez le fichier MSI.

<!--### Q. How to speed up the data copy if the source data has small files (KBs or few MBs)?
A.  To speed up the copy process:

- Create a local VHDx on fast storage or create an empty VHD on the HDD/SSD (slower).
- Mount it to a VM.
- Copy files to the VM's disk.-->


### <a name="q-can-i-use-multiple-storage-accounts-with-data-box"></a>Q. Puis-je utiliser plusieurs comptes de stockage avec Data Box ?
R.  Oui. Data Box prend en charge jusqu’à 10 comptes de stockage, à usage général, classiques ou de stockage d’objets blob. Les objets blob chauds et froids sont pris en charge.


## <a name="ship-device"></a>Expédier un appareil

<!--### Q. How do I schedule a pickup for my Data Box?--> 

### <a name="q-my-device-was-delivered-but-the-device-seems-to-be-damaged-what-should-i-do"></a>Q. Mon appareil a été livré, mais il semble endommagé. Que dois-je faire ?
R. Si votre appareil est arrivé endommagé ou présente des signes de falsification, ne l’utilisez pas. [Contactez Support Microsoft](data-box-disk-contact-microsoft-support.md) et retournez l’appareil dès que possible. Vous pouvez également créer une nouvelle commande Data Box pour obtenir un appareil de remplacement. Dans ce cas, l’appareil de remplacement ne vous sera pas facturé.

### <a name="q-can-i-pick-up-my-data-box-order-myself-can-i-return-the-data-box-via-a-carrier-that-i-choose"></a>Q. Puis-je récupérer ma commande Data Box moi-même ? Puis-je retourner la Data Box via le transporteur de mon choix ?
R. Oui. Microsoft propose également des expéditions autogérées. Quand vous commandez la Data Box, vous pouvez choisir l’option d’expédition autogérée. Pour plus d’informations, consultez [l’expédition autogérées pour Data Box](data-box-portal-customer-managed-shipping.md).

### <a name="q-will-my-data-box-devices-cross-countryregion-borders-during-shipping"></a>Q. Mes appareils Data Box traverseront-t-il des frontières lors de leur expédition ?
R. Les appareils Data Box sont expédiés depuis le même pays ou la même région que leur destination, ils ne franchissent aucune frontière internationale. La seule exception concerne les commandes au sein de l’Union européenne (UE), où les appareils peuvent être expédiés depuis et vers n’importe quel pays ou n’importe quelle région de l’UE. Cela concerne les appareils Data Box et Data Box Heavy.

### <a name="q-i-ordered-a-data-box-in-us-east-but-i-received-a-device-that-was-shipped-from-a-location-in-us-west-where-should-i-return-the-device-to"></a>Q. J’ai commandé un Data Box dans la région USA Est, mais j’ai reçu un appareil qui a été expédié à partir d’un emplacement dans la région USA Ouest. Où renvoyer l’appareil ?
R. Nous essayons de vous procurer un appareil Data Box le plus rapidement possible. Nous donnons la priorité à l’envoi à partir du centre de données le plus proche de l’emplacement de votre compte de stockage, mais nous expédions toujours un appareil à partir d’un centre de données Azure disposant d’un stock. Votre Data Box doit être retourné au même emplacement que celui où il a été expédié, tel qu’affiché sur l’étiquette d’expédition.

### <a name="q-e-ink-display-is-not-showing-the-return-shipment-label-what-should-i-do"></a>Q. L’écran E-ink n’affiche pas l’étiquette de retour. Que dois-je faire ?
R. Si l’écran E-ink n’affiche pas l’étiquette de retour, procédez comme suit :
- Décollez l’ancienne étiquette d’expédition et l’éventuel autocollant du précédent colis.
- Accédez à votre commande dans le portail Azure. Accédez à **Vue d’ensemble**, puis à **Télécharger une étiquette d’expédition**. Pour plus d’informations, consultez [Télécharger une étiquette d’expédition](data-box-portal-admin.md#download-shipping-label).
- Imprimez l’étiquette d’expédition et insérez-la dans la pochette en plastique transparent accolée à l’appareil. 
- Vérifiez que l’étiquette d’expédition est clairement visible. 

### <a name="q-how-is-my-data-protected-during-transit"></a>Q. Comment mes données sont-elles protégées pendant le transfert ? 
R.  Au cours du transit, les fonctionnalités suivantes de l’aide Data Box protègent les données.
 - Les disques Data Box sont chiffrés à l’aide du chiffrement AES 256 bits. 
 - L’appareil est verrouillé et nécessite un mot de passe de déverrouillage pour saisir des données et y accéder.
Pour plus d’informations, consultez [Fonctionnalités de sécurité Data Box](data-box-security.md).  

### <a name="q-i-have-finished-prepare-to-ship-for-my-import-order-and-shut-down-the-device-can-i-still-add-more-data-to-the-data-box"></a>Q. J’ai terminé la préparation de la livraison pour ma commande d’importation et j’ai arrêté l’appareil. Puis-je quand même ajouter des données à l’appareil Data Box ?
R. Oui. Vous pouvez allumer l’appareil et ajouter d’autres données. Vous devrez relancer **Préparer l’expédition**, une fois la copie des données terminée.

### <a name="q-i-received-my-device-and-it-is-not-booting-up-how-do-i-ship-the-device-back"></a>Q. J’ai reçu mon appareil mais il ne démarre pas. Comment le réexpédier ?
R. Si votre appareil ne démarre pas, accédez à votre commande dans le portail Azure. Téléchargez une étiquette d’expédition et attachez-la à l’appareil. Pour plus d’informations, consultez [Télécharger une étiquette d’expédition](data-box-portal-admin.md#download-shipping-label).

## <a name="verify-and-upload"></a>Vérification et chargement

### <a name="q-how-soon-can-i-access-my-data-in-azure-once-ive-shipped-the-data-box-back"></a>Q. Après combien de temps puis-je accéder à mes données dans Azure une fois que j’ai renvoyé la Data Box ? 
R.  Dès que l’état de la commande pour la **copie des données** s’affiche comme **complet**, vous devriez pouvoir accéder à vos données.

### <a name="q-where-is-my-data-located-in-azure-after-the-upload"></a>Q. Où se trouvent mes données dans Azure après leur téléchargement ?
R.  Quand vous copiez les données sur Data Box, selon qu’il s’agit d’objets blob de blocs, d’objets blob de pages ou de fichiers Azure, elles sont chargées sur l’un des chemins suivants dans votre compte Stockage Azure :
- `https://<storage_account_name>.blob.core.windows.net/<containername>` 
- `https://<storage_account_name>.file.core.windows.net/<sharename>`
 
  Vous pouvez également accéder à votre compte de stockage Azure dans le portail Azure et naviguer à partir de cet emplacement.

### <a name="q-i-just-noticed-that-i-did-not-follow-the-azure-naming-requirements-for-my-containers-will-my-data-fail-to-upload-to-azure"></a>Q. Je viens de remarquer que je n’ai pas suivi les conventions de dénomination pour mes conteneurs Azure. Le chargement de mes données sur Azure risque-t-il d’échouer ?
R.  Si les noms des conteneurs contiennent des lettres majuscules, ils sont automatiquement convertis en minuscules. Si les noms ne sont pas conformes d’une autre manière (caractères spéciaux, autres langues, etc.), le chargement échouera. Pour plus d’informations sur l’affectation de noms aux partages, conteneurs et fichiers, consultez :
- [Affectation de noms et de références aux partages](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata)
- [Conventions d’objets blob de blocs et d’objets blob de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).

### <a name="q-how-do-i-verify-the-data-i-copied-onto-data-box"></a>Q. Comment faire pour vérifier les données que j’ai copiées sur Data Box ?
R.  Une fois la copie des données terminée, lorsque vous exécutez **Préparer l’expédition**, vos données sont validées. Data Box génère une liste de fichiers et de sommes de contrôle pour les données pendant le processus de validation. Vous pouvez télécharger cette liste de fichiers et la comparer aux fichiers de la source de données. Pour plus d’informations, consultez [Préparer l’expédition](data-box-deploy-picked-up.md#prepare-to-ship).

### <a name="q-what-happens-to-my-data-after-i-return-the-data-box"></a>Q. Que deviennent mes données une fois que j’ai renvoyé l’appareil Data Box ?
R.  Une fois la copie des données vers Azure terminée, les données des disques sur la Data Box sont effacées en toute sécurité, conformément aux instructions du standard NIST SP 800-88 Revision 1. Pour plus d’informations, consultez [Effacer les données de la Data Box](data-box-deploy-picked-up.md#erasure-of-data-from-data-box).

## <a name="audit-report"></a>Rapport d’audit

### <a name="how-does-azure-data-box-service-help-support-customers-chain-of-custody-procedure"></a>Comment le service Azure Data Box peut-il m’aider à prendre en charge la chaîne de responsabilité des clients ?
R.  En mode natif, le service Azure Data Box fournit des rapports que vous pouvez utiliser pour documenter votre chaîne de responsabilité. Les journaux d’activité d’audit et de copie sont disponibles dans votre compte de stockage dans Azure, et vous pouvez [télécharger l’historique des commandes](data-box-portal-admin.md#download-order-history) dans le portail Azure, une fois la commande terminée.


### <a name="what-type-of-reporting-is-available-to-support-chain-of-custody"></a>Quels sont les types de rapports disponibles pour prendre en charge la chaîne de responsabilité ?
R.  Les rapports suivants sont disponibles pour prendre en charge la chaîne de responsabilité :

- Logistique du transport UPS.
- Journalisation de la mise sous tension et de l’accès au partage utilisateur.
- BOM ou fichier manifeste avec une vérification de redondance cyclique 64 bits (CRC-64) ou somme de contrôle pour chaque fichier ingéré avec succès dans la Data Box.
- Création de rapports sur les fichiers qui n’ont pas pu être téléchargés vers le compte Stockage Azure.
- Nettoyage de l’appareil Data Box (conformément aux normes NIST 800 88R1), une fois les données copiées sur votre compte Stockage Azure.

### <a name="are-the-carrier-tracking-logs-from-ups-available"></a>Les journaux d’activité de suivi du transporteur (UPS) sont-ils disponibles ? 
R.  Les journaux d’activité de suivi du transporteur sont consignés dans l’historique des commandes Data Box. Ce rapport est à votre disposition une fois que l’appareil a été retourné au centre de données Azure et que les données stockées sur les disques de l’appareil ont été nettoyées. Pour vos besoins immédiats, vous pouvez également accéder directement au site web du transporteur et y entrer le numéro de suivi de la commande pour obtenir les informations de suivi.

### <a name="can-i-transport-the-data-box-to-azure-datacenter"></a>Puis-je apporter la Data Box au centre de données Azure ? 
R.  Non. Si vous avez choisi la livraison gérée par Microsoft, vous ne pouvez pas modifier les données. Actuellement, le centre de données Azure n’accepte pas la remise d’appareils Data Box des clients ou de transporteurs autres qu’UPS.

Si vous avez choisi l’expédition autogérée, vous pouvez récupérer ou déposer votre appareil Data Box auprès du centre de données Azure.


## <a name="next-steps"></a>Étapes suivantes

- Relire les [Conditions requises pour le système Data Box](data-box-system-requirements.md).
- Comprendre les limites de [Data Box](data-box-limits.md).
- Déployez rapidement [Azure Data Box](data-box-quickstart-portal.md) sur le portail Azure.