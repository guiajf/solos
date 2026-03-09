# Mapeamento geomorfológico de Juiz de Fora

### Introdução

O **SISURB** - **Sistema Municipal de Informações para o Desenvolvimento Territorial** desenvolve e organiza dados para mapeamento do território de Juiz de Fora, em seus aspectos topográficos, geológicos e geográficos.

São disponibilizados diversos mapas temáticos, dentre os quais o de solos, que contém a classificação dos solos do município, com a descrição de sete classes diferentes: Cambissolos Húmicos, Cambissolos Háplicos, Latossolos Amarelos, Latossolos Vermelho-Amarelos, Latossolos Vermelho-Amarelos, Latossolos Vermelho-Amarelos, Latossolos Vermelho-Amarelos.

### Objetivo

Este projeto tem como objetivo construir o mapa dos solos de Juiz de Fora, a partir dos dados disponibilizados pelo **SISURB**. 

### Importamos as bibliotecas


```python
import os
import zipfile
import geopandas as gpd
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import folium
from folium import plugins
from folium.plugins import Fullscreen
import rasterio
from rasterio.mask import mask
from shapely.geometry import mapping
from tqdm import tqdm 
import warnings
warnings.filterwarnings('ignore')
```

### Configuramos os diretórios

Configuramos os diretórios, para baixar manualmente o arquivo SOLOS.zip do repositório oficial (https://www.pjf.mg.gov.br/desenvolvimentodoterritorio/sistema_informacoes/solos.php).


```python
PATH_ZIP = "SOLOS.zip"
PATH_EXTRACTED = "JF_Solos_JF"
```

### Extraímos o shapefile

*Shapefile* é um arquivo digital que contém a representação geográfica vetorial do objeto geoespacial.


```python
if not os.path.exists(PATH_EXTRACTED):
    print("Descompactando arquivo ZIP...")
    with zipfile.ZipFile(PATH_ZIP, 'r') as zip_ref:
        # Extrai TODOS os arquivos do shapefile
        shapefile_extensions = ['.shp', '.shx', '.dbf', '.prj', '.cpg', '.sbn', '.sbx']
        essential_files = [f for f in zip_ref.namelist() 
                          if any(f.endswith(ext) for ext in shapefile_extensions)]
        
        print(f"Encontrados {len(essential_files)} arquivos do shapefile")
        for file in tqdm(essential_files, desc="Extraindo"):
            zip_ref.extract(file, PATH_EXTRACTED)
```

    Descompactando arquivo ZIP...
    Encontrados 35 arquivos do shapefile


    Extraindo: 100%|████████████████████████████████| 35/35 [00:00<00:00, 52.85it/s]


### Carregamos o shapefile


```python
gdf = gpd.read_file(f"{PATH_EXTRACTED}/Solos.shp")
```

### Definimos o CRS

Definimos o *Sistema de Referência de Coordenadas* (**CRS**) para o modelo **WGS84** (*World Geodesic System 1984*), que é o padrão adotado pelo *Folium*.


```python
gdf = gdf.to_crs(epsg=4326)
```

### Visualizamos o GeoDataFrame

Um *GeoDataFrame* é uma estrutura de dados tabular da biblioteca **GeoPandas** (baseada no **Pandas**) que estende os *DataFrames* comuns para armazenar e manipular dados espaciais. Ele obrigatoriamente contém uma coluna especial chamada *geometry*, que armazena as coordenadas geoespaciais e permite operações como cálculo de áreas ou distâncias


```python
gdf
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>OBJECTID_1</th>
      <th>UM</th>
      <th>FIRST_CLAS</th>
      <th>FIRST_PORC</th>
      <th>FIRST_CL_1</th>
      <th>FIRST_PO_1</th>
      <th>FIRST_CL_2</th>
      <th>FIRST_PO_2</th>
      <th>FIRST_CL_3</th>
      <th>...</th>
      <th>Vegetaçã</th>
      <th>TIPO</th>
      <th>LEVANTAMEN</th>
      <th>classe</th>
      <th>SHAPE_Leng</th>
      <th>SHAPE_Area</th>
      <th>geometry</th>
      <th>AREA_KM2</th>
      <th>PERIM_KM</th>
      <th>CLASSE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>21.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd68</td>
      <td>1</td>
      <td>1</td>
      <td>27146.113453</td>
      <td>1.810461e+07</td>
      <td>POLYGON ((-43.63415 -21.8783, -43.63389 -21.87...</td>
      <td>18.104611</td>
      <td>27.146113</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>22.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd68</td>
      <td>1</td>
      <td>1</td>
      <td>25357.789586</td>
      <td>2.428209e+07</td>
      <td>POLYGON ((-43.50649 -21.85235, -43.50644 -21.8...</td>
      <td>24.282088</td>
      <td>25.357790</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>24.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd68</td>
      <td>1</td>
      <td>1</td>
      <td>45812.482211</td>
      <td>1.607142e+07</td>
      <td>MULTIPOLYGON (((-43.18073 -21.81434, -43.17968...</td>
      <td>16.071423</td>
      <td>45.812482</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>30.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>90004.539458</td>
      <td>1.216135e+08</td>
      <td>POLYGON ((-43.41985 -21.7242, -43.42078 -21.72...</td>
      <td>121.613475</td>
      <td>90.004539</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>33.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>105487.029792</td>
      <td>1.206889e+08</td>
      <td>MULTIPOLYGON (((-43.33895 -21.65291, -43.33878...</td>
      <td>120.688934</td>
      <td>105.487030</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>0.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>CHd4</td>
      <td>0</td>
      <td>3</td>
      <td>34878.559145</td>
      <td>8.613532e+05</td>
      <td>MULTIPOLYGON (((-43.17247 -21.78457, -43.17245...</td>
      <td>0.861353</td>
      <td>34.878559</td>
      <td>Cambissolos Húmicos</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>39.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>1173.080705</td>
      <td>6.075919e+04</td>
      <td>POLYGON ((-43.47097 -21.52798, -43.47086 -21.5...</td>
      <td>0.060759</td>
      <td>1.173081</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>39.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>9805.035543</td>
      <td>3.327967e+06</td>
      <td>POLYGON ((-43.48909 -21.53185, -43.48894 -21.5...</td>
      <td>3.327967</td>
      <td>9.805036</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd59</td>
      <td>1</td>
      <td>4</td>
      <td>277.124327</td>
      <td>1.721548e+03</td>
      <td>POLYGON ((-43.56337 -21.48846, -43.56464 -21.4...</td>
      <td>0.001722</td>
      <td>0.277124</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd59</td>
      <td>1</td>
      <td>4</td>
      <td>291110.834020</td>
      <td>7.232797e+08</td>
      <td>POLYGON ((-43.45241 -21.52139, -43.45223 -21.5...</td>
      <td>723.279666</td>
      <td>291.110834</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd59</td>
      <td>1</td>
      <td>4</td>
      <td>163.749829</td>
      <td>6.854394e+02</td>
      <td>POLYGON ((-43.68565 -21.63403, -43.68548 -21.6...</td>
      <td>0.000685</td>
      <td>0.163750</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>48.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd59</td>
      <td>1</td>
      <td>4</td>
      <td>1194.021078</td>
      <td>1.640655e+04</td>
      <td>POLYGON ((-43.68437 -21.63986, -43.68434 -21.6...</td>
      <td>0.016407</td>
      <td>1.194021</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>49.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>CXbd33</td>
      <td>1</td>
      <td>5</td>
      <td>47010.110658</td>
      <td>4.154106e+07</td>
      <td>POLYGON ((-43.47923 -21.74054, -43.47569 -21.7...</td>
      <td>41.541058</td>
      <td>47.010111</td>
      <td>Cambissolos Háplicos</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>49.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>CXbd33</td>
      <td>1</td>
      <td>5</td>
      <td>11873.735161</td>
      <td>2.988869e+06</td>
      <td>POLYGON ((-43.63503 -21.81874, -43.63502 -21.8...</td>
      <td>2.988869</td>
      <td>11.873735</td>
      <td>Cambissolos Háplicos</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>49.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>CXbd33</td>
      <td>1</td>
      <td>5</td>
      <td>154.430934</td>
      <td>6.805285e+02</td>
      <td>POLYGON ((-43.66261 -21.84515, -43.66332 -21.8...</td>
      <td>0.000681</td>
      <td>0.154431</td>
      <td>Cambissolos Háplicos</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>35.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>47949.903698</td>
      <td>3.653670e+07</td>
      <td>POLYGON ((-43.65644 -21.5982, -43.65634 -21.59...</td>
      <td>36.536702</td>
      <td>47.949904</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>35.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>7517.053715</td>
      <td>1.369124e+06</td>
      <td>POLYGON ((-43.53166 -21.62054, -43.53253 -21.6...</td>
      <td>1.369124</td>
      <td>7.517054</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>17</th>
      <td>18</td>
      <td>35.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LAd4</td>
      <td>1</td>
      <td>2</td>
      <td>3403.809546</td>
      <td>4.047155e+05</td>
      <td>POLYGON ((-43.56831 -21.61591, -43.56796 -21.6...</td>
      <td>0.404716</td>
      <td>3.403810</td>
      <td>Latossolos Amarelos</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>25.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>CHd4</td>
      <td>1</td>
      <td>3</td>
      <td>2282.634129</td>
      <td>1.085028e+05</td>
      <td>POLYGON ((-43.64719 -21.83002, -43.64717 -21.8...</td>
      <td>0.108503</td>
      <td>2.282634</td>
      <td>Cambissolos Húmicos</td>
    </tr>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>47.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd58</td>
      <td>1</td>
      <td>6</td>
      <td>220521.989738</td>
      <td>1.941504e+08</td>
      <td>POLYGON ((-43.46723 -21.62497, -43.4657 -21.62...</td>
      <td>194.150398</td>
      <td>220.521990</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>47.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd58</td>
      <td>1</td>
      <td>6</td>
      <td>113923.683562</td>
      <td>9.325729e+07</td>
      <td>POLYGON ((-43.40851 -21.81091, -43.40869 -21.8...</td>
      <td>93.257287</td>
      <td>113.923684</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22</td>
      <td>47.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd58</td>
      <td>1</td>
      <td>6</td>
      <td>343.701576</td>
      <td>2.483750e+03</td>
      <td>POLYGON ((-43.37488 -21.89544, -43.37489 -21.8...</td>
      <td>0.002484</td>
      <td>0.343702</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>22</th>
      <td>23</td>
      <td>20.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd68</td>
      <td>1</td>
      <td>6</td>
      <td>40798.271707</td>
      <td>3.550050e+07</td>
      <td>POLYGON ((-43.49553 -21.9255, -43.4955 -21.925...</td>
      <td>35.500500</td>
      <td>40.798272</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>23</th>
      <td>24</td>
      <td>23.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd73</td>
      <td>1</td>
      <td>7</td>
      <td>11441.235886</td>
      <td>2.057128e+06</td>
      <td>POLYGON ((-43.40401 -21.87563, -43.40383 -21.8...</td>
      <td>2.057128</td>
      <td>11.441236</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
    <tr>
      <th>24</th>
      <td>25</td>
      <td>23.0</td>
      <td>None</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>0</td>
      <td>None</td>
      <td>...</td>
      <td>Fase Floresta Tropical Subperenifólia</td>
      <td>LVAd73</td>
      <td>1</td>
      <td>7</td>
      <td>5296.910900</td>
      <td>7.293902e+05</td>
      <td>POLYGON ((-43.37642 -21.89303, -43.37649 -21.8...</td>
      <td>0.729390</td>
      <td>5.296911</td>
      <td>Latossolos Vermelho-Amarelos</td>
    </tr>
  </tbody>
</table>
<p>25 rows × 59 columns</p>
</div>



### Tratamento de dados

Para reaproveitamento do código utilizado para análise geomorfológica, renomeamos a coluna "UM_SEQ" e adicionamos a coluna "CLASSE", com uso de dicionário de mapeamento código -> classe.


```python
gdf = gdf.rename(columns={'UM_SEQ': 'TIPO'})

# Dicionário de mapeamento código -> classe
mapeamento_classes = {
    'CHd4': 'Cambissolos Húmicos',
    'CXbd33': 'Cambissolos Háplicos',
    'LAd4': 'Latossolos Amarelos',
    'LVAd58': 'Latossolos Vermelho-Amarelos',
    'LVAd59': 'Latossolos Vermelho-Amarelos',
    'LVAd68': 'Latossolos Vermelho-Amarelos',
    'LVAd73': 'Latossolos Vermelho-Amarelos'
}

# Criar a coluna CLASSE usando o mapeamento
gdf['CLASSE'] = gdf['TIPO'].map(mapeamento_classes)
```

### Analisamos as classes geomorfológicas


```python
# Criar tabela comparativa
print("\n" + "="*85)
print("TABELA COMPARATIVA DAS CLASSES DE SOLO")
print("="*85)

# Calcular métricas adicionais
gdf['AREA_KM2'] = gdf['SHAPE_Area'] / 1_000_000
gdf['PERIM_KM'] = gdf['SHAPE_Leng'] / 1000
#gdf['COMPACIDADE'] = (2 * np.sqrt(np.pi * gdf['AREA_KM2'])) / gdf['PERIM_KM'] * 100
#gdf['DENSIDADE'] = gdf['PERIM_KM'] / np.sqrt(gdf['AREA_KM2'])

# Ordenar por área
gdf_sorted = gdf.sort_values('AREA_KM2', ascending=False)

# Criar tabela formatada
print(f"\n{'TIPO':<15} {'CLASSE':<30} {'ÁREA (km²)':>12} {'PERÍMETRO (km)':>15}")
print("-"*85)

for idx, row in gdf_sorted.iterrows():
    tipo = row['TIPO'][:14]  # Limitar tamanho
    classe = row['CLASSE'][:29]  # Limitar tamanho
    print(f"{tipo:<15} {classe:<30} {row['AREA_KM2']:>11.4f} {row['PERIM_KM']:>14.2f}")

print("-"*85)

# Estatísticas resumo
print(f"\nRESUMO:")
print(f"  Área total: {gdf['AREA_KM2'].sum():.4f} km²")
print(f"  Perímetro total: {gdf['PERIM_KM'].sum():.2f} km")
print(f"  Maior área: {gdf_sorted.iloc[0]['TIPO']} ({gdf_sorted.iloc[0]['CLASSE']}) - {gdf_sorted.iloc[0]['AREA_KM2']:.4f} km²")
print(f"  Menor área: {gdf_sorted.iloc[-1]['TIPO']} ({gdf_sorted.iloc[-1]['CLASSE']}) - {gdf_sorted.iloc[-1]['AREA_KM2']:.4f} km²")
```

    
    =====================================================================================
    TABELA COMPARATIVA DAS CLASSES DE SOLO
    =====================================================================================
    
    TIPO            CLASSE                           ÁREA (km²)  PERÍMETRO (km)
    -------------------------------------------------------------------------------------
    LVAd59          Latossolos Vermelho-Amarelos      723.2797         291.11
    LVAd58          Latossolos Vermelho-Amarelos      194.1504         220.52
    LAd4            Latossolos Amarelos               121.6135          90.00
    LAd4            Latossolos Amarelos               120.6889         105.49
    LVAd58          Latossolos Vermelho-Amarelos       93.2573         113.92
    CXbd33          Cambissolos Háplicos               41.5411          47.01
    LAd4            Latossolos Amarelos                36.5367          47.95
    LVAd68          Latossolos Vermelho-Amarelos       35.5005          40.80
    LVAd68          Latossolos Vermelho-Amarelos       24.2821          25.36
    LVAd68          Latossolos Vermelho-Amarelos       18.1046          27.15
    LVAd68          Latossolos Vermelho-Amarelos       16.0714          45.81
    LAd4            Latossolos Amarelos                 3.3280           9.81
    CXbd33          Cambissolos Háplicos                2.9889          11.87
    LVAd73          Latossolos Vermelho-Amarelos        2.0571          11.44
    LAd4            Latossolos Amarelos                 1.3691           7.52
    CHd4            Cambissolos Húmicos                 0.8614          34.88
    LVAd73          Latossolos Vermelho-Amarelos        0.7294           5.30
    LAd4            Latossolos Amarelos                 0.4047           3.40
    CHd4            Cambissolos Húmicos                 0.1085           2.28
    LAd4            Latossolos Amarelos                 0.0608           1.17
    LVAd59          Latossolos Vermelho-Amarelos        0.0164           1.19
    LVAd58          Latossolos Vermelho-Amarelos        0.0025           0.34
    LVAd59          Latossolos Vermelho-Amarelos        0.0017           0.28
    LVAd59          Latossolos Vermelho-Amarelos        0.0007           0.16
    CXbd33          Cambissolos Háplicos                0.0007           0.15
    -------------------------------------------------------------------------------------
    
    RESUMO:
      Área total: 1436.9559 km²
      Perímetro total: 1144.93 km
      Maior área: LVAd59 (Latossolos Vermelho-Amarelos) - 723.2797 km²
      Menor área: CXbd33 (Cambissolos Háplicos) - 0.0007 km²


### Visualizamos os gráficos correspondentes


```python
# Criar figura com 3 subplots
fig, axes = plt.subplots(1, 3, figsize=(18, 6))
fig.suptitle('Características dos tipos de solo - Juiz de Fora', 
             fontsize=16, fontweight='bold')

# 1. Ranking de áreas
ax1 = axes[0]
cores_area = plt.cm.viridis(np.linspace(0, 1, len(gdf_sorted)))
bars1 = ax1.bar(range(len(gdf_sorted)), gdf_sorted['AREA_KM2'], 
                color=cores_area, edgecolor='black')
ax1.set_xticks(range(len(gdf_sorted)))
ax1.set_xticklabels(gdf_sorted['TIPO'], rotation=45, ha='right')
ax1.set_ylabel('Área (km²)', fontweight='bold')
ax1.set_title('Ranking por área', fontweight='bold')
ax1.grid(True, alpha=0.3, axis='y')

# Adicionar valores
for i, (bar, area) in enumerate(zip(bars1, gdf_sorted['AREA_KM2'])):
    ax1.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 1,
             f'{area:.1f}', ha='center', va='bottom', fontweight='bold', fontsize=9)

