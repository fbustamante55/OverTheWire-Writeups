# Natas 00 → 01: Información expuesta en el código fuente

[Índice](../README.md) · [Nivel 1 →](natas-01.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Exposición de información |
| **Debilidad** | [CWE-615](https://cwe.mitre.org/data/definitions/615.html): Inclusion of Sensitive Information in Source Code Comments |
| **Herramientas** | Navegador (ver código fuente) |
| **URL** | `http://natas0.natas.labs.overthewire.org` |
| **Credenciales** | `natas0` / `natas0` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
La contraseña del siguiente nivel está escrita en un comentario HTML de la página, visible para cualquiera que revise el código fuente.

## Análisis y explotación
Al ingresar al sitio, se revisó el código fuente de la página (`Ctrl + U`), encontrando la contraseña del siguiente nivel oculta en un comentario HTML.

## Resultado

- **Password obtenida (`natas1`):** `[redacted]`

## Lección aprendida y mitigación
Todo lo que llega al navegador es público. No se deben dejar credenciales ni información sensible en comentarios HTML, y conviene revisar el código que se publica antes de desplegarlo.

## Referencias
- [CWE-615](https://cwe.mitre.org/data/definitions/615.html): Inclusion of Sensitive Information in Source Code Comments
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[Índice](../README.md) · [Nivel 1 →](natas-01.md)
