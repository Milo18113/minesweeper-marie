# =============================================================================
#  GENERADOR DE TABLERO - BUSCAMINAS para MARIE.js
#  Autor: Jorge
#  Descripción:
#    - Define minas mediante lista de coordenadas (fila, col) base-0
#    - Calcula conteo de vecinos para cada celda
#    - Genera bloque de memoria compatible con MARIE.js
#    - Imprime visualización del tablero en consola
#
#  ORGANIZACIÓN DE MEMORIA:
#    Tablero Real  → dirección 0x100 .. 0x1FF  (256 celdas)
#      Valor -1  → mina         (se almacena como 0xFFF en MARIE, 12 bits)
#      Valor 0-8 → vecinos
#
#    Tablero Visible → dirección 0x200 .. 0x2FF  (256 celdas)
#      Valor 0 → oculta  (estado inicial de todas las celdas)
#      Valor 1 → revelada
#      Valor 2 → bandera
#
#  ÍNDICE DE CELDA:
#    índice = fila * 16 + columna
#    fila y columna son base-0 (0..15)
# =============================================================================

# =============================================================================
#  CONFIGURACIÓN - MODIFICA AQUÍ LAS MINAS
#  Formato: (fila, columna) — base 0, rango 0..15
# =============================================================================
MINAS = [
    (0, 0), (0, 7), (0, 13),
    (1, 5), (1, 10),(3,2),
    (2, 1), (2, 8), (2, 14),
    (3, 3), (3, 11),(4,4),
    (4, 0), (4, 6), (4, 15),
    (5, 4), (5, 9), (10,5),(10,7),(8,6),
    
]

# Dirección base en memoria para cada tablero (en hexadecimal)
DIR_TABLERO_REAL    = 0x100
DIR_TABLERO_VISIBLE = 0x200

FILAS    = 16
COLUMNAS = 16

# =============================================================================
#  FUNCIONES
# =============================================================================

def construir_tablero_real(minas):
    """Construye la matriz 16x16 del tablero real.
    -1 = mina, 0-8 = cantidad de vecinos con mina."""
    tablero = [[0] * COLUMNAS for _ in range(FILAS)]

    # Colocar minas
    for (f, c) in minas:
        if 0 <= f < FILAS and 0 <= c < COLUMNAS:
            tablero[f][c] = -1
        else:
            print(f"  [ADVERTENCIA] Mina fuera de rango ignorada: ({f}, {c})")

    # Calcular vecinos
    for f in range(FILAS):
        for c in range(COLUMNAS):
            if tablero[f][c] == -1:
                continue
            conteo = 0
            for df in [-1, 0, 1]:
                for dc in [-1, 0, 1]:
                    if df == 0 and dc == 0:
                        continue
                    nf, nc = f + df, c + dc
                    if 0 <= nf < FILAS and 0 <= nc < COLUMNAS:
                        if tablero[nf][nc] == -1:
                            conteo += 1
            tablero[f][c] = conteo

    return tablero


def construir_tablero_visible():
    """Construye la matriz 16x16 del tablero visible.
    Todas las celdas inician como ocultas (valor 0)."""
    return [[0] * COLUMNAS for _ in range(FILAS)]


def valor_a_marie(valor):
    """Convierte un valor del tablero real al formato MARIE de 12 bits.
    -1 (mina) → 0xFFF  (complemento a dos en 12 bits)
    0-8       → valor directo"""
    if valor == -1:
        return 0xFFF  # -1 en complemento a dos 12 bits
    return valor


