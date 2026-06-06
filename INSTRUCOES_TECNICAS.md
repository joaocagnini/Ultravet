# VetRoutine — Instruções Técnicas Completas
> Documento gerado em 02/06/2026 para migração para ambiente PHP/MySQL/XAMPP
> Contém todo o aprendizado acumulado do projeto

---

## 1. VISÃO GERAL DO PRODUTO

**VetRoutine** é um sistema de gestão para Patrine Cagnini — Médica Veterinária | Ultrassonografista | CRMV-SC 09377.

### O que o sistema faz:
- **Laudos ultrassonográficos**: formulário completo abdominal e gestacional com achados por órgão, impressão diagnóstica automática, fotos do exame, impressão/PDF compartilhável
- **Financeiro**: receitas, despesas, a receber, clínicas parceiras com tabela de remuneração automática
- **Agenda**: agendamentos com tipos (laudo/consulta/ambos), cores por tipo, ações rápidas
- **Relatórios**: métricas mensais, ranking clínicas, fluxo de caixa
- **Compartilhar**: link único por laudo (`?laudo=ID`) que abre visualização pública com mensagem formatada para WhatsApp

### Como a Patrine trabalha:
- Atende em domicílio e em clínicas parceiras
- Cobra por exame, com tabela de remuneração diferente por origem (Bruna / Hospital 4 Patas / Meus Clientes)
- Usa tablet durante os exames para laudar (precisa ver imagens na lateral)
- Envia laudos por WhatsApp/e-mail para tutores e veterinários solicitantes
- Trabalha com múltiplas clínicas, cada uma com desconto diferente

---

## 2. STACK ATUAL (HTML single-file + Supabase)

### Arquivos:
```
index.html          ← App completo (~800KB, tudo inline: CSS + HTML + JS)
logo.png            ← Logo da sidebar (VetRoutine)
```

### Banco de dados (Supabase/PostgreSQL):
```
URL: https://iisxmfepvqpswdnurtuo.supabase.co
Key: sb_publishable_4yj6xy0vvxLypajgqtEMCQ_fR6nE2DY

Tabelas (todas com schema: id bigint PRIMARY KEY, payload jsonb):
- uv_laudos
- uv_receitas
- uv_despesas
- uv_clinicas
- uv_agenda

RLS desabilitado. Sem autenticação (acesso por chave pública).
```

### Persistência dupla:
- **localStorage**: cache local (chave: `uv_laudos`, `uv_receitas`, etc.)
- **Supabase**: fonte de verdade na nuvem
- Sync automático a cada save + ao focar a janela + a cada 60s

### Deploy:
- **Netlify**: https://vetroutine.netlify.app/ (auto-deploy do branch main)
- **GitHub**: https://github.com/joaocagnini/Ultravet (público)

---

## 3. ESTRUTURA DE DADOS (objetos JavaScript)

### Laudo (`uv_laudos`):
```javascript
{
  id: Date.now(),                    // timestamp como ID único
  tipo: 'Abdominal total',           // 'Abdominal total' | 'Gestacional'
  resultado: 'Normal',               // 'Normal' | 'Atenção' | 'Alterado'
  paciente: 'Nome do animal',
  especie: 'Canino',                 // Canino | Felino | Ave | Roedor | Réptil | Outro
  raca: 'Labrador',
  sexo: 'Macho',                     // Macho | Fêmea | Macho castrado | Fêmea castrada
  idade: '5 anos',
  data: '2026-06-02',                // formato ISO YYYY-MM-DD
  tutor: 'Nome do tutor',
  vet: 'Dr(a). Nome',               // veterinário solicitante
  clinica: 'Nome da clínica',
  origem: 'Bruna',                   // Bruna | Hospital 4 Patas | Meus Clientes | (vazio = avulso)
  tipoAtend: 'Padrão',              // tipo de atendimento (muda por origem)
  valor: 210,                        // valor cobrado (número)
  valorRec: 147,                     // valor recebido (número)
  forma: 'Pix',                      // forma de pagamento
  fpago: 'Pago',                     // 'Pago' | 'Pendente'
  conclusao: 'Texto da impressão diagnóstica',
  obs: 'Observações adicionais',
  fotos: [                           // fotos em base64
    { src: 'data:image/jpeg;base64,...', name: 'nome_arquivo.jpg' }
  ],
  orgAbdom: {                        // achados abdominal (cada órgão)
    bexiga: { texto: 'Repleção líquida adequada...' },
    rins:   { texto: 'Formato mantido...' },
    figado: { texto: '...' },
    // vbiliar, baco, estomago, intestino, pancreas, adrenais, linfo, utero_f, prostata, testis
  },
  orgGest: {                         // achados gestacional
    utero: { texto: '...' },
    bcf:   { texto: '...' },
    placenta: { texto: '...' },
    amnio: { texto: '...' },
    fetos: { texto: '...' },
    ig:    { texto: '...' },
  }
}
```

