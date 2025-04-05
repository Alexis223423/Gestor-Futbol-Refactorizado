# 1. - Introducción

En este documento se va a explicar como se ha llevado a cabo el proceso de refactorización de un codigo en lenguaje Java usando la herramienta SonarQube.

# 2. - Pasos previos

## 2.1 - Creación del proyecto Java en Eclipse

![CrearProjectoJava](/Fotos/1%20-%20Creacion%20de%20JavaProject%20en%20eclipse.png)

## 2.2 - Creación del proyecto en SonarQube

![CrearProjectoSonarQube](/Fotos/2%20-%20Creacion%20de%20proyecto%20en%20sonarqube.png)

## 2.3 - Archivo sonar-project.properties

![ArchivoSonarProperties](/Fotos/3%20-%20Rellenando%20el%20archivo%20sonar-project.properties.png)

Este archivo contiene la configuración que hace falta para que se lleve a cabo el scaneo de SonarQube sobre el proyecto java que se quiere refactorizar, este archivo debe colocarse en el mismo directorio donde se ubique el archivo ".java".

>Referencias:
>>sonar.projectKey - Nombre del proyecto dentro de SonarQube.
>
>>sonar.host-url - URL local o remota con el puerto que usa SonarQube.
>
>>sonar.token - Token generado para ejecutar el análisis desde CMD.
>
>>sonar.languaje - Lenguaje en el que se va a interpretar al escanear.
>
>>sonar.sources - Ruta donde está el cdódigo a analizar.
>
>>sonar.java.binaries - Ruta de los archivos compilados del proyecto.

## 2.4 - Ubicación de la carpeta del proyecto

![UbicacionCarpeta](/Fotos/4%20-%20Ubicación%20del%20proyecto%20en%20CMD,%20compilado%20y%20lanzado%20del%20comando%20sonar-scanner.png)

Se accede desde CMD a la ruta donde se ubica el proyecto que se va a refactorizar y hay que compilar el proyecto usando el siguiente código:
~~~
for /r src %f in (*.java) do javac -d target\classes "%f"
~~~
Acto seguido hay que hacer el scaneo usando este comando:
~~~
sonar-scanner
~~~

## 2.5 -  Proyecto Java antes de refactorizar

![EscaneoPrincipal](/Fotos/5%20-%20Proyecto%20antes%20de%20refactorizar.png)

Al hacer el escaneo inicial se observa que tiene 31 errores los cuales se van a ir solucionando uno a uno.

# 3. - Refactorización

A continuación se van a nombrar todos los problemas que han surgido a raiz del escaneo, muchos de ellos son problemas similares o repetidos, por lo cual solo se mostraran una vez.

## 3.1 - Problema Nº 1

![Issue1](/Fotos/6%20-%20Issue%20System.out%20a%20logger.png)

Este mensaje indica que se está usando muchos System.out.println() para imprimir información en consola, lo cual no es una buena práctica en aplicaciones profesionales.

>Cambios que se realizan:
>>~~~
>>import java.util.logging.Logger;
>>~~~
>
>>~~~
>>private static Logger logger = Logger.getLogger(GestorFutbol.class.getName());
>>~~~
>
>>~~~
>>logger.info("Jugado como local.");
>>~~~

## 3.2 - Problema Nº 2

![Issue2](/Fotos/7%20-%20Issue%20static%20final%20or%20non-public.png)

Este mensaje indica que se está usando equipoNombre como public y debe ser "private", ademas de poder accederse por los métodos "getEquipoNombre()" y "setEquipoNombre()".

>Cambios que se realizan:
>>~~~
>>private String equipoNombre;
>>~~~
>
>>~~~
>>public String getEquipoNombre() {
>>		return equipoNombre;
>>	}
>>~~~
>
>>~~~
>>public void setEquipoNombre(String equipoNombre) {
>>		this.equipoNombre = equipoNombre;
>>	}
>>~~~

## 3.3 - Problema Nº 3 

