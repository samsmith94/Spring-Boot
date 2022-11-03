# Spring-Boot
## Biztonság
### Alapfogalmak
* autentikáció: a felhasználó atonosítása, pl. felhasználónév-jelszó páros, szoftveres/hardveres kulcs, ujjlenyomat, stb. alapján
* autorizáció: annak ellenőrzése, hogy az adott felhasználó jogosult-e az adott művelet elvégzésére
* adat integritás: az adatok módosítása detektálható
* adatok bizalmassága: csak az arra jogosultak tudják olvasni az adatokat
* letagadhatatlanság: bizonyítható, hogy egy adott felhasználó elvégzett valamilyen műveletet a rendszerben
```java

```

### Servlet API alapok (HttpSession, Filter)
#### HTTP Session
* Kérések sorozata közötti állapot megőrzése igény lehet (pl. belépett fehasználó azonosítója)
* Ugyanakkor a HTTP állapotmentes
* Megoldások:
  * HTTP cookie (neve by default? JSESSIONID)
  * URL rewrite (response.encodeURL())
  * Hidden FORM mező
* Session timeout
  * általában 30 perc a default
* Lejárat: időlimit vagy session.invalidate()
* Elérése:
  * HttpSession session = request.getSession();
  * session.setAttribute("currentUser", myUser);
  * MyUser user = (MyUser)session.getAttribute("currentUser");
* A session fontos erőforrás: minden belehelyezett objektum a szerver memóriahasználatát növeli
* Amire ügyelni kell:
  * A Session objektum megosztott, több szálon kezelhetik
  * Elosztott környezet esetén csak szerializálható objektumokat tegyünk bele, különben elveszik

#### Szervelet szűrők
* Java Servlet Filters (Servlet API 2.3)
* Cél: megfigyelni, módosítani, közbeavatkozni a kérés illetve a válasz alapján
* A szűrőket láncba szervezhetjük
* Használati esetek:
  * Hozzáférés biztosítása, blokkolása
  * Cache, tömörítés, naplózás, titkosítás
  * Tartalom transzformációja
### Spring Security (autentikáció és autorizáció)

### REST autentikáció és autorizáció
