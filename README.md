# An�lise dos dados de interna��o do DATASUS

Este � um projeto de an�lise de dados com Power BI. O projeto � apresentado no BI, mas nas fases de obten��o e tratamento dos dados brutos fez-se necess�rio uso de Python e R.

Dados: consulta do banco de dados do Sistema de Informa��es Hospitalares do SUS (SIHSUS) do DATASUS. Abrangendo as interna��es hospitalares realizadas no �mbito do Sistema �nico de Sa�de no estado do Rio de Janeiro, em dois per�odos diferentes: Janeiro de 2012 a Setembro de 2013 e Janeiro de 2022 a Setembro de 2023.

Quer�amos saber se aumentou o n�mero de doen�as que j� haviam sido extintas por conta da imuniza��o mas que com a baixa da cobertura poderiam estar voltando, tem um movimento no mundo de baixa de cobertura que pode ter sido agravada com movimento anti vacina, p�s covid, dissemina��o de fake news e se acompanhava um movimento de queda que est� acontecendo ao redor do mundo desde 2014 segundo os dados do IPEA.  

Hipotese: com a queda da cobertura pelo SUS, pelo pouco n�mero de pessoas se vacinando algumas doen�as que j� foram exterminadas estariam voltando, doen�as como caxumba, hepatite, tuberculose e sarampo


# Obten��o dos dados via FTP do datasus
Os dados podem ser extra�dos manualmente atrav�s do site do [TabWin](https://datasus.saude.gov.br/transferencia-de-arquivos/), selecionando SIHSUS e o per�odo desejado, mas como precisavamos de um per�odo extenso, utilizamos fun��es para extra��o dos dados de forma mais eficiente, vide c�digo em "coleta_dados.ipynb".

# Convers�o dos dados .DBC para .CSV utilizando R no Windows
!! Aten��o: substituir caminhos de arquivo pelos seus !!  

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
L� a pasta de arquivos dbc com um FOR, transforma um por um em csv e vai guardando na pasta de arquivos csv  
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
Descobrimos que grande parte dos dados vinham em c�digos, sendo assim, precisamos da documenta��o para tratar os dados, uma das fontes de consulta foi este [link](https://pcdas.icict.fiocruz.br/conjunto-de-dados/sistema-de-informacoes-hospitalares-do-sus-sihsus/documentacao/).



1 - Tratamento da coluna "SEXO"  
Atrav�s dos dados que encontramos sobre a coluna, que s� tinha dado num�rico, vimos que Masculino = 1, Feminino = 2 ou 3 e o resto seria ignorado.

2 - Tratamento da coluna "DIAG_PRINC", referente ao diagn�stico principal.
Seguindo o mesmo [link](https://pcdas.icict.fiocruz.br/conjunto-de-dados/sistema-de-informacoes-hospitalares-do-sus-sihsus/documentacao/), achamos as informa��es necess�rias para tratar os dados. Vimos que era necess�rio fazer o download de tabelas auxiliares para interpretar os c�digos, nesse caso utilizamos as [tabela cap�tulos](https://github.com/bigdata-icict/ETL-Dataiku-DSS/blob/master/SIM/cid10_tabela_capitulos.csv) e [tabela subcategoria](https://github.com/bigdata-icict/ETL-Dataiku-DSS/blob/master/SIM/CID-10-SUBCATEGORIAS.CSV.utf8) para gerar duas colunas, fazendo left join com os dados existentes, vide c�digo em "datasus_bronze.ipynb".  
Informa��es: A codifica��o do CID, presente na coluna "DIAG_PRINC", � alfanum�rica e envolve uma letra e 4 n�meros. Cada condi��o de sa�de � atribu�da a um c�digo espec�fico para facilitar a identifica��o. [Dicion�rio CID](https://www.medicinanet.com.br/cid10/b.htm)  

3 - Ap�s o restante dos tratamentos, podemos escolher como separar os dados. A escolha foi um arquivo �nico com ambos per�odos e colunas selecionados para a an�lise.


# Conclus�o

Valores como caxumba teve pouco aumento e tuberculose um aumento de mais de 15%.   
Hepatite caiu e n�o teve caso de interna��o por sarampo, mas se o gr�fico mostra que parece ter tido uma queda em outras doen�as, se a gente aplica o filtro de idade, s�o pessoas mais velhas que est�o sendo internadas.  
O v�rus pode estar circulando pelas crian�as que n�o est�o sendo vacinadas e que vivem em contato com essas pessoas mais velhas.   
Ao longo do tempo, se isso tiver certo tb a m�dia de interna��o vai cair mais, em compensa��o, o aumento de tuberculose � s�rio e pode estar associado ainda a complica��es da covid, cuja taxa de vacina��o � baixa.

O projeto chegou a algumas an�lises interessantes, mas ainda h� muito para explorar. Como s� foi utilizado dados de interna��o, muito provavelmente diversos casos sem complica��es suficientes para tal est�o em outra base de dados.
Em uma futura explora��o, com mais dados, d� pra ter um melhor entendimento sobre as hip�teses em mente.
