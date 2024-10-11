# Questao1-Prova-EspecialistaBI

# Sobre a questão

Utilizando os dados extraídos e transformados dos arquivos CSV (ORCAMENTO, DESPESAS, CENTRO_DE_CUSTO, CONTAS), você foi designado para desenvolver uma stored procedure(s) complexa (s) no SQL Server. Esta(s) procedure(s) será responsável por inserir e atualizar dados na tabela DespesasDetalhadas, que consolida informações financeiras detalhadas de diversas categorias de despesas.
Desenvolva uma stored procedure chamada sp_InserirAtualizarDespesasDetalhadas. Esta procedure deve aceitar dados de entrada
compatíveis com o formato e estrutura dos arquivos CSV DESPESAS.

A procedure deve inserir novos registros ou atualizar registros existentes baseando-se em uma chave única, como um identificador de despesa ou uma combinação de data e categoria.

Implementação de Transações:
Implemente transações para garantir a integridade dos dados. Cada inserção ou atualização de registro deve ser tratada como uma única operação atômica.
Inclua mecanismos de tratamento de erros dentro da transação para lidar com possíveis falhas, utilizando TRY...CATCH para reverter transações em caso de erro e registrar a ocorrência de falhas.

# Código utilizado para solução

```SQL

CREATE PROCEDURE sp_InserirAtualizarDespesasDetalhadas
    @DTBASE DATE,
    @CODIGOCENTROCUSTO INT,
    @CENTROCUSTOMASTER NVARCHAR(100),
    @VLDESPESA DECIMAL(18, 2),
    @CODFILIALPRINCIPAL INT,
    @CODCONTA NVARCHAR(50),
    @MES INT
AS
BEGIN
    -- Início da transação
    BEGIN TRANSACTION;

    BEGIN TRY
        -- Verifica se já existe um registro com base em uma combinação de DTBASE, CODIGOCENTROCUSTO e CODCONTA
        IF EXISTS (SELECT 1 FROM DespesasDetalhadas 
                   WHERE DTBASE = @DTBASE 
                     AND CODIGOCENTROCUSTO = @CODIGOCENTROCUSTO 
                     AND CODCONTA = @CODCONTA)
        BEGIN
            -- Atualiza o registro existente
            UPDATE DespesasDetalhadas
            SET CENTROCUSTOMASTER = @CENTROCUSTOMASTER,
                VLDESPESA = @VLDESPESA,
                CODFILIALPRINCIPAL = @CODFILIALPRINCIPAL,
                MES = @MES,
                DataModificacao = GETDATE()
            WHERE DTBASE = @DTBASE
              AND CODIGOCENTROCUSTO = @CODIGOCENTROCUSTO
              AND CODCONTA = @CODCONTA;
        END
        ELSE
        BEGIN
            -- Insere um novo registro se não houver correspondência
            INSERT INTO DespesasDetalhadas (DTBASE, CODIGOCENTROCUSTO, CENTROCUSTOMASTER, VLDESPESA, CODFILIALPRINCIPAL, CODCONTA, MES, DataCriacao)
            VALUES (@DTBASE, @CODIGOCENTROCUSTO, @CENTROCUSTOMASTER, @VLDESPESA, @CODFILIALPRINCIPAL, @CODCONTA, @MES, GETDATE());
        END

        -- Confirma a transação
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        -- Em caso de erro, desfaz a transação
        ROLLBACK TRANSACTION;

        -- Registro de erro (você pode personalizar esta parte para gravar em uma tabela de logs)
        DECLARE @ErrorMessage NVARCHAR(4000);
        DECLARE @ErrorSeverity INT;
        DECLARE @ErrorState INT;

        SELECT @ErrorMessage = ERROR_MESSAGE(),
               @ErrorSeverity = ERROR_SEVERITY(),
               @ErrorState = ERROR_STATE();

        -- Lançar o erro novamente para propagar o erro ao chamador da procedure
        RAISERROR (@ErrorMessage, @ErrorSeverity, @ErrorState);
    END CATCH
END;
