
Moin zusammen,

ich versuche mal in ein paar Sätzen herunterzuschreiben worum es in der gesamten Diskussion rund um Kernel Level Anti-Cheat ala Vanguard geht, was es ist, warum es zum Teil notwendig ist und was die Bedenken sind... Aber fangen wir mal ganz von Anfang an an...


# Programme auf dem System - User Mode / Kernel Mode

Um die Diskussion zu verstehen fangen wir erstmal bei ein paar "Grundlagen" an. Wie funktioniert so ein Betriebssystem wie Windows eigentlich? Was ist dieses *User Mode* und *Kernel Mode* von dem alle reden.

![Pasted image 20240107014010.png](app://c7e4ce590919c96a181feda23e7fbae9c312/home/anno/notes/My%20vault/Pasted%20image%2020240107014010.png?1704588010909)
Kernel-Level und User-Level sind wie zwei verschiedene Ebenen in einem Computer. Mit unterschiedlichen Berechtigungen. Stell dir vor, dein Computer ist wie ein Haus.

1. **Kernel-Level**: Das ist wie das Fundament und die tragenden Wände des Hauses. Es ist der tiefste Teil des Betriebssystems, der alles zusammenhält und kontrolliert. Hier werden wichtige Entscheidungen getroffen, wie z.B. wie der Speicher verwaltet wird, wie Programme miteinander kommunizieren und wie sie auf die Hardware des Computers zugreifen können. Weil es so grundlegend ist, ist es auch sehr mächtig. Programme, die hier laufen können quasi alles mit allem auf dem PC machen, sind nicht an Sicherheitskonzepte vom Betriebssystem gebunden und kleine Fehler können dazu führen, dass wir mit einem Blue Screen begrüßt werden. Aus diesem Grund  laufen mit solchen Berechtigungen keine "normalen" Programme, sondern vorrangig Treiber für Hardware (z.B. für deine Maus/Tastatur, Netzwerk, aber auch Arbeitsspeicher oder CPUs. etc.). Ein Großteil hiervon kommt direkt vorinstalliert von Windows mit.
    
2. **User-Level**: Das ist wie die Räume in deinem Haus. Hier laufen normale Programme und Spiele. Sie können nicht so viel Macht haben wie das Fundament, weil sie auf dieser oberen Ebene arbeiten. Sie können nur das machen, was das Betriebssystem ihnen erlaubt. Das ist sicherer, weil wenn hier etwas schiefgeht, das ganze System nicht gleich betroffen ist, sondern nur das jeweilige Programm abstürzt.
   Programme, die hier laufen sind in der Regel an Sicherheitskonzepte des Betriebssystems gebunden. 
   Beispiel: Ich möchte z.b. eine PDF-Datei öffnen:
   In unserem Code schreiben wir hier keine Befehle, damit die Festplatte irgendwelche Dinge macht (das dürfen wir auch nicht, da wir als nicht-Treiber keinen direkten Zugriff auf die Hardware haben), wir müssen stattdessen das Betriebssystem fragen. Grob zusammgefasst:
1. **Anfragen statt Anordnen**: Das Programm kann die Hardware nicht direkt steuern. Es sendet Anfragen an das Betriebssystem, um bestimmte Aufgaben auszuführen.
2. **Systemaufrufe (Syscalls)**: Diese sind die Werkzeuge, die das Betriebssystem Entwicklern zur Verfügung stellt. Ein Systemaufruf für das Lesen einer Datei ist ein gutes Beispiel. Das Programm nutzt diesen Aufruf, um auf die Datei zuzugreifen. 
3. **Sicherheitsüberprüfungen durch das Betriebssystem**: Wenn ein Systemaufruf getätigt wird, führt das Betriebssystem Sicherheitsüberprüfungen durch. Es prüft, ob das anfordernde Programm die notwendigen Berechtigungen hat, um auf die Datei zuzugreifen, und ob die Datei überhaupt existiert.
4. **Delegation an Treiber**: Nach der Prüfung delegiert das Betriebssystem die Aufgabe an einen Treiber, der dann die Datei von der Festplatte liest. Erst danach bekommt das Programm Zugriff auf die Datei.
   ![[Pasted image 20240107123015.png]]
# Warum brauchen wir Kernel Level Anti-Cheat


## Wie funktioniert ein Cheat
![[Pasted image 20240107014746.png]]
Erstmal: Wie funktioniert ein Cheat grob?
Wie man im Bild sehen kann sieht hier CS ein wenig "anders" aus als es die meisten wohl gewohnt sind. Wir sehen z.b. Boxen, die andere Spieler hinter Wänden markieren und deren HitPoints, obwohl wir das gar nicht wissen dürften.  Aber wie funktioniert das denn genau?

Jedes Programm, Spiel etc. speichert Sachen im Arbeitsspeicher, um damit zu arbeiten und dir diese auch anziegen zu können. Das ist z.B. der gesamte Text der hier angezeigt wird, die Webseiten und deren Inhalt, die geöffnet sind und natürlich auch Informationen aus dem  Spiel. Irgendwo versteckt (und für einen Menschen nicht so leicht lesbar) steht so im Arbeitsspeicher wo sich ein Spieler gerade auf der Karte befindet (z.B. als Koordination X,Y,Z) oder auch wie viele Lebenspunkte sie haben


Jedes Programm hat seinen zugewiesenen Bereich im Arbeitsspeicher. Nehmen wir an, CS fordert 100 MB RAM an (auch hier: Syscalls um den Speicher zu bekommen). Innerhalb dieses Speicherbereichs sind alle relevanten Spieldaten gespeichert.
Allerdings hält dich wenig davon ab einfach mal in den Arbeitsspeicher von einem Programm rein zu schauen oder alles zu durchsuchen. Unser Programm fragt dann Windows einfach "Hey, ich möchte gern den Arbeitsspeicher von CS lesen" und in der Regel kannst du das dann einfach tun und so versuchen selbst herauszufinden, wo jetzt genau steht, welcher Gegner wo ist.
(Vielleicht hat der ein oder andere schonmal mit Cheat Engine herumgespielt - nichts anderes ist das)

![Pasted image 20240107021634.png](app://c7e4ce590919c96a181feda23e7fbae9c312/home/anno/notes/My%20vault/Pasted%20image%2020240107021634.png?1704590194370)

Mittlerweile haben viele "Hacker" Zeit investiert, um herauszufinden, wo sich die Informationen, wie die Koordinaten eines Spielers, im Arbeitsspeicher befinden (z.b. von den 100MB befinden sich alle Infos für Gegner 1 im 51. MB , Gegner 2 bei 52MB usw. ).
Diese können diese dann ausgelesen und angezeigt werden. Dieses anzeigen können dann diese Boxen sein, deren Position auf dem Bildschirm mit ein wenig Mathematik errechnet werden können.


## Wie funktioniert ein Anti-Cheat
Das ganze klingt ja zu schön um wahr zu sein! Die Möglichkeit, dass jeder den Arbeitsspeicher eines Spiels auslesen kann, stellt damit ja eine große Herausforderung für die Fairness von Spielen dar.
Aber wie versuchen Firmen das ganze zu unterbinden, wenn jeder einfach so Arbeitsspeicher lesen kann? 
Zum einen "passiv", indem es Cheatern schwer gemacht wird, Informationen im Arbeitsspeicher zu finden. Dazu gehört das Verschleiern oder Verändern der Art und Weise, wie Daten im Arbeitsspeicher gespeichert sind. Beispiel: indem zerstückelt wird, wo sich die einzelnen Koordinaten eines Spielers befinden

 Zum anderen "aktiv" mit Programmen, die sicher stellen, dass keine Cheats laufen und Dinge mit dem Spiel machen
Ein Anti-Cheat macht z.B. die folgenden Dinge:

Anti Cheat Programme funktionieren auf unterschiedliche Weisen:
- Welche Programme laufen gerade? Wie heißen diese oder sind das bekannte Cheats?
- Welche Programme greifen gerade auf den Arbeitsspeicher von dem Spiel zu?
- Wurden Dinge in dem Arbeitsspeicher des Spiels verändert? 
- Wie sieht das System aus? Gibt es z.B. bekannte "Cheat"-Treiber? Wurden Windows-"Sicherheitsfeatures" ausgeschaltet?
- Gibt es Dateien auf dem PC, die bekannte Cheats sind?

(veraltetes Beispiel für VAC - hier findet sich anscheinend alter Code - spannend zu lesen! : https://github.com/danielkrupinski/VAC)

Anhand dieser Informationen macht dann ein Anti-Cheat fest, ob ein Spieler cheatet oder nicht. So können oft viele der ganz einfachen und bekannten Cheats schnell erkannt werden.

## Anti Cheat Umgehen 
So wir haben unseren Cheat, unser Spiel hat Anti-Cheat - **Das Wettrüsten beginnt**!

Wir müssen nun irgendwie um diese Mechanismen herum kommen. Da es unser Cheat ist, ist er bisher niemandem  bekannt, daher sollten die einfachen Checks kein Problem sein... Aber was ist mit diesem "Wer greift auf Arbeitsspeicher zu?" Das könnten wir so z.B. umgehen, indem wir uns entweder direkt in CS  einhängen oder in ein anderes Programm, welches bereits auf CS zugreift.... Oder wir hängen uns ein und geben dem Anti-Cheat vorgegaukelte Werte zurück. Viele Möglichkeiten!
![Pasted image 20240107024215.png](app://c7e4ce590919c96a181feda23e7fbae9c312/home/anno/notes/My%20vault/Pasted%20image%2020240107024215.png?1704591735518)

## Kernel Level Cheats 
... Oder wir gehen eine Ebene höher und entwickeln einen "Kernel Level Cheat". 
Wir packen also unseren Cheat auf die Ebene des Betriebssystems, in der es wenige Sicherheitskonzepte gibt und es generell einen Vollzugriff auf alles gibt.
Solche Kernel Level Cheats stellen damit einen signifikanten Sprung in der Cheat-Entwicklung dar. Sie operieren direkt im Kern des Betriebssystems, einer Ebene, die normalerweise nur für die essentiellsten Funktionen wie Treiber für Tastatur, Maus oder andere Hardware reserviert ist. Diese tiefere Integration bedeutet, dass sie eine weitreichende Kontrolle über das System haben und somit auch Anti-Cheat-Maßnahmen, die im Usermode laufen, effektiver umgehen können:

1. **Tiefe Integration**: Kernel Level Cheats sind auf der tiefsten Ebene des Betriebssystems und haben damit direkten Zugriff auf Hardware und können Systemfunktionen manipulieren.
    
2. **Verbergung vor Anti-Cheat-Systemen**: Da sie auf einer tieferen Ebene als das Anti-Cheat-System operieren, können sie ihre Spuren effektiver verbergen. Sie könnten sogar die Daten, die das Anti-Cheat-Programm liest, manipulieren oder einfacher vorgeben, dass alles normal ist.
    
3. **Entwicklungsherausforderungen**: Das Entwickeln auf Kernel-Ebene ist komplex und riskant. Kleinste Fehler im Code können zu schweren Systemfehlern wie Blue Screens führen. Zudem erfordert das Installieren solcher Cheats häufig das Deaktivieren wichtiger Sicherheitsfeatures des Betriebssystems.


Das bedeutet also, dass wir all dieses Auslesen vom Arbeitsspeicher des Spiels in einen eigenen Treiber packen, den wir dann in  Windows installieren. Jedes Mal, wenn wir also jetzt wissen wollen, wo sich ein Gegner befindet greifen wir nicht direkt auf den Speicher vom Spiel zu, sondern machen in etwa das gleiche wie, wenn wir eine Datei öffnen wollen: Wir fragen das Betriebssystem bitte mit unserem Treiber zu kommunizieren, der dann das (unbemerkte) Auslesen übernimmt und unserem Cheat dann sagt, wo die Gegner sind.

![[Pasted image 20240107125710.png]]

Allerdings ist das verwenden nicht so einfach, da alle Treiber in der Regel von Microsoft "abgesegnete"  sein müssen. Wenn dies nicht so ist, muss man Sicherheitsfeatures auf seinem Rechner deaktivieren.
(Es gibt allerdings hier z.B. auch noch weitere Möglichkeiten wie Sicherheitslücken in bestehenden Treibern zu verwenden und da Sachen hinterherzuladen)


Die Verwendung von solchen Cheats bringt natürlich auch Risiken und Konsequenzen mit sich:

1. **Sicherheitsrisiken**: Indem Nutzer Kernel Level Cheats verwenden, geben sie im Wesentlichen dem Cheat (und damit potenziell dem Entwickler des Cheats) uneingeschränkten Zugriff auf ihr System. Dies kann zu schweren Sicherheitsverletzungen führen, insbesondere wenn der Cheat bösartige Funktionen enthält. 
    
2. **Hardware-Gefährdung**: Es gibt Berichte über extreme Fälle, in denen die Verwendung von Cheats zu irreparablen Hardware-Schäden geführt hat, etwa durch Überlastung oder Fehlfunktionen (im Sinne von: Komponenten nicht mehr verwendbar, siehe: https://youtu.be/umF4JsBaK4I?si=w2gQAWEWwCgT8WJX&t=628 ))



Heutzutage gibt es auch viele private Cheats, die noch viel mehr machen, viel Komplexer sind, und auch noch schwerer erkennbar sind... z.B. so auch Cheats, die in USB-Mäusen drin sind, als Treiber laufen.
# Kernel Level Anti-Cheats
## Funktionsweise

Das gleiche Wettrüsten auf Cheat Ebene gibts jetzt auch auf der anderen Ebene - bei den Anti-Cheats

Kernel-Level Anti-Cheats sind wie ein besonders strenger Sicherheitsdienst für dein Computerspiel. Es arbeitet genau so tief im Betriebssystem, also quasi im "Herz" deines Computers, wo es alles ganz genau überwachen kann und Dinge verhindern kann, bevor jemand überhaupt auf z.B. den Speicher vom Spiel zugreifen kann: 
1. **Tiefe Integration im System**: Kernel-Level Anti-Cheats sind im Kernel-Level und haben somit kompletten Zugriff auf alles, was dein Computer so macht und können so das System von einem zentralen Punkt aus überwachen, was ihnen eine umfassende Kontrolle ermöglicht.
3. **Umfassende Überwachung**: Hierdurch kann das Kernel-Level Anti-Cheat nahezu alle Vorgänge auf dem Computer überwachen. Dies schließt den Zugriff auf den Arbeitsspeicher und andere kritische Systemfunktionen ein.
4. **Präventive Maßnahmen**: Im Gegensatz zu traditionellen Anti-Cheat-Systemen, die auf die Erkennung von Cheats nach deren Aktivierung ausgerichtet sind, kann ein Kernel-Level Anti-Cheat viel einfacher potenzielle Cheats blockieren, bevor sie überhaupt aktiv werden.
5. **Frühzeitige Aktivierung**: Beispielsweise startet das Anti-Cheat-System Vanguard mit dem Betriebssystem. Dadurch ist es aktiv, bevor die meisten anderen Programme, einschließlich potenzieller Cheats, geladen werden. Diese Frühstart-Funktion ermöglicht es dem Anti-Cheat-System, Manipulationsversuche von Anfang an zu erkennen und zu unterbinden.

## Probleme 

Ähnlich wie bei der Verwendung von Kernel Level Cheats bringen Kernel Level Anti-Cheats auch gewisse Bendenken hinsichtlichen Datenschutz und Sicherheit mit sich. Der Unterschied ist hier nur: Anders als bei Cheats müssen alle Spieler diesen Kernel Level Anti Cheat laufen lassen:



- Datenschutzbedenken
	- Kernel-Level Anti-Cheat-Systeme wie Vanguard haben weitreichende Zugriffsrechte, was hinsichtlich Datenschutz bedenklich sein kann. Diese Programme können theoretisch in alle Systemprozesse eingreifen. Besonders kritisch wird dies gesehen, wenn das Unternehmen hinter dem Anti-Cheat, wie im Fall von Tencent, in einem Land mit unterschiedlichen Datenschutzstandards ansässig ist. 
	- Zusätzlich sind "Sicherheitsmechanismen" von Windows selbst (wenn z.B. Dateien von einem anderen Nutzer angesehen werden sollen) außer Kraft gesetzt
	- Meine Meinung: 
	  Generell sollte man nichts laufen lassen, was man nicht auf seinem System traut. Allerdings: Auch ein "normales" Anti-Cheat könnte man sowas wie Passwörter etc. in der Regel auslesen. Viele "Sicherheitsfeatures" (wie Berechtigungen auf Order/Dateien für mehrere Nutzer) werden zudem oft von uns "Anwendern" nicht wirklich genutzt oder wir legen wenig Wert drauf.
	  Dennoch: Grundgedanke für mich komplett verständlich und seh ich da auch schwierig
- Sicherheit bzgl. System/Hardware Zugriff
	- Vanguard könnte in der Theorie auch auf sowas wie CPU/Grafikkarte etc. zugreifen oder die Festplatte "zerstören", allerdings wird sowas wohl nicht wirklich passieren
- Sicherheit von Vanguard generell
	- Ein Kernproblem besteht darin, dass bei einer Sicherheitslücke im Anti-Cheat-System der gesamte Computer kompromittiert werden könnte, da diese Sicherheitslücke dann von bösartigen Programmen ausgenutzt werden könnte
	- Bei Vanguard ist das große Problem hier: Es läuft 24/7 im Hintergrund. Wenn es eine Lücke gibt, dann kann diese immer ausgenutzt werden, nicht nur wenn Valorant oder das Anti-Cheat läuft 
	- Dies ist kein theoretisches Konzept, sondern mit Genshin Impact und Street Fighter wirklich so passiert
	  (siehe z.B. unten die Links)
	  
  - Linux 
	  - Vanguard und ähnliche Systeme laufen nicht unter Linux, da sie auf spezifischen Windows-Funktionen basieren. Eine Deaktivierung unter Linux würde zu einem Anstieg von Cheating führen.
	  - In virtuellen Maschinen "erlauben" ? --> VMs werden heute in 95% der Fälle nur zum Bots hochspielen verwendet. Kaum jemand spielt in einer VM, damit fällt das auch raus und mit Vanguard wird das hier unterbunden
  - Externe Tools
	  - Viele externe Tools lesen den Arbeitsspeicher von League um Infos anzuzeigen (z.B. Blitz, Porofessor, TFT Tools) oder nutzen s
	  - Unter anderem auch für Videoproduktion wird auch oft viel über Memory Reading/Writing gemacht (sowohl Spectator als auch Ingame):
	    https://twitter.com/SkinSpotlights/status/1743308455811653974
	    https://github.com/floh22/LeagueBroadcast
	  - Quasi alle Spectator Overlays machen Memory Reading
	  - + Custom Skins nicht möglich (keine Ahnung wieso, da hab ich keinen Plan von)
# Fazit
Alles in allem: Muss jeder selbst wissen.
 Es ist von meiner Sicht aus verständlich, dass ein Anti Cheat auf Kernel Level läuft, da somit viel viel einfacher Cheats erkannt werden können oder auch viele einfacher verhindert werden können.
 So komplex wie teilweise die Programme da hinter sind ist das wohl der einfachste/"günstigste" Weg, um das Problem zu 90% zu beseitigen.
 Für mich ist das größte Problem (neben der Tatsache, dass ich als Linux nutzer raus bin und zuletzt ein wenig Zeit in ein League-Projekt für Replays reingesteckt habe, welches viel Memory Reading betreibt und das vermutlich nicht mehr gehen wird), dass Vanguard 24/7 läuft.
  
