# Guia de Utilização KIS API - Java
***

# Introdução
A API de integração com o Kaffa Integration Server (KIS-API) permite a integração do Kaffa Espresso com sistemas externos de modo eficiente e simplificado. O objetivo deste documento é descrever o funcionamento da API e exemplificar sua utilização.

# Arquitetura
A API de integração do KIS foi projetada de forma a otimizar a integração de grande volumes de dados tabulares. Por dados tabulares, entenda-se dados orientados a registros formados por colunas, como por exemplo dados provenientes de bancos de dados relacionais ou de arquivos CSV.
A interface também permite a transferência eficiente de arquivos arbitrários sem overheads desnecessários bem como o uso de FTPs ou algo similar.

## Conceitos
### Serviço de integração - IntegrationServices
Responsável por permitir o acesso a uma instância do KIS e disponbilizar a interface para o envio e recebimento das informações.
O serviço de integração pode ser usado tanto no modo produção, conectado a um KIS ou em modo desenvolvimento, no qual é possível escrever e testar toda a lógica de integração sem a necessidade de um KIS ou de qualquer outro componente do Espresso ou da Kaffa. Com o uso do modo desenvolvimento (debug) ao fazer um envio de dados para o Espresso, é gerado um arquivo .ZIP que pode ser enviado por qualquer meio para a Kaffa dar andamento na parte que lhe cabe na integração, evitando interdependencias desnecessárias.
Através do IntegrationServices é possível obter os DatasetWriters e DatasetReaders que é por onde as informações são transferidas.

### Datasets
Toda a integração é orientada a *datasets*. Datasets permitem a modularização dos dados.
Toda integração começa pela definição dos datasets que existirão na personalização do Espresso. Um dataset possui um nome semelhante a um caminho
("dataset_group/dataset_name") e um conteúdo esperado (tabelas e arquivos/anexos).

Um cenário comum para soluções de Projeto e Execução de Obras é o seguinte:

    "regional/REGIONAL1"
    "regional/REGIONAL2"
    "projeto/ID_PROJETO_1"
    "projeto/ID_PROJETO_2"
    "geral/TABELAS_AUXILIARES"
    "geral/COMPATIBLE_UNITS"

Neste caso os datasets "regional/*" incluem todos os ativos GIS pertencentes a uma determinada regional. Neste dataset haverá uma tabela para cada tipo de entidade (POSTE, ESTRUTURA, REDEPRIMARIA, etc) e dentro das tabelas haverá os registros dos respectivos ativos. O ideal é que as tabelas sejam exatamente as mesmas já existentes no ambiente pré-Espresso, qualquer conversão necessária poderá ser feita dentro do Espresso.
Além de tabelas, poderão ser enviados anexos como CSVs, XMLs, XLS, fotos, Docs, etc.

### Tabelas
Tabelas possuem um nome ("POSTE", "gis.POSTE", "projeto.MATERIAL") e registros com colunas. Não é necessário definir os nomes e os tipos das colunas a priori para usar a API. As tabelas são dinâmicas sendo que as colunas são criadas automaticamente a partir do uso.

### Anexos
Anexos permitem a trânsferencia de arquivos junto ao dataset. Estes arquivos dependem de cada integração e de cada dataset. No caso de um dataset de projeto faz sentido a existência de anexos do projeto como fotos, diagramas, instruções gerais. Já num dataset geral, faz sentido anexos de sistema como planilhas de cálculos mecânico ou lista de defeitos.

# Preparação do ambiente
O jar da API (kis-api-client-java-build-0.0.9.jar) deve ser adicionado ao build path do projeto que fará a integração com o KIS.

# Usando a API

## Iniciando a interface
Para iniciar a API, é preciso configurar a conexão com o servidor. Para isso, é preciso instanciar 2 objetos, nessa ordem:
1. IntegrationConfig
2. IntegrationServices

IntegrationConfig recebe as configurações da conexão com o servidor. Este pode ser remoto ou local. Se o servidor for remoto, então IntegrationConfig deve receber 3 parâmetros:
1. Host - endereço IP do servidor
2. Port - Porta de comunicação com o servidor
3. Port2 - Porta de comunicação com o servidor

Se for uma configuração local, IntegrationConfig recebe o caminho da pasta onde acontecerá a leitura e escrita de dados. Se esta pasta ainda não existir, ela será criada.

Após configurar a conexão, pode ser necessário configurar a pasta de desenvolvimento. Este passo é opcional pois a própria API se encarrega de criar uma pasta de Debug na pasta temporária do sistema chamada "Kis_Debug_Folder". Ela pode ser alterada chamando a função setDebugFolder da configuração criada passando o caminho da pasta a ser usada. Se a pasta ainda não existir, será criada.

IntegrationServices recebe como parâmetro o IntegrationConfig. Ele é responsável por criar os objetos de leitura e escrita para com o servidor.

#### Exemplo de configuração - Remoto
```java
String host = "127.0.0.1";
int port = 4045;
int port2 = 4046;

IntegrationConfig config = new IntegrationConfig(host, port, port2);
IntegrationService serv = new IntegrationService(config);
```
#### Exemplo de configuração - Local (utilizado em tempo de desenvolvimento)
```java
String path = "C:/Usuário/Pasta_Usuário"
IntegrationConfig configLocal = new IntegrationConfig(path);

IntegrationService servLocal = new IntegrationService(configLocal);
```