# 2. Ranking de perímetros
ax2 = axes[1]
gdf_sorted_perim = gdf.sort_values('PERIM_KM', ascending=False)
cores_perim = plt.cm.plasma(np.linspace(0, 1, len(gdf_sorted_perim)))
bars2 = ax2.bar(range(len(gdf_sorted_perim)), gdf_sorted_perim['PERIM_KM'], 
                color=cores_perim, edgecolor='black')
ax2.set_xticks(range(len(gdf_sorted_perim)))
ax2.set_xticklabels(gdf_sorted_perim['TIPO'], rotation=45, ha='right')
ax2.set_ylabel('Perímetro (km)', fontweight='bold')
ax2.set_title('Ranking por perímetro', fontweight='bold')
ax2.grid(True, alpha=0.3, axis='y')

# Adicionar valores
for i, (bar, perim) in enumerate(zip(bars2, gdf_sorted_perim['PERIM_KM'])):
    ax2.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 5,
             f'{perim:.1f}', ha='center', va='bottom', fontweight='bold', fontsize=9)

# 3. Gráfico de pizza - proporção de áreas
ax3 = axes[2]
cores_pizza = plt.cm.Set3(np.linspace(0, 1, len(gdf)))
wedges, texts, autotexts = ax3.pie(gdf['AREA_KM2'], 
                                   labels=gdf['TIPO'],
                                   autopct='%1.1f%%',
                                   colors=cores_pizza,
                                   startangle=90)