![Issue3](/Fotos/8%20-%20Issue%20eliminar%20private%20field.png)

Este mensaje indica que el campo "private static String NOMBRE_REAL_MADRID" está declarado pero nunca se usa en el código. Si no se utiliza, hay que eliminarlo para mantener el código limpio.

>Estas son las lineas que se eliminan:
>
>>~~~
>>private static String NOMBRE_REAL_MADRID = "Real Madrid club de Fútbol";
>>~~~
>
>>~~~
>>private static String NOMBRE_ATLETICO_MADRID = "Atlético de Madrid";
>>~~~

## 3.4 - Problema Nº 4 

![Issue4](/Fotos/9%20-%20Issue%20simply%20return.png)

Este mensaje indica que devolver "return Integer.MIN_VALUE" no es un valor estándar en la implementación de compareTo() , por lo tanto es mas correcto usar "return -1"

>Codigo anterior:
>>~~~
>>@Override
>>public int compareTo(GestorFutbol otro) {
>>    if (this.equipoNombre == null || otro.equipoNombre == null) {
>>        return Integer.MIN_VALUE;
>>    }
>>    return this.equipoNombre.compareTo(otro.equipoNombre);
>>}
>>~~~
>
>Corrección:
>>~~~
>>@Override
>>public int compareTo(GestorFutbol otro) {
>>    if (this.equipoNombre == null || otro.equipoNombre == null) {
>>        return -1; 
>>    }
>>    return this.equipoNombre.compareTo(otro.equipoNombre);
>>}
>>~~~


## 3.5 - Problema Nº 5

![Issue5](/Fotos/10%20-%20Issue%20built-in.png)

Este mensaje indica que se está concatenando strings dentro de un logger.info(), y es mejor usar formato con logger.log().

>Codigo anterior:
>>~~~
>>logger.info("Victoria. Puntos acumulados: " + puntos);
>>~~~
>
>Corrección:
>>~~~
>>logger.log(Level.INFO,"Victoria. Puntos acumulados: {0}", puntos);
>>~~~

## 3.6 - Problema Nº 6 

![Issue6](/Fotos/11%20-%20Issue%20complejidad%20cognitiva.png)

Este mensaje indica que se está excediendo la complejidad cognitiva al anidar muchos if o usar muchos bucles, al estar el limite en 15 y haberlo superado teniendo una cantidad de 18, hay que bajarla haciendo metodos mas pequeños.

