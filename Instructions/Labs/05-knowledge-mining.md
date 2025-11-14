---
lab:
  title: Erstellen einer Lösung für die Wissensgewinnung
  description: "Verwenden Sie die Azure\_KI-Suche, um wichtige Informationen aus Dokumenten zu extrahieren und die Suche und Analyse zu vereinfachen."
---

# Erstellen einer Lösung für die Wissensgewinnung

In dieser Übung verwenden Sie die KI-Suche, um eine Reihe von Dokumenten zu indizieren, die von Margie's Travel, einem fiktiven Reisebüro, verwaltet werden. Der Indizierungsprozess umfasst die Verwendung von KI-Fähigkeiten, um wichtige Informationen zu extrahieren, sie in eine durchsuchbare Form zu bringen und einen Wissensspeicher mit Datenressourcen für die weitere Analyse zu generieren.

Obwohl diese Übung auf Python basiert, können Sie ähnliche Anwendungen mit mehreren sprachspezifischen SDKs entwickeln, einschließlich:

- [Clientbibliothek für Azure KI-Suche für Python](https://pypi.org/project/azure-search-documents/)
- [Clientbibliothek für Azure KI-Suche für Microsoft .NET](https://www.nuget.org/packages/Azure.Search.Documents)
- [Clientbibliothek für Azure KI-Suche für JavaScript](https://www.npmjs.com/package/@azure/search-documents)

Diese Übung dauert ca. **40** Minuten.

## Erstellen von Azure-Ressourcen

Die Lösung, die Sie für Margie's Travel erstellen werden, erfordert mehrere Ressourcen in Ihrem Azure-Abonnement. In dieser Übung erstellen Sie diese direkt im Azure-Portal. Sie könnten die Ressourcen auch mithilfe eines Skripts oder einer ARM- oder BICEP-Vorlage erstellen. Alternativ könnten Sie auch ein Azure AI Foundry-Projekt erstellen, das eine Ressource für Azure KI-Suche enthält.

> **Wichtig:** Ihre Azure-Ressourcen müssen am selben Standort erstellt werden!

### Erstellen einer Azure KI-Suche-Ressource

1. Öffnen Sie in einem Webbrowser das [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com`, und melden Sie sich mit Ihren Azure-Anmeldeinformationen an.
1. Wählen Sie die Schaltfläche **&#65291;Ressource erstellen** aus, und suchen Sie nach `Azure AI Search`. Erstellen Sie eine Ressource vom Typ **Azure KI-Suche** mit den folgenden Einstellungen:
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Dienstname**: *Ein gültiger Name für die Ressourcengruppe*
    - **Standort:** *Ein beliebiger verfügbarer Speicherort*
    - **Tarif:** Free

1. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Ressource.
1. Überprüfen Sie die Seite **Übersicht** auf dem Blatt für Ihre Azure KI-Suche-Ressource im Azure-Portal. Hier können Sie eine grafische Benutzeroberfläche verwenden, um die verschiedenen Komponenten einer Suchlösung, z. B. Datenquellen, Indizes, Indexer und Skillsets, zu erstellen, zu testen, zu verwalten und zu überwachen.

### Erstellen eines Speicherkontos

1. Kehren Sie zur Startseite zurück, und erstellen Sie anschließend eine Ressource vom Typ **Speicherkonto** mit den folgenden Einstellungen:
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe:** *Die gleiche Ressourcengruppe wie die Ihrer Ressourcen für Azure KI-Suche und Azure KI Services*
    - **Speicherkontoname:** *Ein gültiger Name für die Speicherressource*
    - **Region:** *Die gleiche Region wie Ihre Azure KI-Suche-Ressource*
    - **Primärer Dienst**: Azure Blob Storage oder Azure Data Lake Storage Gen 2
    - **Leistung**: Standard
    - **Redundanz**: Lokal redundanter Speicher (LRS)

1. Warten Sie, bis die Bereitstellung abgeschlossen ist, und wechseln Sie dann zur bereitgestellten Ressource.

    > **Tipp**: Lassen Sie die Portalseite für Ihr Speicherkonto geöffnet, da Sie diese im nächsten Schritt benötigen.

## Hochladen von Dokumenten in Azure Storage

Ihre Knowledge Mining-Lösung extrahiert Informationen aus Reiseprospektdokumenten in einem Azure Storage-Blobcontainer.

1. Laden Sie auf einer neuen Browserregisterkarte die Datei [documents.zip](https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip) von `https://github.com/microsoftlearning/mslearn-ai-information-extraction/raw/main/Labfiles/knowledge/documents.zip` herunter, und speichern Sie sie in einem lokalen Ordner.
1. Extrahieren Sie die heruntergeladene Datei *documents.zip*, und zeigen Sie die darin enthaltenen Reiseprospektdateien an. Aus diesen Dateien werden Sie Informationen extrahieren und indizieren.
1. Wählen Sie auf der Browserregisterkarte mit der Azure-Portalseite für Ihr Speicherkonto im Navigationsbereich auf der linken Seite die Option **Speicherbrowser** aus.
1. Wählen Sie im Speicherbrowser die Option **Blobcontainer** aus.

    Derzeit sollte Ihr Speicherkonto nur den Standardcontainer **$logs** enthalten.

1. Wählen Sie auf der Symbolleiste die Option **+ Container** aus, und erstellen Sie einen neuen Container mit den folgenden Einstellungen:
    - **Name**: `documents`
    - **Anonyme Zugriffsebene**: Privat (kein anonymer Zugriff)\*

    > **Hinweis:** \*Sofern Sie beim Erstellen Ihres Speicherkontos die Option zum Zulassen des anonymen Zugriffs auf Container nicht aktiviert haben, können Sie keine andere Einstellung auswählen!

1. Wählen Sie den Container **documents** aus, um ihn zu öffnen. Laden Sie anschließend über die Schaltfläche **Hochladen** auf der Symbolleiste die PDF-Dateien hoch, die Sie zuvor aus **documents.zip** in das Stammverzeichnis des Containers extrahiert haben:

    ![Screenshot: Azure Storage-Browser mit dem Container „documents“ und dessen Dateiinhalt](./media/blob-containers.png)

## Erstellen und Ausführen eines Indexers

Nachdem sich die Dokumente nun am gewünschten Ort befinden, können Sie einen Indexer erstellen, um Informationen daraus zu extrahieren.

1. Navigieren Sie im Azure-Portal zu Ihrer Azure KI-Suche-Ressource. Wählen Sie dann auf der Seite **Übersicht** der Ressource **Daten importieren** aus.

    ![Screenshot: Azure Search-Diensts mit Hervorhebung von „Importieren von Daten“](./media/overview-panel.png)
    
    > **Hinweis:** Auf der Seite **Übersicht** Ihrer Ressource von Azure KI-Suche bietet die Symbolleiste zwei Optionen:  
    > - **Importieren von Daten** (klassische Benutzeroberfläche)  
    > - **Importieren von Daten (neu)** (neue Benutzeroberfläche)  
    >  
    > Darüber hinaus enthält das Panel **Erste Schritte** unterhalb des Abschnitts „Übersicht“ die Schaltfläche **Importieren**, die Sie zur **neuen Benutzeroberfläche** weiterleitet.  
    >  
    > Die Anweisungen in diesem Kurs beziehen sich auf den **klassischen Workflow zum Importieren von Daten**. Um Verwirrung zu vermeiden, stellen Sie sicher, dass Sie auf der Symbolleiste die Option **Importieren von Daten** auswählen.
1. Wählen Sie auf der Seite **Mit Ihren Daten verbinden** in der Liste **Datenquelle** die Option **Azure Blob Storage** aus. Vervollständigen Sie dann die Datenspeicherdetails mit den folgenden Werten:
    - **Datenquelle**: Azure Blob Storage
    - **Datenquellenname**: `margies-documents`
    - **Zu extrahierende Daten**: Inhalt und Metadaten
    - **Analysemodus**: Standard
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Verbindungszeichenfolge:** 
        - Wählen Sie die Option **Vorhandene Verbindung auswählen** aus.
        - Wählen Sie Ihr Speicherkonto aus.
        - Wählen Sie den Container **documents** aus.
    - **Authentifizierung mithilfe verwalteter Identitäten**: Keine
    - **Containername**: documents
    - **Blobordner:** *Lassen Sie dieses Feld leer.*
    - **Beschreibung:** `Travel brochures`
1. Fahren Sie mit dem nächsten Schritt fort (**Kognitive Fähigkeiten hinzufügen**), der drei erweiterbare Abschnitte umfasst.
1. Wählen Sie im Abschnitt **Azure KI Services anfügen** die Option **Free (begrenzte Anreicherung**)\* aus.

    > **Hinweis:** \*Die kostenlose Azure KI Services-Ressource für die Azure KI-Suche kann zur Indizierung von maximal 20 Dokumenten verwendet werden. In einer echten Lösung sollten Sie eine Azure KI Services-Ressource in Ihrem Abonnement erstellen, um die KI-Anreicherung für eine größere Anzahl von Dokumenten zu ermöglichen.

1. Gehen Sie im Abschnitt **Anreicherungen hinzufügen** folgendermaßen vor:
    - Ändern Sie den **Namen des Skillsets** in `margies-skillset`.
    - Wählen Sie die Option **OCR aktivieren und den gesamten Text im Feld „merged_content“ zusammenführen** aus.
    - Vergewissern Sie sich, dass **Quelldatenfeld** auf **merged_content** festgelegt ist.
    - Übernehmen Sie **Granularitätsstufe für die Anreicherung** als **Quellfeld**, sodass der gesamte Inhalt des Dokuments indiziert wird. Sie können diese Einstellung jedoch ändern, um Informationen auf präziseren Ebenen wie Seiten oder Sätzen zu extrahieren.
    - Wählen Sie die folgenden angereicherten Felder aus:

        | Kognitive Fähigkeit | Parameter | Feldname |
        | --------------- | ---------- | ---------- |
        | **Kognitive Fähigkeiten für Text** | |  |
        | Personennamen extrahieren | | Personen |
        | Ortsnamen extrahieren | | locations |
        | Schlüsselbegriffe extrahieren | | keyphrases |
        | **Kognitive Fähigkeiten für Bild** | |  |
        | Tags aus Bildern generieren | | imageTags |
        | Untertitel aus Bildern generieren | | imageCaption |

        Überprüfen Sie Ihre Auswahl genau (es kann schwierig sein, sie später zu ändern).

1. Führen Sie im Abschnitt **Anreicherungen in einem Wissensspeicher speichern** folgende Aktionen aus:
    - Aktivieren Sie nur die folgenden Kontrollkästchen (es wird ein <font color="red">Fehler</font> angezeigt, den Sie später beheben):
        - **Azure-Dateiprojektionen**:
            - Bildprojektionen
        - **Azure-Tabellenprojektionen**:
            - Dokumente
                - Schlüsselphrasen
        - **Azure-Blobprojektionen**:
            - Document
    - Führen Sie unter **Verbindungszeichenfolge für das Speicherkonto** (unterhalb von <font color="red">Fehlermeldungen</font>) folgende Aktionen aus:
        - Wählen Sie die Option **Vorhandene Verbindung auswählen** aus.
        - Wählen Sie Ihr Speicherkonto aus.
        - Wählen Sie den Container **documents** aus (*dies ist nur erforderlich, um das Speicherkonto in der Navigationsoberfläche auszuwählen – für die extrahierten Wissensressourcen geben Sie einen anderen Containernamen an*).
    - Ändern Sie den **Containernamen** in `knowledge-store`.
1. Fahren Sie mit dem nächsten Schritt fort (**Zielindex anpassen**), bei dem Sie die Felder für Ihren Index angeben. 
1. Ändern Sie den **Indexnamen** in `margies-index`.
1. Stellen Sie sicher, dass der **Schlüssel** auf **metadata_storage_path** festgelegt ist, lassen Sie den **Namen der Vorschlagsfunktion** leer, und vergewissern Sie sich, dass der **Suchmodus** auf **analyzingInfixMatching** festgelegt ist.
1. Nehmen Sie die folgenden Änderungen an den Indexfeldern vor, und übernehmen Sie für alle anderen Felder deren Standardeinstellungen (**WICHTIG**: Sie müssen möglicherweise nach rechts scrollen, um die ganze Tabelle anzuzeigen.):

    | Feldname | Abrufbar | Filterbar | Sortierbar | In Facets einteilbar | Suchbar |
    | ---------- | ----------- | ---------- | -------- | --------- | ---------- |
    | metadata_storage_size | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_last_modified | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | |
    | metadata_storage_name | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | locations | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | Personen | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |
    | keyphrases | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; | | | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#10004; |

    Überprüfen Sie Ihre Auswahl, und achten Sie besonders darauf, dass für jedes Feld die richtigen Optionen für **Abrufbar**, **Filterbar**, **Sortierbar**, **Facettierbar** und **Suchbar** ausgewählt sind, da diese später schwer geändert werden können.

1. Fahren Sie mit dem nächsten Schritt fort (**Indexer erstellen**), bei dem Sie den Indexer erstellen und planen.
1. Ändern Sie den **Indexernamen** in `margies-indexer`.
1. Lassen Sie **Zeitplan** auf **Einmal** festgelegt.
1. Wählen Sie **Senden** aus, um Datenquelle, Skillset, Index und Indexer zu erstellen. Der Indexer wird automatisch ausgeführt und führt die Indizierungspipeline aus. Dieser führt folgende Aufgaben aus:
    - Extrahieren der Felder und Inhalte von Dokumentmetadaten aus der Datenquelle
    - Ausführen des Skillset für kognitive Skills aus, um zusätzliche angereicherte Felder zu generieren
    - Zuordnen der extrahierten Felder zum Index
    - Speichert die extrahierten Datenressourcen im Wissensspeicher.
1. Zeigen Sie im Navigationsbereich auf der linken Seite unter **Suchverwaltung** die Seite **Indexer** an, auf der der neu erstellte Indexer **margies-indexer** angezeigt werden sollen. Warten Sie einige Minuten, und klicken Sie auf **&#8635; Aktualisieren**, bis der **Status** **Erfolgreich** angezeigt wird.

## Durchsuchen des Index

Nachdem Sie nun über einen Index verfügen, können Sie ihn durchsuchen.

1. Kehren Sie zur Seite **Übersicht** Ihrer Ressource für Azure KI-Suche zurück, und wählen Sie auf der Symbolleiste die Option **Suchexplorer** aus.
1. Geben Sie im Suchexplorer im Feld **Abfragezeichenfolge** `*` (ein einzelnes Sternchen) ein, und wählen Sie dann **Suchen** aus.

    Diese Abfrage ruft alle Dokumente im Index im JSON-Format ab. Überprüfen Sie die Ergebnisse, und notieren Sie sich die Felder für jedes Dokument, die Dokumentinhalte, Metadaten und angereicherte Daten enthalten, die von den ausgewählten kognitiven Skills extrahiert wurden.

1. Wählen Sie im Menü **Ansicht** die **JSON-Ansicht** aus, und beachten Sie, dass die JSON-Anforderung für die Suche wie folgt angezeigt wird:

    ```json
    {
      "search": "*",
      "count": true
    }
    ```

1. Die Ergebnisse enthalten oben ein **@odata.count**-Feld, das die Anzahl der von der Suche zurückgegebenen Dokumente angibt.

1. Ändern Sie die JSON-Anforderung wie folgt, sodass sie den Parameter **select** enthält:

    ```json
    {
      "search": "*",
      "count": true,
      "select": "metadata_storage_name,locations"
    }
    ```

    Dieses Mal enthalten die Ergebnisse nur den Dateinamen und alle Orte, die im Dokumentinhalt erwähnt werden. Der Dateiname befindet sich im Feld **metadata_storage_name**, das aus dem Quelldokument extrahiert wurde. Das Feld **locations** wurde von einer KI-Fähigkeit generiert.

1. Probieren Sie nun die folgende Abfragezeichenfolge aus:

    ```json
    {
      "search": "New York",
      "count": true,
      "select": "metadata_storage_name,keyphrases"
    }
    ```

    Diese Suche findet Dokumente, die „New York“ in einem der durchsuchbaren Felder erwähnen, und gibt den Dateinamen und Schlüsselbegriffe im Dokument zurück.

1. Probieren wir eine weitere Abfrage aus:

    ```json
    {
        "search": "New York",
        "count": true,
        "select": "metadata_storage_name,keyphrases",
        "filter": "metadata_storage_size lt 380000"
    }
    ```

    Diese Abfrage gibt den Dateinamen und die Schlüsselbegriffe für alle Dokumente zurück, in denen „New York“ erwähnt wird und die kleiner als 380.000 Bytes sind.

## Erstellen einer Suchclientanwendung

Nachdem Sie nun über einen brauchbaren Index verfügen, können Sie ihn in einer Clientanwendung verwenden. Sie können dafür die REST-Schnittstelle in Anspruch nehmen, um Anforderungen zu übermitteln und Antworten im JSON-Format über HTTP zu empfangen. Sie können auch das Software Development Kit (SDK) für Ihre bevorzugte Programmiersprache verwenden. In dieser Übung wird das SDK verwendet.

> **Hinweis**: Sie können auswählen, ob Sie das SDK für **C#** oder **Python** verwenden möchten. Führen Sie in den folgenden Schritten die entsprechenden Aktionen für Ihre bevorzugte Sprache aus.

### Abrufen des Endpunkts und der Schlüssel für Ihre Suchressource

1. Schließen Sie im Azure-Portal die Suchexplorer-Seite, und kehren Sie zur Seite **Übersicht** Ihrer Ressource für Azure KI-Suche zurück.

    Notieren Sie sich den **URL-Wert**, der ähnlich wie **https://*Name_Ihrer_Ressource*.search.windows.net** aussehen sollte. Dies ist der Endpunkt für Ihre Suchressource.

1. Erweitern Sie im Navigationsbereich auf der linken Seite die Option **Einstellungen**, und zeigen Sie die Seite **Schlüssel** an.

    Beachten Sie, dass zwei **Administratorschlüssel** und ein einzelner **Abfrageschlüssel** vorhanden sind. Ein *Administratorschlüssel* wird zum Erstellen und Verwalten von Suchressourcen verwendet. Ein *Abfrageschlüssel* wird von Clientanwendungen verwendet, die nur Suchabfragen ausführen müssen.

    *Sie benötigen den Endpunkt und den **Abfrageschlüssel** für Ihre Clientanwendung.*

### Vorbereiten der Verwendung des Azure KI-Suche-SDKs

1. Verwenden Sie die Schaltfläche **[\>_]** rechts neben der Suchleiste oben auf der Seite des Azure-Portals, um eine neue Cloud Shell im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung ohne Speicher in Ihrem Abonnement aus.

    Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Fenster am unteren Rand des Azure-Portals. Sie können die Größe dieses Bereichs ändern oder maximieren, um die Arbeit zu vereinfachen. Zunächst benötigen Sie sowohl die Cloud Shell als auch das Azure-Portal (damit Sie den erforderlichen Endpunkt und Schlüssel suchen und kopieren können).

1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** das Menüelement **Zur klassischen Version wechseln** aus (dies ist für die Verwendung des Code-Editors erforderlich).

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im Cloud Shell-Bereich die folgenden Befehle ein, um das GitHub-Repository zu klonen, das die Codedateien für diese Übung enthält (geben Sie den Befehl ein, oder kopieren Sie ihn in die Zwischenablage, und klicken Sie dann mit der rechten Maustaste in die Befehlszeile, und fügen Sie ihn als Nur-Text ein):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Tipp**: Wenn Sie Befehle in die Cloud Shell einfügen, kann die Ausgabe einen großen Teil des Bildschirmpuffers in Anspruch nehmen. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

1. Navigieren Sie nach dem Klonen des Repositorys zu dem Ordner, der die Codedateien der Anwendung enthält:

    ```
   cd mslearn-ai-info/Labfiles/knowledge/python
   ls -a -l
    ```

1. Führen Sie die folgenden Befehle aus, um das SDK für die Azure KI-Suche und die Azure-Identitätspakete zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-search-documents==11.5.1
    ```

1. Führen Sie den folgenden Befehl aus, um die Konfigurationsdatei für Ihre App zu bearbeiten:

    ```
   code .env
    ```

    Die Konfigurationsdatei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Konfigurationsdatei die folgenden Platzhalterwerte:

    - **your_search_endpoint** (*ersetzen Sie diesen durch den Endpunkt Ihrer Ressource für Azure KI-Suche*)
    - **your_query_key** (*ersetzen Sie diesen durch den Abfrageschlüssel Ihrer Ressource für Azure KI-Suche*)
    - **your_index_name** (*ersetzen Sie diesen durch den Namen Ihres Indexes, der `margies-index` lauten sollte*)

1. Wenn Sie die Platzhalter aktualisiert haben, speichern Sie die Datei (**STRG+S**), und schließen Sie sie anschließend (**STRG+Q**).

    > **Tipp**: Nachdem Sie den Endpunkt und den Schlüssel aus dem Azure-Portal kopiert haben, können Sie den Cloud Shell-Bereich maximieren, um die Arbeit darin zu vereinfachen.

1. Führen Sie den folgenden Befehl aus, um die Codedatei für Ihre App zu öffnen:

    ```
   code search-app.py
    ```

    Die Codedatei wird in einem Code-Editor geöffnet.

1. Überprüfen Sie den Code, und beachten Sie, dass er die folgenden Aktionen ausführt:

    - Ruft die Konfigurationseinstellungen für Ihre Ressource für Azure KI-Suche und den Index aus der von Ihnen bearbeiteten Konfigurationsdatei ab.
    - Erstellt einen **SearchClient** mit dem Endpunkt, Schlüssel und Indexnamen, der mit Ihrem Suchdienst verbunden werden soll.
    - Fordert den Benutzer zur Eingabe einer Suchabfrage auf (bis dieser „Quit“ eingibt).
    - Durchsucht den Index mithilfe der Abfrage und gibt die folgenden Felder zurück (sortiert nach „metadata_storage_name“):
        - metadata_storage_name
        - locations
        - Personen
        - keyphrases
    - Analysiert die zurückgegebenen Suchergebnisse, um die Felder anzuzeigen, die für jedes Dokument im Resultset zurückgegeben wurden.

1. Schließen Sie den Code-Editor-Bereich (*STRG+Q*), und lassen Sie den Befehlszeilen-Konsolenbereich der Cloud Shell geöffnet.
1. Geben Sie den folgenden Befehl ein, um die App auszuführen:

    ```
   python search-app.py
    ```

1. Wenn Sie dazu aufgefordert werden, geben Sie eine Abfrage ein (z. B. `London`), und zeigen Sie die Ergebnisse an.
1. Probieren Sie eine weitere Abfrage aus (z. B. `flights`).
1. Wenn Sie mit dem Testen der App fertig sind, geben Sie `quit` ein, um die App zu schließen.
1. Schließen Sie die Cloud Shell, und kehren Sie zum Azure-Portal zurück.

## Anzeigen des Wissensspeichers

Nachdem Sie einen Indexer ausgeführt haben, der ein Skillset zur Erstellung eines Wissensspeichers verwendet, werden die durch den Indizierungsprozess extrahierten angereicherten Daten in den Projektionen des Wissensspeichers dauerhaft gespeichert.

### Anzeigen von Objektprojektionen

Die im Skillset „Margie's Travel“ definierten *Objekt*-Projektionen bestehen aus einer JSON-Datei für jedes indizierte Dokument. Diese Dateien werden in einem BLOB-Container in dem in der Skillset-Definition angegebenen Azure Storage-Konto gespeichert.

1. Zeigen Sie im Azure-Portal das Azure Storage-Konto an, das Sie zuvor erstellt haben.
1. Wählen Sie die Registerkarte **Speicherbrowser** aus (im linken Bereich), um das Speicherkonto auf der Speicher-Explorer-Oberfläche im Azure-Portal anzuzeigen.
1. Erweitern Sie **BLOB-Container**, um die Container im Speicherkonto anzuzeigen. Neben dem Container **documents**, in dem die Quelldaten gespeichert sind, sollten zwei neue Container vorhanden sein: **knowledge-store** und **margies-skillset-image-projection**. Diese wurden durch den Indizierungsprozess erzeugt.
1. Wählen Sie den Container **knowledge-store** aus. Er sollte für jedes indizierte Dokument einen Ordner enthalten.
1. Öffnen Sie einen beliebigen Ordner, und wählen Sie dann die darin enthaltene Datei **objectprojection.json** aus. Verwenden Sie die Schaltfläche **Herunterladen** auf der Symbolleiste, um die Datei herunterzuladen und zu öffnen. Jede JSON-Datei enthält eine Darstellung eines indizierten Dokuments, einschließlich der angereicherten Daten, die, wie hier gezeigt, durch das Skillset extrahiert wurden (für eine bessere Lesbarkeit wurden sie formatiert).

    ```json
    {
        "metadata_storage_content_type": "application/pdf",
        "metadata_storage_size": 388622,
        "<more_metadata_fields>": "...",
        "key_phrases":[
            "Margie’s Travel",
            "Margie's Travel",
            "best travel experts",
            "world-leading travel agency",
            "international reach"
            ],
        "locations":[
            "Dubai",
            "Las Vegas",
            "London",
            "New York",
            "San Francisco"
            ],
        "image_tags":[
            "outdoor",
            "tree",
            "plant",
            "palm"
            ],
        "more fields": "..."
    }
    ```

Durch die Möglichkeit, *Objektprojektionen* wie diese zu erstellen, können Sie angereicherte Datenobjekte generieren, die in eine Analyselösung für Unternehmensdaten integriert werden können.

### Anzeigen von Datei-Projektionen

Die im Skillset definierten *Datei* Projektionen erstellen JPEG-Dateien für jedes Bild, das während des Indizierungsprozesses aus den Dokumenten extrahiert wurde.

1. Wählen Sie in der Benutzeroberfläche *Speicherbrowser* im Azure-Portal den Blobcontainer **margies-skillset-image-projection** aus. Dieser Container enthält einen Ordner für jedes Dokument, das Bilder enthält.
2. Öffnen Sie einen der Ordner, und zeigen Sie den Inhalt an – jeder Ordner enthält mindestens eine \*JPG-Datei.
3. Öffnen Sie eine beliebige Bilddatei, und laden Sie sie herunter. Zeigen Sie diese an, um das Bild anzuzeigen.

Die Möglichkeit, *Datei*-Projektionen wie diese zu generieren, macht die Indizierung zu einer effizienten Methode, um eingebettete Bilder aus einer großen Menge von Dokumenten zu extrahieren.

### Anzeigen von Tabellen-Projektionen

Die im Skillset definierten *Tabellen*-Projektionen bilden ein relationales Schema angereicherter Daten.

1. Erweitern Sie in der Oberfläche *Speicherbrowser* im Azure-Portal die Option **Tabellen**.
2. Wählen Sie die Tabelle **margiesSkillsetDocument** aus, um Daten anzuzeigen. Diese Tabelle enthält eine Zeile für jedes Dokument, das indiziert wurde:
3. Zeigen Sie die Tabelle **margiesSkillsetKeyPhrases** an, die eine Zeile für jeden Schlüsselbegriff enthält, der aus den Dokumenten extrahiert wurde.

Durch die Möglichkeit, *Tabellenprojektionen* zu erstellen, können Sie Analyse- und Berichterstellungslösungen erstellen, die ein relationales Schema abfragen. Die automatisch generierten Schlüsselspalten können verwendet werden, um die Tabellen in Abfragen zu verknüpfen, um z. B. alle aus einem bestimmten Dokument extrahierten Schlüsselbegriffe zurückzugeben.

## Bereinigung

Nachdem Sie die Übung abgeschlossen haben, löschen Sie alle nicht länger benötigten Ressourcen. Löschen der Azure-Ressourcen:

1. Wählen Sie im **Azure-Portal** die Option „Ressourcengruppen“ aus.
1. Wählen Sie die Ressourcengruppe aus, die Sie nicht benötigen, und wählen Sie dann **Ressourcengruppe löschen** aus.

## Weitere Informationen

Weitere Informationen zu Azure KI-Suche finden Sie in der [Azure KI-Suche-Dokumentation](https://docs.microsoft.com/azure/search/search-what-is-azure-search).
