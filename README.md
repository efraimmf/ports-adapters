# Ports & Adapters Backend

## Visão Geral

Este projeto implementa uma **Arquitetura Hexagonal (Ports and Adapters)** para uma aplicação backend moderna construída com **NestJS**, **TypeScript**, **Prisma** e **PostgreSQL**. A arquitetura garante separação clara de responsabilidades, testabilidade e manutenibilidade do código.

## Arquitetura Hexagonal

### Princípios Fundamentais

### Princípios Fundamentais

A arquitetura hexagonal organiza a aplicação em dois grandes domínios:

**Núcleo (Core) e Portas (Ports):**
Essa camada representa o coração da aplicação, onde estão localizadas as regras de negócio e a lógica que define o comportamento do sistema. As "portas" são interfaces que expõem as funcionalidades do domínio e definem como os adaptadores externos podem interagir com o núcleo. Essa estrutura garante que o core seja independente de detalhes de infraestrutura, como frameworks, bancos de dados ou interfaces de usuário.

**Adaptadores (Adapters):**
São implementações concretas que "plugam" nas portas definidas pelo core, permitindo que o sistema se comunique com o mundo externo. Esses adaptadores podem ser substituídos com facilidade, sem impactar a lógica de negócio. Exemplos comuns incluem:
* Persistência de dados (como, Postgres ou MySQL)
* Interfaces de apresentação (como, HTTP ou SOAP)
* Serviços externos (como, APIs ou serviços de mensageria)

Essa separação de responsabilidades contribui para um sistema mais desacoplado, modular e com baixo custo de manutenção.

### Organização do Projeto

Seguindo os princípios acima, este repositório foi estruturado em três camadas principais:

**Core (Domínio):**
Contém toda a lógica de negócio da aplicação. É onde estão definidos os casos de uso, entidades, serviços de domínio e as interfaces (portas) que abstraem a comunicação com o exterior.

**Infrastructure (Infraestrutura):**
Implementa as interfaces responsáveis pela persistência de dados definidas no core.

**Presentation (Apresentação):**
Implementa as interfaces responsáveis pela apresentação e obtenção dos dados do usuário definidas no core.

Essa estrutura permite que o core da aplicação seja reutilizado em diferentes contextos, com diferentes tecnologias, sem a necessidade de reescrita da lógica de negócio. Ao passo que também possibilita que toda a aplicação seja testada em partes isoladas, garantindo a separação clara de responsabilidades, a testabilidade e a manutenibilidade do código.

### Estrutura de Diretórios

```
src/
├── core/                   # Camada de domínio (núcleo da aplicação)
│   ├── domain/            # Entidades de domínio
│   ├── ports/             # Contratos e abstrações
│   │   ├── repositories/  # Interfaces de repositórios
│   │   └── services/      # Interfaces dos serviços de aplicação
│   ├── services/          # Serviços de domínio e casos de uso
│   └── exceptions/        # Exceções específicas do domínio
├── infra/                 # Camada de infraestrutura
│   ├── prisma/           # Configuração do Prisma
│   └── repositories/     # Implementações dos repositórios
├── modules/             # Módulos do NestJS
├── presentation/         # Camada de apresentação
│   ├── controllers/      # Controllers HTTP
│   ├── dtos/            # Data Transfer Objects
│   ├── filters/         # Exception filters
│   └── guards/          # Guards de autenticação/autorização
├── app.module.ts        # Módulo principal da aplicação
└── main.ts              # Ponto de entrada da aplicação
```

## Schema do Banco de Dados

O projeto utiliza **Prisma** como ORM e **PostgreSQL** como banco de dados. O schema define um sistema de reservas de recursos:

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int       @id @default(autoincrement())
  name      String
  email     String    @unique
  password  String
  bookings  Booking[]
}

model Booking {
  id          Int       @id @default(autoincrement())
  userId      Int
  resourceId  Int
  date        DateTime

  user        User      @relation(fields: [userId], references: [id])
  resource    Resource  @relation(fields: [resourceId], references: [id])
}

