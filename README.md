Este README explica qué hace el script `convertir_ldebliqd_a_rdebliqc_corregido.py`, por qué existe, cómo funciona cada parte y cómo usarlo cada vez que necesites convertir un archivo.

---

## 1. Objetivo

El script convierte archivos de devolución de débito `LDEBLIQD` a una estructura compatible con archivos de devolución de crédito `RDEBLIQC`.

Esto se hizo porque el stored procedure existente lee correctamente archivos `RDEBLIQC`, pero no procesa directamente los archivos `LDEBLIQD`.

Ejemplo:

```text
Entrada: LDEBLIQD_202605120950.txt
Salida:  RDEBLIQC_202605120950.txt
```

El archivo convertido conserva la fecha y hora del nombre original. Solo cambia el prefijo:

```text
LDEBLIQD -> RDEBLIQC
```

Importante: el archivo generado es un adaptador interno para el sistema. No es un archivo oficial original emitido por Payway.

---

## 2. Qué hace el script

El script realiza estos pasos:

1. Lee el archivo `LDEBLIQD`.
2. Separa encabezado, cuerpo y cierre.
3. Recorre las líneas del cuerpo.
4. Detecta cuáles partidas no tienen motivo de rechazo.
5. Omite las partidas rechazadas.
6. Convierte las partidas aprobadas al formato fijo de `RDEBLIQC`.
7. Genera un nuevo encabezado `RDEBLIQC`.
8. Genera un nuevo cierre `RDEBLIQC`.
9. Recalcula la cantidad de registros exportados.
10. Recalcula el importe total exportado.
11. Guarda el archivo en una carpeta de salida.

---

## 3. Regla principal: solo partidas sin rechazo

El archivo `LDEBLIQD` puede traer operaciones aprobadas y rechazadas.

El script solo exporta las partidas aprobadas.

Una partida se considera aprobada cuando:

```python
codigo_rechazo == "" or codigo_rechazo == "000"
```

Y además:

```python
motivo_rechazo == ""
```

Si una línea tiene código o descripción de rechazo, se omite.

Ejemplos de partidas omitidas:

```text
019 OPERACION NO PERMITIDA PARA ESA TARJETA
020 RECHAZADA POR EL EMISOR DE LA TARJETA
034 SE REINTENTARA AUTORIZACION AUTOMATICA
```

---

## 4. Por qué el archivo debe tener 300 caracteres por línea

El stored procedure lee el archivo por posiciones fijas.

Eso significa que espera encontrar cada dato en una posición exacta.

Por ejemplo:

```text
posición X = importe
posición Y = fecha
posición Z = estado
```

Si una línea queda corrida, Oracle puede intentar convertir un espacio o texto a número y generar este error:

```text
ORA-06502: PL/SQL: error de conversión de carácter a número
```

Por eso el script corregido garantiza que:

- cada línea tenga exactamente 300 caracteres;
- el `*` quede siempre en la última posición;
- los campos numéricos se completen con ceros;
- los campos vacíos se completen con espacios.

---

## 5. Estructura del archivo generado

El archivo final tiene tres partes:

```text
ENCABEZADO
CUERPO
CIERRE
```

Ejemplo conceptual:

```text
0RDEBLIQC...
1...
1...
1...
9RDEBLIQC...
```

El encabezado empieza con `0RDEBLIQC`.

El cuerpo contiene solo las partidas aprobadas.

El cierre empieza con `9RDEBLIQC`.

---

## 6. Explicación de cada función

---

## 6.1. `ajustar(linea, largo)`

```python
def ajustar(linea: str, largo: int) -> str:
    linea = linea.rstrip("\r\n")
    return linea[:largo].ljust(largo)
```

Esta función normaliza una línea a un largo fijo.

Hace tres cosas:

1. Quita saltos de línea.
2. Si la línea es más larga que el largo esperado, la corta.
3. Si la línea es más corta, la completa con espacios.

Ejemplo:

```python
ajustar("ABC", 5)
```

Devuelve:

```text
ABC  
```

Esto es necesario porque los archivos son de ancho fijo.

---

## 6.2. `fecha_aaaammdd_a_ddmmaa(fecha)`

```python
def fecha_aaaammdd_a_ddmmaa(fecha: str) -> str:
    if len(fecha) == 8 and fecha.isdigit():
        return fecha[6:8] + fecha[4:6] + fecha[2:4]
    return " " * 6
```

Convierte la fecha del formato del archivo de débito:

```text
AAAAMMDD
```

al formato usado dentro del cuerpo del archivo de crédito:

```text
DDMMAA
```

Ejemplo:

```text
20260512 -> 120526
```

---

## 6.3. `parsear_ldebliqd(linea)`

```python
def parsear_ldebliqd(linea: str) -> dict:
```

