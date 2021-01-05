---
title: FAQ Microsoft Azure Data Box Disk | Microsoft Docs sur les données
description: Contient les questions fréquentes et leurs réponses au sujet d’Azure Data Box Disk, une solution cloud qui vous permet de transférer de grandes quantités de données dans Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: alkohli
ms.openlocfilehash: acaa0fdb1fa0e9a338f4e249622cad0115c6b58e
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516444"
---
# <a name="azure-data-box-disk-frequently-asked-questions"></a>Azure Data Box Disk : Forum Aux Questions (FAQ)

La solution de cloud Microsoft Azure Data Box Disk vous permet d’envoyer des téraoctets de données vers Azure de façon rapide, économique et fiable. Cette FAQ contient des réponses aux questions que vous pouvez vous poser lorsque vous utilisez Data Box Disk sur le portail Azure. 

Les questions/réponses sont classées dans les catégories suivantes :

- À propos du service
- Configuration et connexion 
- Suivi de l'état
- Migration des données 
- Vérification et chargement de données 


## <a name="about-the-service"></a>À propos du service

### <a name="q-what-is-azure-data-box-service"></a>Q. Qu’est-ce que le service Azure Data Box ? 
R.  Le service Azure Data Box a été conçu pour l’ingestion des données hors connexion. Ce service gère une gamme de produits tous conçus pour le transport des données pour différentes capacités de stockage. 

### <a name="q-what-are-azure-data-box-disks"></a>Q. Que sont les disques Azure Data Box ?
R. Les disques Azure Data Box permettent un transfert rapide, économique et sécurisé de plusieurs téraoctets de données sur et depuis Azure. Microsoft vous envoie entre 1 et 5 disques d’une capacité de stockage maximale de 35 To. Vous pouvez facilement configurer, connecter et déverrouiller ces disques via le service Data Box du portail Azure.  

Les disques sont chiffrés à l’aide du chiffrement de lecteur BitLocker et vos clés de chiffrement sont gérées sur le portail Azure. Ensuite, vous copiez les données à partir des serveurs du client. Dans le centre de données, Microsoft migre vos données depuis le disque vers le cloud en utilisant une liaison montante rapide de réseau privé, puis les charge dans Azure.

### <a name="q-when-should-i-use-data-box-disks"></a>Q. Quand dois-je utiliser des disques Data Box ?
R. Si vous souhaitez transférer vers Azure 40 To de données (maximum), les disques Data Box pourront vous être utiles.

### <a name="q-what-is-the-price-of-data-box-disks"></a>Q. Combien coûtent les disques Data Box ?
R. Pour plus d’informations sur les tarifs des disques Data Box Disk, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/databox/disk/).

### <a name="q-how-do-i-get-data-box-disks"></a>Q. Comment faire pour obtenir des disques Data Box ? 
R.  Pour obtenir des disques Azure Data Box Disk, connectez-vous au portail Azure et créez une commande de disques Data Box Disk. Indiquez vos coordonnées et vos paramètres de notification. Une fois que vous avez passé une commande, les disques sont expédiés dans les 10 jours, en fonction de leur disponibilité.

### <a name="q-what-is-the-maximum-amount-of-data-i-can-transfer-with-data-box-disks-in-one-instance"></a>Q. Quelle est la quantité maximale de données que je peux transférer avec les disques Data Box dans une seule instance ?
R. Pour 5 disques de 8 To chacun (7 To de capacité utilisable), la capacité utilisable maximale est de 35 To. Par conséquent, vous pouvez transférer 35 To de données dans une seule instance. Pour transférer davantage de données, vous devez commander davantage de disques.

