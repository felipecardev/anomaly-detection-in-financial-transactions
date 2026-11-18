# 💳 Detecção de Fraude em Cartão de Crédito com Machine Learning

Projeto desenvolvido para identificação de transações fraudulentas em cartões de crédito utilizando técnicas avançadas de Machine Learning, avaliação crítica de métricas em dados severamente desbalanceados e explicabilidade com SHAP.

---

## 1. O Problema e o Desafio do Desbalanceamento

Em sistemas de pagamento reais, transações fraudulentas são eventos extremamente raros. Nesta base, apenas **0,17% das transações são fraudes**.

### Por que a Acurácia não serve aqui?
A acurácia tradicional mede simplesmente a taxa global de acertos. Em uma base onde 99,83% das operações são legítimas, um "modelo ingênuo" que carimba todas as transações como legítimas atingiria **99,83% de acurácia**, mas deixaria **100% das fraudes passarem**, gerando um prejuízo financeiro catastrófico.

Por isso, a avaliação deste projeto focou exclusivamente na **classe minoritária (Fraude - Classe 1)** através de:
- **Precisão (Precision):** Percentual de alertas de fraude que eram realmente fraudes. Priorizada para evitar falsos positivos (bloquear cartões de clientes inocentes e causar atrito/churn).
- **Recall (Sensibilidade):** Percentual de fraudes reais que o modelo conseguiu interceptar. Priorizada para estancar o rombo financeiro das fraudes não detectadas.
- **F1-Score:** Balanço harmônico entre Precisão e Recall.
- **Curva Precision-Recall (PR-AUC):** Adotada no lugar da Curva ROC tradicional, pois a ROC é artificialmente inflacionada pelo volume massivo de Verdadeiros Negativos (transações normais).

---

## 2. Preparação dos Dados e Arquitetura do Pipeline

Para garantir reprodutibilidade e evitar vazamento de dados (*data leakage*):

1. **Divisão Estratificada (`stratify=y`):** A divisão entre treino e teste foi realizada preservando a exata proporção de 0,17% de fraudes em ambos os conjuntos.
2. **Scikit-Learn Pipeline:** O pré-processamento foi encapsulado dentro de um `Pipeline` junto com o estimador, garantindo que o escalonamento (`StandardScaler`) aprenda apenas com os dados de treino (`fit_transform`) e seja apenas aplicado no teste (`transform`).

---

## 3. Comparação de Modelos

Ao longo do desenvolvimento, foram testadas diferentes abordagens para lidar com a não-linearidade dos dados e o desbalanceamento severo:

| Modelo | Configuração / Estratégia | Precisão (Fraude) | Recall (Fraude) | F1-Score | Diagnóstico Prático |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Regressão Logística** | (Threshold 0.5) | 0.86 | 0.64 | 0.73 | Modelo linear simples; boa precisão, mas deixa passar 36% das fraudes. |
| **Regressão Logística** | `class_weight='balanced'` | 0.06 | 0.86 | 0.10 | "Segurança paranóico": pegou as fraudes, mas 94% dos alertas eram falsos alarmes. |
| **Random Forest** | `class_weight='balanced'` (Threshold 0.5) | 0.96 | 0.70 | 0.81 | Modelo não-linear robusto; capturou regras complexas com altíssima confiança. |
| **Random Forest** | **Limiar Otimizado (Threshold 0.3)** | **0.80** | **0.80** | **0.80** | **Ponto ótimo de negócio:** capturou 4 em cada 5 fraudes com 80% de precisão nos alertas. |

> **Conclusão:** O Random Forest superou a Regressão Logística por conseguir criar fronteiras de decisão não-lineares, combinando múltiplas variáveis simultaneamente sem explodir os falsos alarmes.

---

## 4. Otimização do Limiar de Decisão e Explicabilidade (SHAP)

### Calibração do Limiar (*Threshold Tuning*)
Avaliando a **Curva Precision-Recall** (Average Precision = **0.815**), observou-se que a precisão se mantinha próxima a 95% até cerca de 78% de Recall, sofrendo uma queda acentuada após 84%.

Para maximizar a captura de fraudes sem inviabilizar a operação com falsos alarmes, o limiar de decisão foi reduzido de **0.50 para 0.30**:
- **Recall subiu para 0.80:** O sistema agora impede 80% das fraudes (contra 70% do limiar padrão).
- **Precisão em 0.80:** Apenas 20% dos alertas disparados são falsos positivos, o que viabiliza tratativas amigáveis de verificação (ex: confirmação via notificação push no app do banco, em vez de bloqueio direto).

### Explicabilidade com SHAP (*SHapley Additive exPlanations*)
Para remover a opacidade do Random Forest ("caixa-preta"), utilizou-se o **SHAP TreeExplainer**:
- **Principais variáveis de risco:** As variáveis V14 e V4 apresentaram os maiores impactos no valor de SHAP..
- **Comportamento observado:** As variáveis anonimizadas V1 a V28 já estavam normalizadas via PCA.

---

## 5. Diferenciais em Relação ao Modelo Base da Expert

Em relação à solução apresentada na aula/mentoria original, este projeto implementou as seguintes melhorias:

1. **Pipeline Anti-Leakage:** Uso do `sklearn.pipeline.Pipeline` para garantir isolamento estrito entre treino e teste em ambientes de produção.
2. **Diagnóstico Crítico de `class_weight`:** Identificação prática do colapso de precisão gerado por pesos automáticos em modelos lineares.
3. **Análise por Curva PR em vez de ROC:** Adoção da Curva Precision-Recall (PR-AUC) como métrica primária de decisão técnica.
4. **Calibração Ativa de Limiar de Decisão:** Ajuste orientado a impacto financeiro/produto (trade-off entre custo da fraude vs atrito com cliente legítimo).
5. **Auditoria de Interpretabilidade (XAI):** Inclusão de análise de SHAP para justificar os motivos de bloqueio de transações.

---

## 🛠️ Tecnologias Utilizadas
- **Python 3.x**
- **Scikit-Learn** (Pipelines, Estimadores e Métricas de Classificação)
- **SHAP** (Explicabilidade de Modelos)
- **Matplotlib / Seaborn** (Visualização de Dados)
- **Pandas / NumPy** (Manipulação de Dados)

---
