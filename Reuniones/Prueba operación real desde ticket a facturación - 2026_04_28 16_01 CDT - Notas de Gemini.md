28 abr 2026

## Prueba operación real desde ticket a facturación \- Transcripción

### 00:00:00

   
**Elian Shair Armendariz Puch:** Buenas tardes,  
**German Castro:** Hola, ¿qué tal? Se Buenas tardes, ¿Qué tal? Buenas tardes a todos. ¿Cómo  
**Angel Huberto Pulido Burgos:** Hola, buenas tardes.  
**Salvador Cervantes Franco:** Buenas tardes.  
**German Castro:** están?  
**Miguel Gomez:** Buenas tardes,  
**German Castro:** Listo, Salvador, muchas gracias por por acompañarnos. hay una funcionalidad que es la de ay octava que este ya se le avanzó a esa parte de la funcionalidad. Entonces, por eso te invitamos a este esta revisión. Esta es la revisión que hacemos de principio a fin para interoperabilidad entre los equipos, donde comenzamos con una operación y de ahí nos vamos hacia adelante hasta que este hasta que terminamos la operación y vamos viendo feedback de todo y ver qué arrectos hay para que puedan interoperar bien todos los equipos.  
**Salvador Cervantes Franco:** Ok.  
**German Castro:** Giovanni, ya te vi también.  
**Salvador Cervantes Franco:** Ah.  
**Geovanny Manuel Martinez:** Hola, ¿qué tal? ¿Cómo están?  
**German Castro:** Entonces, si quieres empezamos contigo este Elián con la funcionalidad que traías y de ahí nos vamos hacia adelante.  
**Elian Shair Armendariz Puch:** Bien, voy me Lo logran  
**Carlos Alexis Galaviz Rosas:** ¿Qué tal?  
**Elian Shair Armendariz Puch:** ver.  
**Carlos Alexis Galaviz Rosas:** Listo.  
   
 

### 00:07:34

   
**Carlos Alexis Galaviz Rosas:** Oye, ¿puedes mostrar dónde se entra otra vez la opción o explicarme porfa?  
**Elian Shair Armendariz Puch:** Voy. Aquí está. El deo está nivel hasta arriba. En el top dice clasificación y estamos en reglas  
**German Castro:** Nico.  
**Carlos Alexis Galaviz Rosas:** Okay.  
**German Castro:** Oh.  
**Elian Shair Armendariz Puch:** octavas.  
**Carlos Alexis Galaviz Rosas:** Gracias. M.  
**Elian Shair Armendariz Puch:** Okay. Bien, de lo que se ha avanzado en la parte de clasificación, eh ya por un lado ya muestra el total de de clasificaciones que están en la base de datos de CAC 2\. Eh, como se puede ver, son muchas también. Eh, podemos ver que estas tres pendientes son las que estaban de prueba. Por este lado está el buscador, el buscado. Eh, bueno, esta parte de nueva solicitud todavía no la he trabajado, la verdad, porque no sé cómo está pensado que el clasificador agregue eh machía no he visto la UX. E para la parte de clasificados, en el T de clasificados se muestran todos los que son clasificados que ya tiene el clasificador.  
   
 

### 00:08:50

   
**Elian Shair Armendariz Puch:** Aquí en buscar entidad puedes buscar por por entidad, por número de parte, por referencia, como ya estaba trabajado aquí antes. Bien, esto es lo principal, reglas octavas. Aquí es donde está el match del número de parte con la fracción. De hecho, podemos aquí está ver los estados, los vigentes, los que están por vencer,  
**German Castro:** Amén.  
**Elian Shair Armendariz Puch:** como estos dos y los vencidos. De hecho, aquí en número de permiso, aquí podemos agregar eh un número de parte para que se machee con una fracción. Está la vigencia desde y hasta la cuota SD y la cota UMC. Y los salos este se están trabajando todavía, no se ha migrado esa parte del SIC 2\. Por eso es que si nos vamos aquí, por ejemplo, a este y le damos ver detalles, el régimen todavía no está. las cuotas falta de la migración, de hecho podemos editarlas. Bien, de a partir de esto ya sale lo que es este en el tab de mercancías para  
**German Castro:** Exato.  
**Elian Shair Armendariz Puch:** que el ejecutivo pueda eh ver si una número de partida tiene una regla octava.  
   
 

### 00:10:17

   
**Elian Shair Armendariz Puch:** Eh, no sé si quieren ver esa parte también.  
**Carlos Alexis Galaviz Rosas:** Se muestra todo, por favor.  
**Elian Shair Armendariz Puch:** Bueno, bien. Este es el de mercancías que se ha estado trabajando. Ya se pusieron lo que nos habían dicho el día viernes, que era la parte de la descripción, la parte del nico y la regla octava ahorita todavía la trae automáticamente, pero si le das aquí en editar partida, campos básicos, ya te selecciona la regla octava que el clasificador trae por defecto. Puede puede ser que esta regla octava tenga otras reglas octavas. también van a aparecer aquí en lista dependiendo de lo que el ejecutivo eh necesite a agregar se va a hacer y está una opción de regla octava. Esa es la parte que se está trabajando de la de lo que es la regla octava. No sé si hay dudas o que estaría mejor aquí en la parte de que les mostré anteriormente, esta de aquí.  
**Carlos Alexis Galaviz Rosas:** Tengo una duda. Validas que no existe el permiso.  
**Elian Shair Armendariz Puch:** Vale.  
**Carlos Alexis Galaviz Rosas:** Validas que ya no existe el permiso que está dando de alta.  
   
 

### 00:11:50

   
**Elian Shair Armendariz Puch:** Sí.  
**Carlos Alexis Galaviz Rosas:** A ver, puedes darle otra vez de nuevo dar de alta un permiso.  
**Elian Shair Armendariz Puch:** Por ejemplo, aquí voy a ponerle eso. Voy a poner demo. Y aquí puedes ponerlo por empresa. Si yo pongo PKN,  
**Carlos Alexis Galaviz Rosas:** Uh.  
**Elian Shair Armendariz Puch:** estas tres empresas que aparecen son las que están repetidas que ya habamos comentado. el tipo de permiso. Le puedo poner permiso previo.  
**Carlos Alexis Galaviz Rosas:** El tipo de permiso existe en un listado de tipos o puede ser cualquier cosa.  
**Elian Shair Armendariz Puch:** Esa es la parte que quería tocar también. No sé si existe un listado oficial y y aquí en esta parte de lo que es de régimen también, ¿cómo cómo lo manejan ahorita en el CC 2?  
**Carlos Alexis Galaviz Rosas:** Aquí está Miguel y no Br lo debería,  
**Elian Shair Armendariz Puch:** Ok.  
**Carlos Alexis Galaviz Rosas:** pero sí creo que deber un catálogo de esa parte. Igual como recomendaciones, yo te diría que lejes place holders a a todos los inputs para que tengan noción de que es.  
   
 

### 00:13:39

   
**Carlos Alexis Galaviz Rosas:** No sé, Salvador, si deba de haber algún catálogo, algún anexo que lo indique por el tipo de permiso los régimenes. Yeah.  
**Angel Huberto Pulido Burgos:** Oye, eh, Elian.  
**Salvador Cervantes Franco:** En el tipo de permiso nada más sería cuestión de de sacarlo porque así como un listado que hay en algún lugar para descargar no pero sí se puede hacer, o sea, son limitados, ¿no? un registro sanitario, un permiso previo, un aviso automático. Entonces, nada más es cuestión de sería cuestión de de irlo armando.  
**Elian Shair Armendariz Puch:** Actualmente en el lo hacen escrito normal.  
**Salvador Cervantes Franco:** Creo que están los más comunes nada  
**Elian Shair Armendariz Puch:** Ah,  
**Salvador Cervantes Franco:** más.  
**Elian Shair Armendariz Puch:** okay. ¿Qué decir,  
**Angel Huberto Pulido Burgos:** Ah, te preguntaba que si a qué quedó ligado el la regla octava.  
**Elian Shair Armendariz Puch:** Ángel?  
**German Castro:** Sí.  
**Angel Huberto Pulido Burgos:** Siempre quedó ligada al cliente y al número de parte o cómo estuvo el show.  
**Elian Shair Armendariz Puch:** Sí, como había comentado Germán, iba a ser el número de parte y aquí  
**Angel Huberto Pulido Burgos:** Ah, ok.  
**Elian Shair Armendariz Puch:** sí.  
   
 

### 00:14:58

   
**Angel Huberto Pulido Burgos:** Ok.  
**Elian Shair Armendariz Puch:** Bien. E entonces esta parte de tipo de permiso sería como un Bueno, ahorita se utilizan como los más comunes ahorita en el SIA 2\.  
**Salvador Cervantes Franco:** Sí,  
**Elian Shair Armendariz Puch:** Okay. La parte de régimen, ¿cómo la manejan en el C?  
**Salvador Cervantes Franco:** normalmente es este régimen definitivo, a menos de que quieran la la clave de pedimento, pero régimen es definitivo, normal.  
**Elian Shair Armendariz Puch:** Okay. Pues esta sería la parte de donde dice permisos de regla octava, sería el match entre la fracción y la regla octava. aquí donde se van a a unir. Todavía está este trabajándose. Eh, lo principal ahorita es lo de la a lo que se tendría que mostrar en la parte de mercancías, que es que el que el ejecutivo pueda eh seleccionar la regla octava que el clasificador está eh sugiriéndole. De hecho, va a estar, me habían comentado en las en la reunión anterior que también tiene que ser masivo. Eh, cuando se hace masivo, eh, hay números de parte que tienen reglas octavas en común.  
   
 

### 00:16:46

   
**Salvador Cervantes Franco:** Sí, seguramente.  
**Geovanny Manuel Martinez:** M.  
**Angel Huberto Pulido Burgos:** Quiero quiero suponer que sí, Elian, porque en la reunión del del último día que tuvimos con el  
**Elian Shair Armendariz Puch:** Okay.  
**Angel Huberto Pulido Burgos:** equipo mencion mencionaron que podíamos tener una regla octava para varias reglas octavas para el mismo número de parte. Si no mal recuerdo, si quedó así, si no mal recuerdo. No.  
**Geovanny Manuel Martinez:** Sí, puede tener puede ser desde la dos, o sea, puede haber varios números de parte en una misma regla octava y a su vez también varias reglas octava para un mismo número de parte. M.  
**Elian Shair Armendariz Puch:** Bueno, pues esto es  
**German Castro:** Aquí veo, aquí veo que lo tienes relacionado solamente a un a una fracción,  
**Elian Shair Armendariz Puch:** Ajá.  
**German Castro:** ¿cierto?  
**Elian Shair Armendariz Puch:** Sí. Aquí  
**German Castro:** La regla la regla octava aplica a fracción,  
**Elian Shair Armendariz Puch:** principal.  
**German Castro:** a nivel de fracción o número de parte.  
**Angel Huberto Pulido Burgos:** Sí, creo que ahí creo que era a fracción.  
**German Castro:** Giovanni.  
**Angel Huberto Pulido Burgos:** Germán, es que nos corrijan porque ya me creo que yo me  
   
 

### 00:17:58

   
**Geovanny Manuel Martinez:** A ver,  
**Angel Huberto Pulido Burgos:** equivoqué.  
**Geovanny Manuel Martinez:** mira, si si quieres te enseño cómo quedaría en un pedimento. digo, no no tengo ahorita el deber octava, pero pero si quieres te enseño cómo sería más o menos y a lo mejor ahí  
**German Castro:** Okay.  
**Geovanny Manuel Martinez:** este aí nos podemos entender. de cuenta, este es un pedimento, ¿no? Normal. Lo la regla octava lo único que quiere decir, ¿ven que explicamos la vez pasada? La Secretaría de Economía nos autoriza, nos autoriza este mercancías, por ejemplo, este que es para para el programa de promoción sectorial de la industria automotriz y aquí dice, ¿no?, cuando las empresas cuentan con la autorización a que se refiere la regla octava. Entonces, lo que tenemos aquí es que la Secretaría de Economia nos emite un aviso, un un aviso automático y en ese aviso, nada más que no tengo una regla octava aquí, pero  
**Elian Shair Armendariz Puch:** Vale.  
**Geovanny Manuel Martinez:** Eh, Okay. Entonces, supongamos que tenemos nuestro pedimento, ¿no?, que es este. Y lo que hace la regla octava es que te dice, "Te autorizo a que a que importes con esta fracción. En este caso solo es un número de parte o más bien solo una solo una descripción,  
   
 

### 00:20:23

   
**Geovanny Manuel Martinez:** vamos a decirlo así, ¿no? Un tipo de mercancía. Pero aquí sí, aquí en esta aquí en este mismo permiso te puede poner varias fracciones arancelarias, por ejemplo, este, ¿no? Puedes poner este 76 15 102, la 73 23 9305 y entonces todas las las fracciones que tú necesitas te las va poniendo aquí y aquí en descripción tú puedes poner la descripción y el número de parte o solamente la descripción y entonces este permiso te lo extiende la Secretaría de Economía, este con una vigencia, ¿no? Por ejemplo, en este caso hasta el hasta el 2 de diciembre del 2026\. Entonces, cuando se agote ese permiso, ya sea que te agotes la cantidad, porque te autoriza una cantidad en ese tiempo. Entonces, ya sea que te ya sea que que agotes la cantidad o agotes el el tiempo, hay que renovar. Ahora, también hay muchas veces que dependiendo del proyecto que maneja la empresa puede sacar varias reglas octavas para el mismo tipo de mercancía, para la misma fracción o para el mismo número de parte. Entonces, por eso es que una por eso es que un número de parte puede tener varias reglas octavas y una regla octava puede tener varios números de parte.  
   
 

### 00:21:54

   
**Geovanny Manuel Martinez:** Entonces, aquí lo único que hacemos es que esta fracción pues se cambia. Ya, ya en vez de ser esta ocupamos la que nos la que nos autorizó la la autoridad y aquí en donde va el N3,  
**German Castro:** Ok.  
**Geovanny Manuel Martinez:** aquí es donde se declararía C1 y se declara la regla octava y ya pone la firma de descargo y y la cantidad. Entonces, pones la cantidad para que a nivel central se vaya haciendo el descargo y después pues ya lo tienes. Es más o menos así como como funciona la regla octava aquí aquí en el perio. Gracias.  
**Carlos Alexis Galaviz Rosas:** Okay, una duda, este entonces será necesario subir ese documento en la plataforma para tener o todas las reglas adoptadas entre ese documento o o no se necesita. Adelante, señor Yeah.  
**Jose Antonio Limon:** Hola, buenas tardes a todos. Ay, perdón que me meta, digo, yo estaba antes manejando las reglas octavas con el cliente GKN. Este, este formato nos lo manda el importador. Nosotros lo sub en el SIGAC 2.0 lo subimos a la plataforma en entidades, o sea, en el cliente GKN. ¿Cómo nos lo hacía llegar el cliente? Este es en general para todas las sucursales.  
   
 

