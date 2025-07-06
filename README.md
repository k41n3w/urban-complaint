Aluno: Caio Ramos Email: cwfr88@gmail.com

# 📋 Urban Complaint

Este é um projeto desenvolvido como parte do curso de Arquitetura de Aplicações Web com Rails. O objetivo é aplicar os conceitos aprendidos em aula para criar uma aplicação web funcional, bem estruturada e com boas práticas de desenvolvimento.

O **Urban Complaint** é um sistema para registro e acompanhamento de reclamações urbanas, permitindo que moradores reportem problemas na cidade e que órgãos públicos gerenciem essas reclamações de forma eficiente e transparente.

## 🛠️ Tecnologias Utilizadas

- **Ruby** 3.2.2
- **Rails** 7.1.3
- **PostgreSQL** - Banco de dados relacional
- **Docker** e **Docker Compose** - Containerização
- **Devise** - Sistema de autenticação
- **Devise JWT** - Autenticação via tokens JWT
- **CanCanCan** - Sistema de autorização
- **Active Admin** - Interface administrativa
- **Kaminari** - Paginação
- **Simple Form** - Formulários simplificados
- **CarrierWave** - Upload de arquivos
- **Geocoder** - Integração com mapas e geolocalização
- **RSpec** - Testes automatizados
- **FactoryBot** - Factories para testes
- **Faker** - Dados fictícios para testes

## 🚀 Como rodar o projeto localmente

### Pré-requisitos
- Docker
- Docker Compose

### Configuração com Dev Container

1. Clone o repositório:
```bash
git clone https://github.com/k41n3w/urban-complaint.git
cd urban-complaint
```

2. Inicie o ambiente de desenvolvimento com Docker:
```bash
docker-compose up -d
```

3. Execute as migrações do banco de dados:
```bash
docker-compose exec web rails db:create db:migrate
```

4. Carregue os dados iniciais (opcional):
```bash
docker-compose exec web rails db:seed
```

5. Acesse a aplicação:
```
http://localhost:3000
```

### Comandos úteis

```bash
# Acessar o console Rails
docker-compose exec web rails console

# Executar testes
docker-compose exec web rspec

# Instalar novas gems
docker-compose exec web bundle install

# Reiniciar a aplicação
docker-compose restart web
```

## ⚙️ Funcionalidades Implementadas

### 🔐 Sistema de Autenticação e Autorização
- Cadastro e login de usuários com Devise
- Autenticação JWT para APIs
- Sistema de permissões com CanCanCan
- Diferenciação entre usuários comuns e administradores
- Proteção contra tentativas de login maliciosas (lockable)

### 📝 Gestão de Reclamações
- Registro de reclamações com título, descrição, categoria e localização
- Validação robusta de dados com prevenção de duplicatas
- Sistema de status (Aberta, Em Análise, Em Atendimento, Resolvida, Rejeitada)
- Filtros avançados por status, categoria, período e usuário
- Paginação para melhor performance

### 🗺️ Integração com Mapas
- Seleção de localização via mapas interativos
- Geocodificação de endereços
- Validação de coordenadas geográficas

### 💬 Sistema de Comentários
- Comentários em reclamações
- Moderação de comentários
- Associação de comentários aos usuários

### 📧 Notificações
- Notificações por e-mail sobre mudanças de status
- Processamento assíncrono de e-mails
- Templates personalizados para diferentes tipos de notificação

### 🔧 Interface Administrativa
- Painel administrativo completo com Active Admin
- Gestão de usuários, reclamações e comentários
- Filtros e buscas avançadas


## 📚 Conceitos Aplicados

### ✅ 1. Sistema de Autenticação Robusto com Devise

**Implementação:**
```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :lockable
end
```

**Justificativa:** O Devise foi escolhido por ser a solução mais madura e segura para autenticação em Rails. A implementação inclui funcionalidades avançadas como bloqueio de conta após tentativas falhadas (:lockable), recuperação de senha e validações robustas. Além disso, foi integrado com JWT para suporte a APIs, demonstrando conhecimento de autenticação stateless para aplicações modernas.

### ✅ 2. Sistema de Autorização com CanCanCan

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb
def authorize_complaint
  unless current_user.admin? || current_user == @complaint.user
    redirect_to complaints_path, alert: 'Você não tem permissão para realizar esta ação.'
  end