Esta función toma una línea del cuerpo del `LDEBLIQD` y extrae los campos importantes usando posiciones fijas.

Campos extraídos:

| Campo | Posición | Descripción |
|---|---:|---|
| `registro` | `0:1` | Tipo de registro. En cuerpo debe ser `1`. |
| `tarjeta` | `1:17` | Número de tarjeta. |
| `factura` | `20:28` | Número de factura o secuencial. |
| `fecha` | `28:36` | Fecha en formato `AAAAMMDD`. |
| `cod_transaccion` | `36:40` | Código de transacción. |
| `importe` | `40:55` | Importe con dos decimales implícitos. |
| `id_cliente` | `55:70` | ID del cliente. |
| `alta` | `70:71` | Código de alta de identificador. |
| `codigo_rechazo` | `100:103` | Código de rechazo. |
| `motivo_rechazo` | `103:143` | Descripción del rechazo. |

Devuelve un diccionario con esos datos.

---

## 6.4. `es_aprobado(data)`

```python
def es_aprobado(data: dict) -> bool:
    codigo = data["codigo_rechazo"].strip()
    motivo = data["motivo_rechazo"].strip()

    return (codigo == "" or codigo == "000") and motivo == ""
```

Decide si una línea debe pasar al archivo final.

Devuelve `True` si no tiene rechazo.

Devuelve `False` si tiene código o motivo de rechazo.

---

## 6.5. `linea_con_asterisco_final(contenido, largo)`

```python
def linea_con_asterisco_final(contenido: str, largo: int) -> str:
    return contenido[:largo - 1].ljust(largo - 1) + "*"
```

Esta función garantiza que el `*` quede exactamente al final de la línea.

Para una línea de 300 caracteres:

```text
caracteres 1 a 299 = contenido y espacios
caracter 300 = *
```

Esta fue una de las correcciones más importantes, porque antes el `*` podía quedar antes de tiempo y desalinear el layout.

---

## 6.6. `construir_header_rdebliqc(...)`

```python
def construir_header_rdebliqc(establecimiento: str, fecha: str, hora: str) -> str:
```

Construye el encabezado del archivo final.

El encabezado contiene:

| Campo | Descripción |
|---|---|
| `0` | Indica encabezado. |
| `RDEBLIQC` | Tipo de archivo esperado por el SP. |
| `900000    ` | Campo fijo. |
| `establecimiento` | Tomado del archivo original. |
| `fecha` | Fecha original del archivo. |
| `hora` | Hora original del archivo. |
| espacios | Relleno. |
| `*` | Fin de línea. |

---

## 6.7. `construir_cuerpo_rdebliqc(data, establecimiento)`

```python
def construir_cuerpo_rdebliqc(data: dict, establecimiento: str) -> str:
```

Esta función convierte una operación aprobada del `LDEBLIQD` en una línea compatible con `RDEBLIQC`.

Toma datos reales del archivo original y completa los campos que no existen en débito con ceros o espacios.

Datos que se conservan del `LDEBLIQD`:

| Dato | Uso en salida |
|---|---|
| Tarjeta | Se copia al cuerpo `RDEBLIQC`. |
| Factura/secuencial | Se copia al cuerpo `RDEBLIQC`. |
| Fecha | Se convierte de `AAAAMMDD` a `DDMMAA`. |
| Código de transacción | Se copia. |
| Importe | Se copia. |
| ID cliente | Se copia. |
| Código de alta | Se copia. |
| Establecimiento | Se toma del encabezado original. |

Campos completados artificialmente para compatibilidad:

| Campo | Valor usado | Motivo |
|---|---|---|
| Banco pagador | `000` | No viene igual en débito. |
| Sucursal | `000` | Requerido por layout crédito. |
| Lote | `0000` | Requerido por layout crédito. |
| Cuenta | `0000000000` | Campo obligatorio en layout crédito. |
| Estado | `0` | Porque solo se exportan aprobados. |
| Rechazo 1 | `00` | Sin rechazo. |
| Descripción rechazo 1 | espacios | Sin rechazo. |
| Rechazo 2 | `00` | Sin segundo rechazo. |
| Tarjeta nueva | espacios | No aplica. |
| Fecha de pago | espacios | No se informa. |
| Cartera | `00` | Valor fijo compatible. |

---

## 6.8. `construir_trailer_rdebliqc(...)`

```python
def construir_trailer_rdebliqc(establecimiento, fecha, hora, cantidad, total)
```

Construye el cierre del archivo final.

Incluye:

| Campo | Descripción |
|---|---|
| `9` | Indica cierre. |
| `RDEBLIQC` | Tipo de archivo esperado por el SP. |
| `900000    ` | Campo fijo. |
| `establecimiento` | Tomado del archivo original. |
| `fecha` | Fecha original. |
| `hora` | Hora original. |
| `cantidad` | Cantidad de registros aprobados exportados. |
| `total` | Suma de importes aprobados exportados. |
| espacios | Relleno. |
| `*` | Fin de línea. |

