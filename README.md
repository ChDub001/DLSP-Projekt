# DLSP-Projekt
DLSP Projetk von Dubendorff, Christoph und Müller, Simon


# FH Aachen - Fachbereich Maschinenbau & Mechatronik

## Abschlussprojekt WS 2022 / 2023

### im Modul: 81307 - Datenmanagement & Leittechnik

#### bei Prof. Dr. Ing. Stephan Kallweit

#### am 08.03.2023

 

vorgelegt von:

##### Christoph Dubendorff
##### Matrikelnummer: 3542721 

und

##### Simon Müller
##### Matrikelnummer: 3251987

_______________________________________________________________________

### Vorwort zur Verwendung der Programme

Das diesige Projekt implementiert mehrere diverse Codes. Einige dieser Codes werden über einen Mikrocontroller verwaltet und bedient. Die restlichen Code-Dateien werden hingegen über Jupyter ausgeführt. Die Ausführungen zu den jeweiligen Dateien, bezüglich ihrer Verwendung, werden einzeln im Verlaufe dieser Projektarbe, erwähnt. Die Reihenfolge, in welcher die Dateien ausgeführt werden ist irrelevant. Die Programm-Elemente wurden so aufgesetzt, dass das vollständige Programm auf die Rückmeldung der jeweils interagierenden Codes wartet.

### Einleitung

Das Projekt des Moduls DLSP wird durch die Aufgabe, ein Programm mittels Python zu schreiben, gestellt. Dieses Programm muss Betriebszustände einer Waschmaschine klassifizieren und im Rahmen einer Prozessüberwachung die Auswertung darstellen. Das Projekt zielt darauf ab, erworbene theoretische Kenntnisse in die Prraxis zu überführen. Im Rahmen der Projektbeschreibung wird die Klassifizierung von mindestens vier unterscheidlichen Betriebszuständen gefordert. Zusätzlich sollen die Daten außerhalb eines Heim-Netzwerkes zugänglich gemacht werden. Die Ausgabe der Analyse soll über ein grafisches User-Interface gestaltet werden.

Die Arbeitsmittel, welche für die Durchführung der Projektarbeit bereit gestellt wurden, bestehen aus einem ESP-8266 und einem darauf angeschlossenen MPU 6050 Gyrometer. Die Amplituden der erzeugten Schwingungen versetzen die Waschmaschine in Bewegung. Die dadurch entstandenen Beschleunigungen sind messbar. Die darauf basierende Auswertung bildet die ablauftypischen Vibrationsmuster ab. Das Projekt nimmt bezug auf eine Waschmaschine von Bauknecht. Diese Waschmaschine ist unterhalb einer Küchen-Arbeitsplatte montiert, sodass die Schwingungen der Waschmaschine mit auf die Arbeitsplatte übertragen werden und somit eine eventuelle Dämpfung der Schwingungen bedeutet.

Die Länge des betrachteten Waschvorgangs wird mit 2 Stunden und 49 Minuten angegeben. Dabei werden die folgenden Vorgänge durchlaufen:
    - Wasser einspülen
    - Vorwäsche
    - Waschen
    - Schleudern
    - Stillstand

Diese fünf Zustände sollen durch den Einsatz von maschinellem Lernen erkannt werden.

### Programme auf dem Mikrocontroller

Zur Aufnahme der Vibrationsdaten benötigt der Mikrocontroller, welcher an der Waschmaschine angebracht wurde, einige Programme. Diese Codes werden auf den Microcontroler überschrieben. Die zusätzliche Boot-Datei des Mikrocontrollers wird als erstes ausgeführt und lädt das relevante Programm in den Arbeitsspeicher. Die Hilfsdateien 'imu.py' und 'vector3d.py' dienen der Ermittlung der Lage (Kippwerte). 'main.py' enthält das Hauptprogramm, welches auf dem Microcontroller ausgeführt wird.

Zu Beginn wird im Sinne einer Funktionskontrolle eine Aufzählung erstellt. Diese enthält die Namen aller Dateien, welche auf dem Controller vorhanden sind. Darauf folgend wird von dem ESP eine Verbindung zum Heimnetzwerk aufgebaut und erzeugt basierend auf den Daten des Netzwerks die Grundlage eines Timestamp. Anschließend wird ein I2C-Bus initialisiert, um den Beschleunigungssensor einzubinden. Das Auslesen der Messdaten wird durch die Initialisierung der Variablen und Funktionen gestartet.

