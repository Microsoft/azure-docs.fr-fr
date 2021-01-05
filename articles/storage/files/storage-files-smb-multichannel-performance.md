---
title: Performances de SMB Multichannel - Azure Files
description: En savoir plus sur les performances de SMB Multichannel.
author: gunjanj
ms.service: storage
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: gunjanj
ms.subservice: files
ms.openlocfilehash: ee3d1335de1b2bb3096e88c4d04cd03daaa665f5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "96014098"
---
# <a name="smb-multichannel-performance"></a>Performances de SMB Multichannel

Azure Files SMB Multichannel (version préliminaire) permet à un client SMB 3.x d’établir plusieurs connexions réseau avec les partages de fichiers Premium dans un compte FileStorage. Le protocole SMB 3.0 a introduit la fonctionnalité SMB Multichannel dans les clients Windows Server 2012 et Windows 8. Pour cette raison, tout client Azure Files SMB 3.x qui prend en charge SMB Multichannel peut tirer parti de la fonctionnalité pour ses partages de fichiers Azure Premium. L’activation de SMB Multichannel sur un compte de stockage n’entraîne aucun coût supplémentaire.

## <a name="benefits"></a>Avantages

Azure Files SMB Multichannel permet aux clients d’utiliser plusieurs connexions réseau qui améliorent les performances tout en réduisant le coût de possession. L’agrégation de bande passante sur plusieurs cartes réseau et l’utilisation de la prise en charge de la mise à l’échelle côté réception pour distribuer la charge d’e/s sur plusieurs UC présentent des performances améliorées.

- **Débit accru** : Plusieurs connexions autorisent le transfert des données sur plusieurs chemins d’accès en parallèle et, par conséquent, permettent de tirer des avantages considérables des charges de travail qui utilisent des fichiers de plus grande taille avec des tailles d’e/s plus élevées et qui nécessitent un débit élevé à partir d’une seule machine virtuelle ou d’un plus petit ensemble de machines virtuelles. Certaines de ces charges de travail incluent des supports multimédia et de divertissement pour la création ou le transcodage de contenu, la génomique et l’analyse des risques de services financiers.
- **IOPS le plus élevé** : La fonctionnalité RSS de la carte réseau permet une distribution efficace de la charge sur plusieurs UC avec plusieurs connexions. Cela permet d’obtenir une mise à l’échelle des IOPS et une utilisation efficace des UC de machines virtuelles. Cela est utile pour les charges de travail qui ont de petites tailles d’e/s, telles que les applications de base de données.
- **Tolérance de panne réseau** :   Les connexions multiples atténuent le risque d’interruption puisque les clients ne s’appuient plus sur une connexion individuelle.
- **Configuration automatique** :   Lorsque SMB Multichannel est activé sur les clients et les comptes de stockage, il permet la découverte dynamique de connexions existantes et peut créer des chemins de connexion supplémentaires si nécessaire.
- **Optimisation des coûts** : Les charges de travail peuvent obtenir une mise à l’échelle supérieure à partir d’une machine virtuelle unique ou d’un petit ensemble de machines virtuelles, tout en se connectant à des partages Premium. Cela peut réduire le coût total de possession en réduisant le nombre de machines virtuelles nécessaires à l’exécution et à la gestion d’une charge de travail.

Pour en savoir plus sur SMB Multichannel, reportez-vous à la [documentation de Windows](/azure-stack/hci/manage/manage-smb-multichannel).