Ejemplo:

```text
cantidad = 53 -> 0000053
total = 261185548 -> 000000261185548
```

---

## 6.9. `convertir_archivo(ruta_entrada, carpeta_salida)`

```python
def convertir_archivo(ruta_entrada: str, carpeta_salida: str | None = None) -> Path:
```

Es la función principal.

Hace todo el flujo:

1. Recibe el archivo de entrada.
2. Define la carpeta de salida.
3. Crea la carpeta de salida si no existe.
4. Cambia el nombre de salida de `LDEBLIQD` a `RDEBLIQC`.
5. Lee el archivo completo.
6. Extrae encabezado, cuerpo y cierre.
7. Obtiene establecimiento, fecha y hora del encabezado.
8. Recorre cada línea del cuerpo.
9. Parsea cada línea.
10. Filtra las rechazadas.
11. Convierte las aprobadas.
12. Acumula cantidad e importe.
13. Genera encabezado nuevo.
14. Genera cierre nuevo.
15. Escribe el archivo final.
16. Imprime un resumen.

---

## 7. Cómo usarlo

## 7.1. Requisitos

Necesitás tener instalado Python 3.

No requiere librerías externas.

No necesitás instalar pandas, numpy ni nada adicional.

---

## 7.2. Carpetas recomendadas

Crear esta estructura:

```text
C:\Debitos
C:\Debitos\Entrada
C:\Debitos\Convertidos
```

Guardar el script en:

```text
C:\Debitos\convertir_ldebliqd_a_rdebliqc_corregido.py
```

Guardar los archivos originales en:

```text
C:\Debitos\Entrada
```

El script genera los convertidos en:

```text
C:\Debitos\Convertidos
```

---

## 7.3. Ejecutarlo desde CMD

Ejemplo:

```bash
python C:\Debitos\convertir_ldebliqd_a_rdebliqc_corregido.py C:\Debitos\Entrada\LDEBLIQD_202605120950.txt
```

Salida esperada:

```text
Archivo generado: C:\Debitos\Convertidos\RDEBLIQC_202605120950.txt
Registros leídos: 83
Registros exportados sin rechazo: 53
Registros omitidos por rechazo: 30
Importe total exportado: 261185548
```

---

## 7.4. Ejecutarlo indicando carpeta de salida

También podés pasar una carpeta de salida manualmente:

```bash
python C:\Debitos\convertir_ldebliqd_a_rdebliqc_corregido.py C:\Debitos\Entrada\LDEBLIQD_202605120950.txt C:\OtraCarpeta\Salida
```

---

## 8. Uso diario recomendado

Cada vez que necesites convertir un archivo:

1. Descargar el archivo `LDEBLIQD`.
2. Guardarlo en `C:\Debitos\Entrada`.
3. Ejecutar el script.
4. Buscar el archivo convertido en `C:\Debitos\Convertidos`.
5. Cargar el archivo `RDEBLIQC` generado en el proceso habitual.

---

## 9. Errores comunes

## 9.1. `python no se reconoce como comando`

Python no está instalado o no fue agregado al PATH.

Solución:

1. Instalar Python.
2. Marcar `Add Python to PATH` durante la instalación.
3. Cerrar y volver a abrir CMD.

---

## 9.2. `ORA-06502`

Este error aparece cuando Oracle intenta convertir a número un campo que no es numérico.

Puede pasar si:

- la línea no tiene 300 caracteres;
- el `*` no está al final;
- el layout quedó corrido;
- el archivo fue modificado manualmente;
- se abrió y guardó desde Excel;
- se usó un archivo incorrecto.

El script corregido evita este problema generando líneas fijas de 300 caracteres.

---

## 9.3. El archivo convertido tiene menos registros

Es correcto.

El script solo exporta partidas sin rechazo.

Las partidas con motivo de rechazo se omiten.

---

## 10. Recomendaciones importantes

No abrir ni guardar estos archivos con Excel.

Excel puede romper el archivo porque puede:

- sacar ceros a la izquierda;
- modificar espacios;
- cambiar el formato;
- alterar el ancho fijo.

Para revisar el archivo, usar:

- VS Code;
- Notepad++;
- Bloc de notas.

---

## 11. Resumen final

El script toma:

```text
LDEBLIQD con aprobados y rechazados
```

filtra:

```text
solo aprobados
```

y genera:

```text
RDEBLIQC compatible con el stored procedure
```

Mantiene:

- nombre base del archivo;
- fecha y hora;
- establecimiento;
- tarjeta;
- importe;
- ID cliente;
- factura/secuencial;
- código de transacción.

Y recalcula:

- cantidad de registros exportados;
- importe total del cierre.

