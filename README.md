# :blue_heart: CRAN-SUBMISSION :earth_americas:
Este repositorio contiene todos los recursos necesarios y una guía paso a paso sobre cómo desarrollar y publicar un paquete en `CRAN` :see_no_evil:. Encontrarás ejemplos de código, configuraciones recomendadas, y mejores prácticas para asegurar que tu paquete cumpla con los estándares de `CRAN` y sea accesible para la comunidad de `R` :heartpulse:.

<center>
	<img src="https://i.gifer.com/92hh.gif" alt="Meme Deploy Software" border="2px solid red" height="120"/>
	<img src="https://i.gifer.com/dYt.gif" alt="Meme Deploy Software" border="2px solid red" height="120"/>
</center>

___
## :key: Disponibilidad del Nombre

Compruebe inicialmente que el nombre está disponible :monocle_face:. En caso de que no tenga una idea clara para éste puede intentar con `available::suggest("<DESCRIPCIÓN DE SU PAQUETE>")`.

```r
available::available("UnalR", browse = FALSE)
# ── UnalR ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
# Name valid: ✔
# Available on CRAN: ✖
# Available on Bioconductor: ✔
# Available on GitHub:  ✖
```

___
## :compass: Preliminares

Ejecute el siguiente código para modificar el fichero `DESCRIPTION` del paquete con la lista de dependencias extraídas de los ficheros `R`, tests, viñetas, etc.

```r
attachment::att_amend_desc()
```

```r
# Encontrar qué funciones están exportadas pero les falta la etiqueta @export
checkhelper::find_missing_tags()
checkhelper::check_clean_userspace()
# Corregir la Ortografía - Sin Erratas
spelling::spell_check_package()
# usethis::use_spell_check()
# Comprobar que las URL son Correcta
urlchecker::url_check()
urlchecker::url_update()
```

___
## :man_mechanic: Escritura de Pruebas

