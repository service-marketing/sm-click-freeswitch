<img width="1477" height="554" alt="Banner freeswitch" src="COLE_AQUI_A_URL_DO_BANNER" />

[![Feature Branch to Develop](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/1-commits.yml/badge.svg)](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/1-commits.yml)
[![Build and Push to ECR - Develop](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/2-develop.yml/badge.svg)](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/2-develop.yml)
[![Build & Deploy - Production](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/4-production.yml/badge.svg)](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/4-production.yml)
[![Hotfix - to master and Develop](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/5-hotfix.yml/badge.svg)](https://github.com/service-marketing/sm-click-freeswitch/actions/workflows/5-hotfix.yml)

Fork do [FreeSWITCH](https://github.com/signalwire/freeswitch) usado pelo SM Click para gerar a **imagem base própria do FreeSWITCH**, compilada do código-fonte sem depender de nenhum serviço, token ou imagem da SignalWire. A imagem é publicada no ECR (`sm-click-freeswitch`) de dev e de produção e serve de `FROM` para o [`sm-click-sip-server`](https://github.com/service-marketing/sm-click-sip-server), que adiciona a nossa configuração e o roteador das chamadas.

> Este repositório não tem configuração do SM Click nem roda em lugar nenhum: ele só builda e publica a imagem. Perfis, ramais, dialplan e tronco ficam no `sm-click-sip-server`.

## Índice

- [Ambiente de desenvolvimento](#ambiente-de-desenvolvimento)
- [Testes](#testes)
- [Deploy](#deploy)
- [Contribuição](#contribuição)
- [O que tem aqui dentro](#o-que-tem-aqui-dentro)
- [Contato](#contato)

## Ambiente de desenvolvimento

```bash
DOCKER_BUILDKIT=1 docker build -f docker/sm-click/Dockerfile -t sm-click-freeswitch:local .

# Sobe com a config vanilla e confere se está UP
docker run -d --name fs-local sm-click-freeswitch:local
docker exec fs-local fs_cli -x status

# Nenhuma lib ausente (fora de módulos não usados)
docker run --rm --entrypoint sh sm-click-freeswitch:local -c \
  'for f in /usr/local/freeswitch/bin/freeswitch /usr/local/freeswitch/lib/*.so* /usr/local/freeswitch/mod/*.so; do ldd "$f" | grep "not found" | sed "s|^|$f: |"; done'
```

Build leva de 30 a 60 min num runner de 4 cores. No Windows, clone com `core.autocrlf=false` ou confie no `.gitattributes`: arquivos de build com CRLF quebram o `bootstrap.sh`.

## Testes

Este repositório não tem testes próprios nem gate no CI além do build. A imagem é validada pelo smoke test do `sm-click-sip-server` (`docker/smoke_test.sh`), que sobe o FreeSWITCH com a nossa config e confere módulos, perfis, WSS e healthcheck. Antes de atualizar `FREESWITCH_IMAGE_TAG` lá, rode o smoke test com a tag nova.

## Deploy

Não há deploy de serviço: o CI builda `docker/sm-click/Dockerfile` e publica a imagem no ECR. A branch de produção deste repositório é `master` (herdada do original), não `main`.

| Workflow | Gatilho | O que faz |
|---|---|---|
| `1-commits.yml` | push em `feature/*` | abre PR para `develop` |
| `2-develop.yml` | merge na `develop` | versão alpha pelo Conventional Commit (`standard-version`), build, push no ECR de dev, cria `release/X.Y.Z` e a PR |
| `3-release.yml` | push em `release/*` | abre a PR `release/X.Y.Z -> master` |
| `4-production.yml` | push na `master` | build e push no ECR de produção com a versão da release |
| `5-hotfix.yml` | push em `hotfix/*` | abre PRs para `master` e `develop` |

A tag da imagem é a versão do `.github/package.json` (`X.Y.Z-alpha.N` em dev; em produção, a versão que está na `master`). Versão e changelog ficam em `.github/package.json` e `.github/CHANGELOG.md`.

**Secrets do repositório:** `AWS_REGION`, `ECR_REPOSITORY`, `AWS_ACCOUNT_ID_DEV`/`AWS_ACCESS_KEY_ID_DEV`/`AWS_SECRET_ACCESS_KEY_DEV`, `AWS_ACCOUNT_ID_PROD`/`AWS_ACCESS_KEY_ID_PROD`/`AWS_SECRET_ACCESS_KEY_PROD`, `GIT_SECRET_TOKEN` (PRs automáticas e push da versão).

O repositório ECR `sm-click-freeswitch` precisa existir nas duas contas; este repositório não cria infraestrutura.

## Contribuição

1. Crie uma branch a partir de `develop` (`git checkout -b feature/minha-feature`)
2. Commit suas alterações (`git commit -m 'fix: minha correção'` — mensagem semântica, define o bump de versão automático)
3. Suba a branch (`git push origin feature/minha-feature`)
4. A PR para `develop` é aberta automaticamente pelo CI
5. Ao mesclar em `develop`, uma tag alpha nova é publicada no ECR de dev — rode o smoke test do `sm-click-sip-server` com ela e **atualize a variável `FREESWITCH_IMAGE_TAG`** de lá para usá-la
6. Para trazer uma versão nova do [FreeSWITCH original](https://github.com/signalwire/freeswitch), faça o merge dele numa `feature/*` e rode o build, o `ldd` e o smoke test antes de abrir a PR

## O que tem aqui dentro

Quase tudo é o código do FreeSWITCH original (documentação em [developer.signalwire.com/freeswitch](https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/), licença MPL 1.1, ver [`LICENSE`](LICENSE)). O que é nosso:

### Estrutura

```
docker/sm-click/
└── Dockerfile          # Build completo em dois estágios (debian:bookworm), base de todas as tags publicadas

.github/
├── workflows/
│   ├── 1-commits.yml     # push em feature/* -> PR para develop
│   ├── 2-develop.yml     # merge na develop -> versão alpha, build, push no ECR de dev, branch release/* e PR
│   ├── 3-release.yml     # push em release/* -> PR para master
│   ├── 4-production.yml  # push na master -> build e push no ECR de produção
│   └── 5-hotfix.yml      # push em hotfix/* -> PRs para master e develop
├── package.json        # Versão da imagem (standard-version)
└── CHANGELOG.md

.gitattributes          # Força LF em scripts e arquivos de build (o bootstrap quebra com CRLF vindo do Windows)
```

O `docker/master/docker-entrypoint.sh` e o `freeswitch.limits.conf` do original são reaproveitados no estágio final.

### Stack

| | |
|---|---|
| Base | `debian:bookworm` nos dois estágios (sem imagem da SignalWire) |
| FreeSWITCH | código deste fork (`1.11.x`), instalado em `/usr/local/freeswitch` |
| Dependências compiladas | sofia-sip `v1.13.19` (tag pública) e spandsp (branch `master` do GitHub da FreeSWITCH) |
| Módulos | `modules.conf.most` menos os que dependem de fornecedor ou de lib indisponível no Debian |
| Publicação | ECR `sm-click-freeswitch`, contas de dev e produção, tag = versão do `.github/package.json` |

### Como funciona

1. **Estágio `builder`**: instala o toolchain e as `-dev` do Debian, compila spandsp e sofia-sip a partir das fontes públicas e depois o FreeSWITCH deste repositório.
   - Remove o `-Werror` do `configure.ac` (warnings de libs novas do Debian quebravam o build).
   - Parte do `modules.conf.most` e comenta `mod_signalwire`, `mod_verto`, `mod_av`, `mod_bv`, `mod_silk`, `mod_siren`, `mod_cv`, `mod_java`, `mod_managed`, `mod_ilbc`, `mod_flite`, `mod_erlang_event`, `mod_mongo`, `mod_python3` e `mod_pocketsphinx`.
   - Instala sons (`sounds-install`) e música de espera (`moh-install`).
2. **Estágio `runtime`**: Debian limpo só com as bibliotecas de execução, o usuário `freeswitch` (uid/gid 499) e o que saiu do builder (`/usr/local/freeswitch`, `libsofia-sip-ua*`, `libspandsp*`).
   - A config vanilla fica em `/usr/local/freeswitch/etc/freeswitch` e os certificados em `/usr/local/freeswitch/certs`; o `sm-click-sip-server` passa `-conf`/`-certs` para os seus caminhos.
   - `HEALTHCHECK` com `fs_cli -x status` na senha padrão do ESL; o `sm-click-sip-server` sobrescreve com a senha dele.
3. **Publicação**: o CI envia a imagem para o ECR e o `sm-click-sip-server` escolhe qual tag usar pela variável `FREESWITCH_IMAGE_TAG`.

### Cuidados

- **Biblioteca de runtime faltando não quebra o build**, só a execução: o binário não carrega (`libfreetype.so.6`...) ou um módulo falha no boot. Ao ativar um módulo ou mudar de base, rode o `ldd` de [Ambiente de desenvolvimento](#ambiente-de-desenvolvimento) antes de publicar.
- O spandsp vem da branch `master`, então duas builds em dias diferentes podem não ser idênticas.

## Contato

- Nome: Alex Oliveira Rocha
- Email: alex.oliveira-rocha@servicemarketing-tecnologia.com
- Cargo: Diretor de Tecnologia
