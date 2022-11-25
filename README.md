# Curvas-epidemiologicas-con-R
Curvas-epidemiologicas-con-R es un script en R que busca brindar una manera no muy complicada y al español de construir algunos informes y gráficos epidemiológicos, principalmente curvas epidemiológicas. Una parte del script fué adaptado, simplificado y al españoldel del tutorial publicado en el portal "epirhandbook", en: https://epirhandbook.com/en/epidemic-curves.html#tentative-data.
##Construyendo un set de datos simulado
Se construirá un set de datos ficticios en el que se busca simular una serie de casos de los serotipos del dengue durante el 2022.
```
#importando/descargando librerías a utilizar
install.packages("pacman") #una vez instalada esta librería se puede eliminar la línea de comando

#importando e instalando las librarías a utilizas con la librería "pacman"
pacman::p_load(
  RColorBrewer, # paleta de colores para R de colorbrewer2.org
  dplyr,     # Manejo de datos
  ggplot2    #construcción gráficos
) 

#generando un set de datos ficticio
p1 = data.frame(Den = rep("D1", 200),
           fecha = seq(as.Date("2022-01-01"), as.Date("2022-05-25"), length = 100)) #Tabla con pocos casos D1 de marzo a mayo

p2 = data.frame(Den = rep(c("D2", "D3"), 300),
                fecha = seq(as.Date("2022-05-26"), as.Date("2022-06-26"), length = 600)) #Tabla con muchos casos D2 y D3 de mayo a junio

p3 = data.frame(Den = rep(c("D2", "D3"), 500),
                fecha = seq(as.Date("2022-06-27"), as.Date("2022-07-27"), length = 1000)) #Tabla con muchisimos casos D2 y D3 de junio a julio

p4 = data.frame(Den = rep(c("D2", "D3"), 250),
               fecha = seq(as.Date("2022-07-28"), as.Date("2022-08-30"), length = 500)) #Tabla con menos casos D2 y D3 de julio a agosto

p5 = data.frame(Den = rep("D3", 200),
                fecha = seq(as.Date("2022-08-30"), as.Date("2022-12-25"), length = 100)) #Tabla con pocos casos D3 de Agosto a Diciembre
  
data = rbind(p1,p2,p3,p4, p5) # uniendo todas las tablas para crear un solo set de datos
remove(p1, p2, p3, p4, p5) #eliminar las tablas creadas para construir la dataframe y que ya no servirán
head(data)

  Den      fecha
1  D1 2022-01-01
2  D1 2022-01-02
3  D1 2022-01-03
4  D1 2022-01-05
5  D1 2022-01-06
6  D1 2022-01-08
```

A partir de los datos, se puede generar un histograma simple para visualizar el curso del brote durante el año

```
ggplot(data = data)+
  geom_histogram(aes(x = fecha))
```
![image](https://user-images.githubusercontent.com/95062993/203907834-7f2e8427-7cd9-4bb5-966d-cec519929b14.png)

Con los datos se pueden crear informes simples agrupando el serotipo de dengue y la fecha en que aparecieron. Es probable reemplazar la variable "Den" que contiene la información del serotipo con otras columnas que contengan información de hospitales, sexo, ciudad, entre otros

```
conteo_de_datos <- data %>% 
  group_by(Den, fecha = as.character(fecha)) %>% 
  summarise(n_cases = dplyr::n()) %>% ungroup()

conteo_de_datos$fecha = as.Date( conteo_de_datos$fecha) #convirtiendo a tipo fecha la columna fecha
conteo_de_datos

# A tibble: 390 × 3
   Den   fecha      n_cases
   <chr> <date>       <int>
 1 D1    2022-01-01       2
 2 D1    2022-01-02       2
 3 D1    2022-01-03       2
 4 D1    2022-01-05       2
 5 D1    2022-01-06       2
 6 D1    2022-01-08       2
 7 D1    2022-01-09       2
 8 D1    2022-01-11       2
 9 D1    2022-01-12       2
10 D1    2022-01-14       2
# … with 380 more rows
```
Es probable agrupar los casos por serotipo y por semana

```
conteo_de_datos_semanal <- data 
conteo_de_datos_semanal$semana <- format(data$fecha, "%V")

conteo_de_datos_semanal %>%
  group_by(Den, fecha = as.character(semana)) %>% 
  summarise(n_cases = dplyr::n()) %>% ungroup()

# A tibble: 68 × 3
   Den   fecha n_cases
   <chr> <chr>   <int>
 1 D1    01         10
 2 D1    02          8
 3 D1    03         10
 4 D1    04         10
 5 D1    05         10
 6 D1    06         10
 7 D1    07         10
 8 D1    08          8
 9 D1    09         10
10 D1    10         10
# … with 58 more rows
```

También se pueden agrupar los casos por serotipo y por mes 

```
conteo_de_datos_mensual <- data 
conteo_de_datos_mensual$mes <- format(data$fecha, "%m")

conteo_de_datos_mensual %>%
  group_by(Den, fecha = as.character(mes)) %>% 
  summarise(n_cases = dplyr::n()) %>% ungroup()

# A tibble: 17 × 3
   Den   fecha n_cases
   <chr> <chr>   <int>
 1 D1    01         44
 2 D1    02         38
 3 D1    03         42
 4 D1    04         42
 5 D1    05         34
 6 D2    05         58
 7 D2    06        309
 8 D2    07        464
 9 D2    08        219
10 D3    05         58
11 D3    06        309
12 D3    07        463
13 D3    08        224
14 D3    09         52
15 D3    10         52
16 D3    11         50
17 D3    12         42
```

```

```
