# Metro Nyam

Pràctica AP2. Explicarem els quatre moduls necessaris pel funcionament d'aquest bot.
Tria restaurant i vés-hi en metro! 🍕 🚇

## Restaurants

La classe Restaurant guarda una sèrie d'atributs que seran utilitzats en dues funcions del bot: per trobar restaurants (la funció que després veurem del bot es ```find```) on s'itera pels diferents atributs de cada restaurant per trobar els que encaixen amb la cerca de l'usuari, i per donar la informació d'un restaurant quan el usuario ho demani amb la funció ```info``` del bot.

#### Read
La funció ```read```el que fa és retorna una llista de restaurants on no hi ha cap repetit. Per a fer-ho itera per les files del fitxer ```restaurants.csv```. Ens vam fixar que es podia donar el cas de que constés a aquest fitxer com a restaurants diferents dos o més files amb el mateix nom però filtre secondari diferent. Això és si, per exemple, en la mateixa localització, un restaurant és també una xampanyeria. Així, constaria com dos restaurants quan en realitat és un. Vam observar que, si això passava, els dos o més restaurants que en realitat eren el mateix apareixien consecutius al fitxer ```csv```. Aprofitant això, a l'hora de llegir els restaurants, per tal de no tenir-ne de repetits, comprovem que el restaurant previament llegit sigui un de diferent, excepte quan es tracta del primer de tots. En cas de ser el mateix, a la llista de filtres secondaris del restaurant anterior li afegim el filtre secondari del restaurant que estem llegint.

## Metro

Aquest mòdul crea un graf que guarda totes les estacions, les connecta, i tots els accessos i els connecta a les seves respectives estacions.

#### Classes
Al mòdul metro tenim tres classes diferents: estació, accés i aresta.  

#### Read stations

La funció read stations ens retorna una llista de totes les estacions de Barcelona, algunes repetides, ja que es llegeixen del fitxer ```estacions.csv``` on estan ordenades per línia, i hi ha estacions que tenen més d'una linia. Per tant, una estació apareix tantes vegades com linies tingui.

#### Read accesses

La funció read accesses funciona exactament igual que la funció read stations i retorna una llista d'accessos.

#### Get metro graph

Aquesta és la funció principal del metro i el que fa és retorna el graf del metro, que conté totes les estacions i accessos connectats.

Primer inicialitzem un graf buit. Després, cridem dues funcions: la primera per tal d'afegir i connectar les estacions entre si i la segona per afegir els accessos i conectar-los a les seves respectives estacions.

##### Add stations

Aquesta funció carrega la llista d'estacions i itera per cada una d'ella afegint cada estació al graf com un node, amb etiqueta el seu id, i com atributs els següents: la informació de tota la estació i la posició de la estació. Vam veure que guardem la posició dues vegades però vam decidir deixar-ho perquè per després dibuixar el graf amb la funcio ```show``` és molt més eficient fer-ho cridant la funció ```get_node_attributes``` que ofereix la llibreria ```networkx``` i que retorna les posicions de tots els nodes en forma de diccionari, que no iterar per tots els nodes per extreure, de la informació de tota la estació, la seva posició.

Com la llista d'estacions està ordenada per línies, mentre recorrem aquesta llista i anem afegint cada estació com a node, si una estació té la mateixa línia que l'anterior, afegim una aresta de tipus tram que representa una línia de metro.

Un cop acabat aquest bucle, cridem la funció ```transhipments```. Aquesta connecta, per a cada estació, tots els nodes d'aquella estació amb diferents línies. Utilitzem la funció ```get_intervals``` que retorna un diccionari que guarda l'interval d'estacions de cada linia de la llista d'estacions. Això és, per exemple, tenir un element que sigui ```L1``` que tingui com a valor la llista de dos elements ```(0,30)``` que ens diu que de la llista d'estacions els elements que van des de la posició 0 fins la 30 corresponen a la linia L1. Amb això evitem fer una funció quadràtica on caldria recorrer totes les estacions, i per a cada estació recórrer totes les estacions un altre cop, i així anar connectant totes aquelles que tenen el mateix nom però diferent línia.  


##### Add accesses
Anàlogament a la funció anterior, aquesta carrega la llista d'accessos i els afegeix al graf, a la vegada que uneix tot accés amb les estacions a les que s'hi connecta. Per a fer tal connexió tot porta pensar que caldrà fer una estructura quadràtica, però fixant-nos en que els accessos vénen ordenats per la id de l'estació a la que connecten, això permet fer una cerca binària en lloc d'una cerca lineal. Així doncs, per a cada estació, busquem els acessos que s'hi connecten a partir de la funció ```binary_search```. Aquesta retorna la posició del primer accés que apareix a la llista, amb una complexitat logarítmica.



