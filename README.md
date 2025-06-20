# Kernel-Level GPIO Interface Driver for Linux

**Asignatura:** Programación de Sistemas Embebidos en Linux (2025-1S)
**Autores:** Juan Camilo Perdomo, Campos Herney Tulcan

---

## 1. Resumen 

Este documento detalla el diseño, implementación y validación de un **controlador de dispositivo de carácter para el kernel de Linux**. El propósito del módulo es abstraer el control de un pin de Entrada/Salida de Propósito General (GPIO) y exponer una interfaz de archivo estandarizada al espacio de usuario, cumpliendo con los requisitos del proyecto final de la asignatura.

El sistema completo consiste en un módulo del kernel cargable (`gpiodev_driver.ko`) y una aplicación cliente en C (`gpio_controller`), que permite la manipulación de un componente de hardware externo (un LED) a través de comandos simples desde la terminal.

---

## 2. Objetivos del curso

Este proyecto fue diseñado para abordar directamente los objetivos de aprendizaje y requisitos técnicos de la asignatura.

### Objetivos de aprendizaje cumplidos
- **Arquitectura de drivers:** Se demuestra la comprensión de la arquitectura de drivers de carácter mediante la implementación de las operaciones de archivo (`open`, `release`, `read`, `write`).
- **Exposición de módulos:** El driver registra un dispositivo y crea un nodo en `/dev/gpiocontrol`, haciéndolo accesible para las aplicaciones.
- **Control de GPIO en el Kernel:** Se utiliza la API `gpiod` moderna para una gestión robusta y segura del pin GPIO desde el espacio del kernel.
- **Diseño de aplicaciones de usuario:** La aplicación `gpio_controller` interactúa correctamente con el nodo del dispositivo utilizando llamadas al sistema estándar.
- **Prácticas de ingeniería de software:** El proyecto está documentado, organizado en un repositorio Git y validado mediante un plan de pruebas sistemático.

### Requisitos técnicos cubiertos
| Requisito | Implementación en este proyecto |
| :--- | :--- |
| **Módulo del Kernel** | `gpiodev_driver.c` implementa un módulo de carácter en C estándar. |
| **Gestión de Dispositivo** | Se registra el dispositivo (`register_chrdev`) y se gestiona el número `major`. |
| **Operaciones de Archivo** | Se implementan `open`, `release`, `read` y `write` en la estructura `fops`. |
| **Control GPIO** | El driver controla un pin de salida para reflejar los comandos `on`/`off`. |
| **Trazabilidad** | Se utiliza `pr_info()`, `pr_warn()` y `pr_alert()` para el logging (`printk`). |
| **Aplicación de Usuario** | `gpio_controller.c` es un cliente en C que usa `open`/`read`/`write`. |
| **Interfaz de Usuario** | La aplicación acepta argumentos de línea de comandos (`on`, `off`, `status`). |
| **Compilación**| Se proveen `Makefiles` para la compilación en el sistema de destino. |
| **Despliegue** | El flujo de `insmod`, `dmesg` y creación de nodo está documentado. |

---

## 3. Arquitectura y diseño técnico

El sistema está diseñado en tres capas que interactúan para proporcionar un control de hardware seguro y eficiente:

1.  **Capa de Hardware:** Compuesta por un LED conectado a un pin GPIO de la placa de desarrollo (ej. Raspberry Pi).

2.  **Capa del Kernel (Driver `gpiodev`):**
    * **Registro del Dispositivo:** Utiliza `register_chrdev` para obtener un número `major` dinámico.
    * **Interfaz GPIO:** Emplea la API `gpiod` para solicitar y gestionar el pin GPIO. Este método es el estándar actual, ofreciendo más seguridad que el enfoque `sysfs` (legacy).
    * **Operaciones de Archivo:** Implementa `open`, `release`, `read` y `write` para procesar las llamadas al sistema.
    * **Creación Automática del Nodo:** Interactúa con `udev` a través de `class_create` y `device_create` para la generación automática del nodo `/dev/gpiocontrol`.

3.  **Capa de Usuario (Cliente `gpio_controller`):**
    * Una aplicación cliente en C que interactúa con el driver a través de llamadas al sistema estándar (`open`, `read`, `write`, `close`).

---

## 4. Entregables del proyecto

Todos los artefactos requeridos se encuentran organizados en este repositorio Git.

| Entregable | Ubicación en el Repositorio |
| :--- | :--- |
| 1. Código Fuente del Módulo del Kernel | `kernel_driver/gpiodev_driver.c` |
| 2. Aplicación de Espacio de Usuario | `userspace_app/gpio_controller.c` |
| 3. Makefiles | `kernel_driver/Makefile` y `userspace_app/Makefile` |
| 4. Documentación (README) | `README.md` (este archivo) |
| 5. Artefactos de Prueba | `documentation/validation_log.md` |
| 6. Repositorio Git | El proyecto completo está versionado con Git. |

---

## 5. Flujo de trabajo y despliegue

### 5.1. Requisitos de compilación

- `gcc` y `make` (paquete `build-essential`).
- Cabeceras del kernel correspondientes a la versión en ejecución (`linux-headers-$(uname -r)`).

### 5.2. Proceso de compilación

1.  **Compilar el driver del kernel:**
    ```bash
    cd kernel_driver/
    make
    ```

2.  **Compilar la aplicación cliente:**
    ```bash
    cd userspace_app/
    make
    ```

### 5.3. Despliegue y ejecución

1.  **Cargar el módulo en el kernel:**
    ```bash
    # El pin GPIO es parametrizable. Por defecto, se usa el 21.
    sudo insmod kernel_driver/gpiodev_driver.ko gpio_pin=21
    ```

2.  **Verificar carga:**
    Revise los últimos mensajes del kernel para confirmar la correcta inicialización.
    ```bash
    dmesg | tail
    ```

3.  **Interactuar con el dispositivo:**
    La aplicación `gpio_controller` se maneja a través de argumentos de línea de comandos.

    | Comando | Descripción |
    | :--- | :--- |
    | `./gpio_controller on` | Establece el pin GPIO en estado ALTO (enciende el LED). |
    | `./gpio_controller off` | Establece el pin GPIO en estado BAJO (apaga el LED). |
    | `./gpio_controller status` | Lee y muestra el estado actual del GPIO. |

---

## 6. Licencia

Este proyecto se distribuye bajo la **Licencia Pública General GNU (GPL)**.

