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

### Spring Security (autentikáció és autorizáció)

### REST autentikáció és autorizáció
