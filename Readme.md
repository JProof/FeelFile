JProof FeelFile the WebFileUtility

Die Kernaufgabe von JProofResources ist einfach: Sämtliche öffentliche und nicht-öffentliche Dateien, die für irgendeine Art der Ausgabe an den Webseitenbesucher bestimmt sind
auf dem Webserver zu verwalten.
Vorab ein paar einleitende Gedanken. In der Regel werden Bilder oder generell Dateien auf den Webserver gestellt. Benamung und Größe der Dateien sind dabei 
lebenswichtig. Das selbe Bild oder Datei für unterschiedliche Sprachen unterschiedlich benennen und dabei die Dateien im Blick behalten ist lästig. Klein-gerenderte Bilder 
wieder groß machen ...aussichtslos. Lange Rede kurzer Sinn. Jeder der mit Webseiten zu tun hat, kennt die Probleme.

Ein anderer Ansatz muss her.
Upload sämtlicher Roh-Dateien(so groß wie möglich) in einen Raw-Folder. Aus diesem Raw-Folder werden keine Bilder direkt über das Web aufgerufen. 
Auch die Benennung der Roh-Dateien erfolgt automatisch und wird nicht von Hand angefasst oder das Verzeichnis betreten werden. 

Sobald ein Bild auf der Webseite benötigt wird, sei es für einen Slider, normale Content-Bilder oder geschützte Dateien, werden diese Dateien automatisch an
genau die richtige Stelle gerendert bzw. kopiert oder für bestimmte Benutzer/Gruppen reservierte Dateien ausgegeben.
  
Gleichzeitig muss sichergestellt werden, dass unterschiedliche Anforderungen an die Dateien abgedeckt sind. Zum Beispiel formatierte Content-Images und
und geschützte PDF-Dateien. Die Unterschiedliche Verarbeitung der Dateien erfolgt in sogenannten Kreisläufen(circles) 

Namenskonventionen
* __Dateiname__ Dateiname mit Punkt und Dateityp
* __Basename__ Der Dateiname ohne Trennpunkt und ohne Dateityp
* __Path__ Vollständiger Pfad zu einer Datei
* __Dir__ Verzeichnisname 
  
## Use Cases

### Basics 

Zum Nachvollziehen wie das System von Grund auf arbeitet.

* basicCycle
	
	Zeigt das Zusammenwirken zwischen Root und Format-Prozessor mit einer Datei. Gleichzeitig werden alle Generierungs-Möglichkeiten dargestellt, die zum
	für einen Root<>Format-Prozess zu Verfügung stehen. 
* basicCopyProcess

	Erweiterung des Basic-Cycle. Kopiert eine Datei mit generiertem Pfad und Dateinamen in das Formatverzeichnis. Beispielsweise für PDF
* basicResizeProcess

	Erweiterung des basicCopyProcess. Einführung in die Bild-Format-Einstellungen. Angewandt bei Bildern.
* multiFilesResizeProcess

	Mehrere Dateien über einen Processor
	
* multiFilesMultiCyclesResizeProcess

	Mehrere Root-Images in mehrere Format-Images

* extraObjectsMultiFilesMultiCyclesResizeProcess

	Mehrere Root-Images in mehrere Format-Images mit jeweils sprachabhängigen Dateinamen 

### Content-Bilder

### Slider-Gallerien

### Payed-Content Files

### All together

## Application Parts