ax3.set_title('Distribuição da área total', fontweight='bold')

# Ajustar tamanho dos textos do gráfico de pizza
for text in texts:
    text.set_fontsize(8)
for autotext in autotexts:
    autotext.set_fontsize(8)
    autotext.set_fontweight('bold')

plt.tight_layout()
#plt.savefig('ranking_morfologia.png', dpi=300, bbox_inches='tight')
plt.show()
```


    
![png](output_24_0.png)
    


### Criamos o mapa estático


```python
# 1. CORREÇÃO DAS GEOMETRIAS INVÁLIDAS

# Função para validar e corrigir geometrias
def validate_geometry(geom):
    if geom is None or geom.is_empty:
        return None
    if not geom.is_valid:
        try:
            # Tentar buffer(0) para corrigir geometrias inválidas
            return geom.buffer(0)
        except:
            try:
                # Se buffer(0) falhar, tentar simplificar
                return geom.simplify(0.001)
            except:
                return None
    return geom

# Aplicar correção às geometrias
gdf_sorted['geometry'] = gdf_sorted['geometry'].apply(validate_geometry)

# Remover geometrias que não puderam ser corrigidas
gdf_sorted = gdf_sorted[gdf_sorted['geometry'].notnull()]


# 2. CÁLCULO SEGURO DO CENTRO

