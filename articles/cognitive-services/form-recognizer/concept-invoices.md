---
title: Factures - Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez les concepts liés à l’analyse des factures avec l’API Form Recognizer - utilisation et limites.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 11/18/2020
ms.author: pafarley
ms.openlocfilehash: 9a3a6bd6489baea90ed4143b42a09e7d697bbc50
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602442"
---
# <a name="form-recognizer-prebuilt-invoice-model"></a>Modèle de facture prédéfini Form Recognizer

Azure Form Recognizer peut analyser et extraire des informations dans les factures à l’aide de ses modèles de facture prédéfinis. L’API Invoice permet aux clients d’accepter des factures dans une multitude de formats et de retourner des données structurées pour automatiser le traitement des factures. Elle combine nos puissantes fonctionnalités de [reconnaissance optique de caractères (OCR)](../computer-vision/concept-recognizing-text.md) avec des modèles de Deep Learning qui comprennent les factures dans le but d’extraire des informations clés de ces factures. Elle extrait le texte, les tables et les informations comme le client, le fournisseur, le numéro de facture, la date d’échéance, le total, le montant dû, le montant des taxes, l’adresse d’expédition, l’adresse de facturation, etc. L’API Invoice prédéfinie est en disponibilité publique dans la préversion de Form Recognizer v2.1.

## <a name="what-does-the-invoice-service-do"></a>Comment le service Invoice fonctionne-t-il ?

L’API Invoice extrait les champs clés des factures et les retourne dans une réponse JSON structurée et organisée. Les factures peuvent être dans divers formats et de diverses qualités. Il peut s’agir notamment d’images capturées par téléphone, de documents numérisés et de fichiers PDF numériques. L’API Invoice va extraire la sortie structurée de toutes ces factures. 

![Exemple de facture Contoso](./media/invoice-example.jpg)

## <a name="try-it-out"></a>Faire un essai

Pour tester le service Invoice de Form Recognizer, accédez à l’outil d’exemple d’interface utilisateur en ligne :