### Clínica (`uv_clinicas`):
```javascript
{
  id: Date.now(),
  nome: 'Nome da clínica',
  sigla: 'H4P',                     // abreviação usada no nome do arquivo PDF
  responsavel: 'Nome',
  telefone: '(XX) XXXXX-XXXX',
  email: 'email@clinica.com',
  cond: 'À vista',                   // condição de pagamento
  desc: 30,                          // desconto em % (número)
  origemClinica: 'Hospital 4 Patas', // origem para tabela de remuneração
  endereco: 'Rua, número...',
  obs: 'Observações'
}
```

### Receita (`uv_receitas`):
```javascript
{
  id: Date.now(),
  data: '2026-06-02',
  desc: 'Abdominal total — Rex [Bruna · Domiciliar]',
  clinica: 'Nome da clínica',
  valor: 147,                        // valor recebido
  valorCobrado: 210,                 // valor cobrado ao tutor
  forma: 'Pix',
  status: 'Pago',                    // 'Pago' | 'Pendente'
  vet: 'Dr(a). Nome',
  obs: ''
}
```

### Despesa (`uv_despesas`):
```javascript
{
  id: Date.now(),
  data: '2026-06-02',
  desc: 'Descrição da despesa',
  valor: 50,
  cat: 'Material',                   // categoria
  forma: 'Pix'
}
```

### Evento de Agenda (`uv_agenda`):
```javascript
{
  id: Date.now(),
  tipoAgenda: 'Laudo',              // 'Laudo' | 'Consulta' | 'Ambos' | 'Geral'
  paciente: 'Nome do animal',
  tutor: 'Nome do tutor',
  especie: 'Canino',
  data: '2026-06-02',
  hora: '14:30',
  tipo: 'Abdominal total',           // tipo de exame (para laudos)
  clinica: 'Nome da clínica',
  local: 'Endereço',
  vet: 'Dr(a). Nome',
  obs: 'Observações',
  status: 'Agendado',               // 'Agendado' | 'Confirmado' | 'Realizado' | 'Cancelado' | 'Faltou'
  pacienteId: null,                  // link com paciente do módulo consultas
  laudoId: null,                     // ID do laudo criado a partir deste agendamento
  consultaId: null                   // ID da consulta criada a partir deste agendamento
}
```

---

## 4. TABELA DE REMUNERAÇÃO

```javascript
// Valores cobrado / recebido por origem e tipo de atendimento
const tabelaRemuneracao = {
  'Bruna': {
    'Domiciliar':  { cobrado: 290, recebido: 203 },  // 30% desconto
    'Padrão':      { cobrado: 210, recebido: 147 },  // 30% desconto
    'Plantão':     { cobrado: 300, recebido: 210 },  // 30% desconto
  },
  'Hospital 4 Patas': {
    'Hospital Padrão':  { cobrado: 247, recebido: 110 },  // 55% desconto
    'Hospital Plantão': { cobrado: 370, recebido: 140 },
  },
  'Meus Clientes': {
    'Padrão':           { cobrado: 210, recebido: 210 },  // sem desconto
    'Plantão':          { cobrado: 270, recebido: 270 },
    'Plantão após 22h': { cobrado: 370, recebido: 370 },
  }
};
```

---

## 5. ÓRGÃOS E PATOLOGIAS

### Abdominal Total — órgãos e keys:
| Órgão | Key JS | ID HTML do textarea |
|-------|--------|---------------------|
| Vesícula urinária | `bexiga` | `o-bexiga` |
| Rins | `rins` | `o-rins` |
| Fígado | `figado` | `o-figado` |
| Vesícula biliar | `vbiliar` | `o-vbiliar` |
| Baço | `baco` | `o-baco` |
| Estômago | `estomago` | `o-estomago` |
| Alças intestinais | `intestino` | `o-intestino` |
| Pâncreas | `pancreas` | `o-pancreas` |
| Adrenais | `adrenais` | `o-adrenais` |
| Linfonodos | `linfo` | `o-linfo` |
| Útero (fêmea) | `utero_f` | `o-utero-f` ← **HÍFEN, não underscore!** |
| Próstata (macho) | `prostata` | `o-prostata` |
| Testículos (macho) | `testis` | `o-testis` |