### 00:23:25

   
**Jose Antonio Limon:** La entidad GKN número 49 se maneja en Laredo, México, Veracruz y Manzanillo, por decir un ejemplo. Entonces, si nosotros traemos eh ese material por esta aduana, se le aplicaba esa regla octava, esa misma regla octava, ese formato que compartió Giovanni, eh ese ese folio se compartía y y a su vez se también lo manejaba en la  
**German Castro:** No.  
**Jose Antonio Limon:** sucursal México y a su vez también lo manejaba la sucursal Manzanillo y Veracruz. O sea, es por donde manejara el cliente. Si ese cliente entraba por allá por por el puerto de Veracruz y trae esa fracción, le aplicaban la regla. Entonces se iba descontando, descontando, por así decirlo, ese saldos que le que le autoriza la Secretaría de Economía, se le va descontando y el validador lo que hace es de que te dice, "Ya no tiene saldo para esta arregla y ya no nosotros no nos permite avanzar." ¿Qué es lo que hacemos nosotros? Nosotros le hablamos con el importador. Hoy es importador, hoy es cliente. Mira, te mando la pantalla. Eso es lo que me marca el error. Dice que ya no contamos con saldo suficiente o no tenemos saldo suficiente en la regla octava de esta fracción.  
   
 

### 00:24:41

   
**Jose Antonio Limon:** Entonces ellos emiten, ellos entran a Busem al apartado de la Secretaría de Economía y vuelven a hacer un un nuevo poo, un nuevo oficio y el oficio se los manda autorizado a través de la eh electrónico, el que mostró como el que mostró Giovanni y nuevamente nosotros lo volvemos a subir a la entidad como que se vayan quedando los registros de cada una de las reglas octavas de ese de de esa fracción. Si me di a entender, expliqué.  
**German Castro:** O sea, ¿cómo cómo lo mejorarías tú? Este tampoco está tan fácil.  
**Carlos Alexis Galaviz Rosas:** Pero no te escuché, German. No sé si es los demás del equipo lo hicieron.  
**Angel Huberto Pulido Burgos:** No, yo sí lo escuché.  
**German Castro:** Te decía este José Antonio que para la administración esa que llevaban en el SIGAC 3, ¿qué harías para que, o sea, en este listado lo puedes mostrar de nuevo, por favor, Elian? En este listado,  
**Elian Shair Armendariz Puch:** M.  
**German Castro:** la naturaleza es llevar el control de las reglas octavas de los clientes y que el sistema nos pueda avisar de cuándo vencen, este, si le estamos aplicando mal, este, que nos pueda ayudar a todos a poder llevar mejor control. M.  
**Jose Antonio Limon:** Okay, ahí lo veo todo en general.  
   
 

### 00:26:18

   
**Jose Antonio Limon:** Eh, veo el número de permiso, este, el cliente, si lo veo bien, el tipo, la fracción, la vigencia, sí, y puede ser antes, o sea, se puede acabar antes. Estado vencido, está bien. Está en rojo. Eh, para mí sería ahí marcar verde que está activo todavía el permiso. Ah, sí, yo creo que lo veo. Ah, okay. Está bien. Sí, lo veo bien. Este más que nada es para temas de es que lo usamos lo usan más para temas en GKN, por ejemplo. Es que voy a hablar de GKN porque es el que eh más se utiliza por acá por esta adana y se utiliza a través de pedimento consolidado. y no sabemos hasta el cierre del pedimento, hasta la siguiente semana si tiene o no saldo, porque el cliente se lo puede acabar en en menos de una semana, una semana en días o en 15 días o a veces puede durar tres tres semanas o meses, pero sí lo lo veo bien, número permiso la empresa si es lo más básico que nosotros utilizamos el tipo permiso. Sí, ahí podrían especificar si es este más específico, si es regla octava o permiso de economía o aviso automático, no sé, algo sí se me ocurriría.  
   
 

### 00:27:38

   
**Jose Antonio Limon:** La fracción está bien, es meramente el permiso porque la regla octava se la estarías aplicando directamente a las fracciones y si llega un nuevo material, un nuevo con esa fracción que tiene que se requiere regla octava, el ejecutivo lo que va a hacer es decirle al clasificador, oyes, esta fracción necesito que le aplique la regla octava. Y lo que hace el clasificador es entra la fracción y le aplica la regla octava.  
**Elian Shair Armendariz Puch:** Bien, entonces en cuanto a mejora, o algo que les gustaría agregar en esta parte.  
**German Castro:** Okay. Este, yo entiendo que una de las mejoras sería poder darle seguimiento a otro tipo de de permiso, ¿cierto? Eso fue lo que entendí.  
**Jose Antonio Limon:** A ver, no le escuché, ingeniero.  
**Angel Huberto Pulido Burgos:** No,  
**Jose Antonio Limon:** ¿Cómo?  
**Angel Huberto Pulido Burgos:** no te entendí,  
**German Castro:** ¿Me  
**Jose Antonio Limon:** Perdón.  
**Angel Huberto Pulido Burgos:** Germán.  
**Jose Antonio Limon:** Sí.  
**Angel Huberto Pulido Burgos:** Germán, creo que no te entendimos.  
**German Castro:** escuchan mejor ahí?  
**Angel Huberto Pulido Burgos:** Sí. M.  
**German Castro:** Ah, les decía que sí. Entonces, la mejora entendí que es que aparte de llevar eh control de la regla octava, también sería bueno llevar control de otro tipo de permisos o de documentos.  
   
 

### 00:29:32

   
**Salvador Cervantes Franco:** Sí, sí,  
**German Castro:** ¿Y  
**Salvador Cervantes Franco:** porque son son diferentes tipos de documentos los que tienen o vigencia o cierto número de o cierta cantidad de mercancía. Entonces, se van haciendo, les llamamos descargos, se van haciendo descargos por operación.  
**German Castro:** cuáles son esos otros documentos, Salvador?  
**Salvador Cervantes Franco:** Por ejemplo, cartas cupo, este, registros sanitarios. autorizada cierta cantidad de  
**German Castro:** y les gusta tenemos de dos opciones,  
**Salvador Cervantes Franco:** mercancía,  
**German Castro:** o que sean en este mismo listado y cambiar el nombre de este listado en vez de regla octava, que sean permisos y darles el filtro por tipo de permiso,  
**Salvador Cervantes Franco:** permisos.  
**German Castro:** o sea, agregarle el esquema de tipo de permiso o que tuvieran varios listados, un listado específico para cada tipo de permiso. ¿Qué  
**Salvador Cervantes Franco:** No,  
**German Castro:** preferirían?  
**Salvador Cervantes Franco:** yo creo que es más sencillo poner ponerle de título permisos a este y que se ocupe para todos. ¿Cómo ves, John?  
**Geovanny Manuel Martinez:** Sí, yo también, yo también creo que sería mejor y y yo creo que mejor, por ejemplo, ahí en tipo, digo, sí está bien la la descripción, pero igual pudiéramos agregar un un una columna donde viniera la clave del permiso, ¿no? Por ejemplo, permiso C1, N6, NM para las NOMS.  
   
 

### 00:31:05

   
**Geovanny Manuel Martinez:** Y eso lo lo pudamos sacar ahí del del apéndice,  
**German Castro:** Sí, te queda claro de dónde sacarlo, Elian.  
**Elian Shair Armendariz Puch:** Es es de la parte de identificadores.  
**Geovanny Manuel Martinez:** más bien en en la clave de permisos. A ver si Ah, esto creo que se descarga completo. Si lo intentargar. Bueno, deja, déjame ver si si lo puedo descargar.  
**German Castro:** M. No, no.  
**Geovanny Manuel Martinez:** A ver, anexo 22 en la página 115\. M.  
**German Castro:** Okay, perfecto. Entonces, de ahí sacamos el listado de los de los permisos, ¿cierto?  
**Geovanny Manuel Martinez:** Sí, sí, sí, así es. Desde la página C 115 a la 117, pues ahí viene la clave de de de los permisos. De hecho,  
**German Castro:** Okay.  
**Geovanny Manuel Martinez:** viene separado por, por ejemplo, todos los de la Secretaría de Salud, todos los de la Secretaría de Economía. viene ahí su clave y su descripción.  
**German Castro:** Ya lo  
**Elian Shair Armendariz Puch:** Ya, bien entiendo. Entonces, eh ya no va a ser permisos de regtava, se va a quedar como permisos,  
   
 

### 00:33:41

   
**German Castro:** tienes,  
**Elian Shair Armendariz Puch:** se va a hacer como un filtro por tipo, es lo que me comentan. y agregar la nueva columna que me pidieron, que es la de las claves,  
**Salvador Cervantes Franco:** Sí, así como está en el catálogo clave, descripción.  
**Elian Shair Armendariz Puch:** ¿no?  
**Salvador Cervantes Franco:** En la descripción está el tipo de de permisos. Te fijas, dice, por ejemplo, mi primerito, ¿qué te dice? Eh, certificado de cupo adicional. Eso es lo que requiere un certificado de cupo y ese trae cantidades que se tienen que controlar. Ajá. O sea, todo lo que diga cupo, como los dos siguientes también dicen cupo, cupo medida de transición y certificado de cupo controla la cantidad de mercancía que está autorizado para  
**Elian Shair Armendariz Puch:** He.  
**Salvador Cervantes Franco:** importar. En el cuarto, que es el C1, dice, "Es un permiso previo de importación definitiva, temporal o depósito fiscal, etcétera, ¿no? Pero es un permiso previo de importación y así.  
**Elian Shair Armendariz Puch:** Bien. Esta parte de la clave eh la agregan ustedes como clasificadores.  
**Geovanny Manuel Martinez:** Nosotros actualmente solamente la seleccionamos y el ejecutivo ya es el que  
   
 

### 00:35:10

   
**Elian Shair Armendariz Puch:** Ah, okay,  
**Geovanny Manuel Martinez:** pone el número de permiso adecuado. Oh.  
**Elian Shair Armendariz Puch:** okay, ya. Mm.  
**German Castro:** Okay. De este este módulo ya lo pueden acceder Salvador y y Giovanni Elial.  
**Elian Shair Armendariz Puch:** No, aún no está  
**German Castro:** Okay.  
**Elian Shair Armendariz Puch:** subido  
**German Castro:** A partir de cuándo lo van a poder acceder para que empiecen a como a moverle y ver si si tienen observaciones.  
**Elian Shair Armendariz Puch:** a partir Mañana, mañana por la tarde estaría haciendo llegar el link para que puedan probarlo.  
**German Castro:** Okay. Perfecto. Este, aprovechando que los tenemos aquí, Salvador, Giovanni, de el dar de alta o el formulario para dar de alta la parte de clasificación de un número de parte, ustedes ya no les queda duda, ¿tienen tickets, tienen alguna observación que hacernos?  
**Geovanny Manuel Martinez:** dar de alto número de parte.  
**German Castro:** Ajá.  
**Geovanny Manuel Martinez:** La verdad es que no hemos revisado completamente ahorita,  
**German Castro:** Okay.  
**Geovanny Manuel Martinez:** pero empezamos a a revisar para a  
**German Castro:** Este, sí, si nos pueden ayudar,  
**Geovanny Manuel Martinez:** Ok.  
**German Castro:** eh, vamos a tener otra reunión el viernes a la misma hora, es de 4 a 6\. Entonces, si quieren que nos ayuden a hacer pruebas durante estos días, levantar tickets y el viernes, si quieren revisamos esa parte.  
   
 

### 00:37:01

   
**Geovanny Manuel Martinez:** Okay.  
**Salvador Cervantes Franco:** Yo no he ingresado desde desde la primera vez, entonces no sé, no me acuerdo cuál es mi usuario. Si si me lo reenas,  
**German Castro:** Sí, claro. Miguel, ¿nos puedes ayudar con con eso? contigo. En realidad es ser el proceso, si no te acuerdas del password, este es el proceso de resetor de password, pero debe de funcionar con el mismo password que estás usando para el CAC 2\. Debe de funcionarte el Sac 3\.  
**Salvador Cervantes Franco:** Okay. El mismo usuario y paso.  
**German Castro:** Es correcto.  
**Salvador Cervantes Franco:** Okay. Nada más la liga si me la confirman, un papá, ¿no? Mir, gracias  
**German Castro:** Sí, pero si no cualquier cosa este nos avisas y te ayudamos a hacer el reseteo. En la misma página puedes hacer el reseteo de password y te llega un correo a eh con una clave. Ingresas esa cable clave y ya te permite cambiar tu password.  
**Elian Shair Armendariz Puch:** Bien, en cuanto a lo que me había comentado Enrique, no sé si está por ahí, eh, para que me pueda validar.  
**German Castro:** A ver, espérate. Primero vamos a ver si está o creo que sí estaba César, al menos.  
**Enrique Lopez:** Buenas tardes. Sí, ya me acabo  
   
 

### 00:38:22

   
**German Castro:** Ah, qué tal. Una disculpa, Enrique. La verdad es que fue fue todo mi  
**Enrique Lopez:** de  
**Cesar Aguirre:** escuchando.  
**German Castro:** error.  
**Enrique Lopez:** cómo se No,  
**Cesar Aguirre:** Okay. No, es que escuché que si estaba yo ahí.  
**Enrique Lopez:** ya nos andaban buscando, pero ya nos nos estaban buscando, pero aquí estamos igual tú o yo alguien,  
**German Castro:** Perfecto.  
**Enrique Lopez:** pero ahora sí.  
**German Castro:** Si si quieres,  
**Enrique Lopez:** M.  
**German Castro:** adelante  
**Elian Shair Armendariz Puch:** Bien, para ver si me puedes validar lo que me habías comentado de la descripción. Ya ahorita ya aparecen. También aparece lo que es el la fracción de y el Nicoo que me habías comentado que esa parte del nico debe aparecerse. A ver, permítame. De hecho, aquí  
**Enrique Lopez:** Okay. La descripción,  
**Elian Shair Armendariz Puch:** me  
**Enrique Lopez:** una duda, ¿es la descripción de la mercancía o es la descripción de la fracción?  
**Elian Shair Armendariz Puch:** es del pedimento.  
**Enrique Lopez:** Sí, pero a ver, déjame ver, voy a checar.  
**Elian Shair Armendariz Puch:** No.  
**Enrique Lopez:** Vamos a ver.  
   
 

