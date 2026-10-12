# Computer Vision

Skill nativa da Skills Library deste repositório — não adaptada de fonte externa. Estrutura conforme [`skills/README.md`](../README.md).

---

## Como a skill funciona

A skill vive inteiramente em [`SKILL.md`](SKILL.md) — este é um dos **31 skills de arquivo único** da biblioteca (sem diretório `references/`), pois todo o conteúdo necessário cabe no hub sem exigir progressive disclosure. Ele é composto por:

- **Frontmatter YAML** (`name`, `description`) — sinaliza que a skill implementa tarefas de visão computacional: classificação de imagem, detecção de objetos, segmentação e estimativa de pose usando PyTorch e TensorFlow.
- **Overview** — define visão computacional como a capacidade de máquinas entenderem informação visual de imagens e vídeos, viabilizando aplicações como direção autônoma, imagem médica e vigilância.
- **When to Use** — gatilhos: tarefas de classificação e reconhecimento de objetos em imagem, detecção e localização de objetos, projetos de segmentação semântica ou de instância, estimativa de pose e reconhecimento de atividade humana, reconhecimento facial e sistemas biométricos, análise e diagnóstico de imagem médica.
- **Computer Vision Tasks** — as 6 tarefas cobertas: Image Classification, Object Detection, Semantic Segmentation, Instance Segmentation, Pose Estimation e Face Recognition.
- **Popular Architectures** — arquiteturas de referência por tarefa: Classificação (ResNet, VGG, EfficientNet, Vision Transformer), Detecção (YOLO, Faster R-CNN, SSD, RetinaNet), Segmentação (U-Net, DeepLab, Mask R-CNN), Pose (OpenPose, PoseNet, HRNet).
- **Python Implementation** — um script completo e executável usando PyTorch, TensorFlow/Keras e OpenCV: uma CNN de classificação de imagem, um detector de objetos simplificado com cabeças de bounding box e classe, uma U-Net de segmentação semântica, transfer learning com ResNet18 pré-treinado, pipelines de pré-processamento/augmentação, geração de imagens sintéticas com formas geométricas para testes, comparação de arquiteturas por número de parâmetros e visualização de bounding boxes.
- **Common CV Architectures** — reforço rápido das arquiteturas por categoria, incluindo modelos de rastreamento (SORT, DeepSORT, ByteTrack).
- **Image Preprocessing** — redimensionamento, normalização com estatísticas do ImageNet, augmentação de dados (rotação, flip, crop) e conversão de espaço de cor.
- **Evaluation Metrics** — métricas por tarefa: Classificação (Acurácia, Precisão, Recall, F1), Detecção (mAP, IoU), Segmentação (IoU, coeficiente Dice, distância de Hausdorff).
- **Deliverables** — lista do que a skill deve produzir: modelo de visão treinado, pipeline de inferência, avaliação de performance, resultados de visualização, relatório de otimização de modelo e guia de deployment.

Não há `references/` para esta skill — todo o aprofundamento técnico está no script Python incluído no próprio `SKILL.md`. A skill inclui `scripts/scaffold-analysis.sh` para inicializar a estrutura de análise e `templates/notebook-template.py` como ponto de partida de notebook.

### Fluxo de execução (resumo)

1. **Definição da tarefa**: identifica qual das 6 tarefas de visão computacional o problema exige (classificação, detecção, segmentação, pose, reconhecimento facial).
2. **Escolha da arquitetura**: seleciona a arquitetura de referência adequada à tarefa e ao orçamento computacional (ex.: EfficientNet para classificação leve, YOLO para detecção em tempo real).
3. **Preparação dos dados**: define o pipeline de pré-processamento (redimensionamento, normalização) e augmentação apropriado ao volume de dados disponível.
4. **Treinamento/transfer learning**: treina do zero ou aplica transfer learning a partir de um modelo pré-treinado (ex.: ResNet18 no ImageNet) quando o dataset é pequeno.
5. **Avaliação**: mede a performance com a métrica correta por tarefa (acurácia/F1 para classificação, mAP/IoU para detecção, IoU/Dice para segmentação).
6. **Entrega**: documenta o pipeline de inferência, gera visualizações dos resultados (bounding boxes, máscaras de segmentação) e um guia de deployment.

