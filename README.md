<a name="br1"></a> 

Universidad De M´alaga

E.T.S Ingenier´ıa Informa´tica

Ingenier´ıa de Computadores, 3A

PROYECTO FINAL

Manual de uso de aceler´ometro Adafruit MMA8451

Disen˜o con Microcontroladores

Francisco Javier Cano Moreno

1 de Junio de 2023



<a name="br2"></a> 

Contents

1 Introducci´on

2

2 Comunicaci´on con MMA8451

2

2

3

3

3

2\.1 I2C . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

2\.2 Funciones de Comunicacion . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

2\.2.1 write register . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

2\.2.2 read register . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3 API

3

3

4

4

4

4

5

5

5

5

5

5

3\.1 Conﬁguraci´on del MMA8451 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2 Funciones de la API . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.1 Header . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.2 init MMA8451 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.3 get orientation . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.4 read . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.5 set range . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.6 get range . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.2.7 print info . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.3 Mejoras . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

3\.4 Prueba de funcionamiento . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .

1



<a name="br3"></a> 

1 Introduccio´n

El acelero´metro es un dispositivo que se encarga de medir la aceleracio´n lineal y angular y se utiliza en

much´ısimos dispositivos de nuestro d´ı a a d´ıa. Gracias a ello es capaz de detectar golpes, movimiento

o impacto. Algunos de los usos que se la da son:

• Relojes Fitness −→ Ayudan a medir el ejercicio f´ısico de la persona que lo lleva.

• Smartphones/tablets −→ Detecci´on de la orientaci´on de la pantalla

• Navegaci´on −→ Ayudan al GPS a tener una mejor precisio´n en el seguimiento.

Para poder utilizar el acelero´metro, se ha creado una API que se encarga de realizar la comunicacio´n

con el acelero´metro y de ofrecernos los registros necesarios para poder leer y escribir en ellos de una

forma sencilla.

El aceler´ometro dispone de 8 pines:

• Vin −→ Pin de alimentaci´on. Se puede conectar a los 5V de la placa directamente.

• GND −→ Pin de tierra que se conecta al GND de la placa.

• 3v3 −→ Pin de alimentaci´on pero de 3.3V.

• I1 −→ L´ınea de interrupcio´n 1.

• I2 −→ L´ınea de interrupcio´n 2.

• SDA −→ Canal I2C por el que se realiza el env´ıo de datos. Se conecta al pin SDA de la placa.

• SCL −→ Canal por el que se env´ıa la sen˜al de reloj de I2C, la cual genera la placa. Se conecta

al pin SCL de la placa.

• A −→ Este pin se utiliza para darle otra direcci´on I2C al acelero´metro (0x1C) ya que por

defecto utiliza como direccio´n 0x1D. Para cambiarla basta con conectar este pin al GND.

Las l´ıneas de interrupci´on se pueden utilizar para saber cuando los datos est´an listos para leer, cuando

se detecta un impacto o si hay movimiento. Gracias a las dos l´ıneas de interrupci´on, se pueden tener

conﬁguradas dos interrupciones diferentes.

2 Comunicaci´on con MMA8451

2\.1 I2C

El aceler´ometro utiliza un protocolo de comunicaci´on serie especial llamado I2C. Este se encarga del

env´ıo de datos entre dispositivos digitales utilizando el mismo bus, una de sus principales ventajas.

Utiliza dos cables de comunicacio´n:

• SCL: L´ınea por la que se transmite la sen˜al de reloj, que se encarga de sincronizar la transfer-

encia de los datos.

• SDA: L´ınea por la que se env´ıan los datos.

2



<a name="br4"></a> 

Cada dispositivo tiene una direcci´on asignada, en el caso del MMA8451 es la 0x1D pudi´endose cam-

biar a 0x1C si lo deseamos, y tiene un funcionamiento basado en maestro y esclavo. El maestro

se encarga de controlar la sen˜al de reloj y de iniciar y de parar la comunicacio´n. Por otro lado, el

esclavo se encarga de recibir las ´ordenes del maestro, que pueden ser escribir en cierto registro o leer

de cierto registro.

En el STM32, debemos conﬁgurar el I2C que queremos en la interfaz gr´aﬁca de conﬁguraci´on acti-

vando los pines de I2C. Ello nos aporta un manejador con el que podemos utilizar diferentes funciones

de la API de STM32 que se encargan de gestionar toda la comunicacio´n I2C en el nivel ma´s bajo, es

decir, inicio de la comunicaci´on y ﬁn, control de llegada de paquetes, env´ıo de paquetes...Adema´s, la

API que se ha creado, permite abstraer au´n ma´s esta gestio´n, como veremos en el siguiente apartado.

2\.2 Funciones de Comunicacion

2\.2.1 write register

• Nombre de la funci´on: void write register(uint8 t reg, uint8 t value, I2C HandleTypeDef

hi2c1, UART HandleTypeDef huart2)

• Descripci´on: Recibe como para´metros el registro en el que se quiere escribir, el valor que se

quiere escribir, el manejador I2C y el manejador de env´ıo por el puerto serie. La funcio´n se

encarga de escribir dicho valor en el registro indicado del MMA8451.

• Valor de retorno: Ninguno.

2\.2.2 read register

• Nombre de la funcio´n: uint8 t read register(uint8 t reg, I2C HandleTypeDef hi2c1, UART HandleTyp

huart2)

• Descripci´on: Recibe como par´ametros el registro que se quiere leer, el manejador I2C y el

