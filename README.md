# Spring-Boot

# Table of contents

- [Spring-Boot](#spring-boot)
  - [Biztonság](#biztonsg)
    - [Alapfogalmak](#alapfogalmak)
    - [Servlet API alapok (HttpSession, Filter)](#servlet-api-alapok-httpsession-filter)
      - [HTTP Session](#http-session)
      - [Szervelet szűrők](#szervelet-szrk)
    - [Spring Security (autentikáció és autorizáció)](#spring-security-autentikci-s-autorizci)
      - [Autentikáció](#autentikci)
      - [Autentikációs lehetőségek](#autentikcis-lehetsgek)
        - [Példa](#plda)
        - [Jelszavak kezelése](#jelszavak-kezelse)
      - [Autentikációs felület testre szabása](#autentikcis-fellet-testre-szabsa)
      - [Autorizáció](#autorizci)
    - [REST autentikáció és autorizáció](#rest-autentikci-s-autorizci)
    
## Biztonság
### Alapfogalmak
* autentikáció: a felhasználó atonosítása, pl. felhasználónév-jelszó páros, szoftveres/hardveres kulcs, ujjlenyomat, stb. alapján
* autorizáció: annak ellenőrzése, hogy az adott felhasználó jogosult-e az adott művelet elvégzésére
* adat integritás: az adatok módosítása detektálható
* adatok bizalmassága: csak az arra jogosultak tudják olvasni az adatokat
* letagadhatatlanság: bizonyítható, hogy egy adott felhasználó elvégzett valamilyen műveletet a rendszerben

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
* A szűrő tevékenysége:
  * Headerek vizsgálata
  * Request objektum átalakítása
  * Következő szűrő meghívása
  * Response objektum átalakítása
  * Szűrő lánc megszakítása (blokkolás), hívás átirányítás
  * Kivétel dobása
* Szűrők sorrendje:
  * A web.xml-ben a <filter> definíciók sorrendje
  * Az URL leképzésektő független
* Az elsőt a konténer hívja, az utolsó a szervletet(service()) hívja

```java
@Webfilter("/*")
public class MyFilter implements Filter {
  public void doFilter() {
    if (servletRequest instanceof HttpServletRequest) {
      HttpServletRequest request = (HttpServletRequest) servletRequest;
      HttpServletResponse response = (HttpServletResponse) servletResponse;
      if (request.getParameter("bu") == null) {
        return;
      }
    }
    filterChain.doFilter(servletRequest, servletResponse);
  }
}
```

### Spring Security (autentikáció és autorizáció)
* Autentikációt és autorizációt támogató keretrendszer
* Rugalmasabb, mint a Java EE szabványban benne lévő JAAS (Java Authentication and Authorizatrion Service)
* Spring-et használ, de bármilyen (nem Spring-es) alkalmazásban felhasználható
  * De Spring Boot környezetben kevesebb konfig szükséges
* Őse az Acegi Security System
#### Autentikáció
* Ha védett URL-hez akar hozzáférni a user, és még nincs autentikálva, bejelentkezteti a Spring Security
* Sikeres login esetén az eredetileg kért oldalra irányít vissza
#### Autentikációs lehetőségek
* Autentikációs adatok bekérése:
  * Saját form
  * Böngésző által feldobott ablak
    * HTTP BASIC
    * HTTP DIGEST
* Autentikációs adatok tárolása:
  * Relációs adatbázisban (JDBC)
    * jelszó akár hash-elve
  * LDAP
  * JAAS
  * X.509 digitális igazolvány
  * OpenID
  * OAuth
##### Példa
```java
@Configuration
@EnableWebSecurity
public class AppConfig extends WebSecurityConfigurerAdapter {
  @Override
  public void configure(AuthenticationManagerBuilder auth) throws Exception
  {
    auth.inMemoryAuthentication()
      .withUser("gabor").password("pass").roles("USER", "ADMIN")
      .and()
      .withUser("bob").password("pass").roles("ADMIN")
  }
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .antMatchers("/admin/*").hasRole("ADMIN")
      .anyRequest().hasRole("USER");
  }
}
```
##### Jelszavak kezelése
* Spring Security 5 kötelező PasswordEncoder definiálása
  * Ez lehet akár a NoOpPasswordEncoder is, de explicit kérni kell
  * Tipikus választás: BCryptEncoder
```java
@Bean
public PasswordEncoder encoder() {
  return new BCryptPasswordEncoder();
}

...
@Configuration
@EnableWebSecurity
public class AppConfig extends WebSecurityConfigurerAdapter {
  @Override
  public void configure(AuthenticationManagerBuilder auth) throws Exception
  {
    auth.inMemoryAuthentication()
      .passwordEncoder(encoder())
      .withUser("gabor").password("pass").roles("USER", "ADMIN")
      .and()
      .withUser("bob").password("pass").roles("ADMIN")
      ...
...
```
#### Autentikációs felület testre szabása
* Az előző példában a spring security egy saját formján jelentkeztet be
* Ha a böngésző saját ablakát akarjuk használni, HTTP BASIC autentikációval: A HTTP BASIC autentikáció base64 kódolva küldi a jelszót, tehát a hálózatot lehallgatva visszafejthető (a HTTPS véd ellene)
* A HTTP DIGEST autentikáció is beállítható, ekkor challenge-response alapú az autentikáció, maga a jelszó nem utazik a hálózaton
  * Konfigja kicsit nehézkes (egy spring security átal nyújtott filtert kell beállítani)
* http.httpBasic();


#### Autorizáció
### REST autentikáció és autorizáció
