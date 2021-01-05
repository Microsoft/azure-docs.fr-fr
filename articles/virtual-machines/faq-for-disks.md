---
title: Forum aux questions sur les disques
description: Forum aux questions sur les disques de machines virtuelles et les disques Premium Linux (gérés et non gérés) Azure IaaS
author: roygara
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 06/15/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: dcfef81f2d7f3413489490d97c143fdec7e11bed
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499320"
---
# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-managed-and-unmanaged-premium-disks"></a>Forum aux questions sur les disques de machines virtuelles et les disques Premium gérés et non gérés Azure IaaS

Dans cet article, nous allons répondre à certaines questions fréquentes sur Azure Disques managés et les disques SSD Premium Azure.

## <a name="managed-disks"></a>Disques managés

**Qu’est-ce qu’Azure Disques managés ?**

La fonctionnalité Disques managés est une fonctionnalité qui simplifie la gestion des disques associés aux machines virtuelles Azure IaaS en prenant en charge pour vous la gestion des comptes de stockage. Pour plus d’informations, consultez la section [Vue d’ensemble de la fonctionnalité Disques managés](managed-disks-overview.md).

**Si je crée un disque géré standard à partir d’un disque dur virtuel existant de 80 Go, combien cela me coûte-t-il ?**

Un disque géré standard créé à partir d’un disque dur virtuel de 80 Go est considéré comme un disque standard de la taille directement supérieure. Dans notre cas, il s’agit du disque S10. Vous êtes facturé en fonction de la tarification de disque S10. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage).

**Des frais de transaction s’appliquent-ils aux disques managés Standard ?**

Oui. Vous êtes facturé pour chaque transaction. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage).

**Pour un disque standard géré, serai-je facturé pour la taille réelle des données sur le disque ou pour la capacité configurée du disque ?**

Vous êtes facturé en fonction de la capacité configurée du disque. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage).

**En quoi la tarification appliquée aux disques managés Premium est-elle différente de celle associée aux disques non managés ?**

La tarification de ces 2 types de disques est identique.

**Puis-je modifier le type de compte de stockage (Standard/Premium) de mes disques managés ?**

Oui. Vous pouvez modifier le type de compte de stockage de vos disques managés en utilisant le portail Azure, PowerShell ou l’interface Azure CLI.

**Puis-je utiliser un fichier VHD dans un compte de stockage Azure pour créer un disque géré avec un autre abonnement ?**

Oui.

**Puis-je utiliser un fichier VHD dans un compte de stockage Azure pour créer un disque géré dans une région différente ?**

Non.

**Existe-t-il des restrictions de mise à l’échelle pour les clients utilisant des disques managés ?**

La fonctionnalité Disques managés élimine les restrictions associées aux comptes de stockage. Toutefois, la limite maximale est de 50 000 disques managés par région et par type de disque pour un abonnement.

**Les machines virtuelles d’un groupe à haute disponibilité peuvent-elles consister en une combinaison de disques managés et non managés ?**

Non. Les machines virtuelles d’un groupe à haute disponibilité doivent utiliser exclusivement des disques managés ou non managés. Lorsque vous créez un groupe à haute disponibilité, vous pouvez définir le type de disques à utiliser.

**La fonctionnalité Disques managés est-elle l’option par défaut dans le portail Azure ?**

Oui.

**Est-il possible de créer un disque géré vide ?**

Oui. Vous pouvez tout à fait créer un disque vide. Un disque géré peut être créé indépendamment d’une machine virtuelle, par exemple, sans le joindre à une machine virtuelle.

**Par défaut, combien de domaines d’erreurs sont pris en charge pour les groupes à haute disponibilité utilisant la fonctionnalité Disques managés ?**

En fonction de la région où se trouve le groupe à haute disponibilité qui utilise la fonctionnalité Disques managés, le nombre de domaines d’erreurs pris en charge peut être de 2 ou 3.

**Comment est configuré le compte de stockage Standard pour les diagnostics ?**

Vous configurez un compte de stockage privé pour les diagnostics de machine virtuelle.

**Quel type de prise en charge du contrôle d’accès en fonction du rôle Azure est disponible pour la fonctionnalité Disques managés ?**

La fonctionnalité Disques managés prend en charge trois rôles principaux par défaut :

* Propriétaire : Gérer tout, y compris l’accès
* Collaborateur : Gérer tout sauf les accès
* Lecteur : Afficher tout, mais ne pas apporter de modifications

**Existe-t-il un moyen de copier ou d’exporter un disque géré vers un compte de stockage privé ?**

