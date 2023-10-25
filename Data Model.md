# Modelo de datos Kueski

## Introducción

Kueski es una importante compañia de BNPL, la cual presta una solucion de pago en quincena que faciliata a los clientes comprar cualquier producto de los socios comerciales.

Como parte del presente trabajo se espera resolver las siguientes preguntas:

    1.- ¿Cuántos préstamos se desembolsaron en Diciembre del 2021?
    2.- ¿El puntaje de riesgo y fraude ayudan a decidir que aplicación se desembolsa?
    3.- ¿Cuáles son los mejores 5 estados en loans desembolsados?
    4.- ¿Cuánto ingreso por interés tuvo la compañía por semana?
    5.- ¿Los clientes prefieren obtener un préstamo con un plazo específico?
    6.- ¿Los clientes prefieren obtener un préstamo para comprar en un comercio en específico?
Por otro lado, se espera encontrar patrones de comportamiento e información relevante de los siguientes conceptos:

    Definición 1. La recurrence_1 se define como la cantidad de préstamos desembolsados de manera histórica para un cliente a la fecha de desembolso, incluyendo el actual.
    Definición 2. Diremos que la recurrence_2 es verdadera si dado un loan, cumple alguna de las siguientes condiciones:
        - Si el cliente ha pagado algún préstamo en su totalidad en el día de desembolso.
        - Si el cliente ha pagado 3 o más quincenas de su primer préstamo desembolsado, en el día de desembolso.

## Metodología

Para la resolución de las preguntas planteadas usaremos lo _datasets_ loan_data.csv y repayment_data.csv. Así como la metodología Kimball para establecer un modelo de datos que nos ayude con ésto junto con Python para la transformación de los datos y desarrollo de las recurrencias.

Por otro lado, usaremos Google Draw para la parte gráfica (diagrama ERD) del modelo dimensional. Usaremos el Datawarehouse Google Bigquery para el almacenamiento y consumo de los _datasets_ proporcionados, así como el diseño físico del modelo de datos. Por último, usaremos Google Looker Studio para la presentación y visualización de los datos.

## Resultados

Este sección constará de 3 partes:

* Diseño y desarrollo del modelo de datos usando la metodología Kimball.
* Desarrollo de las recurrencias.
* Resolución de preguntas planteadas y principales hallazgos.

### Diseño y desarrollo del modelo de datos

Empezaremos estableciendo los 4 pasos del proceso de diseño dimensional.

#### 4 pasos de proceso de diseño dimensional

##### Paso 1. Seleccionar el proceso de negocio

Un proceso de negocio es una actividad de bajo nivel realizada por una organización. Estos procesos suelen expresarse como verbos de acción porque representan las actividades que realiza la emprese.

En Kueski, el proceso de negocio puede describirse como:

    Proporcionar préstamos con interés y la recuperación de éstos en quincenas.

##### Paso 2. Declarar la granularidad

La granularidad (o _grain_ en inglés) es especificar exactamente lo que representa una fila individual de la _fact table_. Transmite el nivel de detalle asociado a las medidas y responde a la pregunta: "¿Cómo se describe una única fila de la tabla de hechos?".
En Kueski, la granularidad se describe como:

    Una fila por aplicación de préstamo. Esto es, un cliente que haya realizado el trámite para obtener el préstamo, se haya desembolsado o no. Así como una fila por pago realizado del préstamo.

##### Paso 3: Identificación de las dimensiones

Las dimensiones surgen de la pregunta: "¿Cómo se describen los datos resultantes de los eventos de medición de los procesos de negocio?". Si se tiene clara la granularidad, las dimensiones suelen ser fáciles de identificar, ya que representan el "quién, qué, dónde, cuándo, por qué y cómo" asociado al evento. En Kueski las dimensiones son:
>
    - Fecha
    - Cliente
    - Comercio
    - Préstamo
    - Pago

##### Paso 4: Identificación de los hechos/_facts_

