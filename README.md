# Uni-Medical-Database
Medical database managment system.

This system contains the following operations:

● It works as a “database” of patients.
● Imports patient data from file.
● Generates statistics of the most relevant type of pathologies (i.e. the pathologies with more patients).

System description:
On startup the program loads the previous database from the file “database.dmp” and presents a menu that has the following mouse-selectable options:
L)ist all patients
l)ist patients of a specific pathology
I)nsert a new patient
P)rocess new file to import patients
D)isplay statistics
A)bout
E)xit

Operations description:
L)ist patients - lists in the monitor all the patients stores in the database with the following format:
<#number> ”<patient name>”, <pathology>
example:
[1] “Mark Anthony”, Pathology 1
[2] “Tom Carr”, Pathology 2


l)ist patients of a specific pathology- lists in the monitor all the patients classified with a specific pathology. Selection of the pathology with the mouse.
<Selected Pathology >
<#number> ”<patient name>”
Example:
Pathology 1
[1] “Mark Anthony”
[2] “Tom Carr”


(I)nsert a new patient - Input for new patient
Name of patient:
Pathology:
<<Main>> <<Next>>
<<Next>> Saves this record and starts a new record
<<Main>> Saves this record and return to Main Menu
  

P)rocess new file to import patients - asks to the user the path name to a file with the new patients and adds all new patient to the database. The patients are identified with the label PAT and terminated with “:”
Example:

The file:
Lorem ipsum dolor sit amet, PAT Mark Anthony:Pathology 4: elit. In vel ipsum
eget erat ultrices molestie. Duis eu elit nulla. Ut sagittis vestibulum sem. Sed
eleifend, nunc PAT Tom Carr:Pathology 1:
adds:
Mark Anthony:Pathology 4
Tom Carr:Pathology 1


D)isplay statistics - The program displays the statistics as depicted below and waits for a key or mouse button to be pressed before returning to the main menu (integer percentages).
Example:
Total patients: 100
Results:
Pathology Num. Percentage
Pathology 1 - 40 40%
Pathology 2 - 20 20%
Pathology 3 - 18 18%
Pathology 4 - 12 12%
Pathology 5 - 10 10%


A)bout - presents addictional information.

E)xit - terminates the program and returns to the operating system. It also stores all database data in the file “database.dmp”. Not in text. It is a dump of memory. It also stores the statistics of the most relevant pathologies in text format, in a file named “statistics.txt”.
