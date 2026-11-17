# Natas 03 → 04: Ruta oculta revelada por `robots.txt`

[← Nivel 2](natas-02.md) · [Índice](../README.md) · [Nivel 4 →](natas-04.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Exposición de información |
| **Debilidad** | [CWE-200](https://cwe.mitre.org/data/definitions/200.html): Exposure of Sensitive Information to an Unauthorized Actor |
| **Herramientas** | `gobuster` |
| **URL** | `http://natas3.natas.labs.overthewire.org` |
| **Credenciales** | `natas3` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El archivo `robots.txt` revela un directorio "oculto" que contiene la credencial del siguiente nivel.

## Análisis y explotación
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

## Resultado

- **Password obtenida (`natas4`):** `[redacted]`

## Lección aprendida y mitigación
`robots.txt` es un archivo público y no es un control de acceso: listar una ruta en él la anuncia en lugar de ocultarla. Las rutas sensibles deben protegerse con autenticación y autorización, no con oscuridad.

## Referencias
- [CWE-200](https://cwe.mitre.org/data/definitions/200.html): Exposure of Sensitive Information to an Unauthorized Actor
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 2](natas-02.md) · [Índice](../README.md) · [Nivel 4 →](natas-04.md)
