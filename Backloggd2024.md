~~~
<meta property="og:title" content="Backloggd 2024">
<meta property="og:description" content="Resumen anual de todos los videojuegos que jugué en el año.">
<meta property="og:image" content="https://gmedia.playstation.com/is/image/SIEPDC/helldivers-2-screenshot-disclaimer-02-en-24may23?$2400px$">
<meta property="og:url" content="https://elfofrit.com/Backloggd2024">
<meta property="og:type" content="article">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Backloggd 2024">
<meta name="twitter:description" content="Resumen anual de todos los videojuegos que jugué en el año.">
<meta name="twitter:image" content="https://elfofrit.com/Backloggd2024">
~~~

+++
title = "Backloggd 2024"
hascode = true
date = Date(2025, 4, 7)
rss = "Resumen anual de todos los videojuegos que jugué en el año."

tags = ["data_science", "hobbies"]

description = "Helldivers II."
image = "https://gmedia.playstation.com/is/image/SIEPDC/helldivers-2-screenshot-disclaimer-02-en-24may23"
+++

~~~
<p style="font-weight:600; font-size:36px">Backloggd 2024</p>
~~~

Realicé esto como ejercicio final del módulo de visualización de datos de un diplomado. Utilicé R porque fue el lenguaje enseñado en el módulo, y porque no lo he usado tanto como Python. Fue una buena forma de conocer RStudio. Me gustó, quisiera trabajar más con R.

