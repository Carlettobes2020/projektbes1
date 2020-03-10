1. Aufgabenstellung
Schreiben Sie eine vereinfachte Version von find(1). Als Vorbereitung macht es Sinn, wenn Sie sich die Hilfe zu find(1) genau durchlesen und damit herum experimentieren.

SYNOPSIS
======================================================
|   myfind <file or directory> [ <aktion> ] ...	     |
======================================================


2. Anleitung
Üblicherweise beginnen Programme mit einer vollständigen Analyse (z.B. mit getopt(3)) und Überprüfung der Parameter. Bei find(1) dürfen Sie getopt(3) nicht verwenden, weil damit unzulässige Parameterangaben wie --name fälschlicherweise erlaubt wären.

Weiters reicht es für dieses Bsp. aus, wenn Sie die Parameterüberprüfung direkt während der eigentlichen Auswertung machen. Dies wird nicht die "technisch beste" Implementierung ergeben, aber dafür den Aufwand in Grenzen halten und eine relativ übersichtliche Lösung ergeben.

Ebenso muß myfind einige Fehler bei System-Calls "überleben", d.h. daß ein einfaches exit(3) in vielen Fällen keine korrekte Fehlerbehandlung sein kann.

Bauen Sie ihr Programm um 2 Funktionen auf:

do_entry(const char * entry_name, const char * const * parms)
wird für jedes zu testenden Directoryeintrag aufgerufen. parms ist dabei die Adresse des ersten Arguments (so wie beim argv), das eine Aktion ist. do_entry() ruft dabei für jedes Subdirectory do_dir() geeignet auf.
do_dir(const char * dir_name, const char * const * parms)
wird für jedes zu testende Directory aufgerufen. parms ist dabei die Adresse des ersten Arguments (so wie beim argv), das eine Aktion ist. do_dir() liest das Directory aus (opendir(3), readdir(3) und closedir(3)) und ruft dabei für jeden Directoryeintrag do_entry() geeignet auf.
<aktion> kann sein
-user	<name>/<uid>	finde Directoryeinträge eines Users
-name	<pattern>	finde Directoryeinträge mit passendem Namen
-type	[bcdpfls]	finde Directoryeinträge mit passendem Typ
-print		gibt den Namen des Directoryeintrags aus
-ls		gibt die Nummer des Inodes, Anzahl der Blocks, Permissions, Anzahl der Links, Owner, Group, Last Modification Time und den Namen des Directoryeintrags aus. Bei Sym-Links wird auch das Ziel ausgegeben werden. Die Ausgabe soll in etwa wie die des bereits vorhandenen find(1) aussehen, z.B.:
=========================================================================================
|    {3}find .bashrc -ls 								|
|    8159772    8 -rw-r--r--   1 bernd    bernd        6657 May 27 00:23 .bashrc	|
=========================================================================================
Benützen Sie dazu readlink(2). Datumsausgaben sind mit strftime(3) so zu formatieren, wie es bei einem (einigermaßen) aktuellen File aussieht. Gehen Sie davon aus, daß nur "normale" Filenamen vorhanden sind — Sie brauchen nicht die speziellen Quoting-Regeln aus dem Abschnitt "UNUSUAL FILENAMES" der Manual-Page implementieren.
Wenn Sie eine 3er Gruppe sind, dann implementieren Sie noch folgende <aktion>en :
-nouser		finde Directoryeinträge ohne User
-path	<pattern>	finde Directoryeinträge mit passendem Pfad (inkl. Namen)

Verwenden Sie Unterfunktionen, um Unteraufgaben zu implementieren, und vermeiden sie doppelten/kopierten Code.

Die Aktionen werden von links nach rechts überprüft und ausgeführt. Sobald eine Aktion nicht erfüllt ist, kann die Bearbeitung des aktuellen Filenamens abgebrochen werden (sollte es sich um eine Directory handeln, muß myfind natürlich trotzdem im Directory weitermachen - so wie beim "echten" find(1)). Die Aktionen werden sozusagen logisch und verknüpft und können Seiteneffekte wie eine Ausgabe haben (so wie beim "echten" find(1)).

