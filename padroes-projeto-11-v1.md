# Plano de Padronização: Padrões de Projeto 11.v1

Este documento descreve os padrões arquiteturais e de codificação identificados no projeto `11autorizacaodiario.v1.a1` (CNU), servindo como referência para manutenção e criação de novos notebooks seguindo este modelo, sem as abstrações introduzidas na versão V2.

## 1. Identificação e Estrutura de Pastas
*   **Sufixo de Versão:** O projeto deve utilizar o sufixo `.v1.a1` no nome da pasta raiz.
*   **Nomenclatura de Pastas:** Utilizar prefixos alfabéticos e numéricos para indicar a ordem de execução:
    *   `A1.SETUP`: Configurações iniciais e scripts de limpeza.
    *   `B1..Bn`: Processos de carga Raw/Trusted (Incremental).
    *   `C1..Cn`: Criação de Dimensões.
    *   `D1..Dn`: Tabelas Work/Fato (Intermediárias).
    *   `E1..En`: Tabelas de Fato Finais.
    *   `x.tools`: Funções específicas do projeto (`x.project_functions`).
    *   `x.executor`: Scripts de orquestração.

## 2. Padrões de Notebooks (Databricks)
### 2.1 Invocação de Ferramentas
*   Utilizar **caminhos absolutos** para o comando `%run`.
*   Sempre incluir as ferramentas comuns e as funções do projeto:
    ```python
    # Ferramentas Comuns
    %run "/projetos/cnu/01comumtools.ferramentascomum/x.Parameters/x.convertobjparametersvariables.v1"
    
    # Funções do Projeto
    %run /projetos/cnu/11autorizacaodiario.v1.a1/x.tools/x.project_functions
    ```

### 2.2 Lógica de Data e Carga
*   **Cálculo de Data Inicial:** A lógica deve ser explícita no notebook, utilizando `max()` da tabela de destino para determinar o ponto de partida incremental.
*   **Parâmetros de Tabela:** Definir explicitamente `tableRoot`, `schema`, `tableDestino`, `key_delete` e `campoParticao`.

### 2.3 Execução de Carga
*   Utilizar a função `cargaIncrementalLake` da V1.
*   **Configuração de Colunas:** A lista de colunas deve ser definida manualmente no notebook para garantir o contrato de dados.

### 2.4 Pós-Processamento
*   Executar o comando `OPTIMIZE` explicitamente após a carga incremental.
*   Registrar dependências usando `lib_dependencia`.

## 3. Padrões de Funções (`x.project_functions`)
*   As funções devem ser autossuficientes e residir no arquivo `x.project_functions.py`.
*   **Carga Incremental:** Utilizar `cargaIncrementalLakeDiario` ou `cargaIncrementalLakeDiarioSemDuplicacoes` dependendo da necessidade de `dropDuplicates`.
*   **Tratamento de Arquivos:** As funções devem varrer o Data Lake usando `dbutils.fs.ls` iterando sobre a hierarquia `ano/mes/dia`.

## 4. O que NÃO usar (Diferenças da V2)
*   **Não usar** caminhos relativos (`../`).
*   **Não usar** a função `cargaIncrementalLakeV2`.
*   **Não usar** `calcular_data_inicio_incremental`.
*   **Não usar** a tabela de controle de arquivos `tb_ctrl_carga_incremental...`.

não pode usar SUB diretorio
Não pode usar espaço nos nomes das pastas e dos notebooks
 os nomes de arquivos e pastas
E construa observando os códigos python que está no arquuivo : C:\Codes\pv\semaforo\plan\ativo\referencias\python\semafbase.py

utilize o sisteminha a baixo q serve para validar nomes de pastas e arquivos e melhore ele caso necessario aqui e execute 

C:\Codes\pv\semaforo\plan\ativo\referencias\python\semafbase.py


    def validarNotebook(self,notebookname):
        """validar primeiro bloco do notebook

        Args:
            firstletter (string): validar se a primeira do notebok é valido

        Returns:
            bool: true=valido, false=invalido
        """
        import re
        validLetter = 0
        notebooknamevalido = 0
        firstletternotebook = ""
        notebooktitle = ""
        etapa = ""
        if "." in notebookname:
            splNot =notebookname.split(".") 
            if len(splNot) == 2:
                firstletternotebook, notebooktitle = notebookname.split('.')
                etapa = ""
                notebooknamevalido = 1
            elif len(splNot) >= 3:
                firstletternotebook, etapa, notebooktitle = notebookname.split('.',2)
                notebooknamevalido = 1
            else:
                firstletternotebook = ""
                etapa = ""
                notebooktitle = ""
            regex = re.compile(r"\b([a-vA-V]{1,5}[0-9]{1,5}?|[0-9]{1,5}?)\b")
            validLetter = 1 if bool(regex.fullmatch(firstletternotebook)) else 0
            if firstletternotebook.lower() in ('teste','temp'):
                validLetter = 0
        if etapa not in ('raw','work','delivery','trusted','shared', 'tools', 'testes','exemplo','validar'):
            erro = f"Notebook não contem as opções no nome A1.raw.exemplo, as opções são: ('raw','work','delivery','trusted','shared', 'tools', 'testes','exemplo','validar') :{notebookname} "
        return validLetter, firstletternotebook, notebooktitle, etapa, notebooknamevalido
    
