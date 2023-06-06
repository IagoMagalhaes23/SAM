
# Segment Anything Model (SAM)

A segmentação — identificar quais pixels da imagem pertencem a um objeto — é uma tarefa central na visão computacional e é usada em uma ampla gama de aplicações, desde a análise de imagens científicas até a edição de fotos. Mas criar um modelo de segmentação preciso para tarefas específicas geralmente requer trabalho altamente especializado de especialistas técnicos com acesso à infraestrutura de treinamento de IA e grandes volumes de dados no domínio cuidadosamente anotados.

## Documentação

### SAM: Uma abordagem generalizada para segmentação
Anteriormente, para resolver qualquer tipo de problema de segmentação, havia duas classes de abordagens. A primeira, segmentação interativa, permitia a segmentação de qualquer classe de objeto, mas exigia que uma pessoa orientasse o método refinando iterativamente uma máscara. A segunda, a segmentação automática, permitia a segmentação de categorias específicas de objetos definidas com antecedência (por exemplo, gatos ou cadeiras), mas exigia quantidades substanciais de objetos anotados manualmente para treinar (por exemplo, milhares ou mesmo dezenas de milhares de exemplos de gatos segmentados). juntamente com os recursos de computação e conhecimento técnico para treinar o modelo de segmentação. Nenhuma das abordagens fornecia uma abordagem geral e totalmente automática para a segmentação.

SAM é uma generalização dessas duas classes de abordagens. É um modelo único que pode executar facilmente segmentação interativa e segmentação automática. A interface de solicitação do modelo (descrita brevemente) permite que ele seja usado de maneiras flexíveis que possibilitam uma ampla gama de tarefas de segmentação simplesmente projetando a solicitação certa para o modelo (cliques, caixas, texto e assim por diante). Além disso, o SAM é treinado em um conjunto de dados diversificado e de alta qualidade de mais de 1 bilhão de máscaras (coletadas como parte deste projeto), o que permite generalizar para novos tipos de objetos e imagens além do que foi observado durante o treinamento. Essa capacidade de generalizar significa que, em geral, os profissionais não precisarão mais coletar seus próprios dados de segmentação e ajustar um modelo para seu caso de uso.

(1) O SAM permite aos usuários segmentar objetos com apenas um clique ou clicando interativamente em pontos para incluir e excluir do objeto. O modelo também pode ser solicitado com uma caixa delimitadora.

(2) O SAM pode gerar várias máscaras válidas quando confrontado com ambigüidade sobre o objeto que está sendo segmentado, uma capacidade importante e necessária para resolver a segmentação no mundo real.

(3) O SAM pode localizar e mascarar automaticamente todos os objetos em uma imagem.

(4) O SAM pode gerar uma máscara de segmentação para qualquer solicitação em tempo real após pré-computar a incorporação da imagem, permitindo a interação em tempo real com o modelo.

### Como o SAM funciona: segmentação pronta
No processamento de linguagem natural e, mais recentemente, na visão computacional, um dos desenvolvimentos mais empolgantes é o dos modelos básicos que podem realizar aprendizado zero e poucos disparos para novos conjuntos de dados e tarefas usando técnicas de "prompt". Nós nos inspiramos nessa linha de trabalho.

Treinamos o SAM para retornar uma máscara de segmentação válida para qualquer prompt, onde um prompt pode ser pontos de primeiro plano/fundo, uma caixa ou máscara aproximada, texto de forma livre ou, em geral, qualquer informação indicando o que segmentar em uma imagem. A exigência de uma máscara válida significa simplesmente que, mesmo quando um prompt é ambíguo e pode se referir a vários objetos (por exemplo, um ponto em uma camisa pode indicar a camisa ou a pessoa que a veste), a saída deve ser uma máscara razoável para um desses objetos. Essa tarefa é usada para pré-treinar o modelo e resolver tarefas gerais de segmentação downstream por meio de solicitação.

Observamos que a tarefa de pré-treinamento e a coleta interativa de dados impuseram restrições específicas ao design do modelo. Em particular, o modelo precisa ser executado em tempo real em uma CPU em um navegador da Web para permitir que nossos anotadores usem o SAM interativamente em tempo real para anotar com eficiência. Embora a restrição de tempo de execução implique um compromisso entre qualidade e tempo de execução, descobrimos que um projeto simples produz bons resultados na prática.

Sob o capô, um codificador de imagem produz uma incorporação única para a imagem, enquanto um codificador leve converte qualquer solicitação em um vetor de incorporação em tempo real. Essas duas fontes de informação são combinadas em um decodificador leve que prevê máscaras de segmentação. Depois que a incorporação da imagem é calculada, o SAM pode produzir um segmento em apenas 50 milissegundos, a partir de qualquer solicitação em um navegador da web.

