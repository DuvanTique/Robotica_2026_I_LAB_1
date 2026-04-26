# 🤖 Laboratorio No. 01 - Robótica Industrial
## Trayectorias, Entradas y Salidas Digitales
 
**Robótica Industrial 2026-I** | Universidad Nacional de Colombia
 
---
## 👥 Integrantes
 
| Nombre | 
|--------|
| Duvan Stiven Tique Osorio | 
| Luis Mendoza | 

---
 
## Descripción del Proyecto
 
Decoración de una torta virtual usando un robot industrial ABB IRB 140. El objetivo es escribir nombres y realizar decoraciones sobre una superficie plana mediante trayectorias de movimiento, calibración de herramientas y control de I/O digitales.
 
---
 
## Objetivos
 
- Calibración de herramientas (TCP) en robot real y RobotStudio
- Programación de trayectorias MOVJ y MOVL en RAPID
- Diseño e implementación de herramientas personalizadas
- Uso de Work Objects para cambio de marcos de referencia
- Configuración de entradas y salidas digitales en IRC5
- Integración de sistemas de transporte automatizado
---
 
## Requerimientos
 
### Software
- **RobotStudio** v5.0 o superior
### Hardware
- Robot ABB **IRB 140**
- Controlador **IRC5**
- Banda transportadora con variador de frecuencia
### Documentación
- Manual de especificaciones ABB IRB 140
---
 
 ## Especificaciones del Trabajo
 
### Restricciones de Diseño
 
| Parámetro | Valor |
|-----------|-------|
| Tamaño de torta | 20 personas |
| Velocidad (rango) | 100 - 1000 mm/s |
| Zona tolerable (z) | ±10 mm |
| Posición inicial | HOME robot |
| Posición final | HOME robot |
| Superficie | Torta virtual |
| Nombres | Espaciados |
 
### Herramienta
- Diseñar herramienta para fijar marcador/plumón
- Calibrar TCP (Método de 4 puntos)
1. Abrir el menú ABB → Calibration → Tool.
2. Seleccionar Define New Tool.
3. Importar modelo CAD a RobotStudio
4. Comparar tooldata creado vs importado
5. signar un nombre a la herramienta.
6. Elegir el método 4-point method (método recomendado por ABB para TCPs fijos), en donde se mueve el robot hasta que la punta de la herramienta toque suavemente el punto de referencia (en este caso un punzon) modificando la orientacion general.
7. Guardar Store point 1,2,3,4.
8. Cambiar masa de la herramienta a 1.

   
![Calibración TCP](Imagenes/Calibracion_Herramienta.jpeg)


AL terminar la calibación de la herramienta se obtuvo un error de aproximadamente 6mm, por lo cual si se tiene el modelo CAD de la herramienta es mejor utilizar robotStudio para cargar el TCP al robot.

### Work Object
1. Seleccionar la herramienta previamente calibrada.
2. identificar en la pieza:
- Punto de origen (P0) del WorkObject.
- Punto sobre el eje X (P1).
- Punto que defina el plano XY (P2).
3. Ir a ABB → Calibration → WorkObject.
4. Seleccionar Define New WorkObject.
5. Asignar un nombre.
6. Elegir el método 3-point method.
7. Confirmar la calibración.

![Calibración Work Object](Imagenes/Calibracion_WorkObject.jpeg)

### Entradas y Salidas Digitales
 
**Entrada 1**: Iniciar decoración → Enciende luz → Regresa a HOME  
**Entrada 2**: Mantenimiento → Pose de cambio de herramienta → Apaga luz
 
**Salida 1**: Indicador de estado  
**Salida 2**: Control banda transportadora
 
### Control de Transporte
Conectar salida digital IRC5 → Entrada variador → Motor banda
 
Secuencia:
1. Robot finaliza decoración
2. Activa salida digital
3. Banda transportadora se enciende
4. Pastel se trasladada automáticamente
5. Activa entrada digital 1
6. El robot se desplaza hacia la posición de inicio del decorado
7. Empieza el decorado del pastel
8. El robot vuelve a home
9. El pastel se traslada automáticamente

### Rutina de decorado

![Diagrama del flujo](ABB_IRC140_flow_diagram.svg)

### Ruina de Mantenimiento

![Diagrama del flujo](ABB_IRC140_maintenance_flow.svg)

## Funciones utilizadas: 
## Constantes
 
### Objetivos Robóticos (robtarget)
 
