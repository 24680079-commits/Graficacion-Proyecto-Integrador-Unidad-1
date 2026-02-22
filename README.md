# Graficacion-Proyecto-Integrador-Unidad-1

# Codigó base
```bash
import bpy
import random

def crear_material(nombre, color_rgb):
    # Crea un material básico con un color específico
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)  # RGBA
    return mat

def generar_escenario():

    # 1. Limpiar la escena previa
    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    # 2. Definir materiales
    mat_pared_a = crear_material("ParedOscura", (0.1, 0.1, 0.1))
    mat_pared_b = crear_material("ParedDetalle", (0.8, 0.2, 0.0))

    # 3. Parámetros del escenario
    largo_pasillo = 10
    ancho_pasillo = 4

    # 4. Construcción de paredes
    for i in range(largo_pasillo):

        # Pared izquierda
        bpy.ops.mesh.primitive_cube_add(location=(-ancho_pasillo, i * 2, 1))
        pared_izq = bpy.context.active_object

        if i % 2 == 0:
            pared_izq.data.materials.append(mat_pared_a)
        else:
            pared_izq.data.materials.append(mat_pared_b)

        pared_izq.scale.z = 1.5

        # Pared derecha
        bpy.ops.mesh.primitive_cube_add(location=(ancho_pasillo, i * 2, 1))
        pared_der = bpy.context.active_object
        pared_der.data.materials.append(mat_pared_a)

    # 5. Agregar suelo
    bpy.ops.mesh.primitive_plane_add(size=1, location=(0, (largo_pasillo - 1), 0))
    suelo = bpy.context.active_object
    suelo.scale.x = ancho_pasillo + 2
    suelo.scale.y = largo_pasillo + 10

generar_escenario()
```
<img width="1792" height="1120" alt="Captura de pantalla 2026-02-22 a la(s) 12 41 28" src="https://github.com/user-attachments/assets/f97f9bd6-6659-4b73-8a6d-95b05b90b46e" />

# Proceso de Desarrollo del Proyecto

El proyecto inició a partir de un código base proporcionado por el profesor, el cual generaba un pasillo recto compuesto por cubos repetidos y un plano como suelo. A partir de esa estructura inicial, el objetivo fue evolucionarlo hacia un escenario más complejo, incorporando curvas y una animación de cámara.

El desarrollo se realizó en Blender utilizando su API de Python (bpy).

 # 1. Reorganización del Código

Primero estructuré el código en funciones más claras:

crear_material()

generar_pista()

Esto permitió que el código fuera más modular y reutilizable.

También agregué una validación para evitar crear materiales duplicados:
```bash
if nombre in bpy.data.materials:
    return bpy.data.materials[nombre]
```
Con esto optimicé la gestión de materiales.

# 2. Ampliación del Pasillo

El código base generaba únicamente un tramo recto.

Yo amplié la estructura definiendo nuevos parámetros:
```bash
ancho_pasillo = 4
separacion = 2
largo_recto = 10
radio = 8
segmentos_curva = 25
```
Aquí comenzó el cambio importante: ya no se trataba solo de repetir cubos, sino de generar una trayectoria completa.

# 3. Creación de Curvas Mediante Trigonometría

Para transformar el pasillo recto en una pista con curvas, utilicé funciones trigonométricas:
```bash
x = centro_x + radio * math.cos(ang)
y = centro_y + radio * math.sin(ang)
```
Con esto generé puntos sobre una circunferencia utilizando la representación paramétrica:

𝑥 = 𝑟 cos(𝜃) x=rcos(θ)
𝑦=𝑟sin⁡(𝜃) y=rsin(θ)

De esta manera, el pasillo dejó de ser lineal y pasó a tener una forma en "U".

Este fue el cambio matemático más importante del proyecto.

# 4. Cálculo del Centro de la Pista
En lugar de colocar objetos manualmente, almacené los puntos centrales de la trayectoria:
```bash
puntos_centro.append((x, y))
```
Esto permitió:

Generar el piso dinámicamente

Utilizar los mismos puntos para animar la cámara

Aquí el proyecto pasó de ser solo geométrico a ser estructuralmente inteligente.

