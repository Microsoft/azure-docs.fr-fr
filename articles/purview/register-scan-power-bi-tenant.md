---
title: Inscrire et analyser un locataire Power BI (préversion)
description: Découvrez comment inscrire et analyser un locataire Power BI à l’aide du portail Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/19/2020
ms.openlocfilehash: 1f08bfd6b7c0439dd08a3091b5ea927781af9b84
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387579"
---
# <a name="register-and-scan-a-power-bi-tenant-preview"></a>Inscrire et analyser un locataire Power BI (préversion)

Cet article explique comment inscrire et analyser un locataire Power BI à l’aide du portail Azure Purview.

> [!Note]
> Si l’instance Purview et le locataire Power BI se trouvent dans le même locataire Azure, vous pouvez uniquement utiliser l’authentification d’identité managée (MSI) pour configurer l’analyse d’un locataire Power BI. 

## <a name="create-a-security-group-for-permissions"></a>Créer un groupe de sécurité pour les autorisations

Pour configurer l’authentification, créez un groupe de sécurité et ajoutez-y l’identité managée du catalogue.

1. Dans le [portail Azure](https://portal.azure.com), recherchez **Azure Active Directory**.
1. Créez un groupe de sécurité dans votre annuaire Azure Active Directory, en suivant les instructions fournies dans [Créer un groupe de base et ajouter des membres avec Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal).

    > [!Tip]
    > Vous pouvez ignorer cette étape si vous disposez déjà d’un groupe de sécurité que vous souhaitez utiliser.

1. Sélectionnez **Sécurité** comme **Type de groupe**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/security-group.png" alt-text="Type de groupe de sécurité":::

1. Ajoutez l’identité managée de votre catalogue à ce groupe de sécurité. Sélectionnez **Membres**, puis **+ Ajouter des membres**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-group-member.png" alt-text="Ajoutez l’instance managée du catalogue au groupe.":::

1. Recherchez votre catalogue et sélectionnez-le.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-catalog-to-group-by-search.png" alt-text="Ajouter un catalogue en le recherchant":::

    Vous devez voir une notification de réussite indiquant qu’il a été ajouté.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/success-add-catalog-msi.png" alt-text="Réussite de l’ajout de l’identité MSI du catalogue":::

## <a name="associate-the-security-group-with-the-tenant"></a>Associer le groupe de sécurité au locataire

1. Connectez-vous au [portail d’administration Power BI](https://app.powerbi.com/admin-portal/tenantSettings).

    > [!Important]
    > Vous devez être Administrateur Power BI pour voir la page des paramètres du locataire.

1. Sélectionnez **Paramètres de développeur** > **Autoriser les principaux de service à utiliser les API d’administration Power BI en lecture seule (préversion)** .
1. Sélectionnez **Groupes de sécurité spécifiques**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/allow-service-principals-power-bi-admin.png" alt-text="Image illustrant comment permettre aux principaux de service d’obtenir des autorisations d’API d’administration Power BI en lecture seule":::

    > [!Caution]
    > Quand vous autorisez le groupe de sécurité que vous avez créé (dont l’identité managée de votre catalogue de données est membre) à utiliser des API d’administration Power BI en lecture seule, vous l’autorisez également à accéder aux métadonnées (telles que le tableau de bord et les noms de rapports, les propriétaires, les descriptions) pour tous vos artefacts Power BI dans ce locataire Une fois que les métadonnées ont été extraites dans Azure Purview, les autorisations de Purview, et non celles de Power BI, déterminent qui peut voir ces métadonnées.

    > [!Note]
    > Vous pouvez supprimer le groupe de sécurité de vos paramètres de développeur, mais les métadonnées précédemment extraites ne seront pas supprimées du compte Purview. Vous pouvez les supprimer séparément, si vous le souhaitez.

## <a name="register-your-power-bi-and-set-up-a-scan"></a>Inscrire votre locataire Power BI et configurer une analyse

Maintenant que vous avez accordé au catalogue les autorisations nécessaires pour se connecter à l’API d’administration de votre locataire Power BI, vous pouvez configurer votre analyse à partir du portail du catalogue.

Tout d’abord, ajoutez un indicateur de fonctionnalité spécial à votre URL Purview. 

1. Ajoutez la chaîne suivante à la fin de l’URI de votre instance Purview : `?feature.ext.catalog={"pbi":"true"}`. Cela active l’option d’inscription Power BI dans votre catalogue.

1. Sélectionnez l’icône **Centre de gestion**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/management-center.png" alt-text="Icône du centre de gestion.":::

1. Sélectionnez ensuite **+ Nouveau** dans **Sources de données**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/data-sources.png" alt-text="Image du bouton de nouvelle source de données":::

    Sélectionnez **Power BI** comme source de données.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/select-power-bi-data-source.png" alt-text="Image montrant la liste des sources de données pouvant être choisies":::

1. Donnez un nom convivial à votre instance de Power BI.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-friendly-name.png" alt-text="Image montrant le nom convivial de la source de données Power BI":::

    Le nom doit comporter entre 3 et 63 caractères, et contenir uniquement des lettres minuscules, des chiffres, des traits de soulignement et des traits d’union.  Les espaces ne sont pas autorisés.

    Par défaut, le système trouvera le locataire Power BI qui existe dans le même abonnement Azure.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-datasource-registered.png" alt-text="Source de données Power BI inscrite":::

1. Donnez un nom à votre analyse. Notez que la seule méthode d’authentification prise en charge est **Identité managée**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-setup.png" alt-text="Image montrant la configuration de l’analyse Power BI":::

    Le nom de l’analyse doit comporter entre 3 et 63 caractères, et contenir uniquement des lettres minuscules, des chiffres, des traits de soulignement et des traits d’union.  Les espaces ne sont pas autorisés.

1. Configurez un déclencheur d’analyse. Les options disponibles sont **Une fois**, **Tous les 7 jours** et **Tous les 30 jours**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/scan-trigger.png" alt-text="Image de déclencheur d’analyse":::

1. Dans **Vérifier la nouvelle analyse**, sélectionnez **Enregistrer et exécuter** pour lancer l’analyse.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/save-run-power-bi-scan.png" alt-text="Image de l’écran Enregistrer et exécuter dans Power BI":::

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment inscrire et analyser un locataire Power BI à l’aide d’applets de commande PowerShell, consultez :
  
- [Utiliser PowerShell pour inscrire et analyser Power BI](powershell-register-scan-power-bi.md)
