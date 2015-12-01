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
```
SELECT * FROM user WHERE email = " dittnamn@vadsomhelst" AND password = "PASS" OR 4 = 4
```
Användaren loggar in utan att ange korekta uppgifter.

#####Teori om SQL injection

SQL Injection är ett problem som ingår i gruppen Injections. Injection brister som OS injection, SQL injection och LDAP injection skär när opålitligt data skickas till interpretör (till exempel SQL Server) som en del av query eller command [1].

#####Följder som  SQL Injection kan skapa
Användare kan logga in i applikation utan att det finns registrerad användarsnamn och lösenord. Förutom det tydliga problem, elak användare kan injectera elak kod till SQL kommand och exekvera mer komplexa och mer kritiska commando på databasen. Det går att få en lista över alla användare. Det är möjligt att radera hela tabell och exekvera DROP kommand på hela databas.

#####Hur  SQL Injection kan åtgärdas

Det är viktigt att använda parametiserade frågor mot databasen. Alternativet är att använda säkra API-er. Det är rekommenderad att använda sig av positiva eller "white list" för validering av inmatade data, men den metod löser INTE alltid problemet, eftersom vissa applikation kan kräva speciella karaktär som inmatade data.
Finns andra preventiva metoder som kan användas. Det är rekommenderad att applikation användare har minska möjliga rättigheter över databas som används (skriv och läs rättigheterna) och att kontrollera inmatad data som kan förhindra SQL attack. Jag kan rekommendera använda "Black list" med tecken som kan användas för sträng konkatenering.

#### Cross Site Scripting (XSS) 

##### Problem
##### Teori om Cross Site Scripting (XSS) 



######Följder som Cross Site Scripting (XSS) kan skapa

######Hur Cross Site Scripting (XSS) kan åtgärdas

####Problem 3

######Teori om problem 3

######Följder som problem kan skapa

######Hur problemet kan åtgärdas

##Prestanda

##Reflektioner

#####Referencer:

[1] OWASP, "Top 10 2013 - A1 Injection", _OWASP_, juni 2013 [Online] Tillgänglig: https://www.owasp.org/index.php/Top_10_2013-A1-Injection [Hämtad: 29 november, 2015].
