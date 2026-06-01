# VetRoutine — Prompt de Inicialização para Claude Code

> Cole este prompt na primeira mensagem de uma sessão do Claude Code para retomar o projeto exatamente do ponto atual (v1.10+).
> Última atualização: 02/06/2026

---

## Contexto do projeto

Você vai continuar o desenvolvimento do **UltraVet**, um aplicativo HTML single-file de gestão para **Patrine Cagnini — Médica Veterinária | Ultrassonografista | CRMV-SC 09377**.

O app roda no navegador e persiste dados via `localStorage` (cache local) **e Supabase** (nuvem, sincronização multi-dispositivo). O entregável é sempre **um único arquivo HTML** (`ultravet_10.html`, incrementando a versão).

**Repositório:** https://github.com/joaocagnini/Ultravet  
**Deploy (Netlify):** atualiza automaticamente a cada push no branch `main`

---

## Arquivo de referência

O arquivo atual é `ultravet_9.html` (~1640 linhas), estruturado assim:

| Seção | Linhas aprox. |
|-------|--------------|
| CSS — variáveis, layout, componentes | 8–192 |
| HTML — sidebar, dashboard, páginas | 193–780 |
| JavaScript — estado, lógica, persistência | 790+ |

**Variáveis de estado globais:**
```js
let laudos = [], receitas = [], despesas = [], clinicas = [];
let editId = null, viewId = null, curFSec = "f-overview", curRpSec = "rp-laudos";
let fotosTmp = []; // fotos do laudo em edição (base64)
```

**Chaves do localStorage:**
- `uv_laudos` — laudos (campo `fotos[]` em base64)
- `uv_receitas`
- `uv_despesas`
- `uv_clinicas`

**Tabelas Supabase (projeto: iisxmfepvqpswdnurtuo):**
- Todas com schema: `id bigint PRIMARY KEY, payload jsonb`
- Tabelas: `uv_laudos`, `uv_receitas`, `uv_despesas`, `uv_clinicas`
- RLS desabilitado
- Chave: `sb_publishable_4yj6xy0vvxLypajgqtEMCQ_fR6nE2DY`

---

## O que já está implementado (v1.9)

### Pilar 1 — Laudos
- Formulário completo: abdominal total e gestacional
- Achados em 3 colunas: Órgão | Select de patologia | Texto livre (preenchido automaticamente)
- Salvar, editar, excluir, pré-visualizar
- Ao salvar laudo com valor > 0 → lança receita automaticamente no financeiro
- Busca por paciente, tutor ou veterinário; filtros por tipo e resultado
- Impressão formatada (abre janela para impressão/PDF)
- Layout do laudo impresso: cabeçalho com grid de dados + logo, paleta verde `#3d7a52`
- **Card "Fotos do exame"**: upload múltiplo, grid de preview, salvo em base64, impresso em 2 colunas grandes (340px, object-fit contain) ao final do laudo

### Pilar 2 — Financeiro
- Visão geral: métricas do mês + gráfico fluxo de caixa (6 meses) + receita por forma de pagamento
- Receitas: formulário + histórico + marcar como pago
- Despesas: formulário + histórico + categorias
- A receber: métricas + lista filtrada + botão "✓ Pago"
- Clínicas parceiras: cadastro completo + listagem com laudos vinculados e receita total

### Pilar 3 — Relatórios
- **Laudos**: total, mês atual, por tipo (abdominal/gestacional), volume mensal (6 meses), distribuição por resultado
- **Financeiro**: receita total, despesas totais, margem líquida, fluxo de caixa, receita por forma de pagamento
- **Clínicas**: ranking por receita com barra de participação
- **Veterinários**: top solicitantes por volume de laudos

### Dados (sidebar)
- **Exportar backup**: baixa JSON com todos os dados do localStorage
- **Importar backup**: restaura dados a partir de JSON exportado

### Dashboard
- Métricas: laudos do mês, receita do mês, a receber, saldo
- Laudos recentes (últimos 5)
- Contas a receber pendentes
- Fluxo de caixa (4 meses) + receita por forma de pagamento

### Infraestrutura
- Sincronização automática com Supabase a cada save
- Fallback para localStorage quando offline
- Indicador de status na sidebar: 🟢 Sincronizado / 🟠 Sincronizando / 🔴 Offline
- Re-sync automático ao retomar foco da janela
- Exclusões propagadas para o Supabase