**ATENÇÃO**: A função `orgElId(k)` converte underscore → hífen para buscar o elemento HTML:
```javascript
function orgElId(k) { return 'o-' + k.replace(/_/g, '-'); }
```

### Gestacional — órgãos:
`utero`, `fetos`, `bcf`, `placenta`, `amnio`, `ig`

### Patologias por órgão (options dos selects):
- **bexiga**: normal, coagulo, neoplasia, cistolitiase (exibido "Urolitíase"), cistite, espessamento, na
- **rins**: normal, nefropatia, nefrolitiase, hidronefrose, cisto
- **figado**: normal, hepatopatia_aguda, hepatopatia, cisto_hepatico, nodulo_hepatico, hepatomegalia
- **vbiliar**: normal, lama_biliar, colelitíase, mucocele, colecistite
- **baco**: normal, esplenomegalia, nodulo_baco, hematoma_baco
- **estomago**: normal, gastrite, corpo_estranho, espessamento_gastrico
- **intestino**: normal, enteropatia, obstrucao, intussuscepcao, corpo_estranho_int
- **pancreas**: normal, pancreatite, edema_pancreas, massa_pancreas
- **adrenais**: normal, hiperplasia_adrenal, nodulo_adrenal, massa_adrenal
- **linfo**: normal, linfadenomegalia
- **utero_f**: normal, piometra, mucometra, hiperplasia_endometrial, cisto_ovariano, na
- **prostata**: normal, hiperplasia_prostata, prostatite, cisto_prostata, na
- **testis**: normal, testis_ectopico, testis_orqui, na

---

## 6. REGRAS DE NEGÓCIO IMPORTANTES

### Visibilidade por sexo:
- Próstata e testículos: só aparecem para **Macho** e **Macho castrado**
- Útero: só aparece para **Fêmea** e **Fêmea castrada**

### Ao salvar laudo com valor > 0:
- Sistema lança **receita automaticamente** no financeiro

### Impressão diagnóstica automática:
- Formato: `Órgão - Doença - Texto da impressão`
- Entradas sem impressão definida: marcadas com ⚠️
- Ao editar laudo: `toggleGest(isEditing=true)` → não apaga conclusão salva

### Nome do arquivo PDF:
- Formato: `(Sigla Clínica) - Laudo Ultrassom - (Paciente) - (Tutor) (Data)`
- Função: `nomeArquivoLaudo(l)`

### Link compartilhável:
- URL: `sistema.com/?laudo=ID`
- Ao abrir com este parâmetro: esconde a interface e mostra só o laudo
- Mensagem formatada para WhatsApp inclui: paciente, tutor, data, link

---

## 7. FEATURES PENDENTES (backlog para PHP/MySQL)

### Alta prioridade:
1. **Módulo de Consultas Veterinárias** (completo mas instável na versão HTML):
   - Cadastro de pacientes (animal + tutor)
   - Registro de consultas com exame físico completo (temp, FC, FR, TPC, mucosas, hidratação)
   - Receituário (comum e especial/controlada) com impressão formatada
   - Dashboard de consultas (métricas, retornos, km rodados)
   - Deslocamento: km × R$/km calculado automaticamente