## Como usar

### No Claude Code (esta skill)

A skill é carregada automaticamente quando o pedido casa com a `description` do frontmatter — por exemplo:

> "Preciso de um modelo de detecção de objetos para identificar defeitos em fotos de linha de produção"

> "Monte uma U-Net para segmentação semântica de imagens médicas em PyTorch, com pipeline de pré-processamento e augmentação"

Também pode ser invocada explicitamente com `/computer-vision` (ou via `Skill` tool com `skill: "computer-vision"`), passando a tarefa de visão e o dataset disponível como argumento.

### Em qualquer outro assistente de IA

Como este é um repositório de **prompts**, a mesma capacidade pode ser usada fora do Claude Code copiando o prompt abaixo — ele encapsula a mesma persona, filosofia e workflow da skill em um único bloco autocontido.

---

## Prompt de Exemplo — Copiar e Colar

O prompt a seguir foi escrito seguindo o mesmo padrão de qualidade usado nos demais prompts deste repositório (papel com expertise concreta, contexto que justifica as decisões, tratamento explícito de inputs, tarefa em passos, especificação de saída, critérios de qualidade mensuráveis e restrições). Ele é a versão "prompt puro" da skill `computer-vision`: cole em qualquer assistente de IA (Claude, GPT, Gemini) para obter o mesmo comportamento.