#### Display del graf

Hem implementat dues maneres de dibuixar el graf del metro: ```show``` i ```plot```.

La funció ```show``` mostra el graf per terminal represenant els nodes com a punts blaus i les arestes amb línies negres. Per a fer-ho, necessita d'un diccionari que guarda la posició de cada node. Aquest es crea molt eficientment amb la funció que ve implementada per la llibreria ```networkx```, ```get_node_attributes```.

La funció ```plot``` pinta sobre un mapa les estacions i els nodes com a punts negres i les arestes que els uneixen depenent del tipus d'aresta. Si és una aresta de tipus *```tram```*, com representa una línia de metro, és pintada amb el color de la línia. Si l'aresta connecta una estació amb una altra amb mateix nom, és a dir una aresta de tipus *```enllaç```*, o una estació amb un accéss, una aresta de tipus *```access```*, les pinta de color negre.


## City

El mòdul city retorna el graf de la ciutat, que és la fusió del graf del metro i el graf de les interseccions de la ciutat de Barcelona que ens descarreguem. La funció principal és ```build_city_graph```. Per a obtenir el graf dels carrers hem creat la funció ```get_osmnx_graph```, que amb la llibreria ```osmnx``` carrega un graf simplificat dels carrers de Barcelona.

#### Build city graph

Aquesta funció retorna el graf de la ciutat de Barcelona. Per a fer-ho, primer inicialitzem un graf buit. A aquest graf hi afegirem el graf dels carrers de Barcelona, que l'anomenem city, i el del metro. A més, connectarem els nodes accessos amb el node de tipus *```street```* més proper a cadascun.

#### add_street_graph

Aquesta funció guarda el graf dels carrers al nostre city graf. El que fa és recòrrer tots els nodes del graf i els va afegint al ```city_graf``` mantenint com a etiqueta el mateix id que té al graf ```street```. Com a atributs, guarda el tipus de node, que són tots *```street```*, i la seva posició, que la obtenim del graf ```street```.