### 00:39:23

   
**Angel Huberto Pulido Burgos:** lo que es la de la  
**Enrique Lopez:** Sí,  
**Angel Huberto Pulido Burgos:** fracción.  
**Enrique Lopez:** por la tipo de la descripción que pusiste, eh, como decir, bolas, rodillos y agujas, yo creo que es de la fracción y a lo mejor la mercancía solo en agujas.  
**Elian Shair Armendariz Puch:** Entonces, la descripción sería  
**Enrique Lopez:** Sí,  
**Elian Shair Armendariz Puch:** otra.  
**Enrique Lopez:** la descripción es la la que va a poner el  
**Elian Shair Armendariz Puch:** Ah, okay,  
**Enrique Lopez:** clasificador.  
**Elian Shair Armendariz Puch:** ya te  
**Angel Huberto Pulido Burgos:** Pues se está poniendo ahí la de la fracción,  
**Elian Shair Armendariz Puch:** entendí.  
**Angel Huberto Pulido Burgos:** ¿no, Elian? ¿O es otra?  
**Elian Shair Armendariz Puch:** Sí, esa es la  
**Enrique Lopez:** Sí,  
**Elian Shair Armendariz Puch:** fracción.  
**Angel Huberto Pulido Burgos:** Entonces,  
**Enrique Lopez:** sí, pero no,  
**Angel Huberto Pulido Burgos:** está bien así como no.  
**Enrique Lopez:** pero, o sea, la descripción que se ve actualmente es la que el clasificador nos pone.  
**Angel Huberto Pulido Burgos:** Ah,  
**Enrique Lopez:** Esa es la descripción de la tarifa. Ya ves que la tarifa te dice,  
**Angel Huberto Pulido Burgos:** okay.  
**Enrique Lopez:** "Ah, bueno, en esta fracción y en este nico pueden entrar cinco cosas distintas o 10,  
   
 

### 00:40:13

   
**Angel Huberto Pulido Burgos:** Ah, okay. Okay.  
**Enrique Lopez:** pero nosotros nosotros solo traemos pasadores." Ahí dice pasadores, chavetas, a mejor traemos pasadores, bolas, rodillos,  
**Angel Huberto Pulido Burgos:** Okay.  
**Enrique Lopez:** solo traemos los rodillos o anillo de decir, creo que esa fracción está como anillo de retención en cigar, creo. Sí, una de esas dos creo que está la de pasadores y chavetas. El número de partes está clasificado como anillo de retención seguro. Sería la descripción. que coloquea el clasificador, la que queremos aquí.  
**Elian Shair Armendariz Puch:** Bien.  
**Angel Huberto Pulido Burgos:** Y si si estamos guardando las dos descripciones.  
**Elian Shair Armendariz Puch:** Sí, las dos.  
**Angel Huberto Pulido Burgos:** A ver.  
**Elian Shair Armendariz Puch:** Sí, sí, sí. De hecho, me había aparecido también la otra, pero tomé tomé esta bien. Eh, la parte del Nico ya está aquí también. De hecho, aquí en las acciones también va a aparecer la descripción, eh, solamente que pues hay que cambiarla la que tú mencionas. Y bien, eh en la configuración de la regla octava, ya aparece la regla octava sugerida por el clasificador, pero obviamente puedes seleccionar otras que tenga esa misma.  
   
 

### 00:41:30

   
**Elian Shair Armendariz Puch:** En este caso solamente tiene una y tiene sin regla octava. Puedes aplicar sin regla octava.  
**Enrique Lopez:** Muy bien. Ahí ahí a la liga. Puede ligarle a una regla octava que está aunque dice que está vencido, ¿no?  
**Elian Shair Armendariz Puch:** Sí,  
**Enrique Lopez:** El permiso.  
**Elian Shair Armendariz Puch:** sí, así es. dice que está vencido.  
**Enrique Lopez:** Okay. Okay. Si estaban sido, pues yo creo que no le debería de aparecer para que digo que solo le aparezcan las  
**Elian Shair Armendariz Puch:** Okay.  
**Enrique Lopez:** vigentes para que él solo eh seleccione la que esté lista para usarse. M.  
**Elian Shair Armendariz Puch:** Bien. Eh, pues todo lo demás de los identificadores es la misma función que te había comentado anteriormente en la en la anterior sesión.  
**Angel Huberto Pulido Burgos:** Una una duda ahí, Enrique. Eh, cuando tenemos reglas octavas vencidas, ¿no no ocuparías ver cuáles están vencidas por si necesitas avisarle al cliente para que te la actualice o algo? O no, o no lo ves necesario.  
**Perla Lopez:** es que el folio no se renueva, o sea, no sigue siendo el mismo folio,  
   
 

### 00:42:51

   
**Elian Shair Armendariz Puch:** Bueno,  
**Perla Lopez:** sino que se hace otro trámite cuando se acaban los los saldos.  
**Angel Huberto Pulido Burgos:** Ah, sí, pero para para que te pase la para que sepas que te tiene que pasar  
**Perla Lopez:** Este ah,  
**Angel Huberto Pulido Burgos:** una que tenga vigencia,  
**Perla Lopez:** okay. Es que ellos ellos llevan el control de sus saldos.  
**Angel Huberto Pulido Burgos:** pues. Ah,  
**Perla Lopez:** Las reglas octavas tienen límites de, no sé,  
**Angel Huberto Pulido Burgos:** okay.  
**Perla Lopez:** $100,000 y 100,000 piezas. Entonces, cuando se van utilizando es porque el cliente iba llevando los el conteo de sus saldos.  
**Angel Huberto Pulido Burgos:** Ah, okay. Ah, pues entonces no  
**Enrique Lopez:** Está está bien. Digo, esta regla octava, ¿dónde la diste, Delta? Este, Elian, ¿cómo la registras?  
**Elian Shair Armendariz Puch:** es en la parte de que aquí acabo de mostrar el clasificador. El clasificador la da de alta y aquí te aparecen como sugerencia.  
**Enrique Lopez:** Okay. Preclasificó la de alta.  
**Angel Huberto Pulido Burgos:** Creo que la podría dar de alta el ejecutivo,  
   
 

### 00:43:56

   
**Elian Shair Armendariz Puch:** Sí.  
**Enrique Lopez:** Sí. Ándale. Actualmente, digo, actualmente eh en el módulo de de entidades, o sea,  
**Angel Huberto Pulido Burgos:** No.  
**Enrique Lopez:** que en este caso compañías, aparece el menú de permisos y ya ahí se guarda el historial de los vencidos. en eh no sé vigentes, vencidos y no me acuerdo qué más. Este, y ahí podemos guardar, o sea, ahí se queda ese registro de los que están ya vencidos, pues ahí se va quedando el registro como lo que mencionabas, pero lo habilitan nuevo, suben el nuevo permiso y colocan ese permiso tiene características, ¿no?, de que ah, bueno, va a aplicar una fracción eh a ciertos países, ah, este, cierto, cantidad de valor dólares o cantidad tarifa, un ejemplo, eh el número de permiso te, o sea, tienes que registrarlo en el en el sistema. para que quede habilitado, para que si el ejecutivo pueda ponerlos y esa misma información pueda estar ahí. Y mira, hay algo que no hace el sig actual, que a lo mejor aquí podría ser buena opción. No sé qué piensen los demás, que te aparezcan los saldos,  
**Elian Shair Armendariz Puch:** Ú.  
**Enrique Lopez:** o sea, que ahí en un pequeño punto ahí te diga, no sé, creo que la regla octava es por cantidad de mercancía y por valor, ¿no?  
   
 

### 00:45:14

   
**Enrique Lopez:** Que te aparezcan los saldos de que actualmente tiene tanto, ¿no? O sea, digo, podría ser algo que ahorita no estamos viendo y que podría ayudarnos a saber eh cómo vamos en el tema de saldos y yo me refiero en cualquier permiso, ¿no? este saber qué es lo que ya importamos que se vaya actualizando cuando un pedimiento esté parado, eh, o que esté modulado o que tenga evento de de desodonamiento libre, este, para que se vaya restando al sistema y aquí aparezca un leve punto de saldos, ¿no? para que en algún momento que se nos vaya a vencer un permiso, ya sea por vigencia de tiempo, ya sea por valor o ya sea por cantidad, que el ejecutivo pueda visualizarlo, ¿no?  
**Elian Shair Armendariz Puch:** Ok.  
**Enrique Lopez:** Así algo chiquito,  
**Elian Shair Armendariz Puch:** segundos.  
**Enrique Lopez:** o sea, no que no quede o que arrastre el el mouse y que le aparezca vigencia, no sé, algo no tan grande para que pudiera nada más ser una notita. Este, pero creo que se podría hacer el y así en general con todos los permisos que pueda tener un cofre, eh, se agarra para sanitario, se den cualquier otro permiso, es el mismo modalidad, lo al telejecutivo, coloca el número de permiso, eh eh lo que viene siendo la la vigencia, cantidad, valores, todo lo que especifique el permiso y a su vez el sistema hace de que cuando tú lo captures te lea contra el registro y pueda decirte a lo mejor si está vigente o no está vigente y que no te deje validar si no está vigente o o que te marque un error o que te diga,  
   
 

### 00:47:05

   
**Enrique Lopez:** "Oye, tu permiso se está venciendo." O sea, sería también un punto eh útil para pues para que el ejecutivo no vaya a hacer algo que está por vencerse o ya se venció o algo. teoría, si venció, pues a lo mejor validador le debería de marcar error, pero pues a lo  
**Elian Shair Armendariz Puch:** Bien. E pues esto es lo que se está está avanzando en esta parte. Eh, lo primordial pues es sacarlo de la regla octava ya para que puedan programar bien todo este módulo del tab de mercancías y puedan hacer tickets si algo está mal. Eso es lo que se lleva hasta el momento. No sé si tengan dudas o alguna sugerencia mejora.  
**German Castro:** Okay.  
**Enrique Lopez:** Sí, adelante, adelante. No, nada más el tema de lo que hablamos ahorita, el tema de la el alta, que se de en modo entidades y llevar un registro de las cantidades, valores y vigencia para que el ejecutivo eh digo para el ejecutivo pueda ver pueda saber en general, o sea, ahorita hablamos de la regla octava, pero yo creo que de cualquier permiso eh es bueno que aparezca alguna alerta o alguna notificación de que se le está venciendo algo. que el ejecutivo pueda solicitar esa actualización, o sea, un mes antes, ¿no?  
   
 

### 00:48:45

   
**Perla Lopez:** También en esa parte donde se vaya a configurar la regla octava se podría ver la opción para subirla, o sea, para que se digitalice, porque actualmente pues no no se digitaliza, pero que se pueda digitalizar y ya para cualquier cosa de que se tenga ahí la regla.  
**Elian Shair Armendariz Puch:** Una duda, ¿cómo funciona lo de la digitalización? Yeah.  
**Perla Lopez:** Sí. O sea, solamente subir el archivo para que se quede guardado como un historial de que sí se nos proporcionó la regla.  
**Angel Huberto Pulido Burgos:** Eso, eh, pero te refieres a digitalización. Es que utilizan digitalización para y documents y para subir archivos. Yo creo que por eso le causa confusión.  
**Perla Lopez:** Sí. Bueno, sería para subir el archivo, el puro archivo.  
**Angel Huberto Pulido Burgos:** Ajá. Para guardarlo en el sistema es lo que se refieren ellos.  
**Perla Lopez:** Mhm.  
**Elian Shair Armendariz Puch:** Ah, okay.  
**Angel Huberto Pulido Burgos:** Ya es que te mencioné que habían algunos que tenían las reglas octavas en su computadora. Ah,  
**Elian Shair Armendariz Puch:** Sí.  
**Angel Huberto Pulido Burgos:** pues en lugar de tenerlas en la compu, pues que lo tengan en el sistema.  
   
 

### 00:49:58

   
**Angel Huberto Pulido Burgos:** Creo que en Ajá. En Sigo si se puede según yo,  
**Elian Shair Armendariz Puch:** ¿Ya  
**Angel Huberto Pulido Burgos:** pero como que no está de fácil acceso. Algo así me habían mencionado en reuniones anteriores.  
**Elian Shair Armendariz Puch:** tienen algún formato?  
**Angel Huberto Pulido Burgos:** Creo que es PDF en el que les llega,  
**Carlos Alexis Galaviz Rosas:** Creo que creo que es el que mostró ahorita que eh no sé si  
**Angel Huberto Pulido Burgos:** ¿no?  
**Carlos Alexis Galaviz Rosas:** fue Salvador.  
**Perla Lopez:** Si nos llega un PDF, si el cliente nos comparte el PDF de la regla. Amén.  
**Angel Huberto Pulido Burgos:** Mm. Creo que ahí donde pone donde estás configurando la el vencimiento y eso, ahí podrías adjuntar el archivo. No.  
**Elian Shair Armendariz Puch:** Okay. Eh, nada más si me podrían pasar el archivo para ver cómo es.  
**Carlos Alexis Galaviz Rosas:** Pues va a ser un PDF a lo que tengo entendido.  
**Elian Shair Armendariz Puch:** Ok.  
**Carlos Alexis Galaviz Rosas:** puede subir cualquier PDF, al menos que lo vanayan a pasar por se sier algo.  
**Angel Huberto Pulido Burgos:** Deja que puedan subir PDFs por el momento.  
   
 

### 00:51:10

   
**Angel Huberto Pulido Burgos:** Tam.  
**Enrique Lopez:** Sí, nada más digo, ahorita puse esto, ahorita lo quito. Esto es lo que vemos ahorita Elian en el Nigac. Vemos vigentes, vencidos. Aquí ya se guarda historial, pero digo, ¿qué nos permite eh dar nuevo? Okay, damos nuevo, nos permite colocar el número de permiso, colocar el número y ese solo aplica para esta entidad o para esta compañía. te pide eh númeroción que va a aplicar ciertos características, país, régimen, datos, pero aquí no te permite subirla, o sea, no, o sea, el documento como mencionaba Ángel lo subimos en cada operación porque el ejecutivo lo tiene que tener en su compo, ¿no? Pero pues si se puede registrar aquí estaría bien para que cualquier cosa o algún otro compañero que tenga la necesidad de manejar ese cliente y en lugar de pedirle al otro compañero la regla octava, pues a lo mejor ahí lo toman y así cada permiso se pueden subir quizás para que quede el registro de que el permiso tal ahí está y ahí están los datos por si alguien quiere eh revisar algún tema y ya queda el registro.  
**Elian Shair Armendariz Puch:** Okay. Bien. ¿Alguna otro mejor? Comentario. Les voy a compartir pantalla si gustan.  
   
 