> [!div class="nextstepaction"]
> [Essayer des modèles prédéfinis](https://fott-preview.azurewebsites.net/)

Vous aurez besoin d’un abonnement Azure ([créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services)), ainsi que d’un point de terminaison et d’une clé de [ressource Form Recognizer](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer) pour tester le service Invoice de Form Recognizer. 

![Exemple de facture analysée](./media/analyze-invoice.png)


### <a name="input-requirements"></a>Critères des entrées 

[!INCLUDE [input reqs](./includes/input-requirements-receipts.md)]

## <a name="the-analyze-invoice-operation"></a>Opération d’analyse d’une facture

L’opération d’[analyse d’une facture](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/5ed8c9843c2794cbb1a96291) prend une image ou un fichier PDF d’une facture en entrée, puis en extrait les valeurs intéressantes. L’appel retourne un champ d’en-tête de réponse appelé `Operation-Location`. La valeur `Operation-Location` est une URL qui contient l’ID de résultat à utiliser à l’étape suivante.

|En-tête de réponse| URL de résultat |
|:-----|:----|
|Operation-Location | `https://cognitiveservice/formrecognizer/v2.1-preview.2/prebuilt/invoice/analyzeResults/49a36324-fc4b-4387-aa06-090cfbf0064f` |

## <a name="the-get-analyze-invoice-result-operation"></a>Opération d’obtention du résultat de l’analyse de la facture

La deuxième étape consiste à appeler l’opération d’[obtention du résultat de l’analyse de la facture](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/5ed8c9acb78c40a2533aee83). Cette opération prend en entrée l’ID de résultat créé par l’opération d’analyse de la facture. Elle retourne une réponse JSON qui contient un champ **État** avec les possibles valeurs suivantes. Vous appelez cette opération de façon itérative jusqu’à ce qu’elle retourne avec la valeur **succeeded**. Utilisez un intervalle de 3 à 5 secondes pour éviter de dépasser le taux de demandes par seconde (RPS).

|Champ| Type | Valeurs possibles |
|:-----|:----:|:----|
|status | string | notStarted : L’opération d’analyse n’a pas commencé.<br /><br />running : L’opération d’analyse est en cours.<br /><br />failed : L’opération d’analyse a échoué.<br /><br />succeeded : L’opération d’analyse a réussi.|

Quand le champ **status** a la valeur de **succeeded**, la réponse JSON inclut les résultats de la compréhension de la facture, les tables extraites et les résultats de la reconnaissance de texte facultative, si nécessaire. Le résultat de la compréhension de la facture est organisé sous la forme d’un dictionnaire de valeurs de champ nommé, où chaque valeur contient le texte extrait, la valeur normalisée, le cadre englobant, la confiance et les éléments de mot correspondants. Le résultat de reconnaissance de texte est organisé sous la forme d’une hiérarchie de lignes et de mots, avec du texte, un cadre englobant et des informations de confiance.

### <a name="sample-json-output"></a>Exemple de sortir JSON

La réponse à l’opération d’obtention du résultat de l’analyse de la facture va correspondre à la représentation structurée de la facture avec toutes les informations extraites. Reportez-vous à cet [exemple de fichier de facture](./media/sample-invoice.jpg) et cet [exemple de sortie de facture](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/curl/form-recognizer/sample-invoice-output.json) structurée.

La sortie JSON comporte trois parties : 
* Le nœud `"readResults"` contient tout le texte reconnu et toutes les marques de sélection. Le texte est organisé par page, puis par ligne, puis par mots individuels. 
* Le nœud `"pageResults"` contient les tables et les cellules extraites avec leurs cadres englobants, la confiance et une référence aux lignes et aux mots dans « readResults ».
* Le nœud `"documentResults"` contient les valeurs spécifiques de la facture découvertes par le modèle. C’est ici que vous allez trouver tous les champs de la facture, comme le numéro de facture, l’adresse d’expédition, l’adresse de facturation, le client, le total, etc.

## <a name="example-output"></a>Exemple de sortie

Le service Invoice extrait le texte, les tables et 26 champs de facture. Voici les champs extraits d’une facture dans la réponse de la sortie JSON (la sortie ci-dessous utilise cet [exemple de facture](./media/sample-invoice.jpg)) :  

|Nom| Type | Description | Texte | Valeur (sortie standardisée) |
|:-----|:----|:----|:----| :----|
| CustomerName | string | Client destinataire de la facture | Microsoft Corp |  |
| CustomerId | string | ID de référence du client | CID-12345 |  |
| PurchaseOrder | string | Numéro de référence du bon de commande | PO-3333 | |  |
| InvoiceId | string | ID de cette facture spécifique (souvent appelé « numéro de facture ») | INV-100 | |  |
| InvoiceDate | Date | Date d’émission de la facture | 15/11/2019 | 
| DueDate | Date | Date d’échéance du paiement de cette facture | 15/12/2019 | 2019-12-15 | 2019-11-15 |
| VendorName | string | Fournisseur qui a créé cette facture | CONTOSO LTD. | |
| VendorAddress | string | Adresse postale du fournisseur | 123 456th St New York, NY, 10001 | |
| VendorAddressRecipient | string | Nom associé à l’adresse postale du fournisseur | Siège social de Contoso | |
| CustomerAddress | string | Adresse postale du client | 123 Other St, Redmond WA, 98052 | |
| CustomerAddressRecipient | string | Nom associé à l’adresse postale du client | Microsoft Corp | |
| BillingAddress | string | Adresse de facturation explicite du client | 123 Bill St, Redmond WA, 98052 | |
| BillingAddressRecipient | string | Nom associé à l’adresse de facturation | Services Microsoft | |
| ShippingAddress | string | Adresse d’expédition explicite du client | 123 Ship St, Redmond WA, 98052 | |
| ShippingAddressRecipient | string | Nom associé à l’adresse d’expédition | Livraison Microsoft | |
| SubTotal | nombre | Champ de sous-total identifié sur cette facture | 100,00 $ | 100 | 
| TotalTax | nombre | Champ du total des taxes identifié sur cette facture | 10,00 $ | 10 |
| InvoiceTotal | nombre | Total des nouveaux frais associés à cette facture | 110,00 $ | 110 |
| AmountDue |  nombre | Montant total dû au fournisseur | 610,00 $ | 610 |
| ServiceAddress | string | Adresse du service ou de la propriété explicite du client | 123 Service St, Redmond WA, 98052 | |
| ServiceAddressRecipient | string | Nom associé à l’adresse du service | Services Microsoft | |
| RemittanceAddress | string | Adresse d’envoi du paiement explicite pour le client | 123 Remit St New York, NY, 10001 |  |
| RemittanceAddressRecipient | string | Nom associé à l’adresse d’envoi du paiement explicite | Facturation Contoso |  |
| ServiceStartDate | Date | Date de début de la période de service (par exemple, pour une période de service de facturation d’utilitaire) | 14/10/2019 | 2019-10-14 |
| ServiceEndDate | Date | Date de fin de la période de service (par exemple, pour une période de service de facturation d’utilitaire) | 14/11/2019 | 2019-11-14 |
| PreviousUnpaidBalance | nombre | Solde précédent impayé explicite | 500,00 $ | 500 |


## <a name="next-steps"></a>Étapes suivantes

- Essayez vos propres factures et exemples dans l’[exemple d’interface utilisateur de Form Recognizer](https://fott-preview.azurewebsites.net/).
- Suivez un guide de [démarrage rapide de la bibliothèque de client Form Recognizer](quickstarts/client-library.md) pour commencer à écrire une application de traitement de factures avec Form Recognizer dans le langage de votre choix.
- Ou, suivez le guide de démarrage rapide sur l’[extraction de données de facture](./quickstarts/python-invoices.md) pour implémenter une telle extraction à l’aide de Python et de l’API REST.
## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que Form Recognizer ?](./overview.md)
* [Documentation de référence sur l’API REST](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/5ed8c9843c2794cbb1a96291)
