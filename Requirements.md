Visualización de versiones y controles de seguridad de dependencias

#### **1. ¿Qué problemas presenta en términos de seguridad y cómo los solucionarías?**
- (Incluye: _Version pinning vs hashing_ y opcionalmente _Lock Files vs Input Files_)

##### **Identificación de problemas de seguridad:**
En el archivo `requirements.txt` se puede observar diversos problemas de seguridad, como:
- Se utiliza versiones fijas `==` sin hashes para verificarlas.![[Pasted image 20260130160737.png]]
- No hay una clara separación entre dependencias de producción y desarrollo están todas mezcladas.![[Pasted image 20260130160813.png]]

- Y no se garantiza la reproducibilidad del entorno.
**Ejemplo:** Un caso típico ocurre cuando un desarrollador instala las dependencias en su ordenador y otro lo hace semanas después. Al no existir un archivo de bloqueo con hashes, el gestor de paquetes puede descargar versiones distintas de dependencias internas, generando comportamientos diferentes entre entornos de desarrollo y producción.

##### **Posibles riesgos:**
Los problemas anteriores pueden provocar diferentes problemas como:
- Ataques directos a la cadena de suministro (`supply chain attacks`).
`Ejemplo`: Un atacante podría publicar una versión maliciosa de una dependencia, que se instalaría automáticamente sin que el desarrollador lo detecte.

- Que se instalen paquetes modificados maliciosamente.
`Ejemplo:` Al no usar hashes, se podría descargar un paquete alterado que ejecute código no deseado en el sistema.

- Que hayan gran diferencia entre entornos de producción y desarrollo.
`Ejemplo:` El proyecto puede funcionar correctamente en el entorno de desarrollo, pero fallar en producción debido a dependencias instaladas de forma distinta.

- Y dificultad para actualizar y auditar dependencias.
`Ejemplo:` Sin un control claro de versiones y cambios, resulta complicado saber qué dependencias se usan y cuándo deben actualizarse.

##### **Soluciones para estos problemas:**
Para solucionar los problemas presentes en `requirements.txt` se debe implementar las siguientes soluciones:
- Mantener el uso de versiones fijadas (_version pinning_), pero complementarlas con hashes criptográficos para garantizar la integridad de los paquetes (_hashing_).
- Convertir el propio archivo `requirements.txt` en un archivo de bloqueo añadiendo hashes, sin modificar la estructura del proyecto.
- Utilizar herramientas como `pip-tools` para generar y mantener los hashes de forma automática.
- Aplicar buenas prácticas de revisión y actualización periódica de las dependencias.


#### **2. Modifica el archivo y entrega una versión nueva y mejorada a través de tu fork del repositorio. ¿Cómo conseguiste ese resultado?**
- (Explicas el proceso y enlazas tu fork)

Para modificar el archivo primero realizaremos un fork del repositorio original:
![[Pasted image 20260130163050.png]]

Después, clonaremos el repositorio en local para modificar el `requirements.txt`, con el comando siguiente lo clonaremos:

```sh
git clone https://github.com/miguelgual2003/sportsclub.git
cd sportsclub
```

![[Pasted image 20260201113348.png]]
Crearemos un entorno específico para el proyecto `sportsclub`. Con los siguientes comandos creremos el entorno:

```sh
python3 -m venv .venv
source .venv/bin/activate
```

![[Pasted image 20260201121333.png]]
![[Pasted image 20260201121444.png]]

Luego, copiaremos en un archivo temporal el contenido de requirements.txt para crear los hashes criptográficos:

```sh
cp requirements.txt requirements.in
```

![[Pasted image 20260201121715.png]]
![[Pasted image 20260201121736.png]]

Para generar los hashes criptográgicos ejecutaremos el siguiente comando:

```sh
pip-compile --generate-hashes requirements.in
```

![[Pasted image 20260201121525.png]]

Para terminar de configurar el `requirements.txt` distinguiremos que dependencias son de producción y desarrollo en donde solo hay la dependencia de `ruff` en desarrollo.

![[Pasted image 20260201121758.png]]
![[Pasted image 20260201121828.png]]

![[Pasted image 20260201124412.png]]
![[Pasted image 20260201124342.png]]

Después, bborraremos el archivo temporal para no subirlo al repositorio:

```sh
rm requirements.in
```

![[Pasted image 20260201125029.png]]

Añadimos `requirements.txt`:

```sh
git add requirements.txt
```

![[Pasted image 20260201125046.png]]

Configuraremos el git para poder hacer el push del respositorio:
```sh
git config --global user.email "miguelgual2003.com"
git config --global user.name "Miquel"
git push
```

![[Pasted image 20260201125104.png]]

![[Pasted image 20260201130132.png]]

Enlace al requirements.txt del fork de sportsclub: https://github.com/miguelgual2003/sportsclub/blob/main/requirements.txt
![[Pasted image 20260201130405.png]]

#### **3. Describa el procedimiento que recomendaría en un escenario de producción para realizar más modificaciones a medida que pasa el tiempo y se lanzan nuevas versiones de paquetes de software y otras versiones quedan obsoletas.**

El escenario que recomendaría para realizar modificaciones a lo largo del tiempo son:
##### **Paso 1: Monitorización**
En este paso lo que haría sería activar herramientas automáticas de detección de vulnerabilidades para nuestro repositorio.

Por ejemplo, se activa GitHub Dependabot para que avise automáticamente cuando alguna dependencia tiene una vulnerabilidad conocida.

**Dependabot** es una herramienta de seguridad de GitHub diseñada para automatizar la gestión de las librerías y paquetes (dependencias) que utiliza un proyecto. Básicamente, actúa como un "centinela" que vigila que el código no use piezas obsoletas o peligrosas.

##### **Paso 2: Actualización controlada**
Se realizaría una actualización de dependencias regenerando el archivo lock. Es decir, cuando se detecta un problema, se actualizan las dependencias regenerando el archivo de bloqueo con el comando `pip-compile --upgrade requirements.txt`, asegurando versiones seguras.

##### **Paso 3: Validación**
En este paso de validación lo que se haría es que antes de desplegarse, el proyecto ejecuta pruebas automáticas y escaneos de seguridad en un sistema de integración continua (CI) para comprobar que todo funciona correctamente.

##### **Paso 4: Despliegue**
Antes de pasar a producción los cambios se prueban primero en un entorno de pruebas o staging y, si no se detectan errores, se despliegan finalmente en producción.

#### **4. ¿Qué escenarios se pretenden prevenir con esta nueva versión?**

Con todas las medidas anteriormente descritas se previenen vulnerabilidades como:
- Ataques a la cadena de suministro (`supply chain attacks`)
- Uso de paquetes con vulnerabilidades conocidas.
- Cambios inesperados entre entornos.
- Instalaciones no reproducibles.
- Errores derivados de dependencias obsoletas.
