🏦 FIAP Bank — Checkpoint 6
Plataforma de Concessão de Crédito e Segurança
Instituição: FIAP
Curso: Engenharia de Software
Disciplina: Domain Driven Design (Java)
Professor: Eduardo dos Santos Ramos
Turma: 2ESPG
Aluno: Eduardo Antonio Delarissia
RM: 563468  
📌 Sobre o projeto
O FIAP Bank é uma plataforma acadêmica para simular a solicitação, consulta e análise de propostas de crédito bancário. No Checkpoint 6, o foco é reforçar a segurança da aplicação com autenticação baseada em JWT, autorização por perfis (RBAC) e proteção declarativa dos endpoints com Spring Security.

A Plataforma de Crédito atua como um OAuth2 Resource Server e valida os tokens enviados pelos clientes. O repositório também inclui um front-end Angular, uma Mesa de Operações em Spring Boot/Thymeleaf e um serviço simplificado de Core Bancário.
Atenção: esta é uma implementação didática criada com base no enunciado do CP6. Não foi feita uma migração comprovada do código original do Checkpoint 5. As integrações com Bureau de Crédito, SCR/Bacen e sistema de notificações não estão implementadas. O projeto não deve ser usado em um banco real.

🎯 Objetivos
- Proteger endpoints REST com Spring Security.
- Emitir e validar tokens JWT assinados com HS256.
- Manter a autenticação da Plataforma de Crédito stateless, sem sessão HTTP.
- Aplicar permissões diferentes para CLIENTE, OPERADOR e ADMIN.
- Utilizar @EnableMethodSecurity e @PreAuthorize.
- Propagar o cabeçalho Authorization: Bearer <token> via interceptadores Angular e OpenFeign.
- Preservar a separação entre domínio, aplicação e apresentação.
🛠️ Tecnologias utilizadas
Tecnologia	Aplicação
Java 21	Microsserviços
Spring Boot 4.1.2	APIs REST e aplicações web
Spring Security / OAuth2 Resource Server	Autenticação e autorização
Nimbus JOSE + JWT	Assinatura e validação de JWT
Maven	Build e dependências
Spring Cloud OpenFeign	Comunicação da Mesa com a API
Thymeleaf	Interface da Mesa de Operações
Angular 20 + TypeScript	Internet Banking Mobile (protótipo web)
JUnit	Testes automatizados
Postman	Testes de API
Git / GitHub	Versionamento


📁 Estrutura do monorepo
fiap-bank-cp6/
├── fiap-bank-plataforma-credito/
│   ├── domain/                 # Entidades e regras de domínio
│   ├── application/            # Serviço de concessão de crédito e teste JUnit
│   ├── presentation/           # API REST, login, JWT e Spring Security
│   └── pom.xml
├── fiap-bank-core-bancario/    # Serviço simplificado de contas/saldo
├── fiap-bank-mesa-operacoes-credito/
│   ├── src/main/java/         # FeignClient, interceptor e controllers
│   └── src/main/resources/templates/mesa.html
├── fiap-bank-internet-banking-mobile/
│   ├── src/main.ts            # Interface Angular
│   └── src/app/auth.interceptor.ts
├── FIAP_Bank_CP6.postman_collection.json
├── TESTES_POSTMAN.md
└── README.md
Responsabilidades dos serviços
- Plataforma de Crédito (8080): login demonstrativo, emissão e verificação de JWT, criação e acompanhamento de propostas, atualização de status por operadores.
- Core Bancário (8081): serviço ilustrativo para operações bancárias básicas, sem integração transacional completa.
- Mesa de Operações (8082): interface Thymeleaf e encaminhamento de solicitações para a API via OpenFeign.
- Internet Banking (4200): interface Angular para login, solicitação e consulta de crédito.
🔐 Segurança e perfis
A API usa SecurityFilterChain, SessionCreationPolicy.STATELESS, autenticação Bearer e validação criptográfica do token. Os dados de autorização são extraídos das claims roles e scope.
Perfil	Criar proposta	Consultar propostas	Alterar status
CLIENTE	✅	✅ (próprias)	❌
OPERADOR	❌	✅	✅
ADMIN	❌	✅	✅


As claims emitidas incluem iss, sub, iat, exp, cpf, conta, roles e scope. A validade configurada é de 7200 segundos (2 horas).

Observações de segurança: o CSRF está desabilitado para a API demonstrativa stateless; o H2 Console possui rota permitida, mas não está instalado/habilitado neste projeto. O código configura frameOptions().sameOrigin() em vez de desativar a proteção globalmente.
▶️ Como executar
Pré-requisitos
- JDK 21
- Maven 3.9+
- Node.js 20.19+ e npm
- Git (opcional)
- IntelliJ IDEA ou VS Code (opcional)
Extraia o .zip e abra quatro terminais na pasta fiap-bank-cp6.

