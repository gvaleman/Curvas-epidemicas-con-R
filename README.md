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

```

```
