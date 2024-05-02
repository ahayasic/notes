# Model Serving

## Introdução

O **model serving** é a etapa do fluxo de criação de um modelo de machine learning (ML) que tem como objetivo disponibilizar o modelo para uso em conjunto de estratégias de implantação (deployment).

A forma como um modelo é disponibilizado para a execução de inferências afeta diretamente, tanto interação do usuário (ou aplicação) com ele, quanto seu desempenho e manutenibilidade. Por isso, ao dedicir qual estratégia utilizar, devemos levar em consideração fatores como:

- Padrão de treinamento e inferência (em lotes ou fluxo)
- Disponibilidade offline (possível fazer inferências offline ou apenas online)
- Latência de rede (caso necessário envio de dados)
- Segurança de dados sensíveis
- Padrão de atualização do modelo
- Recursos de computação disponível

De fato, esses são os principais aspectos que nos levam a escolher uma estratégia de *serving*.

!!! note "Serving x Deployment"
    Termos como _model serving_ e _model deployment_ são utilizados de forma intercambiável na interwebsv:smile:

    Para tornar as explicações mais claras e consistentes, vamos definir:

      - _Model serving_. **Forma** que um modelo é disponiblizado para inferências
      - _Model deployment_. **Como** a forma que o modelo é disponiblizado é realizada

    Por exemplo, podemos servir um modelo como um serviço encapsulado em um container e cuja implantação (ou _deployment_) se dá através de um processo de CI/CD em *Modo Canário*.

## Padrões de Treinamento e Inferência

Os padrões de treinamento e inferência são fatores fundamentais na decisão de como um modelo deve ser disponibilizado, uma vez que há estratégias de serving que não suportam um determinado padrão (ou combinação de padrões) de treinamento e inferência.

### Padrões de (re)Treinamento de Modelos

Considerando um cenário onde um modelo já está treinado e pronto para produção, a preocupação com seu padrão de treinamento se dá pela necessidade do seu **retreinamento**.

Afinal, uma vez que o modelo está em produção, ele é exposto continuamente a dados novos do mundo real (e não apenas a uma amostra estática) e, consequentemente, torna-se obsoleto (fenômeno conhecido como *model decay*). Para que o modelo em produção mantenha um bom desempenho na maior parte do tempo, ele deve ser retreinado com uma certa frequência.

Modelos de ML são treinados de duas formas:

- Em Lotes (batches), conhecido como Aprendizado em Lotes ou Offline/Batch Learning.
- Incremental, conhecido como Aprendizado Incremental Online Learning.

Logo, o retreinamento também pode acontecer em lotes ou de forma incremental.

#### Em Lotes

No retreinamento em lotes, um modelo é retreinado após um tempo considerável em produção. Esse retreinamento pode acontecer em intervalos fixos (por exemplo, a cada 30 dias) ou quando um limite inferior de desempenho é atingindo.

O principal problema do retreinamento em lotes é que, no caso de um intervalos fixo, a degradação do modelo pode acontecer em diferentes velocidades, o que torna difícil encontrar uma frequência de retreinamento ideal.

Por outro lado, monitorar um modelo em produção e retreiná-lo com base em alguma métrica de desempenho, é um processo consideravelmente complexo tanto pela definição e cálculo da métrica em si, quanto pela necessidade de toda uma solução de monitoramento.

#### Aprendizado Incremental

No "aprendizado incremental", o modelo é treinado regularmente conforme novos dados são disponibilizados à aplicação (e.g. real-time data streams). O retreinamento pode ocorrer em um único dado novo ou pequenos grupos de dados, denominados *mini-batches*.

O principal problema do Aprendizado Incremental é que, quando em produção, a entrada de dados ruins (e.g, ruídos) tende a prejudicar consideravalmente o desempenho do modelo.

###Padrões de Inferência do Modelo

Da mesma forma que treinados, modelos de ML podem ser dispostos para inferir dados de duas formas: Em Batches ou Sob-demanda.