def safe_centroid(geom):
    try:
        if geom is not None and not geom.is_empty:
            return geom.centroid
    except:
        pass
    return None

# Calcular centroides seguros
centroids = gdf_sorted['geometry'].apply(safe_centroid)
valid_centroids = centroids[centroids.notnull()]

if len(valid_centroids) > 0:
    center_lat = np.mean([c.y for c in valid_centroids if c is not None])
    center_lon = np.mean([c.x for c in valid_centroids if c is not None])
else:
    # Fallback: usar bounds
    bounds = gdf_wgs84.total_bounds  # [minx, miny, maxx, maxy]
    center_lat = (bounds[1] + bounds[3]) / 2
    center_lon = (bounds[0] + bounds[2]) / 2
    
# 3. CRIAR MAPA COM FOLIUM

# Criar mapa base
m = folium.Map(
    location=[center_lat, center_lon],
    zoom_start=10,
    tiles='CartoDB positron'  # Tile mais leve
)

# 4. ADICIONAR CAMADAS POR TIPO DE SOLO

# Cores para cada tipo de solo (códigos TIPO)
# Usando uma paleta de cores para solos (tons terrosos e naturais)
tipo_cores = {
    'CHd4': '#8B4513',      # Marrom (SaddleBrown) - Cambissolos Húmicos
    'CXbd33': '#CD853F',    # Marrom claro (Peru) - Cambissolos Háplicos
    'LAd4': '#DAA520',      # Dourado (GoldenRod) - Latossolos Amarelos
    'LVAd58': '#B22222',    # Vermelho tijolo (FireBrick) - Latossolos V-A
    'LVAd59': '#A52A2A',    # Marrom avermelhado (Brown) - Latossolos V-A
    'LVAd68': '#8B0000',    # Vermelho escuro (DarkRed) - Latossolos V-A
    'LVAd73': '#800000',    # Marrom vinho (Maroon) - Latossolos V-A
    'default': '#696969'    # Cinza escuro (DimGray) - para outros tipos
}