model Resource {
  id        Int       @id @default(autoincrement())
  name      String
  location  String
  bookings  Booking[]
}
```

### Relacionamentos
- **User** possui múltiplas **Bookings** (1:N)
- **Resource** possui múltiplas **Bookings** (1:N)  
- **Booking** pertence a um **User** e um **Resource** (N:1)

## 🔧 Componentes da Arquitetura

### 1. Core (Domínio)
- **Entidades**: User, Booking, Resource
- **Ports**: Interfaces para repositórios e serviços  
- **Services**: Lógica de negócio (UserService, BookingService)
- **Exceptions**: Exceções específicas do domínio

### 2. Infrastructure (Infraestrutura)  
- **Prisma**: Configuração e cliente do banco
- **Repositories**: Implementações dos contratos de dados

### 3. Presentation (Apresentação)
- **Controllers**: Endpoints REST 
- **DTOs**: Validação de entrada e saída
- **Filters/Guards**: Tratamento de erros e autenticação

### 4. Modules (Configuração NestJS)
- **Modules**: Configuração de injeção de dependências e organização de features

## Fluxo de Dados

### Exemplo: Criação de Usuário
```
POST /users → UserController → IUserService → UserService → [cria User do domínio] → UserRepository → PrismaUserRepository → Prisma → PostgreSQL
```

### Exemplo: Listagem de Usuários
```
GET /users → UserController → IUserService → UserService → UserRepository → PrismaUserRepository → Prisma → PostgreSQL
```

### Tratamento de Erros
```
DomainException → GlobalExceptionFilter → HTTP Response (padronizado)
```

## Benefícios da Arquitetura

### 1. **Separação de Responsabilidades**
- Domínio isolado de detalhes técnicos
- Lógica de negócio independente de frameworks
- Interfaces claras entre camadas

### 2. **Testabilidade**
- Mock repositories para testes unitários
- Injeção de dependências facilita testes
- Domínio testável sem infraestrutura

### 3. **Manutenibilidade**
- Código organizado por responsabilidade
- Mudanças isoladas por camada
- Documentação clara de cada componente

### 4. **Flexibilidade**
- Fácil troca de implementações (Prisma → TypeORM)
- Suporte a diferentes bancos de dados
- Configuração por ambiente

### 5. **Escalabilidade**
- Estrutura preparada para crescimento
- Padrões consistentes do NestJS
- Microserviços friendly

## Padrões Utilizados

### 1. **Repository Pattern**
- Abstração de acesso a dados
- Implementações específicas por ORM
- Contratos claros via interfaces

### 2. **Service Layer**
- Lógica de negócio centralizada
- Orquestração de operações
- Validações e regras de domínio

### 3. **Dependency Injection**
- Inversão de controle via NestJS
- Configuração via decorators
- Facilita testes e mocks

### 4. **DTO Pattern**
- Validação de entrada
- Transformação de dados
- Serialização de saída

### 5. **Ports and Adapters**
- Interfaces definem contratos (Ports)
- Implementações são adaptadores (Adapters)
- Domínio independente de detalhes técnicos

## Tecnologias Utilizadas

- **Framework**: NestJS
- **Linguagem**: TypeScript  
- **ORM**: Prisma
- **Banco**: PostgreSQL
- **Testes**: Jest (configurado)
- **Linting**: ESLint + Prettier

## Configuração e Execução

### Pré-requisitos
- Node.js 18+
- PostgreSQL
- Docker (opcional)

### Instalação
```bash
# Clone o repositório
git clone <repository-url>
cd ports-adapters

# Instale as dependências
npm install

# Suba o banco PostgreSQL com Docker
docker-compose up -d

# Execute as migrações do banco
npx prisma migrate dev

# Inicie a aplicação
npm run start:dev
```

### Variáveis de Ambiente
```env
# PostgreSQL database configuration
PG_USER=postgres
PG_PASSWORD=postgres123
PG_DB=ports_adapters_db
PG_PORT=5432
PG_HOST=localhost
DATABASE_URL="postgresql://${PG_USER}:${PG_PASSWORD}@${PG_HOST}:${PG_PORT}/${PG_DB}"
```

### Docker
O projeto inclui configuração Docker para PostgreSQL:

```bash
# Subir o banco PostgreSQL
docker-compose up -d

# Verificar status
docker-compose ps

# Ver logs
docker-compose logs postgres

# Parar o banco
docker-compose down
```

### Scripts Disponíveis
```bash
# Desenvolvimento
npm run start:dev

# Produção
npm run build
npm run start:prod

# Testes
npm run test
npm run test:e2e
npm run test:cov

# Banco de dados
npx prisma studio
npx prisma migrate dev
npx prisma generate
```

## Exemplo de Implementação

### Criando o módulo de Users

#### 1. Definir a Entidade
```typescript
// src/core/domain/user.entity.ts
export class User {
  constructor(
    public readonly id: number,
    public readonly name: string,
    public readonly email: string,
    public readonly password: string
  ) {}

  // Factory method para criação (sem ID)
  static create(name: string, email: string, password: string): User {
    // Validações de domínio
    if (!email.includes('@')) {
      throw new Error('Invalid email format');
    }
    
    if (password.length < 6) {
      throw new Error('Password must be at least 6 characters');
    }

    // Para criação, ID será 0 (temporário)
    return new User(0, name, email, password);
  }