2. **Cobrança mensal por clínica** (#4):
   - Relatório dos exames do mês por clínica
   - Data, tipo, paciente, valor por exame
   - Total a pagar
   - Exportação (PDF ou Excel)

3. **Editor de modelos de descrição** (#5):
   - Salvar descrições personalizadas como templates reutilizáveis
   - Cada template = descrição do achado + impressão diagnóstica
   - Botão "Salvar como modelo" no próprio laudo
   - Aplicar template no próximo exame similar

4. **Múltiplos modelos de laudo** (#6):
   - Modelo Patrine: Abdominal, Gestacional
   - Modelo 4 Patas: Abdominal, Gestacional
   - Cada clínica pode ter seu modelo preferido
   - Seleção no momento de criar o laudo

5. **Editor visual de layouts** (#iniciado mas não publicado):
   - Split view: controles à esquerda, preview ao vivo à direita
   - 6 abas: Cores, Tipografia, Cabeçalho, Seções, Texto, Página
   - Presets nomeados por clínica
   - Campos adicionais: telefone, email no cabeçalho, texto do rodapé personalizado

### Média prioridade:
6. **Agenda como centro das demandas** (#iniciado):
   - Tipo de agendamento: 🔬 Laudo / 🩺 Consulta / 📋 Ambos / 📅 Geral
   - Iniciar laudo/consulta diretamente do evento da agenda
   - Vinculação automática laudo/consulta → evento
   - Cores por tipo no calendário

7. **Imagens na lateral durante laudagem** (#7):
   - Painel lateral (28% da tela) com fotos do exame
   - Aparece ao adicionar fotos, fecha com backdrop no mobile
   - Zoom ao clicar na imagem

8. **Paginação** (implementado):
   - 20 laudos por página com navegação

---

## 8. PROBLEMAS RECORRENTES E SOLUÇÕES

### Bug crítico: dados somindo
**Causa**: race condition entre push local→Supabase e pull Supabase→local
**Solução implementada**:
1. Após qualquer `save()`, `localSaveTs = Date.now()`
2. `initCloud()` não faz pull se `localSaveTs` for recente (10 minutos)
3. `load()` ao iniciar define `localSaveTs` para proteger por 1 minuto
4. `syncCloud()` tem retry automático (3 tentativas)
5. Pull usa **merge** (não sobrescreve): itens locais não sincronizados são preservados

```javascript
function mergeWithLocal(cloudItems, localItems) {
  var cloudIds = {};
  cloudItems.forEach(function(i){ cloudIds[i.id] = true; });
  var soLocal = localItems.filter(function(i){ return !cloudIds[i.id]; });
  return cloudItems.concat(soLocal);
}
```

### Bug: editClId bloqueando novos cadastros
**Causa**: ao clicar ✏️ Editar numa clínica e navegar sem cancelar, `editClId` ficava setado
**Solução**: 
- Reset de `editClId` ao entrar na aba clínicas
- `addClinica()` verifica o texto do botão: se diz "Cadastrar", força `editClId = null`

### Bug: goPage truncado com código do sidebar dentro
**Causa**: `sidebar_collapse.py` injetou `toggleSidebar`/`initSidebar` DENTRO do goPage (como body do `if (name === 'dashboard')`)
**Consequência**: 6 cópias duplicadas dessas funções, código de inicialização (`load`, `initCloud`) correndo em contexto errado
**Mitigação**: nunca usar o marcador `// ─── SIDEBAR TOGGLE` como ponto de injeção

### Regra de ouro para injeção de JS:
- Âncora segura: `function saveAgenda()` (garantidamente após goPage fechar)
- Nunca injetar antes de validar que goPage fecha e o init block está em profundidade 0
- Validar com: `onclick_fns - defined_fns == empty set` (sem funções onclick sem definição)

---

## 9. CONFIGURAÇÃO DO SISTEMA (localStorage `uv_config`)

```javascript
{
  nome: 'Patrine Cagnini',
  esp: 'Médica Veterinária | Ultrassonografista',
  crmv: 'CRMV-SC 09377',
  laudo: {
    titulo: 'Relatório Ultrassonográfico',
    cor: '#3d7a52',                    // cor principal do laudo impresso
    fonte: 'Arial, sans-serif',
    fontsize: '12',
    lineh: '1.6',
    margens: '12mm',
    papel: 'A4',
    headerStyle: 'gray',              // 'gray' | 'white' | 'minimal'
    logoAltura: '70',                 // altura da logo em px
    secAchados: true,
    secConclusao: true,
    secAssinatura: true,
    secFotos: true,
    secFooter: true,
    modeloConclusao: '',
    modeloObs: '',
    labelAchados: 'Achados Ultrassonográficos',
    labelConclusao: 'Impressão Diagnóstica'
  }
}
```

### Duas logos — NUNCA misturar:
| | Variável | Onde aparece |
|---|---|---|
| Logo do sistema | `LOGO_B64 = 'logo.png'` | Sidebar do app |
| Logo do laudo | `getLogo()` → `uv_logo` | Cabeçalho/rodapé impresso |

---

## 10. PALETA DE CORES

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

/* Sidebar */
background: #1a3a5c;

/* Laudo impresso */
cor principal: #3d7a52
separadores: #e0e0dc / #edf2ef
```

---

## 11. PARA O TIME DE DESENVOLVIMENTO PHP/MySQL

### Migração do banco:
1. Exportar tabelas do Supabase como JSON
2. Converter: cada registro `{id, payload}` → campos da tabela relacional
3. O `payload` é um JSON com todos os campos do objeto (ver seção 3)

### Schema sugerido para MySQL:

```sql
-- Laudos
CREATE TABLE laudos (
  id BIGINT PRIMARY KEY,
  tipo VARCHAR(50),
  resultado VARCHAR(20),
  paciente VARCHAR(200),
  especie VARCHAR(50),
  raca VARCHAR(100),
  sexo VARCHAR(30),
  idade VARCHAR(50),
  data DATE,
  tutor VARCHAR(200),
  vet VARCHAR(200),
  clinica VARCHAR(200),
  origem VARCHAR(100),
  tipo_atend VARCHAR(100),
  valor DECIMAL(10,2),
  valor_rec DECIMAL(10,2),
  forma_pagamento VARCHAR(50),
  status_pagamento VARCHAR(20),
  conclusao TEXT,
  observacoes TEXT,
  org_abdom JSON,
  org_gest JSON,
  fotos LONGTEXT,         -- JSON array de base64
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Clinicas
CREATE TABLE clinicas (
  id BIGINT PRIMARY KEY,
  nome VARCHAR(200),
  sigla VARCHAR(20),
  responsavel VARCHAR(200),
  telefone VARCHAR(30),
  email VARCHAR(200),
  cond_pagamento VARCHAR(50),
  desconto DECIMAL(5,2),
  origem_clinica VARCHAR(100),
  endereco TEXT,
  observacoes TEXT
);

-- Receitas
CREATE TABLE receitas (
  id BIGINT PRIMARY KEY,
  data DATE,
  descricao TEXT,
  clinica VARCHAR(200),
  valor DECIMAL(10,2),
  valor_cobrado DECIMAL(10,2),
  forma_pagamento VARCHAR(50),
  status VARCHAR(20),
  vet VARCHAR(200)
);

-- Despesas
CREATE TABLE despesas (
  id BIGINT PRIMARY KEY,
  data DATE,
  descricao TEXT,
  valor DECIMAL(10,2),
  categoria VARCHAR(100),
  forma_pagamento VARCHAR(50)
);

-- Agenda
CREATE TABLE agenda (
  id BIGINT PRIMARY KEY,
  tipo_agenda VARCHAR(20),
  paciente VARCHAR(200),
  tutor VARCHAR(200),
  especie VARCHAR(50),
  data DATE,
  hora TIME,
  tipo_exame VARCHAR(100),
  clinica VARCHAR(200),
  local TEXT,
  vet VARCHAR(200),
  observacoes TEXT,
  status VARCHAR(30),
  laudo_id BIGINT,
  consulta_id BIGINT
);
```

### Features críticas para manter na versão PHP:
1. **Sync offline-first**: dados devem funcionar sem internet, sincronizar quando conectar
2. **Fotos em base64**: imagens embutidas no laudo (não arquivos externos) para garantir portabilidade
3. **Compartilhar laudo**: URL pública por laudo, sem login para visualizar
4. **Tabela de remuneração**: automática por origem + tipo de atendimento
5. **Impressão do laudo**: layout profissional com logo, assinatura, página única (sem cortes)

### Fluxo principal que NÃO pode quebrar:
1. Criar novo laudo → selecionar patologia → texto auto-preenchido → ajustar → salvar
2. Ao salvar com valor: lança receita automaticamente
3. Visualizar laudo → Compartilhar → copia link + mensagem WhatsApp
4. Abrir link compartilhado → laudo carrega sem login

---

## 12. ORIENTAÇÕES DA PATRINE (requisitos funcionais)

- "Quero que a quebra nunca aconteça no meio de palavras ou imagens no PDF"
- "Quando compartilho, quero já ter a mensagem pronta para copiar"
- "As imagens do exame precisam estar na lateral enquanto laudo"
- "Preciso emitir cobrança mensal por clínica (relação de exames do mês)"
- "Quero poder salvar descrições como modelo para reutilizar"
- "Preciso de múltiplos modelos de laudo (Patrine / Hospital 4 Patas)"
- "Sistema não pode perder dados — confirmação antes de excluir qualquer coisa"
- "Clinicas, laudos e registros financeiros são críticos"
- "Funciona em tablet durante o atendimento"
- "No mobile a sidebar deve ficar compacta (só ícones)"

---

*Documento gerado por Claude Code — VetRoutine v1.10+*
*Projeto: https://github.com/joaocagnini/Ultravet*
*Netlify: https://vetroutine.netlify.app/*
