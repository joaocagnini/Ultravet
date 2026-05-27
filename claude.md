# UltraVet — Prompt de Inicialização para Claude Code

> Cole este prompt na primeira mensagem de uma sessão do Claude Code para retomar o projeto exatamente do ponto atual (v1.8).

---

## Contexto do projeto

Você vai continuar o desenvolvimento do **UltraVet**, um aplicativo HTML single-file de gestão para **Patrine Cagnini — Médica Veterinária | Ultrassonografista | CRMV-SC 09377**.

O app roda 100% offline no navegador e persiste todos os dados via `localStorage`. Não há backend, build system nem dependências externas. O entregável é sempre **um único arquivo HTML** (`ultravet_9.html`, incrementando a versão).

---

## Arquivo de referência

O arquivo atual é `ultravet_8.html` (~1420 linhas), estruturado assim:

| Seção | Linhas aprox. |
|-------|--------------|
| CSS — variáveis, layout, componentes | 8–192 |
| HTML — sidebar, dashboard, páginas | 193–580 |
| JavaScript — estado, lógica, persistência | 685+ |

**Variáveis de estado globais:**
```js
let laudos = [], receitas = [], despesas = [], clinicas = [];
let editId = null, viewId = null, curFSec = "f-overview";
let fotosTmp = []; // fotos do laudo em edição (base64) — adicionado v1.8
```

**Chaves do localStorage:**
- `uv_laudos` — laudos (campo `fotos[]` em base64 desde v1.8)
- `uv_receitas`
- `uv_despesas`
- `uv_clinicas`

---

## O que já está implementado (v1.8)

### Pilar 1 — Laudos
- Formulário completo: abdominal total e gestacional
- Achados em 3 colunas: Órgão | Select de patologia | Texto livre (preenchido automaticamente ao selecionar a patologia)
- Salvar, editar, excluir, pré-visualizar
- Ao salvar laudo com valor > 0 → lança receita automaticamente no financeiro
- Busca por paciente, tutor ou veterinário; filtros por tipo e resultado
- Impressão formatada (abre janela para impressão/PDF)
- Layout do laudo impresso: cabeçalho com grid de dados + logo, paleta verde `#3d7a52`, assinatura com borda verde
- **Card "Fotos do exame"** (NOVO v1.8): upload múltiplo, grid de preview, salvo em base64 no laudo, impresso ao final do laudo em grade 3 colunas

### Pilar 2 — Financeiro
- Visão geral: métricas do mês + gráfico fluxo de caixa (6 meses) + receita por forma de pagamento
- Receitas: formulário + histórico + marcar como pago
- Despesas: formulário + histórico + categorias
- A receber: métricas + lista filtrada + botão "✓ Pago"
- Clínicas parceiras: cadastro completo + listagem com laudos vinculados e receita total

### Dashboard
- Métricas: laudos do mês, receita do mês, a receber, saldo
- Laudos recentes (últimos 5)
- Contas a receber pendentes
- Fluxo de caixa (4 meses) + receita por forma de pagamento

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

### Pilar 3 — Visão de dados (não implementado)
- [ ] Volume de laudos por período (dia / mês / tipo)
- [ ] Faturamento e margem (receita × despesa)
- [ ] Ranking de clínicas parceiras por volume e receita
- [ ] Veterinários solicitantes mais frequentes
- [ ] Acompanhamento de exames gestacionais
- [ ] Exportar relatório em PDF/Excel

### Outros
- [ ] Mais patologias nos selects conforme uso real
- [ ] Busca rápida dentro do select de patologia
- [ ] Exportar dados — backup JSON do localStorage
- [ ] Importar dados — restaurar backup
- [ ] Integração com Supabase para acesso multi-dispositivo

---

## Regras para o desenvolvimento

1. **Sempre entregar um arquivo único** — `ultravet_9.html` (ou versão subsequente). Nunca separar em múltiplos arquivos.
2. **Nunca quebrar o que já funciona.** Antes de alterar qualquer função existente, leia o código completo do trecho afetado.
3. **Manter a paleta de cores** definida acima. Não introduzir novas cores sem justificativa.
4. **localStorage como única persistência** — não usar IndexedDB, sessionStorage ou cookies, a menos que seja explicitamente para a feature de nuvem (Supabase).
5. **Zero dependências externas** — nenhum `<script src="...">` de CDN, nenhum import de biblioteca externa.
6. **Comentários mínimos e em português** — o código já é legível; comentários só onde a lógica for não-óbvia.
7. **Ao concluir a tarefa**, informar quais funções foram adicionadas ou modificadas e atualizar o histórico de versões no topo do arquivo (bloco de comentário).

---

## Como usar este prompt

1. Abra uma nova sessão no **Claude Code**
2. Cole este documento inteiro como primeira mensagem (ou use como `CLAUDE.md` na raiz do projeto)
3. Faça upload do arquivo `ultravet_8.html`
4. Descreva o que deseja implementar ou corrigir

O Claude Code lerá o HTML completo e continuará exatamente de onde parou.

---

*Briefing gerado em maio/2026 — versão base: ultravet_8.html*
