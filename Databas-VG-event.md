###### Events Johan Snis VG
## Del 1: Databasdesign och Implementering

 ##### 1. Databasdesign och Implementering
 Jag började med att lägga upp strukturen på hur min databas skulle se ut och vilka funktioner jag ville ha. Jag tyckte det var intressant att kunna göra beräkningar med hjälp av tabellerna så det var mitt mål och därför fick kostnader vara mitt övergripande tema.

##### 2. Utbyggnad och Design
 Med hälp av dom 2 tabellerna jag fick i grundschemat valde jag att expandera det till totalt 8 tabeller som är följande:

  * Venues
  * Events
  * Attendees
  * Employees
  * Sponsorships

 Utav dom 8 är 3st bridgetabeller där Tickets har annan data också:

  * Event_Attendees
  * Event_Employees
  * Tickets


 ##### 3.  Normalisering och Visualisering
 Jag tycker att det är svårt att förklara varför något uppfyller vissa krav så därför har jag valt att visa hur min databas skulle kunna se ut för att inte uppfylla dom olika normalformerna
 ###### NF1:
 ###### Alla Kolumnvärden ska endast innehålla ett värde och ha primärnycklar.
 Jag anser att detta är ganska självklart och görs utan att lägga märke till det.
 Exempel i min tabell som inte uppfyller NF1:
 ```SQL
 INSERT INTO Attendees (Name, Email, DateOfBirth)
 VALUES
 ('Johan, Snis','johan.snis@example.com','16:e augusti nittonhundra 98')
 ```
 I det här fallet bryter Johan,Snis mot NF1 för att det är flera värden i en kollumn. Man kan tro att 16:e augusti nittonhundra 98 bryter mot NF1 men det gör den inte för att den endast innehåller 1 värde. Däremot är det ett design fel för att det inte följer formatet DATE

 ###### NF2:
 ###### Varje icke-nyckelattribut ska vara beroende av hela primärnyckeln
 Här tog jag bort kolumnen locations i grundtabellen events då jag har valt att göra events inte bunda till en specifik location utan ett Event enligt mig kan vara på flera locations. Exempel: Fotbolls VM eller en artist Tour.
  Exempel i min tabell som inte uppfyller NF2:
 ```SQL
 CREATE TABLE Events (
    EventID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(100) NOT NULL,
    StartDate DATE NOT NULL,
    EndDate DATE NOT NULL,
    Location VARCHAR(100)
 );
 ```
 Här som jag nämnde tidigare var location tidigare med men jag valde att ta bort det för att jag hade lagt till tabellen Venues som Location är beroende
 av. Dvs Locations är inte beroende av Events.
 ###### NF3:
 ###### Inga icke-nyckelattribut är transitivt beroende av primärnyckeln
 Om du ser till att alla attribut är endast beroende av primärnyckeln uppnår du både NF2 och NF3.
 Exempel i min tabell som inte uppfyller NF3:
 ```SQL
 CREATE TABLE Tickets (
    TicketID INT PRIMARY KEY AUTO_INCREMENT,
    TicketType VARCHAR(100) NOT NULL,
    Price Int NOT NULL,
    AttendeeID INT,
    EventID INT,
    AttendeeDOB Date,
    FOREIGN KEY (AttendeeID) REFERENCES Attendees(AttendeeID) ,
    FOREIGN KEY (EventID) REFERENCES Events(EventID)
 );
 ```
 Den här tabellen uppfyller inte NF3 för att AttendeDOB är beroende av AttendeID som i sin tur är relaterad till TicketID. Detta skapar ett transitivt beroende.

 ###### NF4:
 ###### Attribut får inte ha flera oberoende värden för samma nyckel.
 När jag valde att dela upp mina tabeller till bridgetabeller så insåg jag senare att det var ett krav för den fjärde normalformen. Detta tyckte jag var väldigt kul när jag insåg det då jag gjorde det omedvetet och kom fram till det utan hänsyn till NF4.
 Exempel i min tabell som inte uppfyller NF4 :