#### `BaseDraw`
```rapid
CONST robtarget BaseDraw:=[[0,0,0],
    [0.694346616,0.10879539,-0.122149941,0.700803633],
    [-1,-1,0,0],
    [9E9,9E9,9E9,9E9,9E9,9E9]];
```
**Descripción:** Punto de origen (0,0,0) que sirve como referencia base para el cálculo de posiciones relativas de dibujo. Define la orientación inicial de la herramienta mediante cuaterniones.
 
**Componentes:**
- Posición cartesiana: [0, 0, 0] mm
- Orientación (cuaterniones): [0.694, 0.109, -0.122, 0.701]
- Configuración de eje: [-1, -1, 0, 0]
- Datos extendidos: [9E9, 9E9, 9E9, 9E9, 9E9, 9E9]
---
 
#### `HOME`
```rapid
CONST robtarget HOME:=[[-413.972133,684.456,139.157599],
    [0.989015864,0,-0.147809404,0],
    [0,0,0,0],
    [9E9,9E9,9E9,9E9,9E9,9E9]];
```
**Descripción:** Posición de reposo primaria del robot. Posición de seguridad donde el robot aguarda nuevos pasteles y desde donde inicia el ciclo de decorado.
 
**Componentes:**
- Posición cartesiana: [-413.97, 684.46, 139.16] mm
- Orientación: [0.989, 0, -0.148, 0]
- Configuración de eje: [0, 0, 0, 0]
**Uso:** Punto de partida y destino final después de completar el decorado.
 
---
 
#### `HOME2`
```rapid
CONST robtarget HOME2:=[[-370.116379342,684.456,-4.288115021],
    [0.989015864,0,-0.147809404,0],
    [0,0,0,0],
    [9E9,9E9,9E9,9E9,9E9,9E9]];
```
**Descripción:** Posición de aproximación intermedia. Ubicada entre HOME y el área de dibujo.
 
**Componentes:**
- Posición cartesiana: [-370.12, 684.46, -4.29] mm
- Orientación: [0.989, 0, -0.148, 0]
**Uso:** Paso intermedio en el descenso para mejorar estabilidad y precisión.
 
---
 
#### `Target_aproximacion`
```rapid
CONST robtarget Target_aproximacion:=[[172.3445,164.9723,-139.7743],
    [0.694346616,0.10879539,-0.122149941,0.700803633],
    [-1,-1,0,0],
    [9E9,9E9,9E9,9E9,9E9,9E9]];
```
**Descripción:** Posición final de aproximación, 20mm por encima del área de escritura. Punto de transición antes del descenso final para iniciar el decorado.
 
**Componentes:**
- Posición cartesiana: [172.34, 164.97, -139.77] mm
- Orientación: [0.694, 0.109, -0.122, 0.701]
**Uso:** Última posición de seguridad antes de bajar a la altura de escritura.
 
---
 
### Constantes Numéricas
 
#### Dimensiones de Letras
```rapid
CONST num l_width := 40;    ! Ancho de cada letra [mm]
CONST num l_height := 60;   ! Altura de cada letra [mm]
CONST num l_space := 15;    ! Espaciado entre letras [mm]
```
**Descripción:** Definen el tamaño y espaciado de las letras dibujadas.
 
**Valores:**
- `l_width`: 40 mm - ancho estándar para cada letra
- `l_height`: 60 mm - altura estándar de caracteres
- `l_space`: 15 mm - separación horizontal entre letras
**Impacto:** Estos valores determinan la legibilidad y tamaño visual final del texto decorado.
 
---
 
#### Alturas de Operación
```rapid
CONST num z_draw := 0;      ! Altura de escritura [mm]
CONST num z_lift := -20;    ! Altura de levantamiento [mm]
```
**Descripción:** Alturas relativas del eje Z para operaciones de escritura y seguridad.
 
**Valores:**
- `z_draw`: 0 mm - altura donde la herramienta hace contacto para escribir
- `z_lift`: -20 mm - altura de despegue/seguridad
**Nota Técnica:** El comentario en el código sugiere que z_draw puede requerir ajuste a +5 mm si la escritura no es óptima.
 
---
 
## Variables
 
### `pActual`
```rapid
VAR robtarget pActual;
```
**Tipo:** `robtarget` (posición robótica tridimensional)
 
**Descripción:** Variable dinámica que almacena la posición actual durante la secuencia de dibujo. Se actualiza constantemente mediante desplazamientos relativos (`Offs()`) para posicionar cada letra.
 
**Uso en programa:**
- Se inicializa en BaseDraw + offset para LUIS
- Se incrementa por (l_width + l_space) después de cada letra
- Se reinicializa para el grupo de DUVAN
**Ejemplo:**
```rapid
pActual := Offs(BaseDraw, 30, 20, 0);  ! Posición inicial de LUIS
Draw_L;
pActual := Offs(pActual, l_width + l_space, 0, 0);  ! Siguiente letra
```
 
