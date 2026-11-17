# Natas 06 → 07: Archivo incluido accesible desde la web

[← Nivel 5](natas-05.md) · [Índice](../README.md) · [Nivel 7 →](natas-07.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Exposición de información |
| **Debilidad** | [CWE-219](https://cwe.mitre.org/data/definitions/219.html): Storage of File with Sensitive Data Under Web Root |
| **Herramientas** | Navegador |
| **URL** | `http://natas6.natas.labs.overthewire.org` |
| **Credenciales** | `natas6` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El secreto que valida el formulario vive en un archivo `.inc` dentro del directorio web, y el servidor lo entrega como texto plano.

## Análisis y explotación
1. La página pedía un "secret" y ofrecía un enlace a `index-source.html` con el código PHP.
2. El código mostraba que el secreto se cargaba con `include "includes/secret.inc";`.
3. Al abrir directamente `http://natas6.natas.labs.overthewire.org/includes/secret.inc` el servidor devolvía el archivo sin interpretarlo, con la variable `$secret` en texto plano.
4. Se envió ese secreto en el formulario y la aplicación entregó la contraseña.

## Resultado

- **Secret:** `[redacted]`
- **Password obtenida (`natas7`):** `[redacted]`

## Lección aprendida y mitigación
Los archivos incluidos no deben estar dentro del directorio web accesible, o deben ser ejecutables (`.php`) para que no se sirvan como texto.

## Referencias
- [CWE-219](https://cwe.mitre.org/data/definitions/219.html): Storage of File with Sensitive Data Under Web Root
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 5](natas-05.md) · [Índice](../README.md) · [Nivel 7 →](natas-07.md)
