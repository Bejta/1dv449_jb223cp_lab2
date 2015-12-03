

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
Användaren loggar in utan att ange korekta uppgifter.

#####Teori om SQL injection

SQL Injection är ett problem som ingår i gruppen Injections. Injection brister som OS injection, SQL injection och LDAP injection skär när opålitligt data skickas till interpretör (till exempel SQL Server) som en del av query eller command [1].

#####Följder som  SQL Injection kan skapa
Användare kan logga in i applikation utan att det finns registrerad användarsnamn och lösenord. Förutom det tydliga problem, elak användare kan injectera elak kod till SQL kommand och exekvera mer komplexa och mer kritiska commando på databasen. Det går att få en lista över alla användare. Det är möjligt att radera hela tabell och exekvera DROP kommand på hela databas.

#####Hur  SQL Injection kan åtgärdas

Det är viktigt att använda parametiserade frågor mot databasen. Alternativet är att använda säkra API-er. Det är rekommenderad att använda sig av positiva eller "white list" för validering av inmatade data, men den metod löser INTE alltid problemet, eftersom vissa applikation kan kräva speciella karaktär som inmatade data.
Finns andra preventiva metoder som kan användas. Det är rekommenderad att applikations användare har minsta möjliga rättigheter över databas som används (skriv och läs rättigheterna) och att kontrollera inmatad data som kan förhindra SQL attack. Jag kan rekommendera använda "Black list" med tecken som kan användas för sträng konkatenering.

### Cross Site Scripting (XSS) 
##### Problem
Om användare skriver följande meddelande i fältet för meddelande:
``` HTML
<B>XSS IS POSSIBLE</B>
```
Resultat blir så att meddelandet "XSS IS POSSIBLE" visas i fet format. Det visar att opålitligt data är inte validerad och görs inte _escape_ innan inmattade data används.
Med följande enkel inmatning vi kan skriva en länk bland andra meddelanden, och på samma sätt vi kan skriva hela skripter som kan göra mycket mer skada.
``` HTML
<a href="http://www.webbprogrammerare.se">See your message</a>
```
##### Teori om Cross Site Scripting (XSS) 
Enligt OWASP [2] topp 10 säkerhets problem från 2013, XSS brist sker då när opålitligt data skickas till webb läsaren utan validering eller _escape_ . På detta sättet elak användare kan exekvera skripter i användares webbläsare eller manipulera webbläsare genom att göra _redirect_ till en annan sida som kan exekvera elak kód. [2]

#####Följder som Cross Site Scripting (XSS) kan skapa
Det kan hända att elak användare kommer åt innehållet av användare _kakor_ . På så sätt kan elak användare besöka sidor (inehållet av kakor är offta session ID och liknande data för autentisering) som är tillgängliga bara för autentiserad användare.

#####Hur Cross Site Scripting (XSS) kan åtgärdas
Viktigt är att validera och göra _escape_ på alla opålitliga data. Enligt OWASP Prevention cheat sheet [3] alla XXS attack kan inte blockeras av _escape_ eftersom vissa javascript funktioner kan inte använda opåligtlig data utan att vara under risk av XSS.
Användning av "White list" är rekommenderad, men löser inte ALLA problem.
_Escape_ alla opåligtliga data som innan det används i följande list:
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
laddas ner hela databas fil.

#####Teori om autentisering och sessioner
Autentisering, session hantering och andra förknippna funktioner är inte implementerad på korrekt sätt vilket tillåter elaka användaren kompromitera data som session token, user id, lösenord, nycklar o.s.v. [4] .

#####Följder som autentisering och sessioner kan skapa
Om man använder en enhet som är publikt, och en annan användare fortsätter att använda den då kan andra användare fortsätta använda applikation som inloggad användare genom att besöka http://localhost:3000/message .

#####Hur autentisering och sessioner kan åtgärdas
Bättre hantering av autentisering och sessioner.
Enligt OWASP det finns två möjliga sätt:
* Starka autentisering och sessioner hantering kontroller
  * Bemöta alla krav av ASVS [5] för hantering av autentisering och sessioner 
  * Ha ett enkelt gränsnitt
