# SendNF MCP

Servidor MCP do [SendNF](https://sendnf.com/mcp/): emita NFS-e e NFC-e conversando com o seu agente de IA. Consulta de CNPJ, código de serviço, pré-emissão, emissão e cancelamento.

- URL: `https://api.sendnf.com/mcp` (Streamable HTTP, sem sessão)
- Autenticação: `Authorization: Bearer <chave>`. Chave `nt_test_...` é homologação (sem valor fiscal); `nt_live_...` é produção. Sem chave, o servidor responde 401.
- Documentação: https://sendnf.com/mcp/

Este repositório tem só o manifesto, exemplos e a lista de ferramentas. O código do produto é privado.

## Configuração

```json
{
  "mcpServers": {
    "sendnf": {
      "type": "http",
      "url": "https://api.sendnf.com/mcp",
      "headers": { "Authorization": "Bearer nt_test_SUA_CHAVE" }
    }
  }
}
```

## Ferramentas (36)

| Ferramenta | Tipo | O que faz |
|---|---|---|
| `account_overview` | leitura | Modo da chave (teste ou produção), plano, uso do mês, se a chave é restrita a uma empresa e as empresas cadastradas com o que falta em cada uma. |
| `usage_by_company` | leitura | Notas de cada empresa no período (por tipo, status e ambiente), notas reais autorizadas, se o CNPJ ficou ativo e a estimativa de cobrança de cada uma. |
| `go_live_status` | leitura | Por empresa: se o checklist da NFS-e está completo, se já passou uma nota de teste, se está ativada para emitir notas reais (live_enabled_at) e o estado |
| `lookup_cnpj` | leitura | Busca dados públicos da empresa na Receita (razão social, endereço, CNAE, Simples/MEI), descobre se a prefeitura usa o Sistema Nacional ou sistema próprio e sugere códigos de serviço pelo CNAE. |
| `search_service_codes` | leitura | Busca na lista nacional de serviços (LC 116) o código de tributação nacional (cTribNac, 6 dígitos) e o NBS. |
| `get_service_code` | leitura | Descrição oficial de um código de tributação nacional e os NBS possíveis. |
| `how_to_get` | leitura | Passo a passo para conseguir dados fiscais difíceis de achar (CSC por UF, inscrição municipal, senha da prefeitura), com fontes oficiais. |
| `list_companies` | leitura | Empresas emitentes da conta (id, CNPJ, nome, regime, cidade, external_id e metadata). |
| `find_company_by_external_id` | leitura | Dados e checklist da empresa pelo id dela no sistema do integrador (external_id). |
| `get_company` | leitura | Dados da empresa, dados públicos da Receita e o checklist do que falta para emitir NFS-e e NFC-e, com guia para cada pendência. |
| `create_company` | escrita | Cadastra a empresa emitente. |
| `import_invoice` | escrita | Lê o XML de uma nota que a empresa já emitiu (NFS-e Nacional ou municipal, NF-e ou NFC-e) e cadastra ou completa a empresa: inscrições, regime, código de serviço, NBS, alíquota do ISS, código municipal e a numeração (série e próximo número). |
| `update_company` | escrita | Atualiza dados da empresa (ex.: inscrição municipal, código de serviço padrão, CSC da NFC-e, senha da prefeitura, emits_nfce, external_id, metadata; null apaga external_id e metadata). |
| `upload_certificate` | escrita | Envia o certificado digital A1 (.pfx em base64 + senha). |
| `check_readiness` | leitura | O que falta para a empresa emitir, com guias. |
| `create_onboarding_link` | escrita | Gera um link curto da página de cadastro da empresa (com o whitelabel da conta, se ligado), para o próprio dono completar o que falta: certificado A1 e senha, |
| `get_branding` | leitura | Whitelabel da conta: se está ligado, nome, logo, cor, e-mail de suporte, responder-para, origens que podem incorporar o cadastro e o domínio de envio dos e-mails (com o status do DNS). |
| `create_share_link` | escrita | Cria um link seguro, só de leitura, para quem recebe (contabilidade, sócio ou outro sistema) ver e baixar as notas autorizadas e canceladas (PDF, XML e ZIP do mês), sem login. |
| `list_share_links` | leitura | Lista os links de compartilhamento de notas: empresa, período, validade, se está ativo, expirado ou revogado, último acesso e quantos acessos. |
| `revoke_share_link` | escrita | Revoga um link de compartilhamento: ele para de abrir na hora. |
| `preview_nfse` | leitura | Simula a NFS-e sem transmitir: valida contra o layout oficial (ou o provedor da prefeitura), calcula totais e IBS/CBS e lista problemas com guias. |
| `issue_nfse` | escrita | Emite a NFS-e e espera até ~25 s pelo resultado (authorized, rejected, blocked ou pending). |
| `preview_nfce` | leitura | Simula a NFC-e (venda de produto ao consumidor) sem transmitir e lista problemas com guias. |
| `issue_nfce` | escrita | Emite a NFC-e e espera até ~25 s pelo resultado. |
| `list_documents` | leitura | Notas do modo atual, mais recentes primeiro. |
| `get_document` | leitura | Status e detalhes de uma nota: motivo de rejeição explicado, blocked_reason, tentativas de envio (attempts, max_attempts, next_attempt_at, last_error), se ainda pode ser cancelada ou substituída (cancellation) e links de PDF/XML. |
| `create_export` | escrita | Pede a exportação em massa: um ZIP com o XML e o PDF de todas as notas do período (pastas por empresa e mês) e a planilha notas.csv/notas.xlsx (número, data, tomador, CPF/CNPJ, valor, ISS, retenções, situação, chave). |
| `get_export` | leitura | Estado de uma exportação (queued, running, ready, failed, expired), progresso (done/total) e, quando pronta, download_url (link assinado de 1 hora; o arquivo fica guardado 7 dias). |
| `get_document_events` | leitura | Linha do tempo da nota (criada, enviada, autorizada, rejeitada, bloqueada, cancelada…), para explicar ao usuário o que aconteceu com ela. |
| `cancel_document` | escrita | Cancela a nota inteira na Receita/prefeitura/SEFAZ. |
| `get_substitution_draft` | leitura | Diz se a NFS-e pode ser corrigida por substituição ("cancelar parcialmente") e devolve o corpo da original para editar e mandar ao substitute_document. |
| `substitute_document` | escrita | "Cancelar parcialmente" ou corrigir valor/dados de uma NFS-e: emite uma nota substituta com os itens e valores informados, apontando para a original, |
| `create_note_draft` | escrita | Monta uma NFS-e (kind nfse) ou NFC-e (kind nfce) e deixa aguardando aprovação humana, mesmo que a trava esteja desligada: valida como na emissão (cadastro, layout oficial, totais) e devolve o rascunho com id, status pending_approval, approval_url e, na NFS-e, preview_pdf_url (prévia sem valor fiscal). |
| `list_pending_approvals` | leitura | Rascunhos de emissão, cancelamento e substituição do modo atual, mais recentes primeiro. |
| `get_approval_status` | leitura | Estado do rascunho: pending_approval (esperando uma pessoa), approved (traz o documento emitido/cancelado), rejected (traz rejection_reason: corrija e monte de novo), expired (ninguém decidiu em 7 dias) ou failed (aprovado, mas a emissão deu erro; veja error). |
| `retry_document` | escrita | Transmite agora uma nota pending, blocked (depois de corrigir o que faltava) ou failed ("Não enviada", quando o SendNF desistiu de tentar) e espera até ~25 s pelo resultado. |
