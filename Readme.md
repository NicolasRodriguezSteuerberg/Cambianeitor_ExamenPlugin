# Cambianeitor

## Índice
+ [Descripción](#descripcion)
+ [¿Cómo funciona?](#como-funciona)
  + [Remplazo de palabras en el contenido de las entradas y páginas y los titulos](#remplazo-de-palabras-en-el-contenido-de-las-entradas-y-paginas-y-los-titulos)
  + [Creacion de la tabla en la base de datos](#creacion-de-la-tabla-en-la-base-de-datos)
  + [Insertar los datos en la tabla](#insertar-los-datos-en-la-tabla)
  + [Seleccionar los datos de la tabla](#seleccionar-los-datos-de-la-tabla)
  + [Resumen](#resumen)
+ [Cómo usar](#como-usar)
+ [Que es add_filter](#que-es-add_filter)
+ [Uso en el Contexto de WordPress](#uso-en-el-contexto-de-wordpress)


### Descripcion
Cambianeitor es un plugin de simple de filtrado de contenido, diseñada para remplazar palabras de los titulos o del contenido de las entradas y páginas de WordPress por sus respectivos signos  

### ¿Cómo funciona?
+ El plugin escanea el contenido de entradas y páginas y sus respectivos titulo en busca de una lista predefinida de palabras (signos).
+ Cuando encuentra una palabra que representa un signo, la reemplaza por su signo correspondiente.
+ Ahora explicaré las funciones que contiene este plugin

#### Remplazo de palabras en el contenido de las entradas y páginas y los titulos
```
    function cambiar_signos($text){
        // coge las palabras de los signos y los signos de la base de datos
        $words = selectSignos();
    
        foreach ($words as $result){
            $listaDeshechos[] = $result->deshechos;
            $listaQueridos[] = $result->queridos;
        }
        return str_replace($listaDeshechos, $listaQueridos, $text);
    }
    
    add_filter('the_content', 'cambiar_signos');
    add_filter('the_title', 'cambiar_signos');
```
Esta funcion se utiliza para añadir los filtros. Recupera las palabras y signos de la base de datos mediante la función selectSignos, de esta manera despues remplaza las palabras encontradas en el contenido y en los titulo por sus respectivos signos.

#### Creacion de la tabla en la base de datos
```
    function createTable(){
        global $wpdb;
        $table_name = $wpdb->prefix . "cambianeitor";
        //Charset de la tabla
        $charset_collate = $wpdb->get_charset_collate();
        //Sentencia SQL
        $sql = "CREATE TABLE IF NOT EXISTS $table_name (
            id mediumint(9) NOT NULL AUTO_INCREMENT,
            deshechos varchar(255) NOT NULL,
            queridos varchar(255) NOT NULL,
            PRIMARY KEY  (id)
        ) $charset_collate;";
        //Incluir el fichero para poder ejecutar dbDelta
        require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
        dbDelta( $sql );
    }
    // Cuando se activa el plugin, se crea la tabla (si no estaba creada)
    add_action("plugins_loaded", "createTable");
```
La funcion _createTable_ se utiliza para crear la tabla cuando se activa el plugin(gracias al add_action). Se utiliza la función **dbDelta** para garantizar que la tabla se cree correctamente y se respeten las convenciones de la base de datos de Wordpress

#### Insertar los datos en la tabla
```
    function insertSignos($deshechos, $queridos){
        global $wpdb, $listaDeshechos, $listaQueridos;
        $table_name = $wpdb->prefix . "cambianeitor";
        $flag = $wpdb->get_results("SELECT * FROM $table_name");
        if (count($flag)==0){
            for ($i = 0; $i < count($listaDeshechos); $i++){
                $wpdb->insert(
                    $table_name,
                    array(
                        'deshechos' => $listaDeshechos[$i],
                        'queridos' => $listaQueridos[$i]
                    )
                );
            }
        }
    }
    // Cuando se activa el plugin, se insertan los datos en la tabla (si no estaban ya insertados)
    add_action("plugins_loaded", "insertSignos");
```
La funcion _insertSignos_ se utiliza para insertar los datos en la tabla cuando se activa el plugin(igual que con el create gracias al add_action). Verifica si ya hay datos en la tabla, si no es así, inserta las palabras y signos proporcionados en las listas **$listaDeshechos** y **$listaQueridos**.

#### Seleccionar los datos de la tabla
```
    function selectSignos(){
        global $wpdb;
        $table_name = $wpdb->prefix . 'cambianeitor';
        $results = $wpdb->get_results( "SELECT * FROM $table_name" );
        return $results;
    }
```
La función _selectSignos_ se utiliza para seleccionar todas las filas de la tabla de la base de datos. Retorna un array con los resultados que luego se utilizan para realizar el reemplazo de palabras en el contenido y los títulos.

#### Resumen
En resumen, el plugin Cambianeitor reemplaza palabras específicas por sus respectivos signos en el contenido y los títulos de las publicaciones de WordPress. Además, utiliza una tabla en la base de datos para almacenar las asociaciones de palabras y signos, la cual se crea y se llena con datos al activar el plugin.

### Cómo usar
+ Una vez instalado y activado, el plugin comenzará a filtrar automáticamente el contenido de las entradas y páginas durante su visualización.

### Que es add_filter
+ add_filter es una función de WordPress que permite modificar el contenido de una variable antes de que se muestre en la pantalla.
+ este add_filter recibe dos parámetros: el nombre del filtro y el nombre de la función que se ejecutará cuando se llame al filtro.

### Que es add_action
Esta función es una fundamental en WordPress que se utiliza para asociar funciones o métodos a eventos específicos en el ciclo de vida del sistema. En otras palabras permite ejecutar código en respuesta a eventos específicos en WordPress.

En este plugin usamos uno el evento **plugins_loaded** para ejecutar las funciones **createTable** e **insertSignos** cuando se activa el plugin.

### Que función se ejecuta cuando se llama al filtro
+ La función que se ejecuta cuando se llama al filtro es la función `cambiar_signos`.
+ Esta funcion es el núcleo de este plugin. Su propósito es realizar cambios en el contenido de las entradas y páginas de WordPress, remplazando las palabras de los signos por sus respectivos signos.
    + La función examina el titulo y el contenido de una entrada o página en busca de las palabras definidas en la lista $listaDeshechos.
    + Cada vez que encuentra una de estas palabras, la reemplaza por su signo correspondiente, definido en la lista $listaQueridos.

### Uso en el Contexto de WordPress
+ La función se utiliza como filtro en el gancho the_content y the_title. Esto significa que se aplica al titulo y al contenido de las entradas y páginas cada vez que se muestra en el sitio.