* Configuration
* [Import](#import)
  * Upload
    * Upload durch eine Uploadmaske damit das Bild gleichzeitig gruppiert wird(via com_content.article)
* Processing
	* Filename-Rendering
	* Resize
* Deploy
	* Archives

# Configuration Circle

Jeder Circle (__\JProof\Feelfile\Configuration\Circle__) besteht aus mindestens einem Prozessor. Dem Rootprozessor. Dieser steuert den Upload bzw. den Speicherort und die Benennung der Roh-Datei im System.
Dieser Processor muss unbedingt den Namen __root__ erhalten. Der Root-Processor übergibt gleichzeitig einem Format-Processor die nötigen 
Informationen um beispielsweise ein Bild zu kopieren oder zu formatieren.

## Configuration Processor

Jeder Prozesor ist für eine Zeildatei zuständig. Die Einstellungen unterscheiden sich zwischen den Format-Processors und dem Root-Processor nicht.
Lediglich der Root-Processor muss mit __root__ benannt werden.

### Root-Processor

Hier noch eine Anmerkung zum Root-Processor. Der Root-Processor sollte so schlicht wie möglich gehalten werden. Das bedeutet, dass es völlig ausreichend
 ist, wenn der Parameter filenames mit der File-id oder dem File-Hash und der extension gespeichert wird.
 ```php
   'filenames' => [   'fileresource:id',
                      'fileresource:extension',
     ],
```
Somit sind bei dem einfachen Zugriff auf den Root-Processor keine weiteren Objekte notwending ausser eben der fileresource
```php
new RenderObjects(['fileresource' => $frc])
```

### string required: __name__


Beispiele:
* __root__ auschließlich für den Root-Processor
* __my_frontpage_slider__
* __content_images__ 
* __what_ever_you_want__ 


### string optional: __baseDir__ (default:Web)

Base-Directories: 
FeelFile unterscheidet zwischen 2 grundlegenden Verzeichnispfaden(Directories)
+ Web
 	+ Die Web ist das Startverzeichnis welches aufgerufen wird sobald mit dem Browser eine Url aufgerufen wird. [FeelFile.com](FeelFile.com/) 
 		+ /home/someClient/public_html/
+ User
	+ User meint die PHP-User-Directory. Diese Directory bezeichnet den Ausgangspfad für das Hosting. 
	Dateien die in ein Verzeichnis außerhalb der WebDir abgelegt sind, können nicht über eine Url direkt aufgerufen werden. 
	Typische Bezeichnungen und Strukturen in einem Hosting:
		+ /home/someClient/
	
So kann in einem Circle der Root-Processor die Dateie außerhalb des Web speichern und ein Format-Processor, die Zieldatei in das Web legen(i.e. images images/slider...)
Ein weiterer Format-Processor in diesem Circle kann thumbnails erzeugen, die beispielsweise nur im Adminbereich angezeigt werden.  

### array required: __dirs__  

Eines der Schlüsselelement ist die dynamische Generierung von Verzeichnissen bwz. Verzeichnisnamen. 
Jeder Teil des Pfades wird eigenständig erstellt. Pfad-Teile werden dann zu einem Pfad zusammengefügt. 

Folgende generierungs-Arten stehen in __dirs__, __filenames__ und __urls__ zur Verfügung.

* Strings 
* Objekt-Methods (muss in den ausführenden Prozessor übergeben werden)
* Static-Method-Calls 

### array optional: __urls__ //todo

Der Urls-Parameter ist dann sinnvoll, wenn sich die abgeleiteten Dateien außerhalb des Web-Roots und/oder sich in einem gesicherten
Verzeichnis befinden und über eine Url aufrufbar sein sollen. Für die einfache/direkte Anzeige/Ausgabe der Url einfach den __urls__ Teil weglassen. 
Hier wird der __dirs__ Teil verwendet.

Der __urls__ array ist untergliedert in __path__,__query__ und __addFilename__
__path__ und __query__ sind arrays 

#### array optional: __path__ (values)

Der Pfadname nach dem Host. Die einzelnen Teile werden mit einem __/__ Schrägstrich(Slash) zusammen geführt.
Beispiele: 
example.com/myImages/getImage.php

__Hinweis:__ Alle Pfadangaben werden ohne trailing-Slash ausgegeben.

#### array optional: __query__ key-value-pair

query["imgNo"=> "filresource:resourceHash"]

example.com/myImages/getImage.php?imgNo=xrobjk22

### string optional: __uriScheme__  

Welches Uri Scheme soll bei der Ausgabe der Url verwendet werden? So können Dateien unabhängig vom Aufruf der Webseite in einem anderen Scheme erfolgen. 
Dies muss jedoch durch den Server unterstützt werden. Diese Einstellung greift bei der Ausgabe von Datei-Urls.

#### auto oder nicht-angeben

Hier wird das Scheme vom Host-System verwendet. 

#### http

forciert reguläres HTTP Protokoll 
 
#### https

forciert reguläres HTTPS Protokoll 

## Import

## Processing

### Reflection

Mit dem Reflection-Framework können diverse Teile des Processings getestet werden. 
Dies dient als optisches Debugging ohne extra angeschlossenen Debugger

#### Stack

Mit dem Stack-Reflector wird ein File-Root-Format Prozess vollständig durchlaufen. Hierbei werden keine Dateien erzeugt oder abgerufen.


### Dry-Run

## Deploy

## Appendix
