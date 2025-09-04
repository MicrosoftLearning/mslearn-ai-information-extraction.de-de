---
lab:
  title: Entwickeln Sie eine Client-Anwendung zum Verstehen von Inhalten.
  description: "Verwenden der REST-API für das Verständnis von Azure\_AI-Inhalten zur Entwicklung einer Client-App für ein Analysetool"
---

# Entwickeln Sie eine Client-Anwendung zum Verstehen von Inhalten.

In dieser Übung nutzen Sie das Verständnis von Azure AI-Inhalten, um ein Analysetool zu erstellen, mit dem Sie Informationen aus Visitenkarten extrahieren. Anschließend entwickeln Sie eine Clientanwendung, die mithilfe des Analysetools Kontaktdetails aus gescannten Visitenkarten extrahiert.

Diese Übung dauert ca. **30** Minuten.

## Erstellen eines Azure AI Foundry-Hubs und -Projekts

Die Funktionen von Azure KI-Foundry, die wir in dieser Übung verwenden werden, erfordern ein Projekt, das auf einer Azure KI-Foundry-*Hub*-Ressource basiert.

1. Öffnen Sie in einem Webbrowser unter `https://ai.azure.com` das [Azure KI Foundry-Portal](https://ai.azure.com) und melden Sie sich mit Ihren Azure-Anmeldeinformationen an. Schließen Sie alle Tipps oder Schnellstartfenster, die bei der ersten Anmeldung geöffnet werden, und verwenden Sie gegebenenfalls das Logo **Azure AI Foundry** oben links, um zur Startseite zu navigieren, die ähnlich wie die folgende Abbildung aussieht (schließen Sie das **Hilfe**-Fenster, falls es geöffnet ist):

    ![Screenshot des Azure KI Foundry-Portals.](./media/ai-foundry-home.png)

1. Navigieren Sie im Browser zu `https://ai.azure.com/managementCenter/allResources`, und wählen Sie **Neu erstellen** aus. Wählen Sie dann die Option zum Erstellen einer neuen **KI-Hubressource** aus.
1. Geben Sie im Assistenten zum **Erstellen eines Projekts** einen gültigen Namen für Ihr Projekt ein und wählen Sie die Option zum Erstellen eines neuen Hubs aus. Verwenden Sie anschließend den Link **Hub umbenennen**, um einen gültigen Namen für Ihren neuen Hub anzugeben, erweitern Sie **Erweiterte Optionen** und legen Sie die folgenden Einstellungen für Ihr Projekt fest:
    - **Abonnement:** *Geben Sie Ihr Azure-Abonnement an.*
    - **Ressourcengruppe**: *Erstellen Sie eine Ressourcengruppe, oder wählen Sie eine Ressourcengruppe aus*.
    - **Hubname**: Geben Sie einen gültigen Namen für Ihren Hub an
    - **Standort:** Wählen Sie einen der folgenden Standorte aus:\*
        - Australien (Osten)
        - Schweden, Mitte
        - USA (Westen)

    > \*Zum Zeitpunkt der Erstellung dieses Artikels ist das Verständnis von Azure AI-Inhalten nur in diesen Regionen verfügbar.

    > **Hinweis**: Wenn die Schaltfläche **Erstellen** weiterhin deaktiviert ist, benennen Sie Ihren Hub bitte in einen eindeutigen alphanumerischen Wert um.

1. Warten Sie, bis Ihr Projekt erstellt wurde, und navigieren Sie dann zur Projektübersichtsseite.

## Verwenden der REST-API zum Erstellen eines Analysetools für Inhaltsverständnis

Sie werden mithilfe der REST-API ein Analysetool erstellen, das Informationen aus Bildern von Visitenkarten extrahieren kann.

1. Öffnen Sie eine neue Browserregisterkarte (wobei das Azure AI Foundry-Portal auf der vorhandenen Registerkarte geöffnet bleibt). Wechseln Sie dann in der neuen Registerkarte zum [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` und melden Sie sich mit Ihren Azure-Anmeldedaten an, wenn Sie dazu aufgefordert werden.

    Schließen Sie alle Willkommensbenachrichtigungen, um die Startseite des Azure-Portals anzuzeigen.

1. Verwenden Sie die Schaltfläche **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud-Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung ohne Speicher in Ihrem Abonnement.

    Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Fenster am unteren Rand des Azure-Portals. Sie können die Größe dieses Bereichs ändern oder maximieren, um die Arbeit zu vereinfachen.

    > **Tipp**: Passen Sie die Größe des Bereichs so an, dass Sie hauptsächlich in Cloud Shell arbeiten können, aber dennoch die Schlüssel und den Endpunkt auf der Seite des Azure-Portals sehen, da Sie diese in Ihren Code kopieren müssen.

1. Wählen Sie in der Cloud Shell-Symbolleiste im Menü **Einstellungen** das Menüelement **Zur klassischen Version wechseln** aus (dies ist für die Verwendung des Code-Editors erforderlich).

    **<font color="red">Stellen Sie sicher, dass Sie zur klassischen Version der Cloud Shell gewechselt haben, bevor Sie fortfahren.</font>**

1. Geben Sie im Cloud Shell-Bereich die folgenden Befehle ein, um das GitHub-Repository zu klonen, das die Codedateien für diese Übung enthält (geben Sie den Befehl ein, oder kopieren Sie ihn in die Zwischenablage, und klicken Sie dann mit der rechten Maustaste in die Befehlszeile, und fügen Sie ihn als Nur-Text ein):

    ```
   rm -r mslearn-ai-info -f
   git clone https://github.com/microsoftlearning/mslearn-ai-information-extraction mslearn-ai-info
    ```

    > **Tipp**: Wenn Sie Befehle in die Cloud Shell einfügen, kann die Ausgabe einen großen Teil des Bildschirmpuffers in Anspruch nehmen. Sie können den Bildschirm löschen, indem Sie den Befehl `cls` eingeben, um sich besser auf die einzelnen Aufgaben konzentrieren zu können.

1. Nachdem das Repo geklont wurde, navigieren Sie zu dem Ordner, der die Codedateien für Ihre App enthält:

    ```
   cd mslearn-ai-info/Labfiles/content-app
   ls -a -l
    ```

    Der Ordner enthält zwei gescannte Bilder von Visitenkarten sowie die Python-Codedateien, die Sie zum Erstellen Ihrer App benötigen.

1. Geben Sie im Befehlszeilenfenster der Cloud Shell den folgenden Befehl ein, um die zu verwendenden Bibliotheken zu installieren:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt
    ```

1. Geben Sie den folgenden Befehl ein, um die bereitgestellte Konfigurationsdatei zu bearbeiten:

    ```
   code .env
    ```

    Die Datei wird in einem Code-Editor geöffnet.

1. Ersetzen Sie in der Codedatei die Platzhalter **YOUR_ENDPOINT** und **YOUR_KEY** durch Ihren Endpunkt von Azure KI Services und einen der Schlüssel (die Sie aus dem Azure-Portal kopieren). Stellen Sie sicher, dass **ANALYZER_NAME** auf `business-card-analyzer` festgelegt ist.
1. Nachdem Sie die Platzhalter ersetzt haben, verwenden Sie im Code-Editor den Befehl **STRG+S**, um Ihre Änderungen zu speichern und dann den Befehl **STRG+Q**, um den Code-Editor zu schließen, während die Befehlszeile der Cloud Shell geöffnet bleibt.

    > **Tipp**: Jetzt können Sie den Cloud Shell-Bereich maximieren.

1. Geben Sie in der Cloud Shell-Befehlszeile den folgenden Befehl ein, um die bereitgestellte JSON-Datei **biz-card.json** anzuzeigen:

    ```
   cat biz-card.json
    ```

    Scrollen Sie im Cloud Shell-Bereich zum JSON-Code in der Datei, der ein Schema für das Analysetool für eine Visitenkarte definiert.

1. Wenn Sie die JSON-Datei für das Analysetool angezeigt haben, geben Sie den folgenden Befehl ein, um die bereitgestellte Python-Codedatei **create-analyzer.py** zu bearbeiten:

    ```
   code create-analyzer.py
    ```

    Die Python-Codedatei wird in einem Code-Editor geöffnet.

1. Überprüfen Sie den Code, der:
    - Lädt das Schema für das Analysetool aus der Datei **biz-card.json**.
    - Ruft den Endpunkt-, Schlüssel- und Analysetoolnamen aus der Konfigurationsdatei der Umgebung ab.
    - Ruft eine Funktion mit dem Namen **create_analyzer** auf, die derzeit nicht implementiert ist.

1. Suchen Sie in der Funktion **create_analyzer** den Kommentar **Erstellen eines Analysetools für Inhaltsverständnis**, und fügen Sie den folgenden Code hinzu (achten Sie darauf, den richtigen Einzug beizubehalten):

    ```python
   # Create a Content Understanding analyzer
   print (f"Creating {analyzer}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # initiate the analyzer creation operation
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"}

   url = f"{endpoint}/contentunderstanding/analyzers/{analyzer}?api-version={CU_VERSION}"

   # Delete the analyzer if it already exists
   response = requests.delete(url, headers=headers)
   print(response.status_code)
   time.sleep(1)

   # Now create it
   response = requests.put(url, headers=headers, data=(schema))
   print(response.status_code)

   # Get the response and extract the callback URL
   callback_url = response.headers["Operation-Location"]

   # Check the status of the operation
   time.sleep(1)
   result_response = requests.get(callback_url, headers=headers)

   # Keep polling until the operation is no longer running
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(callback_url, headers=headers)
        status = result_response.json().get("status")

   result = result_response.json().get("status")
   print(result)
   if result == "Succeeded":
        print(f"Analyzer '{analyzer}' created successfully.")
   else:
        print("Analyzer creation failed.")
        print(result_response.json())
    ```

1. Überprüfen Sie den hinzugefügten Code, der folgende Aktionen ausführt:
    - Erstellt geeignete Header für die REST-Anforderungen.
    - Sendet die HTTP-Anforderung *DELETE*, um das Analysetool zu löschen, falls es bereits vorhanden ist.
    - Sendet die HTTP-Anforderung *PUT*, um die Erstellung des Analysetools zu initiieren.
    - Überprüft die Antwort, um die Rückruf-URL für *Operation-Location* abzurufen.
    - Sendet wiederholt die HTTP-Anforderung *GET* an die Rückruf-URL, um den Status des Vorgangs zu überprüfen, bis dieser nicht mehr ausgeführt wird.
    - Bestätigt dem Benutzer den Erfolg (oder Fehler) des Vorgangs.

    > **Hinweis:** Der Code enthält einige absichtliche Zeitverzögerungen, um eine Überschreitung des Anforderungsratenlimits für den Dienst zu vermeiden.

1. Verwenden Sie den Befehl **STRG+S**, um die Codeänderungen zu speichern. Lassen Sie den Code-Editor-Bereich jedoch geöffnet, falls Sie Fehler im Code korrigieren müssen. Passen Sie die Größe der Bereiche so an, dass Sie den Befehlszeilenbereich gut sehen können.
1. Geben Sie in der Cloud Shell-Befehlszeile den folgenden Befehl ein, um den Python-Code auszuführen:

    ```
   python create-analyzer.py
    ```

1. Überprüfen Sie die Ausgabe des Programms, die Ihnen nun mitteilen sollte, dass das Analysetool erstellt wurde.

## Verwenden der REST-API zum Analysieren von Inhalten

Nachdem Sie nun ein Analysetool erstellt haben, können Sie es über die Content Understanding-REST-API in einer Clientanwendung nutzen.

1. Geben Sie in der Cloud Shell-Befehlszeile den folgenden Befehl ein, um die bereitgestellte Python-Codedatei **read-card.py** zu bearbeiten:

    ```
   code read-card.py
    ```

    Die Python-Codedatei wird in einem Code-Editor geöffnet:

1. Überprüfen Sie den Code, der:
    - Identifiziert die Bilddatei, die analysiert werden soll, mit dem Standardwert **biz-card-1.png**.
    - Ruft den Endpunkt und den Schlüssel für Ihre Azure KI Services-Ressource aus dem Projekt ab (unter Verwendung der Azure-Anmeldeinformationen aus der aktuellen Cloud Shell-Sitzung für die Authentifizierung).
    - Ruft eine Funktion mit dem Namen **analyze_card** auf, die derzeit nicht implementiert ist.

1. Suchen Sie in der Funktion **analyze_card** den Kommentar **Verwenden von Inhaltsverständnis, um das Bild zu analysieren**, und fügen Sie den folgenden Code hinzu (achten Sie darauf, den richtigen Einzug beizubehalten):

    ```python
   # Use Content Understanding to analyze the image
   print (f"Analyzing {image_file}")

   # Set the API version
   CU_VERSION = "2025-05-01-preview"

   # Read the image data
   with open(image_file, "rb") as file:
        image_data = file.read()
    
   ## Use a POST request to submit the image data to the analyzer
   print("Submitting request...")
   headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/octet-stream"}
   url = f'{endpoint}/contentunderstanding/analyzers/{analyzer}:analyze?api-version={CU_VERSION}'
   response = requests.post(url, headers=headers, data=image_data)

   # Get the response and extract the ID assigned to the analysis operation
   print(response.status_code)
   response_json = response.json()
   id_value = response_json.get("id")

   # Use a GET request to check the status of the analysis operation
   print ('Getting results...')
   time.sleep(1)
   result_url = f'{endpoint}/contentunderstanding/analyzerResults/{id_value}?api-version={CU_VERSION}'
   result_response = requests.get(result_url, headers=headers)
   print(result_response.status_code)

   # Keep polling until the analysis is complete
   status = result_response.json().get("status")
   while status == "Running":
        time.sleep(1)
        result_response = requests.get(result_url, headers=headers)
        status = result_response.json().get("status")

   # Process the analysis results
   if status == "Succeeded":
        print("Analysis succeeded:\n")
        result_json = result_response.json()
        output_file = "results.json"
        with open(output_file, "w") as json_file:
            json.dump(result_json, json_file, indent=4)
            print(f"Response saved in {output_file}\n")

        # Iterate through the fields and extract the names and type-specific values
        contents = result_json["result"]["contents"]
        for content in contents:
            if "fields" in content:
                fields = content["fields"]
                for field_name, field_data in fields.items():
                    if field_data['type'] == "string":
                        print(f"{field_name}: {field_data['valueString']}")
                    elif field_data['type'] == "number":
                        print(f"{field_name}: {field_data['valueNumber']}")
                    elif field_data['type'] == "integer":
                        print(f"{field_name}: {field_data['valueInteger']}")
                    elif field_data['type'] == "date":
                        print(f"{field_name}: {field_data['valueDate']}")
                    elif field_data['type'] == "time":
                        print(f"{field_name}: {field_data['valueTime']}")
                    elif field_data['type'] == "array":
                        print(f"{field_name}: {field_data['valueArray']}")
    ```

1. Überprüfen Sie den hinzugefügten Code, der folgende Aktionen ausführt:
    - Liest den Inhalt der Bilddatei.
    - Legt die Version der REST-API für Inhaltsverständnis fest, die verwendet werden soll.
    - Übermittelt die HTTP-Anforderung *POST* an Ihren Endpunkt für Inhaltsverständnis und weist diesen an, das Bild zu analysieren.
    - Überprüft die Antwort des Vorgangs, um eine ID für den Analysevorgang abzurufen.
    - Sendet wiederholt die HTTP-Anforderung *GET* an Ihren Endpunkt für Inhaltsverständnis, um den Status des Vorgangs zu überprüfen, bis dieser nicht mehr ausgeführt wird.
    - Speichert bei erfolgreichem Vorgang die JSON-Antwort, analysiert sie und zeigt anschließend die Werte an, die für jedes typspezifische Feld abgerufen wurden.

    > **Hinweis:** Im hier verwendeten einfachen Visitenkartenschema stellen alle Felder Zeichenfolgen dar. Der folgende Code verdeutlicht die Notwendigkeit, bei einem komplexeren Schema den Typ jedes Felds zu überprüfen, damit Sie Werte unterschiedlicher Typen extrahieren können.

1. Verwenden Sie den Befehl **STRG+S**, um die Codeänderungen zu speichern. Lassen Sie den Code-Editor-Bereich jedoch geöffnet, falls Sie Fehler im Code korrigieren müssen. Passen Sie die Größe der Bereiche so an, dass Sie den Befehlszeilenbereich gut sehen können.
1. Geben Sie in der Cloud Shell-Befehlszeile den folgenden Befehl ein, um den Python-Code auszuführen:

    ```
   python read-card.py biz-card-1.png
    ```

1. Überprüfen Sie die Ausgabe des Programms, die nun die Werte für die Felder auf der folgenden Visitenkarte anzeigen sollte:

    ![Eine Visitenkarte für Roberto Tamburello, einem Mitarbeiter bei Adventure Works Cycles.](./media/biz-card-1.png)

1. Verwenden Sie den folgenden Befehl, um das Programm für eine andere Visitenkarte auszuführen:

    ```
   python read-card.py biz-card-2.png
    ```

1. Überprüfen Sie die Ergebnisse, die die Werte dieser Visitenkarte darstellen sollten:

    ![Eine Visitenkarte für Mary Duartes, einer Mitarbeiterin bei Contoso.](./media/biz-card-2.png)

1. Verwenden Sie im Cloud Shell-Befehlszeilenbereich den folgenden Befehl, um die vollständige JSON-Antwort anzuzeigen, die zurückgegeben wurde:

    ```
   cat results.json
    ```

    Scrollen Sie bis zum JSON-Code.

## Bereinigen

Wenn Sie Arbeit mit dem Content Understanding-Service abgeschlossen haben, sollten Sie die in dieser Übung erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden.

1. Löschen Sie im Azure-Portal die Ressourcen, die Sie in dieser Übung erstellt haben.