## Escrita de dados

Para a escrita de dados, é preciso criar um objeto do tipo DatasetWriter através do método buildDatasetWriter do service. Este método recebe como parâmetro o nome do dataset a ser criado e o seu modo de criação, que pode ser DatasetMode.Create ou DatasetMode.Update.

Exemplo:

```java
IDatasetWriter dtWriter = serv.buildDatasetWriter("TESTE_DATASET", DatasetMode.Create);
```

Com este DatasetWriter é possível criar um escritor para tabelas ou Attachments, dependendo da necessidade do programa.
### Tabelas
Para a escrita em tabelas, deve-se criar um objeto do tipo TableWriter a partir do método buildTableWriter do DatasetWriter. Esse método recebe o nome da tabela como parâmetro.

```java
ITableWriter tbWriter = dtWriter.buildTableWriter("TABLE_TESTE");
```

A escrita em tabela é feita linha a linha, isto é, deve-se definir o nome das colunas e seu valor para a linha atual e mandar gravar. Para iniciar a escrita de uma linha, chamamos o método beginRecord() do TableWriter definido. Isso prepara a tabela para receber dados. Depois, construímos os valores da linha associando cada valor com uma coluna através do método setColumnValue(columnName, value). Quando todos os valores da linha estiverem associados com uma coluna, podemos chamar o método postRecord() para escrever a linha na tabela. Quando a escrita estiver encerrada, chamamos o método close() para garantir que todos os dados foram escritos e para fechar a tabela.

Exemplo de uso:

```java
// primeiro registro
tbWriter.beginRecord();
tbWriter.setColumnValue("ID", 001);
tbWriter.setColumnValue("NOME", "TABELA_TESTE");
tbWriter.setColumnValue("CAPACIDADE", 10);
tbWriter.setColumnValue("COMPRIMENTO", 1.5);
tbWriter.postRecord();

// segundo registro
tbWriter.beginRecord();
//...
tbWriter.postRecord();
//...

// encerra a escrita
tbWriter.close();
```

### Attachments

Para a escrita de attachments, deve-se criar objeto do tipo AttachmentWriter a partir do método buildAttachment do DatasetWriter. Ess método recebe o nome do attachment como parâmetro.

```java
IAttachmentWriter atWriter = dtWriter.buildAttachmentWriter("TESTE_ATTACHMENT");
```


A classe AttachmentWriter oferece 3 formas de escrita:
- Através de um InputStream
- Através de um arquivo
- Passando um conjunto de bytes

Para usar um InputStream como método de entrada de dados, usa-se o método fromStream(InputStream) do AttachmentWriter passando um InputStream.

```java
atWriter.fromStream(inputStream);
```

Para usar um arquivo, usa-se o método fromFile(Path) passando uma String com o caminho onde o arquivo está armazenado.

```java
atWriter.fromFile(filePath);
```

## Leitura de dados
Depois da fase de configuração, para iniciar a leitura de dados é necessário criar um objeto do tipo DatasetReader a partir do método buildDatasetReader de service. Este método recebe apenas uma string que será o nome do dataset.

Exemplo:
```java
IDatasetReader dtReader = serv.buildDatasetReader("TESTE_DATASET");
```

O método buildDatasetReader irá requisitar ao servidor, ou buscar na pasta de debug (configurado em IntegrationConfig), pelo arquivo zip compactado com o nome do dataset pedido. Dentro deste zip, serão buscados todas as tabelas e attachments que existem neste dataset. Com o objeto criado, é possível recuperar a lista com todas as tabelas ou attachments encontrados. Se nenhum dado tiver sido escrito com esse dataset, então as listas estarão vazias.

Exemplo de utilização.

```java
IDatasetReader dtReader = serv.buildDatasetReader("TESTE_DATASET");
Map<String, ITableReader> tables = dtReader.getTableReaders();
Map<String, IAttachmentReader> attachments = dtReader.getAttachments();

for (Entry<String, ITableReader> entry : tables.entrySet()) {
    String tableName = entry.getKey();
    ITableReader tbReader = entry.getValue();

    System.out.debug(tableName);

	// itera por todos os registros da tabela
    while (tbReader.next()) {
    	// Obtém o valor da coluna de nome "ID"
        Integer id = (Integer) tbReader.getColumnValue("ID");
        System.out.debug("Record - ID: " + id);
    }
}
```


*
**Atenção**
Para recuperar dados das tabelas, deve-se utilizar o método getColumnValue, passando o nome da coluna, e fazer cast para o tipo da coluna desejado (Integer, String, Float, etc). Se a coluna não for encontrada, ou for de um tipo diferente do pedido, uma exception será lançada, portanto é recomendável encapsular tudo em um bloco try-catch para evitar problemas.*


## Leitura de attachments

```java
Map<String, IAttachmentReader> attachments = reader.getAttachments();
for (Entry<String, IAttachmentReader> entry : attachments.entrySet()) {
    String attachmentName = entry.getKey();

    IAttachmentReader attachment = entry.getValue();
    InputStream stream = attachment.getStream();
    // ler o stream com os bytes do arquivo
}
```
