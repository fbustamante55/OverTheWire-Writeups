# OverTheWire - Writeups

Writeups de mi paso por los wargames de [OverTheWire](https://overthewire.org/wargames/): qué vulnerabilidad hay en cada nivel, cómo la exploté, con qué herramientas y cómo se corrige. **Un archivo por nivel.**

> [!NOTE]
> Las contraseñas de los niveles están omitidas (`[redacted]`). Aquí se documenta el método, no la respuesta final.

## Índice

| Wargame | Enfoque | Progreso |
|---------|---------|----------|
| Natas | Seguridad web | Niveles 0 → 19 (en progreso) |

## Natas: nivel por nivel

| Nivel | Vulnerabilidad | Técnica / herramienta | CWE |
|-------|----------------|-----------------------|-----|
| [0](Natas/natas-00.md) | Información expuesta en el código fuente | Inspección del HTML | [CWE-615](https://cwe.mitre.org/data/definitions/615.html) |
| [1](Natas/natas-01.md) | Información expuesta en el código fuente | Inspección del HTML | [CWE-615](https://cwe.mitre.org/data/definitions/615.html) |
| [2](Natas/natas-02.md) | Directory listing activado | `/files/users.txt`, gobuster | [CWE-548](https://cwe.mitre.org/data/definitions/548.html) |
| [3](Natas/natas-03.md) | Ruta oculta revelada por `robots.txt` | gobuster con autenticación | [CWE-200](https://cwe.mitre.org/data/definitions/200.html) |
| [4](Natas/natas-04.md) | Autorización basada en el header `Referer` | Burp Repeater, `curl -e` | [CWE-293](https://cwe.mitre.org/data/definitions/293.html) |
| [5](Natas/natas-05.md) | Estado de sesión controlado por el cliente | Edición de cookie `loggedin` | [CWE-565](https://cwe.mitre.org/data/definitions/565.html) |
| [6](Natas/natas-06.md) | Archivo incluido accesible desde la web | Lectura directa de `secret.inc` | [CWE-219](https://cwe.mitre.org/data/definitions/219.html) |
| [7](Natas/natas-07.md) | Local File Inclusion (LFI) | `?page=/etc/natas_webpass/...` | [CWE-98](https://cwe.mitre.org/data/definitions/98.html) |
| [8](Natas/natas-08.md) | Codificación reversible, no cifrado | `hex2bin`, `strrev`, `base64_decode` | [CWE-261](https://cwe.mitre.org/data/definitions/261.html) |
| [9](Natas/natas-09.md) | Command Injection | Separador `;` | [CWE-78](https://cwe.mitre.org/data/definitions/78.html) |
| [10](Natas/natas-10.md) | Command Injection con blocklist | Argumento extra en `grep` | [CWE-88](https://cwe.mitre.org/data/definitions/88.html) |
| [11](Natas/natas-11.md) | XOR con key estática | Known-plaintext, cookie forjada | [CWE-327](https://cwe.mitre.org/data/definitions/327.html) |
| [12](Natas/natas-12.md) | Upload sin validación de extensión | Webshell PHP, Burp Repeater | [CWE-434](https://cwe.mitre.org/data/definitions/434.html) |
| [13](Natas/natas-13.md) | Upload validado solo por magic bytes | Archivo polyglot JPEG + PHP | [CWE-434](https://cwe.mitre.org/data/definitions/434.html) |
| [14](Natas/natas-14.md) | SQL Injection (login bypass) | `" or 1=1 -- ` | [CWE-89](https://cwe.mitre.org/data/definitions/89.html) |
| [15](Natas/natas-15.md) | Blind SQL Injection (booleana) | sqlmap `--technique=B --prefix --suffix` | [CWE-89](https://cwe.mitre.org/data/definitions/89.html) |
| [16](Natas/natas-16.md) | Blind Command Injection | `$(...)` dentro de comillas dobles, script Python | [CWE-78](https://cwe.mitre.org/data/definitions/78.html) |
| [17](Natas/natas-17.md) | Time-based Blind SQLi | `SLEEP()` + `IF()`, medición de tiempo en Python | [CWE-89](https://cwe.mitre.org/data/definitions/89.html) |
| [18](Natas/natas-18.md) | Session-ID prediction | Fuerza bruta de `PHPSESSID` (1-640) vía cookie | [CWE-334](https://cwe.mitre.org/data/definitions/334.html) |
| [19](Natas/natas-19.md) | Session-ID prediction (ID codificado) | PHPSESSID = hex de `NUMERO-admin`, fuerza bruta 1-640 | [CWE-334](https://cwe.mitre.org/data/definitions/334.html) |

<!-- Cuando subas el video a YouTube, agrega aquí:
## Video
[Ver en YouTube](URL)
-->

## Formato de cada writeup

Todos los niveles siguen la misma estructura:

1. **Ficha técnica:** categoría, debilidad (CWE), herramientas, URL y credenciales.
2. **Resumen:** la vulnerabilidad en una frase.
3. **Análisis y explotación:** el proceso paso a paso, incluidos los intentos fallidos.
4. **Resultado:** lo que se obtuvo.
5. **Lección aprendida y mitigación:** cómo se corrige en un sistema real.

## Herramientas usadas
- Navegador (inspección de código fuente, DevTools)
- `gobuster` (enumeración de directorios)
- Burp Suite (Repeater)
- `curl`
- `sqlmap` (SQL Injection automatizada)
- Python (scripts de apoyo)

## Estructura del repositorio

```
OverTheWire-Writeups/
├── Natas/
│   ├── natas-00.md
│   ├── natas-01.md
│   ├── ...
│   ├── natas-18.md
│   └── natas-19.md
└── README.md
```

## Conclusiones y recomendaciones

- **Listas blancas, no negras:** un filtro de caracteres prohibidos (natas10, natas16) siempre deja algún vector abierto — `$(...)` sobrevive incluso cuando se filtran comillas y backticks.
- **Criptografía real:** XOR con key fija (natas11) se rompe con un solo plaintext conocido. Usar HMAC o cifrado autenticado.
- **Uploads seguros:** validar el tipo real, renombrar el archivo, reprocesar la imagen y servir desde un directorio sin ejecución (natas12 y natas13).
- **Prepared statements:** previenen el login bypass (natas14), la inyección ciega booleana (natas15) y la basada en tiempo (natas17).
- **Canales laterales:** un sí/no visible (natas15), un "aparece/no aparece" (natas16) o incluso solo el tiempo de respuesta (natas17) bastan para exfiltrar un secreto completo. Ocultar el output no mitiga la inyección.
- **IDs impredecibles:** un identificador de sesión (natas18 y natas19) debe ser largo y aleatorio. Un rango pequeño como `rand(1,640)` se fuerza entero en segundos y permite secuestrar la sesión de otro usuario.

## Aviso
Este repositorio es solo con fines educativos. Los retos de OverTheWire son un entorno legal y controlado para practicar.
