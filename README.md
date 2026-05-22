# soja-weed-detection
IA de visão computacional para detecção de soja vs ervas daninhas em lavouras — simulação de controle de bicos pulverizadores
# 🌱 Soja vs Weed Detection — Precision Agriculture AI

> Sistema de visão computacional para detecção de soja e ervas daninhas em lavouras,
> com simulação de controle inteligente de bicos pulverizadores.

![Python](https://img.shields.io/badge/Python-3.13-blue)
![YOLOv8](https://img.shields.io/badge/YOLOv8-Ultralytics-purple)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 Sobre o Projeto

Este projeto foi desenvolvido como parte do meu portfólio de transição de carreira
da manutenção industrial para dados e inteligência artificial.

O objetivo é construir uma IA capaz de distinguir plantas de cultura (crop) de ervas
daninhas (weed) em imagens aéreas de lavoura, simulando o acionamento seletivo de
bicos pulverizadores — abrindo apenas onde há mato e mantendo fechado onde há cultura.

**Impacto real:**
- Redução de 30% a 60% no uso de defensivos agrícolas por passagem
- Menor custo de produção para o agricultor
- Menor impacto ambiental — menos produto no solo e na água

---

## 🎯 Motivação

Sistemas como este já existem no mercado, como o John Deere See & Spray.
Este projeto foi construído do zero com o objetivo de entender profundamente
cada etapa do pipeline — desde a coleta e análise dos dados até a inferência
e a lógica de controle — usando exclusivamente ferramentas gratuitas e abertas.

---

## 🏗️ Arquitetura do Sistema    Câmera (drone/barra pulverizadora)
↓
Captura de frame
↓
Modelo YOLOv8n
(detecção em tempo real)
↓
Mapa de detecção
[crop / weed / background]
↓
Lógica de controle
(divide imagem em N seções)
↓
Acionamento dos bicos
[ABERTO se weed | FECHADO se crop]---

## 📊 Dataset

| Propriedade | Valor |
|---|---|
| Nome | Weed Crop Aerial |
| Fonte | Roboflow 100 / Intel |
| Licença | CC BY 4.0 |
| Total de imagens | 1.176 |
| Treino | 823 imagens (70%) |
| Validação | 235 imagens (20%) |
| Teste | 118 imagens (10%) |
| Classes | crop (0) / weed (1) |
| Resolução | 640 x 640 px |

### Análise Exploratória dos Dados (EDA)

Durante a exploração do dataset foram identificados pontos críticos:

**Desbalanceamento severo de classes:**
- `crop`: 278 anotações (5,1%)
- `weed`: 5.188 anotações (94,9%)

Isso significa que para cada planta de cultura anotada existem 18 ervas daninhas.
Este desequilíbrio foi tratado no treinamento com o parâmetro `cls=2.0`,
que aumenta o peso do erro na classe minoritária.

**Objetos pequenos:**
A maioria das plantas ocupa menos de 10% da largura e altura da imagem,
o que representa um desafio clássico em visão computacional.
O YOLOv8 com resolução 640x640 foi escolhido justamente para preservar
detalhes finos nessa escala.

**Anotações incompletas:**
Algumas imagens apresentam plantas visíveis sem anotação correspondente,
o que contribui para o recall de 69.2% do modelo final.
Identificado durante a EDA como uma limitação do dataset público utilizado.

---

## 🤖 Modelo

| Parâmetro | Valor |
|---|---|
| Arquitetura | YOLOv8n (nano) |
| Base | Transfer learning COCO |
| Epochs | 24 (early stopping na epoch 14) |
| Batch size | 8 |
| Imagem | 640 x 640 |
| Otimizador | AdamW |
| GPU | Tesla T4 (Google Colab) |
| Tempo de treino | ~8 minutos |

### Resultados

| Classe | Precisão | Recall | mAP50 |
|---|---|---|---|
| **Geral** | 56,5% | 69,2% | 70,7% |
| crop | 35,0% | 76,6% | 65,4% |
| weed | 77,9% | 61,9% | 76,1% |

---

## ⚠️ Limitações Identificadas

### 1. Dataset de origem europeia
O dataset utilizado foi coletado na Letônia por pesquisadores europeus.
As plantas rotuladas como `crop` são culturas europeias (canola, beterraba, trigo),
não soja brasileira. Para uso real em lavouras de soja do Brasil, o modelo
precisaria de fine-tuning com imagens coletadas em campo nacional,
anotadas por agrônomos com conhecimento da cultura local.

### 2. Desbalanceamento de classes
A proporção de 95% weed vs 5% crop afeta diretamente a precisão na classe crop (35%).
O modelo aprende muito melhor a detectar mato do que a cultura, pois tem muito mais
exemplos de mato durante o treino.

### 3. Anotações incompletas
O dataset público apresenta imagens com plantas visíveis sem anotação.
O modelo aprende que essas plantas "não existem", o que reduz o recall geral.

### 4. Variação de condições de campo
O modelo foi treinado com imagens em condições específicas de luz e solo.
Em condições diferentes — fim de tarde, solo úmido, plantas em fases distintas —
o desempenho pode variar. Data augmentation adicional poderia mitigar isso.

---

## 🚀 Próximos Passos

- [ ] Coletar imagens reais de soja brasileira em campo
- [ ] Anotar com agrônomos especializados em soja
- [ ] Fine-tuning do modelo com dados nacionais
- [ ] Testar modelo YOLOv8s (small) para ganho de precisão
- [ ] Implementar a lógica de controle em Raspberry Pi real
- [ ] Integrar com sinal PWM para solenóides de bicos reais
- [ ] Medir economia real de defensivo em campo experimental

---

## 🛠️ Stack Tecnológica

| Ferramenta | Uso |
|---|---|
| Python 3.13 | Linguagem principal |
| YOLOv8 (Ultralytics) | Modelo de detecção |
| OpenCV | Processamento de imagens |
| PyTorch | Framework de deep learning |
| Roboflow | Download e análise do dataset |
| Google Colab | Treinamento com GPU gratuita |
| Matplotlib | Visualização de resultados |
| Jupyter Notebook | Exploração e documentação |

---

## 📁 Estrutura do Projetosoja-weed-detection/
├── notebooks/
│   ├── 01_exploracao.ipynb        # EDA do dataset
│   ├── 02_treinamento.ipynb       # Configuração e treino
│   └── 03_inferencia.ipynb        # Detecção e controle de bicos
├── models/
│   └── soja_weed_v1/
│       └── weights/
│           └── best.pt            # Modelo treinado
├── data/
│   └── raw/
│       └── weed-crop-aerial-1/    # Dataset
└── README.md---

## ▶️ Como Reproduzir

```bash
# Clone o repositório
git clone https://github.com/TONIRESILIENTE/soja-weed-detection.git
cd soja-weed-detection

# Crie o ambiente virtual
python -m venv venv
venv\Scripts\activate  # Windows

# Instale as dependências
pip install ultralytics opencv-python matplotlib roboflow jupyter

# Execute os notebooks na ordem
# 01_exploracao.ipynb → 02_treinamento.ipynb → 03_inferencia.ipynb
```

---

## 📚 Referências

- Dataset: [Weed Crop Aerial — Roboflow 100](https://universe.roboflow.com/roboflow-100/weed-crop-aerial)
- Fonte original: Kaspars Sudars, Janis Jasko, Ivars Namatevs, Liva Ozola, Niks Badaukis
- Licença do dataset: CC BY 4.0
- Modelo base: [YOLOv8 — Ultralytics](https://github.com/ultralytics/ultralytics)
- Inspiração comercial: John Deere See & Spray

---

## 👤 Autor

**Toni Almeida Muniz**
Profissional de manutenção industrial em transição para dados e inteligência artificial.

[![GitHub](https://img.shields.io/badge/GitHub-TONIRESILIENTE-black)](https://github.com/TONIRESILIENTE)
