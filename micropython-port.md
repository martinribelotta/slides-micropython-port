<!-- $theme: gaia -->

## ==Micropython en EDU-CIAA==
# ![](images/micropythonpowered-art.png)
# ![](images/ciaa-logo.png)

----

### ==¿Que es micropython?==

 - Interprete de Python ligero y minimalista
 - Pensado para microcontroladores
 - Orientado a tiempo real y control
 - No requiere sistema operativo

----
## Diferencias entre CPython y Micropython

----

|                |     CPython       |    Micropython    |
|---------------:|:------------------|:------------------|
| Memoria Minima | 1...5MB           |  2K...8K          |
| Con pilas      | 30M...XGB         |  32K...64K        |
| Plataformas    | OS Unix, Win, ... | Sin sistema operativo|
| Librerias      | Red, analisis numerico, templates, IA | Control de motores, IoT, procesamiento de seĺal, redes industriales |

----
### ==¿Que es EDU-CIAA?==

  - Parte del proyecto CIAA
  - `CIAA` *Computadora Industrial Abierta Argentina*
  - Desarrollo colectivo de hardware libre
  - Pensado para la industria
  - Con versiones educativas para escuelas y universidades
  - $EDU + CIAA =$ CIAA Educativa

----

###### ==EDU-CIAA-NXP==
### ![](images/diagrama-edu-ciaa.svg)

----
###### ==LPC4337==
### ![](images/lpc4337-internal.png)

----
### ==Micropython en EDU-CIAA==
## ![](images/micropython-diagram.svg)

----
### ==Mapa de memoria==
## ![](images/mmap.svg)

----
#### ==Estructura de carpetas==

```text
/-
 +- ciaa-nxp
 |   +- board_ciaa_edu_4337: Board HAL
 |   +- lpc_chip_43xx: SoC HAL
 |   +- frozen: Modulos python frozen
 |   +- py: Flash filesystem dir
 |   +- testing: Test sinteticos
 |   +- main.c: Entry point
 |   +- mod*.c: Modulos en C
 |   +- mpconfigport.h: Configuracion del port
 |   `- *.c|*.h: Otros archivos de soporte
 +- py: Codigo del interprete/compilador
 +- drivers: Drivers independientes de la plataforma
 +- lib: Librerias de soporte (readline, fatfs, etc.)
 `- <otros>: Otras plataformas

```

----
#### ==main.c simplificado==
```c
int main(int argc, char **argv) {
    memset(HEAP_START, 0, HEAP_SIZE); /* Heap init   */
    gc_init(HEAP_START, HEAP_END);    /* gc init     */
    mp_init();                        /* system init */
    mp_hal_init();                    /* drivers+hal */
    init_flash_fs();                  /* flash fs    */

    /* Try to execute main.py fron flash filesystem  */
    if (!pyexec_file("/flash/Main.py"))
        error("\nMain.py not found\n");

    for (;;)       /* Infinite loop. CTRL+C break it */
        if (pyexec_friendly_repl() != 0)
            break;

    system_reset(); /* System reset (main not return) */
}
```

----
###### ==mod*.c==

![](images/py-modules.svg)

----

## ==Estructura de un modulo mod*.c==

##### Definicion del modulo utimer
```c
STATIC const mp_map_elem_t time_module_globals_table[] = {
    { MP_OBJ_NEW_QSTR(MP_QSTR___name__),
      MP_OBJ_NEW_QSTR(MP_QSTR_utime) },
    /* Otras entradas en el modulo */
};

STATIC MP_DEFINE_CONST_DICT(time_module_globals,
	time_module_globals_table);

const mp_obj_module_t mp_module_utime = {
    .base = { &mp_type_module }, /* tipo modulo */
    .name = MP_QSTR_utime,
    .globals = (mp_obj_dict_t*)&time_module_globals,
}
```

----

#### 