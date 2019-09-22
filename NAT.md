# NAT (Nerwork Address Translation)

REF: https://www.mikroways.net/2010/06/06/tipos-de-nat-y-configuracion-en-cisco/

~~~
1.- static NAT (1:1)

Consiste basicamente en un tipo de nat en el cual se mapea una direccion
IP privada con una direccion IP publica de forma estatica. De esta manera,
cada equipo en la red privada debe tener su correspondiente IP publica
asignada para poder acceder a internet. La principal desventaja de este
esquema es que por cada equipo que se desee que tenga acceso a internet
se debe contratar una IP publica. Ademas, es posible que haya direcciones
IP publicas sin usar (porque los equipos que las tienen asignadas estan
apagados, por ejemplo), mientras que hay equipos que no pueden tener acceso
a internet (porque no tienen ninguna IP publica mapeada).

2.- dynamic NAT

Este tipo de NAT pretende mejorar varios aspectos del static NAT dado que
utiliza un pool de IPs publicas para un pool de IPs privadas que seran
mapeadas de forma dinamica y a demanda. La ventaja de este esquema es que
si se tienen por ejemplo 5 IPs publicas y 10 maquinas en la red privada,
las primeras 5 maquinas en conectarse tendran acceso a internet. Si
suponemos que no mas de 5 maquinas estaran encendidas de forma simultanea
nos garantiza que todas las maquinas de nuestra red privada tendran salida
a internet eventualmente.

3.- NAT overload (PAT)

El caso de NAT con sobrecarga o PAT (Port Address Translation) es el mas
comun de todos y el mas usado en los hogares. Consiste en utilizar una
unica direccion IP publica para mapear multiples direcciones IPs privadas.
Las ventajas que brinda tienen dos enfoques: por un lado, el cliente
necesita contratar una sola direccion IP publica para que las maquinas
de su red tengan acceso a internet, lo que supone un importante ahorro
economico; por otro lado se ahorra un numero importante de IPs publicas,
lo que demora el agotamiento de las mismas.

La pregunta es obvia es como puede ser que con una unica direccion IP
publica se mapeen multiples IPs privadas. Bien, como su nombre lo indica,
PAT hace uso de multples puertos para manejar las conexiones de cada host
interno. Veamos esto con el siguiente ejemplo:

La PCA quiere acceder a www.netstorming.com.ar

El paquete esta formado por:
  - IP origen: PCA		
  - Puerto origen: X
  - IP destino: www.netstorming.com.ar
  - Puerto destino: 80

Al llegar al router que hace PAT, el modifica dicha informacion por la
siguiente:
  - IP origen: router
  - Puerto origen: Y
  - IP destino: www.netstorming.com.ar
  - Puerto destino: 80

Ademas, el router arma una tabla que le permite saber a que maquina de la
red interna debe dirigir la respuesta. De esta manara, cuando recibe un
segmento desde el puerto 80 de www.netstorming.com.ar dirigido al puerto Y
del router, este sabe que debe redirigir dicha informacion al puerto X de
la PCA.
~~
