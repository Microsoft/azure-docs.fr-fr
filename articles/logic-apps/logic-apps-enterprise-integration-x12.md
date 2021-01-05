---
title: Envoyer et recevoir des messages X12 pour le B2B
description: Échanger des messages X12 dans le cadre de scénarios d’intégration d’entreprise B2B à l’aide de Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/29/2020
ms.openlocfilehash: 61f3f2af61bc24f76d061de672a3eaacd54f7f0e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015194"
---
# <a name="exchange-x12-messages-for-b2b-enterprise-integration-in-azure-logic-apps-with-enterprise-integration-pack"></a>Échanger des messages X12 dans le cadre d’une intégration d’entreprise B2B dans Azure Logic Apps avec Enterprise Integration Pack

Pour utiliser des messages X12 dans Azure Logic Apps, vous pouvez vous servir du connecteur X12, qui fournit des déclencheurs et des actions pour la gestion de la communication X12. Pour plus d’informations sur les messages EDIFACT, consultez [Échanger des messages EDIFACT](logic-apps-enterprise-integration-edifact.md).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* L’application logique à partir de laquelle vous souhaitez utiliser le connecteur X12 et un déclencheur qui démarre le flux de travail de votre application logique. Le connecteur X12 fournit uniquement des actions, pas des déclencheurs. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un [compte d’intégration](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) associé à votre abonnement Azure et lié à l’application logique dans laquelle vous prévoyez d’utiliser le connecteur X12. Votre application logique et votre compte d’intégration doivent tous deux exister dans le même emplacement ou dans la même région Azure.

* Au moins deux [partenaires commerciaux](../logic-apps/logic-apps-enterprise-integration-partners.md) déjà définis dans votre compte d’intégration à l’aide du qualificateur d’identité X12.