# 5. Generación Procedural del Piso

En lugar de usar un plano simple, construí la malla manualmente:
```bash
mesh.from_pydata(verts, [], faces)
```
Para ello:

Calculé el vector dirección entre dos puntos.

Obtuve el vector normal.

Usé ese vector para desplazar vértices a izquierda y derecha.

Esto permitió que el piso siguiera exactamente la forma de la pista, incluyendo curvas.

Aquí ya estaba aplicando:

Vectores

Normalización

Construcción de mallas personalizadas

# 6. Creación de la Cámara desde Código

Después de tener la pista completa, agregué la parte más importante: la animación.

Primero creé la cámara:
```
cam_data = bpy.data.cameras.new("CamaraPista")
```
La vinculé a la escena y la configuré como cámara activa.

# 7. Configuración de la Línea de Tiempo

Definí:
```
fps = 24
duracion = 4
total_frames = fps * duracion
```
Esto permitió controlar la duración total de la animación.

# 8. Movimiento Automático de la Cámara

Recorrí todos los puntos centrales:
```
for i, (x, y) in enumerate(puntos_centro):
```
Para cada punto:

Posicioné la cámara

Calculé el ángulo de orientación con atan2

Inserté keyframes
```
cam.keyframe_insert("location", frame=frame)
cam.keyframe_insert("rotation_euler", frame=frame)
```
De esta forma la cámara siguió exactamente la trayectoria generada.

# 9. Suavizado de Animación

Finalmente cambié la interpolación a Bezier:
```
kp.interpolation = 'BEZIER'
```
Esto permitió que el movimiento no fuera rígido, sino fluido.

# Resultado Final

El proceso completo fue:

Tomar un pasillo recto básico.

Convertirlo en una pista con tramos rectos y curvos.

Construir el piso dinámicamente con mallas personalizadas.

Crear una cámara desde código.

Animarla automáticamente siguiendo la trayectoria.

Suavizar el movimiento.

El proyecto evolucionó de una estructura estática a una simulación animada generada completamente de forma procedural.