Den Inhalt eines Inodes - die Metadaten eines Files - bekommen Sie mit den SysCalls stat(2) bzw. lstat(2).

Die Option -user soll einen Usernamen oder eine numerische User-Id als Argument verarbeiten. Im Inode des Files wird lediglich die numerische User-Id gespeichert. Verwenden Sie die Library-Funktion getpwnam(3), um nach dem Eintrag des Users in /etc/passwd zu suchen.

Die Option -nouser testet, ob ein User mit der User-Id des Files existiert. Verwenden Sie die Library-Funktion getpwuid(3), um in /etc/passwd zu suchen.

Die Optionen -name und -path sollen Filenamen-Pattern als Argument verarbeiten. Verwenden Sie die Library-Funktione fnmatch(3), um diese Funktionalität zu erreichen.

Die Option -group soll einen Gruppennamen oder eine numerische Gruppen-Id als Argument verarbeiten. Im Inode des Files wird lediglich die numerische Gruppen-Id gespeichert. Verwenden Sie die Library-Funktion getgrnam(3), um nach dem Eintrag des Users in /etc/group zu suchen.

Die Option -nogroup testet, ob eine Gruppe mit der Gruppen-Id des Files existiert. Verwenden Sie die Library-Funktion getgrgid(3), um in /etc/group zu suchen.

Sollte etwas unklar oder verwirrend sein, ist das Verhalten des "echten" find(1) in diesem Fall zwingend als Vorgabe zu verwenden. D.h. das sie u.U. auch das zu implementierende Verhalten durch "ausprobieren" ermitteln können und/oder müssen.


3. Zwischenabgabe
Um Ihren Lernfortschritt zu überwachen gibt es bei diesem Beispiel in der Präsenzphase laut Zeitplan eine Zwischenabgabe, wo Sie Ihrem Lektor eine Zwischenversion Ihres Beispieles vorführen und dem Lektor den zugehörigen Quellcode zeigen.

Diese Zwischenversion soll die Aktion -print wie in der Anleitung beschrieben umsetzen.


4. Testen
Unter /var/tmp/test-find/simple (am annuminas.technikum-wien.at) finden Sie einige Directoryeinträge verschiedenen Typs und sonstiger Eigenschaften. Benützen Sie dieses Directory, um Ihr myfind gegen ein bekannt-korrektes find zu testen.

Das ist an und für sich das /usr/bin/find, allerdings checkt das erst alle Parameter auf syntaktische Korrektheit und beginnt erst dann zu arbeiten. Deshalb gibt es unter /usr/local/bin/bic-myfind eines, das obiges Verhalten implementiert (aber ansonsten identisch zu /usr/bin/find, soweit es die Übung betrifft).

Um eine minimale Menge an Tests zu erleichtern, gibt es das Shell-Script /usr/local/bin/test-find.sh, das /var/tmp/test-find/simple verwendet. Sie können das gerne auch lokal kopieren und verwenden.

Unter /var/tmp/test-find/full (am annuminas.technikum-wien.at) finden Sie mehr oder minder die maximale Menge an Testfällen. Da läuft obiges Script noch viel länger ....

Diese Directories wurden per Script /usr/local/bin/build-test-env-for-find.sh angelegt. Es werden auch einige spezielle User und Gruppen verwendet, die /usr/local/bin/create-accounts.pl. Sie können das mit selbiges auf anderen Rechnern anlegen. Um allerdings wirklich alles zu machen, muß man "root" sein. Als normaler User kann man kein chown(1) auf beliebige User oder auch mknod(1) machen. D.h. daß man durch Auskommentieren des Checks und der entsprechenden Zeilen eine teilweise Installation hinbekommen kann.
Am Beginn der Scripts sind einige Variablen definiert, die gegebenenfalls anzupassen sind, um das System sonst nicht zu stören. Ebenso sollte man derarige Scripts nicht "einfach so" ausführen und zumindest die Kommentare am Beginn lesen.


