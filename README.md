

#Messy Labbage
##Analys av säkerhetsproblem och prestanda
##### _Jasmin Bejtovic JB223CP_
##### _Laboration 2 - Webbteknik 2_
##### _Linnéuniversitetet, Webbprogrammerare 2014_

##Säkerhetsproblem
### SQL Injection
##### Problem

Genom att skriva:
```
dittnamn@vadsomhelst
```
i fältet för användarnamn inmatning och
```
PASS' OR '4'='4
```
i fältet för lösenord inmatning, kommer inmatade sträng att konkatenera med SQL kommand som finns i koden och koden efter WHERE ord i SQL query kommer att alltid returnera true  (eftersom 4 är lika med 4). Följande linje i koden bygger en SQL query på dynamiskt sätt:
``` javascript
var sqlString = "SELECT * FROM user WHERE email = '" + username +"' AND password = '" + password +"'";
```
Efter injection attacken, kommandot som ska exekvera mot databasen ska se ut som följande:
``` SQL
SELECT * FROM user WHERE email = " dittnamn@vadsomhelst" AND password = "PASS" OR 4 = 4
```
Användaren loggar in utan att ange korrekta uppgifter.

#####Teori om SQL injection

SQL Injection är ett problem som ingår i gruppen Injections. Injection brister som OS injection, SQL injection och LDAP injection sker när opålitlig data skickas till interpretör (till exempel SQL Server) som en del av query eller command [1].

#####Följder som  SQL Injection kan skapa
Användare kan logga in i applikationen utan att det finns registrerade användarnamn och lösenord. Förutom det tydliga problemet, kan elak användare injektera elak kod till SQL kommand och exekvera mer komplexa och mer kritiska commando på databasen. Det går att få en lista över alla användare. Det är möjligt att radera hela tabellen och exekvera DROP kommand på hela databasen.

#####Hur  SQL Injection kan åtgärdas

Det är viktigt att använda parametiserade frågor mot databasen. Alternativet är att använda säkra API-er. Det är rekommenderat att använda sig av positiva eller "white list" för validering av inmatade data, men den metod löser INTE alltid problemet, eftersom vissa applikationer kan kräva speciella karaktärer som inmatade data. Det finns andra preventiva metoder som kan användas. Det är rekommenderat att applikations användare har minsta möjliga rättigheter över databasen som används (skriv och läs rättigheterna) och att kontrollera inmatad data som kan förhindra SQL attack. Jag kan rekommendera att använda "Black list" med tecken som kan användas för sträng konkatenering.

### Cross Site Scripting (XSS) 
##### Problem
Om användare skriver följande meddelande i fältet för meddelande:
``` HTML
<B>XSS IS POSSIBLE</B>
```
Blir resultatet så att meddelandet "XSS IS POSSIBLE" visas i fet format. Det visar att opålitlig data inte är validerad och görs inte _escape_ innan inmattade data används. Med följande enkla inmatning kan vi skriva en länk bland andra meddelanden, och på samma sätt kan vi skriva hela script som kan göra mycket mer skada.
``` HTML
<a href="http://www.webbprogrammerare.se">See your message</a>
```
##### Teori om Cross Site Scripting (XSS) 
Enligt OWASP [2] topp 10 säkerhets problem från 2013, XSS bristen sker då när opålitliga data skickas till webb läsaren utan validering eller _escape_ . På detta sätt kan en elak användare exekvera script i användares webbläsare eller manipulera webbläsare genom att göra _redirect_ till en annan sida som kan exekvera elak kod. [2]

#####Följder som Cross Site Scripting (XSS) kan skapa
Det kan hända att en elak användare kommer åt innehållet av användarens _kakor_. På så sätt kan en elak användare besöka sidor (innehållet av kakor är ofta session ID och liknande data för auktorisering) som är tillgängliga bara för autentiserad användare.

