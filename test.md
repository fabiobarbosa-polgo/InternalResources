# GetUserProfile

## Descrição

Query responsável por retornar o perfil completo do usuário autenticado, garantindo que o usuário tenha acesso à empresa especificada no contexto da requisição

**Para Suporte/Onboarding**: Esta query é usada internamente pela rota `getProfile` para retornar os dados do usuário autenticado. Ela verifica automaticamente se o usuário tem acesso à empresa informada antes de retornar os dados. É útil para visualizar informações completas do perfil de um usuário, incluindo todas as empresas associadas.

## Funcionalidade

Esta query realiza as seguintes operações:

1. **Validação de Autenticação**: Verifica se o usuário está autenticado
2. **Validação de CompanyId**: Verifica se o cabeçalho `x-company-id` foi informado
3. **Busca do Usuário**: Busca o usuário pelo ID da sessão autenticada
4. **Validação de Acesso**: Verifica se o usuário possui acesso à empresa informada no contexto
5. **Retorno do Perfil**: Retorna os dados completos do perfil do usuário

## Validações

- O usuário deve estar autenticado (sessão válida)
- O cabeçalho `x-company-id` é obrigatório
- O usuário deve estar associado à empresa informada no contexto (`companyIds` do usuário deve conter o `companyId` do contexto)

## Input (Payload)

Esta query **não requer payload no body**. Todos os dados necessários vêm do contexto da requisição:
- `userId`: Extraído do token JWT da sessão autenticada
- `companyId`: Informado no cabeçalho `x-company-id`

### Estrutura do Input

Esta query não requer payload no body. Todos os dados necessários são obtidos do contexto da requisição (sessão autenticada e cabeçalhos).

## Output (Resposta)

Retorna um objeto `UserCreatedOrUpdated` com todos os dados do perfil do usuário.

### Estrutura do Output

```typescript
{
  id: string;                    // ID único do usuário
  name: string;                  // Nome completo do usuário
  email: string;                 // Email do usuário
  role: "PARTICIPANT" | "CUSTOMER" | "ADMIN"; // Role do usuário
  isActive: boolean;             // Status de ativação da conta
  companyIds?: string[];         // Array de IDs das empresas associadas
  emailConfirmed?: boolean;      // Indica se o email foi confirmado
  emailConfirmedAt?: Date | null; // Data/hora da confirmação do email
  phone?: string;                // Telefone do usuário (se cadastrado)
  profilePhotoUrl?: string;      // URL da foto de perfil (se cadastrada)
  createdAt: Date;               // Data/hora de criação da conta
  updatedAt: Date;              // Data/hora da última atualização
}
```

### Exemplo de Output

```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "João Silva",
  "email": "joao@example.com",
  "role": "PARTICIPANT",
  "isActive": true,
  "companyIds": [
    "507f1f77bcf86cd799439012",
    "507f1f77bcf86cd799439013"
  ],
  "emailConfirmed": true,
  "emailConfirmedAt": "2024-01-10T08:00:00.000Z",
  "phone": "+5511999999999",
  "profilePhotoUrl": "https://example.com/photo.jpg",
  "createdAt": "2024-01-01T10:00:00.000Z",
  "updatedAt": "2024-01-15T14:30:00.000Z"
}
```

### Campos da Resposta - Detalhamento

| Campo | Tipo | Descrição | Observações |
|-------|------|-----------|-------------|
| `id` | string | ID único do usuário | Identificador único do usuário no sistema |
| `name` | string | Nome completo | Nome cadastrado pelo usuário |
| `email` | string | Email do usuário | Email usado para login |
| `role` | enum | Role/perfil do usuário | Valores: "PARTICIPANT", "CUSTOMER", "ADMIN" |
| `isActive` | boolean | Status de ativação | `true` = conta ativa, `false` = conta desativada |
| `companyIds` | string[] | IDs das empresas associadas | Lista de todas as empresas que o usuário tem acesso |
| `emailConfirmed` | boolean | Email confirmado | `true` = email verificado, `false` = não verificado |
| `emailConfirmedAt` | Date/null | Data da confirmação | `null` se o email não foi confirmado |
| `phone` | string/null | Telefone cadastrado | `null` se não foi informado |
| `profilePhotoUrl` | string/null | URL da foto | `null` se não foi informada |
| `createdAt` | Date | Data de criação | Timestamp de quando a conta foi criada |
| `updatedAt` | Date | Data de atualização | Timestamp da última modificação |

## Erros Possíveis

| Código de Erro | Mensagem | Causa | Solução |
|----------------|----------|-------|---------|
| `UNAUTHORIZED` | "Você precisa estar autenticado para acessar este recurso" | Token JWT não informado, inválido ou expirado | Verificar se o token está sendo enviado corretamente no cabeçalho `Authorization` |
| `BAD_REQUEST` | "O cabeçalho x-company-id é obrigatório e deve ser um ID válido" | Cabeçalho `x-company-id` não informado ou vazio | Garantir que o cabeçalho `x-company-id` está presente na requisição |
| `NOT_FOUND` | "Usuário não encontrado" | Usuário do token não existe no banco de dados | Verificar se a conta foi deletada ou se há problema com o token |
| `FORBIDDEN` | "Você não tem permissão para acessar este recurso" | Usuário não está associado à empresa informada no `x-company-id` | Verificar se o usuário está vinculado à empresa correta ou usar outro `companyId` |

## Observações Importantes

- Esta query é protegida e requer autenticação via token JWT
- O acesso é validado verificando se o `companyId` informado no cabeçalho está presente no array `companyIds` do usuário
- O usuário identificado é obtido automaticamente do token de autenticação