### 00:52:39

   
**Enrique Lopez:** Sí, está bien. Sí. Uh.  
**Elian Shair Armendariz Puch:** Bien, entonces esto sería más enfocado a lo que es la la regla, que es lo que se va trabajando y espero mandarle, no sé. Ah, gracias. No sé quién este me puede validar esto. Creo que sería Enrique, no sé si estoy mal. me puedan confirmar para que una vez ya lo cheque eh bueno, esté ya los cambios hechos eh pueda pasarle el link para que pueda hacer pruebas y y sacar  
**Enrique Lopez:** Okay. Sí,  
**Elian Shair Armendariz Puch:** tickets.  
**Enrique Lopez:** pues digo, si nos da si lo que mencionas ahorita pasar el link, si ahorita entro a al sistema todavía no está para para moverle o checarle o hacerle. Okay. Este y sí digo, "Okay, mira, ahorita pues se ve, o sea, el tema de la descripción lo vas a actualizar. Este, el tema de que, a ver, déjame ver. Sí, necesitaría moverle para poder a ver cuál qué otro detalle se puede se puede presentar y ya poderte darte otro. M.  
**Elian Shair Armendariz Puch:** Bien. No sé si quieres comentar algo, hermano.  
   
 

### 00:54:20

   
**German Castro:** Perdón. Sí, estaba en mute. Okay. Entonces, si ya terminamos esta parte del de de los permisos y de clasificación, entonces vamos a retomar donde nos habíamos quedado, Ángel, o si no empezar una operación desde cero y de nuevo empezar a revisar todo lo demás. Este, ¿en qué parte nos habíamos quedado con con Enrique Ángel?  
**Angel Huberto Pulido Burgos:** A ver, déjame, me acuerdo. No, no, no. En la reunión pasada dejamos un tema pendiente. Déjame reviso los apuntos. No, dejamos un tema.  
**Enrique Lopez:** No quedamos en movimiento de salida.  
**Angel Huberto Pulido Burgos:** Sí,  
**Enrique Lopez:** Que movimiento de salida ahí nos quedamos ahí.  
**Angel Huberto Pulido Burgos:** va.  
**Enrique Lopez:** Eh, digo, creo que se capturaron ahorita varias, hay varias referencias capturadas. Teníamos duda con respecto al proceso de eh subvisión y movimientos de salida. Este ahí eso estaba pendiente.  
**Angel Huberto Pulido Burgos:** Ah, estábamos valeando los campos. Es  
**Enrique Lopez:** Sí,  
**Angel Huberto Pulido Burgos:** verdad.  
**Enrique Lopez:** Andre, que empezamos a checar y que oye, esto no creo que debería estar aquí. Este tampoco, este sí, este no.  
   
 

### 00:55:35

   
**Angel Huberto Pulido Burgos:** Sí.  
**Enrique Lopez:** Sí.  
**Angel Huberto Pulido Burgos:** Okay. Okay. A ver, déjame compartir la pantalla.  
**Elian Shair Armendariz Puch:** De momento para mí no. nada más. Eh, gracias por checarlo de clasificación y no sé si algún compañero tenga algo que checar con Salvador.  
**German Castro:** No, muchas gracias,  
**Perla Lopez:** Oh.  
**German Castro:** Salvador.  
**Angel Huberto Pulido Burgos:** No, todo bien, gracias.  
**Carlos Alexis Galaviz Rosas:** Eh, Germán, yo creo que sí valdría la pena que aprovechen de una vez y le den un vistazo a los a lo que tiene que ver con temas de módulo de clasificación. Igual si hay algún comentario, algún faltante, pues anexar los tickets.  
**German Castro:** Lo que comentaron Giovanni y Salvador es que van a checarlo este en estos días y vamos a retomar el tema justamente de esos comentarios el  
**Carlos Alexis Galaviz Rosas:** Ah, perfecto.  
**German Castro:** viernes porque no habían este no  
**Carlos Alexis Galaviz Rosas:** El viernes. Okay.  
**German Castro:** habían tenido la oportunidad de revisar esos módulos.  
**Carlos Alexis Galaviz Rosas:** Va, gracias. Yeah.  
**German Castro:** Gracias. M.  
   
 

### 00:58:43

   
**Angel Huberto Pulido Burgos:** Eh, bueno, aquí tenemos eh, a ver, perdón, ya se levantaron los tickets para lo de clasificación porque o  
**German Castro:** No,  
**Angel Huberto Pulido Burgos:** no.  
**German Castro:** aún no se han levantado los tickets porque no habían entrado Giovanni y Salvador.  
**Angel Huberto Pulido Burgos:** Ah, okay, ya.  
**German Castro:** Entonces, van a ser las pruebas.  
**Angel Huberto Pulido Burgos:** van a se va a retomar,  
**German Castro:** pruebas. Ajá.  
**Angel Huberto Pulido Burgos:** dijiste.  
**German Castro:** van a ser las pruebas en estos días y retomamos el tema el viernes con la revisión que han hecho Giovanni Salvador.  
**Angel Huberto Pulido Burgos:** Ah, okay, perfecto. Bueno, eh vamos a Sí, ya ya recordé la reunión pasada, eh nos vamos a ir step by step para checar cada apartado y retomar los campos para que se levanten los tickets. exactamente de este apartado porque necesito revalidar los los campos en el ticket otra vez para que no queden dudas. A ver, eh empezamos por este apartado de facturas. Aquí tenemos la tarea, creo ya, de poder seleccionar eh movimiento, o sea, poder crear un movimiento de salida a partir de un movimiento de entrada y ahí adjuntamos varias valiaciones, si no mal recuerdo, eh hay un ticket por ahí que indica esas valiaciones de que una una, por ejemplo, es que eh extraiga los datos que ya vienen de de la del movimiento ento de entrada y ya no se tengan que que poner en en el movimiento  
   
 

### 01:00:27

   
**Angel Huberto Pulido Burgos:** de de salida a menos que sea eh alguna subdivisión. ¿Está está correcto, verdad?  
**Enrique Lopez:** Es correcto.  
**Angel Huberto Pulido Burgos:** Sí. Ajá. Sí. Y bueno, entonces nos vamos a ir aquí al apartado de transporte. Eh, quedamos en que íbamos a tener ETD. Este ya se está eh este ya se migra del del ETD que se puso al crear la referencia. se pone por default el valor que que se puso en la referencia, pero ya te permite modificarlo en caso de que lo necesites. Eh, sobre el el la hora de salida eh real, este, tengo entendido que no lo necesitan los Bueno, se va a dejar aquí, pero no lo necesitan modificar los ejecutivos. Este se va a modificar automáticamente cuando en almacén, Carlos, creo que no no estuviste en esa reunión, se va eh digitalizar eh cuando le den salida en almacén, pues ya se va a poner la hora de salida real y ya el pues lo va a poder ver el ejecutivo aquí el en la fecha que se digitalizó al salir la mercancía. eh este de peso del embarque eh y la unidad del peso lo vamos a estar eh trayendo desde el movimiento de entrada.  
   
 

### 01:02:12

   
**Angel Huberto Pulido Burgos:** En caso de tener eh modificaciones en el movimiento como una subdivisión, el peso del embarque pues se va se va a tener que modificar, pero este de aquí no lo pondría el ejecutivo, sino que lo veríamos desde la configuración. A ver, ahí me surgió una duda. Si no mal recuerdo, dijimos que almacén pesa eh cuando hacen subdivisiones, pero vamos a hacer la subdivisión. Ya, ya, ya, ya me acordé. Vamos a hacer la subdivisión y esa subdivisión va a tener su peso. Entonces, aquí vamos a ver el peso total del embarque que vamos a transportar de las divisiones o o movimientos completos que pongamos eh para este movimiento de salida. Creo que ahí como que estuvo medio enredosa la explicación, pero sí estoy bien con eso del del peso, ¿verdad? si no mal recuerdo. Okay.  
**Enrique Lopez:** Creo que digo, creo que no está aquí Yolanda o Limón, pero este del Pero este del tema de este eh dato de peso de y bultos, eh digo, trae lo original. Okay, pues si hay subdivisión no está mal que nos permita verlo. En teoría, Bodega debería de colocarlo o podría colocarlo, pero también que nos dé la libertad de actualizarlo en caso de de que pues se hayan equivocado, ¿no?  
   
 

### 01:03:57

   
**Enrique Lopez:** O sea, pues digo, que pongan un dato que no es y oye, actualízalo, pues poder actualizarlo, ¿no? este que lo traiga, pero que pues podamos  
**Angel Huberto Pulido Burgos:** Okay. Y y ahí, por ejemplo,  
**Enrique Lopez:** editarlo.  
**Angel Huberto Pulido Burgos:** ¿en qué caso lo editarías tú si el peso del embarque pues no lo conoces o así?  
**Enrique Lopez:** Puede ser, ¿no? ¿Te acuerdas que habláamos en otro punto en el tema documental que si hay manera de nosotros conocer los pesos? O sea, si viene un BL y me dice eh que traemos eh 5 Vos y el peso  
**Angel Huberto Pulido Burgos:** Ah, okay.  
**Enrique Lopez:** neto es esto y el peso bruto es esto. Hay veces que te dice el peso de eh ahí puedes sacar todo el peso de catarin. Dices, ah, bueno, estoy subdividiendo eh 5000 libras para acá y 2000 libras para acá.  
**Angel Huberto Pulido Burgos:** Aha.  
**Enrique Lopez:** Okay. El peso por tarima son 20 libras. Ah, bueno, pues ya sacas, o sea, tú puedes hacer el cálculo y ponerlo los pesos conforme a documentos.  
**Angel Huberto Pulido Burgos:** Ya, ya, sí, ya, ya me acordé cuando mencionaste eso.  
   
 

### 01:04:54

   
**Angel Huberto Pulido Burgos:** Está bien. Entonces, pues lo igual va a estar ahí para que lo puedan modificar entonces el campo  
**Enrique Lopez:** Sí, ándale. O sea, te lo traiga para no volver a escribir y si hay su división y bodega lo coloca,  
**Angel Huberto Pulido Burgos:** este.  
**Enrique Lopez:** pues adelante. Y si no, como quiera el ejecutivo pudiera actualizarlo en caso de algún detalle. Yeah.  
**Angel Huberto Pulido Burgos:** Mm. Okay. Y sobre el campo de tipo de material, ¿qué podríamos decir sobre este campo?  
**Enrique Lopez:** Ese era el que decíamos que solo era de Nestle, que solo Nestle es el que lo va a manipular.  
**Angel Huberto Pulido Burgos:** Okay. Pero se podría utilizar para otros clientes, ¿no? En un futuro. O sea,  
**Enrique Lopez:** Podríamos podría  
**Angel Huberto Pulido Burgos:** puede que es que es que fíjate que me me he estado topando eh ahorita que he estado hablando con varios de los muchachos en estos días de que les pregunto cosas ahí de dudas que me surgen sobre los tickets que tengo ahí. Y me he dado cuenta que cuando empezamos con la con el proceso de de la implementación del programa, eh mencion, o sea, entrábamos a una reunión con Nestleé y nos decían, "Ah, este campo es para Nestleé." Pero resulta que hay campos que también  
   
 

### 01:06:16

   
**Angel Huberto Pulido Burgos:** utilizan, ponle que dos clientes, pues entonces en en una reunión, no sé si fue hace como un mes, se mencionó que ese tipo de campos los íbamos a dejar para la manipulación de cualquier cliente, porque puede que otro cliente nos pueda pedir más adelante, no sé, el tipo de material, pues entonces no sé ¿Qué opinas sobre eso tú o qué opina el equipo sobre ese tema?  
**Enrique Lopez:** que no está mal.  
**German Castro:** Perdón,  
**Enrique Lopez:** A ver si adelante  
**German Castro:** no. Adelante,  
**Enrique Lopez:** adelante.  
**German Castro:** Enrique.  
**Enrique Lopez:** Digo, no está mal. Esto digo, eh, ahorita estaba pensando en un hay una este característica que creo que ahorita resortes está pidiendo, ¿no? Planta no sé qué y planta otra cosa. Pudiera aquí poner una característica para saber cuáles son de una planta, otra, o sea, pudiera servirnos. Ahorita los conceptos que están ahí son únicamente de Nestle, o sea, son tipos de este, no cómo le llaman ellos,  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** pero son el tipo de producto que manejan de tal este línea o tal esto, o sea, materia prima, sólidos y lácteos, o sea, o sea, digo, es puro como categoría de de ellos. Eh, no hay otro que lo tenga, pero no hay.  
   
 

### 01:07:57

   
**Enrique Lopez:** Podría ser una opción si en algún momento eh un cliente me decir,  
**Angel Huberto Pulido Burgos:** Sí.  
**Enrique Lopez:** "Ah, bueno, yo quiero que me pongas todo lo de eh materia prima o todo. Tengo dos plantas y quiero que me digas la de una planta y otra no.  
**Angel Huberto Pulido Burgos:** Okay. Hay una una duda, Germán, o quien me pueda responder sobre y para ti también, Carlos, porque no recuerdo de si tenemos esta configuración en la base de  
**Carlos Alexis Galaviz Rosas:** Se se puede,  
**Angel Huberto Pulido Burgos:** datos.  
**Carlos Alexis Galaviz Rosas:** yo te lo respondo antes de que lo preguntes. Sí, se puede configurar.  
**Angel Huberto Pulido Burgos:** Ah, okay. A nivel  
**Carlos Alexis Galaviz Rosas:** Sí,  
**Angel Huberto Pulido Burgos:** cliente  
**Carlos Alexis Galaviz Rosas:** es la de configuraciones que te ve la vez pasada para los días del almacenaje. Ahí podemos agargar cualquier  
**Angel Huberto Pulido Burgos:** y eso eso lo podría configurar cualquier persona,  
**Carlos Alexis Galaviz Rosas:** cosa.  
**Angel Huberto Pulido Burgos:** ejecutivo, quien de de alta el cliente.  
**Carlos Alexis Galaviz Rosas:** Principalmente sería el equipo de compliance, pero sí se podría configurar.  
**Angel Huberto Pulido Burgos:** Okay. Entonces, podríamos hacerle así, Enrique.  
   
 