5. Vorbereitung
Führen Sie als Vorbereitung für dieses Beispiel folgende Aufgaben durch:

Umgang mit make(1) und doxygen:
Studieren des Makefiles für das "hello world" Beispiels.
Studieren der doxygen Kommentaren des "hello world" Beispiels.
Erstellen eines Makefiles für das "xxx" Beispiel.
Erstellen von doxygen Kommentaren für das "xxx" Beispiel. Sie werden dabei notwendigerweise die Funktionalität des Programms reverse-engineeren müssen - schon um eine entsprechende Funktionsbeschreibung schreiben zu können und auch Funktionen, Variablen, Parameter und Files sinnvoll umzubenennen.
Behandlung der Uid/User-spezifischen Aktionen (-user und -nouser):
Wie funktioniert die (einfachste) User-Verwaltung auf einem Unix-Systems? Welche Daten müssen bzw. können für jeden User gespeichert werden? Welche Felder können bzw. müssen eindeutig sein (und was könnte bzw. wird passieren, wenn sie es nicht sind)?
Wie funktioniert die Implementierung für User, die eine rein numerischen Usernamen haben? Zum Testen und Ausprobieren gibt es am annuminas.technikum-wien.at den User mit dem Usernamen "160", der die (numerische) User-Id "150" hat.
Wie werden die Library-Funktionen getpwnam(3) und getpwuid(3) eingesetzt?
Überlegen und definieren Sie entsprechende Testfälle. Die Testfälle können (und sollen) unter Zuhilfenahme des "echten" find(1) erstellt werden. Die Tests müssen selbstverständlich auch Fehlerfälle und -umstände umfassen.
Ausgabe von -ls und Inode-Information (-type, -print und -ls):
Welche Informationen finden Sie wo in einem Inode bzw. in der entsprechenden struct stat, die von stat(2)/ lstat(2) zurückgeliefert wird.
Wie werden diese Felder richtig ausgewertet?
Wie müssen Sie die Uhrzeit (mit strftime(3)) formatieren.
Überlegen und definieren Sie entsprechende Testfälle. Die Testfälle können (und sollen) unter Zuhilfenahme des "echten" find(1) erstellt werden. Die Tests müssen selbstverständlich auch Fehlerfälle und -umstände umfassen.
Globbing: (-name und -path):
Wie funktioniert Globbing? Welche Operatoren gibt es?
Wie können bzw. müssen Sie fnmatch(3) einsetzten?
Überlegen und definieren Sie entsprechende Testfälle. Die Testfälle können (und sollen) unter Zuhilfenahme des "echten" find(1) erstellt werden. Die Tests müssen selbstverständlich auch Fehlerfälle und -umstände umfassen. Sie können beliebige Directories am annuminas.technikum-wien.at auf zum Testen benützen - insbesondere /dev und /proc sind Fundgruben für Character und Block Devices sowie unlesbarer Files und Directories.
Wenn Ihre Stammgruppe aus 4 Studenten besteht dann überlegen Sie, wie die Optionen -group und -nogroup implementiert werden können:
Wie funktioniert die (einfachste) Gruppen-Verwaltung auf einem Unix-Systems? Welche Daten müssen bzw. können für jede Gruppe und jeden User gespeichert werden? Welche Felder können bzw. müssen eindeutig sein (und was könnte bzw. wird passieren, wenn sie es nicht sind)?
Wie funktioniert die Implementierung für Gruppen, die eine rein numerischen Gruppennamen haben?
Wie werden die Library-Funktionen getgrnam(3) und getgrgid(3) eingesetzt?