* Undvika XSS brister
https://www.owasp.org/index.php/ASVS

### Sensitive Data Exposure
##### Problem
Lösenorden i databasen är lagrade utan hashning eller encryption.
##### Teori om Sensitive Data Exposure
Enligt OWASP[6] känsliga data som, kreditkort nummer, user id och liknande autentiserings uppgifter behöver speciel behandling som kryptering och hasning när lagrad eller i transfer.
##### Följder som Sensitive Data Exposure kan skapa
Eftersom känslig data är kompromiterad, konsekvenserna är stora. Stulta autentiserings uppgifter eller kreditkort nummer ger elaka användare möjligheter för felaktigt autentisering som kan vidare orsaka ytterligare problem.
##### Hur Sensitive Data Exposure kan åtgärdas
Två huvud rekommenderingar är:
* Kryptera alla känsliga data
* Spara inte känsliga data som behövs inte sparas (sessions token, då när session förstörs).

### Insecure Direct Object References
##### Problem
Databas fil nås via URL och laddas ner utan några aukterisering uppgifter:
``` 
http://localhost:3000/static/message.db
```
##### Teori om Insecure Direct Object References
Insecure Direct Object References är ett säkerhets problem när olika implementerade objekt (text fil, JSON objekt o.s.v.) kan nås via specifiskt URL eller som en referens i en formulär, och det finns inga auktoriserings förhinder på det objekt.
##### Följder som Insecure Direct Object References kan skapa
Användare som är inte auktoriserad att använda vissa resurser i applikation, kan nå till resurserna och använda dom på egna vilkor.
Eftersom det är möjligt att ladda ner databas fil, alla användare kan se alla användarsnamn och lösenorden.
##### Hur Insecure Direct Object References kan åtgärdas
Utvecklare ska undvika ge direkta referenser till vissa objekt (JSON, text filer, databas filer o.s.v.).
Istället, man ska använda:
* Kontroll av access (något form av auktorisering och rättigheterna)
* Använda indirekta objekt referenser
[7]


##Prestanda
### Inline CSS och Javascript
#####Problem
I applikation messy labbage CSS och Javascript är implementerade direkt i HTML filer. Det gör att applikation har bättre prestanda (lite bättre) första gång applikation körs av en användare. Långsiktiga resultat ger sämre prestanda. 
#####Teori om Inline CSS och Javascript
Enligt Steve Sounders, JavaScript och CSS filer ska separeras i separata filer. Sådana filer blir cashade och det påverkar prestanda positivt. [8]
#####Hur Inline CSS och Javascript åtgärdas
Skriv CSS och Javascript i separata filer och ange referens i HTML fil.

### Put scripts at the bottom 
#####Problem
I applikation messy labbage  Javascript filer Sessage.js och MessageBoard.js är implementerade i början av HTML filer.
Dem filer ska länkas in efter </body> tagg.
#####Teori om Put scripts at the bottom
Enligt Steve Sounders, JavaScript  filer ska länkas in efter </body> tagg. Detta tillåter HTML randera innan skript. I andra fall skript kan inflytta prestanda negativt, genom att rendering av HTML väntar till skript slutar sin exekvering. [9]
#####Hur Put scripts at the bottom åtgärdas
Placera JavaScript länkar efter body tagg.

##Reflektioner

I min säkerhetsproblem analys använde jag mig av OWASP list över topp 10 största säkerhetsproblem från 2013. Mina slutsatser är att applikation messy labbage innehåller oväntade många säkerhetsproblem. Enligt mig, vissa säkerhetsproblem ingår i flera kateogier. Vissa av problem kan utnyttjas genom att kombinera flera säkerhetsproblem. Utifrån säkerhets perspektiv applikationen Messy Labbage är absolut inte godkänd för att på något sätt användas som färdigt produkt.
Prestanda är tillräckligt bra, men finns mycket plats för förbättringar. Applikation är ganska enkelt och liten, och skillnader i prestanda är säkert inte stora (om applikationen skulle blivit utvecklad med rekommenderade metoder för förbättrad prestanda), men finns tydliga brister och punkter när implementering bryter mot rekommenderade utvecklings metoder.

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