- Na inferência em batches, o modelo executa predições sobre um "grande" volume de dados de uma vez e só então retorna os resultados.
- Na inferência em tempo real, as predições são executadas sob-demanda para cada dado (ou pequenos conjuntos) de entrada e, em seguida, o resultado já é retornado.

!!! note "O que é grande?"
    Grande é relativo, não é mesmo? De qualquer forma, no contexto de inferência em batches significa que ao invés de executar inferências sob-demanda em pequenos conjuntos de dados (e.g. 1 a ~300 instâncias), executa-se para várias entradas em conjunto (e.g., mais do que 500 ou 10 mil instâncias)

## Padrões de Model Serving

Existem diversas abordagens para servir um modelo, cada um com suas vantagens e desvantagens. Alguns exemplos são:

- Modelo como Parte da Aplicação (Static Deployment)
- Model-as-a-Dependency (MaaD)
- Model-as-a-Service (MaaS)
- Serverless Servig/Deployment
- Hybrid-Serving (Federated Learning)

## Referências

- [Three Levels of ML Software](https://ml-ops.org/content/three-levels-of-ml-software#model-machine-learning-pipelines)
- [Machine Learning Engineering by Andriy Burkov](http://www.mlebook.com/)

## Appendix

# Model as a Service (MaaS)

## Introdução

A estratégia Model-as-a-Service é a mais adotada para servir modelos de ML. Nela, o modelo é abstraído em um serviço (tipicamente web) que recebe requisições de inferência. Após executada as inferências, o serviço retorna os resultados ao requerente.

Dependendo da implementação, o serviço pode receber tanto batches de dados para inferência, quanto entradas individuais.

![maas-generic](https://raw.githubusercontent.com/ahayasic/machine-learning-engineering-notes/main/docs/assets/serving/maas-generic.png)
<p class="post__img_legend">
  <b>Imagem:</b> Exemplo genérico da arquitetura Model-as-a-Service, onde um dispositivo faz requisições de inferência através de uma interface e então recebe como resposta o resultado da predição.
</p>

!!! info "Sobre Serviços"

    No contexto de arquitetura de software, um serviço é uma unidade de software auto-contida responsável por executar uma tarefa específica. Um serviço deve conter todo o código e dados necessários para executar sua tarefa, sendo implantando (geralmente) em um ambiente totalmente dedicado para si. Demais componentes do software (ou arquitetura) interagem com o serviço através de uma API definida sobre protocolos de comunicação de rede, tais como REST APIs e HTTP/HTTPS.

    O propósito principal de um serviço é fornecer, ao sistema, acesso a um conjunto de funcionalidades, de modo que o serviço provedor seja totalmente reutilizável e independente do resto do sistema. Com isso, podemos desenvolver, construir e implantar o serviço de forma totalmente desacoplada dos demais componentes.


## Vantagens x Desvantagens

Model as as Service é uma estratégia adequada para a maioria das situações. A principal ressalva é que inferências só estarão disponíveis caso o usuário esteja online.

### Vantagens

As principais vantagens de se implantar modelos de ML como serviços são:

- Integração com o restante do sistema, tecnologias e processos extremamente simplificada.
- Gerenciamento do modelo simplificado.

### Desvantagens

Já os contras são:

- Necessário mais aplicações para gerenciar.
- Não é possível realizar inferências offline.
- Latência de inferência consideravelmente maior quanto comparado com inferências offline, uma vez que os dados precisam ser enviados pela rede.
- Dados sensíveis do usuário são enviados pela rede e executados em um domínio externo ao dele.

### Arquiteturas baseadas em Máquinas Virtuais e Containers

Partindo de uma perspectiva de escalabilidade, podemos implantar os serviços de predição de duas formas principais: máquinas virtuais ou containers.

#### Máquinas Virtuais
Com máquinas virtuais (e.g. instâncias AWS EC2), usamos uma ou mais instâncias onde o serviço web roda em paralelo (no caso de mais de uma instância).

A necessidade de diversas instâncias se dá quando há um grande volume de requisições a ser atendido. Neste caso, também incluímos um load balancer que irá receber as requisições e redirecioná-las para a instância com maior disponibilidade.

Note, entretanto, que a necessidade de virtualização prejudica consideravelmente a eficiência de uso dos recursos de cada instância.

![maas-virtual-instances](https://raw.githubusercontent.com/ahayasic/machine-learning-engineering-notes/main/docs/assets/serving/maas-virtual-instances.png)
<p class="post__img_legend">
  <b>Fonte:</b> <a target="_blank" href="http://www.mlebook.com/">Machine Learning Engineering by Andriy Burkov (2020)</a>
</p>

#### Containers
Diferente de máquinas virtuais, containers são consideravelmente mais eficientes no uso de recursos, tornando os gastos menores sem perda de desempenho (onde desempenho significa atender uma alta demanda de requisições).

Dessa forma, podemos usar um orquestrador de containers como o Kubernetes para gerenciar um conjunto de containers executando em uma ou mais máquinas dentro de um cluster auto-escalável. Com essa estratégia, podemos reduzir o número de réplicas (ou seja, containers ativos em paralelo) para zero, quando não houver qualquer requisição e aumentar para um número suficientemente grande quando houver um grande volume de requisições.

No geral, a implantação de serviços de predição em containers é a mais indicada.

![maas-containers](https://raw.githubusercontent.com/ahayasic/machine-learning-engineering-notes/main/docs/assets/serving/maas-containers.png)
<p class="post__img_legend">
  <b>Fonte:</b> <a target="_blank" href="http://www.mlebook.com/">Machine Learning Engineering by Andriy Burkov (2020)</a>
</p>

### Protocolos de Comunicação

Assim como em qualquer serviço convencional (aplicações web, banco de dados, etc), no Model as a Service (MaaS) a interação com o modelo acontece através de APIs definidas sobre protocolos de comunicação em rede.

As arquiteturas de API mais comuns são:

- REST (Representational State Transfer) com protocolo HTTP
- gRPC (Google Remote Procedure Call) com HTTP 2.0.

## Exemplo

A fim de solidificar o conhecimento, segue um exemplo prático de serving (ou seria implantação? :grin:) de um modelo como um serviço.

### Model as a Service com FastAPI e Docker

O Python contém diversos pacotes como Flask, FastAPI e Uvicorn que nos permite definir facilmente uma API REST, tal como servidores altamente eficientes.

Para detalhes sobre a implementação de APIs em Python acesse a página de [criação de APIs em Python](#):

Supondo que já temos um modelo treinado (no caso, para a análise de sentimentos), a primeira etapa é definir o endpoint de requisição para inferências e qual método REST será utilizado. Geralmente, nomeamos o endpoint como  ``predict`` ou ``inference``  e usamos o método ``POST`` (uma vez que com ``POST`` podemos definir o corpo da requisição).

Para definir o corpo da requisição no FastAPI, utilizamos a estratégia de definir uma classe ``UserRequest`` derivada da classe ``BaseModel`` de ``pydantic``.

Em seguida, basta chamar o preditor para executar a inferência e então retornamos os resultados.

```python
# my_predictor.py

from fastapi import FastAPI
from textblob import TextBlob

app = FastAPI(title="ML Model as a Service")

class UserRequest(BaseModel):
    sentence: str

@app.post("/predict/")
async def predict(user_request: UserRequest):
    testimonial = TextBlob(user_request.sentence)
    return {
        "result": f"Polarity is {testimonial.sentiment.polarity} and subjectivity is {testimonial.sentiment.subjectivity}"  # type: ignore
    }
```

Para executar as requisições, iniciamos um servidor (com o uvicorn), fazendo a chamada no mesmo nível do arquivo Python onde está definda a API:

```shell
$ uvicorn my_predictor:app --port <port_number>
```

Então acessamos o endereço [`http://127.0.0.1:<port_number>/docs`](http://127.0.0.1:8000/docs) e usamos a documentação de API fornecida pela Swagger UI para enviar requisições.

Ainda, dado que o endpoint defindo é da forma `http://<ip_address>:<port_number>/predict/?data=<data>`, podemos enviar requisições de qualquer forma, incluindo via `curl`

```shell
curl -X 'POST' \
  'http://localhost:8000/predict/' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "sentence": "Life is beautiful, enjoy it!"
}'
```

Podemos também definir o endpoint para receber requisições ``GET``, eliminando a passagem de dados via corpo de requisição (embora isso seja mais uma vantagem do que desvantagem). 

```python
# my_predictor.py

from fastapi import FastAPI
from textblob import TextBlob

app = FastAPI(title="ML Model as a Service")

@app.get("/predict/")
async def predict(sentence):
    testimonial = TextBlob(sentence)
    return {
        "result": f"Polarity is {testimonial.sentiment.polarity} and subjectivity is {testimonial.sentiment.subjectivity}"  # type: ignore
    }
```

Então,  executamos as chamadas via web browser, por exemplo:

```shell
http://localhost:8000/predict/?sentence=%22Life%20is%20beautiful,%20enjoy%20it%22
```

#### Conclusão

Este foi um exemplo prático extremamente simples. No mundo real, há diversas outras considerações que devemos levar em conta durante o serving de um modelo como um serviço, por exemplo:

- Quando um modelo é atualizado, precisamos que isso se reflita no serviço. Contudo, não podemos simplesmente desligar e religar um serviço. Portanto, como atualizar os modelos para serviços em produção?
- É muito possível que existam períodos em que a quantidade de requisição é grande o suficiente para derrubar o serviço, tornando necessário o uso de um load balancer. Quando isso deve acontecer? Como deve ser o load balancer?

Comentários sobre estes problemas serão incluídos futuramente!

??? tip "Interessado em mais exemplos? :eyes:"

    Caso queira ver mais exemplos de MaaS acesse o repositório [ahayasic/model-as-a-service-examples](#). Mas já adianto que não há uma explicação aprofundada para nenhum dos exemplos. Apenas código ¯\\\_(ツ)\_/¯

# Modelo como Parte da Aplicação (Static Deployment)

## Introdução

Nesta abordagem $-$ que Andriy Burkov chama de Static Deployment $-$ o modelo é empacotado como parte da aplicação que então é instalada através de um arquivo de distribuição de aplicações, como, por exemplo: arquivo binário executável, JAR, APK, DEB, etc.

## Vantagens x Desvantagens

Static deployment é uma estratégia adequada para modelos simples que precisam estar disponíveis o tempo todo (mesmo se o usuário estiver offline). Também é adequado para inferências em batches ou fluxo. Porém, o retreinamento tende estar restrito a lotes, uma vez que é necessário atualizar toda a aplicação para incluir uma nova versão do modelo.

### Vantagens

- Não é preciso enviar dados do usuário para um servidor (ou qualquer recurso) externo ao dispositivo do usuário
- O modelo estará sempre disponível, mesmo se o usuário estiver offline (sem conexão com a Internet)
- Caso seja um modelo simples, sem necessidade de computações rápidas ou pesadas, o tempo de inferência é muito mais rápido na abordagem "estático" quando comparado com qualquer outra estratégia

### Desvantagens

- Para atualizar o modelo é necessário atualizar toda a aplicação (ou seja, reconstruir o arquivo de distribuição da aplicação mesmo que apenas o modelo tenha sofrido alterações)
- Executar o monitoramento de performance do modelo é extremamente difícil
- Se a computação do modelo for cara, executá-la no dispositivo do usuário pode ser ineficiente ou prejudicar a experiência do usuário

# Model as a Dependency (MaaD)

## Introdução
A estratégia MaaD é bem parecida com *static deployment*. Contudo, ao invés de empacotarmos o modelo como parte da aplicação, ele é definido como uma dependência e empacotado de forma que seja possível atualizá-lo individualmente.

Por exemplo, podemos empacotar um modelo como:

- Um pacote instalável (e.g. um pacote Python) definido como uma dependência da aplicação.
- Arquivo serializado que é importado (e deserializado) pela aplicação durante sua inicialização ou em tempo de execução
- Arquivo com os parâmetros do modelo que pode ser utilizado para atualizá-lo.

Neste cenário, para utilizar o modelo, a aplicação só precisa invocar um método de inferência deste.

## Vantagens x Desvantagens

Model as as Dependency é uma estratégia adequada para modelos simples que precisam estar disponíveis o tempo todo (mesmo se o usuário estiver offline). Também é adequado tanto para inferências quanto retreinamento em batches ou fluxo.

Por outro lado, modelos "pesados" devem ser evitados pelo alto custo de computação no dispositivo do usuário.

### Vantagens

- Não é preciso enviar dados do usuário para um servidor (ou qualquer recurso) externo ao dispositivo do usuário
- O modelo estará sempre disponível, mesmo se o usuário estiver offline.
- Caso seja um modelo simples, sem necessidade de computações rápidas ou pesadas, o tempo de inferência é muito mais rápido quando comparado com qualquer outra estratégia
- O modelo pode ser atualizado sem a necessidade de atualizar toda a aplicação

### Desvantagens

- Executar o monitoramento de performance do modelo é extremamente difícil
- Se a computação do modelo for cara, executá-la no dispositivo do usuário pode ser ineficiente ou prejudicar a experiência do usuário
- Dependendo da estratégia de empacotamento do modelo, técnicas de engenharia reversa podem ser aplicadas para manipular o resultado das inferências

## Exemplos

# Federated Learning

## Introdução

Hybrid-Serving, mais conhecido como Federated Learning é uma forma relativamente nova, porém, em alta, de servir modelos aos usuários, principalmente de dispositivos móveis.

Basicamente, trata-se de estratégia onde um modelo genérico é disponibilizado para uma grande quantidade de usuários e, então, cada usuário passa a ter um modelo específico para si que é retreinado (ou especializado) em seus dados.

Mais precisamente:

- Há um modelo genérico no lado do servidor (ou server-side) pré-treinado em dados do mundo real.
  - Tal modelo é usado como ponto de partida para novos usuários da aplicação.

- Do lado dos usuários (ou user-side), há modelos especializados e únicos para cada usuário (que partem do modelo genérico no server-side), de forma que o retreinamento (i.e. especialização) destes modelos para este usuário ocorre no dispositivo do usuário.
- Uma vez especializados, os (hiper)parâmetros de cada modelo são enviados para o servidor. Assim, o modelo do servidor é ajustado a fim de que as tendências reais de toda a comunidade de usuários sejam cobertas pelo modelo e, então, este novo modelo passa a ser o novo modelo inicial para todos os usuários.
  - Para que não haja desvantagens aos usuários, o processo de atualização dos modelos ocorre somente quando o aparelho está ocioso, conectado à uma rede WiFi e carregando.
  - Ainda, os testes são feitos nos dispositivos. Portanto, o modelo recém-adotado do servidor é enviado aos dispositivos e testado quanto à funcionalidade.

A principal vantagem dessa abordagem é que:

- Os dados pessoais necessários para o treinamento (e teste) nunca saem do domínio do usuario. Enquanto que ainda assim é possível atualizar os modelos com base nas tendências da comunidade.
  - Em outras palavras, é possível treinar modelos de alta precisão sem ter que armazenar toneladas de dados (provavelmente pessoais) na nuvem.

Porém, a grande desvantagem é que a especialização do modelo é custosa para usuários. Afinal, os modelos de ML são desenvolvidos com conjuntos de dados grandes e homogêneos em um hardware poderoso.

## Referências

- [Three Levels of ML Software](https://ml-ops.org/content/three-levels-of-ml-software#model-machine-learning-pipelines)