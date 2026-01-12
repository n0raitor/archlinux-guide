# To Get Sorted

[https://wiki.ubuntuusers.de/Dateisystemgr%C3%B6%C3%9Fe_%C3%A4ndern/](Source)

Um die Größe von Dateisystemen ändern zu können, müssen je nach Dateisystem verschiedene Hilfsprogramme ("tools") installiert sein:

- e2fsprogs (liefert resize2fs)

- reiserfsprogs

- xfsprogs

- jfsutils

- ntfs-3g

#### Anpassen der Dateisysteme**

**ext2 / ext3 / ext4**

Um ein ext3-Dateisystem anzupassen, darf es nicht eingehängt oder fehlerhaft sein. Seit Linux Kernel 2.6.10 kann man ext3 (nicht ext2)-Dateisysteme auch im eingehängten Zustand vergrößern, nicht jedoch verkleinern! Es ist sinnvoll, jedoch nicht nötig, das Dateisystem zuvor mit "e2fsck" auf Fehler zu überprüfen.

```bash
sudo resize2fs -p /dev/gerätename   # Vergrößert das Dateisystem bis zur maximalen Größe des Logical Volumes oder der Partition

sudo resize2fs -M /dev/gerätename   # Verkleinert das Dateisystem bis zur minimalen Größe des Logical Volumes oder der Partition

sudo resize2fs -p /dev/gerätename 5G   # Vergrößert bzw. Verkleinert das Dateisystem auf 5 Gibibyte Gesamtgröße

sudo resize2fs -P /dev/gerätename   # Gibt die Minimalgröße an, wie weit das Dateisystem verkleinert werden kann
```

NOTE: Für Größenangaben (im Beispiel 5G) verwendet "resize2fs" den Teiler 1024, nicht 1000!

NOTE: Der im Beispiel verwendete Parameter -p von "resize2fs" dient dazu, einen Fortschrittsbalken beim Anpassen des Dateisystems anzuzeigen.

**NTFS**

Um ein NTFS-Dateisystem anzupassen, darf es nicht eingehängt oder fehlerhaft sein. Auf dem Dateisystem dürfen keine NTFS-verschlüsselten oder NTFS-komprimierten Dateien oder Ordner liegen, ein vorherige Defragmentierung ist ratsam, jedoch nicht erforderlich.

```bash
sudo ntfsresize /dev/gerätename   # Vergrößert das Dateisystem bis zur maximalen Größe des Logical Volumes oder der Partition

sudo ntfsresize -n --size 5G /dev/gerätename  # Vergrößert bzw. Verkleinert das Dateisystem auf 5 Gigabyte Gesamtgröße, Testlauf im Read-only-Modus (empfehlenswert!)

sudo ntfsresize --size 5G /dev/gerätename     # Vergrößert bzw. Verkleinert das Dateisystem auf 5 Gigabyte Gesamtgröße

sudo ntfsresize -i /dev/gerätename  # Zeigt an, auf welche Größe das Dateisystem minimal verkleinert werden kann
```

NOTE: Der --size Parameter von "ntfsresize" verwendet den Teiler 1000 statt der üblichen 1024 beim Umrechnen von Gigabyte in Megabyte usw..., dies sollte beim Anpassen des Dateisystems beachtet werden! Bei der Verkleinerung ist darauf zu achten mindestens ca. 70 Mbyte über dem Minimalwert (Parameter -i) einzugeben!

NOTE: Um die NTFS-Kompression einzelner Dateien auf einem NTFS-Laufwerk als vorbereitende Maßnahme für ntfsresize abzustellen bzw. rückgängig zu machen, kann man entweder die Eigenschaften jeder einzelnen Datei manuell bearbeiten oder aber in der Windows-Eingabeaufforderung den folgenden Befehl zur Dekomprimierung aller betroffenen Datein nutzen:

```
compact /U /S:\ /I
```

**FAT32**

Ein FAT32-Dateisystem kann mit dem Programm Parted angepasst werden. Zunächst muss das Dateisystem mit umount ausgehängt werden. Anschließend startet man Parted als root wie hier beschrieben. Mit dem Befehl

```
print
```

kann man sich einen Überblick über die Partitionen der Festplatte geben lassen. Um den Vergrößerungs- bzw- Verkleinerungsvorgang zu starten, schreibt man

```
resize [NR] [START] [ENDE]
```

in die Befehlszeile von Parted, wobei [NR]für die Nummer der Partition laut der "print"-Ausgabe ist; [START] steht für die neue Startposition der Partition (z.B. 0 für den Anfang der Festplatte, 200MB als absolute Größe oder 10% als relative Größe), und [ENDE] bezeichnet die Endposition (ebenfalls als absolute oder relative Größe).
