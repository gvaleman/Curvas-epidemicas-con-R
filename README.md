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
  summarise(n_cases = dplyr::n())

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
  group_by(Den, semana = as.character(semana)) %>% 
  summarise(n_cases = dplyr::n())
  
# A tibble: 68 × 3
   Den   semana n_cases
   <chr> <chr>    <int>
 1 D1    01          10
 2 D1    02           8
 3 D1    03          10
 4 D1    04          10
 5 D1    05          10
 6 D1    06          10
 7 D1    07          10
 8 D1    08           8
 9 D1    09          10
10 D1    10          10
# … with 58 more rows
```

También se pueden agrupar los casos por serotipo y por mes 

```
conteo_de_datos_mensual <- data 
conteo_de_datos_mensual$mes <- format(data$fecha, "%m")

conteo_de_datos_mensual %>%
  group_by(Den, mes = as.character(mes)) %>% 
  summarise(n_cases = dplyr::n())
  
# A tibble: 17 × 3
# Groups:   Den [3]
   Den   mes   n_cases
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
Con ggplot2 se pueden crear curvas epidemiológicas muy atractivas y bastante personalizables. A continuación se construye una curva a partir de la tabla "data"

```
ggplot(data = data) +          # set de datos
  geom_histogram(                      # tipo histograma
    mapping = aes(x = fecha),     # fecha al eje x
    binwidth = 1)+                     # casos por día
  labs(title = "Casos diarios DENGUE")                # título
  
```

