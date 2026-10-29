# 🌤️ Lagasse Clima Bot

Chatbot no Telegram desenvolvido com n8n para consultar a temperatura atual de cidades brasileiras usando a API da OpenWeather.

O usuário envia uma mensagem com a cidade e o estado, e o bot responde com a temperatura em graus Celsius.

## 💬 Exemplo de uso

Envie ao bot:

```text
São Paulo,SP,BR
```

Exemplo de resposta:

```text
🌤️ A temperatura em São Paulo é de 25°C.
```

A temperatura apresentada varia conforme o momento da consulta.

## 🛠️ Tecnologias

- n8n para automação do workflow.
- Telegram Bot API para recebimento e envio de mensagens.
- OpenWeather API para consulta do clima.
- JavaScript para normalização e formatação dos dados.

## 📁 Arquivo do workflow

O workflow deve estar disponível no repositório com o nome:

```text
workflow-telegram-chatbot.json
```

Para exportar, abra o workflow no n8n, acesse o menu de opções e selecione **Download**. Salve o arquivo com o nome acima.

## 📋 Pré-requisitos

- Instância do n8n acessível por uma URL pública HTTPS.
- Bot criado no Telegram pelo [BotFather](https://t.me/BotFather).
- Token de acesso do bot.
- Chave ativa da [OpenWeather](https://home.openweathermap.org/api_keys), com acesso ao serviço Current Weather.

## 🔐 Variáveis e credenciais

| Nome | Finalidade | Onde configurar |
| --- | --- | --- |
| `OPENWEATHER_API_KEY` | Chave da API da OpenWeather. | Ambiente do processo ou container do n8n. |
| `TELEGRAM_BOT_TOKEN` | Token fornecido pelo BotFather. | Campo **Access Token** de uma credencial **Telegram API** no n8n. |
| `WEBHOOK_URL` | Endereço público HTTPS do n8n. | Ambiente do n8n. |
| `N8N_BLOCK_ENV_ACCESS_IN_NODE` | Permite o acesso às variáveis de ambiente pelas expressões. | Ambiente do n8n, com valor `false`. |

`TELEGRAM_BOT_TOKEN` representa o segredo utilizado na credencial do Telegram. Definir essa variável no servidor não cria nem preenche automaticamente a credencial no n8n.

### Configuração da OpenWeather no Docker Compose

Acrescente à seção `environment` do serviço n8n:

```yaml
environment:
  - OPENWEATHER_API_KEY=${OPENWEATHER_API_KEY}
  - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
  - WEBHOOK_URL=https://${COMPOSE_PROJECT_NAME}.${TRAEFIK_HOST}/
```

No arquivo `.env` privado ou nas variáveis do projeto no painel da hospedagem, configure:

```env
OPENWEATHER_API_KEY=sua_chave_openweather
```

As variáveis `COMPOSE_PROJECT_NAME` e `TRAEFIK_HOST` devem corresponder à configuração do domínio e do Traefik da instalação.

Aplique a configuração com um redeploy pelo painel ou execute, na pasta do Compose:

```bash
docker compose up -d
```

No nó **Consultar clima**, o parâmetro `appid` utiliza a expressão:

```javascript
{{ $env.OPENWEATHER_API_KEY }}
```

## 📥 Importação do workflow

1. Acesse o editor do n8n.
2. Crie um workflow.
3. Abra o menu de opções e selecione **Import from File**.
4. Escolha o arquivo `workflow-telegram-chatbot.json`.
5. Configure as credenciais do Telegram conforme as instruções abaixo.
6. Salve o workflow.

## 🤖 Configuração do Telegram

1. Abra o nó **Receber mensagem**.
2. Em **Credential to connect with**, selecione **Create new credential**.
3. Crie uma credencial do tipo **Telegram API**.
4. No campo **Access Token**, cole o token do Lagasse Clima Bot fornecido pelo BotFather.
5. Salve e verifique a conexão.
6. Selecione a mesma credencial nos nós **Enviar temperatura** e **Enviar erro**.

O Telegram precisa alcançar o n8n pela internet. O valor de `WEBHOOK_URL` deve apontar para o endereço público HTTPS da instância.

## ⚙️ Funcionamento

1. **Receber mensagem:** inicia o workflow com uma mensagem recebida pelo Telegram.
2. **Normalizar entrada:** captura o texto na variável `queue`, remove acentos, converte para minúsculas e ajusta os espaços.
3. **Entrada valida:** verifica o formato da cidade, a UF e o país.
4. **Consultar clima:** envia a consulta à OpenWeather.
5. **Resposta valida:** verifica o status HTTP e os campos retornados.
6. **Formatar temperatura:** arredonda a temperatura e prepara a mensagem.
7. **Enviar temperatura:** responde ao usuário pelo Telegram.
8. **Enviar erro:** orienta o usuário quando a entrada ou a consulta não pode ser processada.

### Parâmetros da OpenWeather

Endpoint:

```text
https://api.openweathermap.org/data/2.5/weather
```

| Parâmetro | Valor |
| --- | --- |
| `q` | Conteúdo da variável `queue`. |
| `units` | `metric`, para temperatura em graus Celsius. |
| `lang` | `pt_br`. |
| `appid` | Valor de `OPENWEATHER_API_KEY` por expressão. |

A variável interna se chama `queue`, conforme o desafio. O parâmetro aceito pela OpenWeather é `q`.

Espaços entre palavras de cidades compostas são preservados. Espaços extras e ao redor das vírgulas são removidos.

### Tratamento de erros

Quando a entrada é inválida, a cidade não é encontrada ou a resposta da API não contém os dados esperados, o bot envia:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

Falhas de autenticação, limite de requisições e indisponibilidade da API também podem resultar nessa mensagem. Nesses casos, confira a configuração da chave e o status da consulta no n8n.

## ▶️ Como executar

### Teste manual

1. No n8n, clique em **Execute workflow**.
2. Abra o Lagasse Clima Bot no Telegram.
3. Envie:

   ```text
   Curitiba,PR,BR
   ```

4. Aguarde a resposta com a temperatura atual.

Se a mensagem `/start` consumir a execução de teste, execute o workflow novamente antes de enviar a cidade.

### Funcionamento contínuo

Após concluir os testes, ative/publique o workflow.

O Telegram mantém apenas um webhook por bot. Alternar entre testes e produção pode substituir o webhook registrado. Ao encerrar os testes, ative/publique novamente o workflow de produção.

## 🧪 Roteiro de testes

Execute as consultas abaixo e confira os resultados:

| Mensagem | Resultado esperado |
| --- | --- |
| `São Paulo,SP,BR` | Temperatura atual de São Paulo. |
| `Belo Horizonte,MG,BR` | Temperatura atual de Belo Horizonte. |
| `Curitiba,PR,BR` | Temperatura atual de Curitiba. |
| `CidadeInventadaXYZ,SP,BR` | Mensagem de cidade não encontrada. |
| `São Paulo,XX,BR` | Mensagem de orientação sobre o formato. |

Os testes reais dependem de credenciais válidas, disponibilidade da API e configuração do webhook HTTPS.

## 📍 Limitação da consulta por cidade

A consulta por nome segue o formato solicitado no desafio. Entretanto, o suporte ao código de estado na busca integrada da OpenWeather é limitado, e a UF brasileira não garante a distinção entre municípios homônimos.

Para uma evolução do projeto, recomenda-se geocodificar a cidade com validação do estado e consultar o clima por latitude e longitude.

## 🛡️ Segurança

Tokens e chaves reais devem permanecer nas credenciais do n8n ou no ambiente privado do servidor.

Antes de publicar o JSON exportado, confira se ele não contém:

- Token do Telegram ou chave da OpenWeather.
- Segredos escritos diretamente em parâmetros, URLs ou código.
- Dados de teste fixados em `pinData`.
- Informações privadas desnecessárias ou referências a credenciais da instalação.

Caso a exportação contenha referências em `credentials`, remova essas referências da cópia de entrega e configure as credenciais novamente após a importação.

Inclua no `.gitignore`:

```gitignore
.env
.env.*
!.env.example
.n8n/
```

Revise também o histórico do Git antes de tornar o repositório público. Se uma chave real tiver sido publicada, revogue-a e gere outra.

## ✅ Checklist de entrega

- [ ] Workflow exportado como `workflow-telegram-chatbot.json`.
- [ ] README com instruções de importação e configuração.
- [ ] Credenciais do Telegram configuradas nos três nós.
- [ ] Chave da OpenWeather configurada no ambiente.
- [ ] Webhook público HTTPS funcionando.
- [ ] Consultas reais realizadas com pelo menos três cidades.
- [ ] Teste realizado com uma cidade inexistente.
- [ ] JSON revisado e sem tokens ou chaves embutidos.
- [ ] Repositório público para avaliação.

## 📚 Referências

- [Documentação do n8n](https://docs.n8n.io/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [OpenWeather Current Weather](https://openweathermap.org/current)