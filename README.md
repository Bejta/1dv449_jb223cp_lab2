##### _Jasmin Bejtovic-jb223cp_

#Messy Labbage

##Säkerhet
#### SQL Injection
##### Problem

Genom att skriva:
```
dittnamn@vadsomhelst
```
i fältet för användarnamn inmatning och
```
PASS' OR '4'='4
```
i fältet för lösenord inmatning, kommer inmatade sträng att konkatenera med SQL kommand som finns i kóden och kóden efter WHERE ord i SQL query kommer att returnera true alltid (eftersom 4 är lika med 4).
Följande linje i kod bygger en SQL query på dynamiskt sätt:
``` javascript
var sqlString = "SELECT * FROM user WHERE email = '" + username +"' AND password = '" + password +"'";
```
Efter injection attack, kommandot som ska exekvera mot databasen ska se följande ut:
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

#### Cross Site Scripting (XSS) 
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



####Problem 3

######Teori om problem 3

######Följder som problem kan skapa

######Hur problemet kan åtgärdas

##Prestanda

##Reflektioner

#####Referencer:

[1] OWASP, "Top 10 2013 - A1 Injection", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Hämtad: 29 november, 2015].

[2] OWASP, "Top 10 2013 - A3 Cross-Site Scripting (XSS)", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A3-Cross-Site_Scripting_(XSS) [Hämtad: 29 november, 2015].

[3] OWASP, "XSS Prevention cheat sheet", _OWASP_, september 2015 [Online] Tillgänglig: https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet [Online].