#####Hur Cross Site Scripting (XSS) kan åtgärdas
Viktigt är att validera och göra _escape_ på alla opålitliga data. Enligt OWASP Prevention cheat sheet [3] kan inte alla XSS attack blockeras av _escape_ eftersom vissa javascript funktioner inte kan använda opålitlig data utan att vara i risk av XSS. Användning av "White list" är rekommenderad, men löser inte ALLA problem. _Escape_ alla opålitliga data som innan det används i följande list:
* Sträng
  * HTML Body
  * Säker HTML attributer
  * GET parameter
  * Opålitlig URL i href eller src attribut
  * CSS value
  * Jacascript variable
  * DOM XSS
* HTML
  * HTML Body
[3]

###Autentisering och sessioner
#####Problem

När man loggar ut från messages sida, man kan skriva följande URL igen:
``` 
http://localhost:3000/message
```
och då är man inloggad igen.
Om man försöker med URL:
``` 
http://localhost:3000/static/message.db
```
laddas hela databas fil ner.

#####Teori om autentisering och sessioner
Autentisering, session hantering och andra förknippade funktioner är inte implementerade på korrekt sätt vilket tillåter elaka användaren att äventyra data som session token, user id, lösenord, nycklar o.s.v. [4].

#####Följder som autentisering och sessioner kan skapa
Om man använder en enhet som är publikt, och en annan användare fortsätter att använda den då kan den andra användaren att fortsätta använda applikation som inloggad användare genom att besöka http://localhost:3000/message .

#####Hur autentisering och sessioner kan åtgärdas
Bättre hantering av autentisering och sessioner. Enligt OWASP det finns två möjliga sätt:
*	Stark autentisering och sessioner hantering kontroller
  * Bemöta alla krav av ASVS [5] för hantering av autentisering och sessioner 
  * Ha ett enkelt gränsnitt för utvecklare
* Undvika XSS brister
https://www.owasp.org/index.php/ASVS

### Sensitive Data Exposure
##### Problem
Lösenorden i databasen är lagrade utan hashning eller kryptering.
##### Teori om Sensitive Data Exposure
Enligt OWASP[6] behöver känslig data som, kreditkort nummer, user id och liknande auktoriserings uppgifter speciell behandling som kryptering och hashning när lagrad eller i transfer.
##### Följder som Sensitive Data Exposure kan skapa
Eftersom känslig data är äventyrad, är konsekvenserna stora. Stulna autentiserings uppgifter eller kreditkort nummer ger elaka användare möjligheter för felaktigt autentisering som vidare kan orsaka ytterligare problem.
##### Hur Sensitive Data Exposure kan åtgärdas
Två huvud rekommendationer är:
*	Kryptera all känslig data
* Spara inte känslig data som inte behövs sparas (sessions token, då när session förstörs).

### Insecure Direct Object References
##### Problem
Databas fil nås via URL och laddas ner utan några auktoriserings uppgifter:
``` 
http://localhost:3000/static/message.db
```
##### Teori om Insecure Direct Object References
Insecure Direct Object References är ett säkerhets problem när olika implementerade objekt (text fil, JSON objekt o.s.v.) kan nås via specifikt URL eller som en referens i en formulär och det finns inga auktoriserings förhinder på det objekt.
##### Följder som Insecure Direct Object References kan skapa
Användare som är inte auktoriserad att använda vissa resurser i applikationen, kan nå till resurserna och använda dom på egna villkor. Eftersom det är möjligt att ladda ner databas fil, kan alla användare se alla användarnamn och lösenorden.
##### Hur Insecure Direct Object References kan åtgärdas
Utvecklare ska undvika att ge direkta referenser till vissa objekt (JSON, text filer, databas filer o.s.v.). Istället, ska man använda:
*	Kontroll av tillgänglighet (någon form av auktorisering och rättigheterna)
* Använda indirekta objekt referenser [7]
[7]


