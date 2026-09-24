![Status da Build](https://github.com/kong/docker-kong/actions/workflows/test.yml/badge.svg)

# Sobre este Repositório

Este é o repositório Git da
[imagem oficial](https://docs.docker.com/docker-hub/official_repos/) do Docker para o
[kong](https://registry.hub.docker.com/_/kong/).
Consulte [a página do Docker Hub](https://registry.hub.docker.com/_/kong/)
para o readme completo sobre como usar esta imagem Docker e para informações
sobre contribuições e problemas.

O readme completo é gerado em [docker-library/docs](https://github.com/docker-library/docs),
especificamente em [docker-library/docs/kong](https://github.com/docker-library/docs/tree/master/kong).

Viu uma mudança mesclada aqui que ainda não aparece no Docker Hub?
Verifique [o arquivo de manifesto "library/kong" no repositório docker-library/official-images
](https://github.com/docker-library/official-images/blob/master/library/kong),
especialmente [PRs com a label "library/kong" nesse
repositório](https://github.com/docker-library/official-images/labels/library%2Fkong). Para mais informações sobre o processo de imagens oficiais, consulte o [readme do docker-library/official-images](https://github.com/docker-library/official-images/blob/master/README.md).

# Para desenvolvedores Kong

## Publicando uma atualização de patch release do Kong (x.y.Z)

Se a atualização não exigir alterações nos Dockerfiles além de
apontar para o código mais recente do Kong, o processo pode ser semi-automatizado da seguinte forma:

1. Faça o checkout deste repositório.

2. Execute `./update.sh x.y.z`

   Isso criará uma branch de release, modificará os arquivos relevantes automaticamente,
   dará a você a chance de revisar as alterações e pressionar "y", então
   fará o push da branch e abrirá um navegador com o PR
   para este repositório.

3. Revisão por pares, execute o CI e faça o merge do PR enviado.

4. Execute `./submit.sh -p x.y.z`

   Após o merge do PR interno, este script fará o mesmo
   para o repositório [official-images](https://github.com/docker-library/official-images).
   Ele clonará o [fork do Kong](https://github.com/kong/official-images),
   criará uma branch, modificará os arquivos relevantes automaticamente,
   dará a você a chance de revisar as alterações e pressionar "y", então
   fará o push da branch e abrirá um navegador com o PR
   para o repositório docker-library.

## Publicando uma atualização de minor release do Kong (x.Y.0)

Ainda não semi-automatizado. Note que minor releases têm maior probabilidade de exigir
alterações mais extensas nos Dockerfiles.