---

## Paleta de cores

```css
/* App */
--teal: #0f7a5a;
--teal-bg: #e4f5ef;
--teal-text: #0a5c42;
--bg: #f5f4f0;
--surface: #ffffff;
--surface2: #f0efeb;
--border: #e2e1dc;
--text: #1a1a18;
--text2: #6b6a65;
--text3: #9b9a95;
--amber: #b35c00;
--red: #c0392b;
--purple: #5b4fd4;
--blue: #1a5fa8;

/* Laudo impresso */
Verde principal: #3d7a52
Separadores: #c5ddd0 / #d8e8de / #edf2ef
```

---

## Patologias mapeadas por órgão

### Abdominal total

| Órgão | Patologias |
|-------|-----------|
| Vesícula urinária | Normal, Cistolitíase, Cistite, Espessamento de parede, N/A |
| Rins | Normal, Nefropatia crônica, Nefrolitíase, Hidronefrose, Cisto renal |
| Fígado | Normal, Hepatopatia crônica/Esteatose, Doença cística hepática, Nódulo hepático, Hepatomegalia |
| Vesícula biliar | Normal, Lama biliar, Colelitíase, Colecistite |
| Baço | Normal, Esplenomegalia, Nódulo esplênico, Hematoma esplênico |
| Estômago | Normal, Gastrite, Corpo estranho gástrico, Espessamento parietal |
| Alças intestinais | Normal, Enteropatia difusa, Suspeita de obstrução, Intussuscepção, Corpo estranho intestinal |
| Pâncreas | Normal, Pancreatite, Massa pancreática |
| Adrenais | Normal, Hiperplasia adrenal bilateral, Nódulo adrenal, Massa adrenal |
| Linfonodos | Normal, Linfadenomegalia abdominal |
| Próstata | Normal, Hiperplasia prostática, Prostatite, Cisto prostático, N/A |

### Gestacional / obstétrico

| Campo | Opções |
|-------|--------|
| Útero | Normal (gestante), Reabsorção fetal, Óbito fetal |
| Número de fetos | Campo livre |
| Batimentos (BCF) | Presentes e rítmicos, Ausentes, Irregulares |
| Placenta | Normal, Placentite, Descolamento placentário |
| Líquido amniótico | Normal, Oligodrâmnio, Polidrâmnio |
| Idade gestacional | Campo livre |

---

## Backlog — o que ainda falta

### Pilar 3 — Visão de dados
- [ ] Acompanhamento de exames gestacionais
- [ ] Exportar relatório em PDF/Excel

### Outros
- [ ] Mais patologias nos selects conforme uso real
- [ ] Busca rápida dentro do select de patologia
- [ ] Alterações de arte gráfica e identidade visual (a definir)

---

## Regras para o desenvolvimento

1. **Sempre entregar um arquivo único** — `ultravet_10.html` (ou versão subsequente). Nunca separar em múltiplos arquivos.
2. **Nunca quebrar o que já funciona.** Antes de alterar qualquer função existente, leia o código completo do trecho afetado.
3. **Manter a paleta de cores** definida acima. Não introduzir novas cores sem justificativa.
4. **localStorage + Supabase como persistência** — localStorage é cache local, Supabase é a fonte de verdade na nuvem.
5. **Zero dependências externas de CDN** — exceto o Supabase já integrado via fetch nativo.
6. **Comentários mínimos e em português** — o código já é legível; comentários só onde a lógica for não-óbvia.
7. **Ao concluir a tarefa**, informar quais funções foram adicionadas ou modificadas e atualizar o histórico de versões no topo do arquivo (bloco de comentário).
8. **Deploy Netlify** — usar `[skip netlify]` em todos os commits durante desenvolvimento para não gastar build minutes. Só omitir o skip quando o João disser "publica", "faz deploy" ou "sobe para o ar".

---

## Como usar este prompt

1. Abra uma nova sessão no **Claude Code**
2. Use este arquivo como `CLAUDE.md` na raiz do projeto (já está configurado)
3. Faça upload do arquivo `ultravet_9.html` se necessário
4. Descreva o que deseja implementar ou corrigir

O Claude Code lerá o HTML completo e continuará exatamente de onde parou.

---

*Briefing atualizado em maio/2026 — versão base: ultravet_9.html*
