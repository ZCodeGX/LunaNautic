# Luna Naval

Un juego de batalla naval por turnos escrito en [Luna](https://github.com/ZCodeGX/Luna), creado para mostrar qué se puede construir con el lenguaje: módulos, tablas, listas, closures, bucles, manejo de errores, persistencia JSON y una CLI con argumentos.

**Estado:** funcional · **Luna:** 0.7.6+ · **Licencia de Luna:** GPL-3.0

## Qué hace

- Tablero 8×8 con flota de 5 barcos (17 casillas) desplegada al azar.
- Turnos alternos: vos disparás escribiendo coordenadas (`A1`…`H8`); la IA responde.
- IA con dos modos: búsqueda en damero y persecución de las casillas vecinas a un impacto.
- Marcadores de agua (`~`), tocado (`X`) y hundido (`#`), con tu flota revelada y la enemiga oculta.
- Estadísticas persistentes en `datos/estadisticas.json`: partidas, victorias, mejor marca y rachas.
- Menú interactivo, ayuda integrada y modo `--demo` (IA vs IA) reproducible con `--seed`.
- Opción 5 del menú: consola que usa `os.run` (nueva librería de sistema) para ejecutar un comando y mostrar código de salida, stdout y stderr sin imprimirlos directamente.
- La interfaz se limpia sola con `os.system("clear")` (`cls` en Windows) al volver al menú y en cada turno/resultado, para redibujar el tablero desde cero. El modo `--demo` no limpia para no ensuciar la salida automática.

## Requisitos

Necesitás el compilador/runtime de Luna. Por ejemplo, cloná Luna y compilalo:

```bash
git clone https://github.com/ZCodeGX/Luna.git
cd Luna
./luna.py build
cd ..
```

El binario queda en `Luna/build/luna`. Si ya tenés `luna` en el `PATH`, también sirve. La consola del juego necesita una versión de Luna con `os.run`/`os.system` (commit `e1b6466` o posterior); clonar `main` es suficiente.

## Uso

```bash
./run.sh                       # menú interactivo
./run.sh --demo                # partida automática
./run.sh --demo --seed 42      # misma partida siempre
./run.sh --help
```

Si querés invocar el binario directamente:

```bash
RUTA_A_LUNA/build/luna main.luna
RUTA_A_LUNA/build/luna main.luna --demo --seed 42
```

Ejecutá siempre desde la raíz del proyecto: las estadísticas se guardan en `datos/`.

En el menú, la opción `5` abre una consola de prueba: escribí por ejemplo `echo hola` y verás `código: 0 | ok: true` junto a la salida capturada. Los comandos se ejecutan con tus permisos a través del shell del sistema.

### Salida del modo demo (semilla 7, extracto)

```text
 Modo demostración (semilla 7)
 TU -> F2  TOCADO (Portaaviones)
 IA -> D5  TOCADO (Portaaviones)
 TU -> E5  TOCADO (Acorazado)
 IA -> C4  HUNDIDO: Crucero (3)
----------------------------------------------------------
DERROTA: tu flota fue hundida en 55 turnos (25% de aciertos).
----------------------------------------------------------
```

## Cómo jugar

```
     A B C D E F G H
    +---+---+---+---+---+---+---+---+
 1 | . | . | . | . | . | . | . | . |
 ...
    +---+---+---+---+---+---+---+---+
```

Escribí una coordenada como `A1` o `H8`. Si aciertas, seguís; el enemigo dispara después de cada turno. `salir` vuelve al menú. El objetivo es hundir los 5 barcos enemigos antes de que hundan los tuyos.

## Pruebas

```bash
./tests/run.sh
```

Ejecuta 25 comprobaciones (`tests/smoke.luna`), incluidas una llamada real a `os.run` y la elección de `clear`/`cls`, y una partida demo determinista. GitHub Actions hace lo mismo compilando Luna desde cero.

## Estructura

| Archivo | Qué muestra de Luna |
| --- | --- |
| `main.luna` | entrada del programa, `argv`, CLI y menú |
| `juego/config.luna` | constantes y listas de tablas para la flota |
| `juego/coords.luna` | `string.sub`/`string.find`, conversión y `throw` |
| `juego/tablero.luna` | tablas anidadas, listas mutables y estado de disparos |
| `juego/ia.luna` | closures con estado, `list.pop`, `math.shuffle` |
| `juego/render.luna` | construcción de strings, tableros y mensajes |
| `juego/estadisticas.luna` | `io` + `json`, `try`/`catch` y persistencia |
| `juego/partida.luna` | bucle de turnos, `input` y orquestación |
| `juego/shell.luna` | `os.run` y `os.system`, `os.name`, captura de salida y limpieza de consola |
| `tests/smoke.luna` | pruebas sin intervención del usuario |

## Detalles del lenguaje que se ponen a prueba

- Importaciones con paquetes (`import juego`, `import juego.tablero`) y exports `pub`.
- `let`, tablas con claves literales, listas, indexación y asignación compuesta.
- `if`/`else` anidados, `while`, `for ... in`, `break` y `try`/`catch`.
- Funciones de orden superior: `ia.crear()` devuelve una tabla de funciones que comparten estado por closures.
- Random determinista con `math.seed`, `math.randint` y `math.shuffle`.
- Serialización `json.encode`/`json.decode` y archivos con `io.read`, `io.write` e `io.ensure_dir`.
- Ejecución de procesos con `os.run(command, cwd?)`, leyendo `code`, `ok`, `stdout` y `stderr` desde una tabla, y con `os.system(command)` para comandos que heredan la consola (`clear`/`cls`).
- Nota: en esta versión las cadenas no se concatenan con `+`; se usa `string.concat`. Las decisiones de tres o más ramas se escriben con `else:` + `if` anidado porque las cadenas largas de `else if` no compilan correctamente.
