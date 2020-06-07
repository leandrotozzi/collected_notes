# Python en el lab
## Generación de reportes de mediciones
### Introducción
La idea de esta **note** es contar brevemente como generar reportes de mediciones en formato **.docx** (MS Word) de forma programática con ayuda de **Python**.   
En este ejemplo, utilizaremos como datos de entrada, archivos **.mat**, muy comúnmente utilizados para almacenar mediciones utilizando [Matlab.](https://www.mathworks.com).
Una de las primeras preguntas que surgen, es el por qué del formatos de entrada/salida. 
Si bien Latex o Markdown son sumamente utilizado para documentación de desarrollos, otras áreas, como pueden ser soporte al cliente, marketing, pueden llegar a tener preferencia por herramientas del estilo de **Office**. 
La siguiente aclaración es, que si bien muchos adquisidores de datos y/o instrumentos de medición cuentan con interfaces fácilmente programables en Python (**ya vendrá un note**), hay mucho de **legacy** todavía dando vueltas y por lo tanto suele ser buena idea tener soporte para procesar mediciones realizadas utilizando Matlab como soporte. 


![Introduction](https://raw.githubusercontent.com/leandrotozzi/collected_notes/master/python-lab/cn-intro.png)


Entonces, en este note vamos a contar lo siguiente:
1. Cargar los archivos de datos **.mat** utilizando **scikit**
2. Armar los plots que necesitamos utilizando **matplotlib/seaborn**
3. Generar el reporte en formato **.docx** usando **python-docx**.

### Cargar archivos .mat
Como ejemplo de este note vamos a utilizar el siguiente dataset de acceso libre:  [Fisher's Iris Dataset](https://en.wikipedia.org/wiki/Iris_flower_data_set)  (en formato **.mat**).  No tiene nada que ver con conversores ADC, adquisidores de datos, mediciones eléctricas, pero a fines prácticos sirve igual.  Y no tenemos problemas de tamaño ni de propiedad intelectual =)

```python
# Funcion  para leer .mat
from scipy.io import loadmat
# La info la pasamos a un dataframe
import pandas as pd

# Leemos la informacion
data_mat = 'fisheriris.mat'
data = loadmat(data_mat)

# data es un dict con 2 keys importantes:
# - meas    : Contiene cada row de datos
# - species : Tipo de planta

# Armamos el dataframe
col = ['sepal_length', 'sepal_width', 
       'petal_length', 'petal_width']
df = pd.DataFrame(data['meas'], columns=col)
# Agregamos la columna de species
df['specie'] = [e[0][0] for e in data['species']]
```
Listo, ya tenemos los datos en nuestro dataframe:

![df head](https://raw.githubusercontent.com/leandrotozzi/collected_notes/master/python-lab/df_ok.png)

### Generar plots usando matplotlib/seaborn
Una vez que ya tenemos nuestra data en un dataframe, podemos utilizar **matplotlib**, **seaborn** o cualquier otra librería que creamos convenientes para generar los gráficos que necesitamos incluir en nuestro reporte. 
Con el objetivo de mantener simple y compacto el ejemplo, aca solo generaremos un gráfico:

```python
import seaborn as sns

plot_filename = 'cn_out.png'
sns.set_style("whitegrid")
sns_plot = sns.pairplot(df,hue="specie",size=2.5);
sns_plot.savefig(plot_filename)
```

### Generar archivo MS docx
Una vez que tenemos todos los plots que necesitamos para nuestro informe, el último paso es generar el archivo **.docx**. 
Para esto utilizaremos el módulo **python-docx**, que pueden instalar en su sistema mediante:

```bash
pip install python-docx
```

Generalmente para presentar un reporte, tenemos algún template de la empresa/institución que incluye el logo, un header, footer y nosotros tenemos que agregar el contenido nuevo. En este ejemplo, usaremos un archivo como template (**rpt_template.docx**) que ya tiene precargado un logo y un texto en el footer. A este documento le vamos a agregar un título en el header e incluir el gráfico creado en la etapa anterior. 

```python
import docx
from docx.shared import Inches

# Configs
data_mat = 'fisheriris.mat'   # Dataset
tpl_rpt = 'rpt_template.docx' # Word Template file
out_rpt = 'reporte.docx'      # Output File
plot_1 = 'cn_out.png'         # Image to include

doc = docx.Document(tpl_rpt)

# Get Header
section = doc.sections[0]
header = section.header
paragraph = header.paragraphs[0]

# Add Header Text using template
paragraph.runs[1].text = 'Header - Titulo del reporte'
paragraph = header.paragraphs[0]

doc.add_heading('Reporte Mediciones', level=1)
# Add Plots
doc.add_heading(f'Dataset: {data_mat}', level=3)
doc.add_picture(plot_1, width=Inches(6))

# Save Document
doc.save(out_rpt)
```

##### Resultado
![res](https://raw.githubusercontent.com/leandrotozzi/collected_notes/master/python-lab/res.png)

### Finalizando...
* Mediante Python podemos automatizar la creación de reportes. 
* Python incluye librerías muy versátiles para procesar y visualizar los datos.
* Este fue un simple ejemplo, pero cuando empiezan a aparecer cientos de mediciones para validar un producto, resulta increiblemente útil poder generar los reportes de esta manera. 
* [Jupyter Notebook Here!](https://github.com/leandrotozzi/collected_notes)