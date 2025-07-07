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

### ✅ 1. Evitando Fat Controllers (Módulo 3 - Anti-patterns)

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb - Controller enxuto
def create
  @complaint = current_user.complaints.new(complaint_params)
  @complaint.status = 'open'
  
  if @complaint.save
    redirect_to @complaint, notice: 'Reclamação registrada com sucesso.'
  else
    render :new, status: :unprocessable_entity
  end
end
```

**Justificativa:** O controller mantém apenas a lógica de fluxo HTTP, delegando validações e regras de negócio para o modelo. Isso evita o anti-pattern de Fat Controller ensinado no Módulo 3, mantendo os controllers focados apenas em coordenar requisições e respostas.

### ✅ 2. Evitando Fat Models com Scopes (Módulo 3 - Anti-patterns)

**Implementação:**
```ruby
# app/models/complaint.rb - Organização com scopes
scope :by_status, ->(status) { where(status: status) if status.present? }
scope :by_category, ->(category) { where(category: category) if category.present? }
scope :by_period, ->(start_date, end_date) {
  where('created_at >= ? AND created_at <= ?', start_date, end_date) if start_date.present? && end_date.present?
}
```

**Justificativa:** Os scopes organizam queries complexas de forma reutilizável, evitando que o modelo se torne um Fat Model com métodos excessivos. Esta abordagem segue as boas práticas do Módulo 3 para manter modelos organizados e focados.

### ✅ 3. Separação de Responsabilidades com Enums (Módulo 3)

**Implementação:**
```ruby
# app/models/complaint.rb - Type safety com enums
enum status: {
  open: 'Aberta',
  analyzing: 'Em Análise',
  in_progress: 'Em Atendimento',
  resolved: 'Resolvida',
  rejected: 'Rejeitada'
}

enum category: {
  infrastructure: 'Infraestrutura',
  lighting: 'Iluminação',
  sanitation: 'Saneamento',
  security: 'Segurança',
  other: 'Outros'
}
```

**Justificativa:** Os enums garantem type safety e encapsulam a lógica de estados, seguindo o princípio de separação de responsabilidades ensinado no Módulo 3. Eles previnem valores inválidos e centralizam a definição de estados possíveis.

### ✅ 4. Background Jobs para Performance (Módulo 2 - Observer Pattern)

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb - Processamento assíncrono
if old_status != @complaint.status
  NotificationMailer.status_update(@complaint).deliver_later
end
```

**Justificativa:** O uso de `deliver_later` implementa processamento assíncrono, evitando que operações lentas (envio de email) bloqueiem a resposta HTTP. Isso segue o padrão Observer ensinado no Módulo 2, onde mudanças de estado disparam ações secundárias.

### ✅ 5. Validações Robustas e Integridade de Dados (Módulo 3)

**Implementação:**
```ruby
# app/models/complaint.rb - Validações complexas
validates :title, uniqueness: { 
  scope: [:latitude, :longitude, :user_id], 
  message: 'Já existe uma reclamação similar neste local' 
}
validates :latitude, :longitude, presence: true, numericality: true
```

**Justificativa:** As validações garantem integridade dos dados no nível do modelo, seguindo as boas práticas de separação de responsabilidades do Módulo 3. A validação de unicidade composta previne duplicatas de forma elegante.

### ✅ 6. Autorização Baseada em Papéis (Módulo 3 - Separação de Responsabilidades)

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb - Controle de acesso
def authorize_complaint
  unless current_user.admin? || current_user == @complaint.user
    redirect_to complaints_path, alert: 'Você não tem permissão para realizar esta ação.'
  end
end
```

**Justificativa:** A autorização é separada em método específico, seguindo o princípio de responsabilidade única do Módulo 3. Isso garante que apenas usuários autorizados possam modificar reclamações, mantendo a segurança da aplicação.

### ✅ 7. Otimização de Performance - Evitando N+1 Queries (Módulo 3 - Performance)

**Implementação:**
```ruby
# app/controllers/complaints_controller.rb - Preload de associações
def show
  @comment = Comment.new
  @comments = @complaint.comments.includes(:user).page(params[:page]).per(5)
end
```

**Justificativa:** O uso de `includes(:user)` carrega as associações de usuário de forma eficiente em uma única query, evitando o problema N+1 que ocorreria se cada comentário fizesse uma query separada para buscar seu usuário. Esta otimização melhora significativamente a performance da aplicação, especialmente em páginas com muitos comentários.

## 📄 Licença

Este projeto foi desenvolvido para fins educacionais como parte do curso de Arquitetura de Aplicações Web com Rails.