##Prestanda
### Inline CSS och Javascript
#####Problem
I applikation messy labbage är CSS och Javascript implementerade direkt i HTML filer. Det gör att applikationen har bättre prestanda (lite bättre) första gång applikation körs av en användare. Långsiktiga resultat ger sämre prestanda.
#####Teori om Inline CSS och Javascript
Enligt Steve Sounders, ska JavaScript och CSS filer separeras i separata filer. Sådana filer blir cashade och det påverkar prestanda positivt. [8]
#####Hur Inline CSS och Javascript åtgärdas
Skriv CSS och Javascript i separata filer och ange referens i HTML fil.

### Put scripts at the bottom 
#####Problem
I applikation messy labbage är Javascript filer Sessage.js och MessageBoard.js implementerade i början av HTML filer. De filer ska länkas in efter _body_ tagg.
#####Teori om Put scripts at the bottom
Enligt Steve Sounders, ska JavaScript filer länkas in efter tagg. Detta tillåter HTML rendera innan skript. I andra fall kan script påverka prestanda negativt, genom att rendering av HTML väntar till skript slutar sin exekvering. [9]
#####Hur Put scripts at the bottom åtgärdas
Placera JavaScript länkar efter _body_ tagg.
###Möjlighten att använda Cache-headers
#####Problem
Expiration header har värde -1 vilket betyder att ingenting spara i cache fil. Allt ladas om vid varje ny POST eller GET.
Vissa filer laddas utan att dem används senare och det finns några 404 anrop mot några filer som inte finns (se bild).

#####Teori om möjligheten att använda Cache-headers
Webbläsare använder cache att reducera antal av HTTP Requests och minska storlek av HTTP Response. Det hjälper webbplatser ladda snabbare. [10]
#####Hur Cache headers problem återgärdas
Genom att förändra Expire värde till passande max-age förhindrar vi att resurserna laddas om varje gång.

##Reflektioner

I min analys kring säkerhetsproblem använde jag mig av OWASP list över topp 10 största säkerhetsproblem från 2013. Mina slutsatser är att applikation messy labbage innehåller oväntade många säkerhetsproblem. Enligt mig, kan vissa säkerhetsproblem ingå i flera kategorier. Vissa av problem kan utnyttjas genom att kombinera flera säkerhetsproblem. Utifrån säkerhets perspektiv är applikationen Messy Labbage absolut inte godkänd för att på något sätt användas som färdig produkt. Prestanda är tillräckligt bra, men finns mycket plats för förbättringar. Applikation är ganska enkel och liten och skillnaderna i prestanda är säkert inte stora (om applikationen skulle blivit utvecklad med rekommenderade metoder för förbättrad prestanda), men finns tydliga brister och punkter när implementering bryter mot rekommenderade utvecklings metoder.

#####Referenser:

[1] OWASP, "Top 10 2013 - A1 Injection", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Online].

[2] OWASP, "Top 10 2013 - A3 Cross-Site Scripting (XSS)", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS) [Online].

[3] OWASP, "XSS Prevention cheat sheet", _OWASP_, september 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet [Online].

[4] OWASP, "Top 10 2013 - A2 Broken Authentication and Session Management", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A2-Broken_Authentication_and_Session_Management [Online].

[5] ASVS, "Application Security Verification Standard", _ASVS_, november 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/ASVS [Online].

[6] OWASP, "Top 10 2013 - A6 Sensitive Data Exposure", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A6-Sensitive_Data_Exposure [Online].

[7] OWASP, "Top 10 2013 - A4 Insecure Direct Object References", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A4-Insecure_Direct_Object_References [Online].

[8] Steve Sounders, "High Performance Web Sites - Rule 8: Make JavaScript and CSS External", O'Reilly, september 2007

[9] Steve Sounders, "High Performance Web Sites - Rule 6: Put Scripts at the Bottom", O'Reilly, september 2007

[10] Steve Sounders, "High Performance Web Sites - Rule 3: Add an Expires Header", O'Reilly, september 2007
