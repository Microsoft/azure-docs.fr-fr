---
title: Gérer une demande de support Azure
description: Décrit les procédures d’affichage des demandes de support, d’envoi de messages, de modification du niveau de gravité de la demande, de partage des informations de diagnostic avec le support Azure, de réouverture d’une demande de support clôturée et de chargement de fichiers.
tags: billing
ms.assetid: 86697fdf-3499-4cab-ab3f-10d40d3c1f70
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: a9dd703dc0a3f5e8f85b1022fa2a71ff9a8c295d
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745568"
---
# <a name="manage-an-azure-support-request"></a>Gérer une demande de support Azure

Après avoir [créé une demande de support Azure](how-to-create-azure-support-request.md), vous pouvez la gérer dans le [portail Azure](https://portal.azure.com), comme décrit dans cet article. Vous pouvez également créer et gérer des demandes par programme, à l’aide de l’[API REST de ticket de support Azure](/rest/api/support).

## <a name="view-support-requests"></a>Afficher les demandes de support

Pour afficher les détails et l’état des demandes de support, accédez à **Aide + support** >  **Toutes les demandes de support**.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="Toutes les demandes de support":::

Sur cette page, vous pouvez rechercher, filtrer et trier des demandes de support. Sélectionnez une demande de support pour afficher les détails, notamment la sévérité et les messages associés à la demande.

## <a name="send-a-message"></a>Envoyer un message

1. Sur la page **Toutes les demandes de support**, sélectionnez la demande de support.

1. Sur la page **Demande de support**, sélectionnez **Nouveau message**.

1. Entrez votre message et sélectionnez **Envoyer**.

## <a name="change-the-severity-level"></a>Modifier le niveau de gravité

> [!NOTE]
> Le niveau de gravité maximale dépend de votre [plan de support](https://azure.microsoft.com/support/plans).
>

1. Sur la page **Toutes les demandes de support**, sélectionnez la demande de support.

1. Sur la page **Demande de support**, sélectionnez **Modifier**.

    :::image type="content" source="media/how-to-manage-azure-support-request/change-severity.png" alt-text="Modifier la gravité de la demande de support":::

1. Le portail Azure affiche l’un ou l’autre écran, selon que votre demande est déjà attribuée à un ingénieur du support technique ou non :

    - Si votre demande n’a pas été attribuée, un écran similaire à celui ci-dessous s’affiche. Sélectionnez un nouveau niveau de gravité, puis sélectionnez **Modifier**.

        :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Sélectionner un nouveau niveau de gravité":::

    - Si votre demande a été attribuée, un écran similaire à celui ci-dessous s’affiche. Sélectionnez **OK**, puis créez un [message](#send-a-message) pour demander une modification du niveau de gravité.

        :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="Impossible de sélectionner un nouveau niveau de gravité":::

## <a name="share-diagnostic-information-with-azure-support"></a>Partager des informations de diagnostic avec le support Azure

Lors de la création d’une demande de support, par défaut, l’option **Partager les informations de diagnostic** est sélectionnée. Cela permet au support Azure de collecter des [informations de diagnostic](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) à partir de vos ressources Azure :

* Vous ne pouvez pas désactiver cette option après la création d’une demande.

* Si vous avez désactivé l’option lors de la création d’une demande, vous pouvez la sélectionner après la création de la demande.

    1. Sur la page **Toutes les demandes de support**, sélectionnez la demande de support.
    
    1. Sur la page **Demande de support**, sélectionnez **Accorder l’autorisation**, puis sélectionnez **Oui** et **OK**.
    
        :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Accorder des autorisations pour les informations de diagnostic":::

## <a name="upload-files"></a>Charger des fichiers

Vous pouvez utiliser l’option de chargement de fichiers de diagnostic ou tout autre fichier qui vous semble s’appliquer à la demande de support.

1. Sur la page **Toutes les demandes de support**, sélectionnez la demande de support.

1. Sur la page **Demande de support**, recherchez votre fichier, puis sélectionnez **Charger**. Répétez la procédure si vous avez plusieurs fichiers.

    :::image type="content" source="media/how-to-manage-azure-support-request/file-upload.png" alt-text="Charger un fichier":::

### <a name="file-upload-guidelines"></a>Instructions relatives au chargement de fichiers

Suivez ces instructions lorsque vous utilisez l’option de chargement de fichiers :

* Pour protéger votre confidentialité, évitez d’inclure des informations personnelles dans votre téléchargement.
* Le nom de fichier doit être inférieur à 110 caractères.
* Vous ne pouvez pas charger plusieurs fichiers.
* Les fichiers ne peuvent pas dépasser 4 Mo.
* Tous les fichiers doivent avoir une extension de nom de fichier, telle que  *.docx* ou  *.xlsx*. Le tableau suivant indique les extensions de nom de fichier dont le chargement est autorisé.

| 0-9, A-C    | D-G   | H-M         | N-P   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7Z         | .dat  | .hwl        | .odx  | .rar     | .tdb       | .xlam   |
| .a          | .db   | .ics        | .oft  | .rdl     | .tdf       | .xlr    |
| .abc        | .DMP  | .ini        | .old  | .rdlc    | .text      | .xls    |
| .adm        | .do_  | .java       | .one  | .re_     | .thmx      | .xlsb   |
| .aspx       | .doc  | .jpg        | .osd  | .reg     | .tif       | .xlsm   |
| .ATF        | .docm | .LDF        | .OUT  | .remove  | .trc       | .xlsx   |
| .b          | .docx | .letterhead | .p1   | .ren     | .TTD       | .xlt    |
| .ba_        | .dotm | .lnk        | .pcap | .rename  | .tx_       | .xltx   |
| .bak        | .dotx | .lo_        | .pdb  | .rft     | .txt       | .xml    |
| .bat        | .dtsx | .log        | .pdf  | .rpt     | .uccapilog | .xmla   |
| .blg        | .eds  | .lpk        | .piz  | .rte     | .uccplog   | .xps    |
| .CA_        | .emf  | .manifest   | .pmls | .rtf     | .udcx      | .xsd    |
| .CAB        | .eml  | .master     | .png  | .run     | .vb_       | .xsn    |
| .cap        | .emz  | .mdmp       | .potx | .saz     | .vbs_      | .xxx    |
| .catx       | .err  | .mof        | .ppt  | .sql     | .vcf       | .z_     |
| .CFG        | .etl  | .mp3        | .pptm | .sqlplan | .vsd       | .z01    |
| .compressed | .evt  | .mpg        | .pptx | .stp     | .wdb       | .z02    |
| .Config     | .evtx | .ms_        | .prn  | .svclog  | .wks       | .zi     |
| .cpk        | .EX   | .msg        | .psf  |   -       | .wma       | .zi_    |
| .cpp        | .ex_  | .msi        | .pst  |  -        | .wmv       | .zip    |
| .cs         | .ex0  | .mso        | .pub  | -         | .wmz       | .zip_   |
| .CSV        | .FRD  | .msu        | -      |-          | .wps       | .zipp   |
| .cvr        | .gif  | .nfo        | -      |-          | .wpt       | .zipped |
| -            | .guid | -            | -      | -         | .wsdl      | .zippy  |
| -            | .gz   | -            | -      | -         | .wsp       | .zipx   |
| -            | -      | -            | -      | -         | .wtl       | .zit    |
| -            | -      | -            | -      | -         |     -       | .zix    |
| -            | -      | -            | -      | -         |  -          | .zzz    |

## <a name="reopen-a-closed-request"></a>Rouvrir une demande clôturée

Si vous devez rouvrir une demande de support clôturée, créez un [message](#send-a-message), qui rouvre automatiquement la demande.

## <a name="next-steps"></a>Étapes suivantes

[Création d’une demande de support Azure](how-to-create-azure-support-request.md)

[API REST de ticket de support Azure](/rest/api/support)