# Criar FeatureGroups para cada tipo de solo
grupos = {}
for tipo in gdf_sorted['TIPO'].unique():
    if tipo in tipo_cores:
        grupos[tipo] = folium.FeatureGroup(name=f"{tipo} - {gdf_sorted[gdf_sorted['TIPO']==tipo].iloc[0]['CLASSE']}")

# Adicionar geometrias
for idx, row in gdf_sorted.iterrows():
    try:
        tipo = row['TIPO']
        classe = row['CLASSE']
        cor = tipo_cores.get(tipo, tipo_cores['default'])
        
        # Criar popup com informações do solo
        popup_text = f"""
        <div style="font-family: Arial; min-width: 200px;">
            <b style="font-size: 14px;">{classe}</b><br>
            <hr style="margin: 5px 0;">
            <b>Código:</b> {tipo}<br>
            <b>Área:</b> {row['SHAPE_Area']:,.2f} m² ({row['AREA_KM2']:.4f} km²)<br>
            <b>Perímetro:</b> {row['SHAPE_Leng']:,.2f} m ({row['PERIM_KM']:.2f} km)
        </div>
        """
        
        # Adicionar ao grupo apropriado
        folium.GeoJson(
            row.geometry,
            style_function=lambda x, cor=cor: {
                'fillColor': cor,
                'color': 'black',
                'weight': 0.8,
                'fillOpacity': 0.7
            },
            popup=folium.Popup(popup_text, max_width=300),
            tooltip=f"{tipo} - {classe}"
        ).add_to(grupos.get(tipo, m))
        
    except Exception as e:
        print(f"Aviso: Não foi possível adicionar feição {idx}: {e}")

