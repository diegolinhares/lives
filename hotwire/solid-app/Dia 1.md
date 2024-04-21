## Disclaimer

Não sou um especialista em Hotwire. Apenas uso em produção em side projects.

 Ainda estou aprendendo a fazer streams!

# Turbo

## Instalação

```ruby
# Gemfile
gem 'turbo-rails'

# Terminal
./bin/bundle install
./bin/rails turbo:install:redis
```

Um detalhe importante: O adaptador padrão (Async) para o Action Cable que vem no Rails no *cable.yml* não suporta fazer Turbo Stream broadcasting.

Você pode testar desabilitar o turbo para seu app inteiro e ver como ele se comporta.

```js
import "@hotwired/turbo-rails"

Turbo.session.drive = false
```

# Flash messages

O Turbo a novo link que você abre, faz um cache da página atual.

Então se você estiver em um formulário, criou algum recurso na página e surgir uma flash message bonitinha dizendo que deu tudo certo; ela vai ser renderizada para o usuário como preview na próxima vez que o usuário clicar pra ver esse link. Algumas vezes é tão rápido que nem se percebe.

Não queremos esse comportamento.

Para resolver isso, coloque no seu elemento de flash message *data-turbo-temporary*.

"Turbo meu amigo, antes de fazer o cache aqui desse DOM, me tira dele. Não quero aparecer na próxima vez; no próximo preview".

```html
    <% flash.each do |type, message| %>
      <p class="<%= type %>" data-turbo-temporary><strong><%= message %></strong></p>
    <% end %>
```

# Versão 8
### Instaclick

**Que problema quer resolver?**
Apesar do aumento na largura de banda da internet, as páginas web muitas vezes não carregam mais rápido. O maior gargalo não é a quantidade de dados que podem ser transmitidos de uma vez.

O problema é a **latência** que ocorre cada vez que pedimos novos dados.

**Como resolve?**
Para contornar o problema da latência, o InstantClick utiliza um "cheat": ele começa a carregar as páginas que ele acha que você vai querer visitar antes mesmo de você clicar nelas. Por exemplo, quando você passa o mouse sobre um link (antes mesmo de clicar), ou quando toca em um link em um dispositivo móvel, o InstantClick já começa a carregar essa página. Assim, quando você efetivamente clicar, a página já estará carregada, eliminando a espera.

InstantClick monitora os eventos de "hover" (quando o cursor do mouse fica sobre um link) e "touchstart" (quando o dedo toca a tela em dispositivos móveis) para começar a pré-carregar a página. Ele usa uma combinação de técnicas chamada pjax, que envolve Ajax e pushState, para atualizar apenas o conteúdo principal da página e o título sem recarregar toda a página. Isso evita a necessidade de recompilar scripts e estilos a cada mudança de página, o que também contribui para a sensação de rapidez.

