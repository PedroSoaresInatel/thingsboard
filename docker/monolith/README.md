# ThingsBoard em um único servidor (Docker Compose)

Sobe o ThingsBoard como monolito (fila in-memory) com um PostgreSQL dedicado, usando a imagem oficial
`thingsboard/tb-node`. Serve para rodar na sua máquina ou em qualquer servidor com Docker (VPS, máquina
virtual, servidor próprio).

Não compila o código deste repositório: usa a imagem publicada da versão definida em `TB_VERSION`.

## Requisitos

- Docker com o plugin Compose.
- Mínimo: 2 núcleos, 4 GB de RAM, 20 GB de SSD. Para uso contínuo com dezenas de dispositivos, 8 GB de RAM
  dão folga. Em repouso o ThingsBoard ocupa cerca de 1,4 GB e o PostgreSQL cerca de 100 MB.

## Instalação

Todos os comandos são executados dentro desta pasta.

1. Crie o arquivo de configuração e defina a senha do banco:

   ```bash
   cp .env.example .env
   # edite .env e preencha POSTGRES_PASSWORD
   ```

2. Crie o banco de dados. O final da saída mostra o **Cluster Id** da instalação; anote.

   ```bash
   docker compose run --rm -e INSTALL_TB=true thingsboard
   ```

3. Suba os serviços:

   ```bash
   docker compose up -d
   ```

4. Abra `http://localhost:8080` (ou a porta definida em `TB_HTTP_PORT`). O primeiro acesso leva cerca de
   40 segundos para responder. O assistente de configuração pede a licença, cria o administrador do sistema
   e oferece carregar os dados de demonstração.

## Licença

A partir da versão 4.4 o ThingsBoard exige uma destas duas opções:

| Opção | Como | Limites |
|---|---|---|
| Chave gratuita | Crie uma conta em https://license.thingsboard.io, informe o Cluster Id e cole a chave no assistente (ou em `TB_LICENSE_SECRET`). | Uso não comercial: até 1.000 dispositivos. Uso comercial: até 100 dispositivos em um único servidor. A marca ThingsBoard precisa continuar visível. |
| Sem chave | `NON_PRODUCTION_USE=true` no `.env`. | Só para testes: marca d'água, e a instância se desliga ao atingir 30 dias de uptime acumulado. |

A chave é emitida para um Cluster Id, e o Cluster Id é gerado na instalação do banco. Uma instalação nova
em outro servidor gera outro Cluster Id e precisa de outra chave. Para consultar o Cluster Id depois:

```bash
docker compose exec postgres psql -U postgres -d thingsboard -Atc 'select cluster_id from tb_cluster'
```

## Acesso pela rede

Por padrão as portas só atendem na própria máquina (`TB_BIND_ADDRESS=127.0.0.1`). Para aceitar conexões de
outras máquinas, use `TB_BIND_ADDRESS=0.0.0.0` e rode `docker compose up -d` de novo.

Em um servidor exposto à internet, a interface (porta HTTP) e o MQTT (1883) trafegam sem criptografia.
Coloque um proxy reverso com HTTPS na frente da porta HTTP e libere no firewall só o que for necessário.

## Operação

```bash
docker compose logs -f thingsboard   # acompanhar os logs
docker compose stop                  # parar
docker compose start                 # voltar a subir
docker compose down                  # remover os containers (os dados ficam nos volumes)
docker compose down -v               # remover tudo, INCLUSIVE os dados
```

## Por que não a Vercel

O servidor do ThingsBoard não roda na Vercel. Segundo a documentação da própria Vercel:

- Containers lá são funções que só recebem HTTP em uma porta. O ThingsBoard também precisa de MQTT, que é
  TCP puro na porta 1883.
- Uma função sem tráfego por 5 minutos é desligada, e cada execução dura no máximo 300 s (Hobby) ou 800 s
  (Pro). O ThingsBoard precisa ficar ligado o tempo todo: mantém fila, agendamentos e sessões de
  dispositivos em memória e leva cerca de 30 segundos para iniciar.
- Nada gravado em disco ou em memória sobrevive entre instâncias.

A própria Vercel indica outra plataforma (Render, Railway, ECS, Kubernetes) para serviços sempre ligados
com conexões persistentes. Este `docker-compose.yml` funciona em qualquer uma que rode Docker Compose ou em
uma VPS comum.

Fontes: [Container images](https://vercel.com/docs/functions/container-images),
[Limites das funções](https://vercel.com/docs/functions/limitations),
[Docker monolith with workers](https://vercel.com/kb/guide/docker-monolith-workers-vercel),
[Docker on Vercel vs Render](https://vercel.com/kb/guide/docker-on-vercel-vs-render).