Die Beschselunigungswerte sind in XYZ unterteilt. Darauf folgend wird eine TCP-Verbindung mit dem Server aufgebaut. Der Mikrocontroller dient in diesem TCP-Aufbau als Client. Bei erfolgreicher Verbindung wird die Übermittlung der Daten sowie auch ein Timer gestartet. Folgend wird eine Callback-Funktion aufgerufen, welche die Anzahl der Durchläufe betrachtet. Nach 200 Durchläufen wird der Counter zurückgesetzt. Wenn der Zähler 0 erreicht, wird ein Timestamp per TCP gesendet.  Bei Verbndungsabbruch stoppt das programm von selbst.

#### Bestimmung der Abtastfrequenz:

Die maximale Drehzahl der Waschmaschine n_max beträgt 1600 U/min. Die maximale Drehfrequenz der Waschmaschine f_max ergibt sich aus der folgenden Formel:

f_max = (n_max [1/min])/(60 [s/min] )

Bei einem n_max von 1600 U/min ergibt sich eine Maximafrquenz von: (1600 [1/min])/(60 [s/min]) = 26,67 Hz

Um dem Aliasing entgegen zu wirken, wird empfohlen die Abtastrate auf mindesetens 2* f_max zu eröhen. Die so errechnete mindestens notwendige Abtastfrequenz beträgt ca. 54 Hz. Für eine noch genauere Auswertung wird diese Frequenz auf 100Hz erhöht.

### Programme in Jupyter-Notebooks

Parallel zu dem Main-Programm laufen zwei Programme in der Entwicklungsumgebung Jupyter. Ein Programm dient dem Empfang der Beschleunigungsdaten über TCP sowie dessen Verarbeitung. Mit dem anderen Programm werden die Daten in einem grafischen User-Interface visualisiert.

#### Datenempfang und Datenverarbeitung

Mit dem ersten Programm wird die Serverseite für die TCP-Kommunikation eingerichtet, die vom Mikrocontroller gesendeten Daten empfangen, verarbeitet und schließlich in die Datenbank hochgeladen. Im ersten Schritt wird ein Socket erzeugt und darauf gewartet, dass der Mikrocontroller sich mit dem Server verbindet. Danach startet eine While-Schleife,in der die vom Mikrocontroller gesendeten Daten empfangen und entschlüsselt werden. Im nächsten Schritt werden die Daten gefiltert. Dabei wird geprüft, ob die Beschleunigungsdaten vollständig und im richtigen Format übermittelt wurden. Alle unvollständigen oder falsch dargestellten Datensätze werden hierbei gelöscht. Schließlich werden die Daten in die cloudbasierte Datenbank 'MongoDB Atlas' hochgeladen. Diese While-Schleife wird solange durchlaufen, bis das Programm manuell geschlossen wird.

#### Visualisierung im einer Benutzeroberfläche

Mit dem zweiten Programm in Jupyter sollen die aufgenommenen Daten und der ermittelte Betriebszustand in einem grafischen User-Interface visualisiert werden. Beim Start des Programms wird ein Diagramm in einem Pop-up-Fenster erzeugt. In diesem Diagramm werden die aktuellsten 2.000 Datensätze dargestellt. So wird ein schneller Überblick über die Datenlage ermöglicht. Nach dem Schließen dieses Pop-up-Fensters wechselt die Benutzeroberfläche in den nächsten Modus. Dabei werden zwei Dialogfenster aktiv. Im ersten Dialogfenster werden wichtige Informationen und Statusmeldungen des Codes angezeigt. Im zweiten Dialogfenster werden weiterführende Informationen zu den erfassten Daten dargestellt. In einer Anzeige werden die zuletzt gemessenen Beschleunigungswerte dargestellt. Zudem wird ein Diagramm ausgegeben, das die letzten 500 Datensätze visuell abbildet. Die Benutzeroberfläche bietet zudem verschiedene Möglichkeiten, die Datenausgabe anzupassen. Der Nutzer kann die Datenmenge und den Informationsgehalt somit selber bestimmen. Nach Benutzung kann das Programm über einen Stop-Button unterbrochen werden.

#### Anwendung des maschinellen Lernens

Bei der Anwendung des maschinellen Lernens werden zunächst fünf Zustände klassifiziert und trainiert:
* Wasser einspülen
* Vorwäsche
* Waschen
* Schleudern
* Stillstand

Dabei fällt auf, dass das Programm Schwierigkeiten hat, Waschen und Vorwäsche zu unterscheiden. Zusätzlich lässt sich das Spülen nicht eindeutig klassifizieren. Es ist anzunehmen, dass dieser Effekt durch die Platzierung der Waschmaschine unter der Arbeitsplatte verstärkt wird. Daher wurde entschieden, nur drei Klassen zu bilden und damit nur drei Zustände zu unterscheiden. Damit soll die Genauigkeit des Algorithmus erhöht werden.:
* Waschen
* Schleudern
* Stillstand

 
