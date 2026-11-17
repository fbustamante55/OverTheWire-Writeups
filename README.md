# OverTheWire - Writeups

Writeups de mi paso por los wargames de [OverTheWire](https://overthewire.org/wargames/): qué vulnerabilidad hay en cada nivel, cómo la explotué, con qué herramientas y cómo se corrige. **Un archivo por nivel.**

> ⚠️ **Spoilers:** las contraseñas de los niveles están omitidas (`[redacted]`). Aquí se explica el *cómo*, no la respuesta final.

## Índice

| Wargame | Enfoque | Progreso |
|---------|---------|----------|
| Natas | Seguridad web | Niveles 0 → 19 (en progreso) |

## Natas: nivel por nivel

| Nivel | Vulnerabilidad | Técnica / herramienta |
|-------|----------------|-----------------------|
| [0](Natas/natas-00.md) | Información expuesta en el código fuente | Inspección del HTML |
| [1](Natas/natas-01.md) | Información expuesta en el código fuente | Inspección del HTML |
| [2](Natas/natas-02.md) | Directory listing activado | `/files/users.txt`, gobuster |
| [3](Natas/natas-03.md) | Ruta oculta revelada por `robots.txt` | gobuster con autenticación |
| [4](Natas/natas-04.md) | Autorización basada en el header `Referer` | Burp Repeater, `curl -e` |
| [5](Natas/natas-05.md) | Estado de sesión controlado por el cliente | Edición de cookie `loggedin` |
| [6](Natas/natas-06.md) | Archivo incluido accesible desde la web | Lectura directa de `secret.inc` |
| [7](Natas/natas-07.md) | Local File Inclusion (LFI) | `?page=/etc/natas_webpass/...` |
| [8](Natas/natas-08.md) | Codificación reversible, no cifrado | `hex2bin`, `strrev`, `base64_decode` |
| [9](Natas/natas-09.md) | Command Injection | Separador `;` |
| [10](Natas/natas-10.md) | Command Injection con blocklist | Argumento extra en `grep` |
| [11](Natas/natas-11.md) | XOR con key estática | Known-plaintext, cookie forjada |
| [12](Natas/natas-12.md) | Upload sin validación de extensión | Webshell PHP, Burp Repeater |
| [13](Natas/natas-13.md) | Upload validado solo por magic bytes | Archivo polyglot JPEG + PHP |
| [14](Natas/natas-14.md) | SQL Injection (login bypass) | `" or 1=1 -- ` |
| [15](Natas/natas-15.md) | Blind SQL Injection (booleana) | sqlmap `--technique=B --prefix --suffix` |
| [16](Natas/natas-16.md) | Blind Command Injection | `$(...)` dentro de comillas dobles, script Python |
| [17](Natas/natas-17.md) | Time-based Blind SQLi | `SLEEP()` + `IF()`, medición de tiempo en Python |
| [18](Natas/natas-18.md) | Session-ID prediction | Fuerza bruta de `PHPSESSID` (1-640) vía cookie |
| [19](Natas/natas-19.md) | Session-ID prediction (ID codificado) | PHPSESSID = hex de `NUMERO-admin`, fuerza bruta 1-640 |

## Video

🎥 Video del proceso completo: _próximamente_

<!-- Cuando lo subas a YouTube, reemplaza la línea de arriba por: [Ver en YouTube](URL) -->

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
- **IDs impredecibles:** un identificador de sesión (natas18) debe ser largo y aleatorio. Un rango chico como `rand(1,640)` se fuerza entero en segundos y permite secuestrar la sesión de otro usuario.

## Aviso
Este repositorio es solo con fines educativos. Los retos de OverTheWire son un entorno legal y controlado para practicar.