  // Factory method para reconstrução (com ID do banco)
  static fromDatabase(id: number, name: string, email: string, password: string): User {
    return new User(id, name, email, password);
  }
}
```

#### 2. Definir Exceptions do Domínio
```typescript
// src/core/exceptions/user-not-found.exception.ts
export class UserNotFoundException extends Error {
  constructor(userId: number) {
    super(`User with ID ${userId} not found`);
    this.name = 'UserNotFoundException';
  }
}

// src/core/exceptions/user-already-exists.exception.ts
export class UserAlreadyExistsException extends Error {
  constructor(email: string) {
    super(`User with email ${email} already exists`);
    this.name = 'UserAlreadyExistsException';
  }
}
```

#### 3. Criar o Port do Repository
```typescript
// src/core/ports/repositories/user.repository.ts
export interface UserRepository {
  create(user: User): Promise<User>;
  findById(id: number): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  update(id: number, data: Partial<CreateUserData>): Promise<User>;
}
```

#### 3. Criar o Port do Service (Interface)
```typescript
// src/core/ports/services/user.service.ts
export interface IUserService {
  createUser(data: CreateUserData): Promise<User>;
  getUserById(id: number): Promise<User>;
  getUserByEmail(email: string): Promise<User>;
  changePassword(userId: number, oldPassword: string, newPassword: string): Promise<void>;
}
```

#### 4. Implementar o Service
```typescript
// src/core/services/user.service.ts
@Injectable()
export class UserService implements IUserService {
  constructor(private readonly userRepository: UserRepository) {}

  async createUser(data: CreateUserData): Promise<User> {
    // Verificar se email já existe
    const existingUser = await this.userRepository.findByEmail(data.email);
    if (existingUser) {
      throw new UserAlreadyExistsException(data.email);
    }

    // Criar entidade do domínio
    const user = User.create(data.name, data.email, data.password);
    
    // Persistir no banco
    return await this.userRepository.create(user);
  }

  async getUserById(id: number): Promise<User> {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new UserNotFoundException(id);
    }
    return user;
  }
}
```

#### 5. Criar o Adapter (Repository Implementation)
```typescript
// src/infra/repositories/prisma-user.repository.ts
@Injectable()
export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaService) {}

  async create(user: User): Promise<User> {
    const createdUser = await this.prisma.user.create({
      data: {
        name: user.name,
        email: user.email,
        password: user.password
      }
    });
    
    // Usa o factory method para reconstruir com ID do banco
    return User.fromDatabase(createdUser.id, createdUser.name, createdUser.email, createdUser.password);
  }

  async findById(id: number): Promise<User | null> {
    const userData = await this.prisma.user.findUnique({ where: { id } });
    if (!userData) return null;
    
    // Reconstroi a entidade a partir dos dados do banco
    return User.fromDatabase(userData.id, userData.name, userData.email, userData.password);
  }
}
```

#### 6. Configurar o Controller (usa a interface do service)
```typescript
// src/presentation/controllers/user.controller.ts
@Controller('users')
export class UserController {
  constructor(private readonly userService: IUserService) {} // Usa a interface!

  @Post()
  async create(@Body() createUserDto: CreateUserDto) {
    return this.userService.createUser(createUserDto);
  }

  @Get(':id')
  async getById(@Param('id') id: string) {
    return this.userService.getUserById(+id);
  }
}
```

## Configuração do NestJS Module

Após implementar todas as camadas do core e infraestrutura, é necessário configurar o módulo do NestJS para fazer a injeção de dependências:

### UserModule
```typescript
// src/modules/user.module.ts
import { Module } from '@nestjs/common';
import { UserController } from '../../presentation/controllers/user.controller';
import { UserService } from '../../core/services/user.service';
import { IUserService } from '../../core/ports/services/user-service.interface';
import { UserRepository } from '../../core/ports/repositories/user.repository';
import { PrismaUserRepository } from '../../infra/repositories/prisma-user.repository';
import { PrismaModule } from '../../infra/prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [UserController],
  providers: [
    {
      provide: IUserService,
      useClass: UserService,
    },
    {
      provide: UserRepository,
      useClass: PrismaUserRepository,
    },
  ],
  exports: [IUserService],
})
export class UserModule {}
```

> **Nota**: O módulo conecta todas as camadas, respeitando os contratos definidos pelas interfaces (ports) e mantendo a inversão de dependências.

## Conclusão

Esta arquitetura hexagonal proporciona uma base sólida para desenvolvimento de aplicações backend escaláveis e maintíveis. A separação clara de responsabilidades, combinada com os recursos do NestJS, oferece uma experiência de desenvolvimento produtiva e um código de alta qualidade.

A estrutura permite fácil evolução do projeto, adição de novas funcionalidades e manutenção a longo prazo, mantendo sempre a independência do domínio em relação aos detalhes de implementação.