def generar_bloque_memoria(tablero_real, tablero_visible):
    """Genera el bloque de memoria para pegar en MARIE.js.
    Formato por línea:  DIRECCIÓN_HEX, VALOR_HEX"""
    lineas = []
    lineas.append("/ ============================================================")
    lineas.append("/ BLOQUE DE MEMORIA - BUSCAMINAS")
    lineas.append("/ Generado automáticamente por generar_tablero.py")
    lineas.append("/ ------------------------------------------------------------")
    lineas.append(f"/ Tablero Real    → 0x{DIR_TABLERO_REAL:03X} .. 0x{DIR_TABLERO_REAL + 255:03X}")
    lineas.append(f"/ Tablero Visible → 0x{DIR_TABLERO_VISIBLE:03X} .. 0x{DIR_TABLERO_VISIBLE + 255:03X}")
    lineas.append("/ ------------------------------------------------------------")
    lineas.append("/ Valores tablero real:")
    lineas.append("/   0xFFF = mina (-1)")
    lineas.append("/   0x000..0x008 = cantidad de vecinos")
    lineas.append("/ Valores tablero visible:")
    lineas.append("/   0x000 = oculta")
    lineas.append("/   0x001 = revelada")
    lineas.append("/   0x002 = bandera")
    lineas.append("/ ============================================================")
    lineas.append("")

    # Tablero Real
    lineas.append("/ --- TABLERO REAL ---")
    for f in range(FILAS):
        for c in range(COLUMNAS):
            idx = f * COLUMNAS + c
            dir_mem = DIR_TABLERO_REAL + idx
            val = valor_a_marie(tablero_real[f][c])
            lineas.append(f"{dir_mem:03X}, {val:03X}")
    lineas.append("")

    # Tablero Visible
    lineas.append("/ --- TABLERO VISIBLE (todas ocultas = 0) ---")
    for f in range(FILAS):
        for c in range(COLUMNAS):
            idx = f * COLUMNAS + c
            dir_mem = DIR_TABLERO_VISIBLE + idx
            val = tablero_visible[f][c]
            lineas.append(f"{dir_mem:03X}, {val:03X}")

    return "\n".join(lineas)


def imprimir_tablero_consola(tablero_real, minas_set):
    """Imprime una visualización del tablero real en consola."""
    ANCHO_CELDA = 3

    print()
    print("=" * 60)
    print("  VISUALIZACIÓN DEL TABLERO REAL")
    print(f"  Total de minas: {len(minas_set)}")
    print("=" * 60)

    # Encabezado de columnas
    encabezado = "    " + "".join(f"{c:^{ANCHO_CELDA}}" for c in range(COLUMNAS))
    print(encabezado)
    print("    " + "-" * (COLUMNAS * ANCHO_CELDA))

    for f in range(FILAS):
        fila_str = f"{f:2d} |"
        for c in range(COLUMNAS):
            val = tablero_real[f][c]
            if val == -1:
                fila_str += " * "   # mina
            elif val == 0:
                fila_str += " . "   # vacía
            else:
                fila_str += f" {val} "
        print(fila_str)

    print()
    print("  Leyenda:  * = mina   . = 0 vecinos   1-8 = vecinos")
    print("=" * 60)
    print()


def imprimir_resumen_memoria():
    """Imprime el mapa de memoria para documentación."""
    print()
    print("=" * 60)
    print("  MAPA DE MEMORIA PARA MARIE.js")
    print("=" * 60)
    print(f"  Tablero Real    : 0x{DIR_TABLERO_REAL:03X} → 0x{DIR_TABLERO_REAL + 255:03X}  ({256} celdas)")
    print(f"  Tablero Visible : 0x{DIR_TABLERO_VISIBLE:03X} → 0x{DIR_TABLERO_VISIBLE + 255:03X}  ({256} celdas)")
    print()
    print("  Fórmula de índice:")
    print("    índice = fila * 16 + columna")
    print("    dirección real    = 0x100 + índice")
    print("    dirección visible = 0x200 + índice")
    print()
    print("  Codificación tablero real (12 bits MARIE):")
    print("    0xFFF → mina  (-1 en complemento a 2)")
    print("    0x000 → 0 vecinos")
    print("    0x001 .. 0x008 → 1-8 vecinos")
    print()
    print("  Codificación tablero visible:")
    print("    0x000 → oculta")
    print("    0x001 → revelada")
    print("    0x002 → bandera")
    print("=" * 60)
    print()


# =============================================================================
#  MAIN
# =============================================================================
if __name__ == "__main__":
    minas_set = set(MINAS)

    print("\n[1/4] Construyendo tablero real...")
    tablero_real = construir_tablero_real(MINAS)

    print("[2/4] Construyendo tablero visible (todo oculto)...")
    tablero_visible = construir_tablero_visible()

    print("[3/4] Generando bloque de memoria para MARIE.js...")
    bloque = generar_bloque_memoria(tablero_real, tablero_visible)

    # Guardar archivo
    nombre_archivo = "memoria_buscaminas.txt"
    with open(nombre_archivo, "w", encoding="utf-8") as f:
        f.write(bloque)
    print(f"       → Guardado en: {nombre_archivo}")

    print("[4/4] Visualizando tablero...")
    imprimir_tablero_consola(tablero_real, minas_set)
    imprimir_resumen_memoria()

    print("¡Listo! Copia el contenido de 'memoria_buscaminas.txt' en MARIE.js")
    print()