end
```

**Justificativa:** O CanCanCan foi implementado para controlar o acesso às funcionalidades baseado em papéis de usuário. Isso garante que apenas administradores possam gerenciar todas as reclamações, enquanto usuários comuns só podem editar suas próprias reclamações. Esta abordagem segue o princípio de menor privilégio e melhora significativamente a segurança da aplicação.

### ✅ 3. Interface Administrativa com Active Admin

**Implementação:**
```ruby
# Gemfile
gem "activeadmin"

# Configuração completa em app/admin/
# - dashboard.rb
# - users.rb  
# - complaints.rb
# - admin_users.rb
```

**Justificativa:** O Active Admin foi escolhido para criar rapidamente uma interface administrativa robusta e profissional. Ele permite que administradores gerenciem todos os aspectos da aplicação sem necessidade de desenvolvimento de interfaces customizadas, acelerando o desenvolvimento e fornecendo funcionalidades avançadas como filtros, buscas e exportação de dados.

### ✅ 4. Background Jobs com Action Mailer

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb
if old_status != @complaint.status
  NotificationMailer.status_update(@complaint).deliver_later
end
```

**Justificativa:** O uso de `deliver_later` demonstra a implementação de processamento assíncrono para evitar que o envio de e-mails bloqueie a resposta HTTP. Isso melhora significativamente a experiência do usuário, pois as ações são executadas rapidamente enquanto os e-mails são processados em background. Esta é uma prática essencial para aplicações em produção.

### ✅ 5. Validações Robustas e Active Record Scopes

**Implementação:**
```ruby
# app/models/complaint.rb
validates :title, uniqueness: { 
  scope: [:latitude, :longitude, :user_id], 
  message: 'Já existe uma reclamação similar neste local' 
}

scope :by_status, ->(status) { where(status: status) if status.present? }
scope :by_category, ->(category) { where(category: category) if category.present? }
```

**Justificativa:** As validações implementadas garantem a integridade dos dados, incluindo uma validação complexa que previne reclamações duplicadas no mesmo local pelo mesmo usuário. Os scopes organizam as queries de forma reutilizável e legível, seguindo o princípio DRY e facilitando a manutenção do código.

### ✅ 6. Uso Estratégico de Enums

**Implementação:**
```ruby
# app/models/complaint.rb
enum status: {
  open: 'Aberta',
  analyzing: 'Em Análise', 
  in_progress: 'Em Atendimento',
  resolved: 'Resolvida',
  rejected: 'Rejeitada'
}
```

**Justificativa:** Os enums foram utilizados para garantir type safety e otimização no banco de dados. Eles previnem valores inválidos, facilitam queries e tornam o código mais legível. A escolha de usar strings como valores permite melhor debugging e compatibilidade com sistemas externos.

### ✅ 7. Containerização com Docker

**Implementação:**
```dockerfile
# Dockerfile e docker-compose.yml configurados
# Ambiente isolado e reproduzível
# Separação de serviços (web, database)
```

**Justificativa:** A containerização com Docker garante que a aplicação rode de forma consistente em qualquer ambiente, eliminando o problema "funciona na minha máquina". Isso facilita o desenvolvimento em equipe, deployment e manutenção da aplicação. O uso do docker-compose permite orquestrar múltiplos serviços de forma simples.

### ✅ 8. Testes Automatizados com RSpec

**Implementação:**
```ruby
# Gemfile
group :development, :test do
  gem "rspec-rails"
  gem "factory_bot_rails" 
  gem "faker"
end
```

**Justificativa:** A configuração completa de testes com RSpec, FactoryBot e Faker demonstra compromisso com qualidade de código e desenvolvimento sustentável. Os testes garantem que as funcionalidades continuem funcionando após mudanças, facilitam refatorações e servem como documentação viva do comportamento esperado da aplicação.

## 🎯 Próximas Melhorias

Para tornar a aplicação ainda mais robusta e seguir padrões avançados de arquitetura, as próximas implementações incluiriam:

- **Service Objects** para extrair lógica de negócio dos controllers
- **Repository Pattern** para encapsular queries complexas
- **Interactors** para organizar fluxos de negócio
- **Rails Engines** para modularização
- **Ferramentas de qualidade** (RuboCop, Reek, Brakeman)

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais como parte do curso de Arquitetura de Aplicações Web com Rails.