Per cada node del graf ```street```, recorre els seus veins i, a part d'afegir-los al graf ```city```, crea una aresta entre el node i cada veí. La aresta té els següents atributs: el tipus d'aresta, que és sempre *```street```*, la distància, que la obté de la primera aresta que connecta el node i el veí (el street graf es un multigraf, el que vol dir que per dos nodes poden haver-hi més d'una aresta), el color, que hem assignat un groc, i el temps que es triga d'anar des d'un extrem a l'altre. Per trobar el temps, dividim la distancia entre els nodes per la velocitat mitja a la que una persona camina, que és de 1.5 m/s.

#### add_metro_graph

Aquesta funció guarda el graf del metro al graf ```city```. Per a fer-ho, primer recorre tots els nodes del metro i els va afegint al city. Un cop tots afegits, guardant-los amb la mateixa etiqueta que al graf ```metro``` (un identificador), el tipus de node, i la posició, passem a crear les arestes.

Per crear les arestes iterem per la llista d'arestes del graf ```metro```, que obtenim amb la funció ```metro_graph.edges```. Aquesta retorna una llista de tuples de nodes estan connectats. Per a cada aresta guardem, com amb el graf de carrers, la següent informació que obtenim directament de la informació de cada aresta del metro:  el tipus d'aresta, que pot ser de *```tram```*, *```acces```*, o *```enllaç```*, la ```distància```, el ```color```, i el ```temps```. El ```temps``` el calculem dividint la distància per la velocitat. Ara la velocitat no és sempre 1.5 m/s ja que hem de veure si el tipus d'aresta és *```tram```*. Hem decidit assignar a la velocitat mitjana del metro 8 m/s.

#### connect_accesses_to_closest_intersection

Aquesta funció connecta cada accés amb el node de tipus *```sreet```* més proper. Per a fer-ho, primer carreguem la llista d'estacions amb la funció ```read_accesses``` del mòdul metro i a continuació guardem dues llistes la posició x i y de cada accés. Això ho fem amb la funció ```get_accesses_positions``` que recorre cada accés i guarda la seva posició ```x``` i ```y``` en dues llistes.

Amb aquestes dues llistes utilitzem la funció de la llibreria ```osmnx```, ```nearest_nodes```, que ens retorna dues llistes: una que conté el node més proper a cada accés, i l'altra que guarda la distància des de l'accés al node més proper trobat. Veiem per tant que aquestes dues llistes han de tenir la mateixa mida que la llista d'accessos. Per últim, creem una aresta al nostre graf que connecti cada accés amb el seu node més proper guardant el tipus d'aresta, que és sempre *```street```*, la distància, que la obtenim de la llista trobada previament, el color i la velocitat.

#### remove_edges_from
Per tenir un graf city correcte cal afegir la comanda ```remove_edges_from``` que ens elimina els bucles que no volem entre nodes. Aquests es creen en el moment que afegim el graf ```street```, que és un multigraf.


#### Find path

Aquesta funció retorna el camí, que és una llista de nodes, més curts entre dues coordenades. Utilitzem la funció ```shortest_path``` de la llibreria networkx.

#### Display del graf

Com al mòdul ```metro```, implementem de manera idèntica les funcions ```show``` i ```plot```.

#### Display de un path

Aquesta funció guarda un mapa de la ruta entre dues coordenades seguint el camí més rapid. Per fer-ho, primer connectem la coordenada d'inici, la ```src```, amb el seu node més proper. Després anem iterant per la llista de nodes de ```path``` i els anem connectant amb línies. El ```color``` d'aquestes línies depenen del tipus d'aresta: si són arestes de tipus *```street```*, *```enllaç```*, o *```accés```* són pintades de color negre. Si són de tipus *```tram```*, es pinten del color de la línia per tal que l'usuari que utilitza el ```bot``` sàpiga quina línia agafar per anar a un restaurant.

![Representació del metro_graph]("metro.png")

## Bot

El mòdul ```bot``` és el que es connecta amb ***Telegram***. El primer que fa al executar-se és guardar les següents dues variables globals: la ```restaurants_list``` i el graf del ```city```. Per crear el graf utilitzem utilitzem la funció ```build_city_graph```, passant com a paràmetres el graf del ```metro```, que previament creem amb la funció ```get_metro_graph```del mòdul ```metro```, i el graf dels carrers de Barcelona que carreguem amb la funció ```load_osmnx_graph``` del ```city```. La funció per carregar el ```street_graph``` mira si hi ha un arxiu en el directori amb el graf guardat i, si hi és, el retorna, si no el crea i el guarda.  

El ```bot``` conté les següents commandes: ```start```, que dona la benvolguda al usuari, ```help```, que dona les instruccions a l'usuari de com utilitzar el ```bot```, ```author```, que envia els autors d'aquest projecte.

També conté la comanda ```find```, que, cridant la funció ```multiple_search```, retorna una llista de com a màxim 12 restaurants que encaixin amb la cerca de l'usuari. Aquesta cerca pot ser múltiple, és a dir que retornarà una llista que només contindrà restaurants que encaixin amb cada paraula, com ara ```/find pizza sants```, o pot ser una cerca lògica, utilitzant els opradors ```and```, ```or```, o ```not```. La única restricció que hi ha per utilitzar la cerca lògica és que no hi hagi cap espai entre mig.

De fet, veiem que la cerca múltiple no és més que una cerca lògica de múltpiles ```and```, però hem optat per deixar-ho d'aquesta manera perquè pensem que en la realitat un usuari serà la que utilitzarà, en comptes d'anar afegint ```and```, ```or``` o ```not```.
Aquesta comanda a part d'enviar per missatge la llista de dotze restaurants trobada, també la guarda en un diccionari únic per a cada usuari per tal de després poder utilitzar la comanda ```info``` o ```guide```. D'aquesta manera, podem tenir tants usuaris com vulguem fent servir el bot alhora, ja que no es barregen les cerques que fa cadascun.


La comanda ```info``` retorna la informació del restaurant escollit amb un index del 1 al 12. Aquesta és: el *nom*, l'*adreça*, el *número de telèfon*, si en té, i la llista de *filtres secundaris*.

Per últim i més imprescindible, la comanda ```guide``` és la que utilitza l'usuari per saber el camí que ha de seguir per arribar al restaurant que ha escollit, i quant temps trigarà. Hem decidit que quan l'usuari escull a quin restaurant anar, fent per exemple la comanda ```guide 5```, sent 5 l'index de la llista escollit, el ```bot``` guarda aquest índex i li demana la seva ubicació. Un cop enviada, el bot executa la funció ```path```, que enviarà una imatge de la ruta que ha de seguir la persona des de la ubicació fins al restaurant, cridant les funcions ```find_path```, que troba la ruta més ràpida donades dues coordenades. ```plot_path``` crea un arxiu de la imatge de la ruta a seguir i ```find_time_path``` dona el temps que es triga en arribar. Totes tres són del mòdul ```city```.

Cal destacar que si l'usuari no fa cap cerca i envia la seva ubicació, el ```bot``` li demana que busqui un restaurant a on anar. També, si un cop feta una cerca demana la informació, o la ruta amb un índex fora de rang, ja sigui perquè és més gran que 12 o que el tamany de la llista de restaurants retornada, tracta l'error i comunica a l'usuari que introdueixi un índex vàlid.

#

*developed by Lucas Pons and Jan Quer, Data Science and Engineering students at UPC, 2022*
