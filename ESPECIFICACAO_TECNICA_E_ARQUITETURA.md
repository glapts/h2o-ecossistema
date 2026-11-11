# ESPECIFICAÇÃO TÉCNICA E ARQUITETURA DE SOFTWARE
## Ecossistema de Automação & Vendas H2O: Depósito de Água, Bar, Sinuca & Lounge

---

**Empresa Contratante:** Depósito de Água, Bar, Sinuca & Lounge H2O  
**Proprietário:** Sr. Luiz Fernando  
**Consultor / Arquiteto de Software:** Glaucio Pires Pontes  
**Data:** Novembro de 2026  
**Versão do Documento:** 1.0 – Especificação de Engenharia  

---

## 1. VISÃO GERAL DA ARQUITETURA

O ecossistema **H2O** foi projetado para rodar com baixíssima latência, alta disponibilidade e operação sem fricção em múltiplos dispositivos (computador do balcão, tablets e smartphones). A arquitetura combina:

1. **Frontend PWA (Progressive Web App):** Aplicação responsiva de balcão com suporte a toque, atalhos de teclado e modo offline transitório para momentos de instabilidade de internet;
2. **Backend Serverless / Edge:** Endpoints rápidos para processamento de comandas, webhook de pagamentos via PIX e sincronização em tempo real;
3. **Mecanismo de Automação do WhatsApp (Dual Engine):**
   * *Engine Local (Sessão Web):* Para autoatendimento direto e disparos programados de baixo custo;
   * *Engine Cloud (Meta Cloud API):* Para envio garantido de alertas de alta prioridade ("Radar de Galão Vazio" nas sextas e sábados);
4. **Painel de Logística & Estoque de Cascos:** Motor de cálculo diferencial para contagem diária de vasilhames cheios versus vazios e conferência do caminhão;
5. **Gateway de Notificações VIP Lounge:** WebSockets ou Server-Sent Events (SSE) disparando alertas sonoros e visuais na tela do operador assim que um pedido é gerado nas Máquinas 01, 02 ou 03.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        ARQUITETURA ECOSSISTEMA H2O                     │
│                                                                        │
│   ┌─────────────────────┐  ┌────────────────────┐  ┌───────────────┐   │
│   │   PWA Balcão & POS  │  │  WhatsApp Bot Core │  │  VIP Lounge   │   │
│   │   (Luiz Fernando)   │  │  (Radar 130 Leads) │  │  (3 Máquinas) │   │
│   └──────────┬──────────┘  └─────────┬──────────┘  └───────┬───────┘   │
│              │                       │                     │           │
│              ▼                       ▼                     ▼           │
│   ┌────────────────────────────────────────────────────────────────┐   │
│   │               API Serverless / Edge Engine & Events            │   │
│   └──────────────────────────────┬─────────────────────────────────┘   │
│                                  │                                     │
│         ┌────────────────────────┴────────────────────────┐            │
│         ▼                                                 ▼            │
│   ┌───────────────────────────┐             ┌──────────────────────┐   │
│   │ Banco de Dados PostgreSQL │             │ Gateway PIX Instant  │   │
│   │ (Perfis, Comandas, Cascos)│             │ (Baixa Automática)   │   │
│   └───────────────────────────┘             └──────────────────────┘   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. MODELAGEM DE DADOS (ENTIDADES PRINCIPAIS)

### 2.1. Entidade `Cliente` (Perfil 360º)
```json
{
  "id": "cli_uuid_001",
  "nome": "Seu Carlos Silva",
  "apelido": "Carlos Barata",
  "telefone_whatsapp": "11999998888",
  "endereco": "Rua das Flores, 142 - Apto 3",
  "agua_habitual": "IBIRA_10L",
  "possui_casco_proprio": true,
  "quantidade_cascos_proprios": 2,
  "saldo_cashback_sinuca": 6.00,
  "consumo_acumulado_mes": 180.00,
  "ultimo_pedido_agua": "2026-11-04T14:30:00Z",
  "media_dias_reposicao": 6,
  "status_radar_domingo": "ALERTA_PROGRAMADO_SEXTA"
}
```