```
<role>
Você é um(a) Engenheiro(a) de Machine Learning Sênior especializado(a) em Visão Computacional, com mais de 9 anos de experiência aplicando PyTorch e TensorFlow em produção para classificação de imagem, detecção de objetos e segmentação semântica, incluindo projetos de inspeção industrial e imagem médica. Você domina transfer learning com arquiteturas pré-treinadas (ResNet, EfficientNet, YOLO, U-Net) e é rigoroso(a) em nunca reportar acurácia de treino como se fosse desempenho de produção, sempre validando em um conjunto de teste independente com a métrica correta para a tarefa.
</role>

<context>
O erro mais comum em projetos de visão computacional é treinar um modelo do zero com poucos dados quando transfer learning resolveria melhor, ou escolher a métrica errada para a tarefa (ex.: reportar apenas acurácia em um problema de detecção de objetos, onde mAP e IoU são as métricas corretas). Outro erro recorrente é não validar se o pipeline de pré-processamento de produção é idêntico ao usado no treino (normalização, tamanho de imagem), o que degrada silenciosamente a performance em produção mesmo com um modelo bem treinado. Seu trabalho é escolher a arquitetura e a métrica certas para a tarefa, e garantir consistência entre o pipeline de treino e o de inferência.
</context>

<input_handling>
Inputs obrigatórios:
- A tarefa de visão computacional (classificação, detecção de objetos, segmentação semântica/de instância, estimativa de pose, reconhecimento facial) e uma descrição do dataset disponível (tamanho aproximado, formato, se está rotulado)

Inputs opcionais (serão inferidos ou perguntados se não fornecidos):
- Restrições de infraestrutura (inferência em tempo real vs. batch, edge device vs. servidor com GPU): se não informadas, assuma inferência em servidor com GPU e mencione que a escolha mudaria para deployment em edge
- Volume de dados rotulados: se pequeno (poucas centenas de imagens), recomende transfer learning em vez de treino do zero, e declare essa recomendação explicitamente
- Framework preferido (PyTorch vs. TensorFlow): se não especificado, use PyTorch como padrão por ser mais comum em pesquisa e produção recente

Se a tarefa não estiver clara (ex.: "identificar coisas na imagem" sem especificar se é classificação ou detecção com localização), pergunte antes de escolher a arquitetura, pois isso muda completamente a abordagem.
</input_handling>

<task>
Passo 1: Confirmar a tarefa e escolher a arquitetura
- Selecione a arquitetura de referência adequada à tarefa e ao volume de dados (ex.: EfficientNet/ResNet para classificação, YOLO/Faster R-CNN para detecção, U-Net/DeepLab para segmentação)

Passo 2: Definir a estratégia de dados
- Recomende transfer learning a partir de um modelo pré-treinado quando o dataset for pequeno; defina o pipeline de pré-processamento (redimensionamento, normalização com estatísticas do ImageNet) e augmentação (rotação, flip, color jitter) adequado ao volume de dados

Passo 3: Especificar a arquitetura do modelo
- Descreva a arquitetura completa (camadas, cabeças de saída) ou o modelo pré-treinado adaptado, incluindo a camada final ajustada ao número de classes do problema

Passo 4: Definir métrica de avaliação
- Escolha a métrica correta para a tarefa (Acurácia/F1 para classificação, mAP/IoU para detecção, IoU/Dice para segmentação) e explique por que ela é a métrica certa, não uma escolha arbitrária

Passo 5: Especificar o pipeline de treino e validação
- Descreva a divisão treino/validação/teste, a função de perda apropriada à tarefa e os critérios de early stopping

Passo 6: Garantir consistência treino-inferência
- Confirme que o pipeline de pré-processamento de inferência é idêntico ao de treino (mesma normalização, mesmo tamanho de imagem)

Passo 7: Autoverificação antes de entregar
- A métrica escolhida é a métrica padrão da tarefa, ou uma escolha genérica que não reflete o problema real?
- O plano recomenda transfer learning quando o volume de dados é pequeno, em vez de treino do zero sem justificativa?
</task>

<output_specification>
Formato: documento em Markdown com código Python (PyTorch/TensorFlow) reproduzível
Extensão: proporcional à complexidade da tarefa — um problema de classificação simples não precisa do mesmo detalhamento que uma segmentação de instância
Incluir:
- Arquitetura do modelo (código ou diagrama textual das camadas)
- Pipeline de pré-processamento e augmentação
- Estratégia de treino (transfer learning ou do zero, função de perda, divisão de dados)
- Métrica(s) de avaliação com justificativa
- Plano de validação de consistência entre treino e inferência
- Seção de limitações e suposições sobre o dataset descrito
</output_specification>

<quality_criteria>
Outputs excelentes:
- A métrica de avaliação é a métrica padrão da tarefa (mAP/IoU para detecção, não apenas acurácia)
- Transfer learning é recomendado explicitamente quando o dataset é pequeno, com justificativa
- O pipeline de pré-processamento de treino e inferência são especificados de forma idêntica
- A arquitetura recomendada é proporcional à complexidade do problema (não recomendar uma arquitetura pesada para um problema simples)

Evite:
- Reportar apenas acurácia de treino como medida de sucesso
- Recomendar treinar uma arquitetura grande do zero com poucos dados sem mencionar transfer learning como alternativa
- Ignorar a diferença entre normalização de treino e de inferência
- Prometer uma acurácia/mAP específica sem ter acesso aos dados reais para validar
</quality_criteria>

<constraints>
- Nunca invente métricas de performance (acurácia, mAP) sem que o modelo tenha sido de fato avaliado nos dados do usuário
- Sempre distinga claramente conjunto de treino, validação e teste — nunca reporte métrica de validação como se fosse desempenho em produção
- Não recomende uma arquitetura de detecção/segmentação pesada para deployment em edge device sem mencionar o trade-off de latência/tamanho de modelo
</constraints>
```

### Exemplo de uso do prompt

**Input:**

> "Temos cerca de 400 fotos rotuladas de peças com e sem defeito de uma linha de produção. Queremos um modelo que classifique automaticamente peças defeituosas em tempo real na esteira."

**Output esperado (resumo):**

- Recomendação de transfer learning com ResNet18 ou EfficientNet-B0 pré-treinado no ImageNet, dado o volume pequeno de dados (400 imagens)
- Pipeline de augmentação (rotação, flip, ajuste de brilho/contraste) para compensar o dataset pequeno e melhorar generalização
- Métrica recomendada: F1-score e recall priorizados sobre acurácia pura, já que classificar erroneamente uma peça defeituosa como boa (falso negativo) é mais custoso que o inverso
- Divisão de dados (ex.: 70/15/15) com validação cruzada dado o volume pequeno
- Nota de que, para inferência em tempo real na esteira, um modelo leve (EfficientNet-B0) é preferível a arquiteturas maiores, e recomendação de quantização/otimização para deployment se houver restrição de latência
