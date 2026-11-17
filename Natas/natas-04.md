# Natas Level 4 → Level 5

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas4.natas.labs.overthewire.org`
- **Credenciales:** `natas4` / `[redacted]`

## Análisis y Explotación
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

## Lección aprendida
El header `Referer` lo controla el cliente, por lo que **nunca debe usarse como mecanismo de autorización**: cualquiera puede falsificarlo.

# Writeup: OverTheWire - Natas (Niveles 5 a 9)

# Introducción
Continuación del writeup de **Natas**. Estos niveles pasan de la información expuesta a fallos del lado del servidor: manipulación de cookies, archivos incluidos accesibles, Local File Inclusion (LFI), codificación reversible de secretos e inyección de comandos.

> Las contraseñas y los secretos están omitidos (`[redacted]`) para no arruinar el reto a otras personas. Lo importante es el método.