# Adicionar grupos ao mapa
for grupo in grupos.values():
    grupo.add_to(m)



# Adicionar plugin de tela cheia
fullscreen_plugin = Fullscreen(
    position='bottomleft',
    title='Expandir tela',
    title_cancel='Sair da tela cheia',
    force_separate_button=True
).add_to(m)

# 5. ADICIONAR CONTROLES

# Adicionar controle de camadas
folium.LayerControl(collapsed=False).add_to(m)

# Adicionar escala
plugins.MousePosition().add_to(m)

# Adicionar minimapa
minimap = plugins.MiniMap()
m.add_child(minimap)

# Adicionar legenda HTML para solos
legend_html = """
<div style="position: fixed; bottom: 50px; left: 50px; z-index: 1000; background-color: white; 
     padding: 15px; border: 2px solid #8B4513; border-radius: 5px; font-family: Arial; font-size: 13px;
     box-shadow: 3px 3px 5px rgba(0,0,0,0.3);">
    <b style="font-size: 15px; color: #4A3728;">LEGENDA - CLASSES DE SOLO</b><br>
    <hr style="margin: 5px 0 10px 0;">
    <div style="display: grid; grid-template-columns: 20px 1fr; gap: 5px;">
        <div><i style="background: #8B4513; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>CHd4</b> - Cambissolos Húmicos</div>
        <div><i style="background: #CD853F; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>CXbd33</b> - Cambissolos Háplicos</div>
        <div><i style="background: #DAA520; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>LAd4</b> - Latossolos Amarelos</div>
        <div><i style="background: #B22222; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>LVAd58</b> - Latossolos Vermelho-Amarelos</div>
        <div><i style="background: #A52A2A; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>LVAd59</b> - Latossolos Vermelho-Amarelos</div>
        <div><i style="background: #8B0000; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>LVAd68</b> - Latossolos Vermelho-Amarelos</div>
        <div><i style="background: #800000; width: 15px; height: 15px; display: inline-block; border: 1px solid black;"></i></div> <div><b>LVAd73</b> - Latossolos Vermelho-Amarelos</div>
    </div>
    <hr style="margin: 10px 0 5px 0;">
    <div style="text-align: center; font-size: 11px; color: #666;">
        Fonte: Sistema Brasileiro de Classificação de Solos (SiBCS)<br>
        Embrapa Solos
    </div>
</div>
"""

