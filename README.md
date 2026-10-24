# Complementos de GlassPad

Complementos para [GlassPad Launcher](https://github.com/kan0p/asus-screenpad-linux), el launcher táctil del ScreenPad de la ASUS ZenBook Pro 15 UX580. Un complemento agrega una página propia al launcher, con su mosaico en el inicio y en *Todas las apps*, igual que Calculadora o Temporizador.

Requiere GlassPad 0.9.1 o posterior. La lista completa de funciones está en la [referencia](https://github.com/kan0p/asus-screenpad-linux/blob/main/COMPLEMENTOS.md).

## Complementos

| Complemento | Qué hace |
|---|---|
| [Dados](dados/) | Tira dos dados al tocar; un toque largo escribe el total en la ventana con foco. |

## Instalar uno

```sh
git clone https://github.com/kan0p/glasspad-complementos
mkdir -p ~/.local/share/glasspad/complementos
cp -r glasspad-complementos/dados ~/.local/share/glasspad/complementos/
systemctl --user restart glasspad-launcher
```

El mosaico aparece solo en el inicio. Para quitarlo, se borra la carpeta y se reinicia el launcher.

## Crear uno paso a paso

Como ejemplo, un contador que suma uno con cada toque y vuelve a cero con un toque largo.

### 1. La carpeta

```sh
mkdir -p ~/.local/share/glasspad/complementos/contador
```

El nombre de la carpeta identifica al complemento: no conviene cambiarlo después, porque el inicio guarda su posición con ese nombre.

### 2. complemento.py

`~/.local/share/glasspad/complementos/contador/complemento.py`:

```python
from gi.repository import Gtk

NOMBRE = "Contador"
ICONO = "accessories-calculator"


def crear(api):
    api.css(".contador-numero { font-size: 200px; }")
    numero = Gtk.Label(label="0", css_classes=["contador-numero"])
    cuenta = [0]

    def sumar():
        cuenta[0] += 1
        numero.set_label(str(cuenta[0]))

    def reiniciar():
        cuenta[0] = 0
        numero.set_label("0")
        api.avisar("Contador en cero")

    return api.accion(Gtk.Button(child=numero, css_classes=["flat"], vexpand=True),
                      sumar, reiniciar)
```

- `NOMBRE` es el texto del mosaico.
- `ICONO` puede ser un nombre del tema de íconos o un archivo de la carpeta. Sin `ICONO`, se usa `icono.svg` si existe.
- `crear(api)` devuelve el widget de la página.

El dedo sobre el pad no llega a GTK como un clic: el launcher lee el pad y solo entrega los toques a los widgets registrados con `api.accion`. Un botón conectado a `clicked` funciona con el mouse, pero no con el dedo.

### 3. Probarlo

```sh
systemctl --user restart glasspad-launcher
```

Si el mosaico no aparece, el motivo está en el journal:

```sh
journalctl --user -u glasspad-launcher -n 20 --no-pager
```

```
glasspad-launcher: complemento contador: NameError: name 'Gtk' is not defined
```

Un complemento que falla se omite; el resto del launcher sigue funcionando.

Para ver más detalle mientras se desarrolla, se puede correr el launcher a mano:

```sh
systemctl --user stop glasspad-launcher
glasspad-launcher --debug
# Ctrl+C para salir
systemctl --user start glasspad-launcher
```

### 4. Un ícono propio

Los mosaicos del launcher son SVG de 96×96 con esquinas de radio 24 y un degradado de fondo; [dados/icono.svg](dados/icono.svg) sirve de plantilla. Se guarda como `icono.svg` en la carpeta y se quita la línea `ICONO`.

### 5. Más allá

- Varios botones: `api.tecla(al_tocar, texto)` crea teclas con el estilo de Number Key.
- Barras que se arrastran: `api.deslizable(widget, al_cambiar)`.
- Listas largas: dentro de un `Gtk.ScrolledWindow` se desplazan solas.
- Pausar trabajo en segundo plano cuando la página no se ve: `api.al_mostrar(funcion)`.
- Escribir en la app activa: `api.escribir(texto)` o `api.atajo("ctrl+c")`.

El código corre dentro del launcher, así que nada debe bloquear: `GLib.timeout_add` en lugar de `time.sleep`, y `Gio` asíncrono para leer archivos o la red. La página mide 1920×1080.

## Aportar un complemento

Se aceptan PR con una carpeta nueva que tenga `complemento.py`, `icono.svg` y una fila en la tabla de arriba. El código tiene que ser GPL-3.0-or-later o compatible, y no puede necesitar root.

## Licencia

GPL-3.0-or-later.
