---
title: Diagrama de Classes Refatorado (Object Calisthenics)
---
classDiagram
    direction TD

    %% =================================
    %% VALUE OBJECTS (Regra #3)
    %% =================================
    class CPF {
        -String numero
        +CPF(String numero)
        +String getFormatado()
        -boolean ehValido(String)
    }
    class RG {
        -String numero
    }
    class Email {
        -String endereco
        +Email(String endereco)
        -boolean ehValido(String)
    }
    class Telefone {
        -String ddd
        -String numero
    }
    class Senha {
        -String hash
        +Senha(String senhaPura)
        +boolean corresponde(String senhaPura)
    }
    class Dinheiro {
        -BigDecimal quantia
        -Currency moeda
        +Dinheiro somar(Dinheiro)
        +Dinheiro subtrair(Dinheiro)
    }
    class Endereco {
        -String logradouro
        -String bairro
        -String cidade
        -String estado
        -String cep
    }

    %% =================================
    %% ENTIDADES E AGREGAÇÕES (Regras #7, #8, #9)
    %% =================================
    class Person {
        <<Abstract>>
        -NomeCompleto nome
        -DocumentosPessoais documentos
        -InformacoesContato contato
        -LocalDate dataNascimento
    }
    note for Person "Classe base abstrata"

    class DocumentosPessoais {
        -CPF cpf
        -RG rg
    }

    class InformacoesContato {
        -Email email
        -Telefone telefonePrincipal
    }

    class Member {
        -boolean ativo
        -String imagemUrl
        -String notas
    }
    note for Member "Membro da organização"

    class Worker {
        -LocalDate dataAdmissao
        -String cargo
    }
    note for Worker "Trabalhador/Funcionário"

    class User {
        -NomeCompleto nome
        -Email email
        -Senha senha
        +boolean autenticar(Senha senha)
    }

    class Department {
        -String nome
        -String descricao
        +void adicionarMembro(Member membro)
    }
    note for Department "Departamento da organização"

    class Role {
        -String nome
    }

    class Category {
        -String nome
        -String descricao
    }

    class PaymentMethod {
        -String nome
        -String descricao
    }

    %% =================================
    %% POLIMORFISMO (Regra #2)
    %% =================================
    class Transaction {
        <<Interface>>
        +Dinheiro valor()
        +Balanco aplicarEm(Balanco balanco)
    }
    class CreditTransaction {
        -Dinheiro quantia
        +Dinheiro valor()
        +Balanco aplicarEm(Balanco balanco)
    }
    class DebitTransaction {
        -Dinheiro quantia
        +Dinheiro valor()
        +Balanco aplicarEm(Balanco balanco)
    }
    class FinancialRecord {
        -String descricao
        -LocalDate data
        -Transaction transacao
    }
    note for FinancialRecord "Representa um lançamento financeiro"


    %% =================================
    %% COLEÇÕES DE PRIMEIRA CLASSE (Regra #4)
    %% =================================
    class Roles {
        -Set~Role~ roles
        +void adicionar(Role role)
        +boolean temPermissao(Permissao p)
    }

    class DepartmentMembers {
        -Set~Member~ members
        +void adicionar(Member member)
        +int totalAtivos()
    }

    %% =================================
    %% OUTRAS ENTIDADES (Logs etc)
    %% =================================
    class LoginLog {
        -User user
        -LocalDateTime timestamp
        -boolean sucesso
        -String ip
    }

    class AuditLog {
        -User user
        -LocalDateTime timestamp
        -String acao
    }


    %% =================================
    %% RELACIONAMENTOS
    %% =================================

    ' Herança
    Person <|-- Member
    Person <|-- Worker
    Transaction <|.. CreditTransaction
    Transaction <|.. DebitTransaction

    ' Composição (entidades compostas por outras)
    Person "1" *-- "1" DocumentosPessoais : compõe
    Person "1" *-- "1" InformacoesContato : compõe
    Person "1" *-- "1" Endereco : possui

    DocumentosPessoais "1" *-- "1" CPF
    DocumentosPessoais "1" *-- "1" RG

    InformacoesContato "1" *-- "1" Email
    InformacoesContato "1" *-- "1" Telefone

    User "1" *-- "1" Senha
    User "1" *-- "1" Roles : gerencia

    Department "1" *-- "1" DepartmentMembers : gerencia

    FinancialRecord "1" *-- "1" Transaction : registra

    ' Associação (entidades se relacionam)
    User "1" -- "0..*" LoginLog : registra
    User "1" -- "0..*" AuditLog : realiza
    User "1" -- "0..*" FinancialRecord : realizou

    Roles "1" -- "0..*" Role : contém
    DepartmentMembers "1" -- "0..*" Member : contém
    Department "1" -- "0..*" Member : é composto por

    FinancialRecord "1" -- "1" Category : pertence a
    FinancialRecord "1" -- "1" PaymentMethod : usa