### 01:08:52

   
**Carlos Alexis Galaviz Rosas:** Yeah.  
**Angel Huberto Pulido Burgos:** A ver. Y por ejemplo, bueno, si quieres eh voy a dejar en este ticket un comentario para preguntarte cómo podría yo hacer eso y hacer las pruebas para que este tipo de material me salga solamente para Nestle y dar y poder dar de alta las las opciones o el tipo de campo.  
**Carlos Alexis Galaviz Rosas:** Va. M.  
**Angel Huberto Pulido Burgos:** Sale.  
**Enrique Lopez:** Muy  
**Angel Huberto Pulido Burgos:** Okay. Eh, sí, sí.  
**Enrique Lopez:** bien.  
**Angel Huberto Pulido Burgos:** entendiste el lo que mencionó Carlos tú,  
**Enrique Lopez:** Sí, sí, sí,  
**Angel Huberto Pulido Burgos:** Enrique.  
**Enrique Lopez:** entiendo que o sea como eh por cada entidad pudíamos configurar qué cosa queramos ver y creo que va a estar resguardado por compliance,  
**Angel Huberto Pulido Burgos:** Ajá.  
**Enrique Lopez:** me parece. Yeah.  
**Carlos Alexis Galaviz Rosas:** Sí, ellos serían los ideales para idóneos para dar de alta esto, ¿no?  
**Angel Huberto Pulido Burgos:** Okay.  
**Carlos Alexis Galaviz Rosas:** Pero sí ahí podemos agregarlo para que aparezcan los que deben de aparecer. En dado caso que apliques, pues si no que no aparezca nada. Yeah.  
   
 

### 01:09:58

   
**Enrique Lopez:** Me parece. Yeah.  
**Angel Huberto Pulido Burgos:** Esta duda es para ti, Carlos, porque la vez pasada no me la pudieron no recuerdo si me la pudieron responder, la verdad, pero la verificación la podríamos tener, según yo, a nivel eh salida, ¿no? también, si no mal recuerdo,  
**Carlos Alexis Galaviz Rosas:** Sí, pero ahí ahí ya se hace contra buulto,  
**Angel Huberto Pulido Burgos:** porque sí,  
**Carlos Alexis Galaviz Rosas:** pero no sé si lo a nivel pie,  
**Angel Huberto Pulido Burgos:** por ejemplo,  
**Carlos Alexis Galaviz Rosas:** eh.  
**Angel Huberto Pulido Burgos:** si yo podría ponerle bulto o pieza, pues para la salida, pero creo que la reunión pasada comentaron que para una salida no se hace verificación.  
**Carlos Alexis Galaviz Rosas:** No, no hacen como tal verificación, pero bueno, al final de tienen que verificar las etiquetas, pero ya salgo dentro del procedimiento. Es no necesario para la salida.  
**Angel Huberto Pulido Burgos:** Entonces, este campo lo puedo eliminar a nivel movimiento de salida, ¿no? La verificación.  
**Carlos Alexis Galaviz Rosas:** Sí, así es.  
**Angel Huberto Pulido Burgos:** Okay, después tenemos el campo.  
**German Castro:** No, pero eh creo que ese campo lo había pedido eh  
**Angel Huberto Pulido Burgos:** Adelante.  
**German Castro:** Almacén justamente para que les dijeran cuándo sí aplicaba verificación extra y que no tuvieran que estar enviando correo.  
   
 

### 01:11:12

   
**Carlos Alexis Galaviz Rosas:** Sí,  
**German Castro:** No sé si alguien de almacén nos pueda confirmar.  
**Carlos Alexis Galaviz Rosas:** tiene razón.  
**Enrique Lopez:** No, pero es en la entrada.  
**Angel Huberto Pulido Burgos:** Pero es que eso Ajá.  
**Enrique Lopez:** Es es la entrada de referencia en el movimiento de entrada.  
**Angel Huberto Pulido Burgos:** Eso era para la entrada.  
**German Castro:** Ah,  
**Enrique Lopez:** Ya la salí de  
**German Castro:** solo entrada en la salida.  
**Carlos Alexis Galaviz Rosas:** Sí, en salida no.  
**Angel Huberto Pulido Burgos:** Sí,  
**German Castro:** Ah, okay.  
**Angel Huberto Pulido Burgos:** de hecho en la reunión pasada sí estaba Yolanda y fue la que mencionó también que en la salida no se  
**German Castro:** Adelante.  
**Angel Huberto Pulido Burgos:** hacía.  
**German Castro:** Ah, okay. Perdón. Adelante.  
**Angel Huberto Pulido Burgos:** Sí. Okay. Entonces, el campo de tipo de flete eh creo que también lo vamos a quitar, ¿no?, para esta parte de del movimiento de salida porque no manejamos eh pagado por pagar para los las salidas, ¿o sí?  
**Enrique Lopez:** No, no, tampoco. No es un registro que tengamos o que manejemos conflictos. M.  
**Angel Huberto Pulido Burgos:** Okay. Después tenemos este de identificación del transporte, país del vehículo y el CAT. estos campos quedamos, si no mal recuerdo, en el ticket, que estos campos se van a dejar, pero se va a añadir aquí eh yo creo que  
   
 

### 01:12:31

   
**Perla Lopez:** El de es cac.  
**Angel Huberto Pulido Burgos:** es el de ¿Qué? Ah, sí, el de esca. No me acuerdo cómo se escribe, pero  
**Enrique Lopez:** Sí,  
**Angel Huberto Pulido Burgos:** sí.  
**Enrique Lopez:** acuérdate que también habíamos dicho que en la abajo viene como que el nombre del transporte, ¿no? O algo así.  
**Angel Huberto Pulido Burgos:** Sí. Ah, sí,  
**Enrique Lopez:** Sí,  
**Angel Huberto Pulido Burgos:** cierto.  
**Enrique Lopez:** que decíamos de que ah, bueno,  
**Angel Huberto Pulido Burgos:** Aquí.  
**Enrique Lopez:** que te que cuando tú coloques el dato de la línea te pudiera traer datos de K,  
**Angel Huberto Pulido Burgos:** Ajá.  
**Enrique Lopez:** 10K, este, ya sea que te los traiga y que te los coloque y que te deje editarlos, pero pudiera tener el la entidad del transporte, algún registro para guardar historiar.  
**Angel Huberto Pulido Burgos:** Ah, okay. Entonces, ese sería sobre historial, ¿no? Sobre configuración del del proveedor o del  
**Enrique Lopez:** Sí, o sea,  
**Angel Huberto Pulido Burgos:** transportista.  
**Enrique Lopez:** o sea, eh cuando tú das alta la la entidad del transportista pudieras guardar en sus eh campos, aparte de nombre, eh RFC, todo ese dato, contactos, algún teléfono tal vez, o sea, eh además de agregar todo eso, pudieras agregar el C, el CAT y y el scack para que cuando tú selecciones el transportista te jale sus datos, pero o sea, pudiera y y si en algún momento quieres actualizarlo, pues que te deje actualizarlos también, o sea, que te los jale,  
   
 

### 01:13:51

   
**Enrique Lopez:** pero que también pudieras editar si se  
**Angel Huberto Pulido Burgos:** Okay. Entonces, el ticket de ese sería eh a nivel no se hace actualmente,  
**Enrique Lopez:** pueden  
**Angel Huberto Pulido Burgos:** ¿verdad, Fer? ¿O no? Si está F,  
**Enrique Lopez:** en cuál en el C 2.0  
**Angel Huberto Pulido Burgos:** ¿no? En el Sigac 3, en el Sac 3, en la configuración de de la compañía,  
**Fernando Angel Lopez Soto:** No creo que no.  
**Angel Huberto Pulido Burgos:** pero eso solo es para proveedores, ¿verdad? para los que tengan el rol de proveedor. Tú, Enrique, el CAD y el SCAC de  
**Enrique Lopez:** Sí, el error de transportista cáncer,  
**Angel Huberto Pulido Burgos:** transportista.  
**Enrique Lopez:** no sé cómo llamar transportista. St.  
**Angel Huberto Pulido Burgos:** Ajá. Entonces, el ticket sería añadir esos dos campos a nivel eh compañía para el el transportista. Y este, ya me acordé de otro. Aquí el transportista lo vamos a mover el campo que esté antes de datos de cruce, por eso que mencionaste, ¿cierto?  
**Enrique Lopez:** Y para que ya jale los datos y en lugar de colocarlos,  
   
 

### 01:15:01

   
**Angel Huberto Pulido Burgos:** Ajá,  
**Enrique Lopez:** pongo el transportista y me va a jalar este la información.  
**Angel Huberto Pulido Burgos:** sí, claro. Okay.  
**Enrique Lopez:** No.  
**Angel Huberto Pulido Burgos:** Después tenemos pues ya tenemos el de transport, el de transporte, modo de transporte y el transportista, pues este lo vamos a dejar y este se va a mover hacia arriba. el de el de Carrier. Después tenemos este de DBL, tracking, país de origen, país de destino, ubicación origen, ubicación destino. De estos campos de aquí los ves necesarios para un movimiento de salida. Enrique,  
**Enrique Lopez:** No,  
**Angel Huberto Pulido Burgos:** no.  
**Enrique Lopez:** no, no manejamos ahorita. Ubicación de origen, ubicación de estim  
**Angel Huberto Pulido Burgos:** Okay. Bueno, entonces el ticket sería que el apartado en movimiento de salida, quitar el apartado de documentos y ubicaciones en el tab de transporte. Okay. Después tenemos las guías. Esto sí lo ocupamos, ¿no?  
**Enrique Lopez:** Solo en ferrocarril para la salida de de terrestre importación normal. No,  
**Angel Huberto Pulido Burgos:** Okay. Pues para un movimiento de salida terrestre, eh,  
   
 

### 01:16:49

   
**Enrique Lopez:** no, declaramos  
**Angel Huberto Pulido Burgos:** vamos a no vamos a quitar el tab de guías para el de entrada.  
**Enrique Lopez:** bien.  
**Angel Huberto Pulido Burgos:** Sí, tú para la entrada. Sí.  
**Enrique Lopez:** Sí, entradas.  
**Angel Huberto Pulido Burgos:** Ah, okay. Después tenemos el de bultos. ¿Qué opina aquí,  
**Enrique Lopez:** No, pero ese de Bultos quamos que lo iba a tomar también de de de cómo  
**Angel Huberto Pulido Burgos:** señor?  
**Enrique Lopez:** se de lo que se ha en la entrada,  
**Angel Huberto Pulido Burgos:** Ajá.  
**Enrique Lopez:** ¿no?  
**Angel Huberto Pulido Burgos:** del movimiento de entrada o o subdivisión que se haya hecho.  
**Enrique Lopez:** Igual digo, pudiera ser quisieras como el tema de,  
**Angel Huberto Pulido Burgos:** Okay.  
**Enrique Lopez:** si te fijas, esta pestaña de bultos no tiene eh más información más que el bulto. Pudieras poner en donde está el peso enseguida eh bultos para que en una sola eh  
**Angel Huberto Pulido Burgos:** Ajá.  
**Enrique Lopez:** pestaña pudieras tener ambas eh ambos datos, ¿no? mapas, digo, si dices, no se me complica mucho rollo pero como es un solo campito y  
**Angel Huberto Pulido Burgos:** Sí, más que nada es que este más que nada se puso así porque podemos tener varios bultos.  
   
 

### 01:18:04

   
**Angel Huberto Pulido Burgos:** Pues si te bueno, si te das cuenta en Sag eh tienen un tab específico para bultos por lo mismo, porque pueden tener muchos bultos. Entonces si lo o no.  
**Carlos Alexis Galaviz Rosas:** Ah, perdón, disculpa,  
**Angel Huberto Pulido Burgos:** Dime, dime,  
**Carlos Alexis Galaviz Rosas:** pero ahí tendrías que seleccionar los que ya tienes en almacén,  
**Angel Huberto Pulido Burgos:** dime.  
**Carlos Alexis Galaviz Rosas:** ¿no? Para poder sacarlos, porque si pones tres bultos, ¿qué bultos son?  
**Angel Huberto Pulido Burgos:** Sí, cierto.  
**Carlos Alexis Galaviz Rosas:** No sé si me explico,  
**Angel Huberto Pulido Burgos:** Es que yo creo que ese sería un flujo diferente en el movimiento de salida.  
**Carlos Alexis Galaviz Rosas:** ¿no?  
**Angel Huberto Pulido Burgos:** Se me ocurre si o sea,  
**Carlos Alexis Galaviz Rosas:** de facturas, ¿no?  
**Angel Huberto Pulido Burgos:** aquí ya ves que mencioné ahorita que aquí vas a seleccionar eh no solo vas a mencionar facturas, sino también yo creo que podríamos seleccionar bultos que vas a sacar.  
**Enrique Lopez:** No, pero acuérdense que eh por eso es importante cuando bodega subdivide, o sea, si es una mercancía que entra y sale sin un solo movimento de entrada, una sola referencia, una sola factura, todos los bultos que entran,  
   
 

### 01:19:13

   
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** todos los bultos salen. No hay necesidad de mover bultos, no hay necesidad.  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** Puntos entra, bulto al si hay subvisión. Bodega antes de hacer yo el movimiento de salida, tiene que decirme los vultos que pertenecen a cada factura o a cada referencia. Por eso el tema de su división eh es el que digo, no hemos visto, el cómo yo voy a identificar mi salida a si llegan juntas dos facturas y yo solo voy a sacar una factura. Yo cuando de este movimiento de salida,  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** yo voy a saber la cantidad de de peso y bultos y cómo Bodega va a identificar, cómo va a recibir la instrucción bodega de que oye, solo va a sacar una factura de este de esta entrada.  
**Angel Huberto Pulido Burgos:** Sí,  
**Enrique Lopez:** Sí, ahí sí, pero esto no,  
**Angel Huberto Pulido Burgos:** sí,  
**Enrique Lopez:** yo el movimiento de salida no tengo que hacerlo antes. ¿Por qué? Porque en el movimiento de salida yo voy a tener precios. O sea, tiene que haber en un proceso intermedio entre la entrada,  
**Angel Huberto Pulido Burgos:** sí.  
**Enrique Lopez:** verificación ahí y antes de la salida tiene que haber un punto donde Bodega va a tener que separar un una entrada, va a tener que separar y hacer dos identificaciones, dos referencias, dos datos o no sé cómo llamarlo.  
   
 

### 01:20:33

   
**Angel Huberto Pulido Burgos:** Ahí,  
**Enrique Lopez:** M.  
**Angel Huberto Pulido Burgos:** ahí se me ocurre algo, equipo. No sé si si puedan ahí como que apoyarme porque estoy no sé si estoy en enor, pero se me ocurre que tú cuando tienes una factura eh, dentro de la factura podrías tener qué bultos, o sea, así como tenemos ahorita que tenemos de que ah, tenemos la factura y dentro de la factura tenemos los números de parte y tenemos eh las las disponibilidad de cada número de parte, ¿no? Por ejemplo, este número de parte de esta factura, pues son dos. A este nivel yo creo que podríamos añadirle un índex más que sería el de bultos. Entonces, así evitamos la comunicación, eh, o sea, el que le tenga que decir almacén a Enrique que tiene cada bulto y él pueda aquí decidir, ah, de este bulto lo voy a sacar completo o de esta factura, este bulto sacar estas partidas. en específico. No sé si me entienden ahí, porque con esto que menciona Enrique, pues tiene que estar ligado el bulto a la a las partidas que contiene, pero tengo entendido que no siempre es así, porque no siempre se hace la verificación a nivel eh o sea, o sea, se abre el bulto para ver qué trae.  
   
 