```bash
import bpy
import math

def crear_material(nombre, color_rgb):
    if nombre in bpy.data.materials:
        return bpy.data.materials[nombre]
    mat = bpy.data.materials.new(name=nombre)
    mat.diffuse_color = (*color_rgb, 1.0)
    return mat

def generar_pista():

    bpy.ops.object.select_all(action='SELECT')
    bpy.ops.object.delete()

    mat_pared_a = crear_material("ParedOscura", (0.1, 0.1, 0.1))
    mat_pared_b = crear_material("ParedDetalle", (0.8, 0.2, 0.0))
    mat_suelo   = crear_material("SueloBlanco", (1.0, 1.0, 1.0))

    ancho_pasillo = 4
    separacion = 2
    largo_recto = 10
    radio = 8
    segmentos_curva = 25

    puntos_centro = []

    for i in range(largo_recto):
        y = i * separacion
        puntos_centro.append((0, y))

        for lado in [-1, 1]:
            bpy.ops.mesh.primitive_cube_add(location=(lado * ancho_pasillo, y, 1))
            obj = bpy.context.active_object
            obj.data.materials.append(mat_pared_a if i % 2 == 0 else mat_pared_b)

    ultimo_y = (largo_recto - 1) * separacion


    centro_x = -radio
    centro_y = ultimo_y

    for i in range(1, segmentos_curva + 1):
        ang = (i / segmentos_curva) * (math.pi / 2)
        x = centro_x + radio * math.cos(ang)
        y = centro_y + radio * math.sin(ang)
        puntos_centro.append((x, y))

        for lado in [-1, 1]:
            r = radio + (lado * ancho_pasillo)
            px = centro_x + r * math.cos(ang)
            py = centro_y + r * math.sin(ang)
            bpy.ops.mesh.primitive_cube_add(location=(px, py, 1))
            obj = bpy.context.active_object
            obj.rotation_euler[2] = ang
            obj.data.materials.append(mat_pared_a if i % 2 == 0 else mat_pared_b)

    final_x = centro_x
    final_y = centro_y + radio

    for i in range(largo_recto):
        x = final_x - (i * separacion)
        puntos_centro.append((x, final_y))

        for lado in [-1, 1]:
            bpy.ops.mesh.primitive_cube_add(location=(x, final_y + lado * ancho_pasillo, 1))
            obj = bpy.context.active_object
            obj.rotation_euler[2] = math.pi / 2
            obj.data.materials.append(mat_pared_a if i % 2 == 0 else mat_pared_b)

    ultimo_x = final_x - (largo_recto - 1) * separacion

    centro_x2 = ultimo_x
    centro_y2 = final_y + radio

    for i in range(1, segmentos_curva + 1):
        ang = (i / segmentos_curva) * (math.pi / 2)
        x = centro_x2 - radio * math.sin(ang)
        y = centro_y2 - radio * math.cos(ang)
        puntos_centro.append((x, y))

        for lado in [-1, 1]:
            r = radio + (lado * ancho_pasillo)
            px = centro_x2 - r * math.sin(ang)
            py = centro_y2 - r * math.cos(ang)
            bpy.ops.mesh.primitive_cube_add(location=(px, py, 1))
            obj = bpy.context.active_object
            obj.rotation_euler[2] = math.pi / 2 + ang
            obj.data.materials.append(mat_pared_a if i % 2 == 0 else mat_pared_b)


    verts = []
    faces = []

    for i in range(len(puntos_centro) - 1):
        x1, y1 = puntos_centro[i]
        x2, y2 = puntos_centro[i+1]

        dx = x2 - x1
        dy = y2 - y1
        l = math.sqrt(dx*dx + dy*dy)

        nx = -dy / l
        ny = dx / l

        left1  = (x1 + nx*ancho_pasillo, y1 + ny*ancho_pasillo, 0)
        right1 = (x1 - nx*ancho_pasillo, y1 - ny*ancho_pasillo, 0)
        left2  = (x2 + nx*ancho_pasillo, y2 + ny*ancho_pasillo, 0)
        right2 = (x2 - nx*ancho_pasillo, y2 - ny*ancho_pasillo, 0)

        b = len(verts)
        verts.extend([left1, right1, right2, left2])
        faces.append((b, b+1, b+2, b+3))

    mesh = bpy.data.meshes.new("Piso")
    mesh.from_pydata(verts, [], faces)
    mesh.update()
    piso = bpy.data.objects.new("Piso", mesh)
    bpy.context.collection.objects.link(piso)
    piso.data.materials.append(mat_suelo)



    cam_data = bpy.data.cameras.new("CamaraPista")
    cam_data.lens = 50
    cam = bpy.data.objects.new("CamaraPista", cam_data)
    bpy.context.collection.objects.link(cam)
    bpy.context.scene.camera = cam

    fps = 24
    duracion = 4
    total_frames = fps * duracion
    bpy.context.scene.frame_start = 1
    bpy.context.scene.frame_end = total_frames
    bpy.context.scene.render.fps = fps

    cam_z = 1.6
    n = len(puntos_centro)

    for i, (x, y) in enumerate(puntos_centro):
        frame = int(1 + (i / (n - 1)) * (total_frames - 1))

        if i < n - 1:
            nx, ny = puntos_centro[i+1]
        else:
            nx, ny = puntos_centro[i]

        ang = math.atan2(nx - x, ny - y)

        cam.location = (x, y, cam_z)
        cam.rotation_euler = (math.radians(90), 0, ang)

        cam.keyframe_insert("location", frame=frame)
        cam.keyframe_insert("rotation_euler", frame=frame)

    # Suavizar animación
    for fcurve in cam.animation_data.action.fcurves:
        for kp in fcurve.keyframe_points:
            kp.interpolation = 'BEZIER'

    print(" Pista y cámara animada creadas")

generar_pista()
```

![Captura de pantalla 2026-02-22 a la(s) 12 47 01](https://github.com/user-attachments/assets/33a28d3c-388a-4a58-9950-f3046b0bd227)

<img width="1792" height="1120" alt="Captura de pantalla 2026-02-22 a la(s) 12 47 55" src="https://github.com/user-attachments/assets/e0fcf07f-9cb0-49a1-8e92-2add02e68887" />

