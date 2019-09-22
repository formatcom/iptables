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
