# Natas 08 → 09: Codificación reversible, no cifrado

[← Nivel 7](natas-07.md) · [Índice](../README.md) · [Nivel 9 →](natas-09.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Criptografía |
| **Debilidad** | [CWE-261](https://cwe.mitre.org/data/definitions/261.html): Weak Encoding for Password |
| **Herramientas** | PHP (CLI) |
| **URL** | `http://natas8.natas.labs.overthewire.org` |
| **Credenciales** | `natas8` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El secreto se "protege" con una cadena de codificaciones reversibles (base64, inversión y hexadecimal) que se deshace en una línea.

## Análisis y explotación
1. El código fuente de la página mostraba cómo se comparaba el secreto: el valor se ofuscaba con
   ```php
   bin2hex(strrev(base64_encode($secret)))
   ```
   y se comparaba contra un valor ya codificado (`[redacted]`).
2. Basta con aplicar las operaciones **en orden inverso**: `hex2bin` → `strrev` → `base64_decode`. Con PHP en la terminal:
   ```bash
   php -r 'echo base64_decode(strrev(hex2bin("[valor codificado]")));'
   ```
3. El resultado fue el secreto en texto plano (`[redacted]`). Al enviarlo en el formulario, la aplicación mostró la contraseña.

## Resultado

- **Password obtenida (`natas9`):** `[redacted]`

## Lección aprendida y mitigación
Base64, hexadecimal o invertir una cadena son **codificaciones, no cifrado**: no dan confidencialidad. Para verificar secretos se deben usar hashes con sal (por ejemplo Argon2id o bcrypt).

## Referencias
- [CWE-261](https://cwe.mitre.org/data/definitions/261.html): Weak Encoding for Password
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 7](natas-07.md) · [Índice](../README.md) · [Nivel 9 →](natas-09.md)
