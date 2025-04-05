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

![CrearProjectoJava](/Fotos/4%20-%20Ubicación%20del%20proyecto%20en%20CMD,%20compilado%20y%20lanzado%20del%20comando%20sonar-scanner.png)

Se accede desde CMD a la ruta donde se ubica el proyecto que se va a refactorizar y hay que compilar el proyecto usando el siguiente código:
~~~
for /r src %f in (*.java) do javac -d target\classes "%f"
~~~
Acto seguido hay que hacer el scaneo usando este comando:
~~~
sonar-scanner
~~~

## Proyecto Java antes de refactorizar

![CrearProjectoJava](/Fotos/5%20-%20Proyecto%20antes%20de%20refactorizar.png)

El proyecto inicialmente tiene 31 errores los cuales se van a ir solucionando uno a uno.

## Reemplazar el uso de "System.out.print" por "logger.info"

![CrearProjectoJava](/Fotos/6%20-%20Issue%20System.out%20a%20logger.png)

~~~
import java.util.logging.Logger;
~~~

~~~
private static Logger logger = Logger.getLogger(GestorFutbol.class.getName());
~~~

~~~
logger.info("Jugado como local.");
~~~

## Reemplazar el uso de "public" en "equipoNombre" 

![CrearProjectoJava](/Fotos/7%20-%20Issue%20static%20final%20or%20non-public.png)

~~~
private String equipoNombre;
~~~

~~~
public String getEquipoNombre() {
		return equipoNombre;
	}
~~~

~~~
public void setEquipoNombre(String equipoNombre) {
		this.equipoNombre = equipoNombre;
	}
~~~

## Reemplazar el uso de "public" en "equipoNombre" 

![CrearProjectoJava](/Fotos/8%20-%20Issue%20eliminar%20private%20field.png)

Solo eliminar las dos lineas siguientes:

~~~
private static String NOMBRE_REAL_MADRID = "Real Madrid club de Fútbol";
~~~

~~~
private static String NOMBRE_ATLETICO_MADRID = "Atlético de Madrid";
~~~

## Reemplazar "return Integer.MIN_VALUE" por "return -1" 

![CrearProjectoJava](/Fotos/9%20-%20Issue%20simply%20return.png)

~~~
return Integer.MIN_VALUE;
~~~
se cambia por:
~~~
return -1;
~~~


## Reemplazar "return Integer.MIN_VALUE" por "return -1" 

![CrearProjectoJava](/Fotos/10%20-%20Issue%20built-in.png)

~~~
logger.info("Victoria. Puntos acumulados: " + puntos);
~~~
se cambia por:
~~~
logger.log(Level.INFO,"Victoria. Puntos acumulados: {0}", puntos);
~~~

## Conseguir que la complejidad cognitiva sea mas baja" 

![CrearProjectoJava](/Fotos/11%20-%20Issue%20complejidad%20cognitiva.png)

Antes era asi:
~~~
switch (resultado.length()) {
                case 7:
                	logger.info("Resultado corto.");
                    break;
                case 14:
                	logger.info("Resultado medio.");
                    break;
                default:
                	logger.info("Resultado de longitud estándar.");
                    break;
            }
~~~
Se resume en este metodo:
~~~
switch_resultados(resultado);
~~~
Y queda asi:
~~~
private void switch_resultados(String resultado) {
		switch (resultado.length()) {
		    case 7:
		    	logger.info("Resultado corto.");
		        break;
		    case 14:
		    	logger.info("Resultado medio.");
		        break;
		    default:
		    	logger.info("Resultado de longitud estándar.");
		        break;
		}
	}
~~~

## Equals" 

![CrearProjectoJava](/Fotos/12%20-%20Issue%20equals(%20).png)

~~~
 @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof GestorFutbol)) return false;
        GestorFutbol otro = (GestorFutbol) obj;
        return this.equipoNombre.equals(otro.equipoNombre);
    }
~~~
añadirle abajo esto
~~~
 @Override
    public int hashCode() {
		return puntos;
        
    }
~~~

## Clone 

![CrearProjectoJava](/Fotos/13%20-%20Issue%20clone.png)


yo tenia
~~~
public class GestorFutbol implements Cloneable, Comparable<GestorFutbol>
~~~

~~~
@Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
~~~

y ahora tengo

~~~
public class GestorFutbol implements Comparable<GestorFutbol>
~~~

~~~
public GestorFutbol(GestorFutbol original) {
        this.equipoNombre = original.equipoNombre;
       
    }
~~~

## Proyecto refactorizado

![CrearProjectoJava](/Fotos/15%20-%20Proyecto%20refactorizado.png)

![CrearProjectoJava](/Fotos/14%20-%20Issues%20a%20cero.png)