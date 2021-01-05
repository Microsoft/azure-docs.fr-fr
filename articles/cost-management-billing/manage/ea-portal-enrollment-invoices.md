---
title: Factures d’inscription Azure Entreprise
description: Cet article explique comment gérer et traiter votre facture Azure Enterprise.
author: bandersmsft
ms.author: banders
ms.date: 12/09/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.custom: contperf-fy21q1
ms.openlocfilehash: c64127f7ebbb416d9dfebb5e417cd27d51697665
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030720"
---
# <a name="azure-enterprise-enrollment-invoices"></a>Factures d’inscription Azure Entreprise

Cet article explique comment gérer et traiter votre facture Azure Enterprise Agreement (Azure EA). Il s’agit d’une représentation de votre facture. Vérifiez l’exactitude des informations qu’elle contient. Familiarisez-vous aussi avec d’autres tâches éventuellement nécessaires pour gérer votre facture.

## <a name="view-usage-summary-and-download-reports"></a>Consulter le résumé de l’utilisation et télécharger les rapports

Les administrateurs d’entreprise peuvent voir le récapitulatif de leurs données d’utilisation, l’acompte Azure consommé et les frais associés à l’utilisation supplémentaire dans le portail Azure Enterprise. Les frais sont présentés au niveau du résumé pour tous les comptes et abonnements.

Pour voir l’utilisation détaillée de comptes spécifiques, téléchargez le rapport de détails de l’utilisation :

1. Connectez-vous au portail Azure Enterprise.
1. Sélectionnez **Rapports**.
1. Sélectionnez l’onglet **Télécharger l’utilisation**.
1. Dans la liste des rapports, sélectionnez **Télécharger** à côté du rapport mensuel qui vous intéresse.

   > [!NOTE]
   > Le rapport des détails de l’utilisation n’inclut pas les taxes applicables.
   >
   > Une latence pouvant atteindre huit heures peut être observée entre le moment où l’utilisation est engagée et celui où elle apparaît dans le rapport.

Pour voir les rapports et graphes du résumé de l’utilisation :

1. Connectez-vous au portail Azure Enterprise.
1. Sélectionnez un terme pour l’acompte.
   Pour modifier la plage de dates pour le **Résumé de l’utilisation**, vous pouvez basculer de **M** (mensuel) à **C** (personnalisé) en haut à droite de la page, puis entrer des dates de début et de fin personnalisées.  
   ![Créer et afficher un résumé de l’utilisation et télécharger les rapports dans un affichage personnalisé](./media/ea-portal-enrollment-invoices/create-ea-view-usage-summary-and-download-reports-custom-view.png)
1. Pour afficher des détails supplémentaires, vous pouvez sélectionner une période ou un mois sur le graphe.
   - Le graphe montre l’utilisation par mois, en détaillant l’engagement utilisé, le dépassement des frais de service, les frais facturés séparément et les frais de la place de marché Azure.
   - Pour le mois sélectionné, vous pouvez utiliser les champs situés sous le graphe pour filtrer par services, comptes et abonnements.
   - Vous pouvez basculer entre un affichage **Frais par services** et **Frais par hiérarchie**.
   - Affichez les détails du **service Azure**, des **frais facturés séparément** et de la **place de marché Azure** en développant les sections correspondantes.

Regardez cette vidéo pour voir comment afficher l’utilisation :

