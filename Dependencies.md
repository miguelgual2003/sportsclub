#### **1. Github incluye una herramienta para escanear vulnerabilidades conocidas en las dependencias de paquetes, así como para informar de nuevas versiones.** 
- Pasos para activarlo 
- Uso 
- Resultados en tu fork 
- ¿Hace falta cambiar algo?

##### **Paso 1: Activar la herramienta**

GitHub incluye **Dependabot** que es una herramienta para escanear vulnerabilidades y sugerir actualizaciones, para poder activarla:

1. Hay que estar en el **fork del repositorio** en GitHub.
2. Hacer clic en **Settings → Security & analysis**.
![[Pasted image 20260201145212.png]]

3. Donde hay que activar las siguientes opciones para activarla:
    - **Dependency graph**
![[Pasted image 20260201145241.png]]
    - **Dependabot alerts**
![[Pasted image 20260201145301.png]]
    - **Dependabot security updates**
![[Pasted image 20260201145316.png]]

Esto permite que GitHub analice automáticamente todas las dependencias listadas en `requirements.txt`.

##### **Paso 2: Uso**

1. Una vez lista GitHub automáticamente escanea `requirements.txt` en busca de vulnerabilidades conocidas (CVE).

2. Si detecta una vulnerabilidad:
    - Crea una **alerta** en la sección de Security → Dependabot alerts
    - Y puede abrir un **Pull Request** automático con la actualización de la dependencia.

3. También informa si hay nuevas versiones disponibles de los paquetes.

##### **Paso 3: Resultados en el fork**

**Sin la nueva versión de Django 6.0.2:**
Sin la versión nueva de la dependecnia de Django Dependabot no detecta ningún error.
![[Pasted image 20260201145653.png]]

**Con la nueva versión de Django:**
- GitHub ha detectado **varias vulnerabilidades en la dependencia `Django`**, declarada en `requirements.txt`.

- En la pestaña **Security → Dependabot alerts** aparecen:
- **Vulnerabilidades de severidad alta**:
    - _Django has an SQL Injection issue_
- **Vulnerabilidades de severidad baja**:
    - _Django has Inefficient Algorithmic Complexity_.

- Todas las alertas indican que las **versiones afectadas** son: `>= 6.0a1, < 6.0.2` y su **versión corregida** es: `6.0.2`

**Dependabot** funciona de manera adecuada en el fork, identifica vulnerabilidades reales, reporta su nivel de gravedad y sugiere una versión corregida, sin embargo, la actualización se encuentra bloqueada debido a un **Pull Request** que ya está presente.

![[Pasted image 20260204185519.png]]
![[Pasted image 20260204185606.png]]
![[Pasted image 20260204185659.png]]

#####  **Paso 4: ¿Hace falta cambiar algo?**

**Sin la nueva versión de Django 6.0.2:**
Con la versión de la dependencia 6.0 no hace falta modificar nada en nuestro fork.

**Con la nueva versión de Django: 6.0.2**
**Sí,** es necesario cambiar la dependencia que ha sido comprometida, en este caso `Django`, a la versión revisada (`6.0.2`) para erradicar las vulnerabilidades identificadas. Con el fin de resolver el inconveniente, he modificado el `requirements.txt` cambiando la versión de Django a `6.0.2`.


![[Pasted image 20260204195539.png]]

Debido a la modificación de versión he debido de generar el hash criptográfico de esta versión:

![[Pasted image 20260204195434.png]]

Una vez actualizada la dependencia, las alertas de Dependabot desaparecerán y el repositorio quedará protegido frente a estas vulnerabilidades concretas.

![[Pasted image 20260204192101.png]]

#### **2. Si alojara este repositorio en su nube privada (es decir, sin utilizar Github), ¿cómo lograría la misma funcionalidad?** 
- Al menos 3 alternativas 
- Pros y contras 
- Comunidad 
- Uso 
- Elegir una, instalarla, ejecutarla y (si es posible) integrarla en CI



##### **Paso 1: Buscar alternativas**

He buscado alternativas para alojar el repositorio en una nube privada y son:
1. **Trivy** (Aqua Security) **se** **trata de** una herramienta **de** **código** **abierto creada** por Aqua Security que **facilita** **el** **escaneo de** vulnerabilidades **reconocidas** (CVEs).
	- Pros: Ligero, escaneo rápido, compatible con contenedores y ficheros de dependencias.
	- Contras: Menos integración directa con Python que Dependabot.
	- Comunidad: Activa y en GitHub.
	- Uso: para escanear el proyecto:
```sh
trivy fs .
```


2. **OWASP Dependency-Check** **se** **trata** de una herramienta de **código** **abierto** del proyecto **OWASP** que **detecta** vulnerabilidades conocidas (CVEs) en **las** dependencias de software.
    - Pros: Escaneo de CVEs muy completo, funciona con muchos lenguajes.
    - Contras: Requiere Java, más pesado.
    - Comunidad: Amplia, soporte activo.
    - Uso: 
```sh
./bin/dependency-check.sh \
  --project "SportsClub" \
  --scan . \
  --format HTML
```

2. **Safety (Python)** **se** **trata** de una herramienta de **código** **abierto** **diseñada** **particularmente** **para** **proyectos** en **Python** que **examina** las dependencias **mencionadas** en **documentos** como `requirements.txt` y las **contrasta** con una base de datos de vulnerabilidades **reconocidas** (CVEs).
    - Pros: Ligero, enfocado a Python, fácil integración con CI.
    - Contras: Base de datos de CVE menos amplia que OWASP.
    - Comunidad: Activa en Python, open source.
    - Uso:
    ```sh
    pip install safety
    safety check -r requirements.txt
    ```

##### **Paso 2: Elegir una alternativa**

Para nuestro repositorio en este caso como es con Python, la más práctica para implementarse es `Safety` porque nos permite escanear el `requirements.txt` y bloquear los builds si hay alguna vulnerabilidad crítica.

##### **Paso 3: Instalar y ejecutar Safety**

1. Instalar:
```sh
pip install safety
```

![[Pasted image 20260202180000.png]]

2. Ejecutar escaneo en el proyecto:
```sh
safety check -r requirements.txt
```

![[Pasted image 20260202180144.png]]

3. Resultado: lista de paquetes vulnerables y su nivel de gravedad.
![[Pasted image 20260202180224.png]]


##### **Paso 4: Integración en CI**

Para integrar Safety en la pipeline CI lo que haremos es abrir el ci.yml en la ruta `./gituhub/workflows` en github introduciremos un job nuevo con las siguientes instrucciones de la herramienta.

```yaml
jobs:
  # Definición del "safety-scan"
  safety-scan:

    # Sistema operativo donde se ejecutará el job.
    runs-on: ubuntu-latest

    steps:
      # Descargar el código del repositorio.
      - name: Checkout repository
        uses: actions/checkout@v4

      # Configurar versión de Python.
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      # Instalar la herramienta Safety.
      - name: Install Safety
        run: pip install safety

      # Ejecutar el análisis de vulnerabilidades.
      - name: Run Safety check
        run: safety check -r requirements.txt
```

Link al ci.yml del fork de sportsclub: https://github.com/miguelgual2003/sportsclub/blob/main/.github/workflows/ci.yml
![[Pasted image 20260203153944.png]]

Safety se integró en la pipeline mediante GitHub Actions, añadiendo una fase de escaneo de dependencias.

![[Pasted image 20260203154932.png]]

En cada push o pull request, la pipeline instala Safety y analiza el archivo `requirements.txt`. Si se detectan vulnerabilidades de severidad alta o crítica, la ejecución falla automáticamente, evitando el despliegue de código inseguro.