Los _facts_ se determinan respondiendo a la pregunta: "¿Qué mide el proceso?" En Kueski los _facts_ son:
>
    - Monto del préstamo
    - Monto pagado
    - Monto pagado a la fecha de corte
    - Interés pagado
    - Plazo/quincenas
    - Tasa de transacción
    - Puntaje de riesgo
    - Puntaje de fraude

#### Diagrama ERD

##### Diagrama Loan Data

![Diagrama ER Loan Data](/ERD/imageLD.png "Diagrama ER Loan Data")

##### Diagrama Repayment Data

![Diagrama ER Repayment Data](/ERD/imageRD.png "Diagrama ER Repayment Data")

### Desarrollo de las recurrencias

Para el cálculo de las recurrencias planteadas se utilizo el framework de python Pandas.

Para recurrence_1, dado un cliente (customer_id) en el conjunto de clientes únicos. Se definió la variable dln, la cual toma el valor de 1 si la variable is_disbursed=True, en caso contrario toma el valor de cero. Después se consideraron todos los préstamos realizados por el cliente y se creó una lista con la variable dln de esos préstamos. De esta manera la recurrencen_1 es igual al índice que ocupa el préstamo dentro de la lista + dln, que es igual a 1. De esta manera si el cliente solo tuvo un préstamo, entonces la lista de este usuarios tiene la forma [1] y así el índice que ocupa el préstamo es 0 y la suma nos da 1. Por otro lado, si el cliente no ha conseguido un préstamo, entonces su lista tiene la forma [0] y así la suma del índice que ocupa la aplicación (en este caso, cero) más el dln (en este caso, cero) es igual a cero.

Por otro lado, para recurrence_2. Primero se definió la variable recurrence_2=0. Ahora notamos que si recurrence_1 es igual a 0 o 1, entonces recurrence_2=False, esto porque dada una aplicación, si esta aplicación tiene recurrence_1=0, es porque para ese cliente no ha tenido ningún préstamo, entonces no se cumplen las condiciones de recurrence_2. Por otro lado, si el recurrence_1=1, entonces el cliente no ha podido pagar algún préstamo en su totalidad al día de desembolso de préstamo en curso pues es su primer prestamo. De la misma manera, el cliente no ha podido pagar 3 o más quincenas porque es su primer préstamo y se lo acaban de otorgar. Una vez observado esto, el cálculo se centra en encontrar los préstamos que cumplan la definición de recurrence_2.

### Resolución de preguntas planteadas

#### ¿Cuántos préstamos se desembolsaron en Diciembre del 2021?

Se cuentan con datos desde octubre del 2021 hasta abril del 2022 de aplicaciones realizadas. En ese periodo de tiempo se registraron 12,898 aplicaciones y 4,934 préstamos desembolsados.

Durante el mes de diciembre del 2021, se desembolsaron 568 préstamos con un monto total de 1.3M de pesos. De estos, 327 son clientes que obtuvieron su primer prestamo (recurrence_1=1).

#### ¿El puntaje de riesgo y fraude ayudan a decidir que aplicación se desembolsa?

Para esta pregunta se crearon intervalos para el puntaje de riesgo y fraude, dados de la siguiente manera:

    si risk_score >= 0.0 y risk_score<=0.3 entonces se categoriza como: 'risk_bucket1'
    si risk_score>0.3 y risk_score<=0.6 entonces se categoriza como: 'risk_bucket2'
    si risk_score > 0.6 entonces se categoriza como: 'risk_bucket3'
    en caso contrario se categoriza como null

    si fraud_score >= 0.0 y fraud_score<=0.3 entonces se categoriza como: 'fraud_bucket1'
    si fraud_score>0.3 y fraud_score<=0.6 entonces se categoriza como: 'fraud_bucket2'
    si fraud_score > 0.6 entonces se categoriza como: 'fraud_bucket3'
    en caso contrario se categoriza como null

De esta manera, observamos que las aplicaciones dentro de la categoría risk_bucket3, el 68.9% fueron préstamos desembolsados, mientras que para la categoría risk_bucket1, tan solo el 44.3% de las aplicaciones se convirtieron en un préstamo.