Cette fonctionnalité offre de meilleures performances pour les applications multithread, mais elle n’aide généralement pas les applications monothread. Consultez la section [Comparaison des performances](#performance-comparison) pour plus de détails.

## <a name="limitations"></a>Limites

[!INCLUDE [storage-files-smb-multi-channel-restrictions](../../../includes/storage-files-smb-multi-channel-restrictions.md)]

### <a name="regional-availability"></a>Disponibilité régionale

[!INCLUDE [storage-files-smb-multi-channel-regions](../../../includes/storage-files-smb-multi-channel-regions.md)]

## <a name="configuration"></a>Configuration

SMB Multichannel fonctionne uniquement lorsque la fonctionnalité est activée côté client (votre client) et côté service (votre compte de stockage Azure).

Sur les clients Azure, SMB Multichannel est activé par défaut. Vous pouvez vérifier votre configuration en exécutant la commande PowerShell suivante : 

`Get-smbClientConfiguration | select EnableMultichannel`.
 
Sur votre compte de stockage Azure, vous devez activer SMB Multichannel. Consultez [Activer SMB Multichannel (aperçu)](storage-files-enable-smb-multichannel.md).

### <a name="disable-smb-multichannel"></a>Désactiver SMB Multichannel
Dans la plupart des scénarios, en particulier les charges de travail multithread, les clients doivent obtenir des performances améliorées avec SMB Multichannel. Toutefois, pour certains scénarios spécifiques tels que les charges de travail monothread ou à des fins de test, vous pouvez désactiver SMB Multichannel. Consultez la section [Comparaison des performances](#performance-comparison) pour plus de détails.

## <a name="verify-smb-multichannel-is-configured-correctly"></a>Vérifier que SMB Multichannel est configuré correctement

1. Créez un partage de fichiers Premium ou utilisez-en un existant.
1. Vérifiez que votre client prend en charge SMB Multichannel (une ou plusieurs cartes réseau ont une mise à l’échelle côté réception activée). Pour plus d’informations, consultez la [documentation Windows](/azure-stack/hci/manage/manage-smb-multichannel).
1. Montez un partage de fichiers sur votre client.
1. Générez une charge avec votre application.
    Un outil de copie tel que robocopy/MT ou n’importe quel outil de performances tel que Diskspd pour lire/écrire des fichiers, peut générer une charge.
1. Ouvrez PowerShell en tant qu’administrateur et utilisez la commande suivante : `Get-SmbMultichannelConnection |fl`
1. Recherchez des propriétés de **MaxChannels** et **CurrentChannels**

:::image type="content" source="media/storage-files-smb-multichannel-performance/files-smb-multi-channel-connection.PNG" alt-text="Capture d’écran des résultats get-smbmultichannelconnection results." lightbox="media/storage-files-smb-multichannel-performance/files-smb-multi-channel-connection.PNG":::

## <a name="performance-comparison"></a>Comparaison entre les performances

Il existe deux catégories de modèles de charge de travail en lecture/écriture : monothread et multithread. La plupart des charges de travail utilisent plusieurs fichiers, mais il peut y avoir des cas d’usage spécifiques où la charge de travail fonctionne avec un fichier unique dans un partage. Cette section décrit les différents cas d’utilisation et l’impact sur les performances de chacun d’eux. En général, la plupart des charges de travail sont multithread et distribuent la charge de travail sur plusieurs fichiers afin de pouvoir observer des améliorations significatives des performances avec SMB Multichannel.

- **Multithread/plusieurs fichiers** : Selon le modèle de charge de travail, vous devriez constater une amélioration significative des performances des e/s de lecture et d’écriture sur plusieurs canaux. Les gains de performance varient d’un intervalle de 2 à 4 fois en termes d’IOPS, de débit et de latence. Pour cette catégorie, SMB Multichannel doit être activé pour obtenir des performances optimales.
- **Multithread/fichier unique** : Pour la plupart des cas d’usage dans cette catégorie, les charges de travail bénéficient de l’activation de SMB Multichannel, en particulier si la charge de travail a une taille d’e/s moyenne supérieure à 16 Ko. Quelques exemples de scénarios qui tirent parti de SMB Multichannel sont la sauvegarde ou la récupération d’un seul fichier volumineux. Une exception dans laquelle vous souhaiterez peut-être désactiver SMB Multichannel a lieu si votre charge de travail est lourde en e/s de petite taille. Dans ce cas, vous pouvez observer une légère perte de performances de 10 %. Selon le cas d’usage, envisagez la répartition de la charge entre plusieurs fichiers ou désactivez la fonctionnalité. Consultez la section [Configuration](#configuration) pour obtenir des informations.
- **Fichiers monothread/plusieurs fichiers ou fichier unique** : Pour la plupart des charges de travail monothread, il existe des avantages minimaux en matière de performances en raison d’un manque de parallélisme. Il y a généralement une légère dégradation des performances d’environ 10 % si SMB Multichannel est activé. Dans ce cas, il est idéal de désactiver SMB Multichannel, à une exception près. Si la charge de travail monothread peut répartir la charge entre plusieurs fichiers et utilise une taille d’e/s moyenne plus élevée (supérieure à 16 Ko), c’est qu’il doit y avoir de légers avantages en matière de performances par rapport à SMB Multichannel.

### <a name="performance-test-configuration"></a>Configuration des performances test

Pour les graphiques de cet article, la configuration suivante a été utilisée : une seule machine virtuelle standard D32s v3 avec une seule carte réseau compatible RSS avec quatre canaux. Le chargement a été généré à l’aide de diskspd.exe, de plusieurs threads avec une profondeur d’e/s de 10 et d’e/s aléatoires avec différentes tailles d’e/s.

| Taille | Processeurs virtuels | Mémoire : Gio | Stockage temporaire (SSD) en Gio | Disques de données max. | Débit de stockage temporaire et mis en cache max. : IOPS/Mbits/s (taille du cache en Gio) | Débit du disque non mis en cache max. : IOPS/Mbits/s | Nombre max de cartes réseau|Bande passante réseau attendue (Mbit/s) |
|---|---|---|---|---|---|---|---|---|
| [Standard_D32s_v3](../../virtual-machines/dv3-dsv3-series.md) | 32 | 128 | 256 | 32 | 64 000/512 (800)    | 51 200/768  | 8|16000 |

:::image type="content" source="media/storage-files-smb-multichannel-performance/files-smb-multi-channel-nic-settings-all-nics.PNG" alt-text="Capture d’écran des résultats get-smbmultichannelconnection results." lightbox="media/storage-files-smb-multichannel-performance/files-smb-multi-channel-nic-settings-all-nics.PNG":::

### <a name="mutli-threadedmultiple-files-with-smb-multichannel"></a>Multithreads/plusieurs fichiers avec SMB Multichannel

Le chargement a été généré pour 10 fichiers avec différentes tailles d’e/s. Les résultats des tests de mise à l’échelle ont montré des améliorations significatives dans les résultats des tests d’IOPS et de débit avec SMB Multichannel activé. Les diagrammes suivants décrivent les résultats :

:::image type="content" source="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-multiple-files-compared-to-single-channel-iops-performance.png" alt-text="Diagramme des performances" lightbox="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-multiple-files-compared-to-single-channel-iops-performance.png":::

:::image type="content" source="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-multiple-files-compared-to-single-channel-throughput-performance.png" alt-text="Diagramme des performances de débit." lightbox="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-multiple-files-compared-to-single-channel-throughput-performance.png":::

- Sur une seule carte réseau, pour les lectures, une augmentation des performances de 2 à 3 fois a été observée et, pour les écritures, des augmentations de 3 à 4 fois en termes d’IOPS et de débit.
- SMB Multichannel a permis aux IOPS et au débit d’atteindre les limites de la machine virtuelle avec une seule carte réseau et la limite de quatre canaux.
- Étant donné que la sortie (ou les lectures vers le stockage) n’est pas mesurée, le débit de lecture a pu dépasser la limite publiée par la machine virtuelle de 16 000 Mbits/s (2 Gio/s). Le test a atteint plus de 2,7 Gio/s. L’entrée (ou les écritures dans le stockage) est toujours soumise à des limites de machine virtuelle.
- La répartition de la charge sur plusieurs fichiers permet des améliorations substantielles.

Voici un exemple de commande qui a été utilisée dans ce test : 

`diskspd.exe -W300 -C5 -r -w100 -b4k -t8 -o8 -Sh -d60 -L -c2G -Z1G z:\write0.dat z:\write1.dat z:\write2.dat z:\write3.dat z:\write4.dat z:\write5.dat z:\write6.dat z:\write7.dat z:\write8.dat z:\write9.dat `.

### <a name="multi-threadedsingle-file-workloads-with-smb-multichannel"></a>Charges de travail multithread/fichiers uniques avec SMB Multichannel

La charge a été généré par rapport à un seul fichier de 128 Gio. Avec SMB Multichannel activé, le test de montée en puissance parallèle avec plusieurs threads/fichiers uniques affichait des améliorations dans la plupart des cas. Les diagrammes suivants décrivent les résultats :

:::image type="content" source="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-single-file-compared-to-single-channel-iops-performance.png" alt-text="Diagramme des performances d’IOPS." lightbox="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-single-file-compared-to-single-channel-iops-performance.png":::

:::image type="content" source="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-single-file-compared-to-single-channel-throughput-performance.png" alt-text="Diagramme des performances de débit d’un fichier unique." lightbox="media/storage-files-smb-multichannel-performance/diagram-smb-multi-channel-single-file-compared-to-single-channel-throughput-performance.png":::

- Sur une seule carte réseau avec une plus grande taille d’e/s moyenne (supérieure à 16 Ko), des améliorations significatives ont été apportées aux lectures et écritures.
- Pour les plus petites tailles d’e/s, il existait un léger impact d’environ 10 % sur les performances lors de l’activation de SMB Multichannel. Cela peut être atténué en répartissant la charge sur plusieurs fichiers ou en désactivant la fonctionnalité.
- Les performances sont toujours liées par  [des limites de fichiers uniques](storage-files-scale-targets.md#file-level-limits).

## <a name="optimizing-performance"></a>Optimisation des performances

Les conseils suivants peuvent vous aider à optimiser vos performances :

- Assurez-vous que votre compte de stockage et votre client sont colocalisés dans la même région Azure pour réduire la latence du réseau.
- Utilisez des applications multithread et répartissez la charge sur plusieurs fichiers.
- Les avantages en matière de performances de SMB Multichannel augmentent avec le nombre de fichiers distribuant la charge.
- Les performances du partage Premium sont liées à la taille du partage approvisionné (IOPS/sortie/entrée) et les limites de fichiers uniques. Pour plus d’informations, consultez [Comprendre le provisionnement des partages de fichiers Premium](storage-files-planning.md#understanding-provisioning-for-premium-file-shares).
- Les performances maximales d’un seul client de machine virtuelle sont toujours liées aux limites des machines virtuelles. Par exemple, [Standard_D32s_v3](../../virtual-machines/dv3-dsv3-series.md) peut prendre en charge une bande passante maximale de 16 000 Mbits/s (ou 2 Gbits/s), les sorties de la machine virtuelle (écritures vers le stockage) sont limitées, contrairement aux entrées (lectures depuis le stockage). Les performances des partages de fichiers sont soumises aux limites du réseau des machines, aux UC, à la bande passante réseau disponible du stockage interne, aux tailles d’e/s, au parallélisme, ainsi qu’à de nombreux autres facteurs.
- Le test initial est généralement un préchauffage, ignore ses résultats et répète le test.
- Si les performances sont limitées par un seul client et que la charge de travail est toujours inférieure aux limites de partage approvisionnées, les performances peuvent être améliorées en répartissant la charge sur plusieurs clients.

### <a name="the-relationship-between-iops-throughput-and-io-sizes"></a>Relation entre les IOPS, le débit et les tailles d’e/s

**Débit = Taille d’e/s * IOPS**

Les tailles d’e/s supérieures améliorent le débit et entraînent des latences plus élevées, ce qui réduit le nombre d’IOPS. Des tailles d’e/s plus petites augmentent les IOPS, mais entraînent une diminution du débit net et des latences.

## <a name="next-steps"></a>Étapes suivantes

- [Activer SMB Multichannel sur un compte FileStorage (préversion)](storage-files-enable-smb-multichannel.md)
- Pour en savoir plus sur SMB Multichannel, reportez-vous à la [documentation de Windows](/azure-stack/hci/manage/manage-smb-multichannel).