Vous pouvez générer un URI de signature d’accès partagé (SAP) en lecture seule pour le disque managé et l’utiliser pour copier le contenu vers un compte de stockage privé ou un espace de stockage local. Vous pouvez utiliser l’URI SAS avec le Portail Azure, Azure PowerShell, l’interface de ligne de commande Azure ou [AzCopy](../storage/common/storage-use-azcopy-v10.md)

**Puis-je créer une copie de mon disque géré ?**

Les clients peuvent prendre une capture instantanée de leurs disques managés, qu’ils utilisent pour créer un autre disque managé.

**Les disques non managés sont-ils encore pris en charge ?**

Oui, les disques managés et non managés sont pris en charge. Nous vous recommandons d’utiliser des disques managés pour les nouvelles charges de travail et de migrer vos charges de travail en cours vers des disques managés.

**Puis-je localiser conjointement des disques managés et non-managés sur la même machine virtuelle ?**

Non.

**Si je crée un disque de 128 Gio et que j’augmente la taille à 130 Gio, suis-je facturé en fonction de la taille de disque supérieure (256 Gio) ?**

Oui.

**Puis-je créer des disques managés disposant du stockage localement redondant, géoredondant ou redondant interzone ?**

Actuellement, Azure Disques managés prend uniquement en charge les disques managés disposant du stockage localement redondant.

**Puis-je réduire la taille de mes disques managés ?**

Non. Cette fonctionnalité n’est pas prise en charge pour l’instant.

**Puis-je résilier un bail sur mon disque ?**

Non. Cela n’est pas possible actuellement car un bail est présent pour empêcher une suppression accidentelle lorsque le disque est utilisé.

**Puis-je modifier la propriété de nom d’ordinateur lorsqu’un disque du système d’exploitation spécialisé (n’ayant pas été créé via l’outil de préparation système ou généralisé) est utilisé pour configurer une machine virtuelle ?**

Non. Vous ne pouvez pas mettre à jour la propriété de nom d’ordinateur. La nouvelle machine virtuelle a hérité cette particularité de la machine virtuelle parente qui a été utilisée pour créer le disque de système d’exploitation. 

**Où trouver des exemples de modèles Azure Resource Manager pour créer des machines virtuelles avec des disques managés ?**
* [Liste de modèles utilisant la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/)
* https://github.com/chagarw/MDPP

**Lorsque vous créez un disque à partir d’un objet blob, existe-t-il une relation continue avec cet objet blob source ?**

Non, quand le nouveau disque est créé, il constitue à ce moment-là une copie totalement indépendante de cet objet blob et aucune connexion n’existe entre les deux. Si vous voulez, une fois que vous avez créé le disque, vous pouvez supprimer l’objet blob source sans affecter le disque nouvellement créé en aucune façon.

**Puis-je renommer un disque managé ou non managé après l’avoir créé ?**

Vous ne pouvez pas renommer les disques managés. Par contre, vous pouvez renommer les disques non managés tant qu’ils ne sont pas attachés à une machine virtuelle ou un disque dur virtuel.

**Puis-je utiliser le partitionnement GPT sur un disque Azure ?**

Les images de génération 1 peuvent utiliser le partitionnement GPT sur les disques de données, mais pas sur les disques de système d’exploitation. Les disques de système d’exploitation doivent utiliser le style de partition MBR.

Les [images de génération 2](./generation-2.md) peuvent utiliser le partitionnement GPT aussi bien sur les disques de système d’exploitation que sur les disques de données.

**Quels types de disque prennent en charge les captures instantanées ?**

Les disques SSD Premium, SSD Standard et HDD Standard prennent en charge les captures instantanées. Pour ces trois types de disques, les captures instantanées sont prises en charge pour toutes les tailles (dont les disques allant jusqu’à 32 To). Les disques Ultra ne prennent pas en charge les captures instantanées.

**Qu’est-ce que des réservations de disque Azure ?**
La réservation de disque est la possibilité d’acheter à l’avance un an de stockage sur disque, ce qui réduit le coût total pour vous. Pour plus d’informations sur les réservations de disque Azure, consultez notre article sur le sujet : [Comprendre comment la remise de réservation est appliquée à Azure Data Box Disk](../cost-management-billing/reservations/understand-disk-reservations.md).

**Quelles sont les options offertes par la réservation de disque Azure ?**     
La réservation de disque Azure offre la possibilité d’acheter des SSD Premium dans les références SKU spécifiées de P30 (1 Tio) à P80 (32 Tio) pour une période d’un an. Il n’existe aucune limitation sur la quantité minimale de disques nécessaires pour acheter une réservation de disque. En outre, vous pouvez choisir de payer avec un paiement initial unique ou par mensualités. Aucun coût transactionnel supplémentaire n’est appliqué pour les disques managés SSD Premium.    

