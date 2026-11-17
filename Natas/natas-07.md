# Natas 07 → 08: Local File Inclusion (LFI)

[← Nivel 6](natas-06.md) · [Índice](../README.md) · [Nivel 8 →](natas-08.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Inclusión de archivos |
| **Debilidad** | [CWE-98](https://cwe.mitre.org/data/definitions/98.html): Improper Control of Filename for Include/Require Statement in PHP Program |
| **Herramientas** | Navegador |
| **URL** | `http://natas7.natas.labs.overthewire.org` |
| **Credenciales** | `natas7` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El parámetro `page` se pasa sin validar a una función de inclusión, lo que permite leer archivos arbitrarios del servidor.

## Análisis y explotación
1. La aplicación cargaba las vistas con un parámetro GET: `index.php?page=home`.
2. Un comentario en el código fuente indicaba que la contraseña de `natas8` estaba en `/etc/natas_webpass/natas8`.
3. Como `page` se usa directamente para incluir archivos (**Local File Inclusion**), bastó con pasarle la ruta absoluta:
   ```
   http://natas7.natas.labs.overthewire.org/index.php?page=/etc/natas_webpass/natas8
   ```
4. El servidor incluyó el archivo y mostró su contenido en la página.

## Resultado

- **Password obtenida (`natas8`):** `[redacted]`

## Lección aprendida y mitigación
Nunca pasar entrada del usuario a funciones de inclusión (`include`, `require`). Usar una lista blanca de páginas permitidas.

## Referencias
- [CWE-98](https://cwe.mitre.org/data/definitions/98.html): Improper Control of Filename for Include/Require Statement in PHP Program
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 6](natas-06.md) · [Índice](../README.md) · [Nivel 8 →](natas-08.md)