### Segmentando 1 bilhão de máscaras: como construímos o SA-1B
Para treinar nosso modelo, precisávamos de uma fonte de dados massiva e diversificada, que não existia no início do nosso trabalho. O conjunto de dados de segmentação que estamos lançando hoje é o maior até hoje (de longe). Os dados foram coletados usando o SAM. Em particular, os anotadores usaram o SAM para anotar imagens interativamente e, em seguida, os dados recém-anotados foram usados ​​para atualizar o SAM. Repetimos esse ciclo várias vezes para melhorar iterativamente o modelo e o conjunto de dados.

Com o SAM, coletar novas máscaras de segmentação é mais rápido do que nunca. Com nossa ferramenta, leva apenas cerca de 14 segundos para anotar interativamente uma máscara. Nosso processo de anotação por máscara é apenas 2x mais lento do que a anotação de caixas delimitadoras, que leva cerca de 7 segundos usando as interfaces de anotação mais rápidas. Em comparação com os esforços anteriores de coleta de dados de segmentação em grande escala, nosso modelo é 6,5 vezes mais rápido do que a anotação de máscara baseada em polígonos totalmente manual COCO e 2 vezes mais rápido do que o maior esforço anterior de anotação de dados, que também foi assistido por modelo.

No entanto, depender de máscaras de anotação interativa não é dimensionado o suficiente para criar nosso conjunto de dados de 1 bilhão de máscaras. Portanto, construímos um mecanismo de dados para criar nosso conjunto de dados SA-1B. Esse mecanismo de dados tem três “engrenagens”. Na primeira marcha, o modelo auxilia os anotadores, conforme descrito acima. A segunda marcha é uma mistura de anotação totalmente automática combinada com anotação assistida, ajudando a aumentar a diversidade de máscaras coletadas. A última marcha do mecanismo de dados é a criação de máscara totalmente automática, permitindo que nosso conjunto de dados seja dimensionado.

Nosso conjunto de dados final inclui mais de 1,1 bilhão de máscaras de segmentação coletadas em cerca de 11 milhões de imagens licenciadas e de preservação da privacidade. O SA-1B tem 400 vezes mais máscaras do que qualquer conjunto de dados de segmentação existente e, conforme verificado por estudos de avaliação humana, as máscaras são de alta qualidade e diversidade e, em alguns casos, até comparáveis ​​em qualidade às máscaras dos conjuntos de dados anteriores muito menores e totalmente anotados manualmente .


Os recursos do Segment Anything são o resultado do treinamento em milhões de imagens e máscaras coletadas usando um mecanismo de dados. O resultado é um conjunto de dados de mais de 1 bilhão de máscaras de segmentação – 400 vezes maior do que qualquer conjunto de dados de segmentação anterior.
As imagens para SA-1B foram obtidas por meio de um provedor de fotos de vários países que abrangem um conjunto diversificado de regiões geográficas e níveis de renda. Embora reconheçamos que certas regiões geográficas ainda estão sub-representadas, o SA-1B tem um número maior de imagens e uma representação geral melhor em todas as regiões do que os conjuntos de dados de segmentação anteriores. Além disso, analisamos possíveis vieses de nosso modelo em relação à apresentação de gênero percebida, tom de pele percebido e faixa etária percebida das pessoas, e descobrimos que o SAM funciona de maneira semelhante em diferentes grupos. Juntos, esperamos que isso torne nosso trabalho mais igualitário para uso em casos de uso do mundo real.

Embora o SA-1B tenha possibilitado nossa pesquisa, ele também pode permitir que outros pesquisadores treinem modelos básicos para segmentação de imagens. Esperamos ainda que esses dados possam se tornar uma base para novos conjuntos de dados com anotações adicionais, como uma descrição de texto associada a cada máscara.

## Demonstração

Os arquivos desenvolvidos no Google Colab apresentam demostrações do uso do SAM.
- SAM
Neste projeto, é apresentado o uso do SAM para segmentação de uma imagem.
- YOLO_NAS + SAM
Já neste projeto, é apresentado o uso do YOLO NAS para detectar objetos em uma imagem e com o SAM segmentar objetos especificos.

## Aprendizados

- SAM
- YOLO NAS
- Segmentação
- Detecção de objetos

## Referência

 - [Segment anything foundation model image segmentation](https://ai.facebook.com/blog/segment-anything-foundation-model-image-segmentation/)
 - [Segment Anything](https://segment-anything.com/)
