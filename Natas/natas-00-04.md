# Writeup: OverTheWire - Natas (Niveles 0 a 4)

## Introducción
Este writeup documenta la resolución de los primeros niveles del wargame **Natas** de OverTheWire, enfocado en seguridad de aplicaciones web. Cada nivel aborda diferentes vectores de ataque: desde el análisis estático de código fuente hasta vulnerabilidades de control de acceso y manipulación de cabeceras HTTP.

> Las contraseñas están omitidas (`[redacted]`) para no arruinar el reto a otras personas. Lo importante es el método.

---

## Natas Level 0 → Level 1
- **URL:** `http://natas0.natas.labs.overthewire.org`
- **Credenciales iniciales:** `natas0` / `natas0`

### Análisis y Explotación
Al ingresar al sitio, se revisó el código fuente de la página (`Ctrl + U`), encontrando la contraseña del siguiente nivel oculta en un comentario HTML.

- **Password obtenida (`natas1`):** `[redacted]`

---

## Natas Level 1 → Level 2
- **URL:** `http://natas1.natas.labs.overthewire.org`
- **Credenciales:** `natas1` / `[redacted]`

### Análisis y Explotación
Siguiendo la misma dinámica, la revisión del código fuente expuso un comentario HTML que contenía directamente la contraseña para el Nivel 2.

- **Password obtenida (`natas2`):** `[redacted]`

---

## Natas Level 2 → Level 3
- **URL:** `http://natas2.natas.labs.overthewire.org`
- **Credenciales:** `natas2` / `[redacted]`

### Análisis y Explotación
1. Se revisó el código fuente y no había ninguna contraseña a la vista, así que se probó con otras herramientas, empezando por `gobuster`.
2. El primer intento falló: `gobuster` se detuvo con un error indicando que el servidor devolvía `401` incluso para URLs inexistentes (comportamiento "wildcard"). Esto se debe a que la página está protegida con autenticación HTTP y la herramienta no estaba enviando credenciales.
3. Mientras tanto, el código fuente sí mostraba una imagen incrustada apuntando a `files/pixel.png`, lo que delató la existencia del directorio `/files/`.
4. Al navegar a `/files/` (con las credenciales del nivel), el servidor tenía el listado de directorio activado y mostraba un archivo `users.txt` que exponía las credenciales de múltiples usuarios, incluida la de `natas3`.

- **Password obtenida (`natas3`):** `[redacted]`

---

## Natas Level 3 → Level 4
- **URL:** `http://natas3.natas.labs.overthewire.org`
- **Credenciales:** `natas3` / `[redacted]`

### Análisis y Explotación
1. El código fuente contenía una pista sarcástica: *"Not even Google will find it this time..."*, que apunta a un archivo `robots.txt`.
2. Primer intento con `gobuster` **sin credenciales** (usando `-b 401` para ignorar ese código): solo aparecieron respuestas `403` de archivos `.ht*` y no se encontró nada útil. El problema era que, sin autenticación, el servidor responde `401` a todo.
3. Segundo intento, ahora **autenticándose** con las credenciales de `natas3`:
   ```bash
   gobuster dir -u http://natas3.natas.labs.overthewire.org/ -w common.txt -U natas3 -P '[redacted]'
   ```
   Resultado relevante: `index.html` (200) y `robots.txt` (200), además de `cgi-bin/` y archivos `.ht*` con `403`.
4. Se abrió `robots.txt`, cuyo contenido era:
   ```
   User-agent: *
   Disallow: /s3cr3t/
   ```
   El archivo, pensado para que los buscadores no indexen esa ruta, terminó revelando el directorio oculto `/s3cr3t/`. Al acceder a él, se obtuvo un archivo con la credencial del siguiente nivel.

- **Password obtenida (`natas4`):** `[redacted]`

---

## Natas Level 4 → Level 5
- **URL:** `http://natas4.natas.labs.overthewire.org`
- **Credenciales:** `natas4` / `[redacted]`

### Análisis y Explotación
1. La aplicación web mostraba el mensaje:
   `Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"`
2. Por costumbre se probó primero `gobuster` (con credenciales de `natas4`), pero solo apareció `index.php` (200) y rutas `403`: no había directorios ocultos. La pista estaba en el propio mensaje, no en el servidor de archivos.
3. Al pulsar "Refresh page" el enlace llevaba a `index.php`, y la página insistía en saber *desde dónde* se llegaba. Eso apunta a la cabecera HTTP `Referer`, con la que el navegador indica la página de origen.
4. Utilizando Burp Suite (Repeater), se interceptó la petición `GET /index.php` y se modificó manualmente el encabezado `Referer` para que apuntara a `http://natas5.natas.labs.overthewire.org/`.
5. Al enviar la petición modificada (`Send`), el servidor respondió `200 OK` con el mensaje "Access granted" y la contraseña del siguiente nivel.

Equivalente desde la terminal con `curl`:
```bash
curl -u natas4:'[redacted]' -e "http://natas5.natas.labs.overthewire.org/" http://natas4.natas.labs.overthewire.org/
```

- **Password obtenida (`natas5`):** `[redacted]`

### Lección aprendida
El header `Referer` lo controla el cliente, por lo que **nunca debe usarse como mecanismo de autorización**: cualquiera puede falsificarlo.
