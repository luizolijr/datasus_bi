# Análise dos dados de internação do DATASUS

Este é um projeto de análise de dados com Power BI. O projeto é apresentado no BI, mas nas fases de obtenção e tratamento dos dados brutos fez-se necessário uso de Python e R.

# Obtenção dos dados via FTP do datasus
Os dados podem ser extraídos manualmente através do site do [TabWin](https://datasus.saude.gov.br/transferencia-de-arquivos/), selecionando SIHSUS e o período desejado, mas como precisavamos de um período extenso, utilizamos funções para extração dos dados de forma mais eficiente, vide código em "coleta_dados.ipynb".

# Conversão dos dados .DBC para .CSV utilizando R no Windows
!! Atenção: substituir caminhos de arquivo pelos seus !!  

1 - Baixar a biblioteca [read.dbc](https://cran.r-project.org/src/contrib/Archive/read.dbc/) (baixei a mais recente)  

2 - Instalar a biblioteca  
path<-"C:/read.dbc_1.0.6.tar.gz"  
install.packages(path, repos = NULL, type="source")

3 - Criar path do arquivo dbc e rodar comandos  
path_dbc<-"C:/REPOS/data_sus/data_dbc/RDRJ2301.dbc"  
library(read.dbc)  
df<-read.dbc(path_dbc)  
head(df)

4 - Salvar em csv  
load_path<-"C:/REPOS/data_sus/data_dbc/RDRJ2301.dbc"  
export_path<-"C:/REPOS/data_sus/data_csv/RDRJ2301.csv"  
write.csv2(df, export_path)

5 - Transformar em grande escala - dbc em csv  
Lê a pasta de arquivos dbc com um FOR, transforma um por um em csv e vai guardando na pasta de arquivos csv  
dbc_folder <- "C:/REPOS/data_sus/data_dbc/"  
csv_folder <- "C:/REPOS/data_sus/data_csv/"  
for(f in files) {  
  print(f)  
  df = read.dbc(f)  
  lista = strsplit(f, "/")[[1]]  
  file = gsub(".dbc", ".csv", lista[length(lista)])  
  write.csv2(df, paste(csv_folder, file, sep="/"), row.names=FALSE)  
}

# Tratamento dos Dados
Descobrimos que grande parte dos dados vinham em códigos, sendo assim, precisamos da documentação para tratar os dados, uma das fontes de consulta foi este [link](https://pcdas.icict.fiocruz.br/conjunto-de-dados/sistema-de-informacoes-hospitalares-do-sus-sihsus/documentacao/).

1 - Tratamento da coluna "SEXO"  
Através dos dados que encontramos sobre a coluna, que só tinha dado numérico, vimos que Masculino = 1, Feminino = 2 ou 3 e o resto seria ignorado.

2 - Tratamento da coluna "DIAG_PRINC", referente ao diagnóstico principal.  
Seguindo o mesmo [link](https://pcdas.icict.fiocruz.br/conjunto-de-dados/sistema-de-informacoes-hospitalares-do-sus-sihsus/documentacao/), achamos as informações necessárias para tratar os dados. Vimos que era necessário fazer o download de tabelas auxiliares para interpretar os códigos, nesse caso utilizamos as [tabela capítulos](https://github.com/bigdata-icict/ETL-Dataiku-DSS/blob/master/SIM/cid10_tabela_capitulos.csv) e [tabela subcategoria](https://github.com/bigdata-icict/ETL-Dataiku-DSS/blob/master/SIM/CID-10-SUBCATEGORIAS.CSV.utf8) para gerar duas colunas, fazendo left join com os dados existentes, vide código em "datasus_bronze.ipynb".

3 - Após o restante dos tratamentos, podemos escolher como separar os dados. A escolha foi um arquivo único com ambos períodos e colunas selecionados para a análise.