### 01:22:03

   
**Angel Huberto Pulido Burgos:** Pues según yo, si no recuerdo mal,  
**Enrique Lopez:** Este, la inquietud es llegan dos facturas y un movimiento de entrada, una referencia, capturo dos facturas y lo voy a sacar una o voy a hacer dos pedimentos o algo,  
**Angel Huberto Pulido Burgos:** Aha.  
**Enrique Lopez:** pero voy a tener que declarar el peso y bultos de cada una de las facturas. Okay.  
**Angel Huberto Pulido Burgos:** Y es que ahí no puede  
**Enrique Lopez:** Cuando hacen la verificación,  
**Angel Huberto Pulido Burgos:** saber,  
**Enrique Lopez:** o sea, yo no voy a saber y aparte de que pon que yo pueda saber,  
**Angel Huberto Pulido Burgos:** pero almacén sí,  
**Enrique Lopez:** yo puedo saber el peso y bultos. Imagínate que son cinco tarimas, yo tengo dos facturas y yo voy a decir, "Ah, bueno,  
**Angel Huberto Pulido Burgos:** No,  
**Enrique Lopez:** pues voy a poner dos bultos y 300 libras porque en el packing venía, pero físicamente vienen revueltos los cartones." O sea, yo con ellos son los que van a ver y van a decir, "Ah, bueno, de qué cartón voy a sacar piezas. Voy a tener que hacer en ocasiones, si viene revuelta la mercancía 5 bultos, una yo no yo como operación que estoy en Nuevo Laredo no voy a saber en qué en qué bulto está cada pieza, en cada mercancía.  
   
 

### 01:23:20

   
**Enrique Lopez:** Eh, y tampoco puedo mandar instrucción de salida porque, ¿cómo? ya va a salir y apenas van a apenas van a identificar en dónde está cada número de parte de esa factura. O sea, por eso digo, tiene que haber un instrucción intermedia donde yo le pueda decir a Bodega,  
**Angel Huberto Pulido Burgos:** Aha.  
**Enrique Lopez:** "Oye, me llegaron dos facturas, pero solo voy a sacar una factura. Revísame en dónde está esa mercancía de esos 5 Vos y de 5 Vos solo se  
**German Castro:** Sí.  
**Enrique Lopez:** van a armar dos y va a ser este peso.  
**German Castro:** Sí, ese ese es el movimiento de división que hablábamos. Eh, Ángel, ¿te acuerdas que habíamos dicho en la reunión pasada que sí vamos a tener que hacer un movimiento de subdivisión para que hubieras hubiera la instrucción en almacén de solamente subdividir una  
**Angel Huberto Pulido Burgos:** Sí.  
**German Castro:** carga antes de darle salida?  
**Angel Huberto Pulido Burgos:** Sí, es  
**German Castro:** Y es como si como si hicieras,  
**Angel Huberto Pulido Burgos:** cierto.  
**German Castro:** o sea, el movimiento de subdivisión es como si subdividieras una orden de entrada, digo, un movimiento de entrada, porque lo que entró tienes que decirle cómo queda al final dividido.  
**Angel Huberto Pulido Burgos:** Es cierto. Sí. Entonces aquí ya no ocuparías hacer eso porque, o sea, seleccionar el bulto.  
   
 

### 01:24:40

   
**Angel Huberto Pulido Burgos:** Entonces, porque si hiciste la subdivisión e y almacén dice, "Ah, me voy a voy a transportar esta división que contiene esta factura o estas facturas, pues ya sabemos qué bultos son, ¿no?  
**German Castro:** Es correcto. O sea, imagínate, o sea, tienes el movimiento de entrada que va a quedar registrado ahí qué entró y cuándo entró.  
**Angel Huberto Pulido Burgos:** Ajá.  
**German Castro:** Después movimiento de entrada tú le puedes asociar este movimientos de subdivisión.  
**Angel Huberto Pulido Burgos:** Ajá.  
**German Castro:** Entonces te subdivide y te queda en dos partes.  
**Angel Huberto Pulido Burgos:** Ajá.  
**German Castro:** Y de esas dos partes tú jalas alguna de esas dos para movimiento de  
**Angel Huberto Pulido Burgos:** Mm.  
**German Castro:** salida o la otra es que tengas un movimiento de  
**Angel Huberto Pulido Burgos:** Okay, okay,  
**German Castro:** entrada y ese lo jalas directamente hasta movimiento de salida.  
**Angel Huberto Pulido Burgos:** sí, sí,  
**German Castro:** Pero si un movimiento de entrada ya tiene subdivisiones,  
**Angel Huberto Pulido Burgos:** claro.  
**German Castro:** no puede escalar ese movimiento de entrada,  
**Angel Huberto Pulido Burgos:** Tendrías que jalar cada subdivisión.  
**German Castro:** más bien puede escalar las subdivisiones. Es  
**Angel Huberto Pulido Burgos:** M, ya.  
**German Castro:** correcto.  
**Angel Huberto Pulido Burgos:** Y entonces cuando crece el movimiento de salida a partir de una subdivisión o un movimiento, pues ya Enrique va a saber qué bultos pertenecen a ese movimiento aquí, pues porque eso supongo que lo registro almacén en alguna parte en el  
   
 

### 01:25:54

   
**German Castro:** Exactamente,  
**Angel Huberto Pulido Burgos:** sistema.  
**German Castro:** exactamente. verlo con Carlos, pero de tu lado, del lado del ejecutivo, tendría que ser así para que cumpliéramos con el requisito que nos está pidiendo  
**Angel Huberto Pulido Burgos:** Ajá. Entonces, claro,  
**German Castro:** Enrique y que mantuviéramos la consistencia,  
**Angel Huberto Pulido Burgos:** claro, claro. Entonces, bueno,  
**German Castro:** que mantuviéramos la contabilidad, es decir, no puedes subdividir más allá de lo que entró.  
**Angel Huberto Pulido Burgos:** sí, ahí en la UI se me ocurre que no puedo quitar este tabultos, según yo, no sé, pues pueden ser muchas cantidades. Y si lo meto acá,  
**German Castro:** Ahí  
**Angel Huberto Pulido Burgos:** como tú me mencionas, Enrique, creo que se vería bien si son uno o dos bultos nada más,  
**German Castro:** está.  
**Angel Huberto Pulido Burgos:** pero si son muchos se va a ver muy feo.  
**Enrique Lopez:** Pero a verlo,  
**Angel Huberto Pulido Burgos:** A menos que le agregues un un No,  
**Enrique Lopez:** pero a ver ahí del tema de bultos, a  
**Angel Huberto Pulido Burgos:** tú no tú no los vas a añadir. O sea,  
**Enrique Lopez:** ver.  
**Angel Huberto Pulido Burgos:** aquí ya sería nada más este este esto de aquí se va a cambiar y sería informativo nada más para que tú veas qué bultos trae se van a salir en ese movimiento de salida, pero sería solo informativo, pues ya tú no lo  
   
 

### 01:27:04

   
**Enrique Lopez:** Pero qued ponte en bultos.  
**Angel Huberto Pulido Burgos:** seleccionarías  
**Enrique Lopez:** ¿Y qué es lo que vemos ahí? ¿Qué es lo que se va a colocar en ese campo?  
**Angel Huberto Pulido Burgos:** nada. Este se, o sea, hay cuenta que este lo voy a cambiar completo en movimiento de salida y en lugar de tú poner aquí esto que se hace ahorita actualmente está mal, completamente mal, entonces tú ya no verías esto de aquí, sino que verías solamente la información de qué bultos van a salir aquí, porque almacén ya lo registró en el sistema, pues cuando hizo la subdivisión o cuando hizo la verificación o le dio entrada a la mercancía. Entonces, cuando tú selecciones un movimiento completo o un movimiento o las subdivisiones de un movimiento, aquí tú verías eh qué bultos pertenecientes a a esa n cantidad de movimientos o subdivisiones de movimientos. Aquí te saldrían todos los bultos que van a salir en este movimiento que vienen de eso que seleccionaste. Si es así, ¿verdad, Germón?  
**Enrique Lopez:** Pero a nosotros,  
**German Castro:** Sí, es correcto.  
**Enrique Lopez:** digo,  
**German Castro:** No.  
**Enrique Lopez:** está bien que nos informen, pero nosotros lo que vemos actualmente ahorita, ahorita vemos cantidad de bultos, pero nada más como cantidad, ¿no?  
   
 

### 01:28:15

   
**Enrique Lopez:** cinco cinco, o sea, no vemos el detalle de si son son cinco tres cartones, cuatro o qué bulto trae este  
**Angel Huberto Pulido Burgos:** Pero pues supongo que estaría bueno saber si Ah, tengo ah van a ir 20 lotes. Van ahí dos barriles y de esos dos uno es hasmat. Bueno, no puedes combinarlo, ¿no? Pero pues siento que la información no está de más. Pues si tú me dices, "Ah, elimina el tab,  
**German Castro:** Yo creo que yo  
**Angel Huberto Pulido Burgos:** elimina el tab y quiero ver no más yo si son 5 Vos",  
**Enrique Lopez:** O sea, está bien que esté,  
**Angel Huberto Pulido Burgos:** pues lo  
**Enrique Lopez:** no está bien que sea informativo aquí,  
**German Castro:** creo  
**Enrique Lopez:** o sea, no está mal que esté.  
**German Castro:** que  
**Enrique Lopez:** Si si bodega lo va a hacer y va a decir cuántos cartones son y todo el detalle con ganas.  
**Angel Huberto Pulido Burgos:** pongo.  
**Enrique Lopez:** Está bien. O sea, no ahorita no no lo vemos.  
**German Castro:** ¿Estás bien? ¿Qué? Tú qué le acercate y te pones a dar  
**Enrique Lopez:** Pero lo podríamos ver, pero lo que lo que actualmente eh utilizamos solo para el pedimento es la  
**German Castro:** vueltas.  
   
 

### 01:29:14

   
**Enrique Lopez:** cantidad, o sea, esto podría ser informativo.  
**Angel Huberto Pulido Burgos:** Ok.  
**Enrique Lopez:** Mhm.  
**German Castro:** Yo creo que ahí yo  
**Angel Huberto Pulido Burgos:** Sí.  
**German Castro:** creo yo creo que lo que tenemos que hacer es que está diciendo Enrique es no es necesario porque hoy no lo tenemos,  
**Angel Huberto Pulido Burgos:** Adelante,  
**German Castro:** pero no está de más. Entonces hay que usar el drill down en la UI. Es decir, que ellos puedan ver dos bultos. Si le dan clic, pueden ver qué hay dentro o no, porque no en todos no en todos los casos lo va a hacer así almacén. Va a haber casos en donde te va a mostrar a nivel bulto, va a haber casos donde va a ser a nivel este embalaje exterior, va a haber casos en donde te va a mostrar embalaje interior, etcétera. Entonces, como que utilicemos más como la idea de las gráficas de los dashboards de Drill Down, donde les damos a ellos, digamos que el ejecutivo necesita verlo primero alto nivel, de ahí le dan clic al bulto y se abre,  
**Angel Huberto Pulido Burgos:** Ya.  
**German Castro:** de ahí le dan clic a la caja y se abre y hasta donde tengamos detalle, porque no en todos vamos a tener ese  
**Angel Huberto Pulido Burgos:** Ya.  
**German Castro:** detalle.  
**Angel Huberto Pulido Burgos:** Ya. Entonces ahí lo que podría hacer es si eliminar el tab y aquí en donde está el peso aquí añadir la cantidad de bultos y ya si tiene detalle picarle al detalle que te abra algo que te muestre qué bultos son si quieren  
   
 

### 01:30:34

   
**German Castro:** Es correcto.  
**Angel Huberto Pulido Burgos:** verlo.  
**German Castro:** Sí, yo creo que así. ¿Te parece bien así,  
**Angel Huberto Pulido Burgos:** Entonces, el  
**German Castro:** Enrique? No, a tener unas marcas  
**Enrique Lopez:** Sí, digo, igual si lo ponen cuando lo pongan lo podemos ver.  
**Angel Huberto Pulido Burgos:** ticket  
**Enrique Lopez:** Digo, al final a nosotros lo que nos hace ver ahorita el pedimento es cantidad, bultos y bultos, me refiero a tarimas, o sea, pudiera ser que en el detalle, digo, esto ahorita no lo vemos, pero si al momento de que den arriga va a determinar, tenemos una tarima que al final yo declaro un vulto, ¿okay? Teclado uno y en el detalle va a poner una tarima con 50 cartones.  
**Angel Huberto Pulido Burgos:** Bien.  
**Enrique Lopez:** Ah, okay. Y yo voy a poder ver y nada más darle clic y poder observar. Ah, bueno, este bulto trae 50 cartones. Okay, es informativo y está muy bien.  
**Angel Huberto Pulido Burgos:** Sí, sí, claro. De acuerdo. Totalmente tú, Enrique. Entonces, el ticket sería en movimiento de salida. Vamos a eliminar el tab de bultos, no no se pueden añadir bultos aquí al crear el movimiento de salida, sino que eh cuando entremos al tab de transporte, al lado del eh campo de peso del embarque vamos a tener la cantidad de bultos totales que tenemos en base a lo que los movimientos o subdivisiones que se seleccionaron para este movimiento de salida.  
   
 

### 01:32:02

   
**Angel Huberto Pulido Burgos:** y eh en caso de tener detalles sobre los bultos, pues poderlo mostrar de alguna manera. Ese sería el ticket. Okay. Eh, algo similar creo que pasaba con este de contenedores, ¿verdad, Enrique?  
**Enrique Lopez:** de ese contenedores. Hablamos de que sí si era creo que el transporte también traía número de vehículo, traía identificación, algo así también traía  
**Angel Huberto Pulido Burgos:** Si en transporte teníamos aquí tipo dónde  
**Enrique Lopez:** identificación de transporte.  
**Angel Huberto Pulido Burgos:** estaba  
**Enrique Lopez:** eh más abajo donde dice datos de cruce fronterizo,  
**Angel Huberto Pulido Burgos:** aquí.  
**Enrique Lopez:** identificación de transporte.  
**Angel Huberto Pulido Burgos:** Ajá.  
**Enrique Lopez:** Digo, el otro tap está bien en el tema de que este eh tipo de contenedor si lo usamos o si declaramos eso eh número tipo, o sea, esos datos sí los usamos.  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** Sí, sí los hicimos. Ahí fue donde entramos al tema de que las partes dos, de que cómo lo vamos a don el tema de las partes dos  
**Angel Huberto Pulido Burgos:** aquí eh pues ustedes no van a añadir el el en el movimiento de salida no van a añadir los contenedores, ¿no? Eh, el contenedor sí  
**Enrique Lopez:** Si esto va para el pedimento.  
   
 

