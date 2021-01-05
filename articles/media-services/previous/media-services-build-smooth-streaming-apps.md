---
title: Didacticiel d’application Windows Store de diffusion en continu lisse | Microsoft Docs
description: Apprenez à utiliser Azure Media Services pour créer une application Windows Store en C# avec un contrôle MediaElement XML pour lire du contenu de diffusion en continu lisse.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 0fa5d8c5-3d5f-4886-ae55-fb6de4f5256d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 9b616b03698e83d256a7b81e4548edbf21dba058
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021127"
---
# <a name="how-to-build-a-smooth-streaming-windows-store-application"></a>Génération d'une application Windows Store de diffusion en continu lisse

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Le Kit de développement logiciel (SDK) du client de diffusion en continu lisse pour Windows 8 permet aux développeurs de générer des applications Windows Store pour la lecture de contenu de diffusion en continu lisse à la demande et en direct. Outre la lecture de contenu de diffusion en continu lisse, le Kit de développement logiciel propose également des fonctionnalités avancées comme la protection Microsoft PlayReady, la limitation du niveau de qualité, le magnétoscope numérique en direct (Live DVR), la commutation des flux audio, l'écoute des mises à jour d'état (par exemple, les modifications des niveaux de qualité), les événements d'erreur, etc. Pour plus d'informations sur les fonctionnalités prises en charge, consultez les [notes de publicationn](https://www.iis.net/learn/media/smooth-streaming/smooth-streaming-client-sdk-for-windows-8-release-notes). Pour plus d’informations, consultez [Player Framework for Windows 8](https://playerframework.codeplex.com/). 

Le didacticiel se compose de quatre leçons :

1. Création d'une application Windows Store de diffusion en continu lisse de base
2. Ajout d'une barre de curseur pour contrôler la progression des fichiers multimédias
3. Sélection des flux de diffusion en continu lisse
4. Sélection des pistes de diffusion en continu lisse

## <a name="prerequisites"></a>Prérequis
> [!NOTE]
> Les projets Windows Store version 8.1 et versions antérieures ne sont pas pris en charge dans Visual Studio 2017.  Pour en savoir plus, consultez [Plateforme cible et compatibilité dans Visual Studio 2017](https://www.visualstudio.com/en-us/productinfo/vs2017-compatibility-vs).

* Windows 8 32 bits ou 64 bits.
* Visual Studio versions 2012 à 2015.
* [Kit de développement logiciel (SDK) du client Microsoft de diffusion en continu lisse pour Windows 8](https://visualstudiogallery.msdn.microsoft.com/04423d13-3b3e-4741-a01c-1ae29e84fea6?SRC=Home).

Vous pouvez télécharger la solution terminée pour chaque leçon sur le site d'exemples de code développeur MSDN (galerie de code) : 

* [Leçon 1](https://code.msdn.microsoft.com/Smooth-Streaming-Client-0bb1471f) : un lecteur multimédia Smooth Streaming pour Windows 8, 
* [Leçon 2](https://code.msdn.microsoft.com/A-simple-Windows-8-Smooth-ee98f63a) : un lecteur multimédia simple Smooth Streaming pour Windows 8 doté d’une fonction de contrôle avec barre de curseur, 
* [Leçon 3](https://code.msdn.microsoft.com/A-Windows-8-Smooth-883c3b44) : un lecteur multimédia Smooth Streaming pour Windows 8 avec sélection de flux,  
* [Leçon 4](https://code.msdn.microsoft.com/A-Windows-8-Smooth-aa9e4907) : un lecteur multimédia Smooth Streaming pour Windows 8 avec sélection des pistes.

## <a name="lesson-1-create-a-basic-smooth-streaming-store-application"></a>Leçon 1 : Création d'une application Windows Store de diffusion en continu lisse de base

Dans cette leçon, vous allez apprendre à créer une application Windows Store dotée d'un contrôle MediaElement pour lire du contenu de diffusion en continu lisse.  L'application en cours d'exécution ressemble à ceci :

![Exemple d'application Windows Store de diffusion en continu lisse][PlayerApplication]

Pour plus d'informations sur le développement d'une application Windows Store, consultez la rubrique [Développement d'applications fantastiques pour Windows 8](https://developer.microsoft.com/en-us/windows/). Cette leçon aborde les procédures suivantes :

1. Création d'un projet Windows Store
2. Conception de l'interface utilisateur (XAML)
3. Modification du fichier code-behind
4. Compilation et test de l’application

### <a name="to-create-a-windows-store-project"></a>Pour créer un projet Windows Store

1. Exécutez Visual Studio ; les versions 2012 à 2015 sont prises en charge.
1. Dans le menu **FICHIER**, cliquez sur **Nouveau**, puis sur **Projet**.
1. Dans la boîte de dialogue Nouveau projet, entrez ou sélectionnez les valeurs suivantes :

    | Nom | Valeur |
    | --- | --- |
    | Groupe de modèles |Installed/Templates/Visual C#/Windows Store |
    | Modèle |Application vide (XAML) |
    | Nom |SSPlayer |
    | Emplacement |C:\SSTutorials |
    | Nom de solution |SSPlayer |
    | Créer un répertoire pour la solution |(sélectionné) |

1. Cliquez sur **OK**.

### <a name="to-add-a-reference-to-the-smooth-streaming-client-sdk"></a>Pour ajouter une référence au Kit de développement logiciel (SDK) du client Smooth Streaming

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **SSPlayer**, puis cliquez sur **Ajouter une référence**.
1. Tapez ou sélectionnez les valeurs suivantes :

    | Nom | Valeur |
    | --- | --- |
    | Groupe de référence |Windows/Extensions |
    | Informations de référence |Sélectionnez le Kit de développement logiciel (SDK) du client de diffusion en continu lisse pour Windows 8 et le package Runtime Microsoft Visual C++ |

1. Cliquez sur **OK**. 

Après l'ajout des références, vous devez sélectionner la plateforme ciblée (x64 ou x86). L'ajout de références ne fonctionne pas pour les configurations de plateforme CPU.  Dans l'Explorateur de solutions, un symbole d'avertissement jaune s'affiche pour indiquer les références ajoutées.

### <a name="to-design-the-player-user-interface"></a>Pour concevoir l’interface utilisateur du lecteur

1. Dans l'Explorateur de solutions, double-cliquez sur **MainPage.xaml** pour l'ouvrir en mode Création.
2. Recherchez les balises **&lt;Grid&gt;** et **&lt;/Grid&gt;** du fichier XAML et collez le code suivant entre elles :

   ```xml
         <Grid.RowDefinitions>

            <RowDefinition Height="20"/>    <!-- spacer -->
            <RowDefinition Height="50"/>    <!-- media controls -->
            <RowDefinition Height="100*"/>  <!-- media element -->
            <RowDefinition Height="80*"/>   <!-- media stream and track selection -->
            <RowDefinition Height="50"/>    <!-- status bar -->
         </Grid.RowDefinitions>

         <StackPanel Name="spMediaControl" Grid.Row="1" Orientation="Horizontal">
            <TextBlock x:Name="tbSource" Text="Source :  " FontSize="16" FontWeight="Bold" VerticalAlignment="Center" />
            <TextBox x:Name="txtMediaSource" Text="https://ecn.channel9.msdn.com/o9/content/smf/smoothcontent/elephantsdream/Elephants_Dream_1024-h264-st-aac.ism/manifest" FontSize="10" Width="700" Margin="0,4,0,10" />
            <Button x:Name="btnSetSource" Content="Set Source" Width="111" Height="43" Click="btnSetSource_Click"/>
            <Button x:Name="btnPlay" Content="Play" Width="111" Height="43" Click="btnPlay_Click"/>
            <Button x:Name="btnPause" Content="Pause"  Width="111" Height="43" Click="btnPause_Click"/>
            <Button x:Name="btnStop" Content="Stop"  Width="111" Height="43" Click="btnStop_Click"/>
            <CheckBox x:Name="chkAutoPlay" Content="Auto Play" Height="55" Width="Auto" IsChecked="{Binding AutoPlay, ElementName=mediaElement, Mode=TwoWay}"/>
            <CheckBox x:Name="chkMute" Content="Mute" Height="55" Width="67" IsChecked="{Binding IsMuted, ElementName=mediaElement, Mode=TwoWay}"/>
         </StackPanel>

         <StackPanel Name="spMediaElement" Grid.Row="2" Height="435" Width="1072"
                    HorizontalAlignment="Center" VerticalAlignment="Center">
            <MediaElement x:Name="mediaElement" Height="356" Width="924" MinHeight="225"
                          HorizontalAlignment="Center" VerticalAlignment="Center" 
                          AudioCategory="BackgroundCapableMedia" />
            <StackPanel Orientation="Horizontal">
                <Slider x:Name="sliderProgress" Width="924" Height="44"
                        HorizontalAlignment="Center" VerticalAlignment="Center"
                        PointerPressed="sliderProgress_PointerPressed"/>
                <Slider x:Name="sliderVolume" 
                        HorizontalAlignment="Right" VerticalAlignment="Center" Orientation="Vertical" 
                        Height="79" Width="148" Minimum="0" Maximum="1" StepFrequency="0.1" 
                        Value="{Binding Volume, ElementName=mediaElement, Mode=TwoWay}" 
                        ToolTipService.ToolTip="{Binding Value, RelativeSource={RelativeSource Mode=Self}}"/>
            </StackPanel>
         </StackPanel>

         <StackPanel Name="spStatus" Grid.Row="4" Orientation="Horizontal">
            <TextBlock x:Name="tbStatus" Text="Status :  " 
               FontSize="16" FontWeight="Bold" VerticalAlignment="Center" HorizontalAlignment="Center" />
            <TextBox x:Name="txtStatus" FontSize="10" Width="700" VerticalAlignment="Center"/>
         </StackPanel>
   ```
   Le contrôle MediaElement permet de lire des fichiers multimédias. La commande de réglage nommée sliderProgress servira dans la prochaine leçon à contrôler la progression du fichier multimédia.
3. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

Le contrôle MediaElement ne prend pas en charge le contenu de diffusion en continu lisse sans configuration supplémentaire. Pour activer la prise en charge de la diffusion en continu lisse, vous devez enregistrer le Gestionnaire de flux d'octets de diffusion en continu lisse en fonction de l'extension du nom de fichier et du type MIME.  Pour ce faire, utilisez la méthode MediaExtensionManager.RegisterByteStreamHandler de l'espace de noms Windows.Media.

Dans le fichier XAML, certains gestionnaires d'événements sont associés aux contrôles.  Vous devez définir les gestionnaires d'événements en question.

### <a name="to-modify-the-code-behind-file"></a>Pour modifier le fichier code-behind

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Au début du fichier, ajoutez l'instruction using suivante :

    ```csharp
        using Windows.Media;
    ```

3. Au début de la classe **MainPage** , ajoutez le membre de données suivant :

    ```csharp
        private MediaExtensionManager extensions = new MediaExtensionManager();
    ```

4. À la fin du constructeur **MainPage** , ajoutez les deux lignes suivantes :

    ```csharp
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "text/xml");
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "application/vnd.ms-sstr+xml");
    ```

5. À la fin de la classe **MainPage** , collez le code suivant :
   ```csharp
         # region UI Button Click Events
         private void btnPlay_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Play();
         txtStatus.Text = "MediaElement is playing ...";
         }
         private void btnPause_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Pause();
         txtStatus.Text = "MediaElement is paused";
         }
         private void btnSetSource_Click(object sender, RoutedEventArgs e)
         {

         sliderProgress.Value = 0;
         mediaElement.Source = new Uri(txtMediaSource.Text);

         if (chkAutoPlay.IsChecked == true)
         {
             txtStatus.Text = "MediaElement is playing ...";
         }
         else
         {
             txtStatus.Text = "Click the Play button to play the media source.";
         }
         }
         private void btnStop_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Stop();
         txtStatus.Text = "MediaElement is stopped";
         }
         private void sliderProgress_PointerPressed(object sender, PointerRoutedEventArgs e)
         {

         txtStatus.Text = "Seek to position " + sliderProgress.Value;
         mediaElement.Position = new TimeSpan(0, 0, (int)(sliderProgress.Value));
         }
         # endregion
   ```
   Le gestionnaire d'événements sliderProgress_PointerPressed est défini ici.  Pour qu'il fonctionne, il est nécessaire d'effectuer d'autres procédures qui seront décrites dans la leçon suivante de ce didacticiel.
6. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

Le fichier code-behind terminé doit ressembler à ceci :

![Codeview dans Visual Studio d'une application Windows Store de diffusion en continu lisse][CodeViewPic]

### <a name="to-compile-and-test-the-application"></a>Pour compiler et tester l’application

1. Dans le menu **Build**, cliquez sur **Gestionnaire de configuration**.
2. Modifiez l'option **Plateforme de la solution active** en fonction de votre plateforme de développement.
3. Appuyez sur **F6** pour compiler le projet. 
4. Appuyez sur **F5** pour exécuter l'application.
5. Au début de l'application, vous pouvez utiliser l'URL de diffusion en continu lisse par défaut ou entrer une URL différente. 
6. Cliquez sur **Définir la source**. Étant donné que la fonctionnalité **Lecture automatique** est activée par défaut, le fichier multimédia est lu automatiquement.  Vous pouvez contrôler le fichier multimédia à l'aide des boutons **Lecture**, **Pause** et **Arrêt**.  Vous pouvez contrôler le volume du fichier multimédia à l'aide du curseur vertical.  Cependant, le curseur horizontal permettant de contrôler la progression du fichier multimédia n'est pas encore complètement implémenté. 

Vous avez terminé la leçon 1.  Dans cette leçon, vous avez utilisé un contrôle MediaElement pour lire du contenu de diffusion en continu lisse.  Dans la leçon suivante, vous allez ajouter un curseur pour contrôler la progression du contenu de diffusion en continu lisse.

## <a name="lesson-2-add-a-slider-bar-to-control-the-media-progress"></a>Leçon 2 : Ajout d'une barre de curseur pour contrôler la progression des fichiers multimédias

Dans la leçon 1, vous avez créé une application Windows Store dotée d'un contrôle XAML MediaElement pour lire du contenu multimédia de diffusion en continu lisse.  Elle offre des fonctions de lecture multimédia de base, comme le démarrage, l'arrêt et la pause.  Dans cette leçon, vous allez ajouter un contrôle de curseur pour votre application.

Dans le cadre de ce didacticiel, nous utiliserons un minuteur pour mettre à jour la position du curseur en fonction de la position actuelle du contrôle MediaElement.  L'heure de début et de fin du curseur doivent également être mises à jour en cas de diffusion de contenu en direct.  Pour ce faire, mieux vaut utiliser l'événement de mise à jour de la source adaptative.

Les sources multimédias sont des objets qui produisent des données multimédias.  Le résolveur de la source crée la source multimédia appropriée pour un contenu donné à partir d'une URL ou d'un flux d'octets.  Pour créer des sources multimédias dans des applications, la méthode la plus courante est de recourir à un résolveur de source. 

Cette leçon aborde les procédures suivantes :

1. Enregistrement du gestionnaire de diffusion en continu lisse 
2. Ajout de gestionnaires d'événements au niveau d'AdaptiveSourceManager
3. Ajout de gestionnaires d'événements au niveau de la source adaptative
4. Ajout de gestionnaires d'événements MediaElement
5. Ajout du code lié à la barre de curseur
6. Compilation et test de l’application

### <a name="to-register-the-smooth-streaming-byte-stream-handler-and-pass-the-propertyset"></a>Pour enregistrer le gestionnaire de flux d’octets Smooth Streaming et transmettre le paramètre PropertySet

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Au début du fichier, ajoutez l'instruction using suivante :

   ```csharp
        using Microsoft.Media.AdaptiveStreaming;
   ```
3. Au début de la classe MainPage, ajoutez les membres de données suivants :

   ```csharp
         private Windows.Foundation.Collections.PropertySet propertySet = new Windows.Foundation.Collections.PropertySet();             
         private IAdaptiveSourceManager adaptiveSourceManager;
   ```
4. Dans le constructeur **MainPage**, ajoutez le code suivant après la ligne **this.Initialize Components();** et les lignes de code d'enregistrement rédigées dans la précédente leçon :

   ```csharp
        // Gets the default instance of AdaptiveSourceManager which manages Smooth 
        //Streaming media sources.
        adaptiveSourceManager = AdaptiveSourceManager.GetDefault();
        // Sets property key value to AdaptiveSourceManager default instance.
        // {A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}" must be hardcoded.
        propertySet["{A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}"] = adaptiveSourceManager;
   ```
5. Dans le constructeur **MainPage** , modifiez les deux méthodes RegisterByteStreamHandler pour ajouter les paramètres suivants :

   ```csharp
         // Registers Smooth Streaming byte-stream handler for ".ism" extension and, 
         // "text/xml" and "application/vnd.ms-ss" mime-types and pass the propertyset. 
         // http://*.ism/manifest URI resources will be resolved by Byte-stream handler.
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "text/xml", 
            propertySet );
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "application/vnd.ms-sstr+xml", 
         propertySet);
   ```
6. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

### <a name="to-add-the-adaptive-source-manager-level-event-handler"></a>Pour ajouter le gestionnaire d’événements au niveau d’AdaptiveSourceManager

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Dans la classe **MainPage** , ajoutez le membre de données suivant :

   ```csharp
     private AdaptiveSource adaptiveSource = null;
   ```
3. À la fin de la classe **MainPage** , ajoutez le gestionnaire d'événements suivant :

   ```csharp
         # region Adaptive Source Manager Level Events
         private void mediaElement_AdaptiveSourceOpened(AdaptiveSource sender, AdaptiveSourceOpenedEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
         }

         # endregion Adaptive Source Manager Level Events
   ```
4. À la fin du constructeur **MainPage** , ajoutez la ligne suivante pour vous inscrire à l'événement ouvert de la source adaptative :

   ```csharp
         adaptiveSourceManager.AdaptiveSourceOpenedEvent += 
           new AdaptiveSourceOpenedEventHandler(mediaElement_AdaptiveSourceOpened);
   ```
5. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

### <a name="to-add-adaptive-source-level-event-handlers"></a>Pour ajouter des gestionnaires d’événements au niveau de la source adaptative

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Dans la classe **MainPage** , ajoutez le membre de données suivant :

   ```csharp
     private AdaptiveSourceStatusUpdatedEventArgs adaptiveSourceStatusUpdate; 
     private Manifest manifestObject;
   ```
3. À la fin de la classe **MainPage** , ajoutez les gestionnaires d'événements suivants :

   ```csharp
         # region Adaptive Source Level Events
         private void mediaElement_ManifestReady(AdaptiveSource sender, ManifestReadyEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
            manifestObject = args.AdaptiveSource.Manifest;
         }

         private void mediaElement_AdaptiveSourceStatusUpdated(AdaptiveSource sender, AdaptiveSourceStatusUpdatedEventArgs args)
         {

            adaptiveSourceStatusUpdate = args;
         }

         private void mediaElement_AdaptiveSourceFailed(AdaptiveSource sender, AdaptiveSourceFailedEventArgs args)
         {

            txtStatus.Text = "Error: " + args.HttpResponse;
         }

         # endregion Adaptive Source Level Events
   ```
4. À la fin de la méthode **mediaElement AdaptiveSourceOpened** , ajoutez le code suivant pour vous inscrire aux événements :

   ```csharp
         adaptiveSource.ManifestReadyEvent +=

                    mediaElement_ManifestReady;
         adaptiveSource.AdaptiveSourceStatusUpdatedEvent += 

            mediaElement_AdaptiveSourceStatusUpdated;
         adaptiveSource.AdaptiveSourceFailedEvent += 

            mediaElement_AdaptiveSourceFailed;
   ```
5. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

Les mêmes événements sont également disponibles sur AdaptiveSourceManager, qui permet de gérer des fonctionnalités communes à tous les éléments multimédias de l'application. Chaque AdaptiveSource inclut ses propres événements et tous les événements AdaptiveSource sont affichés en cascade dans AdaptiveSourceManager.

### <a name="to-add-media-element-event-handlers"></a>Pour ajouter des gestionnaires d’événements MediaElement

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. À la fin de la classe **MainPage** , ajoutez les gestionnaires d'événements suivants :

   ```csharp
         # region Media Element Event Handlers
         private void MediaOpened(object sender, RoutedEventArgs e)
         {

            txtStatus.Text = "MediaElement opened";
         }

         private void MediaFailed(object sender, ExceptionRoutedEventArgs e)
         {

            txtStatus.Text= "MediaElement failed: " + e.ErrorMessage;
         }

         private void MediaEnded(object sender, RoutedEventArgs e)
         {

            txtStatus.Text ="MediaElement ended.";
         }

         # endregion Media Element Event Handlers
   ```
3. À la fin du constructeur **MainPage** , ajoutez le code suivant pour vous inscrire aux événements :

   ```csharp
         mediaElement.MediaOpened += MediaOpened;
         mediaElement.MediaEnded += MediaEnded;
         mediaElement.MediaFailed += MediaFailed;
   ```
4. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

### <a name="to-add-slider-bar-related-code"></a>Pour ajouter le code lié à la barre de curseur

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Au début du fichier, ajoutez l'instruction using suivante :

   ```csharp
        using Windows.UI.Core;
   ```
3. Dans la classe **MainPage** , ajoutez les membres de données suivants :

   ```csharp
         public static CoreDispatcher _dispatcher;
         private DispatcherTimer sliderPositionUpdateDispatcher;
   ```
4. À la fin du constructeur **MainPage** , ajoutez le code suivant :

   ```csharp
         _dispatcher = Window.Current.Dispatcher;
         PointerEventHandler pointerpressedhandler = new PointerEventHandler(sliderProgress_PointerPressed);
         sliderProgress.AddHandler(Control.PointerPressedEvent, pointerpressedhandler, true);    
   ```
5. À la fin de la classe **MainPage** , ajoutez le code suivant :

   ```csharp
         # region sliderMediaPlayer
         private double SliderFrequency(TimeSpan timevalue)
         {

            long absvalue = 0;
            double stepfrequency = -1;

            if (manifestObject != null)
            {
                absvalue = manifestObject.Duration - (long)manifestObject.StartTime;
            }
            else
            {
                absvalue = mediaElement.NaturalDuration.TimeSpan.Ticks;
            }

            TimeSpan totalDVRDuration = new TimeSpan(absvalue);

            if (totalDVRDuration.TotalMinutes >= 10 && totalDVRDuration.TotalMinutes < 30)
            {
               stepfrequency = 10;
            }
            else if (totalDVRDuration.TotalMinutes >= 30 
                     && totalDVRDuration.TotalMinutes < 60)
            {
                stepfrequency = 30;
            }
            else if (totalDVRDuration.TotalHours >= 1)
            {
                stepfrequency = 60;
            }

            return stepfrequency;
         }

         void updateSliderPositionoNTicks(object sender, object e)
         {

            sliderProgress.Value = mediaElement.Position.TotalSeconds;
         }

         public void setupTimer()
         {

            sliderPositionUpdateDispatcher = new DispatcherTimer();
            sliderPositionUpdateDispatcher.Interval = new TimeSpan(0, 0, 0, 0, 300);
            startTimer();
         }

         public void startTimer()
         {

            sliderPositionUpdateDispatcher.Tick += updateSliderPositionoNTicks;
            sliderPositionUpdateDispatcher.Start();
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderStartTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.StartTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Minimum = absvalue;
            });
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderEndTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Maximum = absvalue;
            });
         }

         # endregion sliderMediaPlayer
   ```

   > [!NOTE]
   > CoreDispatcher permet d'apporter des modifications au thread d'interface utilisateur à partir d'un autre type de thread. En cas de goulot d’étranglement sur le thread du répartiteur, le développeur peut choisir d’utiliser le répartiteur fourni par l’élément d’interface utilisateur à mettre à jour.  Par exemple :

   ```csharp
         await sliderProgress.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => { TimeSpan 

         timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime); 
         double absvalue  = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero); 

         sliderProgress.Maximum = absvalue; }); 
   ```
6. À la fin de la méthode **mediaElement_AdaptiveSourceStatusUpdated**, ajoutez le code suivant :

   ```csharp
         setSliderStartTime(args.StartTime);
         setSliderEndTime(args.EndTime);
   ```
7. À la fin de la méthode **MediaOpened** , ajoutez le code suivant :

   ```csharp
         sliderProgress.StepFrequency = SliderFrequency(mediaElement.NaturalDuration.TimeSpan);
         sliderProgress.Width = mediaElement.Width;
         setupTimer();
   ```
8. Appuyez sur **Ctrl+S** pour enregistrer le fichier.

### <a name="to-compile-and-test-the-application"></a>Pour compiler et tester l’application

1. Appuyez sur **F6** pour compiler le projet. 
2. Appuyez sur **F5** pour exécuter l'application.
3. Au début de l'application, vous pouvez utiliser l'URL de diffusion en continu lisse par défaut ou entrer une URL différente. 
4. Cliquez sur **Définir la source**. 
5. Testez la barre de curseur.

Vous avez terminé la leçon 2.  Dans cette leçon, vous avez ajouté un curseur à l'application. 

## <a name="lesson-3-select-smooth-streaming-streams"></a>Leçon 3 : Sélection des flux de diffusion en continu lisse
La diffusion en continu lisse permet de diffuser du contenu en continu proposant des pistes audio multilingues sélectionnables par les utilisateurs.  Cette leçon vous expliquera comment permettre aux utilisateurs de sélectionner des flux. Cette leçon aborde les procédures suivantes :

1. Modification du fichier XAML
2. Modification du fichier code-behind
3. Compilation et test de l’application

### <a name="to-modify-the-xaml-file"></a>Pour modifier le fichier XAML

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Concepteur de vues**.
2. Recherchez &lt;Grid.RowDefinitions&gt; et modifiez les paramètres RowDefinitions pour qu'ils ressemblent à ceci :

   ```xml
         <Grid.RowDefinitions>            
            <RowDefinition Height="20"/>
            <RowDefinition Height="50"/>
            <RowDefinition Height="100"/>
            <RowDefinition Height="80"/>
            <RowDefinition Height="50"/>
         </Grid.RowDefinitions>
   ```
3. Dans les balises &lt;Grid&gt;&lt;/Grid&gt;, ajoutez le code suivant pour définir un contrôle ListBox, afin que les utilisateurs puissent afficher la liste des flux disponibles et sélectionner des flux :

   ```xml
         <Grid Name="gridStreamAndBitrateSelection" Grid.Row="3">
            <Grid.RowDefinitions>
                <RowDefinition Height="300"/>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="250*"/>
                <ColumnDefinition Width="250*"/>
            </Grid.ColumnDefinitions>
            <StackPanel Name="spStreamSelection" Grid.Row="1" Grid.Column="0">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Name="tbAvailableStreams" Text="Available Streams:" FontSize="16" VerticalAlignment="Center"></TextBlock>
                    <Button Name="btnChangeStreams" Content="Submit" Click="btnChangeStream_Click"/>
                </StackPanel>
                <ListBox x:Name="lbAvailableStreams" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                    ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
                    <ListBox.ItemTemplate>
                        <DataTemplate>
                            <CheckBox Content="{Binding Name}" IsChecked="{Binding isChecked, Mode=TwoWay}" />
                        </DataTemplate>
                    </ListBox.ItemTemplate>
                </ListBox>
            </StackPanel>
         </Grid>
   ```
4. Appuyez sur **Ctrl+S** pour enregistrer les modifications.

### <a name="to-modify-the-code-behind-file"></a>Pour modifier le fichier code-behind

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Dans l'espace de noms SSPlayer, ajoutez une nouvelle classe :

   ```csharp
        #region class Stream
   
        public class Stream
        {
            private IManifestStream stream;
            public bool isCheckedValue;
            public string name;
   
            public string Name
            {
                get { return name; }
                set { name = value; }
            }
   
            public IManifestStream ManifestStream
            {
                get { return stream; }
                set { stream = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    // mMke the video stream always checked.
                    if (stream.Type == MediaStreamType.Video)
                    {
                        isCheckedValue = true;
                    }
                    else
                    {
                        isCheckedValue = value;
                    }
                }
            }
   
            public Stream(IManifestStream streamIn)
            {
                stream = streamIn;
                name = stream.Name;
            }
        }
        #endregion class Stream
   ```
3. Au début de la classe MainPage, ajoutez les définitions de variables suivantes :

   ```csharp
         private List<Stream> availableStreams;
         private List<Stream> availableAudioStreams;
         private List<Stream> availableTextStreams;
         private List<Stream> availableVideoStreams;
   ```
4. Dans la classe MainPage, ajoutez la région suivante :
   ```csharp
        #region stream selection
        ///<summary>
        ///Functionality to select streams from IManifestStream available streams
        /// </summary>
   
        // This function is called from the mediaElement_ManifestReady event handler 
        // to retrieve the streams and populate them to the local data members.
        public void getStreams(Manifest manifestObject)
        {
            availableStreams = new List<Stream>();
            availableVideoStreams = new List<Stream>();
            availableAudioStreams = new List<Stream>();
            availableTextStreams = new List<Stream>();
   
            try
            {
                for (int i = 0; i<manifestObject.AvailableStreams.Count; i++)
                {
                    Stream newStream = new Stream(manifestObject.AvailableStreams[i]);
                    newStream.isChecked = false;
   
                    //populate the stream lists based on the types
                    availableStreams.Add(newStream);
   
                    switch (newStream.ManifestStream.Type)
                    {
                        case MediaStreamType.Video:
                            availableVideoStreams.Add(newStream);
                            break;
                        case MediaStreamType.Audio:
                            availableAudioStreams.Add(newStream);
                            break;
                        case MediaStreamType.Text:
                            availableTextStreams.Add(newStream);
                            break;
                    }
   
                    // Select the default selected streams from the manifest.
                    for (int j = 0; j<manifestObject.SelectedStreams.Count; j++)
                    {
                        string selectedStreamName = manifestObject.SelectedStreams[j].Name;
                        if (selectedStreamName.Equals(newStream.Name))
                        {
                            newStream.isChecked = true;
                            break;
                        }
                    }
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function set the list box ItemSource
        private async void refreshAvailableStreamsListBoxItemSource()
        {
            try
            {
                //update the stream check box list on the UI
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableStreams.ItemsSource = availableStreams; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function creates a selected streams list
        private void createSelectedStreamsList(List<IManifestStream> selectedStreams)
        {
            bool isOneVideoSelected = false;
            bool isOneAudioSelected = false;
   
            // Only one video stream can be selected
            for (int j = 0; j<availableVideoStreams.Count; j++)
            {
                if (availableVideoStreams[j].isChecked && (!isOneVideoSelected))
                {
                    selectedStreams.Add(availableVideoStreams[j].ManifestStream);
                    isOneVideoSelected = true;
                }
            }
   
            // Select the first video stream from the list if no video stream is selected
            if (!isOneVideoSelected)
            {
                availableVideoStreams[0].isChecked = true;
                selectedStreams.Add(availableVideoStreams[0].ManifestStream);
            }
   
            // Only one audio stream can be selected
            for (int j = 0; j<availableAudioStreams.Count; j++)
            {
                if (availableAudioStreams[j].isChecked && (!isOneAudioSelected))
                {
                    selectedStreams.Add(availableAudioStreams[j].ManifestStream);
                    isOneAudioSelected = true;
                    txtStatus.Text = "The audio stream is changed to " + availableAudioStreams[j].ManifestStream.Name;
                }
            }
   
            // Select the first audio stream from the list if no audio steam is selected.
            if (!isOneAudioSelected)
            {
                availableAudioStreams[0].isChecked = true;
                selectedStreams.Add(availableAudioStreams[0].ManifestStream);
            }
   
            // Multiple text streams are supported.
            for (int j = 0; j < availableTextStreams.Count; j++)
            {
                if (availableTextStreams[j].isChecked)
                {
                    selectedStreams.Add(availableTextStreams[j].ManifestStream);
                }
            }
        }
   
        // Change streams on a smooth streaming presentation with multiple video streams.
        private async void changeStreams(List<IManifestStream> selectStreams)
        {
            try
            {
                IReadOnlyList<IStreamChangedResult> returnArgs =
                    await manifestObject.SelectStreamsAsync(selectStreams);
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
        #endregion stream selection
   ```
5. Recherchez la méthode mediaElement_ManifestReady, puis ajoutez le code suivant à la fin de la fonction :
   ```csharp
        getStreams(manifestObject);
        refreshAvailableStreamsListBoxItemSource();
   ```
    Lorsque le manifeste MediaElement est prêt, le code obtient une liste des flux disponibles et l'insère dans la zone de liste d'interface utilisateur.
6. Dans la classe MainPage, recherchez la région d’événements Click des boutons de l’interface utilisateur, puis ajoutez la définition de fonction suivante :
   ```csharp
        private void btnChangeStream_Click(object sender, RoutedEventArgs e)
        {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();
   
            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);
   
            // Change streams on the presentation
            changeStreams(selectedStreams);
        }
   ```

### <a name="to-compile-and-test-the-application"></a>Pour compiler et tester l’application

1. Appuyez sur **F6** pour compiler le projet. 
2. Appuyez sur **F5** pour exécuter l'application.
3. Au début de l'application, vous pouvez utiliser l'URL de diffusion en continu lisse par défaut ou entrer une URL différente. 
4. Cliquez sur **Définir la source**. 
5. La langue par défaut est audio_eng. Essayez de basculer entre audio_eng et audio_es. Vous devez cliquer sur le bouton Envoyer chaque fois que vous sélectionnez un nouveau flux.

Vous avez terminé la leçon 3.  Dans cette leçon, vous avez ajouté la fonctionnalité permettant de choisir des flux.

## <a name="lesson-4-select-smooth-streaming-tracks"></a>Leçon 4 : Sélectionner des pistes Smooth Streaming

Une présentation de diffusion en continu lisse peut contenir plusieurs fichiers vidéo encodés comportant des niveaux de qualité (débit) et des résolutions différents. Cette leçon vous expliquera comment permettre aux utilisateurs de sélectionner des pistes. Cette leçon aborde les procédures suivantes :

1. Modification du fichier XAML
2. Modification du fichier code-behind
3. Compilation et test de l’application

### <a name="to-modify-the-xaml-file"></a>Pour modifier le fichier XAML

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Concepteur de vues**.
2. Recherchez la balise &lt;Grid&gt; nommée **gridStreamAndBitrateSelection**, puis ajoutez le code suivant à la fin de la balise :
   ```xml
         <StackPanel Name="spBitRateSelection" Grid.Row="1" Grid.Column="1">
         <StackPanel Orientation="Horizontal">
             <TextBlock Name="tbBitRate" Text="Available Bitrates:" FontSize="16" VerticalAlignment="Center"/>
             <Button Name="btnChangeTracks" Content="Submit" Click="btnChangeTrack_Click" />
         </StackPanel>
         <ListBox x:Name="lbAvailableVideoTracks" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                  ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
             <ListBox.ItemTemplate>
                 <DataTemplate>
                     <CheckBox Content="{Binding Bitrate}" IsChecked="{Binding isChecked, Mode=TwoWay}"/>
                 </DataTemplate>
             </ListBox.ItemTemplate>
         </ListBox>
         </StackPanel>
   ```
3. Appuyez sur **Ctrl+S** pour enregistrer les modifications

### <a name="to-modify-the-code-behind-file"></a>Pour modifier le fichier code-behind

1. Dans l'Explorateur de solutions, cliquez avec le bouton droit sur **MainPage.xaml**, puis cliquez sur **Afficher le code**.
2. Dans l'espace de noms SSPlayer, ajoutez une nouvelle classe :
   ```csharp
        #region class Track
        public class Track
        {
            private IManifestTrack trackInfo;
            public string _bitrate;
            public bool isCheckedValue;
   
            public IManifestTrack TrackInfo
            {
                get { return trackInfo; }
                set { trackInfo = value; }
            }
   
            public string Bitrate
            {
                get { return _bitrate; }
                set { _bitrate = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    isCheckedValue = value;
                }
            }
   
            public Track(IManifestTrack trackInfoIn)
            {
                trackInfo = trackInfoIn;
                _bitrate = trackInfoIn.Bitrate.ToString();
            }
            //public Track() { }
        }
        #endregion class Track
   ```
3. Au début de la classe MainPage, ajoutez les définitions de variables suivantes :
   ```csharp
        private List<Track> availableTracks;
   ```
4. Dans la classe MainPage, ajoutez la région suivante :
   ```csharp
        #region track selection
        /// <summary>
        /// Functionality to select video streams
        /// </summary>
   
        /// This Function gets the tracks for the selected video stream
        public void getTracks(Manifest manifestObject)
        {
            availableTracks = new List<Track>();
   
            IManifestStream videoStream = getVideoStream();
            IReadOnlyList<IManifestTrack> availableTracksLocal = videoStream.AvailableTracks;
            IReadOnlyList<IManifestTrack> selectedTracksLocal = videoStream.SelectedTracks;
   
            try
            {
                for (int i = 0; i < availableTracksLocal.Count; i++)
                {
                    Track thisTrack = new Track(availableTracksLocal[i]);
                    thisTrack.isChecked = true;
   
                    for (int j = 0; j < selectedTracksLocal.Count; j++)
                    {
                        string selectedTrackName = selectedTracksLocal[j].Bitrate.ToString();
                        if (selectedTrackName.Equals(thisTrack.Bitrate))
                        {
                            thisTrack.isChecked = true;
                            break;
                        }
                    }
                    availableTracks.Add(thisTrack);
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = e.Message;
            }
        }
   
        // This function gets the video stream that is playing
        private IManifestStream getVideoStream()
        {
            IManifestStream videoStream = null;
            for (int i = 0; i < manifestObject.SelectedStreams.Count; i++)
            {
                if (manifestObject.SelectedStreams[i].Type == MediaStreamType.Video)
                {
                    videoStream = manifestObject.SelectedStreams[i];
                    break;
                }
            }
            return videoStream;
        }
   
        // This function set the UI list box control ItemSource
        private async void refreshAvailableTracksListBoxItemSource()
        {
            try
            {
                // Update the track check box list on the UI 
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableVideoTracks.ItemsSource = availableTracks; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }        
        }
   
        // This function creates a list of the selected tracks.
        private void createSelectedTracksList(List<IManifestTrack> selectedTracks)
        {
            // Create a list of selected tracks
            for (int j = 0; j < availableTracks.Count; j++)
            {
                if (availableTracks[j].isCheckedValue == true)
                {
                    selectedTracks.Add(availableTracks[j].TrackInfo);
                }
            }
        }
   
        // This function selects the tracks based on user selection 
        private void changeTracks(List<IManifestTrack> selectedTracks)
        {
            IManifestStream videoStream = getVideoStream();
            try
            {
                videoStream.SelectTracks(selectedTracks);
            }
            catch (Exception ex)
            {
                txtStatus.Text = ex.Message;
            }
        }
        #endregion track selection
   ```
5. Recherchez la méthode mediaElement_ManifestReady, puis ajoutez le code suivant à la fin de la fonction :
   ```csharp
         getTracks(manifestObject);
         refreshAvailableTracksListBoxItemSource();
   ```
6. Dans la classe MainPage, recherchez la région d’événements Click des boutons de l’interface utilisateur, puis ajoutez la définition de fonction suivante :
   ```csharp
         private void btnChangeStream_Click(object sender, RoutedEventArgs e)
         {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();

            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);

            // Change streams on the presentation
            changeStreams(selectedStreams);
         }
   ```
   
### <a name="to-compile-and-test-the-application"></a>Pour compiler et tester l’application

1. Appuyez sur **F6** pour compiler le projet. 
2. Appuyez sur **F5** pour exécuter l'application.
3. Au début de l'application, vous pouvez utiliser l'URL de diffusion en continu lisse par défaut ou entrer une URL différente. 
4. Cliquez sur **Définir la source**. 
5. Par défaut, toutes les pistes du flux vidéo sont sélectionnées. Pour tester l'effet des modifications de débit, vous pouvez sélectionner le débit le plus bas disponible, puis sélectionner le débit le plus élevé disponible. Vous devez cliquer sur Envoyer après chaque modification.  Vous pouvez dès lors constater les modifications de la qualité vidéo.

Vous avez terminé la leçon 4.  Dans cette leçon, vous avez ajouté la fonctionnalité permettant de choisir des pistes.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="other-resources"></a>Autres ressources :
* [Développement d'une application JavaScript de diffusion en continu lisse pour Windows 8 dotée de fonctionnalités avancées](https://blogs.iis.net/cenkd/archive/2012/08/10/how-to-build-a-smooth-streaming-windows-8-javascript-application-with-advanced-features.aspx)
* [Présentation technique de la diffusion en continu lisse](https://www.iis.net/learn/media/on-demand-smooth-streaming/smooth-streaming-technical-overview)

[PlayerApplication]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-1.png
[CodeViewPic]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-2.png