**Detalhes**
- Vem ativo por padrão no Turbo 8. [Enable InstaClick by default by afcapel · Pull Request #1162 · hotwired/turbo (github.com)](https://github.com/hotwired/turbo/pull/1162)
- Desabilitado em dispositivos móveis* [Disable InstantClick for touch events by afcapel · Pull Request #1167 · hotwired/turbo (github.com)](https://github.com/hotwired/turbo/pull/1167)
- Uso do HEADER X-Sec-Purpose: prefetch [Sec-Purpose - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Purpose)
- Para mais informações: [InstantClick — JS library to make your website instant](http://instantclick.io/)
- Você pode desabilitar na aplicação inteira adicionando: <meta name="turbo-prefetch" content="false">
- Você pode desabilitar por link usando: *data-turbo-prefetch="false"*
- Você também pode desabilitar programaticamente usando: *turbo:before-prefetch*

**Valeria a pena estudar?**
- pjax
- pushState

**Cuidados**
Os pré-carregamentos podem distorcer as estatísticas de visitas, cada carregamento pode ser contado como uma visita real, mesmo que o usuário não tenha realmente visto a página. Além disso, se alguém passar o mouse várias vezes sobre o mesmo link, isso pode gerar pedidos repetidos desnecessariamente e sobrecarrega o servidor.

*Quando usamos um celular para acessar um site e tocamos em um link (`touchstart`), o navegador começa a carregar a página. Mas, por uma característica do navegador no celular, ele também simula um clique de mouse (`mouseenter`) logo após você soltar o toque (`touchend`), tentando carregar a página novamente. Isso causa um problema porque a página tenta carregar duas vezes ao mesmo tempo. Para evitar essa confusão, os desenvolvedores decidiram desligar essa função nos celulares por enquanto, deixando-a ativa apenas nos computadores, onde esse problema não acontece.

# Morph

**Que problema quer resolver?**

O "morphing" permite atualizações suaves ao re-renderizar páginas, mantendo estados do cliente como rolagem e foco. Isso evita a complexidade das atualizações parciais, típicas em aplicações que requerem alta fidelidade na interface. A nova abordagem reduz o esforço em manter interações específicas na UI e desacopla modelos de suas representações visuais, simplificando o desenvolvimento

**Como resolve?**
Imagine que você está editando um documento e, em vez de reescrever todo o texto cada vez que faz uma mudança pequena, você só quer ajustar uma palavra ou uma frase. O Idiomorph faz algo parecido para websites. Ele ajuda a atualizar apenas as partes de uma página web que realmente precisam de mudança sem recarregar toda a página. Isso é especialmente útil para sites que mudam frequentemente, como painéis de informação em tempo real ou feeds de notícias.

Idiomorph usa um truque inteligente: ele olha para o que mudou na estrutura da página usando algo chamado "IDs". Cada parte da página que pode ser atualizada tem um ID único, como uma etiqueta em itens numa loja. Quando alguma informação precisa ser atualizada, Idiomorph compara os IDs da versão antiga da página com os da nova. Ele identifica exatamente quais partes precisam mudar, e só atualiza essas partes. Isso é feito de forma que mantenha o resto da página intacto, o que economiza tempo e recursos, pois não é necessário recarregar tudo.

Veja aqui como ele faz isso nessa animação: https://twitter.com/calebporzio/status/1679105399058837505

**Detalhes**
- [37signals Dev — A happier happy path in Turbo with morphing](https://dev.37signals.com/a-happier-happy-path-in-turbo-with-morphing/?utm_source=bhumi&utm_medium=email&utm_campaign=when-do-we-use-turbo-8-page-refresh-and-morphing)
- Preservar a posição do scroll e algum estado do DOM interno
- Fazer uma requisição para o mesmo formato de URL Neste caso, o Turbo percebe que o endereço não mudou e, ao invés de fazer uma navegação normal, ele atualiza apenas a parte da página que mudou. Isso ajuda a manter a posição de rolagem, deixando a experiência mais fluida para o usuário.
- Ao fazer broadcast de refreshes (*broadcasts_refreshes* ou )

**Como ativar?**

```ruby
# application.html.erb

# Juntinho
turbo_refreshes_with(method: :replace, scroll: :reset) # Padrão do Turbo

turbo_refreshes_with(method: :morph, scroll: :preserve) # Morph + Preservar Scroll

# Isolado
turbo_refresh_method_tag(:replace | :morph)
turbo_refresh_scroll_tag(:reset | :preserve)
```

**Como fazer broadcasts refreshes?**

```ruby
# No Model
class Task < ApplicationRecord
  broadcasts_refreshes

  # ou

  after_create_commit  -> { broadcast_refresh_later_to(model_name.plural) }
  after_update_commit  -> { broadcast_refresh_later }
  after_destroy_commit -> { broadcast_refresh }
end

# Fora do Model
Turbo::StreamsChannel.broadcast_refresh_to(@task)

# Na View
<%= turbo_stream_from @task %>
```

**Pattern de Uso**
- Podemos enviar um formulário que nos leve de volta à mesma página.
- Padrão "página base". No caso de um calendário, queremos ficar na mesma página mas adicionar, remover ou atualizar eventos. Ou em um aplicativo de anotações de livros, onde se tem uma "lista de leitura" para adicionar, remover ou atualizar livros, e também para atualizar a quantidade e a barra de progresso.
- https://www.colby.so/posts/turbo-8-morphing-refreshes-on-rails

**Cuidados**
- Gerenciar formulários e elementos dinâmicos da página pode ser desafiador e pode exigir código adicional para gerenciar efetivamente o estado.
- As atualizações de morphing são intensivas em recursos, pois requerem a renderização de toda a página e depois a realização de um diff, o que pode ser exigente em dispositivos menos potentes.
- As atualizações transmitidas envolvem uma viagem adicional ao servidor, que pode ser mais lenta do que as atualizações diretas do Turbo Stream, que entregam o conteúdo em uma etapa.

!!! Comece usando o simples com o modo padrão do Turbo *replace* e *reset*. Se seu contexto realmente pedir use o Morph.

**Observações**

Tive esse problema na live: [Separate turbo meta tag generation from provides by bradgessler · Pull Request #542 · hotwired/turbo-rails (github.com)](https://github.com/hotwired/turbo-rails/pull/542/files)

## Page Refresh x Morphing

Page Refresh é o novo paradigma de obter automaticamente uma atualização parcial da página (sem a necessidade de definir Turbo Frames ou Streams).

Morphing é a técnica usada na biblioteca Idiomorph para fazer o "merge" do DOM [bigskysoftware/idiomorph: A DOM-merging algorithm (github.com)](https://github.com/bigskysoftware/idiomorph)).