> [!VIDEO https://www.youtube.com/embed/Cv2IZ9QCn9E]

### <a name="download-csv-reports"></a>Télécharger les rapports CSV

Les administrateurs d’entreprise utilisent la page de téléchargement de rapport mensuel pour télécharger les rapports suivants dans des fichiers au format CSV :

- Solde et frais
- Détails de l’utilisation
- Frais de la place de marché Azure
- Price sheet

Pour télécharger des rapports :

1. Dans le portail Azure Enterprise, sélectionnez **Rapport**.
2. Sélectionnez **Télécharger l’utilisation** en haut de la page.
3. Sélectionnez **Download** (Télécharger) à côté du rapport mensuel qui vous intéresse.

   > [!NOTE]
   > Une latence pouvant atteindre 72 heures peut être observée entre la date effective de l’utilisation et celle où l’utilisation est indiquée dans les rapports.
   >
   > Les utilisateurs qui téléchargent les fichiers CSV dans Excel avec Safari peuvent rencontrer des erreurs de mise en forme. Pour éviter de telles erreurs, ouvrez ces fichiers à l’aide d’un éditeur de texte.

![Exemple de page Download Usage (Télécharger l’utilisation)](./media/ea-portal-enrollment-invoices/create-ea-download-csv-reports.png)

Regardez cette vidéo pour voir comment télécharger les données d’utilisation :

> [!VIDEO https://www.youtube.com/embed/eY797htT1qg]

### <a name="advanced-report-download"></a>Téléchargement des rapports avancés

Vous pouvez utiliser le téléchargement de rapports avancés pour obtenir des rapports sur des plages de dates ou des comptes spécifique. Le fichier de sortie est au format CSV pour prendre en charge les jeux d’enregistrements volumineux.

1. Dans le portail Azure Enterprise, sélectionnez **Téléchargement des rapports avancés**.
1. Sélectionnez une plage de dates appropriée et les comptes appropriés.
1. Sélectionnez **Request Usage Data** (Demander les données d’utilisation).
1. Sélectionnez le bouton **Actualiser** jusqu’à ce que l’état du rapport soit **Télécharger**.
1. Télécharger le rapport.

### <a name="download-usage-reports-and-billing-information-for-a-prior-enrollment"></a>Télécharger les rapports d’utilisation et les informations de facturation pour une inscription antérieure

Vous pouvez télécharger des rapports d’utilisation et des informations de facturation pour une inscription antérieure après un transfert d’inscription. Les rapports d’historique sont disponibles dans le portail Azure Enterprise et sous la gestion des coûts.

Le portail Azure Enterprise filtre les inscriptions inactives en dehors de l’affichage. Vous devez désactiver la case à cocher **Active** pour afficher les inscriptions transférées inactives.  

![Désactiver la case à cocher active permet à l’utilisateur de voir les inscriptions inactives](./media/ea-portal-enrollment-invoices/unchecked-active-box.png)

## <a name="change-a-po-number-for-an-upcoming-overage-invoice"></a>Modifier un numéro de bon de commande pour une facture de dépassement à venir

Le portail Azure Enterprise génère automatiquement un numéro de bon de commande par défaut, sauf si l’administrateur de l’entreprise en définit un avant la date de la facture. L’administrateur de l’entreprise peut également mettre à jour le numéro de bon de commande dans un délai de sept jours après la réception d’un e-mail de notification de facture automatisée.

### <a name="to-update-the-azure-services-purchase-order-number"></a>Pour mettre à jour le numéro de bon de commande des services Azure :

1. Sur le portail Azure Enterprise, sélectionnez **Rapport** > **Résumé de l'utilisation**.
1. Sélectionnez **Edit PO Numbers** (Modifier les numéros de bon de commande) dans le coin supérieur droit.
1. Sélectionnez la case d’option **Services Azure**.
1. Sélectionnez une **Période de facturation** dans le menu déroulant de plages de dates.

   Vous pouvez modifier un numéro de bon de commande pendant sept jours après avoir reçu une notification de facture, mais avant d'avoir payé la facture.
1. Entrez un nouveau numéro de bon de commande dans le champ **PO Number** (Numéro de bon de commande).
1. Cliquez sur **Save** (Enregistrer) pour envoyer votre modification.

### <a name="to-update-the-azure-marketplace-purchase-order-number"></a>Pour mettre à jour le numéro de bon de commande de Place de marché Azure :

1. Sur le portail Azure Enterprise, sélectionnez **Rapport** > **Résumé de l'utilisation**.
1. Sélectionnez **Edit PO Numbers** (Modifier les numéros de bon de commande) dans le coin supérieur droit.
1. Sélectionnez la case d’option **Place de marché**.
1. Sélectionnez une **Période de facturation** dans le menu déroulant de plages de dates.

   Vous pouvez modifier un numéro de bon de commande pendant sept jours après avoir reçu une notification de facture, mais avant d'avoir payé la facture.
1. Entrez un nouveau numéro de bon de commande dans le champ **PO Number** (Numéro de bon de commande).
1. Cliquez sur **Save** (Enregistrer) pour envoyer votre modification.

## <a name="azure-enterprise-billing-frequency"></a>Fréquence de facturation d’Azure Enterprise

Chaque année à la date d’effet de l’inscription, Microsoft facture tous les achats avec Paiement anticipé des services Microsoft Azure. Microsoft vous facturera pour les arriérés de Paiement anticipé relatifs à toute utilisation au-delà des montants de cet engagement.

- Les frais de Paiement anticipé sont établis selon un taux mensuel et facturés une fois par an à l’avance.
- Les frais de dépassement sont calculés chaque mois et facturés en arriérés à la fin de votre période de facturation.

### <a name="billing-intervals"></a>Intervalles de facturation

Votre intervalle de facturation dépend de la manière dont vous choisissez d’effectuer vos achats Paiement anticipé. Votre Paiement anticipé annuel coïncide avec l’un ou l’autre des termes suivants :

- La date anniversaire de votre inscription
- La date d'entrée en vigueur de votre abonnement d'un an aux avenants.

Vous recevrez votre facture de dépassement en fonction de la date de début et de la configuration de votre inscription :

- **Inscriptions directes avec une date de début antérieure au 1er mai 2018** :
  - Si vous avez conclu un Contrat Entreprise (EA) direct, vous serez facturé selon un cycle de facturation annuel pour les services Azure, à l’exception des services de la Place de marché Azure. Votre cycle de facturation est basé sur la date d'anniversaire, la date d’entrée en vigueur de votre contrat.
  - Si vous dépassez 150 % du seuil de votre Paiement anticipé Azure EA, vous basculerez automatiquement vers un cycle de facturation trimestriel basé sur votre date d’anniversaire. Vous recevrez également une facture de dépassement de service Azure.
  - Si vous ne dépassez pas 150 % du seuil de votre Paiement anticipé Azure, votre inscription restera sur un cycle de facturation annuel. Vous recevrez la facture de dépassement à la fin de l’année de Paiement anticipé.

- **Inscriptions directes avec une date de début postérieure au 1er mai 2018** :
  - Votre consommation Azure et les frais facturés séparément sont calculés sur un cycle de facturation mensuel.
  - Les frais qui ne sont pas couverts par votre Paiement anticipé Azure sont dus en tant que paiement de dépassement.  

- **Inscriptions indirectes avec une inscription qui a commencé avant le 1er mai 2018** :

  Si vous êtes un client ayant conclu un Contrat Entreprise (EA) dont la date de début est antérieure au 1er mai 2018, vous êtes soumis à un cycle de facturation trimestriel. Le partenaire de canal (CP) vous facture directement.  

- **Inscriptions indirectes avec une date de début postérieure au 1er mai 2018** :

  Vous êtes sur un cycle de facturation mensuel.  

### <a name="increase-your-azure-prepayment"></a>Augmenter votre Paiement anticipé Azure

Vous pouvez augmenter votre Paiement anticipé à tout moment. Vous serez facturé pour le nombre de mois restant dans la période de Paiement anticipé de l’année en cours. Par exemple, si vous signez un avenant d’abonnement d’un an et que vous augmentez votre Paiement anticipé au cours du sixième six mois, vous serez facturé pour les six mois restants de cette période. Vos quantités de Paiement anticipé seront ensuite mises à jour au cours des six derniers mois de votre période de Paiement anticipé. Les nouvelles quantités serviront à déterminer les frais de dépassement.

### <a name="overage"></a>Dépassement

Pour le dépassement, vous êtes facturé pour l’utilisation ou les réservations qui ont dépassé votre Paiement anticipé pendant la période de facturation. Pour afficher une répartition du calcul des quantités de dépassement pour des éléments individuels, consultez le rapport Résumé de l’utilisation ou contactez votre partenaire de canal.

Chaque ligne de la facture affiche les informations suivantes :

- **Montant total** : total des frais
- **Utilisation de Paiement anticipé** : montant de votre Paiement anticipé utilisé pour couvrir les frais
- **Montant net** : frais qui dépassent votre Paiement anticipé

Les taxes applicables sont calculées uniquement sur le montant net qui dépasse votre Paiement anticipé.

La facturation de dépassement est automatisée. Le calendrier des notifications et des factures dépend de la date de fin de votre période de facturation.

- La notification de dépassement est normalement envoyée sept jours après la date de fin de facturation.
- Les factures de dépassement sont envoyées sept à neuf jours après la notification.
- Vous pouvez consulter les frais et mettre à jour les numéros de bon de commande générés par le système pendant les sept jours qui s'écoulent entre la notification du dépassement et la facturation.

### <a name="azure-marketplace"></a>Place de marché Azure

À compter du cycle de facturation d’avril 2019, les clients ont commencé à recevoir une seule facture Azure, car nous regroupons tous les frais liés à Azure et à la Place de marché Azure sur une seule et même facture au lieu de deux factures distinctes. Cette modification n’affecte pas les clients en Australie, au Japon ou à Singapour.

Pendant la transition vers une facture combinée, vous recevrez une facture de la Place de marché Azure partielle. Cette facture distincte finale indiquera les frais liés à la Place de marché Azure avant la date de la mise à jour de facturation. Les frais liés à la Place de marché Azure facturés après cette date apparaissent sur votre facture Azure. Après la période de transition, vous verrez tous les frais liés à Azure et à la Place de marché Azure sur la facture combinée.  

### <a name="adjust-billing-frequency"></a>Ajuster la fréquence de la facturation

La fréquence de la facturation d’un client est annuelle, trimestrielle ou mensuelle. Le cycle de facturation est déterminé au moment où le client signe son contrat. La facturation mensuelle constitue l’intervalle de facturation le plus petit.

- Une **approbation** de l’administrateur d’entreprise est nécessaire pour passer d’un cycle de facturation annuel à un cycle trimestriel pour les inscriptions directes. Une approbation donnée par l’administrateur d’un partenaire est nécessaire pour les inscriptions indirectes. La modification prend effet à la fin du trimestre de facturation en cours.
- Un **avenant** au contrat est nécessaire pour passer d’un cycle de facturation annuel ou trimestriel à un cycle mensuel.  Toute modification apportée au cycle de facturation de l’inscription existante nécessite l’approbation d’un administrateur d’entreprise ou du « Contact facturation ».
- **Envoyez** votre approbation au [support du portail Azure Enterprise](https://support.microsoft.com/supportrequestform/cf791efa-485b-95a3-6fad-3daf9cd4027c). Sélectionnez la catégorie de problème : **Facturation**.

La modification prend effet à la fin du trimestre de facturation en cours.

Si vous signez un avenant M503, vous pouvez déplacer tout contrat, quelle qu’en soit la fréquence, vers une facturation mensuelle. 

### <a name="request-an-invoice-copy"></a>Demander une copie de facture

Pour demander une copie de votre facture, contactez votre partenaire.

## <a name="credits-and-adjustments"></a>Crédits et ajustements

Vous pouvez consulter tous les crédits ou ajustements appliqués à votre inscription dans la section **Rapports** du portail [ Azure Enterprise](https://ea.azure.com).

Pour afficher les crédits :

1. Dans le [portail Azure Enterprise](https://ea.azure.com), sélectionnez la section **Rapports**.
1. Sélectionnez **Résumé de l’utilisation**.
1. Dans le coin supérieur droit, modifiez l’affichage **M** en **C**.
1. Étendez le champ d’ajustement dans la table de Paiement anticipé de service Azure.
1. Vous verrez les crédits appliqués à votre inscription ainsi qu’une courte explication. Par exemple : Crédit de contrat de niveau de service.

## <a name="pay-your-overage-with-your-azure-prepayment"></a>Payer votre dépassement avec votre Paiement anticipé Azure

Pour pouvoir appliquer votre Paiement anticipé Azure aux dépassements, vous devez remplir les critères suivants :

- Vous devez avoir des frais de dépassement engagés et qui n’ont pas encore été payés, compris dans un délai d’un an à compter de la date de fin du service facturé.
- Le montant de votre Paiement anticipé Azure disponible doit couvrir le montant total des frais engagés, y compris toutes les factures Azure échues non payées.
- La durée de facturation que vous demandez doit être entièrement clôturée. La facturation est entièrement clôturée après le cinq du mois.
- La période de facturation que vous demandez pour le décalage doit être entièrement clôturée.
- Votre remise ACD (Paiement anticipé Azure) se base sur le nouveau Paiement anticipé réel moins les fonds prévus pour la consommation précédente. Cette exigence s’applique uniquement aux frais de dépassement engagés. Elle n’est valable que pour les services qui consomment Paiement anticipé Azure, et exclut donc les frais relatifs à la Place de marché Azure. Les frais relatifs à la Place de marché Azure sont facturés séparément.

Pour effectuer un décalage de dépassement, vous ou l’équipe en charge des comptes pouvez ouvrir une demande de support. Une approbation par e-mail de la part de l'administrateur de votre entreprise ou du contact facturation est nécessaire.

## <a name="move-charges-to-another-enrollment"></a>Déplacer les frais vers une autre inscription

Les données d’utilisation sont déplacées uniquement lorsqu’un transfert est antidaté. Il existe deux options pour déplacer des données d’utilisation d’une inscription à une autre :

- Transfert de comptes depuis une inscription vers une autre
- Transferts d’inscriptions depuis une inscription vers une autre

Quelle que soit l’option, vous devez envoyer une [demande de support](https://support.microsoft.com/supportrequestform/cf791efa-485b-95a3-6fad-3daf9cd4027c) à l’équipe du support technique EA pour obtenir de l’aide. 

## <a name="enterprise-agreement-usage-calculations"></a>Calculs de l’utilisation dans le cadre d’un Contrat Entreprise

Reportez-vous à [Services Azure](https://azure.microsoft.com/services/) et à [Tarification Azure](https://azure.microsoft.com/pricing/) pour obtenir des informations sur la tarification publique de base, les unités de mesure, la FAQ et des instructions de création de rapports d’utilisation pour chaque service individuel. Vous trouverez de plus amples informations sur les calculs EA dans les sections suivantes.

### <a name="enterprise-agreement-units-of-measure"></a>Unités de mesure du Contrat Entreprise

Les unités de mesure des Contrats Entreprise sont souvent différentes de celles présentées dans nos autres programmes tels que le programme de Contrat d’abonnement à Microsoft Online (MOSA). Cette disparité signifie que, pour un certain nombre de services, l’unité de mesure est agrégée pour fournir la tarification normalisée. L’unité de mesure affichée dans la vue Résumé de l’utilisation du portail Enterprise est toujours la mesure Enterprise. Une liste complète des unités de mesure et des conversions actuelles pour chaque service est fournie avec le fichier Excel des [noms de services conviviaux](https://azurepricing.blob.core.windows.net/supplemental/Friendly_Service_Names.xlsx).

### <a name="conversion-between-usage-detail-report-and-the-usage-summary-page"></a>Conversion entre le rapport de détails de l’utilisation et la page de résumé de l’utilisation

Dans le rapport de téléchargement des données d’utilisation, vous pouvez voir l'utilisation des ressources brutes jusqu'à six décimales. Toutefois, les données d’utilisation affichées dans le portail Azure Enterprise sont arrondies à quatre décimales pour les unités de Paiement anticipé et tronquées à zéro décimale pour les unités de dépassement. Les données d’utilisation brutes sont d’abord arrondies à quatre chiffres avant d’être converties en unités utilisées dans le portail Azure Enterprise. Les unités Enterprise converties sont ensuite à nouveau arrondies à quatre chiffres. Vous pouvez afficher les heures consommées réelles avant la conversion dans le rapport de téléchargement de l’utilisation et non dans le portail Azure Enterprise.

Par exemple : Supposons que 694,533404 heures SQL Server réelles sont indiquées dans le rapport de détails de l’utilisation. Ces unités sont converties en unités 6,94533404 de 100 heures de calcul, puis arrondies à 6,9453 pour être affichées dans le portail Azure Enterprise.

- Pour déterminer le montant de facturation total, les unités affichées sont multipliées par le prix unitaire de Paiement anticipé, et le résultat est tronqué à deux décimales. Pour le yen japonais (JPY) et le won coréen (KRW), le montant total est arrondi à zéro décimale.
- Pour calculer le dépassement, les unités de facturation sont tronquées à six chiffres, puis multipliées par le prix unitaire de dépassement pour déterminer le montant de facturation total.
- Pour la facturation du fournisseur de services managés (MSP), toute utilisation associée à un service marqué comme MSP est tronquée à zéro décimale après la conversion en unité de mesure EA. En conséquence, la somme de cette utilisation peut être inférieure à la somme totale de l’utilisation indiquée dans le portail Azure Enterprise. Cela dépend selon que le MSP se trouve toujours dans le solde de son Paiement anticipé ou s’il est déjà en dépassement.

### <a name="graduated-pricing"></a>Tarification progressive

La tarification Contrat Entreprise n’inclut pas actuellement la tarification progressive, dans laquelle les frais par unité diminuent à mesure que l’utilisation augmente. Lors du passage d’un programme MOSA avec une tarification progressive à un Contrat Entreprise, la tarification est définie en fonction du niveau de base de ce service, moins les ajustements des remises du Contrat Entreprise, le cas échéant.

### <a name="partial-hour-billing"></a>Facturation à l’heure partielle

Toute l’utilisation est facturée sur la base des minutes converties en heures partielles, et non sur la base d’incréments d’heures entières. Les taux horaires Enterprise indiqués sont appliqués au total des heures plus les heures partielles.

### <a name="average-daily-consumption"></a>Consommation quotidienne moyenne

Certains services sont facturés sur une base mensuelle, mais l'utilisation est rapportée sur une base quotidienne. Dans ces cas, cette utilisation est évaluée une fois par jour, divisée par 31 et totalisée sur le nombre de jours de ce mois de facturation. Les tarifs ne sont ainsi jamais plus élevés que prévu pour un mois et ils sont légèrement inférieurs pour les mois comptant moins de 31 jours.

### <a name="compute-hours-conversion"></a>Conversion des heures de calcul

Avant le 28 janvier 2016, toute utilisation de machines virtuelles A0, A2, A3 et A4 Standard et De base et de services cloud était calculée en minutes de machines virtuelles A1. Les machines virtuelles A0 comptaient comme des fractions de minutes de machine virtuelle A1, tandis que les machines virtuelles A2, A3 et A4 étaient converties en multiples. Cette stratégie entraînait une certaine confusion pour nos clients, et nous avons donc implémenté un changement afin d’assigner une utilisation par minute aux mesures dédiées des machines virtuelles A0, A2, A3 et A4.

La nouvelle mesure a pris effet entre le 27 janvier et le 28 janvier 2016. Sur le téléchargement CSV montrant l'utilisation pendant cette période de transition, vous pouvez voir les deux éléments :

- Utilisation générée sur le compteur A1
- Utilisation générée sur le nouveau compteur dédié correspondant à la taille de votre déploiement.

| **Taille du déploiement** | **Utilisation calculée en multiples de A1 jusqu’au 26 janvier 2016** | **Utilisation calculée sur une mesure dédiée à partir du 27 janvier 2016** |
| --- | --- | --- |
| A0 | 0,25 heure de A1 | 1 heure de A0 |
| A2 | 2 heures de A1 | 1 heure de A2 |
| A3 | 4 heures de A1 | 1 heure de A3 |
| A4 | 8 heures de A1 | 1 heure de A4 |

### <a name="regions"></a>Régions

Pour les services où la zone et la région affectent la tarification, consultez le tableau suivant présentant un mappage des zones géographiques et des régions :

| Zone géographique | Régions de transfert de données | Régions CDN |
| --- | --- | --- |
| Zone 1 | Europe septentrionale <br> Europe occidentale <br> USA Ouest <br> USA Est <br> USA Centre Nord <br> USA Centre Sud <br> USA Est 2 <br> USA Centre | Amérique du Nord <br> Europe |
| Zone 2 | Est de l’Asie-Pacifique <br> Sud-est de l’Asie-Pacifique <br> Japon Est <br> OuJapon Est <br> Australie Est <br> Sud-Australie Est | Asie-Pacifique <br> Japon <br> Amérique latine <br> Moyen-Orient / Afrique <br> Australie Est <br> Sud-Australie Est |
| Zone 3 | Brésil Sud |   |

Aucuns frais ne sont facturés pour la sortie de données entre des services hébergés dans le même centre de données. Par exemple, Microsoft 365 et Azure.

### <a name="azure-prepayment-and-unbilled-usage"></a>Paiement anticipé Azure et utilisation non facturée

Paiement anticipé Azure est un montant payé à l’avance pour les services Azure. Paiement anticipé Azure est consommé en tant que services utilisés. Les services Azure internes sont facturés sur Paiement anticipé Azure. Toutefois, certains frais sont facturés séparément, et les services de la Place de marché Azure ne consomment pas Paiement anticipé Azure.

### <a name="charges-billed-separately"></a>Frais facturés séparément

Certains produits et services fournis par des sources tierces ne consomment pas Paiement anticipé Azure. Au lieu de cela, ces éléments sont facturés séparément dans le cadre de la facture de dépassement du cycle de facturation standard.

Nous avons regroupé tous les frais liés à Azure et à la Place de marché Azure dans une facture unique qui s’aligne sur le cycle de facturation de l’inscription. La facture combinée ne s’applique pas aux clients en Australie, au Japon ou à Singapour.

La facture combinée indique d'abord l'utilisation Azure, puis les frais éventuels de la Place de marché Azure. Les clients en Australie, au Japon ou à Singapour verront les frais liés à la Place de marché Azure sur une facture distincte.

S’il n’y a pas d’utilisation de dépassement à la fin du cycle de facturation standard, les frais facturés séparément sont facturés séparément. Ce processus s'applique aux clients en Australie, au Japon et à Singapour.

Les services suivants sont facturés séparément :

- Canonical
- Citrix XenApp Essentials
- Utilisateur inscrit de Citrix XenDesktop
- Openlogic
- Utilisateur inscrit de Remote Access Rights XenApp Essentials
- Ubuntu Advantage
- Visual Studio Enterprise (mensuel)
- Visual Studio Enterprise (annuel)
- Visual Studio Professional (mensuel)
- Visual Studio Professional (annuel)

## <a name="what-to-expect-after-change-of-channel-partner"></a>À quoi s’attendre après un changement de partenaire de canal

Si le changement de partenaire de canal se produit au milieu du mois, un client reçoit une facture pour l’utilisation sous le partenaire associé précédent et une autre facture pour l’utilisation sous le nouveau partenaire.

Les factures sont publiées après le mois suivant la fin de la période de facturation. Si la cadence de facturation est mensuelle, la facture de septembre est publiée en octobre pour les deux partenaires. Si le cycle de facturation est trimestriel ou annuel, le client peut s’attendre à recevoir une facture pour le partenaire associé précédent concernant l’utilisation pendant leur période et le reste sera facturé au nouveau partenaire en fonction de la cadence de facturation.

## <a name="next-steps"></a>Étapes suivantes

- Les fichiers Excel suivants fournissent des détails sur les services Azure et sont mis à jour le 6 et le 20 de chaque mois :

   | Intitulé | Description | Nom de fichier |
   | --- | --- | --- |
   | [Noms de services conviviaux](https://azurepricing.blob.core.windows.net/supplemental/Friendly_Service_Names.xlsx) | Répertorie de tous les services actifs, et inclus les informations suivantes : <br>  <ul><li>catégorie de service</li>   <li>nom de service convivial</li>   <li>nom de Paiement anticipé et numéro de référence</li> <li>nom de la consommation et numéro de pièce</li>   <li>unité de mesure</li>   <li>facteurs de conversion entre l'utilisation déclarée et l'utilisation affichée sur le portail Enterprise</li></ul> | Friendly\_Service\_Names.xlsx |
   | [Champs de téléchargement de service](https://azurepricing.blob.core.windows.net/supplemental/Service_Download_Fields.xlsx) | Cette feuille de calcul fournit une liste de toutes les combinaisons possibles de champs liés au service dans le rapport de téléchargement de l’utilisation. | Service\_Download\_Fields.xlsx |

- Pour plus d’informations sur la compréhension de votre facture et de vos frais, consultez [Comprendre votre facture Contrat Entreprise Azure](../understand/review-enterprise-agreement-bill.md).
- Pour commencer à utiliser le portail Azure Enterprise, consultez [Bien démarrer avec le portail Azure EA](ea-portal-get-started.md).
