# IPTABLES

REF: https://wiki.archlinux.org/index.php/Iptables_(Espa%C3%B1ol)

iptables se usa para inspeccionar, modificar, reenviar, redirigir
y/o eliminar paquetes de red de IP. El codigo para filtrar paquetes
de IP ya esta integrado en el kernel y esta organizado en una
coleccion de tablas, cada una con un proposito especifico. Las
tablas estan formadas por un conjuntos de cadenas predefinidas, y
las cadenas contienen reglas que se recorren en orden consecurivo.
Cada regla consiste en una condicion asociada a una aventual accion
(llamada target) que si se cumple, se ejecuta la regla; esto es, se
aplican las reglas si las condiciones coinciden. iptables es la
utilidad de usuario que le permite trabajar con estas cadenas/reglas.
La mayoria de los usuarios nuevos consideran que las complejidades del
enrutamiento IP de linux son bastantes desalentadoras, pero, en la
practica, los casos de uso mas comunes (NAT y/o cortafuegos basicos
de internet) son considerablemente menos complejos.

![tables traverse](https://www.frozentux.net/iptables-tutorial/images/tables_traverse.jpg)

La palabra minuscula en la parte superior es la tabla y la palabra en
mayuscula siguiente en la cadena, Cada paquete de IP que viene en
-cualquier interfaz de red- pasa a traves de este diagrama de flujo
de arriba habia abajo. Una confusion muy comun es creer que los paquetes
que ingresan desde, una interfaz interna para consumo interno, se maneja
de forma diferente a los paquetes desde una interfaz orientada a internet.
Todas las interfaces se manejan de la misma manera. Depende de uno definir
las reglas que las traten de manera diferente. Por supuesto, algunos
paquetes estan destinados a procesos locales, por lo tanto, vienen desde
la parte superior del grafico y se detienen en -proceso local-; mientras
que otros paquetes son generados por procesos locales; por lo tanto,
comienzan en -proceso local- y continuan hacia abajo a travez del diagrama
de flujo.

El la gran mayoria de los casos, no sera necesario utilizar las tablas
raw, mangle, o security a la vez. En consecuencia, el siguiente cuadro
muestra un flujo simplificado de paquetes de red a travez de iptables:

~~~
                               XXXXXXXXXXXXXXXXXX
                               XXX     Red    XXX
                               XXXXXXXXXXXXXXXXXX
                                        +
                                        |
                                        v
 +--------------+             +-------------------+
 |tabla:  filter| <---+       | tabla:  nat       |
 |cadena: INPUT |     |       | cadena: PREROUTING|
 +-----+--------+     |       +--------+----------+
       |              |                 |
       v              |                 v
 [proceso local]      |     ************************       +---------------+
       |              +---+ Decisión de enrutamiento +---> |tabla: filter  |
       v                    ************************       |cadena: FORWARD|
************************                                   +------+--------+
Decisión de enrutamiento                                          |
************************                                          |
       |                                                          |
       v                    ************************              |
+--------------+      +---+ Decisión de enrutamiento <------------+
|tabla:  nat   |      |     ************************
|cadena: OUTPUT|      |                +
+-----+--------+      |                |
      |               |                v
      v               |      +--------------------+
+---------------+     |      | tabla: nat         |
|tabla:  filter | +---+      | cadena: POSTROUTING|
|cadena: OUTPUT |            +--------+-----------+
+---------------+                      |
                                       v
                               XXXXXXXXXXXXXXXXXX
                               XXX    Red     XXX
                               XXXXXXXXXXXXXXXXXX
~~~

## Tablas

iptables cuenta con cinco tablas:
-------------------------------
1. raw filtra los paquetes antes que cualquier otra tabla. Se utiliza
principalmente para configurar exenciones de seguimiento de conexines
en combinacion con el objetivo NOTRACK.
2. filter es la tabla por defecto (si no se pasa la opcion -t).
3. nat se utiliza para la traduccion de direcciones de red (por
ejemplo, la redireccion de puertos). Debido a las limitaciones en
iptables, el filtrado no se debe hacer aqui.
4. mangle se utiliza para la alteracion de los paquetes de red
especializados.
5. security se utiliza para reglas de conexion de red Mandatory Access
Control (por ejemplo, SELinux).

En la mayoria de los casos al uso, solo utilizara dos de estas tablas:
filter y nat. Las otras tablas estan destinadas a configuraciones
complejas que afectan a multiples enrutadores y decisiones de enrutamiento.


## Cadenas

Las tablas contienen cadenas, que son listas de reglas que se siguen
consecutivamente. Por defecto, la tabla filter contiene tres cadenas
integradas: INPUT, OUTPUT, FORWARD que se activan en diferentes puntos
del proceso de filtrado de paquetes de red, como se ilustra en el
diagrama de flujo. La tabla nat incluye las cadenas PREROUTING,
POSTROUTING, y OUTPUT.

vease iptables(8) para optener una descripcion de las cadenas integradas
en otras tablas.

Por defecto, ninguna de las cadenas contiene ninguna regla. Depende de
uno agregar reglas a la cadenas que quiere usar. Las cadenas hacen lo
definido en su politica predeterminada, que generalmente se establece en
ACCEPT, pero se puede restablecer a DROP, si quiere asegurarse de que
nada se desliza a traves de su conjunto de reglas. la politica
predeterminada siempre se aplica al final de cada cadena solamente. Por
lo tanto, el paquete de red debe pasar por todas las reglas existentes en
la cadena antes de aplicar la politica predeterminada.


## Reglas

El filtrado de los paquetes de red se basa en rules, que se especifican
por diversos matches, y un target (accion a tomar cuando el paquete
coincide con la condicion plenamente).

Los targets se especifican mediante la opcion -j o --jump. Los targets
pueden ser bien cadenas definidas por el usuario, bien uno de los targets
integrados especiales, o bien una extension de target. Los targets
integrados son ACCEPT, DROP, QUEUE y RETURN; las extensiones de target son,
por ejemplo, REJECT y LOG. Si el objetivo es un target integrado, el destino
del paquete es decidido inmendiatamente y el procesamiento del paquete de
red en la tabla actual se detiene. Si el objetivo (target) es una cadena
definida por el usuario y el paquete supera con exito esta segunda cadena,
se movera a la siguiente regla de la cadena original. Las extensiones de
target pueden ser tanto terminating (como los targets integrados) como
no-terminanting (como las cadenas especificadas por el usuario). Vease
iptables-extensions(8) para obtener mas detalles.

## cadenas transversales

Un paquete de red recibido en cualquier interfaz atraviesa las cadenas
de control de trafico de las tablas en el orden que se muestra en el 
diagrama. La primera decision de enrutamiento implica decidir si el 
destino final del paquete es el equipo local (en cuyo caso el paquete
atraviesa las cadenas INPUT) o va a otro lugar (en cuyo caso el paquete
atraviesa las cadenas FORWARD). Las decisiones de enrutamiento posteriores
implican decidir que interfaz asignar a un paquete saliente. En cada cadena
subsiguiente, cada regla de esa cadenase atraviesa en el orden fijado, y
siempre que una regla coincida, se ejecuta la accion, target/jump,
correspondiente. Los 3 objetivos (targets) mas comunmente utilizados son
ACCEPT, DROP y jump para una cadena definida por el usuario. Mientras que
las cadenas integradas pueden tener politicas predeterminadas, las cadenas
definidas por el usuario no pueden. Si todas las reglas de una cadena no
proporcionan una coincidencia completa que permita ejecutarla, el paquete
se devuelve a la cadena de llamadas. Si en algun momento se logra una
coincidencia completa para una regla con un objetivo DROP, el paquete se
descarta y no se realiza ninguna otra procesamiento. Si un paquete esta
ACCEPT dentro de una cadena, tambien estara ACCEPT en todas las cadenas
del superconjunto y no atravesara ninguna cadena mas de ese superconjunto.
Sin embargo, tenga en cuenta que el paquete continuara atravesando todas
las demas cadenas en otras tablas de la manera descrita.