manejador de env´ıo por el puerto serie. La funcio´n se encarga de leer el registro indicado en el

MMA8451.

• Valor de retorno: Devuelve el byte le´ıdo.

3 API

3\.1 Conﬁguracio´n del MMA8451

El MMA8451 tiene tres modos de funcionamiento, los cuales se conﬁguran en el registro 0x2A (deﬁndo

en el header como MMA8451 REG CTRL REG1) modiﬁcando el primer bit, que indica si es el

modo activo o no, y en el registro 0x2B (deﬁndo en el header como MMA8451 REG CTRL REG2)

modiﬁcando el tercer bit, que indica si se activa el auto sleep.

• Standby (00): En este modo, el MMA8451 est´a encendido pero no tiene todas las funcional-

idades activas. En este caso, las partes analo´gicas y los relojes internos esta´n deshabilitadas.

• Wake (01): Es un modo donde el MMA8451 tiene un funcionamiento normal y no se desconecta

hasta que se cambie el modo o se desconecte.

3



<a name="br5"></a> 

• Sleep (10): Este modo es igual que el anterior pero con una adici´on. Cuando el MMA8451

lleva un tiempo sin recibir interrupciones, se duerme para ahorrar energ´ıa. Estas interrupciones

son las comentadas en el apartado de introduccio´n.

Por otro lado, tenemos conﬁguraciones m´as concretas:

• Frecuencia de muestreo: Es el periodo de muestreo de los datos. Adopta 3 bits de periodos

diferentes que van desde los 1.56 Hz a los 800 Hz. Por defecto, utiliza 800 Hz y se conﬁgura en

el 0x2A (deﬁndo en el header como MMA8451 REG CTRL REG1).

• Escalado de los datos: Es el rango ma´ximo de aceleracio´n que puede tomar el aceler´ometro.

En funcio´n de las aceleraciones que se vayan a medir, elegiremos un rango u otro, por ejemplo, si

queremos medir aceleraciones pequen˜as, elegiremos un rango ma´s pequen˜o para que sea m´as pre-

ciso. Se conﬁgura en el registro 0x0E (deﬁnido en el header como MMA8451 REG XYZ DATA CFG).

• Ruido: El modo de ruido reducido se puede activar en el mismo registro de la frecuencia de

muetreo y sirve para reducir el ruido en las mediciones para obtener m´as precisi´on en ellas.

3\.2 Funciones de la API

3\.2.1 Header

El header de la API deﬁne datos u´tiles para utilizar cuando se trabaje con el MMA8451. Estos

datos pueden ser los valores de la gravedad en distintos planetas, factores de conversi´on de grados a

radianes, los registros que se han utilizado en la elaboracio´n de las funciones, distintos valores que

pueden tomar esos registros, el valor por defecto de la direccio´n I2C del MMA8451, los prototipos

de las diferentes funciones y la estructura mma8451 t, que guarda los valores de la velocidad y la

aceleracio´n en los ejes del acelero´metro y su orientacio´n.

3\.2.2 init MMA8451

• Nombre de la funcio´n: void init MMA8451(I2C HandleTypeDef hi2c1, UART HandleTypeDef

huart2)

• Descripci´on: Inicializa el MMA8451 a una conﬁguracio´n por defecto.

• Valor de retorno: Ninguno.

3\.2.3 get orientation

• Nombre de la funci´on: void get orientation(mma8451 t \*data, I2C HandleTypeDef hi2c1,

UART HandleTypeDef huart2)

• Descripci´on: Modiﬁca la estructura data con la nueva orientaci´on le´ıda.

• Valor de retorno: Ninguno.

4



<a name="br6"></a> 

3\.2.4 read

• Nombre de la funcio´n: void read(mma8451 t \*data, I2C HandleTypeDef hi2c1,

UART HandleTypeDef huart2)

• Descripci´on: Actualiza la estructura data con los nuevos valores de la velocidad y la acel-

eracio´n le´ıdas del acelero´metro.

• Valor de retorno: Ninguno.

3\.2.5 set range

• Nombre de la funci´on: void set range(mma8451 range t range, I2C HandleTypeDef hi2c1,

UART HandleTypeDef huart2)

• Descripci´on: Actualiza el rango del MMA8451 con el nuevo rango.

• Valor de retorno: Ninguno.

3\.2.6 get range

• Nombre de la funcio´n: mma8451 range t get range(I2C HandleTypeDef hi2c1,

UART HandleTypeDef huart2)

• Descripci´on: Lee el rango del MMA8451.

• Valor de retorno: Devuelve el rango le´ıdo.

3\.2.7 print info

• Nombre de la funci´on: void print info(mma8451 t data, UART HandleTypeDef huart2)

• Descripci´on: Muestra la informacio´n de la estructura data.

• Valor de retorno: Ninguno.

3\.3 Mejoras

Algunas de las mejoras que se pueden introducir es la capacidad de gestio´n de un evento como el

movimiento o la detecci´on de ca´ıda. Esta implementaci´on ser´ıa muy u´til para dispositivos de gestio´n

de un airbag.

3\.4 Prueba de funcionamiento

Esto ser´ıa un registro del funcionamiento del MMA8451:

POSICION

x = -614

y= 212

5



<a name="br7"></a> 

z= 1941

ACELERACION

x = -2.940080

y= 1.015141

z= 9.294291

ORIENTACION -> Landscape Left Front

POSICION

x = 224

y= -1310

z= 1713

ACELERACION

x = 1.072602

y= -6.272809

z= 8.202535

ORIENTACION -> Portrait Up Front

6


