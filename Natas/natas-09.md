# Natas 09 → 10: Command Injection

[← Nivel 8](natas-08.md) · [Índice](../README.md) · [Nivel 10 →](natas-10.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Inyección |
| **Debilidad** | [CWE-78](https://cwe.mitre.org/data/definitions/78.html): OS Command Injection |
| **Herramientas** | Navegador |
| **URL** | `http://natas9.natas.labs.overthewire.org` |
| **Credenciales** | `natas9` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El input del buscador se concatena en un comando de shell, lo que permite ejecutar comandos arbitrarios con un separador `;`.

## Análisis y explotación
1. La página era un buscador de palabras. Su código fuente mostraba que el parámetro `needle` se insertaba directamente en un comando de shell:
   ```php
   passthru("grep -i $key dictionary.txt");
   ```
2. Sin ninguna sanitización, se puede inyectar un separador de comandos (`;`) para ejecutar un segundo comando. Payload en el campo de búsqueda:
   ```
   ;cat /etc/natas_webpass/natas10
   ```
3. El servidor ejecutó `grep -i ` (sin argumento) y luego el `cat`, mostrando la contraseña en la respuesta.

## Resultado

- **Password obtenida (`natas10`):** `[redacted]`

## Lección aprendida y mitigación
Nunca pasar entrada del usuario a funciones de ejecución de comandos (`passthru`, `exec`, `system`). Si es inevitable, validar con lista blanca y escapar con `escapeshellarg()`.

## Referencias
- [CWE-78](https://cwe.mitre.org/data/definitions/78.html): OS Command Injection
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 8](natas-08.md) · [Índice](../README.md) · [Nivel 10 →](natas-10.md)