* Les [schémas](../logic-apps/logic-apps-enterprise-integration-schemas.md) à utiliser pour la validation XML que vous avez déjà ajoutée à votre compte d’intégration. Si vous utilisez des schémas HIPAA (Health Insurance Portability and Accountability Act), consultez les [schémas HIPAA](#hipaa-schemas).

* Pour pouvoir utiliser le connecteur X12, vous devez créer un [contrat](../logic-apps/logic-apps-enterprise-integration-agreements.md) X12 entre vos partenaires commerciaux et le stocker dans votre compte d’intégration. Si vous utilisez des schémas HIPAA (Health Insurance Portability and Accountability Act), vous devez ajouter une section `schemaReferences` à votre contrat. Pour plus d’informations, consultez [Schémas HIPAA](#hipaa-schemas).

<a name="receive-settings"></a>

## <a name="receive-settings"></a>Paramètres de réception

Après avoir défini les propriétés du contrat, vous pouvez configurer la manière dont ce contrat identifie et traite les messages entrants reçus par votre partenaire par l’intermédiaire de ce contrat.

1. Sous **Ajouter**, sélectionnez **Paramètres de réception**.

1. Configurez ces propriétés selon le contrat conclu avec le partenaire qui échange des messages avec vous. Les **Paramètres de réception** sont organisés selon les sections suivantes :

   * [Identificateurs](#inbound-identifiers)
   * [Accusé de réception](#inbound-acknowledgement)
   * [Schémas](#inbound-schemas)
   * [Enveloppes](#inbound-envelopes)
   * [Numéros de contrôle](#inbound-control-numbers)
   * [Validations](#inbound-validations)
   * [Paramètres internes](#inbound-internal-settings)

   Pour obtenir les descriptions des propriétés, consultez les tableaux de cette section.

1. Une fois que vous avez terminé, sélectionnez **OK** pour enregistrer vos paramètres.

<a name="inbound-identifiers"></a>

### <a name="receive-settings---identifiers"></a>Paramètres de réception-identificateurs

![Propriétés d’identificateur pour les messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-identifiers.png)

| Propriété | Description |
|----------|-------------|
| **ISA1 (qualificateur d’autorisation)** | Valeur du qualificateur d’autorisation que vous souhaitez utiliser. La valeur par défaut est **00 - Aucune information d'autorisation présente**. <p>**Remarque** : Si vous sélectionnez d’autres valeurs, spécifiez une valeur pour la propriété **ISA2**. |
| **ISA2** | Valeur des informations d’autorisation à utiliser lorsque la propriété **ISA1** n’est pas **00-aucune information d’autorisation n’est présente**. Cette valeur de propriété doit présenter au minimum un caractère alphanumérique et 10 au maximum. |
| **ISA3 (qualificateur de sécurité)** | Valeur du qualificateur de sécurité que vous souhaitez utiliser. La valeur par défaut est **00 - Aucune information de sécurité présente**. <p>**Remarque** : Si vous sélectionnez d’autres valeurs, spécifiez une valeur pour la propriété **ISA4**. |
| **ISA4** | Valeur des informations de sécurité à utiliser lorsque la propriété **ISA3** n’est pas **00-aucune information de sécurité n’est présente**. Cette valeur de propriété doit présenter au minimum un caractère alphanumérique et 10 au maximum. |
|||

<a name="inbound-acknowledgement"></a>

### <a name="receive-settings---acknowledgement"></a>Paramètres de réception-accusé de réception

![Accusé de réception des messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-acknowledgement.png)

| Propriété | Description |
|----------|-------------|
| **TA1 attendu** | Renvoie un accusé de réception technique (TA1) à l’expéditeur de l’échange. |
| **FA attendu** | Renvoie un accusé de réception fonctionnel (FA) à l’expéditeur de l’échange. <p>Pour la propriété **version FA**, en fonction de la version du schéma, sélectionnez les accusés de réception 997 ou 999. <p>Pour activer la génération de boucles AK2 dans les accusés de réception fonctionnels pour des documents informatisés acceptés, sélectionnez **Inclure une boucle AK2 / IK2**. |
||||

<a name="inbound-schemas"></a>

### <a name="receive-settings---schemas"></a>Paramètres de réception-schémas

![Schémas pour les messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-schemas.png)

Pour cette section, sélectionnez un [schéma](../logic-apps/logic-apps-enterprise-integration-schemas.md) à partir de votre [compte d’intégration](./logic-apps-enterprise-integration-create-integration-account.md) pour chaque type de transaction (ST01) et application d’expéditeur (GS02). Le pipeline de réception EDI désassemble le message entrant en faisant correspondre les valeurs et le schéma définis dans cette section avec les valeurs ST01 et GS02 dans le message entrant et avec le schéma du message entrant. Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

| Propriété | Description |
|----------|-------------|
| **Version** | La version X12 du schéma |
| **Type de transaction (ST01)** | Le type de transaction |
| **Application de l’expéditeur (GS02)** | l’application de l’expéditeur |
| **Schéma** | Le fichier du schéma que vous voulez utiliser |
|||

<a name="inbound-envelopes"></a>

### <a name="receive-settings---envelopes"></a>Paramètres de réception-enveloppes

![Séparateurs à utiliser dans les documents informatisés pour les messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-envelopes.png)

| Propriété | Description |
|----------|-------------|
| **Utilisation d’ISA11** | Le séparateur à utiliser dans un document informatisé : <p>- **Identificateur standard** : Utilisez un point (.) pour la notation décimale, au lieu de la notation décimale du document entrant dans le pipeline de réception EDI. <p>- **Séparateur de répétition** : Spécifiez le séparateur des occurrences répétées d’un élément de données simple ou d’une structure de données répétée. Par exemple, l’accent circonflexe (^) est généralement utilisé comme séparateur de répétition. Pour les schémas HIPAA, vous pouvez uniquement utiliser l’accent circonflexe. |
|||

<a name="inbound-control-numbers"></a>

### <a name="receive-settings---control-numbers"></a>Paramètres de réception-numéros de contrôle

![Gestion des doublons de numéros de contrôle pour les messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-control-numbers.png) 

| Propriété | Description |
|----------|-------------|
| **Interdire les doublons de numéros de contrôle d’échange** | Bloquer les échanges en double. Vérifiez le numéro de contrôle d’échange (ISA13) du numéro de contrôle de l’échange reçu. Si une correspondance est détectée, le pipeline de réception EDI ne traite pas l’échange. <p><p>Pour spécifier la fréquence en nombre de jours à laquelle le contrôle est effectué, saisissez une valeur appropriée pour la propriété **Vérifier les doublons ISA13 tous les (jours)** . |
| **Interdire les numéros de contrôle de groupe en double** | Bloquez les échanges avec des numéros de contrôle de groupe en double. |
| **Interdire les numéros de contrôle de document informatisé en double** | Bloquez les échanges avec des numéros de contrôle de document informatisé en double. |
|||

<a name="inbound-validations"></a>

### <a name="receive-settings---validations"></a>Paramètres de réception-validations

![Validations des messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-validations.png)

La ligne **Par défaut** affiche les règles de validation utilisées pour un type de message EDI. Si vous souhaitez définir des règles différentes, sélectionnez chaque boîte dans laquelle l’ensemble de règles est défini sur **true**. Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

| Propriété | Description |
|----------|-------------|
| **Type de message** | Le type de message EDI |
| **Validation EDI** | Effectuez une validation EDI sur les types de données selon les propriétés EDI du schéma, les restrictions de longueur, les éléments de données vides et les séparateurs de fin. |
| **Validation étendue** | Si le type de données n’est pas EDI, la validation s’effectue sur l’exigence des éléments de données et la répétition autorisée, les énumérations et la validation de la longueur des élément de données (min/max). |
| **Autoriser les zéros de début ou de fin** | Conservez les zéros de début ou de fin ainsi que les espaces supplémentaires. Ne supprimez pas ces caractères. |
| **Découper les zéros de début ou de fin** | Supprimez les zéros de début ou de fin ainsi que les espaces. |
| **Stratégie de séparateur de fin** | Générez des séparateurs de fin. <p>- **Non autorisé** : Interdisez les délimiteurs et les séparateurs de fin dans l’échange entrant. Si l’échange contient des délimiteurs et des séparateurs, il est déclaré non valide. <p>- **Facultatif** : Acceptez les échanges avec ou sans délimiteurs et séparateurs de fin. <p>- **Obligatoire** : L’échange entrant doit avoir des délimiteurs et des séparateurs de fin. |
|||

<a name="inbound-internal-settings"></a>

### <a name="receive-settings---internal-settings"></a>Paramètres de réception-paramètres internes

![Paramètres internes des messages entrants](./media/logic-apps-enterprise-integration-x12/x12-receive-settings-internal-settings.png)

| Propriété | Description |
|----------|-------------|
| **Convertir le format décimal implicite « Nn » en valeur numérique de base 10** | Convertissez un numéro EDI spécifié au format « Nn » en une valeur numérique de base 10. |
| **Créer des balises XML vides si les séparateurs de fin sont autorisés** | Incluez des balises XML vides pour les séparateurs de fin de l’expéditeur de l’échange. |
| **Fractionner l’échange en documents informatisés - suspendre les documents informatisés en cas d’erreur** | Analysez chaque document informatisé d’un échange dans un document XML distinct en appliquant l’enveloppe appropriée au document informatisé. Suspendez uniquement les transactions dont la validation échoue. |
| **Fractionner l’échange en documents informatisés - suspendre l’échange en cas d’erreur** | Analysez chaque document informatisé d’un échange dans un document XML distinct en appliquant l’enveloppe appropriée. Suspendez l’intégralité de l’échange lorsque la validation d’un ou plusieurs documents informatisés de l’échange échoue. |
| **Préserver l’échange - suspendre les documents informatisés en cas d’erreur** | Préservez l’échange intacte et créez un document XML pour l’intégralité de l’échange par lot. Suspendez uniquement les documents informatisés dont la validation échoue, tout en continuant à traiter tous les autres documents informatisés. |
| **Préserver l’échange - suspendre l’échange en cas d’erreur** |Préserve l’échange : crée un document XML pour l’intégralité de l’échange par lot. Suspend l’intégralité de l’échange lorsque la validation d’un ou plusieurs documents informatisés de l’échange échoue. |
|||

<a name="send-settings"></a>

## <a name="send-settings"></a>Paramètres d’envoi

Après avoir défini les propriétés du contrat, vous pouvez configurer la manière dont ce contrat identifie et traite les messages entrants envoyés à votre partenaire par l’intermédiaire de ce contrat.

1. Sous **Ajouter**, sélectionnez **Paramètres d’envoi**.

1. Configurez ces propriétés selon le contrat conclu avec le partenaire qui échange des messages avec vous. Pour obtenir les descriptions des propriétés, consultez les tableaux de cette section.

   Les **Paramètres d’envoi** sont organisés selon les sections suivantes :

   * [Identificateurs](#outbound-identifiers)
   * [Accusé de réception](#outbound-acknowledgement)
   * [Schémas](#outbound-schemas)
   * [Enveloppes](#outbound-envelopes)
   * [Numéro de version de contrôle](#outbound-control-version-number)
   * [Numéros de contrôle](#outbound-control-numbers)
   * [Jeux de caractères et séparateurs](#outbound-character-sets-separators)
   * [Validation](#outbound-validation)

1. Une fois que vous avez terminé, sélectionnez **OK** pour enregistrer vos paramètres.

<a name="outbound-identifiers"></a>

### <a name="send-settings---identifiers"></a>Paramètres d’envoi-identificateurs

![Propriétés d’identificateur pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-identifiers.png)

| Propriété | Description |
|----------|-------------|
| **ISA1 (qualificateur d’autorisation)** | Valeur du qualificateur d’autorisation que vous souhaitez utiliser. La valeur par défaut est **00 - Aucune information d'autorisation présente**. <p>**Remarque** : Si vous sélectionnez d’autres valeurs, spécifiez une valeur pour la propriété **ISA2**. |
| **ISA2** | Valeur des informations d’autorisation à utiliser lorsque la propriété **ISA1** n’est pas **00-aucune information d’autorisation n’est présente**. Cette valeur de propriété doit présenter au minimum un caractère alphanumérique et 10 au maximum. |
| **ISA3 (qualificateur de sécurité)** | Valeur du qualificateur de sécurité que vous souhaitez utiliser. La valeur par défaut est **00 - Aucune information de sécurité présente**. <p>**Remarque** : Si vous sélectionnez d’autres valeurs, spécifiez une valeur pour la propriété **ISA4**. |
| **ISA4** | Valeur des informations de sécurité à utiliser lorsque la propriété **ISA3** n’est pas **00-aucune information de sécurité n’est présente**. Cette valeur de propriété doit présenter au minimum un caractère alphanumérique et 10 au maximum. |
|||

<a name="outbound-acknowledgement"></a>

### <a name="send-settings---acknowledgement"></a>Paramètres d’envoi-accusé de réception

![Propriétés de l’accusé de réception pour les messages entrants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-acknowledgement.png)

| Propriété | Description |
|----------|-------------|
| **TA1 attendu** | Renvoie un accusé de réception technique (TA1) à l’expéditeur de l’échange. <p>Ce paramètre spécifie que le partenaire hôte qui envoie le message demande un accusé de réception de la part du partenaire invité figurant dans le contrat. Ces accusés de réception sont attendus par le partenaire hôte en fonction des paramètres de réception du contrat. |
| **FA attendu** | Renvoie un accusé de réception fonctionnel (FA) à l’expéditeur de l’échange. Pour la propriété **version FA**, en fonction de la version du schéma, sélectionnez les accusés de réception 997 ou 999. <p>Ce paramètre spécifie que le partenaire hôte qui envoie le message demande un accusé de réception de la part du partenaire invité figurant dans le contrat. Ces accusés de réception sont attendus par le partenaire hôte en fonction des paramètres de réception du contrat. |
|||

<a name="outbound-schemas"></a>

### <a name="send-settings---schemas"></a>Paramètres d’envoi-schémas

![Schémas pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-schemas.png)

Pour cette section, sélectionnez un [schéma](../logic-apps/logic-apps-enterprise-integration-schemas.md) à partir de votre [compte d’intégration](./logic-apps-enterprise-integration-create-integration-account.md) pour chaque type de transaction (ST01). Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

| Propriété | Description |
|----------|-------------|
| **Version** | La version X12 du schéma |
| **Type de transaction (ST01)** | Le type de transaction du schéma |
| **Schéma** | Le fichier du schéma que vous voulez utiliser. Si vous sélectionnez d’abord le schéma, la version et le type de transaction sont automatiquement définis. |
|||

<a name="outbound-envelopes"></a>

### <a name="send-settings---envelopes"></a>Paramètres d’envoi-enveloppes

![Séparateurs dans un document informatisé à utiliser pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-envelopes.png)

| Propriété | Description |
|----------|-------------|
| **Utilisation d’ISA11** | Le séparateur à utiliser dans un document informatisé : <p>- **Identificateur standard** : Utilisez un point (.) pour la notation décimale, au lieu de la notation décimale du document sortant dans le pipeline d’envoi EDI. <p>- **Séparateur de répétition** : Spécifiez le séparateur des occurrences répétées d’un élément de données simple ou d’une structure de données répétée. Par exemple, l’accent circonflexe (^) est généralement utilisé comme séparateur de répétition. Pour les schémas HIPAA, vous pouvez uniquement utiliser l’accent circonflexe. |
|||

<a name="outbound-control-version-number"></a>

### <a name="send-settings---control-version-number"></a>Paramètre d’envoi-numéro de version du contrôle

![Numéro de version du contrôle pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-control-version-number.png)

Pour cette section, sélectionnez un [schéma](../logic-apps/logic-apps-enterprise-integration-schemas.md) à partir de votre [compte d’intégration](./logic-apps-enterprise-integration-create-integration-account.md) pour chaque échange. Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

| Propriété | Description |
|----------|-------------|
| **Numéro de version du contrôle (ISA12)** | La version de la norme X12 |
| **Indicateur d’utilisation (ISA15)** | Le contexte d’un échange, qui représente des données de **Test**, des données d’**Informations** ou des données de **Production** |
| **Schéma** | Le schéma à utiliser pour générer les segments GS et ST pour un échange X12 encodé envoyé au pipeline d’envoi EDI. |
| **GS1** | Facultatif, sélectionnez le code fonctionnel. |
| **GS2** | Facultatif, spécifiez l’expéditeur de l’application. |
| **GS3** | Facultatif, spécifiez le destinataire de l’application. |
| **GS4** | Facultatif, sélectionnez **CCYYMMDD** ou **YYMMDD**. |
| **GS5** | Facultatif, sélectionnez **HHMM**, **HHMMSS** ou **HHMMSSdd**. |
| **GS7** | Facultatif, sélectionnez une valeur pour l'agence responsable. |
| **GS8** | Facultatif, spécifiez la version du document de schéma. |
|||

<a name="outbound-control-numbers"></a>

### <a name="send-settings---control-numbers"></a>Paramètres d’envoi-numéros de contrôle

![Numéros de contrôle pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-control-numbers.png)

| Propriété | Description |
|----------|-------------|
| **Numéro de contrôle de l’échange (ISA13)** | Plage de valeurs pour le numéro de contrôle de l’échange, qui peut avoir une valeur minimale de 1 et une valeur maximale de 999999999 |
| **Numéro de contrôle de groupe (GS06)** | Plage de valeurs pour le numéro de contrôle de groupe, qui peut avoir une valeur minimale de 1 et une valeur maximale de 999999999 |
| **Numéro de contrôle de document informatisé (ST02)** | Plage de valeurs pour le numéro de contrôle de document informatisé, qui peut avoir une valeur minimale de 1 et une valeur maximale de 999999999 <p>- **Préfixe** : Facultatif, une valeur alphanumérique <br>- **Suffixe** : Facultatif, une valeur alphanumérique |
|||

<a name="outbound-character-sets-separators"></a>

### <a name="send-settings---character-sets-and-separators"></a>Paramètres d’envoi-jeux de caractères et séparateurs

![Délimiteurs pour les types de messages dans les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-character-sets-separators.png)

La ligne **Par défaut** affiche le jeu de caractères utilisé comme délimiteurs pour un schéma de message. Si vous voulez utiliser le jeu de caractères **Par défaut**, vous pouvez entrer un autre ensemble de délimiteurs pour chaque type de message. Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

> [!TIP]
> Pour fournir des valeurs de caractère spécial, modifiez l’accord au format JSON et fournissez la valeur ASCII du caractère spécial.

| Propriété | Description |
|----------|-------------|
| **Jeu de caractères à utiliser** | Le jeu de caractères X12, qui est **De base**, **Étendu** ou **UTF8**. |
| **Schéma** | Le schéma que vous voulez utiliser. Après avoir sélectionné le schéma, sélectionnez le jeu de caractères à utiliser, en fonction des descriptions de séparateurs ci-dessous. |
| **Type d’entrée** | Le type d’entrée pour le jeu de caractères |
| **Séparateur de composants** | Un caractère unique qui sépare les éléments de données composites |
| **Séparateur d'éléments de données** | Un caractère unique qui sépare les éléments de données simples au sein des données composites |
| **Séparateur de caractères de remplacement** | Un caractère de remplacement qui remplace tous les caractères de séparation dans les données de charge utile lors de la génération du message X12 sortant |
| **Terminateur de segment** | Un caractère unique qui indique la fin d’un segment EDI |
| **Suffixe** | Le caractère utilisé avec l’identificateur de segment. Si vous désignez un suffixe, l'élément de données du terminateur de segment peut être vide. Si le terminateur de segment est vide, vous devez désigner un suffixe. |
|||

<a name="outbound-validation"></a>

### <a name="send-settings---validation"></a>Paramètres d’envoi-validation

![Propriétés de validation pour les messages sortants](./media/logic-apps-enterprise-integration-x12/x12-send-settings-validation.png) 

La ligne **Par défaut** affiche les règles de validation utilisées pour un type de message EDI. Si vous souhaitez définir des règles différentes, sélectionnez chaque boîte dans laquelle l’ensemble de règles est défini sur **true**. Une fois chaque ligne remplie, une nouvelle ligne vide s’affiche automatiquement.

| Propriété | Description |
|----------|-------------|
| **Type de message** | Le type de message EDI |
| **Validation EDI** | Effectuez une validation EDI sur les types de données selon les propriétés EDI du schéma, les restrictions de longueur, les éléments de données vides et les séparateurs de fin. |
| **Validation étendue** | Si le type de données n’est pas EDI, la validation s’effectue sur l’exigence des éléments de données et la répétition autorisée, les énumérations et la validation de la longueur des élément de données (min/max). |
| **Autoriser les zéros de début ou de fin** | Conservez les zéros de début ou de fin ainsi que les espaces supplémentaires. Ne supprimez pas ces caractères. |
| **Découper les zéros de début ou de fin** | Supprimez les zéros de début ou de fin ainsi que les espaces. |
| **Stratégie de séparateur de fin** | Générez des séparateurs de fin. <p>- **Non autorisé** : Interdisez les délimiteurs et les séparateurs de fin dans l’échange sortant. Si l’échange contient des délimiteurs et des séparateurs, il est déclaré non valide. <p>- **Facultatif** : Envoyez des échanges avec ou sans délimiteurs et séparateurs de fin. <p>- **Obligatoire** : L’échange sortant doit avoir des délimiteurs et des séparateurs de fin. |
|||

<a name="hipaa-schemas"></a>

## <a name="hipaa-schemas-and-message-types"></a>Schémas HIPAA et types de messages

Lorsque vous utilisez des schémas HIPAA et les types de message 277 ou 837, vous devez effectuer quelques étapes supplémentaires. Les [numéros de version de document (GS8)](#outbound-control-version-number) pour ces types de messages comportent plus de 9 caractères, par exemple, « 005010X222A1 ». En outre, certains numéros de version de document sont mappés à des types de messages des variants. Si vous ne faites pas référence au type de message approprié dans votre schéma et dans votre contrat, vous recevez ce message d’erreur :

`"The message has an unknown document type and did not resolve to any of the existing schemas configured in the agreement."`

Ce tableau répertorie les types de messages affectés, les variantes et les numéros de version de document mappés à ces types de messages :

| Type de message ou variant |  Description | Numéro de version de document (GS8) |
|-------------------------|--------------|-------------------------------|
| 277 | Notification d'état d'informations relatives aux services de santé | 005010X212 |
| 837_I | Demande de remboursement de frais médicaux par les organismes institutionnels | 004010X096A1 <br>005010X223A1 <br>005010X223A2 |
| 837_D | Demande de remboursement de soins dentaires | 004010X097A1 <br>005010X224A1 <br>005010X224A2 |
| 837_P | Demande de remboursement de frais médicaux par les organismes professionnels | 004010X098A1 <br>005010X222 <br>005010X222A1 |
|||

Vous devez également désactiver la validation EDI lorsque vous utilisez ces numéros de version de document, car ils provoquent une erreur indiquant que la longueur des caractères n’est pas valide.

Pour spécifier ces numéros de version de document et types de messages, procédez comme suit :

1. Dans votre schéma HIPAA, remplacez le type de message actuel par le type de message du variant correspondant au numéro de version du document que vous souhaitez utiliser.

   Supposons, par exemple, que vous souhaitiez utiliser le numéro de version du document `005010X222A1` avec le type de message `837`. Dans votre schéma, remplacez chaque valeur `"X12_00501_837"` par la valeur `"X12_00501_837_P"`.

   Pour mettre à jour votre schéma, procédez comme suit :

   1. Dans le Portail Azure, accédez à votre compte d’intégration. Recherchez et téléchargez votre schéma. Remplacez le type de message et renommez le fichier de schéma, puis chargez votre schéma révisé dans votre compte d’intégration. Pour plus d’informations, consultez [Modifier les schémas](../logic-apps/logic-apps-enterprise-integration-schemas.md#edit-schemas).

   1. Dans les paramètres de message de votre contrat, sélectionnez le schéma révisé.

1. Dans l’objet `schemaReferences` de votre contrat, ajoutez une autre entrée qui spécifie le type de message du variant correspondant au numéro de version de document.

   Par exemple, supposons que vous voulez utiliser le numéro de version de document `005010X222A1` pour le type de message `837`. Votre contrat a une section `schemaReferences` avec les propriétés et valeurs suivantes :

   ```json
   "schemaReferences": [
      {
         "messageId": "837",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837"
      }
   ]
   ```

   Dans cette section `schemaReferences`, ajoutez une autre entrée qui a les valeurs suivantes :

   * `"messageId": "837_P"`
   * `"schemaVersion": "00501"`
   * `"schemaName": "X12_00501_837_P"`

   Une fois que vous avez terminé, votre section `schemaReferences`ressemble à cet exemple :

   ```json
   "schemaReferences": [
      {
         "messageId": "837",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837"
      },
      {
         "messageId": "837_P",
         "schemaVersion": "00501",
         "schemaName": "X12_00501_837_P"
      }
   ]
   ```

1. Dans les paramètres de message de votre contrat, désactivez la validation EDI en décochant la case **Validation EDI** pour chaque type de message ou pour tous les types de message si vous utilisez les valeurs **Par défaut**.

   ![Désactiver la validation pour tous les types de messages ou chaque type de message](./media/logic-apps-enterprise-integration-x12/x12-disable-validation.png) 

## <a name="connector-reference"></a>Référence de connecteur

Pour obtenir des informations techniques complémentaires sur ce connecteur, notamment au sujet des actions et des limites décrites dans le fichier Swagger du connecteur, consultez la [page de référence du connecteur](/connectors/x12/).

> [!NOTE]
> Pour les applications logiques utilisées dans un [environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la version de ce connecteur avec l’étiquette ISE applique les [limites de messages B2B de l’ISE](../logic-apps/logic-apps-limits-and-config.md#b2b-protocol-limits).

## <a name="next-steps"></a>Étapes suivantes

* Plus d’informations sur les autres [connecteurs pour Logic apps](../connectors/apis-list.md)
