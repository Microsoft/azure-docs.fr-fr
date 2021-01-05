---
title: Tarifs Sauvegarde Azure
description: Découvrez comment estimer vos coûts de budgétisation de la tarification de la Sauvegarde Azure.
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: 03ec0076d3089562ddaace6db413fb3f1ba949a6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88654529"
---
# <a name="azure-backup-pricing"></a>Tarifs Sauvegarde Azure

Pour plus d’informations sur la tarification de la Sauvegarde Azure, consultez la [page de tarification de la Sauvegarde Azure](https://azure.microsoft.com/pricing/details/backup/).

## <a name="download-detailed-estimates-for-azure-backup-pricing"></a>Télécharger des estimations détaillées pour la tarification de la Sauvegarde Azure

Si vous souhaitez estimer vos coûts à des fins de budgétisation ou de comparaison des coûts, téléchargez l’[estimateur de la tarification de la Sauvegarde Azure](https://aka.ms/AzureBackupCostEstimates) détaillé.  

### <a name="what-does-the-estimator-contain"></a>Que contient l’estimateur ?

L’estimateur des coûts de la Sauvegarde Azure propose une option vous permettant d’estimer toutes les charges de travail possibles que vous souhaitez sauvegarder à l’aide de la Sauvegarde Azure. Voici quelques charges de travail :

- Machines virtuelles Azure
- Serveurs locaux
- SQL dans les machines virtuelles Azure
- SAP HANA dans les machines virtuelles Azure
- Partages de fichiers Azure

## <a name="estimate-costs-for-backing-up-azure-vms-or-on-premises-servers"></a>Estimer les coûts de sauvegarde des machines virtuelles Azure ou des serveurs locaux

Pour estimer les coûts liés à la sauvegarde des machines virtuelles Azure ou des serveurs locaux à l’aide de la Sauvegarde Azure, vous avez besoin des paramètres suivants :

- Taille des machines virtuelles ou des serveurs locaux que vous essayez de sauvegarder
  - Entrez la « taille utilisée » des disques ou serveurs requis pour la sauvegarde

- Nombre de serveurs avec cette taille

- Quelle est la quantité attendue d’attribution des données sur ces serveurs ?<br>
  L’attribution fait référence à la quantité de modifications des données. Par exemple, si vous avez une machine virtuelle avec 200 Go de données à sauvegarder et que 10 Go de données sont modifiées quotidiennement, l’attribution quotidienne est de 5 %.

  - Une plus haute attribution signifie que vous sauvegardez plus de données

  - Choisissez **Faible** ou **Modéré** pour les serveurs de fichiers et **Haut** si vous exécutez des bases de données

  - Si vous connaissez votre **attribution%** , vous pouvez utiliser l’option **Entrer votre propre%**

- Choisir la stratégie de sauvegarde

  - Combien de temps prévoyez-vous de conserver les sauvegardes « quotidiennes » ? (en jours)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « hebdomadaires » ? (en semaines)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « mensuelles » ? (en mois)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « annuelles » ? (en années)

  - Combien de temps prévoyez-vous de conserver les « captures instantanées de restauration instantanée » ? (de 1 à 5 jours)

    - Cette option vous permet de restaurer rapidement jusqu’à sept jours en arrière à l’aide de captures instantanées stockées sur des disques.

- **Facultatif** : sauvegarde sélective du disque

  - Si vous utilisez l’option **Sauvegarde sélective du disque** lors de la sauvegarde de machines virtuelles Azure, choisissez l’option **Exclure le disque** et entrez le pourcentage de disques exclus de la sauvegarde en termes de taille. Par exemple, si vous avez une machine virtuelle connectée à trois disques avec 200 Go utilisés sur chaque disque et si vous souhaitez exclure deux d’entre eux de la sauvegarde, entrez 66,7 %.

- **Facultatif** : redondance du stockage de sauvegarde

  - Cela indique la redondance du compte de stockage dans lequel les données de sauvegarde sont stockées. Nous recommandons l’utilisation du **GRS** pour une disponibilité maximale. Dans la mesure où elle garantit qu’une copie de vos données de sauvegarde est conservée dans une autre région, elle vous permet de respecter plusieurs normes de conformité. Modifiez la redondance en **LRS** si vous sauvegardez des environnements de développement ou de test qui n’ont pas besoin d’une sauvegarde au niveau de l’entreprise. Sélectionnez l’option **RAGRS** dans la feuille si vous souhaitez comprendre les coûts lorsque [Restauration inter-régions](backup-azure-arm-restore-vms.md#cross-region-restore) est activée pour vos sauvegardes.

- **Facultatif** : modifier la tarification régionale ou appliquer des taux de remise

  - Si vous souhaitez vérifier vos estimations pour une région différente ou des taux de remise, sélectionnez **Oui** pour l’option **Essayer des estimations pour une autre région ?** , puis entrez les taux avec lesquels vous souhaitez exécuter les estimations.

## <a name="estimate-costs-for-backing-up-sql-servers-in-azure-vms"></a>Estimer les coûts de sauvegarde des serveurs SQL dans les machines virtuelles Azure

Pour estimer les coûts liés à la sauvegarde des serveurs SQL s’exécutant dans des machines virtuelles Azure à l’aide de la Sauvegarde Azure, vous avez besoin des paramètres suivants :

- Taille des serveurs SQL que vous essayez de sauvegarder

- Nombre de serveurs SQL avec la taille ci-dessus

- Quelle est la compression attendue pour les données de sauvegarde de vos serveurs SQL ?

  - La plupart des clients de Sauvegarde Azure voient que les données de sauvegarde ont une compression de 80 % par rapport à la taille du serveur SQL lorsque la compression SQL est **activée**.

  - Si vous envisagez de voir une autre compression, entrez le nombre dans ce champ

- Quelle est la taille attendue des sauvegardes de journaux ?

  - Le % indique la taille quotidienne du journal en % de la taille du serveur SQL

- Quelle est la quantité attendue d’attribution des données quotidienne sur ces serveurs ?

  - En règle générale, les bases de données ont une attribution « Élevée »

  - Si vous connaissez votre **attribution%** , vous pouvez utiliser l’option **Entrer votre propre%**

- Choisir la stratégie de sauvegarde

  - Type de sauvegarde

    - La stratégie la plus efficace que vous pouvez choisir est les **Différentiels quotidiens** avec des sauvegardes complètes hebdomadaires/mensuelles/annuelles. Sauvegarde Azure peut également effectuer une restauration à partir de différentiels à l’aide d’un simple clic.

    - Vous pouvez également choisir d’avoir une stratégie avec des sauvegardes complètes quotidiennes/hebdomadaires/mensuelles/annuelles. Cette option consomme un peu plus de stockage que la première option.

  - Combien de temps prévoyez-vous de conserver les sauvegardes de « journal » ? (en jours) [de 7 à 35]

  - Combien de temps prévoyez-vous de conserver les sauvegardes « quotidiennes » ? (en jours)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « hebdomadaires » ? (en semaines)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « mensuelles » ? (en mois)

  - Combien de temps prévoyez-vous de conserver les sauvegardes « annuelles » ? (en années)

- **Facultatif** : redondance du stockage de sauvegarde

  - Cela indique la redondance du compte de stockage dans lequel les données de sauvegarde sont stockées. Nous recommandons l’utilisation du **GRS** pour une disponibilité maximale. Dans la mesure où elle garantit qu’une copie de vos données de sauvegarde est conservée dans une autre région, elle vous permet de respecter plusieurs normes de conformité. Modifiez la redondance en **LRS** si vous sauvegardez des environnements de développement ou de test qui n’ont pas besoin d’une sauvegarde au niveau de l’entreprise.

- **Facultatif** : modifier la tarification régionale ou appliquer des taux de remise

  - Si vous souhaitez vérifier vos estimations pour une région différente ou des taux de remise, sélectionnez **Oui** pour l’option **Essayer des estimations pour une autre région ?** , puis entrez les taux avec lesquels vous souhaitez exécuter les estimations.

## <a name="estimate-costs-for-backing-up-sap-hana-servers-in-azure-vms"></a>Estimer les coûts de sauvegarde des serveurs SAP HANA dans les machines virtuelles Azure

Pour estimer les coûts liés à la sauvegarde des serveurs SAP HANA s’exécutant dans des machines virtuelles Azure à l’aide de la Sauvegarde Azure, vous avez besoin des paramètres suivants :

- Taille totale des bases de données SAP HANA que vous essayez de sauvegarder. Il doit s’agir de la somme de la taille totale de la sauvegarde de chacune des bases de données, comme indiqué par SAP HANA.
- Nombre de serveurs SAP HANA avec la taille ci-dessus
- Quelle est la taille attendue des sauvegardes de journaux ?
  
  - Le % indique la taille moyenne quotidienne du journal en % de la taille totale des bases de données de SAP HANA que vous sauvegardez sur le serveur de SAP HANA
- Quelle est la quantité attendue d’attribution des données quotidienne sur ces serveurs ?
  - Le % indique la taille moyenne quotidienne d’attribution en % de la taille totale des bases de données de SAP HANA que vous sauvegardez sur le serveur de SAP HANA
  - En règle générale, les bases de données ont une attribution « Élevée »
  - Si vous connaissez votre **attribution%** , vous pouvez utiliser l’option **Entrer votre propre%**
- Choisir la stratégie de sauvegarde
  - Type de sauvegarde
    - La stratégie la plus efficace que vous pouvez choisir est les **Différentiels quotidiens** avec des sauvegardes complètes **hebdomadaires/mensuelles/annuelle** s. Sauvegarde Azure peut également effectuer une restauration à partir de différentiels à l’aide d’un simple clic.
    - Vous pouvez également choisir d’avoir une stratégie avec des sauvegardes complètes **quotidiennes/hebdomadaires/mensuelles/annuelles**. Cette option consomme un peu plus de stockage que la première option.
  - Combien de temps prévoyez-vous de conserver les sauvegardes de « journal » ? (en jours) [de 7 à 35]
  - Combien de temps prévoyez-vous de conserver les sauvegardes « quotidiennes » ? (en jours)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « hebdomadaires » ? (en semaines)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « mensuelles » ? (en mois)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « annuelles » ? (en années)
- **Facultatif** : redondance du stockage de sauvegarde
  
  - Cela indique la redondance du compte de stockage dans lequel les données de sauvegarde sont stockées. Nous recommandons l’utilisation du **GRS** pour une disponibilité maximale. Dans la mesure où elle garantit qu’une copie de vos données de sauvegarde est conservée dans une autre région, elle vous permet de respecter plusieurs normes de conformité. Modifiez la redondance en **LRS** si vous sauvegardez des environnements de développement ou de test qui n’ont pas besoin d’une sauvegarde au niveau de l’entreprise.
- **Facultatif** : modifier la tarification régionale ou appliquer des taux de remise
  
  - Si vous souhaitez vérifier vos estimations pour une région différente ou des taux de remise, sélectionnez **Oui** pour l’option **Essayer des estimations pour une autre région ?** , puis entrez les taux avec lesquels vous souhaitez exécuter les estimations.
  
## <a name="estimate-costs-for-backing-up-azure-file-shares"></a>Estimer les coûts de sauvegarde des partages de fichiers Azure

Pour estimer les coûts liés à la sauvegarde des partages de fichiers Azure à l’aide de la [solution de sauvegarde basée sur une capture instantanée](azure-file-share-backup-overview.md) offerte par Sauvegarde Azure, vous avez besoin des paramètres suivants :

- Taille (**en Go**) des partages de fichiers que vous souhaitez sauvegarder.

- Si vous souhaitez sauvegarder des partages de fichiers répartis sur plusieurs comptes de stockage, spécifiez le nombre de comptes de stockage hébergeant les partages de fichiers à la taille supérieure.

- Quantité attendue d’attribution des données sur les partages de fichiers que vous souhaitez sauvegarder. <br>L’attrition fait référence à la quantité de modifications des données et a un impact direct sur la taille de stockage des instantanés. Par exemple, si vous avez un partage de fichiers avec 200 Go de données à sauvegarder et que 10 Go de données sont modifiées quotidiennement, l’attribution quotidienne est de 5 %.
  - Une plus grande attrition signifie que la quantité de données modifiées dans le contenu du partage de fichiers chaque jour est élevée, et que l’instantané incrémentiel (en capturant uniquement les données modifiées) serait également plus important.
  - Sélectionnez Faible (1 %), Modérée (3 %) ou Élevée (5 %) en fonction des caractéristiques et de l’utilisation de votre partage de fichiers.
  - Si vous connaissez le **pourcentage d’attrition** exact de votre partage de fichiers, vous pouvez sélectionner l’option **Entrer votre propre pourcentage** dans la liste déroulante. Spécifiez les valeurs (en pourcentage) pour une attrition quotidienne, hebdomadaire, mensuelle et annuelle.

- Type de compte de stockage (Standard ou Premium) et paramètre de redondance de stockage du compte de stockage hébergeant le partage de fichiers sauvegardé. <br>Dans la solution de sauvegarde actuelle pour les partages de fichiers Azure, les instantanés sont stockés dans le même compte de stockage que le partage de fichiers sauvegardé. Le coût de stockage associé aux instantanés est donc facturé dans le cadre de votre facture Azure Files, en fonction de la tarification des instantanés pour le type de compte et du paramètre de redondance du compte de stockage hébergeant le partage de fichiers et les instantanés sauvegardés.

- Rétention pour différentes sauvegardes
  - Combien de temps prévoyez-vous de conserver les sauvegardes « quotidiennes » ? (en jours)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « hebdomadaires » ? (en semaines)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « mensuelles » ? (en mois)
  - Combien de temps prévoyez-vous de conserver les sauvegardes « annuelles » ? (en années)

  Consultez [la matrice de prise en charge de partage de fichiers Azure](azure-file-share-support-matrix.md#retention-limits) pour obtenir les valeurs de rétention maximales prises en charge dans chaque catégorie.

- **Facultatif** : Modifier la tarification régionale ou appliquer des tarifs réduits.
  - Les valeurs par défaut définies pour le coût de stockage des instantanés par Go et le coût de l’instance protégée dans l’estimateur s’appliquent à la région USA Est. Si vous souhaitez vérifier vos estimations pour une région différente ou des tarifs réduits, sélectionnez **Oui** pour l’option **Essayer des estimations pour une autre région ?** , puis entrez les tarifs avec lesquels vous souhaitez effectuer les estimations.

## <a name="next-steps"></a>Étapes suivantes

[Qu’est-ce que le service Sauvegarde Azure ?](backup-overview.md)
