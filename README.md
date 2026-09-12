# Sistema de Gestión Hospitalaria — Cliente de Escritorio

Cliente de escritorio en **Java Swing** de una aplicación hospitalaria cliente–servidor. Presenta
una interfaz distinta según el rol del usuario autenticado —administrador, médico, farmacéutico y
paciente— y se comunica con el servidor por un **protocolo propio de sockets con mensajes JSON**.

Proyecto académico desarrollado en la Universidad Nacional de Costa Rica (UNA).

La API que consume este cliente vive en
[ProyectoPrograBackEnd](https://github.com/Killazz17/ProyectoPrograBackEnd).

## ✨ Funcionalidades

**Administrador**

- Gestión de médicos, farmacéuticos y pacientes
- Catálogo de medicamentos: alta, búsqueda y modificación
- Tablero con indicadores y gráficos (JFreeChart)

**Médico**

- Búsqueda de pacientes
- Prescripción de recetas: selección de medicamentos, indicaciones, cantidad y duración
- Modificación del detalle de una receta antes de su despacho

**Farmacéutico**

- Cola de recetas por despachar y avance de su estado
- Consulta del catálogo de medicamentos

**Común a todos los roles**

- Inicio de sesión y cambio de contraseña
- Histórico de recetas con filtros
- Ventana de comunicación entre usuarios conectados, alimentada por las notificaciones que el
  servidor difunde por WebSocket

## 🏗️ Arquitectura

Tres capas bien separadas: presentación (Swing, en MVC), servicios (el cliente del protocolo) y
DTO compartidos con el servidor.

```
src/main/java/
├── Presentation/
│   ├── Views/              # Formularios Swing (.form + .java)
│   ├── Controllers/        # Un controlador por vista
│   ├── Models/             # Modelo de cada vista
│   ├── Observable.java     # Patrón Observer
│   └── IObserver.java      # Las vistas se re-dibujan cuando su modelo cambia
├── Services/
│   ├── BaseService.java    # Cliente del protocolo: socket, envío y lectura
│   ├── AuthService.java
│   ├── PacienteService.java · MedicoService.java · FarmaceutaService.java
│   ├── MedicamentoService.java · MedicamentoPrescritoService.java
│   ├── RecetaService.java · PrescripcionService.java · DespachoService.java
│   ├── HistoricoRecetaService.java
│   └── MessageService.java # Mensajería entre usuarios
├── Domain/Dtos/            # RequestDto y ResponseDto (misma forma que en el servidor)
├── Utilities/
└── hospital/               # Ventana de comunicación
```

**Patrón MVC con Observer.** Cada vista tiene su controlador y su modelo; el modelo extiende
`Observable` y la vista implementa `IObserver`, así que un cambio de estado se refleja en todas las
vistas suscritas sin que ellas se consulten entre sí.

**Capa de servicios.** Toda la comunicación pasa por `BaseService`: abre un socket contra el
servidor (por defecto `localhost:7070`, configurable), serializa el `RequestDto` con Gson, lo
escribe terminado en salto de línea, lee la línea de respuesta y la deserializa como `ResponseDto`.
Usa un timeout de 5 segundos y cierra la conexión en el `finally`, de modo que una petición colgada
no deja el socket abierto. Cada servicio concreto solo declara qué controlador y qué operación
invoca.

### Vistas

| Vista | Rol | Propósito |
|---|---|---|
| `LoginView` · `ChangePasswordView` | Todos | Autenticación y cambio de contraseña |
| `MainWindow` | Todos | Ventana principal y navegación según rol |
| `DashboardView` | Administrador | Indicadores y gráficos |
| `MedicoView` · `PacienteView` · `FarmaceutaView` | Administrador | Gestión de cada tipo de usuario |
| `MedicamentosView` · `AgregarMedicamentoView` | Administrador | Catálogo de medicamentos |
| `BuscarPacienteView` | Médico | Búsqueda de pacientes |
| `PrescribirView` · `ModificarDetalleView` | Médico | Prescripción y ajuste del detalle |
| `DespachoView` | Farmacéutico | Despacho de recetas |
| `HistoricoRecetaView` | Todos | Histórico de recetas |

## 🚀 Puesta en marcha

Requisitos: **JDK 17+**, **Maven 3.8+** y el servidor corriendo en `localhost:7070`.

```bash
git clone https://github.com/Killazz17/ProyectoPrograFrontEnd1.git
cd ProyectoPrograFrontEnd1
mvn clean install
mvn exec:java -Dexec.mainClass="hospital.Main"
```

Si el servidor corre en otra máquina o en otro puerto, `BaseService` recibe host y puerto por
constructor.

Las vistas se editaron con el diseñador de formularios de IntelliJ IDEA, por eso cada una tiene su
archivo `.form` junto al `.java`.

## 🧰 Dependencias principales

| Dependencia | Uso |
|---|---|
| `gson` | Serialización JSON del protocolo |
| `jfreechart` | Gráficos del tablero |
| `jcalendar` | Selector de fechas en los formularios |
| `Java-WebSocket` | Recepción de notificaciones en tiempo real |

## 👥 Autores

[@Killazz17](https://github.com/Killazz17) — Sebastián Benavides Madrigal ·
[@FSPABLO](https://github.com/FSPABLO) · [@carta01](https://github.com/carta01)
