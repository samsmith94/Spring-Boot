# Spring-Boot

[# Table of contents](https://luciopaiva.com/markdown-toc/)
működik, csak javítani kell a magyar karaktereket pl. példa helyett plda-t generált, ezért nem ugrik oda

# Table of contents

- [Spring-Boot](#spring-boot)
  - [Biztonság](#biztonság)
    - [Alapfogalmak](#alapfogalmak)
    - [Servlet API alapok (HttpSession, Filter)](#servlet-api-alapok-httpsession-filter)
      - [HTTP Session](#http-session)
      - [Szervelet szűrők](#szervelet-szűrők)
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

##### Saját login form és adatbázis alapú autentikáció
```java
  .formLogin()
  .loginPage("/login")
  .defaultSuccessUrl("/index")
  .permitAll()
```
###### Saját login form használata
* A /login-ra küldött GET kérés feladata a login form generálása (ide irányít át, ha login szükséges)
* A /login URL-re kell POST-olni a felhasználónév/jelszó párost (ha nincs `loginProcessingUrl()` konfig)
* Sikertelen login kísérlet esetén a /login?error URL-re irányít (ha nincs `failureUrl()`)
* Logout után a /login?logout URL-re irányít
* A `permitAll()` azért szükséges, hogy a login page elérhető legyen bejelentkezés nélkül is (így is lehetne: `http.antMatchers("/login*").permitAll()`)
* Sikeres login esetén oda irányít, ahonnan a login oldalra lettünk irányítva. De ha közvetlenül adtuk meg a login oldalt, a `defaultSuccessUrl()` definiálja a login utáni kezdő oldalt

###### Saját login form
* Bárhogy kinézhet, de kötött
  * a form action attribútuma: contextRoot + "/login"
    * vagy a loginProcessingUrl() értéke
  * a felhasználónév és a jelszó input neve:
    * "username" és "password"
      * vagy a userNameParameter() ill. passwordParameter() értéke
* A kijelentkezést ez az URL végzi el: "logout"
  * de csak POST kérésre (CSRF védelem)
  * testre szabási lehetőségek:
    * logoutUrl()
    * logoutSuccessUrl()

###### Adatbázis alapú autentikáció
* Az első példában a konfig fájlba voltak beleégetve a user adatok
Valós környezetben ezek máshol vannak tárolva, elérésükhöz megfelelő AuthenticationProvider-t kell beállítani, pl.
```java
@Autowired
private DataSource dataSource;

@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth.
    .jdbcAuthentication()
    .dataSource(dataSource)
    .withDefaultSchema();
}
```
* A withDefaultSchema() létrehozza a szükséges táblákat:
  * Felhasználók: users(username, password, enabled)
  * Szerepek: authorities(username, authority)
  * Csoportok: groups(id, group_name)
  * group_authorities(group_id, authority)
  * group_members(id, username, group_id)
* Ha ez nem megfelelő, testreszabható, pl. usersByUserNameQuery(), authoritiesByUserNameQuery(), groupAuthoritiesByUsername()


#### Autorizáció
### REST autentikáció és autorizáció
