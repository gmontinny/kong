# Kong com Docker Compose

O template oficial do Docker Compose para o Kong Gateway.

> **Nota**
> O arquivo Docker Compose do Kong usa o schema compose v3.9; portanto,
> requer o Docker Engine versão 20.10.0+.

## O que é o Kong?

Kong ou Kong API Gateway é um API Gateway cloud-native, agnóstico de plataforma e escalável,
reconhecido pelo seu alto desempenho e extensibilidade via plugins.

- A documentação oficial do Kong pode ser encontrada em [docs.konghq.com][kong-docs-url].
- Você pode encontrar a distribuição oficial do Docker para o Kong no [Docker Hub][kong-docker-url].

## Como usar este arquivo Compose

Este `docker-compose.yml` sobe o Kong com Postgres por padrão. Basta executar:

```shell
$ docker compose up -d
```

Isso iniciará  4 contêineres na ordem correta:

1. `db` — Postgres 13
2. `kong-migrations` — cria as tabelas no banco na primeira execução
3. `kong-migrations-up` — atualiza o schema em novas versões
4. `kong` — o gateway em si

```shell
$ docker compose ps
NAME                      IMAGE          STATUS
compose-db-1              postgres:13    Up (healthy)
compose-kong-migrations-1 kong:latest    Exited (0)
compose-kong-1            kong:latest    Up (healthy)
```

## Portas expostas

| Porta | Descrição |
|-------|-----------|
| `8000` | Proxy HTTP — onde suas rotas respondem |
| `8443` | Proxy HTTPS |
| `8001` | Admin API — gerenciamento do Kong via REST |
| `8444` | Admin API HTTPS |
| `8002` | Kong Manager (interface web) |

## Configurando serviços e rotas

Ao acessar `http://localhost:8000/` sem nenhuma rota configurada, o Kong retornará:

```
{ "message": "no Route matched with those values" }
```

Isso é esperado. A porta `8000` é o proxy e só responde quando há serviços e rotas configurados.
Com Postgres ativo, você pode configurar pelo **Kong Manager** em `http://localhost:8002`,
pela **Admin API** na porta `8001`, ou via `kong.yaml` na inicialização.

Consulte o [Tutorial FastAPI](./TUTORIAL_FASTAPI.md) para exemplos completos.

## Tutoriais

- [Gerenciando uma API FastAPI com o Kong](./TUTORIAL_FASTAPI.md) — proxy, rate limiting, autenticação por API Key, JWT, CORS e logs.

## Problemas

Se você tiver algum problema ou dúvida sobre esta imagem, entre em contato conosco
através de uma [issue no GitHub][github-new-issue].

## Contribuindo

Você está convidado a contribuir com novos recursos, correções ou atualizações, grandes ou pequenas;
ficamos sempre felizes em receber pull requests e fazemos o possível para processá-los
o mais rápido possível.

Antes de começar a codificar, recomendamos discutir seus planos através de uma
[issue no GitHub][github-new-issue], especialmente para contribuições mais ambiciosas. Isso
dá a outros contribuidores a chance de apontá-lo na direção certa, fornecer
feedback sobre seu design e ajudá-lo a descobrir se outra pessoa está trabalhando na
mesma coisa.

[kong-docs-url]: https://docs.konghq.com/
[kong-docs-dbless]: https://docs.konghq.com/gateway/latest/production/deployment-topologies/db-less-and-declarative-config/#main
[kong-docs-dbless-file]: https://docs.konghq.com/gateway/latest/production/deployment-topologies/db-less-and-declarative-config/#declarative-configuration-format
[kong-docker-url]: https://hub.docker.com/_/kong
[github-new-issue]: https://github.com/Kong/docker-kong/issues/new