### <a name="q-how-can-i-check-if-data-box-disks-are-available-in-my-region"></a>Q. Comment puis-je vérifier si les disques Data Box sont disponibles dans ma région ? 
R.  Pour voir où les disques Data Box Disk sont actuellement disponibles, accédez à [Disponibilité dans la région](data-box-disk-overview.md#region-availability).  

### <a name="q-which-regions-can-i-store-data-in-with-data-box-disks"></a>Q. Dans quelles régions puis-je stocker des données avec des disques Data Box ?
R. La solution Data Box Disk est prise en charge dans toutes les régions des États-Unis, du Canada, de l’Australie, de l’Europe Ouest, de l’Europe Nord, de la Corée et du Japon. Seules les régions du cloud public Azure sont prises en charge. Azure Government et les autres clouds souverains ne sont pas pris en charge.

### <a name="q-which-regions-can-i-store-data-in-with-data-box-disks"></a>Q. Dans quelles régions puis-je stocker des données avec des disques Data Box ?
R. La solution Data Box Disk est prise en charge dans toutes les régions des États-Unis, du Canada, de l’Australie, de l’Europe Ouest, de l’Europe Nord, de la Corée et du Japon. Seules les régions du cloud public Azure sont prises en charge. Azure Government et les autres clouds souverains ne sont pas pris en charge.

### <a name="q-how-can-i-import-source-data-present-at-my-location-in-one-countryregion-to-an-azure-region-in-a-different-country"></a>Q. Comment importer des données sources présentes sur le site où je me trouve dans un pays/une région vers une région Azure située dans un autre pays ?
R. Data Box Disk prend en charge l'ingestion de données, mais uniquement dans le même pays/la même région que leur destination. Aucune frontière internationale ne peut être franchie. La seule exception concerne les commandes au sein de l'Union européenne (UE), où les disques Data Box Disk peuvent être expédiés depuis et vers n'importe quel pays/région de l'UE.

Par exemple, si vous souhaitez transférer des données de votre emplacement au Canada vers un compte de stockage Azure USA Ouest, vous pouvez procéder comme suit :

### <a name="option-1"></a>Option 1 : 

Expédiez un [disque pris en charge](../storage/common/storage-import-export-requirements.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#supported-disks) contenant des données à l’aide du [service Azure Import/Export](../storage/common/storage-import-export-service.md) depuis l’emplacement source au Canada vers le centre de données Azure USA Ouest.

### <a name="option-2"></a>Option n°2 :

1. Commandez un appareil Data Box Disk au Canada en choisissant un compte de stockage, par exemple dans la région Canada Centre. Le ou les disques SSD sont expédiés du centre de données Azure situé dans la région Canada Centre à l'adresse de livraison (au Canada) fournie lors de la création de la commande.

2. Une fois les données de votre serveur local copiées sur les disques, renvoyez-les au centre de données Azure au Canada en utilisant les étiquettes de retour fournies par Microsoft. Les données présentes sur le ou les disques Data Box Disk sont ensuite chargées sur le compte de stockage de destination dans la région canadienne Azure choisie lors de la création de la commande.

3. Vous pouvez ensuite utiliser un outil comme AzCopy pour copier les données dans un compte de stockage situé dans la région USA Ouest. Cette étape entraîne des frais de [stockage standard](https://azure.microsoft.com/pricing/details/storage/) ainsi que des frais de [bande passante](https://azure.microsoft.com/pricing/details/bandwidth/) qui ne sont pas inclus dans la facturation de Data Box Disk.

### <a name="q-whom-should-i-contact-if-i-encounter-any-issues--with-data-box-disks"></a>Q. Qui dois-je contacter si je rencontre des problèmes avec des disques Data Box ?
R. Si vous rencontrez des problèmes avec des appareils Data Box Disk, [contactez Support Microsoft](./data-box-disk-contact-microsoft-support.md).

## <a name="order-device"></a>Commander un appareil

### <a name="q-how-do-i-get-data-box-disk"></a>Q. Comment obtenir un appareil Data Box Disk ? 
R.  Pour obtenir un appareil Azure Data Box Disk, connectez-vous au portail Azure et créez une commande Data Box Disk. Indiquez vos coordonnées et vos paramètres de notification. Une fois que vous avez passé commande, un appareil Data Box Disk vous sera expédié sous 10 jours, en fonction de la disponibilité. Pour plus d’informations, accédez à [Commander une Data Box](data-box-disk-deploy-ordered.md).

### <a name="q-i-couldnt-create-a-data-box-disk-order-in-the-azure-portal-why"></a>Q. Je n’ai pas pu créer de commande Data Box Disk dans le portail Azure. Pourquoi ?
R. Si vous ne pouvez pas créer de commande Data Box Disk, le problème vient de votre type d’abonnement ou de votre accès.

Vérifiez votre abonnement. Data Box Disk est disponible uniquement pour les offres d’abonnement Contrat Entreprise et Fournisseur de solutions Cloud. Si vous ne disposez pas de l’un de ces types d’abonnement, contactez Support Microsoft pour mettre à niveau votre abonnement.

Si vous disposez d’un type d’offre pris en charge pour l’abonnement, vérifiez le niveau d’accès de votre abonnement. Vous devez être contributeur ou propriétaire de votre abonnement pour créer une commande.

### <a name="q-how-long-will-my-order-take-from-order-creation-to-data-uploaded-to-azure"></a>Q. Combien de temps s’écoulera entre la création de ma commande et le chargement des données vers Azure ?

R. Les délais estimés suivants pour chaque phase du traitement des commandes vous donneront une bonne idée de ce à quoi vous pouvez vous attendre.  

Ces délais sont des *estimations*. Le temps nécessaire à chaque étape du traitement des commandes est déterminé par la charge du centre de données, les commandes simultanées et d’autres conditions environnementales.

**Délais estimés pour une commande Data Box Disk :**

1. Commande d’un appareil Data Box Disk : quelques minutes, à partir du portail
2. Allocation et préparation du disque : 1 à 2 jours ouvrables
3. Expédition : 2 à 3 jours ouvrables
4. Copie des données sur le site du client : dépend de la nature des données, de la taille et du nombre de fichiers
5. Réexpédition : 2 à 3 jours ouvrables
6. Traitement du disque au centre de données : 1 à 2 jours ouvrables
7. Charger les données sur Azure : commence dès que le traitement est terminé et que le disque est connecté. Le temps de chargement dépend de la nature des données, de la taille et du nombre de fichiers.


## <a name="configure-and-connect"></a>Configuration et connexion
 
### <a name="q-can-i-specify-the-number-of-data-box-disks-in-the-order"></a>Q. Puis-je spécifier le nombre de disques Data Box dans ma commande ?
R.  Non. Vous recevez des disques de 8 To (5 disques au maximum) en fonction du volume de vos données et de la disponibilité des disques.  

### <a name="q-how-do-i-unlock-the-data-box-disks"></a>Q. Comment faire pour déverrouiller les disques Data Box ? 
R.  Dans le portail Azure, accédez à votre commande Data Box Disk et accédez à **Informations sur l’appareil**. Copiez la clé d’accès. Téléchargez et extrayez l’outil de déverrouillage Data Box Disk depuis le Portail Azure pour votre système d’exploitation. Exécutez l’outil sur l’ordinateur qui héberge les données que vous voulez copier sur les disques. Entrez la clé d'accès pour déverrouiller vos disques. La même clé d’accès déverrouille tous les disques. 

Pour obtenir des instructions détaillées, accédez à [Unlock disks on a Windows client](data-box-disk-deploy-set-up.md#unlock-disks-on-windows-client) (Déverrouiller des disques sur un client Windows) ou [Unlock disks on a Linux client](data-box-disk-deploy-set-up.md#unlock-disks-on-linux-client) (Déverrouiller des disques sur un client Linux).

### <a name="q-can-i-use-a-linux-host-computer-to-connect-and-copy-the-data-on-to-the-data-box-disks"></a>Q. Puis-je utiliser un ordinateur hôte Linux pour me connecter et copier les données sur les disques Data Box ?
R.  Oui. Le client Linux et le client Windows peuvent être utilisés pour connecter et copier des données sur les disques Data Box Disk. Pour plus d’informations, accédez à la liste des [systèmes d’exploitation pris en charge](data-box-disk-system-requirements.md) pour votre ordinateur hôte.

### <a name="q-my-disks-are-dispatched-but-now-i-want-to-cancel-this-order-why-is-the-cancel-button-not-available"></a>Q. Mes disques ont été expédiés, mais je souhaite quand même annuler ma commande. Pourquoi le bouton Annuler n’est-il pas disponible ?
R.  Vous pouvez annuler une commande passée, mais avant l’expédition des disques uniquement. Une fois que les disques ont été expédiés, vous ne pouvez plus annuler la commande. Toutefois, vous pouvez retourner vos disques à vos frais. 

### <a name="q-can-i-connect-multiple-data-box-disks-at-the-same-to-the-host-computer-to-transfer-data"></a>Q. Puis-je connecter plusieurs disques Data Box à mon ordinateur hôte simultanément pour transférer des données ?
R. Oui. Vous pouvez connecter plusieurs disques Data Box au même ordinateur hôte pour transférer des données et plusieurs travaux de copie peuvent être exécutés en parallèle.

## <a name="track-status"></a>Suivi de l'état

### <a name="q-how-do-i-track-the-disks-from-when-i-placed-the-order-to-shipping-the-disks-back"></a>Q. Comment faire pour suivre les disques depuis leur commande à leur renvoi ? 
R.  Vous pouvez suivre l’état de la commande de disques Data Box dans le portail Azure. Lorsque vous créez la commande, vous êtes également invité à fournir une adresse e-mail pour recevoir des notifications. Si vous en avez fourni une, vous serez averti par e-mail de toutes les modifications d’état de la commande. Plus d’informations sur la manière de [configurer les notifications par e-mail](data-box-portal-ui-admin.md#edit-notification-details).

### <a name="q-how-do-i-return-the-disks"></a>Q. Comment faire pour renvoyer les disques ? 
R.  Microsoft inclut une étiquette d’expédition dans le colis contenant les disques Data Box. Collez l’étiquette sur la boîte d’envoi et déposez le paquet scellé chez votre livreur. Si vous avez endommagé ou perdu l’étiquette, vous pouvez accéder à **Vue d’ensemble > Télécharger une étiquette d'expédition** et télécharger une nouvelle étiquette d’expédition.

### <a name="can-i-pick-up-my-data-box-disk-order-myself-can-i-return-the-disks-via-a-carrier-that-i-choose"></a>Puis-je récupérer ma commande Data Box Disk moi-même ? Puis-je retourner les disques via un transporteur de mon choix ?
R. Oui. Microsoft propose également des expéditions autogérées dans la région US Gov uniquement. Quand vous commandez le Data Box Disk, vous pouvez choisir l’option d’expédition autogérée. Pour récupérer votre commande Data Box Disk, effectuez les étapes suivantes :
    
1. Une fois la commande passée, elle est traitée et les disques sont préparés. Vous recevez une notification par e-mail indiquant que votre commande est prête à être récupérée. 
2. Une fois la commande prête à être récupérée, accédez à votre commande dans le portail Azure et accédez au panneau **Vue d’ensemble**. 
3. Vous voyez une notification avec un code dans le portail Azure. Envoyez un e-mail à l’[équipe des opérations Azure Data Box](mailto:adbops@microsoft.com) et fournissez-lui le code. L’équipe vous indique alors l’emplacement et planifie une date et une heure pour la récupération. Vous devez appeler l’équipe dans les 5 jours ouvrables suivant la réception de la notification par e-mail.

Une fois la copie et la validation des données terminées, effectuez les étapes suivantes pour renvoyer votre disque :

1. Une fois que la validation des données est terminée, déconnectez les disques. Débranchez les câbles de connexion.
2. Emballez l’ensemble des disques et câbles de connexion dans un emballage à bulles et placez le tout dans la boîte d’expédition. Des frais peuvent s’appliquer si les accessoires sont manquants.

    - Réutilisez l’emballage de la première expédition. Nous vous recommandons d’emballer les disques avec du papier bulle pour les protéger.
    - Vérifiez qu’ils sont suffisamment serrés dans la boîte pour éviter qu’ils ne bougent.
3. Accédez au **panneau Vue d’ensemble** de votre commande dans le portail Azure. Vous devez voir une notification avec un code.
4. Utilisez ce code, et envoyez un e-mail à l’[équipe des opérations Azure Data Box](mailto:adbops@microsoft.com) en lui fournissant ce code. Ils vous indiqueront où et quand déposer les disques.


## <a name="migrate-data"></a>Migration des données

### <a name="q-what-is-the-maximum-data-size-that-can-be-used-with-data-box-disks"></a>Q. Quel volume de données puis-je utiliser au maximum avec des disques Data Box ?  
R.  La solution de disques Data Box peut inclure jusqu'à 5 disques avec une capacité utilisable maximale de 35 To. Les disques eux-mêmes font 8 To (7 To utilisables).

### <a name="q-what-are-the-maximum-block-blob-and-page-blob-sizes-supported-by-data-box-disks"></a>Q. Quelle est la taille maximale des objets blob de blocs et des objets blob de pages prise en charge par chaque disque Data Box ? 
R.  Les tailles maximales dépendent des limites du Stockage Azure. La taille maximale pour un objet blob de blocs est d’environ 4,768 Tio et la taille maximale pour un objet blob de page est de 8 Tio. Pour plus d’informations, consultez [Objectifs de performance et de scalabilité pour le stockage d’objets blob](../storage/blobs/scalability-targets.md).

### <a name="q-what-is-the-data-transfer-speed-for-data-box-disks"></a>Q. Quelle est la vitesse de transfert des données des disques Data Box ?
R. Lors de tests utilisant des disques connectés via USB 3.0, les performances des disques ont atteint 430 Mo/s. Les chiffres réels varient en fonction de la taille de fichier utilisée. Pour les fichiers plus petits, vous pouvez constater des performances moins élevées.

### <a name="q-how-do-i-know-that-my-data-is-secure-during-transit"></a>Q. Comment faire pour savoir si mes données sont sécurisées pendant le transfert ? 
R.  Les disques Data Box sont chiffrés à l’aide du chiffrement BitLocker AES-128 bits et la clé d’accès est uniquement disponible dans le portail Azure. Connectez-vous au portail Azure à l’aide de vos informations d’identification pour obtenir la clé d'accès. Vous devez fournir cette clé d’accès lorsque vous exécutez l’outil de déverrouillage Data Box Disk.

### <a name="q-how-do-i-copy-the-data-to-the-data-box-disks"></a>Q. Comment faire pour copier des données sur les disques Data Box ? 
R.  Utilisez un outil de copie SMB tel que `Robocopy`, `Diskboss` ou même la fonction glisser-déplacer d’Explorateur de fichiers Windows pour copier des données sur les disques.

### <a name="q-are-there-any-tips-to-speed-up-the-data-copy"></a>Q. Y a-t-il des astuces permettant d’accélérer la copie des données ?
R.  Pour accélérer le processus de copie :

- Utilisez plusieurs flux de copie des données. Par exemple, avec `Robocopy`, utilisez l’option multithread. Pour plus d’informations sur la commande exacte utilisée, accédez à [Didacticiel : Copier des données sur Azure Data Box Disk et procéder à une vérification](data-box-disk-deploy-copy-data.md#copy-data-to-disks).
- Utilisez plusieurs sessions.
- Au lieu de copier sur un partage réseau (où vous pouvez être limité par la vitesse du réseau), vérifiez que les données résident en local sur l’ordinateur auquel les disques sont connectés.
- Vérifiez que vous utilisez l’USB 3.0 ou une version ultérieure tout au long du processus de copie. Téléchargez et utilisez l’[outil `USBView`](/windows-hardware/drivers/debugger/usbview) pour identifier les contrôleurs USB et les périphériques USB connectés à l’ordinateur.
- Évaluez les performances de l’ordinateur utilisé pour copier les données. Téléchargez et utilisez l’[outil `FIO` Bluestop](https://ci.appveyor.com/project/axboe/fio) pour tester les performances du matériel du serveur. Sélectionnez la build x86 ou x64 la plus récente, l’onglet **Artefacts**, puis téléchargez le fichier MSI.

### <a name="q-how-to-speed-up-the-data-if-the-source-data-has-small-files-kbs-or-few-mbs"></a>Q. Comment accélérer les données si la source de données contient des petits fichiers (quelques Ko ou Mo seulement) ?
R.  Pour accélérer le processus de copie :

- Créez un VHDx local sur un stockage rapide ou créez un disque dur virtuel vide sur le HDD/SSD (plus lent).
- Montez-le sur une machine virtuelle.
- Copiez les fichiers sur le disque de la machine virtuelle.

### <a name="q-can-i-use-multiple-storage-accounts-with-data-box-disks"></a>Q. Puis-je utiliser plusieurs comptes de stockage avec les disques Data Box ?
R.  Non. Pour l’instant, seul un compte de stockage, général ou classique, est pris en charge par les disques Data Box. Les objets blob chauds et froids sont pris en charge. Seuls les comptes de stockage aux États-Unis, en Europe Ouest et en Europe Nord dans le cloud public Azure sont actuellement pris en charge.

### <a name="q-what-is-the-toolset-available-for-my-data-with-data-box-disks"></a>Q. Quel est l’ensemble d’outils disponible pour mes données avec des disques Data Box Disk ?
R. L’ensemble d’outils Data Box Disk contient trois outils :
 - **Outil de déverrouillage Data Box Disk** : utilisez cet outil pour déverrouiller les disques chiffrés qui sont fournis par Microsoft. Au déverrouillage des disques à l’aide de l’outil, vous devez fournir une clé d’accès indiquée sur la commande Data Box Disk dans le portail Azure. 
 - **Outil de validation Data Box Disk** : utilisez cet outil pour valider la taille, le format et les noms d’objets blob conformément aux conventions de nommage Azure. Il génère également des sommes de contrôle pour les données copiées qui sont ensuite utilisées pour vérifier les données chargées sur Azure.
 - **Outil de copie et de division Data Box Disk** : utilisez cet outil si vous avez plusieurs disques et que vous devez diviser et copier un jeu de données volumineux entre tous les disques. Cet outil est actuellement disponible pour Windows. Ces outil n’est pas pris en charge avec des disques managés. Cet outil valide également pendant la copie les données, de sorte que vous pouvez ignorer l’étape de validation lors de l’utilisation de cet outil.

L’ensemble d’outils est disponible pour Windows et Linux. Vous pouvez télécharger l’ensemble d’outils ici :
- [Télécharger l’ensemble d’outils Data Box Disk pour Windows](https://aka.ms/databoxdisktoolswin) 
- [Télécharger l’ensemble d’outils Data Box Disk pour Linux](https://aka.ms/databoxdisktoolslinux)
 
### <a name="q-can-i-use-data-box-disk-to-transfer-data-to-azure-files-and-then-use-the-data-with-azure-file-sync"></a>Q. Puis-je utiliser Data Box Disk pour transférer des données vers Azure Files, puis utiliser les données avec Azure File Sync ? 
R. Les fichiers Azure sont pris en charge avec Data Box Disk, mais ne fonctionnent pas correctement avec Azure File Sync. Les métadonnées ne sont pas conservées si les données des fichiers sont utilisées avec Azure File Sync.


## <a name="verify-and-upload"></a>Vérification et chargement

### <a name="q-how-soon-can-i-access-my-data-in-azure-once-ive-shipped-the-disks-back"></a>Q. Après combien de temps puis-je accéder à mes données dans Azure une fois que j’ai renvoyé les disques ? 
R.  Dès que l’état de la commande pour la copie des données s’affiche comme complet, vous devriez pouvoir accéder à vos données.

### <a name="q-where-is-my-data-located-in-azure-after-the-upload"></a>Q. Où se trouvent mes données dans Azure après leur téléchargement ?
R.  Lorsque vous copiez des données dans les dossiers *BlockBlob* et *PageBlob* sur votre disque, un conteneur est créé sur le compte de stockage Azure pour chaque sous-dossier situé dans les dossiers *BlockBlob* et *PageBlob*. Si vous avez copié les fichiers dans les dossiers *BlockBlob* et *PageBlob* directement, ils se trouvent dans un conteneur *$root* par défaut sur le compte de stockage Azure. Lorsque vous copiez les données dans un dossier sous le dosser *AzureFile*, un partage de fichiers est créé.

### <a name="q-i-just-noticed-that-i-did-not-follow-the-azure-naming-requirements-for-my-containers-will-my-data-fail-to-upload-to-azure"></a>Q. Je viens de remarquer que je n’ai pas suivi les conventions de dénomination pour mes conteneurs Azure. Le chargement de mes données sur Azure risque-t-il d’échouer ?
R. Les lettres majuscules dans les noms de vos conteneurs sont automatiquement converties en minuscules. Si les noms ne sont pas conformes d’une autre manière (p. ex., ils contiennent des caractères spéciaux ou d’autres langues), le chargement échouera. Pour plus d’informations, accédez à [Conventions de nommage des objets blob de blocs et des objets blob de pages Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions).

### <a name="q-how-do-i-verify-the-data-i-copied-onto-multiple-data-box-disks"></a>Q. Comment faire pour vérifier les données que j’ai copiées sur plusieurs disques Data Box ?
R.  Une fois la copie des données terminée, vous pouvez exécuter `DataBoxDiskValidation.cmd`, disponible dans le dossier *DataBoxDiskImport* pour générer des sommes de contrôle pour la validation. Si vous avez plusieurs disques, vous devez ouvrir une fenêtre de commande par disque avant d’exécuter cette commande. N’oubliez pas que cette opération peut être longue (plusieurs heures) en fonction de la taille de vos données.

### <a name="q-what-happens-to-my-data-after-i-have-returned-the-disks"></a>Q. Que deviennent mes données une fois que j’ai renvoyé les disques ?
R.  Une fois la copie des données vers Azure terminée, les données des disques sont effacées en toute sécurité, conformément aux instructions du standard NIST SP 800-88 Revision 1.  

### <a name="q-how-is-my-data-protected-during-transit"></a>Q. Comment mes données sont-elles protégées pendant le transfert ? 
R.  Les appareils Data Box Disk sont chiffrés à l’aide du chiffrement AES-128 de Microsoft BitLocker, et une clé d’accès unique est nécessaire pour déverrouiller tous les disques et accéder aux données.

### <a name="q-do-i-need-to-rerun-checksum-validation-if-i-add-more-data-to-the-data-box-disks"></a>Q. Dois-je réexécuter la validation des sommes de contrôle si j’ajoute d’autres données aux disques Data Box ?
R. Oui. Si vous décidez de valider vos données (et nous vous recommandons de le faire), vous devez réexécuter la validation si vous avez ajouté de nouvelles données sur les disques.

### <a name="q-i-used-all-my-disks-to-transfer-data-and-need-to-order-more-disks-is-there-a-way-to-quickly-place-the-order"></a>Q. J’ai utilisé tous mes disques pour transférer des données et j’ai besoin de commander davantage de disques. Existe-t-il un moyen de passer rapidement une commande ?
R. Vous pouvez cloner votre commande précédente. Ainsi, vous créez une commande identique à la précédente et vous pouvez en modifier les détails sans avoir à entrer votre adresse, vos informations de contact et vos préférences de notification.

### <a name="q-i-copied-data-to-the-manageddisk-folder-i-dont-see-any-managed-disks-with-the-resource-group-specified-for-managed-disks-was-my-data-uploaded-to-azure-how-can-i-locate-it"></a>Q. J’ai copié les données dans le dossier ManagedDisk. Je ne vois pas de disque managé avec le groupe de ressources spécifié pour les disques managés. Mes données ont-elles été chargées sur Azure ? Comment puis-je les trouver ?
R. Oui. Vos données ont été chargées sur Azure, mais, si vous ne voyez aucun disque managé avec les groupes de ressources spécifiés, il est probable que les données n’étaient pas valides. Si les objets blob de pages, les objets blob de blocs, les fichiers Azure ou les disques managés ne sont pas valides, ils seront placés dans les dossiers suivants :
 - Les objets blob de pages seront placés dans un conteneur d’objets blob de blocs dont le nom commence par *databoxdisk-invalid-pb-* .
 - Les fichiers Azure seront placés dans un conteneur d’objets blob de blocs dont le nom commence par *databoxdisk-invalid-af-* .
 - Les disques managés seront placés dans un conteneur d’objets blob de blocs dont le nom commence par *databoxdisk-invalid-md-* .

## <a name="next-steps"></a>Étapes suivantes

- Relire les [Conditions requises pour le système Data Box Disk](data-box-disk-system-requirements.md).
- Prenez connaissance des [limites de Data Box Disk](data-box-disk-limits.md).
- Déployez rapidement [Azure Data Box Disk](data-box-disk-quickstart-portal.md) sur le portail Azure.