El archivo RMarkodown original, su salida en HTML, el CSV, y las gráficas se pueden encontrar en [GitHub](https://github.com/elfofrit/backloggd2024).

\toc

# Introducción

Cada vez es más común que cada fin de año las principales plataformas de entretenimiento compartan estadísticas de uso como celebración o resumen anual. Creo que la más popular es el Rewind de Spotify.

La industria de videojuegos también está adoptando está tendencia. El problema es que en cada resumen solo puede verse los datos de juego de cada plataforma por separado, nunca en conjunto. Asimismo, estos resúmenes solo están disponibles para plataformas vigentes con conexión a internet, así que se descarta la información de consolas retro.

Backloggd es un sitio web donde es posible llevar un registro de las sesiones de juego de cada día, sin importar la plataforma, siempre y cuando el título se encuentre en la base de datos. También permite compartir reseñas y listas.

Durante 2024 registré mis sesiones de juego en Backloggd con el fin de extraer los datos para generar un verdadero resumen global de todos los videojuegos que llegué a jugar en ese año, sin importar en que launcher de PC o consola.

# Obtención de datos

De momento Backloggd no cuenta con una función de exportar tus datos en un CSV, por lo que descargué el HTML de las páginas (12 en total) del Journal del 2024 y utilicé el siguiente script de Python para extraer los datos de juego (fecha, título, duración) en un CSV.

~~~
<details>
  <summary>Mostrar código</summary>

  <pre><code class="language-python">
import csv
from bs4 import BeautifulSoup

# Abrir y leer el contenido del archivo HTML
with open('00.html', 'r', encoding='utf-8') as f:
    soup = BeautifulSoup(f, 'html.parser')

# Variables para almacenar los datos extraídos
entries = []
current_month_year = ""

# Iterar sobre cada bloque de entrada del diario
for journal_entry in soup.find_all('div', class_='journal_entry'):
    # Actualizar el mes y año si el bloque contiene el encabezado
    header = journal_entry.find('div', class_='month-year-date')
    if header:
        header_text = header.get_text(strip=True)
        if header_text:
            current_month_year = header_text  # Ejemplo: "December, 2024"

    # Buscar cada resultado individual dentro del bloque
    for result in journal_entry.find_all('div', class_='result'):
        # Extraer el día de la fecha
        day_tag = result.find('h4', class_='date-day')
        day = day_tag.get_text(strip=True) if day_tag else ""
        # Por si el día está vacío o marcado como "invisible"
        day_text = day if day and "invisible" not in day_tag.get('class', []) else ""

        # Extraer nombre
        game_tag = result.find('div', class_='game-name')
        game_name = game_tag.get_text(strip=True) if game_tag else ""

        # Extraer la duración de la sesión
        time_tag = result.find('p', class_='journal-time')
        duration = time_tag.get_text(strip=True) if time_tag else ""

        # Combinar el día con el encabezado de mes/año
        full_date = f"{day_text} {current_month_year}".strip()

        # Agregar los datos a la lista
        entries.append({
            'fecha': full_date,
            'nombre': game_name,
            'duracion': duration
        })

# Guardar la información extraída en un archivo CSV
with open('journal.csv', 'w', newline='', encoding='utf-8') as csvfile:
    fieldnames = ['fecha', 'nombre', 'duracion']
    writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

    writer.writeheader()
    for entry in entries:
        writer.writerow(entry)

print("Datos exportados exitosamente a journal.csv")

from google.colab import files
files.download('journal.csv')
  </code></pre>
</details>
~~~

En la quinta línea sustituí `00.html` por el nombre correcto del archivo. Las últimas dos líneas es porque lo ejecuté en Google Colab y quería que descargara automáticamente el CSV en lugar de guardarlo en la sesión de Colab.

Obtuve doce CSVs con la información de todas mis sesiones de juego registradas en Backloggd durante 2024. Cada CSV lucía así:

```
## # A tibble: 6 × 3
##   fecha             nombre                                            duracion
##   <chr>             <chr>                                             <chr>   
## 1 31 December, 2024 Pokémon Trading Card Game Pocket                  0h 9m   
## 2 31 December, 2024 Dragon Quest VII: Fragments of the Forgotten Past 0h 55m  
## 3 29 December, 2024 Lumines Remastered                                0h 44m  
## 4 29 December, 2024 Pokémon Trading Card Game Pocket                  0h 13m  
## 5 28 December, 2024 Dragon Quest VII: Fragments of the Forgotten Past 0h 45m  
## 6 28 December, 2024 Higurashi no Naku Koro ni: Ch.1 Onikakushi-hen    0h 46m
```

# Tratamiento de datos

Se ajustará el formato de fecha de la columna `fecha` de los CSV.

```r
library(tidyverse)
library(lubridate)

# Convertir columna "fecha" a Date
journal <- journal %>%
  mutate(fecha = dmy(fecha))  # Día-Mes-Año
```

Los datos de tiempo en `duracion` están en horas y minutos, se añadirá otra columna donde se convertirán a solo minutos para facilitar los cálculos más adelante.

```r
# Función para convertir duración tipo "1h 30m" a minutos
parse_duration <- function(x) {
  h <- as.numeric(str_extract(x, "\\d+(?=h)"))
  m <- as.numeric(str_extract(x, "\\d+(?=m)"))
  
  h[is.na(h)] <- 0
  m[is.na(m)] <- 0
  
  return(h * 60 + m)
}

# Aplicar la función
journal <- journal %>%
  mutate(duracion_min = parse_duration(duracion))

# Verifica el resultado
head(journal)
```

```
## # A tibble: 6 × 4
##   fecha      nombre                                        duracion duracion_min
##   <date>     <chr>                                         <chr>           <dbl>
## 1 2024-12-31 Pokémon Trading Card Game Pocket              0h 9m               9
## 2 2024-12-31 Dragon Quest VII: Fragments of the Forgotten… 0h 55m             55
## 3 2024-12-29 Lumines Remastered                            0h 44m             44
## 4 2024-12-29 Pokémon Trading Card Game Pocket              0h 13m             13
## 5 2024-12-28 Dragon Quest VII: Fragments of the Forgotten… 0h 45m             45
## 6 2024-12-28 Higurashi no Naku Koro ni: Ch.1 Onikakushi-h… 0h 46m             46
```

# Gráficos

Recomiendo hacer click derecho y abrir la imagen en una nueva pestaña. Todavía no logro domar esta plantilla con mi propio código HTML que ignore los márgenes y muestre mejor las imágenes. Una disculpa.

## Top 10 más jugados

~~~
<details>
  <summary>Mostrar código</summary>

  <pre><code class="language-r">
library(tidyverse)
# Calcular top 10
top10 <- journal %>%
  group_by(nombre) %>%
  summarise(minutos = sum(duracion_min, na.rm = TRUE)) %>%
  arrange(desc(minutos)) %>%
  slice_head(n = 10) %>%
  mutate(
    horas = floor(minutos / 60),
    mins = minutos %% 60,
    etiqueta = sprintf("%dh %02dm", horas, mins)
  )

# Gráfica
ggplot(top10, aes(x = reorder(nombre, minutos), y = minutos)) +
  geom_col(fill = "red") +
  geom_text(
    aes(label = etiqueta),
    hjust = -0.05,
    size = 4
  ) +
  coord_flip() +
  scale_y_continuous(expand = expansion(mult = c(0, 0.15))) +
  labs(
    title = "Videojuegos 2024",
    x = "",
    y = "Minutos"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(face = "bold", size = 18, hjust = 0.5),
    axis.title = element_text(size = 12),
    axis.text = element_text(size = 10),
    panel.grid.major.y = element_blank()
  )
  </code></pre>
</details>
~~~

~~~
<img src="/assets/videojuegos2024.png" />
~~~

Helldivers II es genial. Hacía mucho que no me divertía tanto con un shooter, y que mejor que con amigos.

Civilization VI es mi videojuego favorito. ¿Les he dicho alguna vez que Sid Meier himself respondió a mi pregunta en un evento de Zoom? El mejor día de mi vida.

Me sorprende que Balatro se haya colado al Top 3. Superó a los JRPGs que dejé inconclusos. Wow. Jueguen Balatro.

## Tiempo de juego por mes

~~~
<details>
  <summary>Mostrar código</summary>

  <pre><code class="language-r">
library(tidyverse)
library(lubridate)

# Asegurarse de tener columna de mes
journal <- journal %>%
  mutate(
    fecha = as.Date(fecha),
    mes = floor_date(fecha, "month")
  )

# 1. Calcular los top 6 juegos del año
top6_juegos <- journal %>%
  group_by(nombre) %>%
  summarise(minutos = sum(duracion_min, na.rm = TRUE)) %>%
  slice_max(minutos, n = 6) %>%
  pull(nombre)

# 2. Crear nueva columna para agrupar "Otros"
journal <- journal %>%
  mutate(
    juego_cat = if_else(nombre %in% top6_juegos, nombre, "Otros")
  )

# Convertir en factor con orden deseado
niveles_ordenados <- c(top6_juegos, "Otros")
journal <- journal %>%
  mutate(juego_cat = factor(juego_cat, levels = niveles_ordenados))

# 3. Agrupar por mes y categoría
tiempo_mensual <- journal %>%
  group_by(mes, juego_cat) %>%
  summarise(minutos = sum(duracion_min, na.rm = TRUE), .groups = "drop") %>%
  mutate(horas = minutos / 60)

# 4. Calcular totales por mes para mostrar texto
totales_mes <- tiempo_mensual %>%
  group_by(mes) %>%
  summarise(
    total_minutos = sum(minutos),
    horas = total_minutos %/% 60,
    minutos = total_minutos %% 60,
    etiqueta = paste0(horas, "h ", minutos, "m")
  )

# 5. Crear gráfica
ggplot(tiempo_mensual, aes(x = mes, y = horas, fill = juego_cat)) +
  geom_col() +
  geom_text(data = totales_mes,
            aes(x = mes, y = total_minutos / 60 - 5, label = etiqueta),
            inherit.aes = FALSE,
            size = 4.2, angle = 90, color = "black") + 
  labs(
    title = "Videojuegos 2024",
    x = "",
    y = "Horas",
    fill = "Videojuego"
  ) +
  scale_x_date(
    date_labels = "%B",
    date_breaks = "1 month"
  ) +
  scale_fill_brewer(palette = "Accent") +
  theme_minimal() +
  theme(
    plot.title = element_text(face = "bold", size = 16, hjust = 0.5),
    axis.text.x = element_text(angle = 45, hjust = 1),
    legend.position = "right"
  )
  </code></pre>
</details>
~~~

~~~
<img src="/assets/meses2024.png" />
~~~

Mi gráfica favorita. De las mejores que he hecho.

Nótese los picos de actividad en vacaciones y el impacto de Helldivers II. Las primeras semanas jugaba diario con mis amigos de Discord, que luego se fueron a Space Marine II y Monster Hunter en los meses siguientes. Puede observarse como dejé de jugar tanto después.

En verano me invaden unas ganas inexplicables de jugar Los Sims.

A finales de septiembre conseguí un iPad con tres meses gratis de Apple Arcade. Aproveché para jugar Balatro y desde entonces no lo he dejado. Terminó el periodo de prueba y pagué por el juego aparte para jugarlo sin suscripción. Es sencillamente el mejor título disponible para móviles. ¡Viva Balatro!

Algo importante de mis hábito de consumo es que juego un poco de todo. Me gustan los juegos cortos. No obstante, el 2024 se destacó porque no terminé tantos juegos como en años previos. Los JRPGs que suelen aportar muchas horas no me engancharon o descubrí que descargué la versión incompleta y mi partida no es compatible con la versión definitiva por lo que debo empezar desde el inicio (gracias Atlus).


## Histograma de sesiones según el día de la semana

~~~
<details>
  <summary>Mostrar código</summary>

  <pre><code class="language-r">
# Crear columna con el día de la semana
journal <- journal %>%
  mutate(dia_semana = wday(fecha, label = TRUE, abbr = FALSE))  # Día de la semana como nombre completo

# Crear histograma del día de la semana
ggplot(journal, aes(x = dia_semana)) +
  geom_bar(fill = "#4C9E8C", color = "#2C6F57", size = 0.5) +  # Colores más suaves
  geom_text(
    stat = "count", aes(label = ..count..), vjust = -0.5, size = 5, fontface = "bold", color = "black"
  ) +  # Agregar la cantidad exacta sobre las barras
  labs(
    title = "Sesiones de juego por dia de la semana",
    x = "",
    y = "Cantidad de sesiones"
  ) +
  scale_y_continuous(limits = c(0, 65)) +  # Limitar el eje Y a un máximo de 60
  theme_minimal() +
  theme(
    plot.title = element_text(face = "bold", size = 18, hjust = 0.5),
    axis.title = element_text(size = 12),
    axis.text = element_text(size = 10, color = "#333333"),
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank(),
    axis.ticks.x = element_blank()
  )
  </code></pre>
</details>
~~~

~~~
<img src="/assets/histograma2024.png" />
~~~

Aclaro que Backloggd registra como sesión a cada juego registrado en ese día, sin importar la duración. Esta gráfica se puede interpretar como en qué tan común es que juegue uno o varios videojuegos según el día de la semana, porque hubo días donde no jugué nada.

## Tiempo total por día de la semana

~~~
<details>
  <summary>Mostrar código</summary>

  <pre><code class="language-r">
# Crear columna con el día de la semana y convertir la duración en minutos
journal <- journal %>%
  mutate(dia_semana = wday(fecha, label = TRUE, abbr = FALSE),  # Día de la semana como nombre completo
         duracion_min = parse_duration(duracion))  # Asegúrate de convertir la duración a minutos

# Calcular horas y minutos de la duración total
journal <- journal %>%
  mutate(duracion_horas = floor(duracion_min / 60),
         duracion_restante_min = duracion_min %% 60)

# Calcular horas totales de juego por día de la semana
horas_por_dia <- journal %>%
  group_by(dia_semana) %>%
  summarise(horas_totales = sum(duracion_horas), minutos_totales = sum(duracion_restante_min)) %>%
  mutate(
    horas_totales = horas_totales + floor(minutos_totales / 60),
    minutos_totales = minutos_totales %% 60,
    etiqueta = sprintf("%dh %02dm", horas_totales, minutos_totales)
  ) %>%
  arrange(dia_semana)

# Crear gráfico de barras para las horas y minutos de juego
ggplot(horas_por_dia, aes(x = dia_semana, y = horas_totales + minutos_totales / 60)) +
  geom_bar(stat = "identity", fill = "#4C9E8C", color = "#2C6F57", size = 0.5) +  # Colores más suaves
  geom_text(
    aes(label = etiqueta), vjust = -0.5, size = 5, fontface = "bold", color = "black"
  ) +  # Agregar la cantidad exacta de horas y minutos sobre las barras
  labs(
    title = "Horas de juego totales por dia de la semana",
    x = "",
    y = "Horas"
  ) +
  scale_y_continuous(limits = c(0, 65)) +  # Limitar el eje Y a un máximo de 60
  theme_minimal() +
  theme(
    plot.title = element_text(face = "bold", size = 18, hjust = 0.5),
    axis.title = element_text(size = 12),
    axis.text = element_text(size = 10, color = "#333333"),
    panel.grid.major.x = element_blank(),
    panel.grid.minor.x = element_blank(),
    axis.ticks.x = element_blank()
  )
  </code></pre>
</details>
~~~

~~~
<img src="/assets/semana2024.png" />
~~~

Esta gráfica es más interesante. Muestra en qué día de la semana acostumbraba jugar por más tiempo. Me sorprende lo uniforme que es. Recordando que el año tiene 52 semanas, se puede apreciar que más o menos jugaba una hora diaria, siendo el lunes y martes los días con más actividad. Que sorpresa.

# Tiempo total

```r
total_duracion <- sum(journal$duracion_min, na.rm = TRUE)

horas <- total_duracion %/% 60
minutos <- total_duracion %% 60

cat("Tiempo total: ", horas, "horas y", minutos, "minutos\n")
```

```
## Tiempo total:  357 horas y 28 minutos
```

# Trabajo a futuro

* Webscraping más eficiente que recopile más información.

* Añadir datos de género, desarrollador, plataforma, año de lanzamiento, país, etcétera.

* Crear un dashboard interactivo para conocer a detalle cada juego o mes.
