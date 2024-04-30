[X] - Application Visit
- Toda vez que eu clico em algum link eu faço uma visita.
- Uma visita me faz uma request.
- Quando existe a resposta. Turbo Drive renderiza  o HTML e completa a Visita
- Turbo tenta renderizar um preview de uma página que está no cache imediatamente quando a visita inicia.
- Essa preview do cache é tão rápido que você pensa que é a nova página e não é.
- Comportamento de Scroll: Se existir um anchor o Turbo Drive vai tentar scrollar pro elemento. Se não existe. Ele usa o comportamento padrão de scrollar pro topo da página.
- Você tem duas actions. A default é a advance e adiciona a página no histórico do navegador. E você tem a replace, pode descartar a entrada atual do histórico do browser e fazer o replace.
- Turbo.visit("/edit", { action: "replace" })
- Também pode ser usado para Frames

[X] - Restoration Visit
- Feito quando uso os botões do browser para voltar ou avançar
- Se possível o Turbo Drive vai tentar fazer uma cópia da página do cache sem fazer uma request.
- Ele salva automaticamente a posição do scroll de cada página antes de sair e retorna para a posição salva em uma Restoration Visit.
- Não faça uma visita com Turbo.visit e uma action restore.

[X] - Cache
- Dois propósitos: mostrar páginas sem fazer uma requisição durante Restoration Visits (uso dos botões avançar e voltar do browser) e "aumentar" a performance mostrando previews temporários durante uma Application Visit.
- Em uma Application Visit ele restaura a página do cache se existir e mostra enquanto faz um request simultaneamente para uma nova página. Isso dá a ilusão de "instant" page load.
- Um ponto importante o Turbo Drive copia a página usando cloneNode(true). Isso significa que event listeners, animações e dados associados são descartados.
- Evento: turbo:before-cache
- Pode usar esse evento para resetar formulários, componentes de UI animações etc. Com o objetivo de não guardar o status deles no cache. Quando for carregado do cache eles poderão ser vistos novamente.
- Alguns elementos como flash messages tem uma característica de serem temporários. Se você colocar esses elementos na "página cacheada" pode apresentar inconsistências na UI.
- Para resolver esse problema você pode usar o data-turbo-temporary. O Turbo Drive automaticamente remove esses elementos da página antes de serem cacheados.
- Programaticamente podemos detectar se é um "preview", isto é uma página que veio do cache pelo atributo no "data-turbo-preview" no HTML.
- Escolha de cache: Usa a meta tag no header <meta name="turbo-cache-control">
- Isso pode ser feito por página
- <meta name="turbo-cache-control" content="no-preview"> A página que estiver cacheada é usada apenas para Restoration Visit.
- <meta name="turbo-cache-control" content="no-cache"> Quando você usa a opção no-cache não vai existir nenhum preview, nenhuma contéudo na cache, essas páginas sempre vão ter de ser requisitas para o servidor; incluindo as Restoration Visits.
- Para fazer o disable de caching por completo todas as páginas devem ter um "no-cache"
- Pode fazer opting out do caching pelo lado do client
- Turbo.cache.exemptPageFromCache() -> no-cache
- Turbo.cache.exemptPageFromPreview() -> no-preview
- Turbo.cache.resetCacheControl()