Les réservations sont effectuées sous forme de disques, et non de capacité. En d’autres termes, lorsque vous réservez un disque P80 (32 Tio), vous recevez un seul disque P80 ; vous ne pouvez pas ensuite diviser cette réservation spécifique en deux disques P70 plus petits (16 Tio). Vous pouvez, bien sûr, réserver autant de disques que vous le souhaitez, y compris deux disques P70 (16 Tio) distincts.

**Comment la réservation de disque Azure est-elle appliquée ?**     
La réservation de disque suit un modèle similaire aux instances de machines virtuelles réservées. La différence réside dans le fait qu’une réservation de disque ne peut pas être appliquée à différentes références SKU, contrairement aux instances de machines virtuelles. Pour plus d’informations sur les instances de machine virtuelle, consultez [Économiser grâce aux instances de machines virtuelles Azure réservées](./prepay-reserved-vm-instances.md).     

**Puis-je utiliser mon stockage de données acheté par le biais de la réservation de disques Azure dans plusieurs régions ?**     
La réservation de disque Azure est achetée pour une région et une référence SKU spécifiques (par exemple, P30 dans USA Est 2) et ne peut donc pas être utilisée en dehors de ces limites. Vous pouvez toujours acheter une réservation de disque Azure supplémentaire pour vos besoins de stockage sur disque dans d’autres régions ou références SKU.    

