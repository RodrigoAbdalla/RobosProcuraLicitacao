# Projeto RPA - Captura Leads

Esse projeto foi feito com o intuito de criar robôs que possam capturar licitações para a área de RPA da empresa.

Vale lembrar que além das pastas com os robôs, existem outras configurações e processos que devem ser feitos antes de tentar rodar. Segue um resumo de cada step:

## prj_dispatcher_licitacoes
Realiza a leitura do excel de termos que devem ser procurados, envia cada termo para todos os steps que procuram as licitações.

## prj_performer_comprasNet
Performer que procura os termos enviados pelo dispatcher no site "Compras.Net", sempre que volta um resultado na pesquisa, é feito o download e enviado por e-mail.

Observação: Não foi o foco do projeto e em caso de continuidade, deve ser testado novamente.

## prj_performer_portalTransparencia
Performer que procura os termos enviados pelo dispatcher no portal Transparencia, sempre que volta um resultado na pesquisa, é feito o download e enviado por e-mail.

Observação: Não foi o foco do projeto e em caso de continuidade, deve ser testado novamente. Houve também dificuldade pois nunca um resultado era retornado.

## prj_performer_google
Performer principal do projeto, onde foi dedicado praticamente a totalidade do tempo.

Nele, o robô recebe os termos que devem ser procurados, faz uma requisição na Custom Search API (API da google que realiza pesquisas de termos), coleta X resultados (estipulados via asset no orchestrator) e inicia sua busca.

Na busca, o robô entra em cada link que foi encontrado, imprime a página (CTRL + P) e envia o PDF gerado para uma skill Machine Learning previamente criada no AI Center.

Essa skill adivinhará se o site encontrado se trata de uma licitação de RPA ou não. Apenas para respostas positivas o robô adiciona no relatório que será enviado para um supervisor.

Observação: Esse é o step mais complexo do projeto, precisa de muita configuração e processos antes de começar a rodar. 
#### 1 - Custom Search API
Para utilização da Custom Search API, você deve criar um cadastro no [Google Console](https://console.cloud.google.com/) e ativar essa API (ela vem desabilitada por padrão). 

Após realizar o cadastro no Google Console e ativar a API desejada, você deve criar uma credencial (API KEY) que será usada para autenticação.

Após conseguir a API KEY, deve-se criar uma [Search Engine](https://programmablesearchengine.google.com/controlpanel/) com suas configurações desejadas no projeto. 

Ao criar a Search Engine, é gerado um ID que deve ser enviado ao Orchestrator, junto à API KEY gerada anteriormente.

É valido a realização de testes usando essa API KEY e a Search Engine ID no Postman antes de seguir para o Studio para garantir que foram criados corretamente.

#### 2 - Skill Machine Learning (AI CENTER)
Para iniciar com a configuração do AI Center, é necessário solicitar uma licença enterprise da UiPath, para então ter acesso às ferramentas necessárias.

Recomendo realizar o curso do AI Center na UiPath Academy, lá eles explicam como é possível criar projetos Machine Learning usando pacotes já criados pela UiPath ou comunidade.

No caso deste projeto, foi usado o pacote "DocumentClassifier", que é uma base de machine learning, mas totalmente bruta, que necessita que o utilizador configure e treine essa ML, para que se adeque à necessidade específica.

Por exemplo, foi necessário criar duas possibilidades de escolha, "Licitacao RPA" ou "Outros", pois não importava qual era o tipo do site, se não fosse uma licitação RPA, não deve ser considerado.

Parar criar a base da inteligência dessa ML, deve-se treinar manualmente utilizando algumas atividades do STUDIO, recomendo ver um [vídeo explicativo](https://www.youtube.com/watch?v=k3GD5L7mmRs&ab_channel=LahiruFernando) para entender esse treinamento primário.

Após esse treinamento primário, a ML é capaz de realizar algumas previsões, porém no começo a maioria é errada, é necessário então continuar com os treinamentos, dandos os feedbacks e adicionando mais cases no dataset do AI Center.

Depois de vários treinamentos, quando for perceptível que uma versão é aceitável, é possível voltar no studio e usar a skill (ML) criada, que realizará previsões se o PDF é ou não uma licitação de RPA.

Vale se atentar que o pacote UiPath.DocumentUnderstanding.ML.Activities é bem bugado e apresenta um erro TODA vez que o projeto é reiniciado.
Para resolver, é necessário toda vez fazer um downgrade para a versao anterior e um upgrade para a atual novamente, e quando rodar o robo ele consegue fazer as previsões normalmente.

Outra observação é que como se trata de uma pesquisa no google, não tem um padrão definido de sites e estruturas para licitações, então a inteligência da Machine Learning vai depender muito dos PDFs que foram treinados previamente, então novos sites e novas estruturas provavelmente não serão reconhecidos.

Outro ponto é que por se tratar de sites aleatórios, não é possível realizar captura de informações de cada site, então a probabilidade de coletar licitações que já foram fechadas é grande.

Para coletar informações mais precisas e com confiança mais alta, o correto seria criar mais performers com pesquisas diretas nos sites específicos, como no Compras Net e Portal Transparencia, não sendo necessário a Machine Learning.