m.get_root().html.add_child(folium.Element(legend_html))

# 6. SALVAR MAPA
output_file = "mapa_solos_jf.html"
m.save(output_file)

print(f"Mapa salvo como: {output_file}")

# 7. VISUALIZAÇÃO ESTÁTICA SIMPLIFICADA

# Criar figura
fig, ax = plt.subplots(1, 1, figsize=(14, 10))

# Plotar mapa corrigido
gdf.plot(column='TIPO', 
         ax=ax, 
         legend=True,
         cmap='YlOrRd',  # Mapa de cores amarelo-laranja-vermelho (tons de solo)
         edgecolor='black',
         linewidth=0.7,
         alpha=0.8,
         legend_kwds={'title': 'Classes de Solo (Códigos)', 
                     'loc': 'center left', 
                     'bbox_to_anchor': (1, 0.5),
                     'fontsize': 10})

# Configurar título e rótulos
ax.set_title('Mapa de Solos - Juiz de Fora (MG)', fontsize=16, fontweight='bold')
ax.set_xlabel('Coordenada X (metros)', fontsize=12)
ax.set_ylabel('Coordenada Y (metros)', fontsize=12)
ax.grid(True, linestyle='--', alpha=0.3)

# Adicionar anotação com as classes completas
classes_text = "Classes: CHd4 (Cambissolos Húmicos) | CXbd33 (Cambissolos Háplicos) | LAd4 (Latossolos Amarelos) | LVAd58-73 (Latossolos Vermelho-Amarelos)"
ax.annotate(classes_text, xy=(0.02, 0.02), xycoords='axes fraction', 
            fontsize=9, bbox=dict(boxstyle="round,pad=0.3", facecolor="white", alpha=0.8))

# Ajustar layout
plt.tight_layout()

# Salvar
plt.savefig('mapa_solos.png', dpi=300, bbox_inches='tight', facecolor='white')
print("Mapa estático salvo como: mapa_solos.png")
plt.show()

```

    Mapa salvo como: mapa_solos_jf.html
    Mapa estático salvo como: mapa_solos.png



    
![png](output_26_1.png)
    


**Considerações finais**

Todo o conhecimento acumulado foi insuficiente para estimular políticas públicas capazes de evitar a tragédia provocada por fortes chuvas em fevereiro de 2026, em Juiz de Fora, que vitimou 65 pessoas e desalojou mais de seis mil moradores.

**Fontes**:

https://www.embrapa.br/solos/sibcs
https://www.pjf.mg.gov.br/desenvolvimentodoterritorio/sistema_informacoes/solos.php
https://www.infoteca.cnptia.embrapa.br/infoteca/bitstream/doc/1176834/1/Sistema-Brasileiro-de-Classificacao-de-Solos-2025.pdf

**Referências**

Eduardo, C. C. (2018). *Cartografia Geomorfológica Comparada: aplicações no município de Juiz de Fora(MG) como subsídio ao planejamento*. Dissertação (Mestrado acadêmico) em Geografia - Universidade Federal de Juiz de Fora, Juiz de Fora, 2018. Disponível em: https://repositorio.ufjf.br/jspui/handle/ufjf/6764?locale=pt_BR