Apoyado de la librería [testthat](https://testthat.r-lib.org/), genere las pruebas unitarias siguiendo el esquema utilizado por miles de paquetes `CRAN` :dizzy:.

```r
pak::pak("r-lib/testthat")
usethis::use_testthat()

devtools::test() 			 # Ejecute todas las pruebas de su paquete
devtools::test_active_file() # Ejecuta test() en el archivo activo
```

> :v: **Tip:** Siempre que encuentre un error, escriba una prueba para comprobar que no se repita en el futuro.

___
## :test_tube: Check Package Coverage | [COVR](https://covr.r-lib.org/)

Este paquete le ayudará a dar seguimiento de la **cobertura de las pruebas** :100: en su paquete de `R`, todo lo anterior apoyado de informes en su equipo o, si lo prefiere, subirlos a [codecov](https://about.codecov.io/).

```r
install.packages("covr")
usethis::use_github_action("test-coverage")
covr::package_coverage()
covr::report()
covr::codecov(path = "test", token = "INCLUDE_YOUR_CODECOV_TOKEN_HERE")
```

:computer_mouse: Adicionalmente puede ejecutar, ver y guardar los resultados de una forma personalizada:

```r
# Test Coverage of a Whole Package
covr <- package_coverage()
covr
report(covr)
```

```r
# Coverage of Individual Files
library(tidyr); library(dplyr); library(highcharter)
covr <- file_coverage(
  c(
    "R/Auxiliares.R",
     "R/Plot_Series.R"
  ),
  c(
    "tests/testthat/test-Plot_Series.R"
  )
)
covr
report(covr)
```

___
## :spider_web: Look up Dependencies | [PAK](https://pak.r-lib.org/)

Es importante analizar detalladamente las dependencias **reales** de nuestro paquete, para ello es bueno dibujar y tener a la mano el :deciduous_tree: árbol de dependencias :thread:. Pues usualmente al realizar el debugger en el despliegue multiplataforma saldrán optimizaciones posibles de aquí.

```r
pak::pkg_deps_tree("https://github.com/estadisticaun/UnalR")
pak::pkg_deps_tree("local::D:/Documentos/JOBS/DNPE/UnalR_1.0.0.zip")
```

Al ejecutar el código anterior obtendrá una salida como la siguiente :mag_right:,

```r
! Using bundled GitHub PAT. Please add your own PAT using `gitcreds::gitcreds_set()`.
✔ Updated metadata database: 5.30 MB in 15 files.
✔ Updating metadata database ... done
https://github.com/estadisticaun/UnalR 1.0.0 [new][bld][cmp][dl] (unknown size)
├─d3r 1.1.0 [new][dl] (723.96 kB)
│ ├─dplyr 1.1.4 [new][dl] (1.58 MB)
│ │ ├─cli 3.6.2 [new][dl] (1.36 MB)
│ │ ├─generics 0.1.3 [new][dl] (83.69 kB)
│ │ ├─glue 1.7.0 [new][dl] (163.29 kB)
│ │ ├─lifecycle 1.0.4 [new][dl] (140.93 kB)
│ │ │ ├─cli
│ │ │ ├─glue
│ │ │ └─rlang 1.1.3 [new][dl] (1.61 MB)
│ │ ├─magrittr 2.0.3 [new][dl] (229.42 kB)
...
Key:  [new] new | [upd] update | [dl] download | [bld] build | [cmp] compile
```

___
## :traffic_light: Comprobaciones Multiplataforma | [R-Hub](https://r-hub.github.io/rhub/)

Antes de iniciar a usa la librería en cuestión, es necesario crear un token temporal del repositorio (*Personal Access Token - PAT*) :octocat:, si tiene dudas de cómo crearlo puede ejecutar la función `usethis::create_github_token()`. Adicionalmente si requiere más información consulte la [documentación](https://usethis.r-lib.org/articles/git-credentials.html).

Una vez tenga en el portapapeles :paperclip: el token es necesario agregar dicho `PAT` en el archivo `.Renviron`, mediante el comando `usethis::edit_r_environ()`. Si realizó los pasos de manera correcta, debió quedarle algo así:

```r
GITHUB_PAT=xxxxx
```

> :warning: ¡Recuerde agregar dicho archivo en el `.gitignore` pues no se puede publicar tokens y GitHub, por seguridad, no dejará hacer el commit!

Una vez cuente con dicha configuración de manera **local**, ejecute de manera secuencial los siguientes comandos, los cuales le permitirán comprobar que su paquete compile en varias plataformas con la configuración predeterminada de `CRAN`, utilizando `GitHub Actions`.

```r
pak::pkg_install("rhub")
rhub::rhub_setup()
rhub::rhub_doctor()
rhub::rhub_platforms()
rhub::rhub_check()
```

Si todo sale de manera correcta, la última línea le solicitará por consola que ingrese los sistemas a incluir, digitando el código de cada uno, recuerde que cuenta con más de :two::zero: Virtual Machines y Containers a su disposición.

```r
> Selection (comma separated numbers, 0 to cancel):
```

Tenga cuidado ya que la última línea inicializa las comprobaciones de `R-hub v2` en `GitHub Actions` :exploding_head: (*¡de manera automática!, para eso se requería el* `PAT`).

Obtendrá como resultado la siguiente ejecución automatizada en `GitHub Actions` (*puede variar dependiendo la personalización del archivo `YAML` y las VM elegidas*):

![](https://github.com/JeisonAlarcon/CRAN-Submission/blob/main/assets/images/R-Hub.jpg)

___
## :bookmark: Buenas Practicas | [goodpractice](https://ropensci-review-tools.github.io/goodpractice/)

Esta librería proporciona recomendaciones sobre las mejores prácticas para compilar paquetes en `R` :nerd_face:. Estas recomendaciones abarcan aspectos como las funciones y la sintaxis que conviene evitar, la organización de la estructura del paquete, la complejidad del código, el formato del código, entre otros.

```r
remotes::install_github("mangothecat/goodpractice")
goodpractice::gp()
```

- :male_detective: Son tantas las reglas que valida que no enlistaremos todas, solamente algunas de las más comunes y que obtuvimos en nuestro camino:

```
use '<-' for assignment instead of '='. '<-' is the standard, and R
users and developers are used it and it is easier to read your code
for them if you use '<-'.
```

```
avoid 1:length( ... ), 1:nrow( ... ), 1:ncol( ... ),
1:NROW( ... ) and 1:NCOL( ... ) expressions. They are error
prone and result 1:0 if the expression on the right hand
side is zero. Use seq_lenO or seq_along() instead.
```

```
avoid sapply(), it is not type safe.
It might return a vector, or a list, depending on the
input data. Consider using vapply() instead.
```

```
avoid 'T' and 'F", as they are just variables which are set to the
logicals 'TRUE' and 'FALSE' by default, but are not reserved words
and hence can be overwritten by the user. Hence, one should
always use 'TRUE' and 'FALSE' for the logicals.
```

```
write short and simple functions. These functions have high cyclomatic
complexity (>50): Plot.Mapa (141), Plot.Mundo (98), Plot.Series (82),
Plot.Boxplot (68), .... You can make them easier to reason about by
encapsulating distinct steps of your function into subfunctions.
```

```
avoid long code lines, it is bad for
readability. Also, many people prefer editor windows
that are about 80 characters wide. Try make your lines
shorter than 80 characters
```

___
## :love_letter: CRAN Comments

Recuerde crear una plantilla para sus comunicaciones con `CRAN` al enviar un paquete :blush:. El objetivo es detallar claramente los pasos que ha seguido para verificar su paquete en diversos sistemas operativos :nail_care:. Si está enviando una actualización para un paquete que es utilizado por otros paquetes, también es necesario resumir los resultados de las comprobaciones de dependencias inversas :robot: en el archivo `cran-comments.md`.

```r
# Add comments for CRAN
usethis::use_cran_comments()
```

Si no sabe muy bien qué va, en qué orden y su estructura, puede guiarse de los siguientes ejemplos:

- [Example 1.](https://github.com/estadisticaun/UnalR/blob/master/cran-comments.md)
- [Example 2.](https://github.com/r-spatial/leafpm/blob/master/cran-comments.md)
- [Example 3.](https://github.com/vnijs/gitgadget/blob/main/cran-comments.md)
- [Example 4.](https://lab.compute.dtu.dk/packages/onlineforecast/-/blob/develop/cran-comments.md)
- [Example 5.](https://rdrr.io/github/christophsax/indiedown/f/cran-comments.md)

___
## :green_circle: Comprobación Final

La siguiente función ejecuta una comprobación local de `R CMD`, :eye: ojo, tenga presente que es diferente a usar en RStudio `Build > Check` :raised_eyebrow:.

> :triangular_flag_on_post: Si bien darán resultados idénticos una vez estabilizado su código, en un inicio puede que obtenga más errores, advertencias o notas en un lado que otro, recomiendo usar siempre los dos.

```r
devtools::check()
```

- Si considera que alguna advertencia o nota no están justificadas, deja un comentario al enviarlas donde especifiques por qué crees que esto no está justificado :judge:.

```r
# _win devel CRAN
devtools::check_win_devel() # ¿Compila con la herramienta win-builder?
# _win release CRAN
devtools::check_win_release()
# _macos CRAN
# Es necesario seguir la URL para ver los resultados
devtools::check_mac_release()
```

:round_pushpin: Si bien algunas notas son fácilmente solventables, como, por ejemplo,

```
❯ checking Rd line widths ... NOTE
	\examples lines wider than 100 characters:
```

```
Note checking data for non-ASCII characters (899ms)
	 Note: found 100 marked Latin-1 strings
	 Note: found 267537 marked UTF-8 strings
```

:pushpin: Y otras se pueden negociar/justificar con el equipo del `CRAN` :speech_balloon: (*el paquete [UnalR](https://github.com/estadisticaun/unalr) es ejemplo vivo de ello*), como, por ejemplo,

```
❯ checking package dependencies ... NOTE
	Imports includes >20 non-default packages.
	Importing from so many packages makes the package vulnerable to any of
	them becoming unavailable.  Move as many as possible to Suggests and
	use conditionally.
```

```
❯ checking installed package size ... NOTE
	installed size is  5.4Mb
	sub-directories of 1Mb or more:
	  R      1.8Mb
	  help   2.6Mb
```

:gear: Otras se deben corregir sí o sí, pues hacen parte del sistema de validación interna, y si no pasan se regresará el paquete sin chance de argumentar, como, por ejemplo,

```
❯ checking examples ... [68s] NOTE
  Examples with CPU (user + system) or elapsed time > 5s
				user system elapsed
  Plot.Mundo   13.67   1.19   14.91
  Plot.Mapa    10.85   1.64   12.58
  Plot.Series   5.70   4.78    6.39
  Plot.Boxplot  5.75   1.00    6.84
```

___
## :man_technologist: On its way to CRAN

```r
# Upgrade Version Number
usethis::use_version()
# Verify you're Ready for Release
devtools::release() # Envío automático a CRAN desde R
```

Luego de estar de acuerdo con la versión que se publicará y de qué **no obtenga ningún :x: error, :warning: warning o nota :capital_abcd:** en los pasos anteriores. Se le preguntará por consola si:

- Have you checked for spelling errors (with `spell_check()`)?
- Have you run `R CMD check` locally?
- Were devtool's checks successful?
- Have you checked on R-hub (with `check_rhub()`)?
- Have you checked on win-builder (with `check_win_devel()`)?
- Have you updated `NEWS.md` file?
- Have you updated cDESCRIPTION`?
- Have you updated `cran-comments.md?`
- Were Git checks successful?
- Is your email address email@example.com?
- Ready to submit `UnalR (1.0.0)` to CRAN?

> :anger: Tenga presente que para este paso ya debe estar al día con los cambios, es decir, no tener nada en el staging area y que todo este rastreado con el último commit :eyes:. En caso contrario obtendrá el siguiente error:
![](https://github.com/JeisonAlarcon/CRAN-Submission/blob/main/assets/images/ErrorGitChecks.jpg)

Finalmente se creará/compilará el `.tar.gz` :card_file_box: de su paquete en una ruta temporal y se cargará dicho archivo al [CRAN](https://cran.r-project.org/submit.html), tenga presente que se le enviará de manera inmediata un email de aceptación de política interna :email: al correo previamente mostrado, deberá aceptarlo de manera inmediata.

![](https://github.com/JeisonAlarcon/CRAN-Submission/blob/main/assets/images/Submit2CRAN.jpg)

Y eso es todo, :tada: **su paquete se encontrará en proceso de validación y publicación** :sparkles:

> :fire: Puede dar seguimiento de cómo va su paquete en [CRAN Incoming Dashboard](https://r-hub.github.io/cransays/articles/dashboard.html) :bar_chart:, el cual cada hora actualiza el estado dentro del flujo de revisión del `CRAN` (*también puede ingresar para leer una descripción detallada de lo que significa el nombre de cada carpeta por la que pasará su paquete*) , o de manera directa en https://cran.r-project.org/incoming/

___
## :star2: Problemas Resueltos

- Migración de funciones, al encontrar que algunos paquetes no estaban disponibles en el [CRAN](https://github.com/d3treeR/d3treeR/issues/33), y existía una dependencia directa, es necesario examinar qué funciones se usan de dicho paquete y tratar de incorporarla al código (*¡acuérdate de dar los créditos!*).
- Si está probando en reiteradas ocasiones correr `check_win_devel()`, es posible que en alguna ocasión le aparezca este error, tranquil@, la máquina externa en win-builder.r-project.org para realizar pruebas está tomando un descanso, espere un momento y vuelva a intentarlo.
	```r
	R devtools check win functions throw Error in
	curl::curl_fetch_memory(url, handle = h) : Recv failure: Connection was reset
	```
- Si obtiene el error: `R CMD check, WARNING, Rd cross-reference, no package available`, la solución es muy sencilla, pues se debe a que en la documentación de un parámetro dentro de una de sus funciones está invocando librerías que no exporta, elimínela o inclúyala dependiendo sus necesidades.
- Actualizar el nuevo estilo de `CITATION`, se cambió de `citEntry()` por `bibentry()`, tranquilo a nosotros nos pasó y a los [grandes igual](https://github.com/tidyverse/ggplot2/commit/96a6b547984e71b4b4fb8e947187dddcec71c545).

___
## :books: Resources

### *Documentación Base*

- [Checklist for CRAN Submissions](https://cran.r-project.org/web/packages/submission_checklist.html)
- [CRAN Repository Policy](https://cran.r-project.org/web/packages/policies.html)
- [R Packages (2e)](https://r-pkgs.org/)
- [ExtraChecks](https://github.com/DavisVaughan/extrachecks)
- [Style](https://style.tidyverse.org/)

### *Avanzado*
- https://github.com/ThinkR-open/prepare-for-cran
- https://www.r-bloggers.com/2020/07/how-to-write-your-own-r-package-and-publish-it-on-cran/
- https://www.mzes.uni-mannheim.de/socialsciencedatalab/article/r-package/

___
## :man: Acerca del Autor

Soy estadístico de profesión y apasionado por la ciencia de datos :chart_with_upwards_trend:, he dedicado gran parte de mi carrera profesional a explorar patrones, tendencias y relaciones en conjuntos de datos complejos.

Sin embargo, también creo firmemente en la importancia de que el conocimiento sea accesible para todos :unlock:. Creo en la filosofía de que el aprendizaje y la información no deben tener barreras :world_map:. Por eso, me dedico a compartir mi conocimiento y experiencia a través de plataformas en línea, blogs y comunidades de código abierto :speaking_head:.

Al momento de escribir estas notas :collision: hay más de [20.808](https://cran.r-project.org/web/packages/)  paquetes disponibles :exploding_head:. Esto puede resultar abrumador, especialmente al buscar el paquete adecuado para un problema específico o al intentar aprender sobre lo que otros están haciendo. Existen algunas cuentas de [Twitter](https://x.com/RLangPackage) y publicaciones de [blog](https://www.r-bloggers.com/) realmente útiles y perspicaces que ofrecen una excelente visión general de los temas más relevantes y las figuras emergentes en el mundo de los paquetes de `R`.

___
## :building_construction: Directory & File Structure

```
└── 📁CRAN-Submission
    └── 📁assets
        └── 📁images
            └── ErrorGitChecks.jpg
            └── Submit2CRAN.jpg
    └── LICENSE
    └── README.md
```