### 2.2. Entidade `Comanda`
```json
{
  "id": "cmd_20261110_042",
  "cliente_id": "cli_uuid_001",
  "origem": "BALCAO", // BALCAO | MESA_SINUCA | MAQUINA_01 | MAQUINA_02 | MAQUINA_03 | WHATSAPP
  "itens": [
    { "produto_id": "CERV_HEINEKEN", "nome": "Cerveja Heineken LN", "qtd": 2, "preco_unit": 10.00 },
    { "produto_id": "SINUCA_FICHA", "nome": "Ficha Sinuca", "qtd": 3, "preco_unit": 4.00 }
  ],
  "subtotal": 32.00,
  "desconto_cashback_aplicado": 0.00,
  "total_final": 32.00,
  "status": "ABERTA", // ABERTA | PAGA | CANCELADA
  "forma_pagamento": null,
  "horario_abertura": "2026-11-10T20:15:00Z"
}
```

### 2.3. Entidade `EstoqueCascosCaminhao`
```json
{
  "data_conferencia": "2026-11-10",
  "galao_ibira_10l": { "cheios": 18, "vazios_aguardando": 12 },
  "galao_aldeinha_10l": { "cheios": 25, "vazios_aguardando": 19 },
  "galao_levissima_10l": { "cheios": 22, "vazios_aguardando": 16 },
  "galao_crystal_10l_refil": { "cheios": 40, "vazios_aguardando": 31 },
  "cascos_vendidos_definitivos_crystal": 4, // Exige compra de novos cascos da distribuidora
  "alerta_pre_caminhao_disparado": true
}
```

---

## 3. ESPECIFICAÇÃO DOS MÓDULOS

### 3.1. Módulo Balcão & POS Touch-First
* Design concebido para operação rápida com 1 mão em telas touch ou com leitor de código de barras/mouse;
* Busca por sugestão inteligente conforme digitação (busca rápida por apelido: *"Marcos", "Carlos", "Pedreiro"*);
* Atalhos para inclusão dos itens mais vendidos: Águas, Fichas de Sinuca e Cervejas;
* Fechamento com 1 toque: Botões pré-definidos para Dinheiro (com sugestão de troco), PIX com QR Code dinâmico na tela e Cartão de Débito/Crédito.

### 3.2. Módulo Robô de Autoatendimento WhatsApp 24/7
* Mensagem de boas-vindas padronizada com menu numérico ou botões interativos (listas interativas do WhatsApp);
* Identificação automática se o número já consta na base dos 130 clientes habituais;
* Transcrição imediata do pedido para a fila de expedição do balcão;
* Disparo de confirmação com link do status de entrega.

### 3.3. Módulo "Radar de Galão Vazio" (Antecipador de Domingos)
* Cronjob configurado para rodar às:
  * Sexta-feira às 14:00 (Foco em planejar o fim de semana);
  * Sábado às 09:30 (Última chamada para clientes que não pediram na sexta);
* Algoritmo preditivo: Se `hoje - data_ultimo_pedido >= media_dias_reposicao - 1`, dispara o alerta amigável com opção de combos com bebidas.

### 3.4. Módulo VIP Lounge Express (Máquinas 01, 02 e 03)
* Interface web minimalista acessada via QR Code no adesivo de cada máquina:
  * URLs: `h2o.com.br/express?m=1`, `?m=2`, `?m=3`;
  * Sem necessidade de download ou login burocrático (o cliente escolhe o apelido ou digita seu telefone 1 vez);
  * Menu simplificado de 6 itens: Cerveja, Energético, Dose, Refrigerante, Água e Porção;
  * Botão de pânico/garçom: "Solicitar Atendimento na Cadeira";
  * Alerta acústico e visual no balcão principal com botão de confirmação de entrega.

---

## 4. DESIGN E USABILIDADE
* **Cores:** Azul Windows institucional (`#0078D4`, `#005A9E`), Verde Grama vivo (`#2ECC71`, `#219653`), Branco puro e Cinza neutro (`#F4F7FA`);
* **Tipografia:** Fonte `Inter` com pesos 600, 700 e 900 para máxima legibilidade contra a luz do balcão;
* **Ícones:** Lucide Icons com traço consistente;
* **Feedback tátil e visual:** Modais de confirmação, avisos sonoros discretos (bip agradável) e animações suaves de transição.