Por otro lado, la relación se invierte en el puntaje de fraude. De las aplicaciones en la categoría fraud_bucket1, el 39.7% se convirtieron en préstamos. Mientras que, con una diferencia menor al caso anterior, el 38% de las aplicaciones en la categoría fraud_bucket3 se convirtieron en préstamos.

Otra pregunta que puede surgir de manera natural es que tanto afecta la recurrence_2 en la decisión de desembolsar una aplicación. Esto influye fuertemente pues vemos que si la recurrence_2=true, el 100% de las aplicaciones se desembolsan. Mientras que solo el 27% de las aplicaciones con recurrence_2=false se desembolsan, esto en gran parte se debe a los usuarios a quienes no se les a concedido un préstamo o a los que solo tiene una aplicación.

#### ¿Cuáles son los mejores 5 estados en loans desembolsados?

Kueski, a la fecha de abril del 2022, tiene presencia en toda la república mexicana. La Ciudad de México es el estado con las préstamos desembolsados 822, seguido del Estado de México con 785. El estado con la menor cantidad de préstamos desembolsados es Zacatecas con 17 préstamos. Para todos los estados, la mayor cantidad de prestamos pertenecen a recurrence_1=1 y recurrence_2=false. Esto hace cada vez más evidente que el grueso de los usuarios son nuevos o no han logrado conseguir un préstamo.

Los cinco mejores estados son:

* Ciudad de México: 822 préstamos.
* Estado de México: 785 préstamos.
* Jaliso: 342 préstamos.
* Nuevo León: 324 préstamos.
* Veracruz: 292 préstamos.

#### ¿Cuánto ingreso por interés tuvo la compañía por semana?

Se cuentan con datos desde octubre del 2021 hasta agosto del 2023 de préstamos con fecha de vencimiento o se realizó el pago de alguna quincena.

En ese intervalo de fechas, se tuvo un ingreso promedio por semana de 18,210 pesos. Mientras que los usuarios que más pagan interés son los recurrence_1=1 y los de recurrence_2=false. Es interesante notar que a medida que los usuarios disponen de más préstamos, pagan menos intereses. Esto revela un comportamiento interesante del usuarios, en donde mientras más usa Kueski como método de pago, son más cumplidos al momento de realizar el pago del préstamo.

#### ¿Los clientes prefieren obtener un préstamo con un plazo específico?

De acuerdo a los datos, Kueski ofrece como máximo diferir el pago del préstamo en 12 quincenas. Las usuarios tienen una clara inclinación en diferir su compra a 6 quincenas.

Por otro lado, es interesante notar que a mayor recurrence_1 las quincenas tienden a 1. Esto de nuevo revela un comportamiento interesante del usuario, en donde mientras más usa Kueski como método de pago, menos difiere sus compras.

#### ¿Los clientes prefieren obtener un préstamo para comprar en un comercio en específico?

Kueski cuenta más de 700 comercios afiliados, con una gama completa de comerción, desde chicos hasta extra grandes.

El comercio que más buscan los usuarios es el merchant_2, en el cual el 36.8% de los usuarios prefiere diferir sus comprar a 8 quincenas. El monto promedio del préstamo en este comercio es de 4,161 pesos, esto nos ayuda a entender el porqué el 16% de los usuarios que comprar en este comercio prefieren diferir sus compras a 12 quincenas.

Por último, los usuarios suelen utilizar Kueski como método de pago para comprar en comercios de tamaño mediano. Por otro lado, los usuarios nuevos suelen preferir comprar en comercios de talla grande.

## Material extra

1. [Presentación](https://docs.google.com/presentation/d/1RZ8DREuFZapUFZIj5xNfCcWLyXfN1jnVjXx1KxkeLrk/edit?usp=sharing)

2. [Fuente: The Data Warehouse Toolkit](Reference/Kimball_The-Data-Warehouse-Toolkit-3rd-Edition.pdf)