### 01:33:38

   
**Angel Huberto Pulido Burgos:** si se ocupa para el pedimento. Sí, desde aquí se puede tomar.  
**Enrique Lopez:** Sí se ocupa, ¿no? Sí, se ocupa, o sea, si sí lo colocamos y además porque con esta instrucción de salida se supone que tenemos que colocar el vehículo en el que va cargar bodega, M.  
**Angel Huberto Pulido Burgos:** Sí, claro. Ese ese número de vehículo ustedes lo tienen, ¿no? O sea, eh ustedes se lo pasan a bodega.  
**Carlos Alexis Galaviz Rosas:** Lo puede tener logística también,  
**Angel Huberto Pulido Burgos:** Pues ah o podemos hacerle aquí. Sí, es que sí, ya me acordé. Aquí quedamos en que para movimiento de salida terrestre no ocupamos el tab de contenedores, se va a eliminar y eh solo es un contenedor por movimiento de salida. Entonces ese lo podemos mostrar aquí que mencionabas Enrique en la reunión pasada a nivel identificación del transporte, pues aquí abajo añadir esos tres campos nada más de tipo de contenedor, número de contenedor y número de sello. Aquí la duda sería de dónde van a salir esos datos. Si el identificación del transporte y eh tipo de contenedor, número de contenedor y número de sello lo conoce el ejecutivo o se jalaría de alguna parte de de algún eh de alguna parte que haya puesto el almacén.  
   
 

### 01:35:09

   
**Angel Huberto Pulido Burgos:** o o de dónde se obtendrían esos datos serían modificables, ¿no? Para que los ponga el ejecutivo o no. Listo. Okay, entonces aquí Si está bien así, Enrique o equipo, ponemos los tres campos modificables aquí a un lado de identificación del transporte o qué opinan.  
**Carlos Alexis Galaviz Rosas:** Enrique estaba en llamada. No sé si te escucho esa parte.  
**Angel Huberto Pulido Burgos:** Ah, bueno, ahorita que termine pues ya es bueno contigo,  
**Carlos Alexis Galaviz Rosas:** Creo que  
**Angel Huberto Pulido Burgos:** Carlos, nada más la duda, estos estos datos no los pone almacén, ¿verdad? ¿O sí?  
**Carlos Alexis Galaviz Rosas:** no es que  
**Angel Huberto Pulido Burgos:** identificación del transporte. Eh, tipo de contenedor.  
**Carlos Alexis Galaviz Rosas:** ahí puede que se cruce Ángel.  
**Angel Huberto Pulido Burgos:** Ajá.  
**Carlos Alexis Galaviz Rosas:** Tendríamos que revisarlo. No sé si está y aquíón,  
**Angel Huberto Pulido Burgos:** No, no está.  
**Carlos Alexis Galaviz Rosas:** pero este tendríamos que ver si no se cruza con alguna cita y si se cruza pues ya tendríamos esos datos. Déjame los reviso yo con ellos.  
   
 

### 01:37:03

   
**Angel Huberto Pulido Burgos:** Okay. Sí.  
**Carlos Alexis Galaviz Rosas:** Gracias.  
**Angel Huberto Pulido Burgos:** Espera.  
**German Castro:** Escucha, perdón, dejé de escuchar por un momento. Ángel,  
**Angel Huberto Pulido Burgos:** Eh, creo que Enrique está en llamada.  
**German Castro:** estás tirando. Deja de mira  
**Angel Huberto Pulido Burgos:** Pero nos quedamos aquí en la duda, bueno, no en la duda,  
**German Castro:** más  
**Angel Huberto Pulido Burgos:** sino en el comentario de que eh este tab de contenedores se va a eliminar. Esto se comentar la reunión pasada, se va a eliminar y eh como habíamos quedado que en movimiento de salida terrestre eh solamente es un contenedor por movimiento, no no hay más de uno, se va a mover eh nada más estos tres campos al al a la parte de transporte y se van a poner aquí a un lado de identificación del transporte.  
**German Castro:** Okay, mira, acuérdate que todo esto salió porque querían ver justamente cómo se iba a subdividir. Entonces,  
**Angel Huberto Pulido Burgos:** Ándale.  
**German Castro:** yo creo que a partir de esto, o sea, si ahorita ahorita que terminas de de validar los campos, es generar así con lo que tienes dos movimientos de salida y llevarlo a la  
**Angel Huberto Pulido Burgos:** Ajá.  
**German Castro:** operación para que valide ese salto entre movimiento de salida y operación a ver si está correcto como lo tenemos.  
   
 

### 01:39:11

   
**Angel Huberto Pulido Burgos:** Okay. Sí,  
**German Castro:** No puse el espacio donde van y  
**Angel Huberto Pulido Burgos:** de acuerdo. Aquí podemos hacerlo.  
**German Castro:** quedar todo lo puse determin. Y otra cosa que nos falta validar con almacén es cómo ve almacén cuando ellos generan movimiento de salida. ¿Qué es lo que ve almacén? ¿Lo ven a nivel de la aplicación web con Yoli y con Limón o lo ven a través de la aplicación móvil? ¿Dónde es donde lo van a ver? Eso es otra interacción también falta verla. para también entender la el movimiento de de subdivisión que se va a hacer ese cómo lo debe de reflejar Carlos del lado de  
**Angel Huberto Pulido Burgos:** Ah.  
**German Castro:** almacén  
**Angel Huberto Pulido Burgos:** Sí.  
**German Castro:** limón. En el caso de almacén, cuando les llega subdivisión, ¿qué es lo que ustedes necesitan hacer hoy en día?  
**Daniel Peña:** Hola, Germán, buenas tardes. Veo que no está ni Limón ni Yolanda conectados.  
**Carlos Alexis Galaviz Rosas:** están  
**German Castro:** Estaban hace rato, pero no sé. No sé si sigan.  
**Carlos Alexis Galaviz Rosas:** creo que iban a cerrar temas de almacén ahorita.  
**Daniel Peña:** Sí, ahorita no están, pero lo que tenemos conocimiento es de que una vez que nosotros enviamos el formato de subdivisión,  
   
 

### 01:40:41

   
**German Castro:** Okay.  
**Daniel Peña:** ellos regresan al primer paso que es la verificación otra vez,  
**German Castro:** Ajá.  
**Daniel Peña:** pero ya separados cada quien con su referencia, que en actualmente es el guion 1, guion 2, guion 3, pero vuelven a verificar toda la emergencia. Vuen a verificar bultos y anotanlo en la nueva  
**German Castro:** Claro.  
**Daniel Peña:** información.  
**German Castro:** Y tenemos una cola de trabajo para esas para esas casos, Carlos,  
**Carlos Alexis Galaviz Rosas:** No, pero lo podemos agregar nada más. Creo que  
**German Castro:** pero ahí como lo vamos a manejar. ¿Va a ser una sola fila de trabajo para todo warehouse o va a ser una una fila de trabajo por cada tipo  
**Carlos Alexis Galaviz Rosas:** la  
**German Castro:** de movimiento que tienen que hacer?  
**Carlos Alexis Galaviz Rosas:** es una cola de trabajo por para todos los warehouse porque cualquier todos los warehous deben de poder hacer todas las actividades,  
**German Castro:** Okay.  
**Carlos Alexis Galaviz Rosas:** pero si es necesario lo podemos dividir por tipos de cola y termina Hacemos asignaciones en base  
**German Castro:** Oye,  
**Carlos Alexis Galaviz Rosas:** a perfil.  
**German Castro:** ya acércate a la mesa. Más bien hay que hay que validarlo para hay que validarlo con almacén, más bien el viernes que entre Yoli o en la semana hacer nada más esa verificación rápida con Yoli o con Limón,  
   
 

### 01:42:10

   
**Carlos Alexis Galaviz Rosas:** Va, de acuerdo. Yeah.  
**German Castro:** porque eso les queda mucha duda de cuando el ejecutivo está generando el movimiento de entrada, movimiento de salida y ahora el movimiento de de subdivisión y el movimiento de reexpedición, ver qué va a pasar del otro lado. Todos salón.  
**Carlos Alexis Galaviz Rosas:** Okay, de acuerdo. Yeah.  
**Angel Huberto Pulido Burgos:** Bueno, eh no sé si ya podemos continuar o o  
**Enrique Lopez:** ¿Qué? No dije en qué en qué en qué nos detuvimos.  
**Angel Huberto Pulido Burgos:** Ah, okay. No, te estamos esperando a ti.  
**Enrique Lopez:** Ya volví. Te ha una llamada en Capoía.  
**Angel Huberto Pulido Burgos:** No, está bien. A ver, eh, pues en el en el caso este Enrique de contenedores mencionábamos que pues vamos a mover estos estos tres campos, vamos a quitar el tab de contenedores y vamos a mover estos tres campos de tipo de contenedor, número de contenedor y número de sello aquí al apartado de aquí en esta parte donde está identificación del transporte para que puedan poner esos tres campos aquí, pues. Eh,  
**Enrique Lopez:** Muy bien.  
**Angel Huberto Pulido Burgos:** nada más que me surgió la duda de que si el estos  
**Enrique Lopez:** Yeah.  
   
 

### 01:48:03

   
**Angel Huberto Pulido Burgos:** datos eh ustedes son los que los tienen o es almacel el que los proporciona o o cómo funciona con esta parte de aquí.  
**Enrique Lopez:** En el tema de las cajas directas, o sea, las cajas que llegan con que no se descargan, pues desde la entrada pudiéramos jalarlos. O sea,  
**Angel Huberto Pulido Burgos:** Claro.  
**Enrique Lopez:** si le pones directa y coloca los datos, que te los jale en un movimiento de salida porque se ve en la misma caja.  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** Si en el movimiento de entrada o referencia no queremos vehículo, si est si aquí son los datos donde se van a declarar para pedimento, aquí se agregaría por parte del ejecutivo, si es caja, si son placas, el tipo,  
**Angel Huberto Pulido Burgos:** Okay.  
**Enrique Lopez:** etcétera.  
**Angel Huberto Pulido Burgos:** Y y esos datos los tienen ustedes o te los pasa almacén.  
**Enrique Lopez:** Esos datos no los proporciona eh pues ya sea el transportista del cliente, lo etcétera, dependiendo cómo lo manejamos,  
**Angel Huberto Pulido Burgos:** Ah,  
**Enrique Lopez:** pero son datos que nos proporcionen nosotros a operación.  
**Angel Huberto Pulido Burgos:** okay.  
**Enrique Lopez:** Ya nosotros nos encargamos de agregarlos.  
**Angel Huberto Pulido Burgos:** Bueno, okay. Entonces, los añadimos ahí. Eh, bueno, ya terminamos con el formulario de movimiento de salida terrestre, que va a quedar con ese nombre, porque recordemos que vamos a tener varios movimientos de salida, no solamente uno.  
   
 

### 01:49:27

   
**Angel Huberto Pulido Burgos:** Y y así quedaría como mencionamos ahorita, el ticket para el formulario de movimiento de salida ahí para que lo anote la inteligencia artificial y quede eh completo el movimiento de este. Ya. Entonces, ahorita lo que queríamos ver, Enrique también, eh, no sé, Germán, no sé hasta qué hora será la reunión por ustedes, pues yo yo puedo seguirle, pero no sé ustedes.  
**German Castro:** Es hasta las  
**Angel Huberto Pulido Burgos:** Bueno,  
**German Castro:** 6\.  
**Angel Huberto Pulido Burgos:** yo digo que sí alcanzamos. Entonces, aquí, por ejemplo, eh después de haber generado el movimiento de salida, lo que se hace actualmente con lo que hay ahorita en Sac 3 es que nos aparece este botoncito de aquí. eh que nos dice generar pedimiento y nos abre el formulario para generar la operación. Okay. Entonces, lo que quería validar ahorita, lo que queríamos validar que mencionó Germán ahorita Enrique Canabas, era que nos validaras este salto de un movimiento de salida hacia la generación de la operación. Nosotros aquí generamos el movimiento de salida y a partir de ese movimiento creamos la operación que si si recuerdas en un ticket quedó que eh tendríamos que tener la posibilidad de marcar varios movimientos de salida y generar un movimi una operación eh para todos los movimientos eh o o solamente un un movimiento, pero más Más que nada lo que queríamos ver era la transición.  
   
 

### 01:51:05

   
**Angel Huberto Pulido Burgos:** la transición si no si no estoy mal, Germán o si quieres  
**German Castro:** Sí, lo que me queda duda con estos cambios que estamos haciendo es  
**Angel Huberto Pulido Burgos:** Tu  
**German Castro:** que la idea del movimiento de salida es que justamente fueran preparando la carga en almacén para que después se generara la operación y entonces ya saliera. Pero aquí lo que quiero entender es que si se hace la subdivisión, ahí te preparan la carga y luego tú ya les das la salida y después con esa salida haces la operación. O si el orden es subdividimos, generamos la operación y de ahí después generamos la orden de salida. Ese orden lógico es el que no me queda todavía muy claro para que quede bien modelado a como en realidad ocurren las cosas.  
**Angel Huberto Pulido Burgos:** Creo que sí sería primero movimiento de salida, después operación, Germán, porque quiero quiero mencionarlo para que Enrique me corrija y y que aclarar mi idea. Según yo es eh almacén o los de operación hacen la solicitud almacén y ya después se genera la operación, ¿no? Se genera la operación y en base a la operación se genera la solicitud en ese mismo momento.  
**German Castro:** Eso, eso es lo que quiero entender yo.  
**Angel Huberto Pulido Burgos:** Ajá. En Sigac 2\. Si es así, ¿verdad, Enrique?  
   
 

