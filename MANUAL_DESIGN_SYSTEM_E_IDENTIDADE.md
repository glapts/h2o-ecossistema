# MANUAL DO DESIGN SYSTEM & IDENTIDADE VISUAL
## Ecossistema H2O: Depósito de Água, Bar, Sinuca & Lounge

---

**Destinatário:** Sr. Luiz Fernando  
**Autor:** Glaucio Pires Pontes  
**Paleta Oficial:** Azul Windows, Verde Grama e Branco  
**Versão:** 1.0  

---

## 1. CONCEITO VISUAL E PRINCÍPIOS DA MARCA

A identidade visual do **Ecossistema H2O** une a **pureza e confiança da água mineral** com a **energia, conveniência e descontração de um bar e lounge de entretenimento**. 

Os três princípios norteadores de design são:
1. **Claridade Imediata:** Telas sem ruído visual, botões grandes e legíveis mesmo à distância no balcão;
2. **Confiança e Saúde:** Tons de azul límpido que remetem à água mineral de alta qualidade (Ibirá, Aldeinha, Levíssima, Crystal);
3. **Agilidade e Lucro:** Toques de verde vibrante destacando ações de compra, cashback, comanda fechada e lucro líquido.

---

## 2. PALETA CROMÁTICA OFICIAL

| Variável | Código Hex | Amostra | Aplicação Principal |
| :--- | :---: | :---: | :--- |
| **Azul Windows Principal** | `#0078D4` | 🟦 | Botões primários, cabeçalhos, destaques da marca H2O |
| **Azul Profundo (Dark)** | `#005A9E` | 🟦 | Barras de navegação, rodapé institucional, cards executivos |
| **Azul Suave (Light)** | `#E5F3FF` | 🧊 | Fundo de cards informativos, badges de produtos de água |
| **Verde Grama (Vibrante)** | `#2ECC71` | 🟩 | Botões de ação rápida, fechamento de comanda, status positivo |
| **Verde Floresta (Dark)** | `#219653` | 🟩 | Badges de cashback, títulos de lucro e tags de maior margem |
| **Verde Suave (Light)** | `#EAFAF1` | 🍃 | Fundo de destaque para bônus de sinuca e Clube Ibirá |
| **Branco Puro** | `#FFFFFF` | ⬜ | Fundo de cards, contraste em botões e superfícies elevadas |
| **Cinza de Apoio (Background)** | `#F4F7FA` | 🌫️ | Fundo geral da aplicação e telas de balcão |
| **Texto Escuro Principal** | `#1E293B` | ⬛ | Tipografia de títulos e valores monetários |
| **Texto Secundário / Muted** | `#475569` | 🔘 | Descrições, subtítulos e instruções operacionais |

---

## 3. TIPOGRAFIA

* **Fonte Primária:** `Inter` (Google Fonts)
  * `Font-weight 400`: Texto de apoio e descrições;
  * `Font-weight 600 / 700`: Nomes de produtos, títulos de seções e badges;
  * `Font-weight 800 / 900`: Valores monetários (R$), números de destaque e cabeçalhos de alta ênfase.
* **Fonte Secundária / Display:** `Plus Jakarta Sans` para banners magnéticos e títulos do hero.

---

## 4. COMPONENTES VISUAIS PADRONIZADOS

### 4.1. Badge Numérico de Seção
Círculo de 36px com numeração em negrito branco sobre fundo Azul Windows ou Verde Grama, com sombra suave (`box-shadow: 0 3px 8px rgba(0, 120, 212, 0.3)`).

### 4.2. Card de Produto Touch
Cards brancos com borda superior colorida de 4px:
* **Borda Azul:** Produtos de Água e controle de estoque;
* **Borda Verde:** Produtos de alto giro de bar, sinuca e bônus;
* **Borda Dourada / Escura:** Produtos Premium (Água Ibirá medicinal e VIP Lounge das 3 máquinas).

### 4.3. Alertas e Modais Sonorizados
Modais com cantos arredondados (24px a 30px), backdrop escuro fosco (`backdrop-blur-sm`), ícone circular superior e botões de ação full-width com altura mínima de 48px para facilitar o toque no balcão.
