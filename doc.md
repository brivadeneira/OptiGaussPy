## Diseño

Se generan dos formas de onda:

* Cosenoidal con factor de chirp variable.
* Envolvente de pulso gaussiano con ensanchamiento variable.

Se modula la forma de onda cosenoidal con el pulso gaussiano generado, habiendo determinado numéricamente los parámetros de **k *(factor de chirp)* ** y ** $T_0$ *(ensanchamiento temporal)* **

### Parámetros involucrados

Se definen los siguientes parámetros, necesarios para producir las dos formas de onda que finalmente conforman un pulso gaussiano con dispersión y chirp en función de la longitud del enlace de fibra óptica y el tiempo:

* $z$: Longitud de la fibra, seteada por el usuario. *[km]*
* $T_0$: Ensanchamiento temporal inicial. *[s]*
* **k**: Factor de chirp inicial, **lo setea el usuario, puede ser nulo**, por defecto se calcula según: $\frac{\Delta f}{T}$.
    * $\lambda$: Según valor típico de fabricante. *[nm]*
    * $\Delta \lambda$: Según valor típico de fabricante.
* $\beta_2$: Dispersión de la velocidad de grupo, según:
    * $\beta_2= -\frac{\lambda^2}{2\pi c}D$
        * **D**: Dispersión cromática, según valor típico del fabricante. $\frac{ps}{nm·km}$
* $k_z$, chirp a $z km$ de fibra, se calcula según:
    * $k_z = k + \frac{(1+k^2)\beta_2z}{T_0^2}$
* $\frac{T_z}{T_o}$, relación entre ensanchamiento temporal inicial y ensanchamiento temporal a $k km$, se calcula según:
    * $\frac{T_z}{T_o} = \sqrt{(1 + \frac{\beta_2zk_o}{To^2})^2+(\frac{\beta_2z}{T_o^2})^2}$
* $L_D$, longitud de dispersión, se da cuando con un chirp inicial nulo, $T_z^2 = 2T_o^2$, se calcula según:
    * $L_D = \frac{T_o^2}{\beta_2}$

### Cosenoidal con chirp

Se genera una forma de onda cosenoidal con frecuencia variable a lo largo del tiempo, que se modula con un pulso gaussiano, conformando así la forma de onda de interés.

#### Scipy Signal Chirp

Se emplea la librería **scipy** y sus herramientas para procesamiento de señales, más específicamente [**scipy.signal.chirp**](https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.chirp.html) para generar de manera dinámica formas de onda con chirp.

`y = scipy.signal.chirp(t, f0, t1, f1, method='linear', phi=0, vertex_zero=True)`

donde los argumentos de entrada son: 

* **t** es un arreglo los valores de **tiempo** en los que se evalúa el pulso.
* **f0** es una variable numérica de tipo *float* que almacena la **frecuencia** en t=0 en (Hz).
* **t1** es una variable numérica de tipo *float* que almacena el valor de **tiempo** para el cual el pulso adquiere la **frecuencia** *f1*.
* **f1** es una variable numérica de tipo *float* que almacena la **frecuencia** en t1 (Hz).
* **method** es un parámetro opcional que especifica el tipo de **barrido de frecuencia**, {‘linear’, ‘quadratic’, ‘logarithmic’, ‘hyperbolic’}, si no se especifica se asume un barrido lineal.
* **phi** variable numérica del tipo float, opcional, que almacena el desvío de fase en grados, por defecto es nulo.
* **tex_zero** variable booleana, opcional que se usa sólo si el método de barrido de frecuencia deseado es el cuadrático, determina si el vértice de la parábola de frecuencia sucede en t=0 o t=1.

Retorna:

* **y**, variable del tipo ndarray *(numpy array)* que contiene los valores de la forma de onda para cada valor de t y variación de frecuencia. Más precisamente la salida es $cos(\phi + \frac{\pi}{180}\phi)$

Siendo el chirp inicial una variable del tipo *int* seteada por el usuario, puede ser nula, por defecto es -3.

el chirp a una longitud $z$:

$k_z = k + \frac{(1+k^2)\beta_2z}{T_0^2}$

donde $\beta_2 = -\frac{\lambda^2}{2\pi c}D$

**IMPORTANTE**: Inicialmente se pensó a las variables para graficar una forma de onda con chirp como $f_o$ del transmisor y $f_1$ obtenida desde el valor de chirp calculado, sin embargo *los resultados numéricos no son útiles para la representación gráfica*, por lo que se opta por interpretar *a fines pragmáticos* a $k_z$ calculado como una relación de proporcionalidad entre $f_o$ y $f_1$. *Por ejemplo: si $k_z = 3$, entonces $f_1 = 3f_o$, $k_z = -4$, entonces $f_o = 4f_o$*.

### Pulso Gaussiano

Se genera un pulso gaussiano con la dispersión de interés, y se emplea para modular la forma de onda consenoidal con chirp.

#### Scipy Signal Gausspulse

Se emplea la librería "scipy" y más específicamente sus herramientas para procesamiento de señales y de generación de pulsos gaussianos [scipy.signal.gausspulse](https://docs.scipy.org/doc/scipy/reference/generated/scipy.signal.gausspulse.html)

`scipy.signal.gausspulse(t, fc=1000, bw=0.5, bwr=-6, tpr=-60, retquad=False, retenv=False)`

Donde:

* **retquad** es una variable *booleana*
    * *True*, retorna la parte real e imaginaria, en fase y cuadratura.
* **retenv** es una variable *booleana*
    * *True*, retorna la envolvente de la señal, sin modular.
    * *False*, retorna la señal modulada
* **t** es un arreglo los valores de **tiempo** en los que se evalúa el pulso. *O un string de valor "cutoff"*.
* **fc** variable tipo *int*, opcional que almacena el valor de la **frecuencia central** en *Hz*, por defecto es 1000 Hz.
* **bw** variable de tipo *float*, opcional, indica el ancho de banda fraccional en el dominio de la frecuencia, por defecto es *0.5*.
* **bwr** variable de tipo *float*, opcional, indica el nivel como fracción del ancho de banda en dB, por defecto es *6dB*.
* **tpr** variable del tipo *float*, opcional, si f='cutoff', la función retorna el tiempo en el que la amplitud cae a tpr, por defecto -60dB
* **retquad**, variable *booleana*, opcional,
    * True, retorna la parte real e imaginaria de la onda, por defecto es falso.
* **retenv**, variable *booleana*, opcional
    * *True*, retorna la envolvente de la señal, por defecto es falso.

Retorna:

Sinusoidal modulada con un pulso gaussiano

        exp(-a t^2) exp(1j*2*pi*fc*t).

**yI** : ndarray (arreglo numpy) con la parte real de la señal. Siempre la retorna.

**yQ** : ndarray (arreglo numpy) con la parte imaginaria de la señal. La retorna sólo si la var *retquad* es True.

**yenv** : ndarray (arreglo numpy) que contiene la envolvente de la señal. La retorna si *retenv* es True.

**fc** se setea de manera proporcional a $\frac{T_z}{T_o}$ según:

$\frac{T_z}{T_o} = \sqrt{(1 + \frac{\beta_2zk_o}{To^2})^2+(\frac{\beta_2z}{T_o^2})^2}$
