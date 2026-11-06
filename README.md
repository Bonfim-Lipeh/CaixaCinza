<div align="center">

# FoodExpress — Testes Funcionais Caixa Cinza da API de Autenticação (Supabase + Postman)

**UniFacens** · Curso Análise e Desenvolvimento de Sistemas · **AS406TSN3** · UX/UI e Testes de Software · **Aula 11**

</div>

---

## 1. Introdução

O **FoodExpress** é um e-commerce de comida cujo protótipo de baixa fidelidade (React + Vite) possui as telas de cadastro e login, mas ainda não conta com conexão nativa com banco de dados. Os requisitos funcionais cobertos nesta etapa são:

| ID | Requisito | Operação | Resultado Esperado |
| :---: | :--- | :--- | :--- |
| **1** | Registrar-se | Cliente cria uma conta | Conta criada com sucesso |
| **2** | Fazer Login | Cliente acessa o sistema | Login realizado com sucesso |

Para validar a autenticação antes de integrá-la ao front-end, utilizou-se o **Supabase Auth** como back-end e o **Postman Desktop** como cliente de testes. Como a atividade é de **caixa cinza**, o testador conhece parte da estrutura interna (endpoint, headers, formato do body, códigos HTTP e o papel do API Gateway), mas não o código-fonte proprietário do servidor de autenticação.

---

## 2. Estrutura do Repositório

