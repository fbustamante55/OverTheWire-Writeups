# Writeup: OverTheWire - Natas (Niveles 5 a 9)

## Introducción
Continuación del writeup de **Natas**. Estos niveles pasan de la información expuesta a fallos del lado del servidor: manipulación de cookies, archivos incluidos accesibles, Local File Inclusion (LFI), codificación reversible de secretos e inyección de comandos.

> Las contraseñas y los secretos están omitidos (`[redacted]`) para no arruinar el reto a otras personas. Lo importante es el método.

---

## Natas Level 5 → Level 6
- **URL:** `http://natas5.natas.labs.overthewire.org`
- **Credenciales:** `natas5` / `[redacted]`

### Análisis y Explotación
1. La aplicación bloqueaba el contenido con el mensaje *"You are not logged in"*.
2. Al inspeccionar las cookies del navegador (DevTools → Application/Storage) apareció una cookie `loggedin` con valor `0`.
3. Se cambió el valor a `1` y se recargó la página: el servidor dio la sesión por válida y mostró la contraseña.

Equivalente con `curl`:
```bash
curl -u natas5:'[redacted]' --cookie "loggedin=1" http://natas5.natas.labs.overthewire.org/
```

- **Password obtenida (`natas6`):** `[redacted]`

### Lección aprendida
El estado de autenticación no puede depender de un valor que el cliente puede editar. Hay que usar sesiones gestionadas por el servidor.

---

## Natas Level 6 → Level 7
- **URL:** `http://natas6.natas.labs.overthewire.org`
- **Credenciales:** `natas6` / `[redacted]`

### Análisis y Explotación
1. La página pedía un "secret" y ofrecía un enlace a `index-source.html` con el código PHP.
2. El código mostraba que el secreto se cargaba con `include "includes/secret.inc";`.
3. Al abrir directamente `http://natas6.natas.labs.overthewire.org/includes/secret.inc` el servidor devolvía el archivo sin interpretarlo, con la variable `$secret` en texto plano.
4. Se envió ese secreto en el formulario y la aplicación entregó la contraseña.

- **Secret:** `[redacted]`
- **Password obtenida (`natas7`):** `[redacted]`

### Lección aprendida
Los archivos incluidos no deben estar dentro del directorio web accesible, o deben ser ejecutables (`.php`) para que no se sirvan como texto.

---

## Natas Level 7 → Level 8
- **URL:** `http://natas7.natas.labs.overthewire.org`
- **Credenciales:** `natas7` / `[redacted]`

### Análisis y Explotación
1. La aplicación cargaba las vistas con un parámetro GET: `index.php?page=home`.
2. Un comentario en el código fuente indicaba que la contraseña de `natas8` estaba en `/etc/natas_webpass/natas8`.
3. Como `page` se usa directamente para incluir archivos (**Local File Inclusion**), bastó con pasarle la ruta absoluta:
   ```
   http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
   ```
4. El servidor incluyó el archivo y mostró su contenido en la página.

- **Password obtenida (`natas8`):** `[redacted]`

### Lección aprendida
Nunca pasar entrada del usuario a funciones de inclusión (`include`, `require`). Usar una lista blanca de páginas permitidas.

---

## Natas Level 8 → Level 9
- **URL:** `http://natas8.natas.labs.overthewire.org`
- **Credenciales:** `natas8` / `[redacted]`

### Análisis y Explotación
1. El código fuente de la página mostraba cómo se comparaba el secreto: el valor se ofuscaba con
   ```php
   bin2hex(strrev(base64_encode($secret)))
   ```
   y se comparaba contra un valor ya codificado (`[redacted]`).
2. Basta con aplicar las operaciones **en orden inverso**: `hex2bin` → `strrev` → `base64_decode`. Con PHP en la terminal:
   ```bash
   php -r 'echo base64_decode(strrev(hex2bin("[valor codificado]")));'
   ```
3. El resultado fue el secreto en texto plano (`[redacted]`). Al enviarlo en el formulario, la aplicación mostró la contraseña.

- **Password obtenida (`natas9`):** `[redacted]`

### Lección aprendida
Base64, hexadecimal o invertir una cadena son **codificaciones, no cifrado**: no dan confidencialidad. Para verificar secretos se deben usar hashes con sal (por ejemplo Argon2id o bcrypt).

---

## Natas Level 9 → Level 10
- **URL:** `http://natas9.natas.labs.overthewire.org`
- **Credenciales:** `natas9` / `[redacted]`

### Análisis y Explotación
1. La página era un buscador de palabras. Su código fuente mostraba que el parámetro `needle` se insertaba directamente en un comando de shell:
   ```php
   passthru("grep -i $key dictionary.txt");
   ```
2. Sin ninguna sanitización, se puede inyectar un separador de comandos (`;`) para ejecutar un segundo comando. Payload en el campo de búsqueda:
   ```
   ;cat /etc/natas_webpass/natas10
   ```
3. El servidor ejecutó `grep -i ` (sin argumento) y luego el `cat`, mostrando la contraseña en la respuesta.

- **Password obtenida (`natas10`):** `[redacted]`

### Lección aprendida
Nunca pasar entrada del usuario a funciones de ejecución de comandos (`passthru`, `exec`, `system`). Si es inevitable, validar con lista blanca y escapar con `escapeshellarg()`.

---

## Conclusiones y recomendaciones

- **Sanitización de entradas:** no pasar variables del usuario a funciones de consola ni de inclusión; usar listas blancas.
- **Autenticación robusta:** no basar la seguridad en valores del cliente (cabecera `Referer`, cookies simples como `loggedin=1`). Usar sesiones del servidor con cookies `HttpOnly` y `Secure`.
- **Cifrado vs. ofuscación:** Base64, hex o inversión de cadenas no protegen secretos. Usar hashes con sal.
