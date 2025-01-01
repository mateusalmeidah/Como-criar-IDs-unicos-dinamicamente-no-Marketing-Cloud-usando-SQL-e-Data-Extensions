# Como-criar-IDs-unicos-dinamicamente-no-Marketing-Cloud-usando-SQL-e-Data-Extensions

Passo 1: Estrutura da Data Extension
Primeiro, você precisará criar uma Data Extension que armazene os dados recebidos e associe um ID único. Suponha que a estrutura da DEX seja a seguinte:

ID (Primary Key, Auto Increment ou UniqueIdentifier)
EmailAddress
FullName
PhoneNumber
Passo 2: Atualização ou Inserção
A lógica geral será:

Receber os dados (parâmetros) em uma Data Extension temporária.
Verificar na DEX principal se o dado já existe (comparando Email, Nome ou Telefone).
Se encontrar um registro, retorna o ID.
Caso contrário, cria um novo registro com um novo ID.
Estratégia com SQL
Aqui está um exemplo de como implementar:

Tabela Temporária (TempData)
Crie uma tabela temporária para armazenar os dados recebidos. Essa DEX pode ter os campos:

EmailAddress (Nullable)
FullName (Nullable)
PhoneNumber (Nullable)
Tabela Final (FinalData)
A DEX principal será a FinalData:

ID (Unique Identifier, Auto Increment)
EmailAddress (Nullable)
FullName (Nullable)
PhoneNumber (Nullable)
Query 1: Encontrar Registros Existentes
Crie uma query para verificar se o registro já existe na DEX principal:

SELECT
    t.EmailAddress,
    t.FullName,
    t.PhoneNumber,
    f.ID AS ExistingID
FROM TempData t
LEFT JOIN FinalData f
    ON (
        (t.EmailAddress = f.EmailAddress AND t.EmailAddress IS NOT NULL) OR
        (t.FullName = f.FullName AND t.FullName IS NOT NULL) OR
        (t.PhoneNumber = f.PhoneNumber AND t.PhoneNumber IS NOT NULL)
    )


Query 2: Inserir Novos Registros
Crie uma segunda query para inserir novos registros quando o ID não for encontrado:


SELECT
    NEWID() AS ID, -- Cria um novo ID único
    t.EmailAddress,
    t.FullName,
    t.PhoneNumber
FROM TempData t
LEFT JOIN FinalData f
    ON (
        (t.EmailAddress = f.EmailAddress AND t.EmailAddress IS NOT NULL) OR
        (t.FullName = f.FullName AND t.FullName IS NOT NULL) OR
        (t.PhoneNumber = f.PhoneNumber AND t.PhoneNumber IS NOT NULL)
    )
WHERE f.ID IS NULL -- Insere apenas quando o ID não existe

Query 3: Atualizar Registros Existentes
Se precisar atualizar os registros com novos valores (caso sejam enviados), pode fazer algo assim:

sql
Copiar código
UPDATE FinalData
SET
    EmailAddress = COALESCE(t.EmailAddress, f.EmailAddress),
    FullName = COALESCE(t.FullName, f.FullName),
    PhoneNumber = COALESCE(t.PhoneNumber, f.PhoneNumber)
FROM FinalData f
INNER JOIN TempData t
    ON f.ID = t.ID

Passo 3: Automação no Marketing Cloud
Use o Automation Studio para automatizar as etapas:

Importar Dados para a tabela temporária.
Rodar as queries em sequência:
Query 1: Para encontrar os registros existentes.
Query 2: Para inserir novos registros.
Query 3 (opcional): Para atualizar os registros existentes.

