# Natas 01 → 02: Información expuesta en el código fuente

[← Nivel 0](natas-00.md) · [Índice](../README.md) · [Nivel 2 →](natas-02.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Exposición de información |
| **Debilidad** | [CWE-615](https://cwe.mitre.org/data/definitions/615.html): Inclusion of Sensitive Information in Source Code Comments |
| **Herramientas** | Navegador (ver código fuente) |
| **URL** | `http://natas1.natas.labs.overthewire.org` |
| **Credenciales** | `natas1` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
Igual que el nivel anterior: la contraseña está en un comentario HTML y basta con ver el código fuente para leerla.

## Análisis y explotación
Siguiendo la misma dinámica, la revisión del código fuente expuso un comentario HTML que contenía directamente la contraseña para el Nivel 2.

## Resultado

- **Password obtenida (`natas2`):** `[redacted]`

## Lección aprendida y mitigación
Igual que en el nivel 0: los comentarios HTML son públicos. Las restricciones del lado del cliente (por ejemplo, bloquear el clic derecho con JavaScript) no protegen el código fuente, que se puede leer con `Ctrl + U`, DevTools o `curl`.

## Referencias
- [CWE-615](https://cwe.mitre.org/data/definitions/615.html): Inclusion of Sensitive Information in Source Code Comments
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 0](natas-00.md) · [Índice](../README.md) · [Nivel 2 →](natas-02.md)
