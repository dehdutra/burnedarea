README

Monitoramento de Áreas Queimadas com Google Earth Engine e Geemap

Este script utiliza o Google Earth Engine (GEE) e a biblioteca Geemap para visualizar e exportar dados de áreas queimadas no Brasil, utilizando imagens do conjunto de dados MODIS (MCD64A1). O código autentica o usuário, carrega um grid específico, processa imagens filtradas por data e exporta os resultados para o Google Drive.

Requisitos

Antes de executar o script, certifique-se de que você possui:

Uma conta no Google Earth Engine

A biblioteca geemap instalada: pip install geemap

Autenticação no GEE utilizando ee.Authenticate()

Funcionamento do Código

1. Autenticação e Inicialização do GEE

O script começa autenticando e inicializando o GEE com um projeto específico:

import ee
import geemap

ee.Authenticate()
ee.Initialize(project='ee-seuusuario')

2. Carregamento do Grid e Definição do Conjunto de Dados

Carrega um grid geográfico (um asset no GEE) e define o conjunto de dados MODIS:

grid = ee.FeatureCollection('seuasset')

dataset = ee.ImageCollection('MODIS/061/MCD64A1')\
    .filter(ee.Filter.date('2023-01-01', '2023-12-31'))

3. Processamento e Visualização de Áreas Queimadas

Seleciona a camada de data de queimadas e aplica um esquema de cores para visualização:

burned_area = dataset.select('BurnDate')

burned_area_vis = {
    'min': 30.0,
    'max': 341.0,
    'palette': ['4e0400', '951003', 'c61503', 'ff1901']
}

4. Criação do Mapa e Recorte por Região

O código cria um mapa interativo e recorta os dados ao grid especificado:

Map = geemap.Map()
Map.setCenter(6.746, 46.529, 2)

burned_area_clipped = burned_area.map(lambda img: img.clip(grid))
Map.addLayer(burned_area_clipped, burned_area_vis, 'Burned Area')
Map.addLayer(grid, {}, 'Brasil Grid')

5. Adição de Camadas Mensais

Para facilitar a análise temporal, o código adiciona camadas mensais de áreas queimadas:

def add_monthly_layers(start_year, start_month, end_year, end_month):
    for year in range(start_year, end_year + 1):
        for month in range(start_month, end_month + 1):
            start_date = f"{year}-{month:02d}-01"
            end_date = f"{year}-{month:02d}-31"
            dataset = ee.ImageCollection('MODIS/061/MCD64A1')\
                .filter(ee.Filter.date(start_date, end_date))
            burned_area = dataset.select('BurnDate')
            burned_area_clipped = burned_area.map(lambda img: img.clip(grid))
            Map.addLayer(burned_area_clipped, burned_area_vis, f'{year}-{month:02d}')

6. Exportação das Imagens como GeoTIFF

O código também permite exportar os dados como arquivos GeoTIFF para o Google Drive:

def add_monthly_layers_and_export(start_year, start_month, end_year, end_month):
    for year in range(start_year, end_year + 1):
        for month in range(start_month, end_month + 1):
            start_date = f"{year}-{month:02d}-01"
            end_date = f"{year}-{month:02d}-31"
            dataset = ee.ImageCollection('MODIS/061/MCD64A1')\
                .filter(ee.Filter.date(start_date, end_date))
            burned_area = dataset.select('BurnDate')
            burned_area_clipped = burned_area.map(lambda img: img.clip(grid))
            export_task = ee.batch.Export.image.toDrive(
                image=burned_area_clipped.median(),
                description=f"Burned_Area_{year}-{month:02d}",
                folder='Burned_Area_TIFs',
                fileNamePrefix=f"Burned_Area_{year}-{month:02d}",
                region=grid.geometry(),
                scale=500,
                fileFormat='GeoTIFF',
                maxPixels=1e8
            )
            export_task.start()

7. Chamada das Funções

O script adiciona as camadas mensais ao mapa e inicia a exportação:

add_monthly_layers(2023, 1, 2023, 12)
add_monthly_layers_and_export(2023, 1, 2023, 12)

Resultado

O código gera um mapa interativo mostrando as áreas queimadas ao longo do tempo e exporta os dados para análise posterior no Google Drive.

Contato

Caso tenha dúvidas ou sugestões, entre em contato.
