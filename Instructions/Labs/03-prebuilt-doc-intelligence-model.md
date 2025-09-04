---
lab:
  title: Analysieren von Formularen mit vorgefertigten Azure KI Dokument Intelligenz-Modellen
  description: 'Verwenden Sie vordefinierte Azure KI Dokument Intelligenz-Modelle, um Textfelder aus Dokumenten zu verarbeiten.'
---

# Analysieren von Formularen mit vorgefertigten Azure KI Dokument Intelligenz-Modellen

In dieser Übung richten Sie ein Azure AI Foundry-Projekt mit allen erforderlichen Ressourcen für die Dokumentanalyse ein. Sie verwenden sowohl das Azure AI Foundry-Portal als auch das Python-SDK, um Formulare zur Analyse an diese Ressource zu übermitteln.

Obwohl diese Übung auf Python basiert, können Sie ähnliche Anwendungen mit mehreren sprachspezifischen SDKs entwickeln, einschließlich:

- [Azure KI Dokument Intelligenz-Clientbibliothek für Python](https://pypi.org/project/azure-ai-formrecognizer/)
- [Azure KI Dokument Intelligenz-Clientbibliothek für Microsoft .NET](https://www.nuget.org/packages/Azure.AI.FormRecognizer)
- [Azure KI Dokument Intelligenz-Clientbibliothek für JavaScript](https://www.npmjs.com/package/@azure/ai-form-recognizer)

Diese Übung dauert ca. **30** Minuten.

## Erstellen eines Azure KI Foundry-Projekts

Beginnen wir mit dem Erstellen eines Azure AI Foundry-Projekts.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipps oder Schnellstartfenster, die bei der ersten Anmeldung geöffnet werden, und verwenden Sie gegebenenfalls das Logo **Azure AI Foundry** oben links, um zur Startseite zu navigieren, die ähnlich wie die folgende Abbildung aussieht (schließen Sie das **Hilfe**-Fenster, falls es geöffnet ist):

    ![Screenshot des Azure KI Foundry-Portals.](./media/ai-foundry-home.png)

1. Navigieren Sie im Browser zu `https://ai.azure.com/managementCenter/allResources`, und wählen Sie **Neu erstellen** aus. Wählen Sie dann die Option zum Erstellen einer neuen **KI-Hubressource** aus.
1. Geben Sie im Assistenten zum **Erstellen eines Projekts** einen gültigen Namen für Ihr Projekt ein und wählen Sie die Option zum Erstellen eines neuen Hubs aus. Verwenden Sie anschließend den Link **Hub umbenennen**, um einen gültigen Namen für Ihren neuen Hub anzugeben, erweitern Sie **Erweiterte Optionen** und legen Sie die folgenden Einstellungen für Ihr Projekt fest:
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Region:**  *Eine beliebige verfügbare Region*

    > **Hinweis**: Wenn Sie in einem Azure-Abonnement arbeiten, in dem Richtlinien zur Einschränkung zulässiger Ressourcennamen verwendet werden, müssen Sie möglicherweise den Link unten im Dialogfeld **Neues Projekt erstellen** verwenden, um den Hub über das Azure-Portal zu erstellen.

    > **Hinweis**: Wenn die Schaltfläche **Erstellen** weiterhin deaktiviert ist, benennen Sie Ihren Hub bitte in einen eindeutigen alphanumerischen Wert um.

1. Warten Sie, bis Ihr Projekt erstellt wurde.
1. Sobald Ihr Projekt erstellt wurde, schließen Sie alle angezeigten Tipps und überprüfen Sie die Projektseite im Azure AI Foundry-Portal, die in etwa wie in der folgenden Abbildung aussehen sollte:

    ![Screenshot eines Azure KI-Projekts im Azure AI Foundry-Portal.](./media/ai-foundry-project.png)

## Verwenden des Lesemodells

Beginnen wir mit dem **Azure AI Foundry-Portal** und dem Modell „Lesen“, um ein Dokument in mehreren Sprachen zu analysieren.

1. Wählen Sie im Navigationsbereich auf der linken Seite **KI Services** aus.
1. Wählen Sie auf der Seite **Azure KI Services** die Kachel **Vision + Dokument** aus.
1. Überprüfen Sie auf der Seite **Vision + Dokument**, ob die Registerkarte **Dokument** ausgewählt ist, und wählen Sie dann die Kachel **OCR/Lesen** aus.

    Auf der Seite **Lesen** sollte die mit Ihrem Projekt erstellte Azure KI Services-Ressource bereits verbunden sein.

1. Wählen Sie links in der Liste der Dokumente **read-german.pdf** aus.

    ![Screenshot: Seite „Lesen“ in Azure KI Dokument Intelligenz-Studio.](./media/read-german-sample.png#lightbox)

1. Wählen Sie in der oberen Symbolleiste **Analyseoptionen** aus, aktivieren Sie das Kontrollkästchen **Sprache** (unter **Optionale Erkennung**) im Bereich **Analyseoptionen**, und wählen Sie **Speichern** aus. 
1. Wählen Sie links oben **Analyse ausführen** aus.
1. Nach Abschluss der Analyse wird der aus dem Bild extrahierte Text rechts auf der Registerkarte **Inhalt** angezeigt. Überprüfen Sie diesen Text, und vergleichen Sie ihn mit dem Text im Originalbild auf Richtigkeit.
1. Wählen Sie die Registerkarte **Ergebnis** aus, auf der der extrahierte JSON-Code angezeigt wird. 

## Vorbereiten der Entwicklung einer App in Cloud Shell

Sehen wir uns nun die App an, die das Dienst-SDK von Azure KI Dokument Intelligenz verwendet. Sie entwickeln Ihre App mit Cloud Shell. Die Codedateien für Ihre App wurden in einem GitHub-Repository bereitgestellt.

Dies ist die Rechnung, die Ihr Code analysieren wird.

![Screenshot eines Beispielrechnungsdokuments.](./media/sample-invoice.png)

1. Wechseln Sie im Azure AI Foundry-Portal zur **Übersichtsseite** Ihres Projekts.
1. Wählen Sie im Bereich **Endpunkte und Schlüssel** die Registerkarte **Azure KI Services** aus, und notieren Sie sich den **API-Schlüssel** und **Azure KI Services-Endpunkt**. Sie verwenden diese Anmeldeinformationen, um eine Verbindung mit Ihrer Azure KI Services-Instanz in einer Clientanwendung herzustellen.
1. Öffnen Sie eine neue Browserregisterkarte (wobei das Azure AI Foundry-Portal auf der vorhandenen Registerkarte geöffnet bleibt). Wechseln Sie dann in der neuen Registerkarte zum [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldedaten an, wenn Sie dazu aufgefordert werden.
1. Verwenden Sie die Taste **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung aus. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals.

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** das Menüelement **Zur klassischen Version wechseln** aus (dies ist für die Verwendung des Code-Editors erforderlich).

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um das GitHub-Repository für diese Übung zu klonen:

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Tipp**: Wenn Sie Befehle in die Cloudshell einfügen, kann die Ausgabe einen großen Teil des Bildschirmpuffers einnehmen. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

    ***Folgen Sie nun den Schritten für die von Ihnen gewählte Programmiersprache.***

1. Nachdem das Repository geklont wurde, navigieren Sie zu dem Ordner, der die Codedateien enthält:

    ```
   cd mslearn-ai-info/Labfiles/prebuilt-doc-intelligence/Python
    ```

1. Geben Sie im Befehlszeilenbereich von Cloud Shell den folgenden Befehl ein, um die Bibliotheken zu installieren, die Sie verwenden möchten:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-formrecognizer==3.3.3
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei die Platzhalter **YOUR_ENDPOINT** und **YOUR_KEY** durch Ihren Azure KI Services-Endpunkt und dessen API-Schlüssel (aus dem Azure KI Foundry-Portal kopiert).
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, um Ihre Änderungen zu speichern und dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

## Hinzufügen von Code zur Verwendung des Azure KI Dokument Intelligenz-Diensts

Jetzt können Sie das SDK verwenden, um die PDF-Datei auszuwerten.

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte App-Datei zu bearbeiten:

    ```
   code document-analysis.py
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Suchen Sie in der Codedatei den Kommentar **Import the required libraries**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Add references
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.formrecognizer import DocumentAnalysisClient
    ```

1. Suchen Sie den Kommentar **Create the client**, und fügen Sie den folgenden Code hinzu (achten Sie darauf, die richtige Einzugsebene beizubehalten):

    ```python
   # Create the client
   document_analysis_client = DocumentAnalysisClient(
        endpoint=endpoint, credential=AzureKeyCredential(key)
   )
    ```

1. Suchen Sie den Kommentar **Analyze the invoice**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Analyse the invoice
   poller = document_analysis_client.begin_analyze_document_from_url(
        fileModelId, fileUri, locale=fileLocale
   )
    ```

1. Suchen Sie nach dem Kommentar **Display invoice information to the user**, und fügen Sie den folgenden Code hinzu:

    ```python
   # Display invoice information to the user
   receipts = poller.result()
    
   for idx, receipt in enumerate(receipts.documents):
    
        vendor_name = receipt.fields.get("VendorName")
        if vendor_name:
            print(f"\nVendor Name: {vendor_name.value}, with confidence {vendor_name.confidence}.")

        customer_name = receipt.fields.get("CustomerName")
        if customer_name:
            print(f"Customer Name: '{customer_name.value}, with confidence {customer_name.confidence}.")


        invoice_total = receipt.fields.get("InvoiceTotal")
        if invoice_total:
            print(f"Invoice Total: '{invoice_total.value.symbol}{invoice_total.value.amount}, with confidence {invoice_total.confidence}.")
    ```

1. Drücken Sie im Code-Editor **STRG+S**, oder verwenden Sie **Rechtsklick > Speichern**, um die Änderungen zu speichern. Lassen Sie den Code-Editor geöffnet, falls Sie Fehler im Code beheben müssen, aber ändern Sie die Größe der Bereiche, damit der Befehlszeilenbereich gut sichtbar ist.

1. Geben Sie im Befehlszeilenbereich den folgenden Befehl ein, um die Anwendung auszuführen.

    ```
    python document-analysis.py
    ```

Das Programm zeigt den Namen des Lieferanten und Kunden sowie die Gesamtsumme der Rechnung mit entsprechendem Konfidenzniveau an. Vergleichen Sie die gemeldeten Werte mit der Beispielrechnung, die Sie am Anfang dieses Abschnitts geöffnet haben.

## Bereinigung

Wenn Sie mit Ihrer Azure-Ressource fertig sind, denken Sie daran, die Ressource im [Azure-Portal](https://portal.azure.com) (`https://portal.azure.com`) zu löschen, um zukünftige Gebühren zu vermeiden.