Terminal 1 — Plataforma de Crédito
cd fiap-bank-plataforma-credito
mvn clean install
cd presentation
mvn spring-boot:run
A API ficará disponível em http://localhost:8080.

Terminal 2 — Core Bancário
cd fiap-bank-core-bancario
mvn spring-boot:run
O serviço utilizará http://localhost:8081.

Terminal 3 — Mesa de Operações
cd fiap-bank-mesa-operacoes-credito
mvn spring-boot:run
Abra http://localhost:8082.

Terminal 4 — Internet Banking Angular
cd fiap-bank-internet-banking-mobile
npm install
npm start
Abra http://localhost:4200.
Os comandos pressupõem que cada terminal foi aberto na pasta raiz fiap-bank-cp6. Em caso de incompatibilidades locais de dependências, verifique as versões dos pré-requisitos e os logs da compilação.

🧪 Testando a API
1. Obter token de cliente
POST http://localhost:8080/api/auth/login
{
  "usuario": "cliente",
  "senha": "cliente123"
}
A resposta contém access_token e token_type. Copie o token.
2. Solicitar crédito
POST http://localhost:8080/api/v1/solicitacoes

Headers:
Authorization: Bearer SEU_TOKEN
Content-Type: application/json
Body:
{
  "valor": 4500
}
Resultado esperado: 201 Created.
3. Consultar propostas
GET http://localhost:8080/api/v1/solicitacoes

Envie o token do cliente ou do operador no cabeçalho Authorization.
4. Login do operador
POST http://localhost:8080/api/auth/login
{
  "usuario": "operador",
  "senha": "operador123"
}
5. Aprovar uma proposta
PUT http://localhost:8080/api/v1/solicitacoes/1/status

Utilize o token do operador e o body:
{
  "status": "APROVADA"
}
O ID 1 é ilustrativo: utilize o ID retornado na criação da proposta.
6. Testes de segurança
Cenário	Resultado esperado
Rota protegida sem Bearer Token	401 Unauthorized
Token adulterado ou expirado	401 Unauthorized
Cliente tentando aprovar proposta	403 Forbidden
Cliente criando proposta com token válido	201 Created
Operador atualizando proposta existente	200 OK


Importe a coleção FIAP_Bank_CP6.postman_collection.json para facilitar os testes. Consulte também TESTES_POSTMAN.md.
Testes unitários
Na pasta fiap-bank-plataforma-credito, execute:
mvn test
O projeto inclui um teste JUnit da camada de aplicação. A existência do teste não substitui a execução dos cenários integrados.
⚙️ Configuração e limitações
As configurações da API ficam em:
fiap-bank-plataforma-credito/presentation/src/main/resources/application.yml
Variáveis de ambiente aceitas:
Variável	Finalidade
JWT_SECRET	Segredo HMAC de assinatura dos tokens
DEMO_CLIENTE_PASSWORD	Senha do login de cliente demonstrativo
DEMO_OPERADOR_PASSWORD	Senha do login de operador demonstrativo


Importante: os valores-padrão de segredo e senhas servem somente para uso acadêmico local. Não publique segredos reais, não utilize credenciais fixas em produção e use HTTPS em sistemas reais. Um ambiente bancário exigiria um Authorization Server adequado, autenticação robusta, armazenamento persistente, auditoria, tratamento de dados pessoais e outras proteções.

As propostas são armazenadas em memória: ao reiniciar o serviço, os registros são perdidos. Isso não cria sessões de autenticação no servidor, mas também não representa persistência bancária de produção. O armazenamento do token no navegador é uma simplificação didática.
📦 Entrega do Checkpoint 6
1. Publicar o monorepo no GitHub, incluindo os fontes dos quatro serviços e os arquivos src/main/java.
2. Conferir se o README.md contém o nome completo e RM de todos os integrantes, caso o trabalho seja em grupo.
3. Validar compilação e execução, registrar testes e evidências no Postman.
4. Compactar o projeto em .zip (não .rar).
5. Entregar o ZIP e o link do repositório pela tarefa do Microsoft Teams indicada pelo professor.
Nome sugerido do arquivo:
RM563468_Eduardo_Delarissia_CP6.zip
Nota: o enunciado do Checkpoint 6 menciona a tarefa do “Checkpoint 4” na seção de entrega. Confirme o local correto com o professor.

👨‍🎓 Autor
Eduardo Antonio Delarissia
RM: 563468
FIAP — Engenharia de Software
Disciplina: Domain Driven Design (Java)
