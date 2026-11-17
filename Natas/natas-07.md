# Natas Level 7 → Level 8

> ⚠️ Spoilers: las contraseñas están omitidas (`[redacted]`). Lo importante es el método.

- **URL:** `http://natas7.natas.labs.overthewire.org`
- **Credenciales:** `natas7` / `[redacted]`

## Análisis y Explotación
1. La aplicación cargaba las vistas con un parámetro GET: `index.php?page=home`.
2. Un comentario en el código fuente indicaba que la contraseña de `natas8` estaba en `/etc/natas_webpass/natas8`.
3. Como `page` se usa directamente para incluir archivos (**Local File Inclusion**), bastó con pasarle la ruta absoluta:
   ```
   http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
   ```
4. El servidor incluyó el archivo y mostró su contenido en la página.

- **Password obtenida (`natas8`):** `[redacted]`

## Lección aprendida
Nunca pasar entrada del usuario a funciones de inclusión (`include`, `require`). Usar una lista blanca de páginas permitidas.