```sql
CREATE TABLE Events (
    EventID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(100) NOT NULL,
    StartDate DATE NOT NULL,
    EndDate DATE NOT NULL,
 );

 CREATE TABLE Attendees (
    AttendeeID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) NOT NULL UNIQUE,
    DateOfBirth DATE NOT NULL,
    EventID INT,
    FOREIGN KEY (EventID) REFERENCES Events(EventID)
 );
 ```
 Om jag inte skulle ha bridge tabellen Event_Attendees skulle tabellerna bli sammanlänkade på följande sätt. Detta medföjer att det kommer finnas data med dubletter av attendees där enda skillnaden är EventID. Detta uppfyller inte NF4.

 #### er Diagram
 Här är mitt er Diagram vilket jag har valt att göra i mermaid.

 ```mermaid
   erDiagram
    Events {
        INT EventID PK
        VARCHAR Title
        DATE StartDate
        DATE EndDate
    }

    Attendees {
        INT AttendeeID PK
        VARCHAR Name
        VARCHAR Email
        DATE DateOfBirth
    }

    Event_Attendees {
        INT EventID PK, FK
        INT AttendeeID PK, FK
    }

    Venues {
        INT VenueID PK
        VARCHAR VenueName
        INT Capacity
        BOOLEAN Outdoors
        VARCHAR City
        VARCHAR Country
        INT Cost
    }

    Tickets {
        INT TicketID PK
        VARCHAR TicketType
        INT Price
        INT AttendeeID FK
        INT EventID FK
        INT VenueID FK
    }

    Employees {
        INT EmployeeID PK
        VARCHAR Name
        VARCHAR Role
        INT Salary
    }

    Event_Employees {
        INT EventID PK, FK
        INT EmployeeID PK, FK
    }

    Sponsorships {
        INT SponsorshipID PK
        VARCHAR SponsorName
        VARCHAR ContributionType
        INT Amount
        INT EventID FK
    }

    Events ||--o{ Event_Attendees : "has"
    Attendees ||--o{ Event_Attendees : "attends"
    Events ||--o{ Tickets : "sells"
    Venues ||--o{ Tickets : "has tickets for"
    Attendees ||--o{ Tickets : "buys"
    Events ||--o{ Event_Employees : "has"
    Employees ||--o{ Event_Employees : "works at"
    Events ||--o{ Sponsorships : "has"

```
 Liten kommentar om diagrammet är att jag försökte att få diagrammet ha följande struktur {ColumnName} {DataType} {Constraint} för varje attribut men av någon anledning gick det inte och det löste sig efter att jag stuvade om ordningen. Varför vet jag inte. (Expecting 'ATTRIBUTE_WORD', got 'BLOCK_STOP)

 ##### 4.

 ##### Skapande av Tabeller

 ```SQL
 -- SKAPANDE AV SAMTLIGA TABELLER
CREATE TABLE Events (
    EventID INT PRIMARY KEY AUTO_INCREMENT,
    Title VARCHAR(100) NOT NULL,
    StartDate DATE NOT NULL,
    EndDate DATE NOT NULL
);

CREATE TABLE Attendees (
    AttendeeID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) NOT NULL UNIQUE,
    DateOfBirth Date NOT null
);

CREATE TABLE Event_Attendees (
    EventID INT,
    AttendeeID INT,
    PRIMARY KEY (EventID, AttendeeID),
    FOREIGN KEY (EventID) REFERENCES Events(EventID) ,
    FOREIGN KEY (AttendeeID) REFERENCES Attendees(AttendeeID)
);

CREATE TABLE Venues (
    VenueID INT PRIMARY KEY AUTO_INCREMENT,
    VenueName VARCHAR(100) NOT NULL,
    Capacity INT NOT NULL,
    Outdoors BOOLEAN DEFAULT TRUE,
    City VARCHAR(100) NOT NULL,
    Country VARCHAR(100) NOT NULL,
    Cost Int
);

CREATE TABLE Tickets (
    TicketID INT PRIMARY KEY AUTO_INCREMENT,
    TicketType VARCHAR(100) NOT NULL,
    Price INT NOT NULL,
    AttendeeID INT,
    EventID INT,
    VenueID INT,
    FOREIGN KEY (AttendeeID) REFERENCES Attendees(AttendeeID),
    FOREIGN KEY (EventID) REFERENCES Events(EventID),
    FOREIGN KEY (VenueID) REFERENCES Venues(VenueID)
);

CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Role VARCHAR(100) NOT NULL,
    Salary Int NOT NULL
);

CREATE TABLE Event_Employees (
    EventID INT,
    EmployeeID INT,
    PRIMARY KEY (EventID, EmployeeID),
    FOREIGN KEY (EventID) REFERENCES Events(EventID) ,
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID)
);

CREATE TABLE Sponsorships (
    SponsorshipID INT PRIMARY KEY AUTO_INCREMENT,
    SponsorName VARCHAR(100) NOT NULL,
    ContributionType VARCHAR(100) NOT NULL,
    Amount Int NOT NULL,
    EventID INT,
    FOREIGN KEY (EventID) REFERENCES Events(EventID)
);

```

 ```SQL
 -- ALL DATA I TABELLERNA
INSERT INTO Attendees (Name, Email, DateOfBirth)
VALUES
('Anna Karlsson', 'anna.karlsson@example.com', '1990-03-15'),
('Erik Svensson', 'erik.svensson@example.com', '1985-06-20'),
('Sara Lind', 'sara.lind@example.com', '1992-11-02'),
('Lars Johansson', 'lars.johansson@example.com', '1987-04-12'),
('Eva Andersson', 'eva.andersson@example.com', '1995-01-30'),
('Mats Nilsson', 'mats.nilsson@example.com', '1991-08-14'),
('Emma Eriksson', 'emma.eriksson@example.com', '1993-07-23'),
('Nils Larsson', 'nils.larsson@example.com', '1989-05-16'),
('Karin Gustafsson', 'karin.gustafsson@example.com', '1990-11-09'),
('Peter Berg', 'peter.berg@example.com', '1986-03-02'),
('Maria Olsson', 'maria.olsson@example.com', '1992-09-25'),
('Sven Holm', 'sven.holm@example.com', '1983-02-18'),
('Eva Bergström', 'eva.bergstrom@example.com', '1988-12-04'),
('Oskar Sjöberg', 'oskar.sjoberg@example.com', '1994-10-12'),
('Linda Lund', 'linda.lund@example.com', '1993-06-08'),
('Fredrik Axelsson', 'fredrik.axelsson@example.com', '1990-01-05'),
('Maja Persson', 'maja.persson@example.com', '1997-11-22'),
('David Hansson', 'david.hansson@example.com', '1985-04-17'),
('Helena Ström', 'helena.strom@example.com', '1996-03-15'),
('Anders Ek', 'anders.ek@example.com', '1991-12-20'),
('Sofia Norén', 'sofia.noren@example.com', '1993-07-30'),
('Thomas Falk', 'thomas.falk@example.com', '1989-06-19'),
('Jenny Sund', 'jenny.sund@example.com', '1994-02-14'),
('Henrik Björk', 'henrik.bjork@example.com', '1987-09-10'),
('Caroline Dahl', 'caroline.dahl@example.com', '1995-08-21');

INSERT INTO Events (Title, StartDate, EndDate)
VALUES
('Stockholm Jazz Festival', '2024-07-20', '2024-07-25'),
('Stockholm Film Festival', '2024-11-06', '2024-11-17'),
('Taylor swift World tour", "2024-05-09","2024-08-17');

INSERT INTO Venues (VenueName, Capacity, Outdoors, City, Country, Cost)
VALUES
('Avicii Arena', 16500, FALSE, 'Stockholm', 'Sweden', 200000),
('Strawberry Arena', 65000, FALSE, 'Stockholm', 'Sweden', 320000),
('Kungsträdgården', 5000, TRUE, 'Stockholm', 'Sweden', 100000),
('Berwaldhallen', 1300, FALSE, 'Stockholm', 'Sweden', 70000),
('Gärdet Festivalområde', 20000, TRUE, 'Stockholm', 'Sweden', 150000);


INSERT INTO Tickets (TicketType, Price, AttendeeID, VenueID, EventID)
VALUES
('Standard', 500, 1, 5, 1),
('Standard', 500, 1, 1, 3),
('Standard', 500, 1, 2, 3),
('Standard', 500, 2, 1, 3),
('VIP', 1200, 2, 3, 1),
('Standard', 500, 3, 5, 3),
('Standard', 500, 4, 4, 2),
('Standard', 500, 5, 3, 1),
('Standard', 500, 6, 2, 3),
('VIP', 1200, 7, 5, 1),
('VIP', 1200, 8, 2, 2),
('Standard', 500, 9, 5, 3),
('Standard', 500, 9, 2, 3),
('Standard', 500, 10, 5, 3),
('Standard', 500, 11, 5, 1),
('Standard', 500, 12, 5, 1),
('VIP', 1200, 13, 5, 3),
('Standard', 500, 14, 2, 3),
('VIP', 1200, 15, 5, 1),
('Standard', 500, 16, 2, 3),
('Standard', 500, 17, 4, 2),
('Standard', 500, 18, 4, 2),
('Standard', 500, 19, 5, 1),
('VIP', 1200, 20, 1, 3),
('Standard', 500, 21, 3, 1),
('Standard', 500, 22, 2, 3),
('Standard', 500, 23, 1, 3),
('Standard', 500, 24, 2, 3),
('Standard', 500, 25, 2, 3),
('Standard', 500, 22, 3, 1),
('Standard', 500, 25, 4, 2);

INSERT INTO Event_Attendees (EventID, AttendeeID)
VALUES
(1, 1),
(1, 2),
(1, 5),
(1, 7),
(1, 11),
(1, 12),
(1, 15),
(1, 19),
(1, 21),
(2, 4),
(2, 8),
(2, 17),
(2, 18),
(2, 25),
(3, 1),
(3, 2),
(3, 3),
(3, 6),
(3, 9),
(3, 10),
(3, 13),
(3, 14),
(3, 16),
(3, 20),
(3, 22),
(3, 23),
(3, 24);

INSERT INTO Sponsorships (SponsorName, ContributionType, Amount, EventID)
VALUES
('Sony Music Entertainment AB', 'Equipment', 30000, 1),
('Café Jazz', 'Advertising', 20000, 1),
('Filmstaden AB', 'Financing', 75000, 2),
('CineTech', 'Equipment', 40000, 2),
('LiveNation', 'Financial support', 100000, 3),
('Swifties Sweden', 'Advertising', 50000, 3),
('MusicGear AB', 'Equipment', 60000, 3);


INSERT INTO Employees (Name, Role, Salary)
VALUES
('Anna Karlsson', 'Event Manager', 45000),
('Erik Svensson', 'Technician', 35000),
('Sara Lind', 'Marketing Specialist', 40000),
('David Andersson', 'Sound Engineer', 38000),
('Emma Johansson', 'Stage Manager', 42000);

INSERT INTO Event_Employees (EventID, EmployeeID)
VALUES
(1, 1),
(1, 2),
(1, 4),
(2, 1),
(2, 3),
(2, 5),
(3, 1),
(3, 2),
(3, 4),
(3, 5);

```

 För att nu få fram användbarheten har jag skapat en Vy (View) som får fram biljettintäkter per event.

 ```SQL
CREATE VIEW TicketRevenuePerEvent as
SELECT
    e.EventID,
    e.Title AS EventName,
    COUNT(t.TicketID) AS TicketsSold,
    SUM(t.Price) AS TotalRevenue
FROM
    Events e
JOIN
    Tickets t ON e.EventID = t.EventID
GROUP BY
    e.EventID, e.Title;
```
 * Select: Tar fram kolumnerna som ska visas. Count räknar antalet biljetter som har sålts för varje event. SUM summererar priset price på alla biljetter för varje event.
 * From: Anger att tabellen Events är huvudtabell
 * Join: Definierar vilkoret för att länka tabellerna: en biljett tillhör ett Event baserat på det gemensamma EventID:t
 ```SQL
SELECT * FROM TicketRevenuePerEvent;

EventID|EventName              |TicketsSold|TotalRevenue|
-------+-----------------------+-----------+------------+
      1|Stockholm Jazz Festival|         10|        7100|
      2|Stockholm Film Festival|          5|        3200|
      3|Taylor swift World tour|         16|        9400|
```

 ## Del 2: Stored Procedure
 Med vyn har jag skapat en store procedure som tar ut alla intäkter på ett specifikt event. Där

 ```SQL
CREATE PROCEDURE EventRevenue(IN x_EventID INT )
BEGIN
    SELECT
        t.EventID,
        t.EventName,
        t.TotalRevenue AS TicketRevenue,
        SUM(s.Amount) AS SponsorRevenue,
        t.TotalRevenue + SUM(s.Amount) AS TotalEventRevenue
    FROM
        TicketRevenuePerEvent t
    JOIN
        Sponsorships s ON t.EventID = s.EventID
    WHERE
        t.EventID = x_EventID AND s.EventID = x_EventID
    GROUP BY
        t.EventID, t.EventName, t.TotalRevenue;
END$$
```
 Det här valde jag att göra för att jag tyckte det var intressant att bygga ut på en vy. Det är inte så stor skillnad på Vyn TicketRevenuePerEvent men den tar med sponsor intäkterna också och gör en sammanställd kolumn för TotalEventRevenue som består av biljettintäkterna och sponsorintäkterna.

 ```SQL
 CALL EventRevenue(1);

 -- Query: Call EventRevenue()
1	Stockholm Jazz Festival	7100	50000	57100
 ```
 ##  Del 3: Prestandaanalys

 Jag har valt att analysera min store procedure. Det som går att säga direkt är att i och med att databasen är såpass liten så spelar i princip optimeringsjustering som görs ingen roll. Men om vi leker med tanken att databasen är en etablerad biljettförsäljare.

 ```SQL
 EXPLAIN
 SELECT
    t.EventID,
    t.EventName,
    t.TotalRevenue AS TicketRevenue,
    SUM(s.Amount) AS SponsorRevenue,
    t.TotalRevenue + SUM(s.Amount) AS TotalEventRevenue
 FROM
    TicketRevenuePerEvent t
 JOIN
    Sponsorships s ON t.EventID = s.EventID
 WHERE
    t.EventID = 1
 GROUP BY
    t.EventID, t.EventName, t.TotalRevenue;

 --Query:Explain
 1	PRIMARY	s	ref	EventID	EventID	5	const	2	Using temporary; Using filesort
 1	PRIMARY	<derived2>	ALL				10	Using where; Using join buffer (flat, BNL join)
 2	DERIVED	e	const	PRIMARY	PRIMARY	4	const	1
 2	DERIVED	t	ref	EventID	EventID	5	const	10
 ```
 Om vi kollar vad Explain ger oss så ser vi att vi framför allt har två intressanta delar. Using temporary, Using filesort kan vi jobba med.

 Using temporary: Innebär att en tillfällig tabell används för göra Query. Tillfälliga tabeller kan orsaka prestanda problem då det kan vara krävande för hårdvaran. Vi kan underlätta tillfälliga tabellerna genom att skapa indexex för columnerna i ORDER BY och GROUP BY då det är främst dom som skapar Tillfälliga tabeller.
 ```SQL
 -- Exempel
 CREATE INDEX idx_grouping_ticket ON TicketRevenuePerEvent(EventID, EventName, TotalRevenue);
 ```
 Using filesort: Innebär att databasen sorterar datan för att leta efter rätt data. Detta innebär även här att det saknar lämpligt index för att databasen ska slippa använda filesort.

 ```SQL
 -- Exempel
 CREATE INDEX idx_eventid_sponsorship ON Sponsorships(EventID);
 ```
 Utöver detta skulle vi även kunna göra Where clausen mer specifik och lägga till en till begränsning.

 ```SQL
 -- Exempel
 AND t.TotalRevenue > 5000

 ```
 ## Del 4: Användarhantering och Säkerhet
 Här skapade jag 2 användare.
 En super user som har alla rättighter över alla databaser. En super user helt enkelt.

 Sen skapade jag en lite och svag användare som endast har rättigheterna att "läsa" på en databas med rollen mini_user.
 ```SQL
 -- Skapar en roll och tilldelar rollen till mini_user
 CREATE ROLE 'Read_only';
 GRANT SELECT ON examinationsuppgift.* TO 'Read_only';
 CREATE USER 'mini_user'@'localhost' IDENTIFIED BY '<choose-a-local-password>';
 GRANT 'Read_only' TO 'mini_user'@'localhost';

 -- Skapar en user och ger den Alla rättigheter på alla databaser samt rättigtheter att ändra ändras rättigheter
 CREATE USER 'super_user'@'localhost' IDENTIFIED BY '<choose-a-local-password>';
 GRANT ALL PRIVILEGES ON *.* TO 'super_user'@'localhost' WITH GRANT OPTION;
 ```

 Skapa en backup av en databas och lagrar den på skrivbortet
 ```bash
 mysqldump -u root -p examinationsuppgift > examinationsuppgift.sql

 ```
 När det kommer till datahaveri skulle det kunna vara att ett systemfel händer och databasen blir korrupt eller vilket jag skulle gissa är vanligare är att det blir ett användarfel som orsakar att data förloras. I det här fallet antar vi bara att hela databasen försvinner.

 ```SQL
 -- Tar bort databasen
 DROP DATABASE examinationsuppgift;
 -- Skapar en ny databas som är tom nu
 CREATE DATABASE examinationsuppgift;
 ```
 Återställer backupen
 ```Bash
 mysql -u root -p examinationsuppgift < examinationsuppgift.sql
 ```
 ## Projektrapport
 Jag vill börja med att säga att jag tycker att det har varit väldigt roligt projket. Jag tycker det är väldit intressant att hoppa in i något ganska snabbt och dyka in sig lite på djupet men det har inte kommit utan sina probelm. Jag har lagt ner väldigt mycket tid för att det inte alltid går som man vill speciellt inte när man är nybörjare.

 Min första tanke var att jag skulle göra G-uppgiften och sedan utöka Databasen men jag insåg att det inte var rätt strategi för att allt i inlämningsuppgiften var baserat på hur din databas såg ut så jag kunder inte riktigt börja med inlämningsuppgiften innan databasen var komplett. Med det sagt kan jag även säga att det här är min tredje database då dom tidigare hade problem och jag ansåg att det bara var lättare att börja om.

 Den största delen av tiden togs till att planera struktuern på databasen. Hur tabellerna ska vara sammankopplade, vilka kopplingar dom ska ha och vilken data som är relevant att ha i tabellerna. Det senaste exemplet jag kan ge är att tickets inte tidigare hade en foreign key för VenueID. Men jag insåg att det är rimligt att en biljett borde vara kopplat till en Venue så då la jag till det men insåg då att jag kunde ta bort en bridge tabell som jag hade för att länka ihop Event till Venues för att båda dom nu finns i tickets.
 Utöver det kommer jag ihåg att jag ville undvika att tabeller skulle ha flera kolumner med samma värder. Exempel en attendee för ett event och sen samma attendee med en ny foreign key för EventID. Alltså att samma attende är i databasen 2 gånger. Så när jag kom på att jag kunde använde mig utav bridge tabeller fick jag en riktig Eureka moment.

 Att skapa få in datan tabellerna var inte speciellt svårt men det var lite pilligt. Jag använde AI för att generera data såsom namn, epost, ålder osv. Det som var pilligt var att se till att alla bridge tabellerna hade värden som inte krockade med övriga tabeller. När jag genererade den datan på egen hand var jag tvungen att kolla på mina tabeller konstant och se till att jag la in data som stämde överens.

 Att skapa Vyn var inte speciellt svårt. Jag behövde testa den frekvent och se vad som fungerade och vad som inte fungerade. Det som var lite klurigt var strukturen på Join men nu förstår jag det bättre.

 Har har fler select metoder men känner att inlämningsuppgiften redan är lång så jag lägger inte in dom då dom funktionellt är lika.

 Planen var att min store procedure skulle var ännu mer komplex med utgifter också för venues och Employees men det blev lite för komplext så jag bestämde mig för att bygga ut på min Vy och lägga till en variabel. Sen insåg jag att min store procedure lika gärna skulle kunna vara en view. Då bestämde jag mig för att lägga till att den endast Selectar EventIDt för det nummret som användaren skriver i proceduren för att kunna skilja sig ifrån en view.

 Skapa användare och roller var självklart in mina ögon. Det var likt att ha en discord server med roller, rättigheter osv.

 För backups ville jag först använda mig utav mariabackup men fick det inte att fungera så jag hörde att många använde sig utav mysqldump vilket var smidigare men det kanske saknar lite funktionalitet. Oavestt så fungerade det bra och var smidigt.

 Slutsats. Jag är nöjd med min databas och speciellt strukturen på den. Det är svårt att veta hur en komplex databas faktiskt ser ut arbetslivet. Och eftersom jag inte riktigt har någon referens så tror jag att jag har gjort en bra simpel databas som kan hantera mycket data, går att expandera och är funktionell. I Efterhand borde jag inte gjort den lite mer simpel för att jag har lagt ner väldigt mycket tid på den däremot har jag lärt mig väldigt mycket under mina försök och misslyckanden.



