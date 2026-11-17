# Natas Level 3 → Level 4

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas3.natas.labs.overthewire.org`
- **Credenciales:** `natas3` / `[redacted]`

## Análisis y Explotación
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