![image](https://user-images.githubusercontent.com/95062993/204058161-c0311c46-6a67-46ed-8116-0c63f11d4e89.png)


```
# semanal
ggplot(data = data) +          # configuramos el set de datos 
  geom_histogram(                      # tipo histograma
    mapping = aes(x = fecha),   # fecha al eje x
    binwidth = 7)+                   # casos por cada 7 días
  labs(title = "Casos semanales DENGUE") # título

```

![image](https://user-images.githubusercontent.com/95062993/204058218-cff165ce-f603-4023-ae4c-b73b8b0238a7.png)

```
# mensual 
ggplot(data = data) +          # configuramos el set de datos 
  geom_histogram(                      # tipo histograma
    mapping = aes(x = fecha),   # fecha al eje x
    binwidth = 30.41)+                   # casos cada 30.41 día (mensual)
  labs(title = "Casos Mensuales DENGUE") # Título
  
 
```

![image](https://user-images.githubusercontent.com/95062993/204058245-c1d45c94-d4ed-42f0-95f5-43fe5e9655bb.png)

De la siguiente manera podemos obtener la fecha del primer y del último caso en nuestro set de datos

```

#### DETERMINAR EL PRIMER CASO
format(min(data$fecha, na.rm=T), "%A %d %b, %Y")
#### DETERMINAR EL ÚLTIMO CASO
format(max(data$fecha, na.rm=T), "%A %d %b, %Y")
```


Graficamos una curva epidémica más personalizada y visualmente más atractiva

```
# TOTALIZAR DATOS DE MANERA SEMANAL (INICIANDO LUNES) Y CREANDO CURVA EPIDEMIOLÓGICA
#############################
# Definir una secuencia semanal (7 días)
secuencia_semanal <- seq.Date(
  from = floor_date(min(data$fecha, na.rm=T),  "week", week_start = 1), # Lunes antes del primer caso
  to   = ceiling_date(max(data$fecha, na.rm=T), "week", week_start = 1), # Lunes después del primer caso
  by   = "week")    # los bloques son de 7 días

ggplot(data = data) + #llamamos a la función ggplot y especificamos la tabla a utilizar

  # hacer un histograma: especificar los puntos de ruptura: comienza el lunes anterior al primer caso, finaliza el lunes posterior al último caso
  geom_histogram(
    
    # mapping aesthetics
   mapping = aes(x = fecha),  # fecha al eje X
    
    # especificar los puntos de quiebre semanal, definido previamente
    breaks = secuencia_semanal, 
    
    closed = "left",  # contar casos desde el inicio del punto de interrupción
    
    # bars
    color = "darkblue",     # color de la linea que bordea las barras
    fill = "lightblue"      # color de las barras
  )+ 
  
  # x-axis labels
  scale_x_date(
    expand            = c(0,0),           # elimine el espacio en el eje x antes y después de las barras
    date_breaks       = "4 weeks",        # las etiquetas de fecha y las principales líneas de cuadrícula verticales aparecen cada 3 lunes de semana
    date_minor_breaks = "week",           # líneas verticales menores aparecen todos los lunes de la semana
    date_labels       = "%a\n%d %b\n%Y")+ # formato de las etiquetas de fecha
  
  # y-axis
  scale_y_continuous(
    expand = c(0,0))+             # eliminar el exceso de espacio en el eje y por debajo de 0 (alinee el histograma con el eje x)
  
  # aesthetic themes
  theme_minimal()+                # aplicar un tema que simplifique el fondo
  
  theme(
    plot.caption = element_text(hjust = 0,        # ubicar titulos a la izquierda
                                face = "italic"), # Titulos en letra itálica
    axis.title = element_text(face = "bold"))+    # Títulos en los ejes en Negrita
  
  # etiquetas que incluyen subtítulos
  labs(
    title    = "Incidencia semanal de casos (semana iniciando lunes)",
    subtitle = "Tenga en cuenta la alineación de las barras, las líneas de cuadrícula verticales y las etiquetas de los ejes los lunes de la semana",
    x        = "Semana de inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))

```
![image](https://user-images.githubusercontent.com/95062993/203910029-f2cb9b4f-b3c4-4e42-8191-5885162937af.png)

La curva epidemiológica anterior fué creada en semanas que inician el día LUNES, si se dea construir la curva con semanas que inicien en DOMINGO se debe reemplazar la sección "scale_x_date" con el código a continuación:

```
scale_x_date(
  expand = c(0,0),

  # especificar el intervalo de las etiquetas de fecha y las principales líneas de cuadrícula verticales
  breaks = seq.Date(
    from = floor_date(min(data$fecha, na.rm=T),   "week", week_start = 7), # Domingo ántes del primer caso
    to   = ceiling_date(max(data$fecha, na.rm=T), "week", week_start = 7), # Domingo después del último caso
    by   = "4 weeks"),
  
  # especificar el intervalo de la línea de cuadrícula vertical menor
  minor_breaks = seq.Date(
    from = floor_date(min(data$fecha, na.rm=T),   "week", week_start = 7), # Domingo ántes del primer caso
    to   = ceiling_date(max(data$fecha, na.rm=T), "week", week_start = 7), # Domingo después del último caso
    by   = "week"),
  
  # date label format
  date_labels = "%a\n%d %b\n%Y")        # día, encima de la abreviatura del mes, encima del año de 2 dígitos

```

Es útil visualizar una curva epidemiológica con datos agrupados. Los datos que se pueden agrupar pueden ser el sexo, la procedencia (Municipios, provicia), alguna condición, grupos etarios, entre otros. En esta ocación se construirá una curva epidemiológica agrupada por los serotipos de dengue

```
ggplot(data = data) + #llamamos a la función ggplot y especificamos la tabla a utilizar
  
  # hacer un histograma: especificar los puntos de ruptura: comienza el lunes anterior al primer caso, finaliza el lunes posterior al último caso
  geom_histogram(
    
    # mapping aesthetics
    mapping = aes(x = fecha, group = Den, fill = Den),
    
    # especificar los puntos de quiebre semanal, definido previamente
    breaks = secuencia_semanal, 
    closed = "left",
    alpha=0.7 #Configura la transparencia de las barras 0 es sin color, 1 es full color
  )+ 
  
  scale_fill_manual(values=c("#a0e77d", "#82b6d9", "#ef8677")) + #configuramos los colores de cada barra
  
  # x-axis labels
  scale_x_date(
    expand            = c(0,0),           # elimine el espacio en el eje x antes y después de las barras
    date_breaks       = "4 weeks",        # las etiquetas de fecha y las principales líneas de cuadrícula verticales aparecen cada 3 lunes de semana
    date_minor_breaks = "week",           # líneas verticales menores aparecen todos los lunes de la semana
    date_labels       = "%a\n%d %b\n%Y")+ # formato de las etiquetas de fecha
  
  # y-axis
  scale_y_continuous(
    expand = c(0,0))+             # eliminar el exceso de espacio en el eje y por debajo de 0 (alinee el histograma con el eje x)
  
  # aesthetic themes
  theme_minimal()+                # aplicar un tema que simplifique el fondo
  
  theme(
    plot.caption = element_text(hjust = 0,        # ubicar titulos a la izquierda
                                face = "italic"), # Titulos en letra itálica
    axis.title = element_text(face = "bold"),
    legend.position = "top")+    # Títulos en los ejes en Negrita
  
  # etiquetas que incluyen subtítulos
  labs(
    title    = "Incidencia semanal de casos (semana iniciando lunes)",
    subtitle = "Tenga en cuenta la alineación de las barras, las líneas de cuadrícula verticales y las etiquetas de los ejes los lunes de la semana",
    x        = "Semana de inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    fill     = " ", #Que la leyenda no tenga títulos
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))
```
![image](https://user-images.githubusercontent.com/95062993/203910970-826aa3cd-6061-4fff-813c-846ef7bbcb8d.png)

A continuación se construye la curva epidemiológica trazada con una línea (en vez de barras como en los ejemplos anteriores)

```
ggplot(data = data) + #llamamos a la función ggplot y especificamos la tabla a utilizar
 
  # hacer un histograma: especificar los puntos de ruptura: comienza el lunes anterior al primer caso, finaliza el lunes posterior al último caso
  geom_freqpoly(binwidth = 1, size= 1.5,
    
    # mapping aesthetics
    mapping = aes(x = fecha),
    
    # especificar los puntos de quiebre semanal, definido previamente
    breaks = secuencia_semanal, 
    closed = "left",
    alpha=0.7 #Configura la transparencia de las barras 0 es sin color, 1 es full color
  )+ 
  
  scale_fill_manual(values=c("#a0e77d", "#82b6d9", "#ef8677")) + #configuramos los colores de cada barra
  
  # x-axis labels
  scale_x_date(
    expand            = c(0,0),           # elimine el espacio en el eje x antes y después de las barras
    date_breaks       = "4 weeks",        # las etiquetas de fecha y las principales líneas de cuadrícula verticales aparecen cada 3 lunes de semana
    date_minor_breaks = "week",           # líneas verticales menores aparecen todos los lunes de la semana
    date_labels       = "%a\n%d %b\n%Y")+ # formato de las etiquetas de fecha
  
  # y-axis
  scale_y_continuous(
    expand = c(0,0))+             # eliminar el exceso de espacio en el eje y por debajo de 0 (alinee el histograma con el eje x)
  
  # aesthetic themes
  theme_minimal()+                # aplicar un tema que simplifique el fondo
  
  theme(
    plot.caption = element_text(hjust = 0,        # ubicar titulos a la izquierda
                                face = "italic"), # Titulos en letra itálica
    axis.title = element_text(face = "bold"),
    legend.position = "top")+    # Títulos en los ejes en Negrita
  
  # etiquetas que incluyen subtítulos
  labs(
    title    = "Incidencia semanal de casos (semana iniciando lunes)",
    subtitle = "Tenga en cuenta la alineación de las barras, las líneas de cuadrícula verticales y las etiquetas de los ejes los lunes de la semana",
    x        = "Semana de inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))
```
![image](https://user-images.githubusercontent.com/95062993/203912618-8e93aa1f-bc06-4b31-9b69-a45fdc7c896b.png)

Ahora trazamos una curva epidémica trazada con líneas, pero agrupando las líneas por grupo/color
```
ggplot(data = data) + #llamamos a la función ggplot y especificamos la tabla a utilizar
 
  # hacer un histograma: especificar los puntos de ruptura: comienza el lunes anterior al primer caso, finaliza el lunes posterior al último caso
  geom_freqpoly(binwidth = 1, size= 1.5,
    
    # mapping aesthetics
    mapping = aes(x = fecha,),
    
    # especificar los puntos de quiebre semanal, definido previamente
    breaks = secuencia_semanal, 
    closed = "left",
    alpha=0.7 #Configura la transparencia de las barras 0 es sin color, 1 es full color
  )+ 
  
  scale_fill_manual(values=c("#a0e77d", "#82b6d9", "#ef8677")) + #configuramos los colores de cada barra
  
  # x-axis labels
  scale_x_date(
    expand            = c(0,0),           # elimine el espacio en el eje x antes y después de las barras
    date_breaks       = "4 weeks",        # las etiquetas de fecha y las principales líneas de cuadrícula verticales aparecen cada 3 lunes de semana
    date_minor_breaks = "week",           # líneas verticales menores aparecen todos los lunes de la semana
    date_labels       = "%a\n%d %b\n%Y")+ # formato de las etiquetas de fecha
  
  # y-axis
  scale_y_continuous(
    expand = c(0,0))+             # eliminar el exceso de espacio en el eje y por debajo de 0 (alinee el histograma con el eje x)
  
  # aesthetic themes
  theme_minimal()+                # aplicar un tema que simplifique el fondo
  
  theme(
    plot.caption = element_text(hjust = 0,        # ubicar titulos a la izquierda
                                face = "italic"), # Titulos en letra itálica
    axis.title = element_text(face = "bold"),
    legend.position = "top")+    # Títulos en los ejes en Negrita
  
  # etiquetas que incluyen subtítulos
  labs(
    title    = "Incidencia semanal de casos (semana iniciando lunes)",
    subtitle = "Tenga en cuenta la alineación de las barras, las líneas de cuadrícula verticales y las etiquetas de los ejes los lunes de la semana",
    x        = "Semana de inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))

#Curva con lineas agrupadas
ggplot(data = data) + #llamamos a la función ggplot y especificamos la tabla a utilizar
  
  # hacer un histograma: especificar los puntos de ruptura: comienza el lunes anterior al primer caso, finaliza el lunes posterior al último caso
  geom_freqpoly(binwidth = 0.5, size= 1.5,
                
                # mapping aesthetics
                mapping = aes(x = fecha, colour = Den),
                
                # especificar los puntos de quiebre semanal, definido previamente
                breaks = secuencia_semanal, 
                closed = "left",
                alpha=0.7 #Configura la transparencia de las barras 0 es sin color, 1 es full color
  )+ 
  
  scale_fill_manual(values=c("#a0e77d", "#82b6d9", "#ef8677")) + #configuramos los colores de cada barra
  
  # x-axis labels
  scale_x_date(
    expand            = c(0,0),           # elimine el espacio en el eje x antes y después de las barras
    date_breaks       = "4 weeks",        # las etiquetas de fecha y las principales líneas de cuadrícula verticales aparecen cada 3 lunes de semana
    date_minor_breaks = "week",           # líneas verticales menores aparecen todos los lunes de la semana
    date_labels       = "%a\n%d %b\n%Y")+ # formato de las etiquetas de fecha
  
  # y-axis
  scale_y_continuous(
    expand = c(0,0))+             # eliminar el exceso de espacio en el eje y por debajo de 0 (alinee el histograma con el eje x)
  
  # aesthetic themes
  theme_minimal()+                # aplicar un tema que simplifique el fondo
  
  theme(
    plot.caption = element_text(hjust = 0,        # ubicar titulos a la izquierda
                                face = "italic"), # Titulos en letra itálica
    axis.title = element_text(face = "bold"),
    legend.position = "top")+    # Títulos en los ejes en Negrita
  
  # etiquetas que incluyen subtítulos
  labs(
    title    = "Incidencia semanal de casos (semana iniciando lunes)",
    subtitle = "Tenga en cuenta la alineación de las barras, las líneas de cuadrícula verticales y las etiquetas de los ejes los lunes de la semana",
    x        = "Semana de inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    colour = " ",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))

```
![image](https://user-images.githubusercontent.com/95062993/204058810-8747707d-0aa6-42ff-88a5-5bbf7807592d.png)


Construimos un gráfico con diferentes facetas o bloques. Los bloques proceden de la columna "Den" que contiene los serotipos de Dengue. A como se mencionó anteriormente, el agrupamiento puede realizarse por los grupos que interesen según el set de datos que se esté utilizando o la información que se cuente. Los grupos pueden ser Grupos etarios, sexo, provincias, hospital de procedencia, etc. 
NOTA: Para evitar ser redundante, el siguiente código no contiene comentarios. Se espera que a este punto se sepa la función que realiza cada argumento del código

```
ggplot(data) + 
  
  geom_histogram(
    mapping = aes(
      x = fecha,
      group = Den,
      fill = Den),  
    color = "black",  
    breaks = secuencia_semanal,
    closed = "left" ) +  
  
  scale_x_date(
    expand            = c(0,0),        
    date_breaks       = "2 months",    
    date_minor_breaks = "1 month",     
    date_labels       = "%b\n'%y") + 
  
  scale_y_continuous(expand = c(0,0)) +
  
  theme_minimal() +
  
  theme(
    plot.caption = element_text(face = "italic", hjust = 0), 
    axis.title = element_text(face = "bold"),
    legend.position = "bottom",
    strip.text = element_text(face = "bold", size = 10),
    strip.background = element_rect(fill = "grey")) +
  
  # En esta sección se especifica que se desean hacer varias facetas a partir de la columna Den
  facet_wrap(
    ~Den,
    ncol = 3,
    strip.position = "top" ) +             
  
  # labels
  labs(
    title    = "Incidencia semanal de los casos de Dengue",
    subtitle = "Las semanas inician en Lunes",
    fill     = "Serotipos",                                   
    x        = "Semanas de el inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))

```
![image](https://user-images.githubusercontent.com/95062993/204059779-3aa3b85c-04c0-4d8d-bf9d-09ab94e81102.png)

Curva epidmémica total en el fondo de la faceta.

Para mostrar la curva epidémica total en el fondo de cada faceta, agregue la función gghighlight() con paréntesis vacíos al ggplot. Esto es del paquete gghighlight. Tenga en cuenta que el máximo del eje y en todas las facetas ahora se basa en el pico de toda la epidemia. Hay más ejemplos de este paquete en la página de consejos de ggplot.
```
ggplot(data) + 
  
  geom_histogram(
    mapping = aes(
      x = fecha,
      group = Den,
      fill = Den),  
    color = "black",  
    breaks = secuencia_semanal,
    closed = "left" ) +  

    gghighlight::gghighlight()+ #este argumento/función indica graficar todos los casos
  
  scale_x_date(
    expand            = c(0,0),        
    date_breaks       = "2 months",    
    date_minor_breaks = "1 month",     
    date_labels       = "%b\n'%y") + 
  
  scale_y_continuous(expand = c(0,0)) +
  
  theme_minimal() +
  
  theme(
    plot.caption = element_text(face = "italic", hjust = 0), 
    axis.title = element_text(face = "bold"),
    legend.position = "bottom",
    strip.text = element_text(face = "bold", size = 10),
    strip.background = element_rect(fill = "grey")) +
  
  # En esta sección se especifica que se desean hacer varias facetas a partir de la columna Den
  facet_wrap(
    ~Den,
    ncol = 3,
    strip.position = "top" ) +             
  
  labs(
    title    = "Incidencia semanal de los casos de Dengue",
    subtitle = "Las semanas inician en Lunes",
    fill     = "Serotipos",                                   
    x        = "Semanas de el inicio de síntomas",
    y        = "Incidencia de casos reportados por semana",
    caption  = stringr::str_glue("n = {nrow(data)} de la tabla data; Los inicios de los casos van desde {format(min(data$fecha, na.rm=T), format = '%a %d %b %Y')} to {format(max(data$fecha, na.rm=T), format = '%a %d %b %Y')}\n{nrow(data %>% filter(is.na(fecha)))} casos a los que les falta la fecha de inicio y no se muestran"))

```
![image](https://user-images.githubusercontent.com/95062993/204060540-20ab5cd5-0e44-461f-ae4f-7c65a2a388f7.png)

A continuación se construirá una curva de casos acumulativa.
Reto: EL código contiene las instrucciones para generar la curva de casos acumulativo. Intente personalizarla (Configurar títulos, subtítulos, temas, etc)

Primero, debemos construir a partir de nuestro datos (data) un set de datos con los casos acumulativos por fecha
```
conteo_acumulativo <- data %>% 
  count(fecha) %>%                # count of rows per day (returned in column "n")   
  mutate(                         
    casos_acumulativo = cumsum(n)       # new column of the cumulative number of rows at each date
  )

head(conteo_acumulativo)

       fecha n casos_acumulativo
1 2022-01-01 2                 2
2 2022-01-02 2                 4
3 2022-01-03 2                 6
4 2022-01-05 2                 8
5 2022-01-06 2                10
6 2022-01-08 2                12
```

Construímos nuestra curva de datos acumulativos con el set de datos construído anteriormente (conteo_acumulativo)
```
#graficando los casos acumulativos
ggplot()+
  geom_line(
    data = conteo_acumulativo,
    aes(x = fecha, y = casos_acumulativo ),
    size = 2,
    color = "blue")
```
![image](https://user-images.githubusercontent.com/95062993/204061509-12fb9098-c6bd-4591-b7ce-159bdc7eff14.png)