```text
/README.md                      → Documentação técnica completa (este arquivo)
/postman/                       → Coleção e environment importáveis no Postman
    FoodExpress_API_Tests.postman_collection.json
    FoodExpress_Supabase.postman_environment.json
/prints/                        → Evidências (telas simuladas do Supabase e do Postman)
/planilha/                      → Planilha de testes (.xlsx com 4 abas) e Registro_dos_Testes.csv


sequenceDiagram
    autonumber
    actor T as Tester (Postman)
    participant G as API Gateway (Supabase)
    participant A as Auth / GoTrue
    T->>G: POST /auth/v1/token?grant_type=password<br/>headers: apikey, Authorization, Content-Type<br/>body: {email, password}
    alt apikey ausente ou inválida
        G-->>T: 401 Unauthorized (No API key found / Invalid API key)
    else apikey válida
        G->>A: Encaminha a requisição
        alt credenciais corretas
            A-->>T: 200 OK + access_token (JWT), refresh_token, user
        else credenciais incorretas ou inexistentes
            A-->>T: 400 invalid_credentials
        else campos vazios
            A-->>T: 400 validation_failed
        end
    end

    4. Objetivo da Atividade
Compreender e aplicar testes funcionais de caixa cinza em uma API de autenticação, utilizando ferramentas de mercado. Os objetivos específicos incluem:

Configurar um ambiente de autenticação no Supabase;

Criar um usuário válido para testes;

Compreender o funcionamento de requisições HTTP (método, endpoint, headers, body, status);

Configurar e executar testes no Postman (Workspace, Environment, Collection, scripts de teste);

Validar status HTTP, corpo JSON, mensagens de erro e access token;

Documentar tecnicamente todo o processo neste README, acompanhado da planilha de registro.

Nota: Não há implementação de interface gráfica nesta atividade.

5. Configuração do Supabase
5.1. Como o projeto foi criado
Acesso a Supabase → Start your project → cadastro com conta GitHub (plano Free, limite de 2 projetos).

Criação de um novo projeto (New project) com os dados abaixo (opções Security Options e Advanced Configuration mantidas no padrão: Data API + Connection String, public schema, Postgres):

Campo,Valor
Organization,org-aluno-facens (Free)
Project name,foodexpress-auth-tests
Database password,Gerada pelo botão Generate a password (guardada fora do repositório)
Region,South America (São Paulo) — sa-east-1

Aguardado o provisionamento; o painel exibiu o status Healthy e o Project ref xkqzmvtbnhdwlyrcjepa (fictício).


5.2. Tipo de autenticação
Em Authentication → Sign In / Providers:

Provedor Email: Habilitado (Enabled para autenticação por e-mail e senha);

Allow new users to sign up: Ligado;

Confirm email: Desmarcado (a confirmação é opcional e o e-mail de teste não possui caixa de entrada real, permitindo que o usuário criado possa autenticar imediatamente);

Provedores de terceiros, login anônimo e manual linking: Desligados.

5.3. Criação do usuário de teste
Em Authentication → Users → Add user → Create new user, com a opção Auto Confirm User marcada:
Campo,Valor
E-mail,teste.cinza@foodexpress.test
Senha,Teste@12345 (apenas para ambiente de teste)
UID,3f8a6c1e-7b24-4d9a-a1c5-92e0d6b7f401
Provider,Email

5.4. Credenciais da API
Obtidas em Project Settings → API:
Item,Valor (Fictício)
API URL,https://xkqzmvtbnhdwlyrcjepa.supabase.co
API KEY (anon / public),eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIs…SIMULADO_NAO_E_UMA_CHAVE_REAL
A chave service_role não é utilizada, pois ela ignora o Row Level Security e jamais deve ser exposta no cliente.


5.5. Identificação do agente no ambiente
Item,Valor
Agente executor,"IA (Claude), em nome do aluno"
Data da execução,06/11/2026
Conta / organização,org-aluno-facens (plano Free)
Projeto,foodexpress-auth-tests

Objetivo da configuração: Dispor de um provedor de identidade real (Postgres + GoTrue) para validar, antes de integrar o front-end, que a API autentica credenciais válidas, rejeita as inválidas e protege o acesso via cabeçalho apikey.


6. Configuração do Postman
6.1. Workspace
O Postman Desktop foi instalado a partir do site oficial do Postman e conectado à conta do aluno. Em Workspaces → Create Workspace, criou-se o workspace FoodExpress Supabase (Personal), onde foram alocados o Environment e a coleção FoodExpress API Tests.

6.2. Environment e variáveis
Em Environments, criou-se o ambiente FoodExpress Supabase, selecionado como ativo no canto superior direito. As variáveis evitam repetir dados sensíveis em cada requisição e permitem migrar de projeto Supabase alterando apenas um local.


Variável,Descrição,Valor
base_url,URL da API do Supabase,https://xkqzmvtbnhdwlyrcjepa.supabase.co
api_key,API KEY (anon public),(Tipo secret)
email,Usuário criado,teste.cinza@foodexpress.test
password,Senha criada,(Tipo secret)
email_inexistente,Extra: e-mail não cadastrado (CT03),nao.existe@foodexpress.test
password_errada,Extra: senha incorreta (CT02),(Tipo secret)
access_token,Extra: preenchida pelo script do CT01,(Vazia até o login)

6.3. Como o Postman é usado nos testes
Cada cenário corresponde a uma requisição da coleção, contendo scripts de teste na aba Scripts → Post-response que validam automaticamente o status HTTP, o corpo JSON e as mensagens.

A coleção é executada via Runner (Run FoodExpress API Tests), gerando um relatório detalhado de aprovados/reprovados. Para reproduzir, importe os dois arquivos disponíveis na pasta /postman e preencha as variáveis api_key e password com os valores do seu projeto.

7. Configuração das Requisições
7.1. Parâmetros principais
Item,Valor
Endpoint,/auth/v1/token?grant_type=password
URL completa,{{base_url}}/auth/v1/token?grant_type=password
Método HTTP,POST
Finalidade,Autenticar o usuário por e-mail e senha e obter o access token (JWT) da sessão

7.2. Headers
Key,Value,Função
apikey,{{api_key}},Identifica o projeto no API Gateway (obrigatório)
Authorization,Bearer {{api_key}},Autorização do gateway (com a chave anon)
Content-Type,application/json,Indica que o corpo da requisição é JSON

POST {{base_url}}/auth/v1/token?grant_type=password HTTP/1.1
apikey: {{api_key}}
Authorization: Bearer {{api_key}}
Content-Type: application/json

7.3. Body (raw · JSON)
JSON
{
  "email": "{{email}}",
  "password": "{{password}}"
}
Com as variáveis resolvidas no CT01:

JSON
{
  "email": "teste.cinza@foodexpress.test",
  "password": "Teste@12345"
}
7.4. Script de teste do CT01 (aba Scripts)
JavaScript
pm.test("CT01 - Status code é 200", function () { 
  pm.response.to.have.status(200); 
});

pm.test("CT01 - Resposta contém access_token", function () {
  var j = pm.response.json();
  pm.expect(j).to.have.property("access_token");
  pm.expect(j.token_type).to.eql("bearer");
  pm.expect(j.user.email).to.eql(pm.environment.get("email"));
  pm.environment.set("access_token", j.access_token);
});

pm.test("CT01 - Tempo de resposta < 2000 ms", function () { 
  pm.expect(pm.response.responseTime).to.be.below(2000); 
});
Os demais cenários foram criados duplicando a requisição do CT01, alterando o título, o corpo da requisição e os scripts correspondentes (verifique a coleção completa na pasta /postman).

8. Execução dos Testes
Execução simulada realizada em 06/11/2026, às 14:58, no ambiente FoodExpress Supabase (1 iteração, 7 requisições / 23 asserções).

7.3. Body (raw · JSON)
JSON
{
  "email": "{{email}}",
  "password": "{{password}}"
}
Com as variáveis resolvidas no CT01:

JSON
{
  "email": "teste.cinza@foodexpress.test",
  "password": "Teste@12345"
}
7.4. Script de teste do CT01 (aba Scripts)
JavaScript
pm.test("CT01 - Status code é 200", function () { 
  pm.response.to.have.status(200); 
});

pm.test("CT01 - Resposta contém access_token", function () {
  var j = pm.response.json();
  pm.expect(j).to.have.property("access_token");
  pm.expect(j.token_type).to.eql("bearer");
  pm.expect(j.user.email).to.eql(pm.environment.get("email"));
  pm.environment.set("access_token", j.access_token);
});

pm.test("CT01 - Tempo de resposta < 2000 ms", function () { 
  pm.expect(pm.response.responseTime).to.be.below(2000); 
});
Os demais cenários foram criados duplicando a requisição do CT01, alterando o título, o corpo da requisição e os scripts correspondentes (verifique a coleção completa na pasta /postman).

8. Execução dos Testes
Execução simulada realizada em 06/11/2026, às 14:58, no ambiente FoodExpress Supabase (1 iteração, 7 requisições / 23 asserções).

Cenário	Entrada	Resultado Esperado	Resultado Obtido	Status
CT01 - Login válido	teste.cinza@foodexpress.test / Teste@12345	Status 200 + access token	200 OK, access_token JWT, token_type: bearer, expires_in: 3600	✅ Aprovado
CT02 - Senha incorreta	E-mail válido / SenhaErrada#999	Erro de autenticação	400, invalid_credentials	✅ Aprovado
CT03 - Usuário inexistente	nao.existe@foodexpress.test / Teste@12345	Acesso negado	400, invalid_credentials	✅ Aprovado
CT04 - Campos vazios	"" / ""	Erro de validação	400, validation_failed	✅ Aprovado
CT05 - Credenciais inválidas	isto-nao-e-um-email / 123	Mensagem de erro	400, "Invalid login credentials"	✅ Aprovado
CT06 (Comp.) - Sem apikey	Sem headers de chave	401 do gateway	401, No API key found in request	✅ Aprovado
CT07 (Comp.) - apikey inválida	chave-invalida-123	401 do gateway	401, Invalid API key	✅ Aprovado
CT08 (Revisão) - Front-end aceita dados falsos	qualquer@x.com / x em Login.jsx	Acesso bloqueado	Redireciona para /catalog sem consultar o Supabase	❌ Reprovado

8.2. Evidências — Testes Válidos
CT01 — Login válido (200 OK, 412 ms):

JSON
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOi…SIMULADO_ASSINATURA",
  "token_type": "bearer",
  "expires_in": 3600,
  "expires_at": 1793003600,
  "refresh_token": "k3p9x2vq7mzl",
  "user": {
    "id": "3f8a6c1e-7b24-4d9a-a1c5-92e0d6b7f401",
    "aud": "authenticated",
    "role": "authenticated",
    "email": "teste.cinza@foodexpress.test",
    "email_confirmed_at": "2026-11-06T17:52:10Z"
  }
}
Validações aplicadas: status 200 ✔ · access_token presente ✔ · token_type = bearer ✔ · e-mail do user confere ✔ · tempo < 2000 ms ✔ · token gravado em {{access_token}} ✔.

8.3. Evidências — Testes Inválidos
CT02 — Senha incorreta (400 Bad Request, 287 ms) e CT03 — Usuário inexistente (400, 301 ms) retornam o mesmo corpo:

JSON
{
  "code": 400,
  "error_code": "invalid_credentials",
  "msg": "Invalid login credentials"
}
CT04 — Campos vazios (400, 118 ms):

JSON
{
  "code": 400,
  "error_code": "validation_failed",
  "msg": "Email and password are required"
}
CT05 — Credenciais inválidas (400, 142 ms): Mesmo corpo do CT02.

CT06 — Sem apikey (401, 96 ms) e CT07 — apikey inválida (401, 104 ms) respondidos diretamente pelo API Gateway:

JSON
{ "message": "No API key found in request", "hint": "No `apikey` request header or url param was found." }
JSON
{ "message": "Invalid API key", "hint": "Double check your Supabase `anon` or `service_role` API key." }
8.4. Resumo do Runner
8.5. Erros Encontrados
CT08 (Defeito real no código do projeto): Em src/pages/Login.jsx, a função handleLogin executa apenas if (email && password) navigate("/catalog"). Qualquer par preenchido libera o acesso sem efetuar a chamada ao Supabase. Em src/pages/Signup.jsx, handleSignup faz console.log("Novo usuário:", { email, password }) (expondo a senha no console) e navega para /catalog sem criar a conta de fato. Em resumo: a API de autenticação opera corretamente, mas o requisito de login/registro no front-end não foi implementado.

Observação sobre o CT03: O Supabase devolve a mesma resposta estruturada para "usuário inexistente" e "senha incorreta". Isso não se trata de um defeito, mas de uma medida de segurança voltada a evitar a enumeração de usuários cadastrados. O resultado esperado de "acesso negado" foi atendido plenamente.

9. Registro dos Testes

<img width="889" height="244" alt="Captura de tela 2026-11-06 150855" src="https://github.com/user-attachments/assets/2e4c4df9-03e8-4005-9a2a-fa5722ee3a42" />

<img width="1280" height="560" alt="13_postman_runner_resumo" src="https://github.com/user-attachments/assets/4ea4030c-d7d1-49bf-89b2-326300ac4485" />

10. Resultados ObtidosIndicadorResultadoCenários obrigatórios (CT01–CT05)5/5 aprovadosCenários complementares (CT06–CT07)2/2 aprovadosVerificação do front-end (CT08)0/1 — reprovadoTotal geral7 aprovados · 1 reprovado (8 testes)Asserções do Postman23/23 aprovadasTempo médio de resposta (API)209 ms (mín. 96 ms · máx. 412 ms)Rastreabilidade com os requisitos (Aba: Aceitação):1 — Registrar-se: Parcial (o usuário existe no Supabase por ter sido criado via painel; o cadastro efetuado pela tela não aciona a API).2 — Fazer Login: Parcial (a API autentica corretamente; a interface de login não realiza a chamada para a API).11. ConclusãoOs testes foram executados corretamente? Sim, dentro do escopo proposto. Os cinco cenários obrigatórios e os dois complementares foram devidamente cobertos com suas respectivas entradas, resultados esperados e obtidos. Ressalva importante: a execução e as evidências são simuladas conforme o enunciado; os resultados refletem o comportamento documentado do Supabase Auth, mas mensagens textuais exatas (como no CT04) podem variar dependendo da versão do GoTrue e devem ser confirmadas em uma execução real.A autenticação funcionou? Sim. A API retorna 200 acompanhada de um JWT válido para credenciais corretas, rejeita com 400 invalid_credentials as credenciais incorretas ou inexistentes, aponta erro de validação para campos vazios e o gateway bloqueia com 401 requisições desprovidas de uma apikey válida.11.1. Dificuldades MapeadasCompreender a distinção entre apikey / Authorization (nível de gateway) e o access_token (sessão de usuário);Definir a configuração do Confirm email: mantê-lo ligado exige uma caixa de e-mail real, enquanto desligá-lo simplifica o ambiente de testes acadêmicos;O Supabase unifica a resposta para "senha incorreta" e "usuário inexistente", exigindo interpretação correta do resultado esperado no CT03;Garantir que chaves sensíveis fiquem fora do versionamento público por meio de variáveis de ambiente (secret).11.2. Falhas IdentificadasCT08: O front-end (Login.jsx) libera o acesso a qualquer combinação de e-mail e senha preenchidos; Signup.jsx falha em criar a conta e imprime a senha diretamente no console do navegador.11.3. Melhorias SugeridasIntegrar o front-end utilizando o pacote @supabase/supabase-js através dos métodos signInWithPassword e signUp, tratando os erros adequadamente e exibindo mensagens visuais para o usuário;Proteger rotas privadas (/catalog, /cart, /checkout...) exigindo a validação prévia de uma sessão ativa;Remover o console.log que expõe credenciais de senha;Habilitar a confirmação de e-mail (Confirm email) e políticas de senha robustas para ambientes de produção;Adicionar testes de borda: e-mails com espaçamento ou letras maiúsculas, senhas excessivamente longas, tentativas de injeção (' OR 1=1 --), bloqueio por excesso de tentativas (rate limit), além de testes de expiração e renovação de token (grant_type=refresh_token);Automatizar a execução da coleção no pipeline de CI com o Newman:newman run postman/FoodExpress_API_Tests.postman_collection.json -e postman/FoodExpress_Supabase.postman_environment.jsonConsumir o access_token obtido no CT01 em requisições autenticadas subsequentes (ex.: GET /auth/v1/user).11.4. Importância dos Testes Caixa Cinza em APIsEles combinam a perspectiva comportamental de caixa preta com o conhecimento parcial da arquitetura (endpoints, headers, contratos de dados e códigos de erro). Essa abordagem permite testar a camada de serviço de forma isolada e direta, sem depender da interface gráfica, antecipando problemas de contrato, validação e segurança. Além disso, facilita o isolamento de defeitos: neste caso, a API estava íntegra e o erro residia na integração do front-end (CT08), situação que só pôde ser diagnosticada com clareza devido à separação dos níveis de teste.
---

### Próximos passos recomendados
Deseja ajustar algum detalhe específico neste documento? Posso ajudar com:
* **Ajustar termos técnicos** ou adaptar a formatação para os padrões específicos do seu curso.
* **Gerar o conteúdo complementar** para a planilha de testes ou scripts do Postman.