>Codigo anterior:
>>~~~
>>public void procesarTemporada(List<String> resultados) {
>>    for (String resultado : resultados) {
>>        if (resultado.equals("victoria")) {
>>            puntos += 3;
>>            System.out.println("Victoria. Puntos acumulados: " + puntos);
>>        } else if (resultado.equals("empate")) {
>>            puntos += 1;
>>            System.out.println("Empate. Puntos acumulados: " + puntos);
>>        } else if (resultado.equals("derrota")) {
>>            System.out.println("Derrota. Puntos acumulados: " + puntos);
>>        }
>>
>>        if (resultado.contains("local")) {
>>            System.out.println("Jugado como local.");
>>            if (resultado.length() > 10) {
>>                System.out.println("Detalle adicional: " + resultado);
>>            }
>>        } else if (resultado.contains("visitante")) {
>>            System.out.println("Jugado como visitante.");
>>            if (resultado.length() > 8) {
>>                System.out.println("Comentario: " + resultado);
>>            }
>>        }
>>
>>        switch (resultado.length()) {
>>            case 7:
>>                System.out.println("Resultado corto.");
>>                break;
>>            case 14:
>>                System.out.println("Resultado medio.");
>>                break;
>>            default:
>>                System.out.println("Resultado de longitud estándar.");
>>                break;
>>        }
>>
>>        if (resultado.endsWith("!")) {
>>            System.out.println("¡Resultado enfatizado!");
>>        }
>>
>>        System.out.println("----------------------");
>>    }
>>}
>>~~~
> Se ha reducido la complejidad en estos 3 métodos más pequeños:
>>~~~
>>sumapuntos(resultado);
>>tipopartido(resultado);
>>switchresultados(resultado);
>>~~~
>Quedando de esta manera:
>>~~~
>>private void tipopartido(String resultado) {
>>		if (resultado.contains("local")) {
>>			logger.log(Level.INFO,"Jugado como local.");
>>		    if (resultado.length() > 10) {
>>		    	logger.log(Level.INFO,"Detalle adicional: {0}", resultado);
>>		    }
>>		} else if (resultado.contains("visitante")) {
>>			logger.info("Jugado como visitante.");
>>		    if (resultado.length() > 8) {
>>		    	logger.log(Level.INFO,"Comentario: {0}", resultado);
>>		    }
>>		}
>>	}
>>
>>	private void sumapuntos(String resultado) {
>>		if (resultado.equals("victoria")) {
>>		    puntos += 3;
>>		    logger.log(Level.INFO,"Victoria. Puntos acumulados: {0}", puntos);
>>		} else if (resultado.equals("empate")) {
>>		    puntos += 1;
>>		    logger.log(Level.INFO,"Empate. Puntos acumulados: {0}", puntos);
>>		} else if (resultado.equals("derrota")) {
>>			logger.log(Level.INFO,"Derrota. Puntos acumulados: {0}", puntos);
>>		}
>>	}
>>
>>	private void switchresultados(String resultado) {
>>		switch (resultado.length()) {
>>		    case 7:
>>		    	logger.info("Resultado corto.");
>>		        break;
>>		    case 14:
>>		    	logger.info("Resultado medio.");
>>		        break;
>>		    default:
>>		    	logger.info("Resultado de longitud estándar.");
>>		        break;
>>		}
>>	}
>>~~~

## 3.7 - Problema Nº 7 

![Issue7](/Fotos/12%20-%20Issue%20equals(%20).png)

Este mensaje indica que si se sobrescribe el método equals() en una clase, como es en "GestorFutbol", tambien se debe sobreescribir hasCode(), ya que Java utiliza ambos métodos juntos.

>Codigo anterior:
>>~~~
>>@Override
>>    public boolean equals(Object obj) {
>>        if (!(obj instanceof GestorFutbol)) return false;
>>        GestorFutbol otro = (GestorFutbol) obj;
>>        return this.equipoNombre.equals(otro.equipoNombre);
>>    }
>>~~~
>Se le añade justo después la sobrescritura de hasCode() :
>>~~~
>>@Override
>>    public int hashCode() {
>>		return puntos;  
>>    }
>>~~~

## 3.8 - Problema Nº 8 

![Issue8](/Fotos/13%20-%20Issue%20clone.png)

Este mensaje indica que no es recomendable sobrescribir el método clone() de la interfaz Cloneable de Java. Puede producir objetos incompletos y resultar confuso para otros desarrolladores que lean el código, por eso se recomienda usar un constructor de copia.

>Codigo anterior:
>>~~~
>>public class GestorFutbol implements Cloneable, Comparable<GestorFutbol>
>>~~~
>
>>~~~
>>@Override
>>    public Object clone() throws CloneNotSupportedException {
>>        return super.clone();
>>    }
>>~~~
>Corrección con constructor de copia:
>
>>~~~
>>public class GestorFutbol implements Comparable<GestorFutbol>
>>~~~
>>
>>~~~
>>public GestorFutbol(GestorFutbol original) {
>>        this.equipoNombre = original.equipoNombre;
>>       
>>    }
>>~~~

## 3.9 - Proyecto refactorizado

![RefactorizacionCompletada1](/Fotos/15%20-%20Proyecto%20refactorizado.png)

![RefactorizacionCompletada2](/Fotos/14%20-%20Issues%20a%20cero.png)

Finalmente despues de todas las correciones del código, el escaner de SonarQube da como resultado 0 problemas, de este modo acaba la refactorización del proyecto.