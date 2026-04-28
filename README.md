# JARVIS ERP — Gestão Automotiva
**Powered by thIAguinho Soluções Digitais**

Sistema 100% hospedado no **GitHub Pages** (front-end estático, sem servidor próprio).
Backend: **Firestore (Firebase)** · Mídia: **Cloudinary** · IA: **Gemini API**

---

## Estrutura de arquivos

```
/
├── index.html              # Login (Admin, Gestor, Atendente, Equipe por PIN)
├── jarvis.html             # Painel principal (Admin / Gestor / Atendente)
├── equipe.html             # Painel do mecânico (kanban simplificado)
├── cliente.html            # Portal do cliente B2C (PIN + OBD2)
├── clienteOficial.html     # Portal cliente governamental
├── selecionar-perfil.html  # Tela de escolha de perfil (entrada do PWA)
├── superadmin.html         # Gestão SaaS master (todos os tenants)
├── manifest.json           # PWA manifest
├── service-worker.js       # Cache offline (Network First)
├── elm327-service.js       # Scanner OBD2 via Bluetooth BLE
├── elm-bridge.js           # Bridge Capacitor + ELM327 Bluetooth Clássico
├── css/
│   └── design.css          # Design system dark mode
├── js/
│   ├── config.js           # Firebase config + white-label + Cloudinary
│   ├── core.js             # Namespace J{} + RBAC + listeners Firestore
│   ├── os.js               # Ordens de Serviço: kanban, CRUD, PDF, WhatsApp, Cília
│   ├── financeiro.js       # DRE, fluxo de caixa, comissões, parcelamento
│   ├── fiscal.js           # NF-e XML, estoque, PMSP
│   ├── ia.js               # Gemini RAG (contexto 24 meses de OS)
│   ├── tabela-tempa.js     # Tabela TEMPA — tempos padrão de mão de obra
│   └── exportar-pmsp.js    # Exportação planilha PMSP (cliente governamental)
├── data/
│   ├── tabela-tempa.min.json        # Tabela TEMPA comprimida (produção)
│   └── tabela-tempa-completa.json   # Tabela TEMPA completa (referência)
└── capacitor-android/      # Configuração para build do APK Android
```

---

## Configuração inicial

### 1. Firebase

Edite `js/config.js` com suas credenciais:

```javascript
window.JARVIS_FB_CONFIG = {
  apiKey:            "SUA_API_KEY",
  authDomain:        "seu-projeto.firebaseapp.com",
  projectId:         "seu-projeto",
  storageBucket:     "seu-projeto.firebasestorage.app",
  messagingSenderId: "SEU_ID",
  appId:             "SEU_APP_ID"
};
```

### 2. Criar a oficina no Firestore

Coleção `oficinas`, documento `{id_oficina}`:

```json
{
  "nomeFantasia": "S O S Mecânica e Elétrica",
  "usuario": "admin",
  "senha": "suasenha",
  "status": "Ativo",
  "brandColor": "#00D4FF",
  "apiKeys": {
    "gemini": "SUA_CHAVE_GEMINI",
    "cloudName": "seu-cloudinary",
    "cloudPreset": "seu-preset-upload"
  }
}
```

Subcoleções de cada oficina (isoladas por `tenantId`):

```
oficinas/{id}/
├── ordens_servico/    clientes/    veiculos/    funcionarios/
├── estoqueItems/      financeiro/  fornecedores/  mensagens/
├── chat_equipe/       agendamentos/  conhecimento_ia/  lixeira_auditoria/
```

### 3. Cloudinary

1. Crie conta em cloudinary.com
2. Crie um Upload Preset sem assinatura (unsigned)
3. Salve `cloudName` e `cloudPreset` no documento da oficina no Firestore

### 4. Gemini IA

1. Acesse aistudio.google.com e gere uma chave de API
2. Salve em `oficinas/{id}/apiKeys/gemini`

### 5. Ícones do PWA

Crie a pasta `assets/` e coloque:
- `assets/icon-192.png` — 192x192 px, logo da oficina
- `assets/icon-512.png` — 512x512 px, logo da oficina

Sem esses arquivos o PWA funciona normalmente, mas sem ícone personalizado na tela do celular.

---

## Deploy no GitHub Pages

```bash
git add .
git commit -m "Deploy JARVIS ERP"
git push origin main
```

No repositório: Settings → Pages → Source: Deploy from branch → main → / (root)

---

## Perfis de acesso

| Perfil | Acesso |
|---|---|
| superadmin | Painel SaaS master — gerencia todos os tenants e planos |
| admin | Acesso total à oficina |
| gestor | Acesso operacional completo |
| atendente | OS, agenda e cadastro de clientes |
| mecanico | Kanban próprio, upload de fotos, logs |
| cliente | Portal PIN — acompanha OS e aprova orçamento |

---

## Módulos

**Ordens de Serviço**
Kanban 7 etapas: Triagem → Orçamento → Orçamento Enviado → Aprovado → Andamento → Pronto → Entregue.
Ao mover para Orçamento Enviado: WhatsApp automático com link + PIN. Ao marcar Pronto/Entregue: aviso ao cliente.
Importação de orçamento do sistema Cília (PDF ou XML) diretamente na aba de peças da OS.
Busca de histórico por placa + serviço/peça diretamente dentro da OS (pré-preenchida com a placa do veículo).

**Cliente governamental (PM, Prefeitura)**
Desconto por contrato configurado no cadastro (% MO e % Peças). Aplicado automaticamente na OS com exibição de valor bruto e líquido. Portal clienteOficial.html mostra peças com desconto. Exportação PMSP com colunas Valor Tabela / Desconto (%) / Valor com Desconto.

**Portal do cliente B2C**
Acesso por login + PIN. Acompanha etapas da OS, vê peças e serviços aprovados, fotos/vídeos, timeline. Scanner OBD2 via Bluetooth ELM327 orienta sobre saúde do veículo.

**Financeiro**
DRE e fluxo de caixa em tempo real. Lançamento automático ao fechar OS. Comissão separada por mecânico (% MO + % peças).

**IA Gemini RAG**
Disponível para admin/gestor/mecânico. Contexto: OS, estoque, clientes, histórico 24 meses. Diagnóstico de códigos OBD2.
Não disponível nos portais cliente.html e clienteOficial.html.

**Tabela TEMPA**
Base SINDIREPA-SP. Autocomplete por serviço. Calcula valor (tempo × R$/hora configurado).

---

## Regras Firestore (desenvolvimento)

```
allow read, write: if true;
```

Em produção: implemente Firebase Auth e restrição por tenantId.

---

## Build APK Android

O repositório inclui `capacitor-android/` com GitHub Actions.
1. Suba todos os arquivos incluindo a pasta `.github/` (habilite arquivos ocultos no Windows)
2. Settings → Actions → General → Allow all actions + Read and write permissions
3. Aba Actions → execute o workflow Build APK
4. Baixe o APK nos Artifacts

---

## Troubleshooting

**Dados não aparecem** → Verifique `tenantId` em `js/config.js`
**IA não responde** → Confirme a chave Gemini em `oficinas/{id}/apiKeys/gemini`
**Upload de fotos falha** → Preset deve ser `unsigned` no Cloudinary
**PWA não instala** → Coloque `assets/icon-192.png` e `assets/icon-512.png`

---

Versão 1.5.0 · Abril 2026 · Status: Produção