**Que se passe-t-il lorsque ma réservation de disques Azure expire ?**     
Vous recevez une notification par e-mail 30 jours avant l’expiration, puis à nouveau au moment de l’expiration. Une fois la réservation expirée, les disques déployés continuent à fonctionner et sont facturés en fonction au [tarif de paiement à l'utilisation](https://azure.microsoft.com/pricing/details/managed-disks/) le plus récent.

**Les disques SSD standard prennent-ils en charge les « SLA de machine virtuelle à instance unique » ?**

Oui, tous les types de disques prennent en charge les SLA de machine virtuelle à instance unique.

### <a name="azure-shared-disks"></a>Disques partagés Azure

**La fonctionnalité Disques partagés est-elle prise en charge pour les disques non managés ou les objets blob de pages ?**

Non, elle est prise en charge uniquement pour les disques Ultra et managés SSD Premium.

**Dans quelles régions les disques partagés sont pris en charge ?**

Pour obtenir des informations régionales, consultez notre [article conceptuel](disks-shared.md).

**Les disques partagés peuvent-ils être utilisés comme disque de système d’exploitation ?**

Non, les disques partagés sont pris en charge uniquement pour les disques de données.

**Quelles sont les tailles de disque qui prennent en charge les disques partagés ?**

Pour connaître les tailles prises en charge, consultez notre [article conceptuel](disks-shared.md).

**Est-il possible d’activer la fonctionnalité Disques partagés sur un disque existant ?**

La fonctionnalité Disques partagés peut être activée sur tous les disques managés créés avec l’API version 2019-07-01 ou supérieure. Pour cela, vous devez démonter le disque sur toutes les machines virtuelles auxquelles il est attaché. Ensuite, modifiez ensuite la propriété **maxShares** sur le disque.

**Si je ne souhaite plus utiliser un disque en mode partagé, que dois-je faire pour le désactiver ?**

Démontez le disque sur toutes les machines virtuelles auxquelles il est attaché. Modifiez ensuite la propriété maxShare sur le disque en lui attribuant la valeur 1.

**Est-il possible de redimensionner un disque partagé ?**

Oui.

**Est-ce que je peux activer l’accélérateur d’écriture pour un disque sur lequel la fonctionnalité Disques partagés est aussi activée ?**

Non.

**Est-ce que je peux activer la mise en cache de l’hôte pour un disque sur lequel la fonctionnalité Disques partagés est activée ?**

Seule l’option de mise en cache de l’hôte **« Aucune »** est prise en charge.

## <a name="ultra-disks"></a>Disques Ultra

**Quel débit dois-je définir pour mon disque Ultra ?**
Si vous ne savez pas comment configurer le débit de votre disque, nous vous recommandons de commencer par une taille d’E/S de 16 Kio et d’ajuster les performances à partir de cette valeur quand vous supervisez votre application. La formule est : Débit en Mbits/s = nombre d’IOPS * 16 / 1 000.

**J’ai configuré mon disque à 40 000 IOPS mais je ne vois que 12 800 IOPS. Pourquoi ne vois-je pas les performances du disque ?**
En plus de la limitation de disque, il existe une limitation d’IO (E/S) imposée au niveau de la machine virtuelle. Vérifiez que la taille de la machine virtuelle que vous utilisez peut prendre en charge les niveaux configurés pour vos disques. Pour plus d’informations sur les limites d’IO (E/S) imposées par votre machine virtuelle, consultez [Tailles des machines virtuelles dans Azure](sizes.md).

**Puis-je utiliser les niveaux de mise en cache avec un disque Ultra ?**
Non, les disques Ultra ne prennent pas en charge les différentes méthodes de mise en cache prises en charge par d’autres types de disque. Affectez la valeur **Aucune** à la mise en cache de disque.

**Puis-je attacher un disque Ultra à ma machine virtuelle existante ?**
Éventuellement. Votre machine virtuelle doit se trouver dans une région et une zone de disponibilité prenant en charge les disques Ultra. Pour plus d’informations, consultez les détails permettant de [bien démarrer avec les disques Ultra](disks-enable-ultra-ssd.md).

**Puis-je utiliser un disque Ultra en tant que disque de système d’exploitation pour ma machine virtuelle ?**
Non, les disques Ultra sont uniquement pris en charge en tant que disques de données et sont uniquement pris en charge en tant que disques natifs 4K.

**Puis-je convertir un disque existant en disque Ultra ?**
Non, mais vous pouvez migrer les données d’un disque existant vers un disque Ultra. Pour migrer un disque existant vers un disque Ultra, attachez les deux disques à la même machine virtuelle, puis copiez les données d’un disque à l’autre, ou utilisez une solution tierce pour effectuer la migration des données.

**Puis-je créer des captures instantanées pour les disques Ultra ?**
Non, les captures instantanées ne sont pas encore disponibles.

**Le service Sauvegarde Azure est-il disponible pour les disques Ultra ?**
Non, la prise en charge de Sauvegarde Azure n’est pas encore disponible.

**Puis-je attacher un disque Ultra à une machine virtuelle s’exécutant dans un groupe à haute disponibilité ?**
Non, cela n’est pas encore pris en charge.

**Puis-je activer Azure Site Recovery pour les machines virtuelles qui utilisent des disques Ultra ?**
Non, Azure Site Recovery n'est pas encore pris en charge pour les disques Ultra.

## <a name="uploading-to-a-managed-disk"></a>Chargement sur un disque managé

**Puis-je charger des données sur un disque managé existant ?**

Non, le chargement ne peut être utilisé que lors de la création d'un disque vierge dont l'état est **ReadyToUpload**.

**Comment puis-je charger des données sur un disque managé ?**

Créez un disque managé en définissant la propriété [createOption](/rest/api/compute/disks/createorupdate#diskcreateoption) de [creationData](/rest/api/compute/disks/createorupdate#creationdata) sur « Upload » (Charger). Vous pourrez ensuite y charger des données.

**Puis-je joindre un disque en état de chargement à une machine virtuelle ?**

Non.

**Puis-je prendre une capture instantanée d'un disque managé en état de chargement ?**

Non.

## <a name="standard-ssd-disks"></a>Disques SSD Standard

**Qu’est-ce qu’un disque SSD Standard Azure (préversion)**
Les disques SSD Standard sont des disques standard à semi-conducteur, optimisés en tant que stockage économique pour les charges de travail nécessitant des performances soutenues à des niveaux d’IOPS plus faibles.

<a id="standard-ssds-azure-regions"></a>**Quelles sont les régions actuellement prises en charge pour les disques SSD Standard ?**
Toutes les régions Azure prennent actuellement en charge les disques SSD Standard.

**La Sauvegarde Azure est-elle disponible pendant l’utilisation de disques SSD Standard ?**
Oui, le service Sauvegarde Azure est à présent disponible.

**Quel est l’avantage d’utiliser des disques SSD Standard au lieu des disques HDD ?**
Les disques SSD standard offrent une meilleure latence, cohérence, disponibilité et fiabilité par rapport aux disques HDD. Pour cette raison, les charges de travail des applications s’exécutent beaucoup plus facilement sur les disques SSD Standard. Notez que les disques SSD Premium constituent la solution recommandée pour la plupart des charges de travail de production gourmandes en E/S.

**Puis-je utiliser des disques SSD Standard en tant que disques non managés ?**
Non, les disques SSD Standard sont disponibles seulement comme disques managés.

## <a name="migrate-to-managed-disks"></a>Migrer vers la fonctionnalité Disques managés

**La migration a-t-elle un impact sur les performances de la fonctionnalité Disques managés ?**

La migration implique le déplacement du disque d’un emplacement de stockage à un autre. Cette opération est effectuée via une copie en arrière-plan des données, qui peut prendre plusieurs heures (généralement moins de 24 heures), selon la quantité de données stockées sur les disques. Pendant ce temps, votre application peut afficher une latence de lecture supérieure à la normale car certaines lectures sont redirigées vers l’emplacement d’origine et prennent donc plus de temps. Il n’y a aucun impact sur la latence d’écriture pendant cette période.  

**Quelles sont les modifications nécessaires dans une configuration de service Sauvegarde Azure préexistante avant/après la migration vers la fonctionnalité Disques managés ?**

Aucune modification n’est nécessaire.

**Mes sauvegardes de machines virtuelles créées par le biais du service Sauvegarde Azure avant la migration continueront-elles à fonctionner ?**

Oui, les sauvegardes fonctionnent de manière transparente.

**Quelles sont les modifications nécessaires dans une configuration Azure Disk Encryption préexistante avant/après la migration vers la fonctionnalité Disques managés ?**

Aucune modification n’est nécessaire.

**La migration automatisée d’un groupe de machines virtuelles identiques existant à partir de disques non managés vers la fonctionnalité Disques managés est-elle prise en charge ?**

Non. Vous pouvez créer un groupe de machines virtuelles identiques à l’aide de l’image de votre ancien groupe de machines virtuelles identiques avec des disques non managés.

**Puis-je créer un disque managé à partir d’un instantané d’objet blob de pages antérieur à la migration vers Managed Disks ?**

Non. Vous pouvez exporter un instantané d’objet blob de pages en tant qu’objet blob de pages, puis créer un disque managé à partir de l’objet blob de pages exporté.

**Puis-je basculer mes machines locales protégées par Azure Site Recovery vers une machine virtuelle avec la fonctionnalité Disques managés ?**

Oui, vous pouvez choisir de basculer vers une machine virtuelle avec la fonctionnalité Disques managés.

**La migration impacte-t-elle les machines virtuelles Azure protégées par Azure Site Recovery par le biais de la réplication Azure vers Azure ?**

Non. La protection Azure Site Recovery vers Azure pour machines virtuelles avec la fonctionnalité Disques managés est disponible.

**Puis-je convertir des machines virtuelles avec des disques non managés résidant sur des comptes de stockage qui sont ou ont été chiffrés sur des disques managés ?**

Oui

## <a name="managed-disks-and-storage-service-encryption"></a>Disques managés et Storage Service Encryption

**Le chiffrement côté serveur est-il activé par défaut lors de la création d’un disque managé ?**

Oui. Les Disques managés sont chiffrés à l’aide d’un chiffrement côté serveur avec des clés gérées par la plateforme. 

**Le volume de démarrage est-il chiffré par défaut sur un disque managé ?**

Oui. Tous les disques managés sont chiffrés par défaut, y compris le disque du système d’exploitation.

**Qui gère les clés de chiffrement ?**

Les clés gérées par la plateforme sont managées par Microsoft. Vous pouvez également utiliser et gérer vos propres clés stockées dans Azure Key Vault. 

**Puis-je désactiver le chiffrement côté serveur pour mes disques managés ?**

Non.

**Le chiffrement côté serveur est-il uniquement disponible dans des régions spécifiques ?**

Non. Le chiffrement côté serveur avec des clés gérées par la plateforme et par le client est disponible dans toutes les régions où les Disques managés sont disponibles. 

**Azure Site Recovery prend-il en charge le chiffrement côté serveur avec une clé gérée par le client pour les scénarios de récupération d’urgence Local vers Azure et Azure vers Azure ?**

Oui. 

**Puis-je sauvegarder des Disques managés chiffrés à l’aide d’un chiffrement côté serveur avec une clé gérée par le client en utilisant le service Sauvegarde Azure ?**

Oui.

**Les instantanés et les images gérés sont-ils chiffrés ?**

Oui. Toutes les captures instantanées et images managées sont automatiquement chiffrées. 

**Puis-je convertir des machines virtuelles avec des disques non managés situés sur des comptes de stockage qui sont ou ont été chiffrés sur des disques managés ?**

Oui

**Un disque dur virtuel exporté à partir d’un disque géré ou un instantané seront-ils également chiffrés ?**

Non. Mais si vous exportez un disque dur virtuel vers un compte de stockage chiffré depuis un disque géré chiffré ou un instantané, il sera chiffré. 

## <a name="premium-disks-managed-and-unmanaged"></a>Disques Premium : gérés et non gérés

**Si une machine virtuelle utilise une taille qui prend en charge les disques SSD Premium, comme DSv2, puis-je attacher des disques de données Standard et Premium ?** 

Oui.

**Puis-je attacher des disques de données Standard et Premium à une taille qui ne prend pas en charge les disques SSD Premium, comme les séries D, Dv2, G ou F ?**

Non. Vous ne pouvez joindre que des disques de données Standard aux machines virtuelles n’utilisant pas une gamme de taille qui prend en charge les disques SSD Premium.

**Si je crée un disque de données Premium à partir d’un disque dur virtuel existant de 80 Go, combien cela me coûte-t-il ?**

Un disque de données Premium créé à partir d’un disque dur virtuel de 80 Go est considéré comme un disque standard de taille directement supérieure. Dans notre cas, il s’agit du disque P10. Vous êtes facturé en fonction de la tarification de disque P10.

**L’utilisation de disques SSD Premium occasionne-t-elle des coûts de transaction ?**

Il existe un coût fixe pour chaque taille de disque. Il s’ajoute aux limites spécifiques d’E/S par seconde et de débit. Les seuls autres coûts sont liés à la bande passante sortante et à la capacité de captures instantanées, le cas échéant. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/storage).

**Quelles sont les limites d’E/S par seconde et de débit associées à la mise en cache sur disque ?**

Les limites combinées pour le cache et le disque SSD local d’une machine de la série DS sont de 4 000 E/S par cœur et de 33 Mio par seconde par cœur. La série GS offre 5 000 E/S par seconde par cœur et 50 Mio par seconde par cœur.

**Le disque local SSD est-il pris en charge pour les machines virtuelles de la fonctionnalité Disques managés ?**

Le disque local SSD est un stockage temporaire inclus avec une machine virtuelle de la fonctionnalité Disques managés. Ce stockage temporaire n’occasionne aucun frais supplémentaire. Nous vous déconseillons d’utiliser ce disque SSD local pour stocker les données de vos applications, car il n’est pas rendu persistant dans le stockage d’objets Blob Azure.

**L’utilisation de la fonction TRIM sur les disques Premium a-t-elle des répercussions ?**

Il n’existe aucun inconvénient à l’utilisation de TRIM sur des disques Azure Premium ou Standard.

## <a name="new-disk-sizes-managed-and-unmanaged"></a>Nouvelles tailles de disque : gérés et non gérés

**Quelles régions prennent en charge la fonctionnalité de rafale pour la taille de disque SSD Premium applicable ?**

La capacité de rafale est actuellement prise en charge dans toutes les régions du cloud public Azure, avec une prise en charge des clouds souverains très prochainement. 

**Dans quelles régions les tailles de disque géré 4/8/16 Gio (P1/P2/P3, E1/E2/E3) sont-elles prises en charge ?**

Ces nouvelles tailles de disque sont actuellement prises en charge dans toutes les régions du cloud public Azure, avec une prise en charge des clouds souverains très prochainement. 

**Les tailles de disque P1/P2/P3 sont-elles prises en charge pour les disques non managés ou les objets blob de pages ?**

Non, elles ne sont prises en charge que sur les disques managés SSD Premium. 

**Les tailles de disque E1/E2/E3 sont-elles prises en charge pour les disques non managés ou les objets blob de pages ?**

Non, quelle que soit leur taille, les disques managés SSD standard ne peuvent pas être utilisés avec des disques non managés ou des objets blob de pages.

**Quelle est la plus grande taille de disque managé prise en charge pour les disques de système d’exploitation et de données sur les machines virtuelles Gen1 ?**

Le type de partition pris en charge par Azure pour les disques de système d’exploitation Gen1 est l’enregistrement de démarrage principal (MBR). Bien que les disques de système d’exploitation Gen1 prennent uniquement en charge MBR, les disques de données prennent en charge GPT. Bien que vous puissiez allouer un disque de système d’exploitation allant jusqu’à 4 Tio, le type de partition MBR ne peut utiliser que 2 Tio de cet espace disque pour le système d’exploitation. Azure prend en charge jusqu’à 32 Tio pour les disques de données managés.

**Quelle est la plus grande taille de disque managé prise en charge pour les disques de système d’exploitation et de données sur les machines virtuelles Gen2 ?**

Le type de partition pris en charge par Azure pour les disques de système d’exploitation Gen2 est GPT (GUID Partition Table). Les machines virtuelles Gen2 prennent en charge un disque de système d’exploitation allant jusqu’à 4 Tio. Azure prend en charge jusqu’à 32 Tio pour les disques de données managés.


**Quelle est la plus grande taille de disque non managé prise en charge pour les disques de système d’exploitation et de données ?**

Le type de partition pris en charge par Azure pour un disque de système d’exploitation utilisant des disques non managés est l’enregistrement de démarrage principal (MBR).  Bien que vous puissiez allouer un disque de système d’exploitation allant jusqu’à 4 Tio, le type de partition MBR ne peut utiliser que 2 Tio de cet espace disque pour le système d’exploitation. Azure prend en charge jusqu’à 4 Tio pour les disques de données non managés.


**Quelle est la plus grande taille d’objet blob de page prise en charge ?**

La plus grande taille d’objet blob de pages prise en charge par Azure est de 8 Tio (8 191 Gio). La taille blob de page maximale en cas d’attachement à une machine virtuelle en tant que disques de données ou de système d’exploitation est de 4 Tio (4 095 Gio).

**Dois-je utiliser une nouvelle version des outils Azure pour créer, joindre, redimensionner et charger des disques de taille supérieure à 1 Tio ?**

Vous n’avez pas besoin de mettre à niveau votre version des outils Azure pour créer, joindre ou redimensionner des disques de taille supérieure à 1 Tio. Pour charger votre fichier de disque dur virtuel en local directement vers Azure en tant qu’objet blob de pages ou disque non managé, vous devez utiliser l’ensemble d’outils le plus récent indiqué ci-dessous. Nous prenons en charge les chargements de disque dur virtuel uniquement jusqu’à 8 To.

|Outils Azure      | Versions prises en charge                                |
|-----------------|---------------------------------------------------|
|Azure PowerShell | Numéro de version 4.1.0 : Version de juin 2017 ou ultérieure|
|Azure CLI v1     | Numéro de version 0.10.13 : Version de mai 2017 ou ultérieure|
|Azure CLI v2     | Numéro de version 2.0.12 : Version de juillet 2017 ou ultérieure|
|AzCopy              | Numéro de version 6.1.0 : Version de juin 2017 ou ultérieure|

**Les tailles de disque P4 et P6 sont-elles prises en charge pour les disques non managés ou les objets blob de pages ?**

Les tailles de disque P4 (32 Gio) et P6 (64 Gio) ne sont pas prises en charge en tant que niveaux de disque par défaut pour les disques non managés et les objets blob de pages. Vous devez explicitement [définir le niveau d’objets blob](/rest/api/storageservices/set-blob-tier) sur P4 et P6 pour que votre disque soit mappé à ces niveaux. Si vous déployez un objet blob de pages ou un disque non managé avec une taille de disque ou une longueur de contenu inférieure à 32 Gio ou entre 32 Gio et 64 Gio sans définir le niveau de l’objet blob, vous continuez à utiliser P10 avec 500 IOPS et 100 Mio/s et le niveau tarifaire mappé.

**Si mon disque managé Premium de moins de 64 Gio a été créé avant l’activation du petit disque (aux environs du 15 juin 2017), comment est-il facturé ?**

Les disques Premium de petite taille de moins de 64 Gio continuent à être facturés selon le niveau de tarification P10.

**Comment puis-je changer la couche de disque de petits disques Premium inférieurs à 64 Gio en la faisant passer de P10 à P4 ou P6 ?**

Vous pouvez prendre un instantané de vos disques de petite taille et ensuite créer un disque pour changer automatiquement le niveau de tarification vers P4 ou P6 selon la taille fournie.

**Est-il possible de redimensionner des disques managés existants à partir de tailles de moins de 4 Tio pour les nouvelles tailles de disque jusqu’à 32 Tio ?**

Oui.

**Quelles sont les plus grandes tailles de disque prise en charge par Sauvegarde Azure et Azure Site Recovery ?**

La plus grande taille de disque prise en charge par Sauvegarde Azure est 32 Tio (4 Tio pour les disques chiffrés). La plus grande taille de disque prise en charge par Azure Site Recovery est 8 Tio. La prise en charge de disques plus grands (jusqu’à 32 Tio) n’est pas encore disponible dans Azure Site Recovery.

**Quels sont les tailles de machines virtuelles recommandées pour les disques de grande taille (> 4 Tio) pour les disques SSD Standard et HDD Standard afin de bénéficier d’IOPS et d’une bande passante optimisées ?**

Pour atteindre un débit de disques SSD standard et de disques durs HDD standard de grande capacité (> 4 To) supérieur à 500 IOPS et 60 Mio/s, nous vous recommandons de déployer une des tailles de machine virtuelle suivantes pour optimiser vos performances : machines virtuelles des séries B, DSv2, Dsv3, ESv3, Fs, Fsv2, M, GS, NCv2, NCv3 ou Ls. L’attachement de disques plus grands à des machines virtuelles existantes ou des machines virtuelles qui n’utilisent pas les tailles recommandées peut entraîner une baisse de performances.

**Comment mettre à niveau mes disques (> 4 Tio) déployés lors de la préversion de disques plus grands afin d’obtenir une taux d’E/S et une bande passante supérieurs sur GA ?**

Vous pouvez arrêter et démarrer la machine virtuelle à laquelle le disque est attaché ou détacher et rattacher votre disque. Les objectifs en matière de performances de disques plus grands ont été augmentés pour les disques SSD Premium et Standard sur GA.

**Dans quelles régions les tailles de disques managés de 8 Tio, 16 Tio et 32 Tio sont-elles prises en charge ?**

Les disques 8 Tio, 16 Tio et 32 Tio sont prises en charge dans toutes les régions sous Azure global, Microsoft Azure Government et Azure China 21Vianet.

**Prenez-vous en charge l’activation de la mise en cache de l’hôte sur toutes les tailles de disque ?**

La mise en cache de l’hôte (**lecture seule** et **lecture/écriture**) est prise en charge sur les tailles de disque inférieures à 4 Tio. Cela signifie que tout disque approvisionné jusqu’à 4095 Gio peut tirer parti de la mise en cache de l’hôte. La mise en cache de l’hôte n’est pas prise en charge pour les tailles de disque supérieures ou égales à 4096 Gio. Par exemple, un disque P50 Premium approvisionné à 4095 Gio peut tirer parti de la mise en cache de l’hôte et un disque P50 approvisionné à 4096 Gio ne peut pas tirer parti de la mise en cache de l’hôte. Nous vous recommandons de tirer parti de la mise en cache pour les plus petites tailles de disque, où vous pouvez vous attendre à observer une amélioration des performances avec les données mises en cache dans la machine virtuelle.

## <a name="private-links-for-securely-exporting-and-importing-managed-disks"></a>Liaisons privées pour l’exportation et l’importation sécurisés des disques managés

**Quel est l’avantage de l’utilisation de liaisons privées pour l’exportation et l’importation de disques managés ?**

Vous pouvez désormais tirer parti des liaisons privées pour restreindre l’exportation et l’importation de disques managés à votre réseau virtuel Azure uniquement. 

**Comment puis-je vérifier qu’un disque peut être exporté ou importé uniquement par le biais de liaisons privées ?**

Vous devez définir la propriété `DiskAccessId` sur une instance d’un objet d’accès au disque et définir la propriété NetworkAccessPolicy sur `AllowPrivate`.

**Puis-je lier plusieurs réseaux virtuels au même objet d’accès au disque ?**

Non. Actuellement, vous pouvez lier un objet d’accès au disque à un seul réseau virtuel.

**Puis-je lier un réseau virtuel à un objet d’accès au disque dans un autre abonnement ?**

Non. Actuellement, vous pouvez lier un objet d’accès au disque à un seul réseau virtuel dans le même abonnement.

**Puis-je lier un réseau virtuel à un objet d’accès au disque dans un autre abonnement ?**

Non. Actuellement, vous pouvez lier un objet d’accès au disque à un seul réseau virtuel dans le même abonnement.

**Combien d’exportations ou d’importations utilisant le même objet d’accès au disque peuvent se produire en même temps ?**

5

**Puis-je utiliser l’URI SAS d’un disque ou d’une capture instantanée pour télécharger le disque dur virtuel sous-jacent d’une machine virtuelle dans le même sous-réseau que le sous-réseau du point de terminaison privé associé au disque ?**

Oui.

**Puis-je utiliser l’URI SAS d’un disque ou d’une capture instantanée pour télécharger le disque dur virtuel sous-jacent d’une machine virtuelle dans le même sous-réseau que le sous-réseau du point de terminaison privé non-associé au disque ?**

Non.

## <a name="what-if-my-question-isnt-answered-here"></a>Que dois-je faire si je n’ai pas trouvé de réponse à ma question ici ?

Si votre question n’est pas répertoriée ici, faites-le-nous savoir et nous vous aiderons à trouver une réponse. Vous pouvez poser une question à la fin de cet article dans les commentaires. Pour prendre contact avec l’équipe de Stockage Azure et d’autres membres de la Communauté concernant cet article, consultez la [page de Q&R de Microsoft sur Stockage Azure](/answers/products/azure?product=storage).

Pour soumettre une demande de fonctionnalité, transmettez vos questions et vos idées sur le [forum dédié aux commentaires sur le stockage Azure](https://feedback.azure.com/forums/217298-storage).