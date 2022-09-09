# Air Quality Sensor

## Información General

En este proyecto se implementa un sensor de calidad de aire el cual mide la cantidad de partículas que hay presentes en su entorno, para implementar este se utiliza la placa la SMT32F103c8xx, el cual debe monitorear tanto la salida digital como la salida analógica del sensor MQ135. El valor analógico debe ser procesado mediante ADC. El estado de la salida digital y el valor analógico capturado se mostraran por UART, ademas se crea un filtro digital MAF

## Hardware

Para llevar acabo este proyecto, primero se debe conocer las conexiones de un sensor MQ-135, este consta con 4 pin, donde uno es la alimentación,salida digital,analógica y tierra, ademas consta con un potenciómetro interno el cual se regula dependiendo la velocidad que uno quiera que se tome la muestra ya que moviendo este, se hace mas sensible o menos. Cabe resaltar que la variable analógica se debe conectar al PIN que se activa como ADC, y el digital o logic se debe conectar a un Pin de la placa que leerá la variable analógica.

![MP511_sp.jpg](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/MP511_sp.jpg)

La información recaudada por este sensor se enviá a través de un sistema de comunicación serial UART el cual tiene 3 conexiones y estas son: tierra (gnd),Tx,Rx. importante la conexión Tx y Rx es cruzada, por lo que Rx debe estar conectada a Tx y viceversa, ya que el significado de estas son Transmisor y Receptor

![Untitled](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/Untitled.png)

## Software

Para crear el código se necesita en el programa de STM-32 seleccionar la placa y en esta habilitar el Debug, ADC, UART, Pin digital. El debug debe estar en Serial Wire, ADC tiene que tener habilitado el Continuous Conversion Mode, el Protocolo de conexión serial UART, se debe regular la velocidad en baudios del sistema en este caso usaremos la velocidad de 115200 y por ultimo el Pin que leerá la variable digital, este pin se configura directo el software de la placa activando como GPIO_imput, en nuestro caso sera el PIN_PB12.

![                 Continuous mode de ADC](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_12.09.41.jpeg)

                 Continuous mode de ADC

![                                       Baud Rate UART](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_12.11.11.jpeg)

                                       Baud Rate UART

![       Imagen de Configuracion placa STM-32 con GPIO,UART,ADC](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_13.31.15(1).jpeg)

       Imagen de Configuracion placa STM-32 con GPIO,UART,ADC

con esto listo se procede a compilar, para generar el código 

## Filtro digital MAF

El filtro digital MAF lo que hace es promediar una cantidad de muestras que en nuestro caso son 4 muestras y se promedian estas para así eliminar posibles ruidos que tengan las mediciones y tener un valor mas cercano al real, entonces que es lo que hace este filtro, para comprender de mejor manera su funcionamiento se creara una formula que explicara a grandes rasgos como funciona, para esto se asignaran 4 variables A1, A2,A3,A4, donde la variable A1= al valor **analógico** censado y A2=A3=A4=0; y luego A1=A2,A2=A3,A3=A4, y cuando se tengan todas las muestras se realiza el promedio con $P=(A1+A2+A3+A4)/4$;  entonces esto funcionaria a modo de ejemplo así 

| inicio | 1ra muestra | 2da muestra | 3ra muestra | 4ta muestra |
| --- | --- | --- | --- | --- |
| A1=0 | A1=1200 | A1=3010 | A1= 2020 | A1=3200 |
| A2=0 | A2=0 | A2=1200 | A2=3010 | A2=2020 |
| A3=0 | A3=0 | A3=0 | A3=1200 | A3=3010 |
| A4=0 | A4=0 | A4=0 | A=0 | A4=1200 |
|  |  |  |  | P=2357,5 |

 

## Realización de Código de forma cronológica

Para comenzar se crean las variables que se medirán estas serán air_data que guardara el valor analógico del sensor, y digital_data,que guardara el valor digital del sensor, estas se insertaran en int main(void), luego de esto se activa el ADC con HAL_ADC_Start(&hadc1);

![ingreso de Variables](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_13.36.07.jpeg)

ingreso de Variables

![inicio ADC](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_13.38.02.jpeg)

inicio ADC

De forma cronológica primero se creo el código para muestrear la información que se les esta entregando a los pines digital y analógica, creado estos comandos se procede a mandar la información por UART, pero no se puede hacer de forma directa, ya que, el protocolo de comunicación solo enviá caracteres, y en este caso tenemos números, entonces se recurre al comando sprintf y este convierte los valores numéricos en caracteres, la variable que transforma sprintf se guardo como air, entonces en la variable air se encontrara en forma de carácter los valores censados. al completar esto se procede a crear el filtro MAF de forma digital (por código) que promediara las señales, para esto se tuvo que declarar las variables que se usaran para el filtro, al igual que el ejemplo las variables usadas fueron A1, A2,A3,A4 y para poder promediar las cuatro mediciones se usa N=4, se crea el filtro, y se vuelve a usar sprintf, con el mismo propósito explicado anteriormente y mandar la información por uart.

### Ciclos For and IF

Ya que el propósito del filtro es empezar a funcionar cuando ya se hayan realizado 4 mediciones, entonces se creo un ciclo for infinito. Para crear un ciclo for infinito se requiere de dos condiciones que siempre se cumplirán o que nunca se cumplirán, en nuestro caso se hizo un ciclo el cual siempre se cumplirá, y gracias al ciclo For se pudo Crear el ciclo if que determinara cuando se empiece a transmitir por UART, el valor analógico, digital y del filtro, antes de que la medición empiece a tomar los 4 valores requeridos para que funcione el filtro, pasara el código por un else, el cual mandara por uart unicamente el valor analógico y digital que se maestrearon.

La explicación anterior fue la forma cronológica de como se desarrollo el filtro, las siguientes imágenes mostraran como queda finalmente

### Imágenes

![Declaración de variables ](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_14.06.07.jpeg)

Declaración de variables 

![Código funcional del sistema](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/WhatsApp_Image_2022-09-09_at_14.06.54.jpeg)

Código funcional del sistema

## Demostración funcional

corriendo el código se procede a visualizar lo que se esta enviando, para esto se utiliza “MOSERIAL” en linux, para su utilización se debe abrir el puerto donde se encuentra conectado el UART por la terminal y con esto hacer la conexión en los mismos baudios en que se programo 115200. en la imagen se puede ver como se muestra en pantalla y que a partir de la 4 medición aparece el valor del filtro MAF

![código funcionando ](Air%20Quality%20Sensor%207223c5f93ea441258dbc023cf6648b8f/Untitled%201.png)

código funcionando 

# Datos de utilidad

La programación de la placa se realizo en un notebook con el sistema operativo WINDOWS, y este no tiene los drivers necesarios para poder visualizar por command shell console, por lo que se conecto el UART a un notebook con linux, y con este poder visualizar mientras se programa con Windows.