---
 
## Procedimientos Principales
 
### `main()`
 
**Firma:**
```rapid
PROC main()
    ! Cuerpo del programa
ENDPROC
```
 
**Descripción:** Procedimiento principal que orquesta todo el sistema. Implementa el ciclo de control, monitoreo de señales de entrada y coordinación de movimientos y escritura.
 
**Secuencia de ejecución:**
 
1. **Inicialización de banda (líneas 18-21)**
   ```rapid
   Set BWD_Conveyor;
   WaitTime 2;
   Reset BWD_Conveyor;
   ```
   - Enciende banda hacia atrás 2 segundos
   - Prepara para nuevo ciclo
2. **Bucle infinito (línea 23)**
   ```rapid
   WHILE TRUE DO
   ```
   - Sistema en espera permanente de pasteles
3. **Detección de pastel (línea 24)**
   ```rapid
   IF DI_01 = 1 THEN
   ```
   - Verifica entrada digital DI_01
   - Continúa si hay pastel disponible
4. **Activación de indicador (línea 26)**
   ```rapid
   Set DO_01;
   ```
   - Enciende luz testigo de proceso en curso
5. **Movimiento y posicionamiento (líneas 28-34)**
   - Activa banda transportadora
   - HOME → HOME2 → Target_aproximacion (descenso gradual)
   - MoveJ: movimientos rápidos entre posiciones
   - MoveL: movimiento lineal final precisado a 300 mm/s
6. **Dibujo de LUIS (líneas 37-42)**
   ```rapid
   pActual := Offs(BaseDraw, 30, 20, 0);
   Draw_L;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_U;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_I;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_S;
   ```
   - Inicia en offset [30, 20, 0] respecto a BaseDraw
   - Dibuja L, U, I, S con espacios de 55mm entre ellas
7. **Dibujo de DUVAN (líneas 44-50)**
   ```rapid
   pActual := Offs(BaseDraw, 10, 110, 0);
   Draw_D;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_U;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_V;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_A;  pActual := Offs(pActual, l_width + l_space, 0, 0);
   Draw_N;
   ```
   - Segunda línea: offset [10, 110, 0]
   - Dibuja D, U, V, A, N
8. **Retorno a posición segura (líneas 52-55)**
   - MoveL hacia arriba 20mm
   - MoveJ regresa por Target_aproximacion → HOME2 → HOME
9. **Expulsión del pastel (líneas 59-62)**
   ```rapid
   Set FWD_Conveyor;
   WaitTime 2;
   Reset FWD_Conveyor;
   ```
   - Banda hacia adelante 2 segundos
   - Pastel sale del área de decorado
10. **Desactivación de indicador (línea 66)**
    ```rapid
    Reset DO_01;
    ```
 
**Parámetros de movimiento:**
- `v1000`: 1000 mm/s - aproximación inicial rápida
- `v800`: 800 mm/s - aproximación intermedia
- `v500`: 500 mm/s - aproximación final controlada
- `v300`: 300 mm/s - descenso seguro
- `v100`: 100 mm/s - velocidad de escritura precisada
- `fine`: precisión fina en punto destino
**Condiciones de operación:**
- Se repite indefinidamente hasta detención manual
- Requiere presencia de DI_01 = 1 para iniciar ciclo

---

### Código RAPID

[Ver módulo RAPID Module1](/Module1.mod)


   
---

### Diseño de la Herramienta
La herramienta del robot fue diseñada mediante impresión 3D utilizando PLA como material base. Consta de dos piezas principales: la primera se acopla directamente al robot, mientras que la segunda sostiene el marcador. Ambas piezas se unen mediante un pasador, al cual se incorpora un resorte que permite al marcador tener un rango de movimiento controlado. Este mecanismo asegura que la herramienta no se fracture al entrar en contacto con el workobject, garantizando tanto la funcionalidad como la durabilidad del sistema.


![EnsamblajeHerramienta](Imagenes/Ensamblaje_Herramienta.png)
[Planos](PlanosHerramienta.pdf)

---

### Simulación en robotStudio
[![Ver video](https://img.youtube.com/vi/98ZQVTQvyAo/0.jpg)](https://youtu.be/98ZQVTQvyAo)

---

### Implementación en el laboratorio

[![Ver video](https://img.youtube.com/vi/yADe7vtRQms/0.jpg)](https://youtu.be/yADe7vtRQms)

---



 

