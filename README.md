## PortSwigger Academy -  Lab: CSRF where token validation depends on request method
## 📚 Studijní materiál
Tento repozitář slouží jako můj osobní studijní zápisník a přehled řešení laboratoří z platformy PortSwigger Web Security Academy. Dokumentuji zde postupy a hledání netradičních míst, kde dochází k reflexi uživatelského vstupu.

Přihlášení a zadání e-mailu: Přihlásil jsem se pomocí údajů wiener a peter. V sekci My Account jsem viděl aktuální e-mail (wiener@normal-user.net), který jsem přepsal na flip@hudson.com a potvrdil tlačítkem.

Zachycení v Burpu: Zapnul jsem Burp Suite a v HTTP history našel požadavek na změnu e-mailu. Poslal jsem ho do Repeateru, kde úplně poslední řádek vypadal takto: email=flip%40hudson.com&csrf=bpyxT4V04ZqNPp4M4Z1oJpiYYG4h1ZGm.

Otestování chyby v tokenu schválně: Zkusil jsem v tom řádku přepsat hodnotu tokenu na nesmysl (csrf=12345) a dal Send. Dole v odpovědi vyskočilo hlášení o neplatném CSRF tokenu (invalid csrf token).

Změna metody požadavku: Kliknul jsem v okně Request pravým tlačítkem myši, vybral možnost Change request method (čímž se požadavek přepnul z POST na GET) a dal Send. Najednou to prošlo bez chyby!

Vygenerování exploitu: Kliknul jsem v Repeateru pravým tlačítkem na požadavek, vybral Engagement tools -> Generate CSRF PoC. V okně jsem v URL hodnotě u e-mailu upravil jméno na flip@hudson.com, zaškrtnul volbu Include auto-submit script, kliknul na Regenerate a pak na Copy HTML.

Vložení na Exploit server: Šel jsem do laboratoře, otevřel Exploit server a do pole Body vložil následující kód:

HTML
<html>
    <body>
        <script>history.pushState('', '', '/');</script>
        <form action="https://0a9d00ce03a4689480f6033300e20003.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="flip&#64;hudson&#46;com" />
            <input type="hidden" name="csrf" value="12345" />
            <input type="submit" value="Submit request" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
Dokončení: Kliknul jsem na Store a hned potom na Deliver exploit to victim – lab byl úspěšně vyřešen.

## TECHNICKÉ VYSVĚTLENÍ: PROČ TO FUNGOVALO?

Chybná validace podle metody: Server sice kontroloval platnost CSRF tokenu, ale dělal to jen u požadavků typu POST. Jakmile byl požadavek odeslán jako GET, server kontrolu tokenu úplně přeskočil a vyhověl mu i s neplatným tokenem (12345).

Zneužití formuláře: I když server přijímal GET požadavky bez tokenu, běžný HTML formulář posílá data metodou POST. Pomocí skriptu na Exploit serveru se ale požadavek na pozadí odeslal a aplikoval požadovanou změnu e-mailu na oběť.