### 01:52:31

   
**Angel Huberto Pulido Burgos:** ¿Cómo se hace actualment?  
**German Castro:** Porque lo que no quiero es que les quede un paso extra este Ángel, o sea, que si hoy en día al generar operación automáticamente se hace el movimiento de salida,  
**Angel Huberto Pulido Burgos:** Ajá.  
**German Castro:** pues no tiene caso que nosotros les hagamos hacer movimiento de salida y la demás operación, porque entonces les estamos agregando un paso en vez de quitárselos.  
**Angel Huberto Pulido Burgos:** Sí, claro. No, de acuerdo.  
**Enrique Lopez:** Sí, mira, te voy a explicar lo que hacemos y igual aquí hay varios y a ver si tiramos lo mismo.  
**Angel Huberto Pulido Burgos:** Totalmente.  
**Enrique Lopez:** Ahorita ¿Qué pasa? ¿Qué pasa, señorita? Tengo déjame formular. Pues es que si hacemos la operación después, eh, yo aunque genere el movimiento de salida, yo todavía no quiero que salga. ¿Por qué? Porque todavía tengo que generar un pedimento, tengo que generar una operación, todavía no tengo nada. Y quizás en ese pedimento diga, "Voy a cotizar, voy a hacer esto." O sea, aún y cuando haga un movimiento de salida no quiere decir que ya voy a enviar ese movimiento de salida o ya le diga a bodega que ya va a salir porque tengo que generar operación, pedimento, solicitud de fondos y demás.  
   
 

### 01:53:40

   
**Enrique Lopez:** O sea, aún y aún y cuando haga mi movimiento de salida, pues no quiere decir que ya lo voy a sacar, sino simplemente tengo que hacerlo porque sin movimiento de salida no tengo operación y pedimio y la requiero para generar unos solitos fondos. O si yo o si yo quiero tener una operación anticipada de que va a llegar la mercancía,  
**Angel Huberto Pulido Burgos:** ¿Y  
**Enrique Lopez:** pero ya quiero tener el pedimiento listo losado, tengo que hacer un movimiento de salida, pero no quiere decir que ya va a salir. Okay.  
**Angel Huberto Pulido Burgos:** cómo y cómo crees tú,  
**Enrique Lopez:** Y ahorita me va a obligar a hacerlo.  
**Angel Huberto Pulido Burgos:** Enrique, que que podría quedar más  
**Enrique Lopez:** Yo creo que el movimiento de salida debería de ser después de la operación de impedimento,  
**Angel Huberto Pulido Burgos:** fácil?  
**Enrique Lopez:** o sea, después. ¿Por qué? Porque debería ser ¿Qué es lo que hacemos ahorita? Hacemos una operación, pedimento, en el pedimento tenemos COVID, document, podemos validar, podemos hacer 1000 cosas, pero la instrucción de salida que nosotros enviamos es a darle un evento que le damos envío de ordón de carga. Es cuando en realidad bodega dice, "Ah, okay, ya me está haciendo esa instrucción de salida para la mercancía, pero yo ya hice impedimento, yo ya generé solito de fondos, yo ya generé COVID, yo generé document, eh yo ya hice hasta pruebas de validación, yo ya hice 1000 cosas.  
   
 

### 01:54:47

   
**Enrique Lopez:** Es más, pudo haber hecho hasta pagar el pedimento y todavía no le mando la salida porque a lo mejor la mercancía todavía no llega, pero yo sé que en qué va cruzar. Yo voy yo puedo hacer todo antes sin siquiera hacer la salida. Y ahorita si lo hacemos de esta forma tendría que hacer yo una salida cuando a lo mejor ni siquiera estoy preparado para hacerlo. No sé los demás que cómo ven las cosas. M.  
**Angel Huberto Pulido Burgos:** servicio.  
**German Castro:** A ver, entonces, para que para estar claros, eh la operación puede generarse a la par y hasta hasta después de la operación, en algún punto de la operación se genera el movimiento de salida.  
**Enrique Lopez:** Sí, pudiera ser, o sea, pudiera ser que ahorita como lo tenemos que decimos movimiento de salida, hago mi operación, pedimento, genero cobes y document, este, pero yo todavía no le voy a mandar una instrucción a bodega de su salida. Okay, la hago porque el proceso me lo está pidiendo, pero no quiere decir que ahorita lo vaya a generar, no Ok.  
**Angel Huberto Pulido Burgos:** Lo lo que se hace en Sigac dos, German es que eh ellos generan la operación y ya que ellos ya quieren mandan un evento de orden de carga almacén y almacén hace la orden de carga. Entonces, si queremos que quede igual que en Sigac 2, sería crear la operación y después el movimiento de y después enviar la salida, la orden de salida almacén, pero creo que no se presta.  
   
 

### 01:56:44

   
**Angel Huberto Pulido Burgos:** Sí, actri. Para eso lo que sí se prestaría sería eh se me ocurriría a mí es generar la operación y el movimiento al mismo tiempo, poder generar operación y movimiento al mismo tiempo, pero enviarle la orden almacén de carga cuando yo quiera, que almacén sepa que hay una salida, pero la orden de carga para esa salida todavía no está hasta que yo le indiqué. Es lo que se me ocurre que podríamos hacer en C  
**German Castro:** Okay,  
**Angel Huberto Pulido Burgos:** 3\.  
**German Castro:** pero a ver, es que el movimiento de salida en realidad nos va, o sea, su naturaleza es para controlar cuando la mercancía ya va a salir en realidad,  
**Angel Huberto Pulido Burgos:** Sí. Yeah.  
**German Castro:** ¿cierto? Ahorita tú tienes eh eh digamos que ligado la  
**Angel Huberto Pulido Burgos:** Sí,  
**German Castro:** operación a un movimiento de salida, ¿cierto?  
**Angel Huberto Pulido Burgos:** sí.  
**German Castro:** Pero podríamos asociar la generación de una operación a un movimiento de entrada o un movimiento de subdivisión.  
**Angel Huberto Pulido Burgos:** Ahí ya no. A ver, no te entendí. Podríamos qué?  
**German Castro:** En vez de que tengas que jalar información de un movimiento de salida, que jales información de un movimiento de entrada o de un movimiento de subdivisión.  
   
 

### 01:58:16

   
**Angel Huberto Pulido Burgos:** Mhm.  
**German Castro:** Y como ya tienes esa esa información desde ahí es lo que te va a decir qué es lo que vas a cargar y qué es lo que ellos pueden hacer todas operaciones y en el momento que le das play llega un momento en donde parte de ese play debe de ser el la generación del movimiento de salida o lo dejamos como un botón como la el de solicitud de fondos.  
**Angel Huberto Pulido Burgos:** para que ahí se genere el movimiento de salida.  
**German Castro:** Exacto. Y lo generamos a partir de la información que ya está en operación y la información que está a partir del movimiento de entrada o del movimiento de  
**Angel Huberto Pulido Burgos:** Entonces,  
**German Castro:** subdivisión.  
**Angel Huberto Pulido Burgos:** lo que tú dices sería tener todo este formulario de movimiento de salida, tenerlo dentro del mismo formulario de operación.  
**German Castro:** pues no dentro del formulario de operación, pero sí sí como parte de ese flujo. Es como es como un paso más de de lo que tienes que hacer dentro de tu flujo de operación,  
**Angel Huberto Pulido Burgos:** Aha.  
**German Castro:** porque inclusive si no necesitas ningún dato, porque lo que vi fue que nos fueron quitando datos, nos fueron quitando datos, ¿qué datos te van a quedar? Son datos que ya jalamos porque es parte de lo que te pedí Enrique, jalar esos datos del movimiento de entrada o jalar esos datos del movimiento de subdivisión.  
   
 

### 01:59:45

   
**German Castro:** Entonces, en realidad el movimiento de salida es solamente decirle a a almacén, va a salir esto y todos esos datos los jalas de del movimiento de entrada o el movimiento de subdivisión y queda automatizado.  
**Angel Huberto Pulido Burgos:** Mhm. Es que ahí de hecho sí quedarían, o sea, de hecho van a quedar datos mínimos, pues porque si nosotros Ajá,  
**German Castro:** Exacto. No.  
**Angel Huberto Pulido Burgos:** con la idea que tú tienes sería no crear la salida a partir de las entradas, sino crear la operación a partir de las entradas y y al crear la operación tenemos el formulario de operación. que hace los cálculos y puedes crear pedimento, etcétera, pero como un paso extra, en lugar de tener este como estos pasos de aquí o luego veo cómo lo añado la Yai, tener los campos que quedaron en el movimiento de salida requeridos, por ejemplo, el el el de transporte de lo que sí pone almacén para el el o almacén, el ejecutivo como el el la identificación del transporte, el número de sello, número de contenedor y tipo de contenedor, que sería, creo que lo único que ocupa poner, entre otros campos que puede modificar como el peso, fecha estimada de salida y no más tipo de material para Nestle, pero este lo podemos tener directamente en el movimiento de entrada, no lo ocupamos aquí en movimiento de salida.  
   
 

### 02:01:38

   
**Angel Huberto Pulido Burgos:** Este se va a quitar, este se va a quitar. Sí, lo podríamos simplificar así como tú dices, crear este lo jalas. ¿Y cómo podría quedar? Entonces ellos jalan, crean operación, la terminan y cuando yo tenga la operación desde aquí yo podría decir solicitar salida y ahí generar el movimiento de, o sea, que se abra un minimodal con esos campos que son  
**German Castro:** Ajá,  
**Angel Huberto Pulido Burgos:** bien poquitos y ya los puedan ellos Dios poner.  
**German Castro:** es correcto. Eso queda más natural. ¿Estás de acuerdo?  
**Angel Huberto Pulido Burgos:** Sí. Y bueno,  
**German Castro:** Señor.  
**Angel Huberto Pulido Burgos:** no sería tan chiquito el modal porque también ocupan ver la información del movimiento de salida. Bueno, no es que ni siquiera ocupan en el movimiento de salida información de de que jaló de acá, pues esa la podrías tener a nivel de talle de operación, pues, por  
**German Castro:** Es correcto.  
**Angel Huberto Pulido Burgos:** ejemplo. Okay, okay, okay, de acuerdo. Totalmente. Muy muy buena idea. No sé qué opina el equipo sobre eso que se mencionó.  
   
 

### 02:03:06

   
**Angel Huberto Pulido Burgos:** Ş.  
**Enrique Lopez:** Sí, digo, creo, digo, ahora sí que escuchaba un poquito el punto es de que minimizar pasos y traer datos que ya tenemos para que sea más fácil el crear algo, ¿no? Como ahorita vemos en movimiento salida, eh se puede funcionar con el tema de la operación, como mencionaban y y si ya estos campos la mayoría los vamos a tener desde antes, pues va a ser, o sea, nos va a ahorrar un paso o pasos. Está bien. Lo que sí, mira, ya sea cuando se manda una salida o algo, quizás. Bueno, vamos, digo, no más ese evento el cómo vamos a notificar cuando en realidad sí de salida, como este tiro el evento de orden de carga que tenemos, nada más en qué momento si digamos, vamos a decirle ya ahora sí a ver de esta salida ya está habilitado o no o este movimiento para carga y despacho va a estar habilitado o no. ¿Cómo se van cómo se va a llevar ese ese proceso?  
**Angel Huberto Pulido Burgos:** No entendí tu duda. O sea, tu duda sería cuándo vas a hacerle la solicitud de salida a  
**Enrique Lopez:** Sí, sí, sí. No es que quede definido en qué, o sea, eh que creo que tú ahorita lo mencionaste bien,  
**Angel Huberto Pulido Burgos:** almacén.  
   
 

### 02:04:28

   
**Enrique Lopez:** ahorita solo le damos un clic y es esa instrucción se envía, o sea, nada más ese esa ese disparador de cuando se crea una salida este que esté ahí presentando un plan eh podamos hacer todo,  
**Angel Huberto Pulido Burgos:** Mhm.  
**Enrique Lopez:** pero bodega no va a hacer nada hasta que yo le indique. Ahora sí va para allá.  
**Angel Huberto Pulido Burgos:** Exactamente así.  
**Enrique Lopez:** Okay.  
**Angel Huberto Pulido Burgos:** Entonces, lo que quitaríamos sería este paso de crear la salida porque ha de cuenta que lo que dice Germán es que tú tienes, por ejemplo, aquí tres movimientos y tú dices, "Ah, se van a ir en una sola salida, en un solo en una sola salida de los tres movimientos. creo la operación a partir de de los movimientos y al crear la operación me va a jalar los datos de esas entradas que mencionábamos aquí. Bueno, que tú mencionabas, por ejemplo, eh los bultos, eh pues guías nos ocupa, eh los bultos, eh el peso del embarque, eh el la hora estimada de salida en caso de de venir, ¿no? Esta creo que ustedes la pondrían, pues jalaría pues datos que es que este también pues los datos que tenga que jalar y ya cuando tengamos los datos para el movimiento de salida que que ustedes se ocupan poner, estos los pondrías a nivel solicitud de salida aquí en la operación.  
   
 

### 02:05:58

   
**Angel Huberto Pulido Burgos:** Pues me explico.  
**Enrique Lopez:** Okay. Bueno, digo este ahora sí que a lo mejor te va a tocar eh hacer algunos cambios ahora sí un poco más detallados para para ya actualizar.  
**Angel Huberto Pulido Burgos:** Sí, claro.  
**German Castro:** Okay, pero ese ese movimiento queda más natural así, ¿no? Ese flujo trabajo, o sea, nos vamos por ese camino. ¿Estás de acuerdo?  
**Enrique Lopez:** Sí, sí, sí. Digo, el eh digo, ya cuando veamos esos datos acá en la operación, creo que está bien, o sea, porque digo, nos va a ahorrar un proceso y al final mucha la información ya la vamos a tomar. O sea, me parece todo lo que sea eh utilizar lo que ya tenemos, quitar pasos, está bien. Ya cuando este Ángel pueda trabajar y mostrárnos cómo va a quedar ahora la nueva pantalla de de la operación, pues ya lo vamos visualizando y viendo. Si teníamos un paso que quizás ya no era tan útil, pues mejor.  
**German Castro:** Okay, perfecto. Vamos a hacerle así, Ángel. Vamos a analizar cómo queda y ya se los mostramos, ¿te parece?  
**Angel Huberto Pulido Burgos:** Sí, sí, de acuerdo. Completamente sale.  
**German Castro:** Perfecto.  
**Enrique Lopez:** Bueno,  
**Angel Huberto Pulido Burgos:** Muchas gracias.  
**German Castro:** Pues muchas gracias, muchachos.  
**Enrique Lopez:** yo creo que porcias  
**German Castro:** Nos vemos el viernes.  
**Angel Huberto Pulido Burgos:** Nos vemos. Bye.  
**Enrique Lopez:** Adi  
**German Castro:** Hasta luego.  
**Carlos Alexis Galaviz Rosas:** Podemos.  
   
 

### La transcripción finalizó después de 02:08:05

*Esta transcripción editable se ha generado por ordenador y puede contener errores. Los usuarios también pueden cambiar el texto después de que se haya generado.*