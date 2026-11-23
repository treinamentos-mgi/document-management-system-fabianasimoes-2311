# Especificação - Document Management System

> Especificação (spec) do DMS, elaborada a partir de `docs/specs/spec-template.md`
> para o desenvolvimento guiado por especificação (Spec Driven Development).
> Esta spec contém apenas o planejamento: nenhum arquivo de backend ou frontend
> é criado ou alterado por ela.

## 1. Objetivo

Entregar uma aplicação web que permita a um usuário enviar, listar e baixar
documentos, armazenados localmente no servidor, com metadados mantidos em memória.

## 2. Escopo

### Dentro do escopo

- Upload de documentos
- Listagem de documentos
- Download de documentos
- Gestão simples por usuário (cada documento pertence a um dono)

### Fora do escopo

- Armazenamento externo ou em nuvem (S3, GCS, Azure Blob etc.)
- Serviços de upload de terceiros
- Versionamento de documentos
- Autenticação e autorização completas (login, senha, tokens)
- Persistência de metadados em banco de dados (fase futura)
- Edição, exclusão e compartilhamento de documentos

## 3. Requisitos funcionais

| ID    | Requisito                                                                                                  |
| ----- | ---------------------------------------------------------------------------------------------------------- |
| RF-01 | O usuário pode enviar um documento; o sistema grava o arquivo e registra seus metadados                    |
| RF-02 | O usuário pode listar os documentos de sua propriedade, em ordem de upload (mais recente primeiro)         |
| RF-03 | O usuário pode baixar um documento pelo identificador, recebendo o arquivo original                        |
| RF-04 | O sistema rejeita uploads sem arquivo ou com arquivo acima do tamanho máximo configurado                   |
| RF-05 | O sistema responde 404 quando o identificador não existe ou pertence a outro usuário                       |
| RF-06 | A interface web exibe a lista de documentos, permite selecionar um arquivo para envio e disparar o download |
| RF-07 | A interface exibe mensagens de erro e de sucesso em português                                              |

Decisão: a identificação do usuário nesta fase é feita pelo header `X-User-Id`
(sem autenticação). Requisições sem esse header são rejeitadas com 400.
O frontend envia o valor configurado no seu estado local.

## 4. Requisitos não funcionais

| ID     | Requisito                                                                                   |
| ------ | ------------------------------------------------------------------------------------------- |
| RNF-01 | Arquivos gravados no filesystem local da aplicação via `multer` com `diskStorage`, em `backend/storage` |
| RNF-02 | Metadados mantidos em memória nesta fase (perdidos ao reiniciar o processo)                 |
| RNF-03 | Configuração via variáveis de ambiente (12-Factor), com valores padrão seguros              |
| RNF-04 | Nenhum provedor de armazenamento externo ou serviço de upload de terceiros                  |
| RNF-05 | Nomes de símbolos em inglês; mensagens ao usuário e comentários em português                |
| RNF-06 | Não expor o caminho físico do arquivo nas respostas da API                                  |
| RNF-07 | Backend testado com o runner nativo `node:test`                                             |

### Variáveis de ambiente

| Variável           | Padrão           | Descrição                                          |
| ------------------ | ---------------- | -------------------------------------------------- |
| `PORT`             | `3000`           | Porta HTTP do backend                              |
| `UPLOAD_DIR`       | `backend/storage` | Pasta onde os arquivos são gravados              |
| `MAX_FILE_SIZE_MB` | `10`             | Tamanho máximo de cada arquivo, em MB              |

## 5. Modelo de dados (metadados do documento)

| Campo        | Tipo   | Descrição                                                        |
| ------------ | ------ | ---------------------------------------------------------------- |
| id           | string | Identificador único do documento (UUID v4)                       |
| originalName | string | Nome original do arquivo enviado                                 |
| size         | number | Tamanho em bytes                                                 |
| mimeType     | string | Tipo MIME informado no upload                                    |
| storedName   | string | Nome do arquivo gravado no disco (uso interno, não exposto)      |
| uploadedAt   | string | Data/hora do upload (ISO 8601, UTC)                              |
| owner        | string | Valor do header `X-User-Id` no momento do upload                 |

Armazenamento: um `Map` em memória no repositório, indexado por `id`.
Os arquivos ficam em `UPLOAD_DIR` com o nome `storedName`.

## 6. Contratos de API

Prefixo do frontend: `/api` (proxy do Vite para o backend). Os caminhos abaixo
são os do backend. Todas as respostas de erro usam o formato:

```json
{ "error": "Mensagem em português" }
{
  "id": "3f2c1a9e-8b7d-4e5f-9a0b-1c2d3e4f5a6b",
  "originalName": "contrato.pdf",
  "size": 20480,
  "mimeType": "application/pdf",
  "uploadedAt": "2026-11-23T19:15:30.000Z",
  "owner": "user-42"
}
[
  {
    "id": "3f2c1a9e-8b7d-4e5f-9a0b-1c2d3e4f5a6b",
    "originalName": "contrato.pdf",
    "size": 20480,
    "mimeType": "application/pdf",
    "uploadedAt": "2026-11-23T19:15:30.000Z",
    "owner": "user-42"
  }
]
