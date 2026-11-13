# CBAS: Chemically-resolved Bond-Angle Spectrum (Descritor Topológico-Geométrico - DTG)

[![Status](https://img.shields.io/badge/Status-Em%20andamento-blue)](#)
[![Instituição](https://img.shields.io/badge/Instituição-Universidade%20de%20Brasília%20(UnB)-green)](#)

O **CBAS** (Chemically-resolved Bond-Angle Spectrum), também desenvolvido sob o escopo do Descritor Topológico-Geométrico (DTG), é um descritor estrutural concebido para converter geometrias atômicas de arquivos CIF em vetores numéricos contínuos de tamanho fixo, otimizados para modelos de aprendizado de máquina. 

Em vez de utilizar grades geométricas suavizadas, o CBAS-v1 codifica diretamente a distribuição de comprimentos de ligação e ângulos no domínio espectral complexo amostrado em frequências ω. O método baseia-se na transformação espectral S(ω) = (1/N) ∑ exp(-iωx_j), gerando fases complexas (médias de cossenos e senos) para uma representação compacta e robusta de alótropos de carbono 2D e 3D.

## 🚀 Principais Características

* **Codificação Espectral:** Representação compacta sem o uso de "broadening" gaussiano.
* **Vetor Compacto (112 Features):** 64 features radiais/ligação (canal C-C) e 48 features angulares (canal C-(C,C)) para sistemas de carbono puro.
* **Periodicidade Adaptável:** Suporta topologias em modo 2D (réplicas nas direções a e b) e modo 3D (réplicas nas direções a, b e c).
* **Invariâncias Físicas:** Mantém invariância a translação, rotação e permutação de átomos do mesmo elemento.

## 📁 Estrutura do Repositório

* `00_code/`: Código fonte principal (`CBAS.py`, `analysis.py`).
* `01_structures_original/`: Arquivos CIF originais extraídos de bancos de dados.
* `02_structures_standardized/`: Estruturas com orientação padronizada.
* `03_supercells/`: Células escaladas (2x2, 3x3) para testes de invariância.
* `04_strain/`: Estruturas com strain isotrópico aplicado (-5% a +5%).
* `05_noise/`: Estruturas com deslocamentos atômicos aleatórios.
* `06_defects/`: Casos de estudo com defeitos diluídos (Stone-Wales, monovacância V1, divacância V2).
* `07_descriptors/`: Vetores de saída CSV gerados (CBAS e MBTR).
* `08_results/`: Matrizes de distância, logs de tempo, PCA e classificações.
* `09_figures/`: Visualizações, trajetórias no plano complexo e matrizes.
* `metadata.csv`: Banco de dados mestre contendo IDs e metadados de simulação.

## ⚙️ Instalação e Uso

O script central `CBAS.py` é executado via linha de comando. Recomenda-se manter os parâmetros físicos fixos para evitar *overfitting* metodológico.

**Baseline para Sistemas 2D (Graphene, Phagraphene, etc.)**
```bash
python3 CBAS.py \
  --input-dir structures_2D \
  --output cbas_2D.csv \
  --dim 2 \
  --rep 1 \
  --rcut 1.85 \
  --k-bond 32 \
  --k-angle 24 \
  --bond-w-min 0.5 --bond-w-max 10.0 \
  --angle-w-min 1.0 --angle-w-max 20.0 \
  --angle-variable cos \
  --normalization channel_count \
  --no-stats
```

**Baseline para Sistemas 2D (Graphene, Phagraphene, etc.)**
```bash

python3 CBAS.py \
  --input-dir structures_3D \
  --output cbas_3D.csv \
  --dim 3 \
  --rep 1 \
  --rcut 1.85 \
  --k-bond 32 \
  --k-angle 24 \
  --bond-w-min 0.5 --bond-w-max 10.0 \
  --angle-w-min 1.0 --angle-w-max 20.0 \
  --angle-variable cos \
  --normalization channel_count \
  --no-stats
  ```
## 🔬 Fases de Validação e Pesquisa

O desenvolvimento e validação do CBAS seguem um roteiro progressivo, estruturado para testar a robustez metodológica do descritor contra perturbações físicas e compará-lo ao estado da arte.

### Fase 1: Fundamentos e Invariâncias Físicas
O foco inicial é a validação da hipótese topológica em sistemas 2D bidimensionais de carbono. Utilizando estruturas fundamentais como Graphene, Phagraphene e Cey-graphene, esta fase isola os componentes do espectro (análises *bond-only* e *angle-only*) para observar o comportamento das trajetórias no plano complexo. Inclui testes rigorosos de estabilidade, submetendo as redes a:
* **Ruído:** Deslocamentos atômicos aleatórios para simular agitação térmica.
* **Strain Isotrópico:** Compressão e tração da rede variando de -5% a +5%.
* **Escala:** Construção de supercélulas (2x2, 3x3) para garantir a conservação da invariância translacional e rotacional.

### Fase 2: Expansão de Base e Análise de Dimensionalidade
Aplica-se o descritor a um banco de dados abrangente contendo aproximadamente 120 estruturas de carbono previamente padronizadas. O objetivo é realizar a classificação e um *benchmark* computacional em larga escala. A capacidade do CBAS de separar diferentes topologias é avaliada quantitativamente através da construção de matrizes de distância (mensurando a similaridade entre alótropos) e análises de redução de dimensionalidade (PCA - *Principal Component Analysis*).

### Fase 3: Detecção de Defeitos Locais Diluídos
Investiga-se a sensibilidade do vetor espectral na detecção de perturbações estruturais pontuais dentro de matrizes extensas. Esta fase modela cenários práticos de imperfeições em materiais 2D, acompanhando como o sinal do vetor global é alterado pela presença de concentrações diluídas de:
* Rotações de ligações (defeitos Stone-Wales).
* Monovacâncias (V1).
* Divacâncias (V2).

### Fase 4: Sistemas 3D, Empilhamento e Benchmark Final
A metodologia é expandida para investigar matrizes complexas e interações de van der Waals em três dimensões. 
* **Modelagem 3D:** Análise de alótropos densos, contrastando estruturas como Diamond e Lonsdaleite.
* **Cutoffs Progressivos:** Estudo detalhado de sistemas multicamadas, diferenciando empilhamentos de grafite (AA, AB, ABC) através do aumento seletivo do raio de corte (*rcut*).
* **Comparação com Estado da Arte:** Culmina em um *benchmark* direto contra o descritor MBTR (utilizando tensores equivalentes de $k=2$ e $k=3$). A comparação avalia criticamente o custo computacional, o tamanho final do vetor (dimensionalidade) e o poder de discriminação topológica entre os métodos.
